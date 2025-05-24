---
创建时间: 2025-05-07 10:32:32
作者: wangxiaoming
tags:
  - Android
  - AsyncTask
---
#### 一、解决的问题
- **背景**​：Android 中 `UI` 操作必须在主线程执行，而耗时任务（如网络请求、文件读写）需在子线程运行，传统方式需手动管理线程和 `Handler`，代码复杂。
- ​**目标**​：简化异步任务流程，提供 ​**后台执行 → 进度更新 → 结果回调**​ 的一站式解决方案。
#### 二、原理
- **本质**​：封装了线程池（`ThreadPoolExecutor`）和 `Handler`，通过回调方法实现线程切换。
- ​**关键组件**​：
    - ​**`WorkerRunnable`**​：实现 `Callable` 接口，封装后台任务。
    - ​**`FutureTask`**​：管理任务状态和结果，与 `WorkerRunnable` 绑定。
    - ​**`SerialExecutor`**​：默认串行执行器，任务按提交顺序执行（Android 3.0+）。
    - ​**`InternalHandler`**​：将结果从子线程传递到主线程。
- ​**执行流程**​：
    1. `execute()` 触发任务，调用 `onPreExecute()`（主线程）。
    2. 子线程执行 `doInBackground()`。
    3. 通过 `publishProgress()` 触发 `onProgressUpdate()`（主线程）。
    4. 任务完成后调用 `onPostExecute()`（主线程）。
#### 三、使用方法
```java
	public class DownloadTask extends AsyncTask<String,Integer,String> {
	    @Override
	    protected void onPreExecute() {
	       //显示进度条（主线程）
	    }
        @Override
	    protected String doInBackground(String... urls) {
		    //下载文件（子线程）
		    publishProgress(50);
		    return "下载完成";
	    }
	    @Override
	    protected void onProgressUpdate(Integer... values) {
		    //更新进度条（主线程）
	    }
	    @Override
	    protected void onPostExecute(String result) {
		    //隐藏进度条，显示结果（主线程）
	    }
	}

```
#### 四、注意事項
- ​**内存泄漏**​：非静态内部类持有外部 `Activity` 引用，需改用静态内部类 + `WeakReference`。
- ​**生命周期**​：`AsyncTask` 不绑定 `Activity` 生命周期，需在 `onDestroy()` 调用 `cancel(true)`。
- ​**结果丢失**​：配置变更（如屏幕旋转）可能导致 `Activity` 重建，需通过 `ViewModel` 或保存状态处理。
- ​**单次执行**​：同一实例只能执行一次，多次调用会抛出 `IllegalStateException`。
#### 五、优化点
- **线程池配置**​：通过 `executeOnExecutor(THREAD_POOL_EXECUTOR)` 改为并行执行。
- ​**任务取消**​：定期检查 `isCancelled()`，避免无效计算。
- ​**替代方案**​：复杂任务使用 `RxJava`、`Coroutines` 或 `WorkManager`。

#### 六、适用场景
|​**场景**​|​**说明**​|
|---|---|
|短时后台任务|如图片加载、简单网络请求（耗时 < 1 秒）。|
|进度更新需求|需要实时显示进度条或加载动画。|
|轻量级 UI 交互|需在任务完成后更新少量 UI 元素（如显示结果文本）。|
**不适用场景**​：
- ​**长时间任务**​：如大数据处理、复杂计算（易导致 `ANR`）。
- ​**高并发需求**​：默认串行执行，需手动切换线程池。
- ​**严格生命周期绑定**​：需与 `Lifecycle` 组件（如 `ViewModel`）配合。

#### 七、面试考察点
1. ​**与 `Thread` + `Handler` 的区别**​：
    - `AsyncTask` 封装了线程管理和消息传递，代码更简洁。
    - `Thread` 需手动创建和销毁，`Handler` 需处理消息队列。
2. ​**内部线程池参数**​：
    - 默认核心线程数 5，最大线程数 128，任务队列容量 10（Android 3.0+）。
3. ​**为何被弃用**​：
    - 生命周期管理缺陷、内存泄漏风险、无法适配 `Jetpack` 组件。
4. ​**与 `RxJava`/`Coroutines` 对比**​：
    - `AsyncTask` 仅适合简单场景，`RxJava` 支持复杂流式操作，`Coroutines` 更轻量且与协程集成。