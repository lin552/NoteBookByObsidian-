---
创建时间: 2025-08-11 19:03:46
作者: wangxiaoming
tags:
  - Fragment
---
`FragmentManager`是 Android 系统中管理 ​**Fragment 生命周期**​ 和 ​**动态事务操作**​ 的核心组件，而 ​**Fragment 事务（`FragmentTransaction`）​**​ 是其提供的操作接口，用于定义 Fragment 的添加、替换、移除等行为。两者的协同工作实现了 Android 动态 `UI` 的核心能力。以下从 ​**事务的本质**、**执行流程**、**回退栈管理**、**生命周期绑定**​ 四个维度深入解析其原理。

#### 一、事务的本质：操作指令的容器
`FragmentTransaction`是开发者与 `FragmentManager`交互的接口（如 `add()`、`replace()`、`remove()`），但其本质是 ​**操作指令的容器**。真正执行操作的是其内部实现类 `BackStackRecord`，它通过记录一系列 ​**操作指令（Op）​**​ 来描述需要执行的 Fragment 变更。
##### 1.1 事务的底层实现：`BackStackRecord`
`BackStackRecord`内部维护一个操作列表 `mOps`，每个操作是一个 `Op`对象，封装了具体的 Fragment 变更逻辑（如添加、替换）。例如：
```java
// BackStackRecord.java（简化）
class BackStackRecord extends FragmentTransaction {
    final ArrayList<Op> mOps = new ArrayList<>(); // 存储所有操作指令

    // 添加操作（如 add()、replace()）
    void addOp(Op op) {
        mOps.add(op);
    }

    // Op 类定义（简化）
    static class Op {
        int cmd; // 操作类型（OP_ADD=1, OP_REPLACE=2, OP_REMOVE=3...）
        Fragment fragment; // 目标 Fragment
        int containerViewId; // 容器 ID
        String tag; // Fragment 标签（用于查找）
        // 其他属性（如退出动画、入栈标识等）
    }
}
```
**关键特性**​：
- 事务操作是 ​**惰性执行**​ 的：所有 `add()`、`replace()`等调用仅将 `Op`存入 `mOps`，直到调用 `commit()`才触发实际执行。
- 事务是 ​**原子性**​ 的：同一事务中的多个操作（如先 `add(A)`再 `add(B)`）会作为一个整体执行，要么全部完成，要么失败（避免 `UI` 状态不一致）。
#### 二、事务的执行流程：从`commint()` 到界面更新
调用 `FragmentTransaction.commit()`后，事务需要经过 ​**状态检查**、**入队**、**异步执行**​ 三个阶段，最终触发 Fragment 的实际变更。
##### 2.1 阶段 1：状态检查（commit()）
`commit()`方法首先检查当前 Activity 的状态，避免在非法状态下提交事务（如 Activity 已销毁）：
```java
// FragmentTransaction.java
public int commit() {
    return commitInternal(false); // 不允许状态丢失
}

int commitInternal(boolean allowStateLoss) {
    if (mManager.mStateSaved || mManager.mStopped) {
        if (!allowStateLoss) {
            throw new IllegalStateException("Can not perform this action after onSaveInstanceState");
        }
        // 允许状态丢失时记录警告
    }
    mManager.enqueueAction(this, allowStateLoss); // 入队操作
    return mIndex; // 返回事务索引（用于后续引用）
}
```
**关键检查点**​：
 - `mManager.mStateSaved`：Activity 已调用 `onSaveInstanceState()`，此时提交事务可能导致状态丢失（如 Fragment 未保存数据）。
- `mManager.mStopped`：Activity 已停止（如被其他 Activity 覆盖），此时提交事务可能无效。
##### 2.2 阶段 2：入队操作（`enqueueAction`）
`FragmentManager.enqueueAction()`将事务加入待处理队列 `mPendingActions`，并通过 `Handler`触发异步执行：
```java
// FragmentManager.java
void enqueueAction(Action action, boolean allowStateLoss) {
    synchronized (mPendingActions) {
        if (mTmpRecords == null) {
            mTmpRecords = new ArrayList<>();
            mTmpIsPop = new ArrayList<>();
        }
        mTmpRecords.add(action instanceof BackStackRecord ? ((BackStackRecord) action).mRecord : action);
        mTmpIsPop.add(action instanceof BackStackRecord ? ((BackStackRecord) action).mIsPop : false);
    }
    // 通过主线程 Handler 触发执行
    mHandler.post(mExecCommit);
}
```
**设计目的**​：
- 避免在主线程同步执行事务（可能导致 `ANR`），通过异步批量处理提升流畅性。
- 支持合并连续事务（如短时间内多次 `commit()`），减少渲染次数。

##### 2.3 阶段 3：执行事务（`execPendingActions`）
`FragmentManager.execPendingActions()`是事务执行的最终入口，它会遍历 `mPendingActions`队列，逐个执行事务：
```java
// FragmentManager.java
void execPendingActions() {
    if (mExecutingActions) {
        return; // 防止递归执行
    }
    mExecutingActions = true;
    try {
        do {
            // 取出一个事务（BackStackRecord）
            final BackStackRecord record = ...;
            if (record != null) {
                executeOpsTogether(record, ...); // 执行事务中的所有 Op
            }
        } while (!mPendingActions.isEmpty());
    } finally {
        mExecutingActions = false;
    }
}
```
​**执行细节**​：
- `executeOpsTogether()`会遍历 `BackStackRecord.mOps`中的所有 `Op`，按顺序执行（如先 `add(A)`再 `replace(A, B)`）。
- 对于 `replace()`操作，会先移除容器中现有的 Fragment（触发其 `onDestroyView()`），再添加新的 Fragment（触发其 `onCreateView()`）。
#### 三、回退栈（`BackStack`）的管理机制
回退栈是 Fragment 事务的“历史记录”，用于支持用户通过系统返回键回退到上一个页面。`FragmentManager`通过 `mBackStack`列表存储所有可回退的事务（`BackStackRecord`）。
##### 3.1 事务入栈：`addToBackStack()`
调用 `FragmentTransaction.addToBackStack(String name)`时，事务会被标记为可回退，并加入 `mBackStack`：
```java
// BackStackRecord.java
public FragmentTransaction addToBackStack(String name) {
    if (!mAllowAddToBackStack) {
        throw new IllegalStateException("This transaction is not allowed to be added to back stack");
    }
    mAddToBackStack = true; // 标记为可回退
    mName = name; // 回退栈名称（可选）
    return this;
}

// FragmentManager.java（事务提交后）
if (transaction.mAddToBackStack) {
    mBackStack.add(transaction); // 将事务加入回退栈
    transaction.mIndex = mBackStack.size() - 1; // 记录索引
}
```
**关键属性**​：
- `mAddToBackStack`：标记事务是否可回退（默认 `false`）。
- `mName`：回退栈的标识符（可通过 `popBackStack(String name)`按名称回退）。
- `mIndex`：事务在回退栈中的位置（用于精准定位）。

##### 3.2 回退操作：`popBackStack()`
调用 `FragmentManager.popBackStack()`时，会从 `mBackStack`栈顶取出最近的事务，逆序执行其操作的逆操作（如 `replace(A, B)`的逆操作是 `remove(B)`或 `add(A)`）：
```java
// FragmentManager.java
boolean popBackStackImmediate(int id, int flags) {
    checkStateLoss(); // 检查状态丢失
    // 创建一个“弹出”事务（逆操作）
    BackStackRecord popRecord = new BackStackRecord(mManager);
    popRecord.mOps.addAll(mBackStack.get(backStackIndex).mOps); // 复制原事务操作
    popRecord.mIsPop = true; // 标记为弹出操作
    // 执行逆操作（如原操作是 OP_REPLACE，逆操作是 OP_REMOVE）
    execSingleAction(popRecord, false);
    return true;
}
```
**逆操作逻辑**​：
- 原操作是 `OP_ADD`→ 逆操作是 `OP_REMOVE`（移除添加的 Fragment）。
- 原操作是 `OP_REPLACE`→ 逆操作是 `OP_REMOVE`（移除新添加的 Fragment，恢复旧的 Fragment）。
- 原操作是 `OP_REMOVE`→ 逆操作是 `OP_ADD`（重新添加被移除的 Fragment）。

#### 四、事务对Fragment生命周期的影响
Fragment 的生命周期与事务操作强绑定，不同操作会触发不同的生命周期回调。以下是常见操作对生命周期的影响：

|**事务操作**​|旧 Fragment 生命周期|新 Fragment 生命周期|说明|
|---|---|---|---|
|​**add(container, A)​**​|无（若 A 未添加过）|`onAttach()`→ `onCreate()`→ ... → `onResume()`|A 被添加到容器，触发完整的生命周期。|
|​**replace(container, B)​**​|`onPause()`→ `onStop()`→ `onDestroyView()`|`onAttach()`→ `onCreate()`→ ... → `onResume()`|旧 Fragment（如 A）的视图被销毁，但实例保留（除非被移除）；新 Fragment（B）被创建并显示。|
|​**remove(A)​**​|`onPause()`→ `onStop()`→ `onDestroyView()`（可能后续 `onDestroy()`）|无（A 被移除）|A 被从容器中移除，视图销毁；若未被其他事务引用，实例可能被回收。|
|​**attach(A)​**​|`onCreateView()`→ `onViewCreated()`|无（A 已存在）|A 的视图被重新创建（适用于之前被 `detach()`的 Fragment）。|
|​**detach(A)​**​|`onDestroyView()`|无（A 仍存在）|A 的视图被销毁，但实例保留（适用于需要保留状态的场景）。

**关键点**​：
- `replace()`是最常用的操作，它通过销毁旧 Fragment 的视图（`onDestroyView()`）来释放资源，同时保留实例（除非被显式移除）。
- 事务加入回退栈时，旧 Fragment 的状态（如输入内容、滚动位置）会被保存，返回时通过 `onCreateView()`恢复视图（而非重新创建实例）。

#### 五、`FragmentManager`的核心数据结构
`FragmentManager`内部通过多个关键数据结构管理 Fragment 和事务：
##### 5.1 `mFragments`：存储所有Fragment实例
`FragmentManager.mFragments`是一个 `ArrayList<Fragment>`，存储当前被管理的所有 Fragment 实例（包括未显示的）。通过 `findFragmentById()`或 `findFragmentByTag()`可快速查找。
##### 5.2 `mBackStack`：存储可回退的事务
`FragmentManager.mBackStack`是一个 `ArrayList<BackStackRecord>`，存储所有标记为 `addToBackStack(true)`的事务，支持回退操作。
##### 5.3 `mActive`：记录活跃的Fragment
`FragmentManager.mActive`是一个 `SparseArray<Fragment>`，通过容器 ID 映射当前显示在该容器中的 Fragment（仅记录活跃状态的 Fragment）。