---
创建时间: 2025-07-28 12:47:24
作者: wangxiaoming
tags:
  - Activity
---
在 Android 中，`Intent` 的标志位（`Flags`）用于控制 `Activity` 的启动行为、任务栈管理、生命周期交互等。不同标志位可组合使用，实现复杂的导航逻辑。以下是 ​**Android 官方定义的所有 Intent 标志位**​（截至 API 34），按功能分类整理，并附作用说明、适用场景及注意事项：

#### 一、任务栈（Task）管理类
用于控制 `Activity` 如何关联到任务栈（Task），以及任务栈的创建、清理规则。

| 标志位                                   | API 级别      | 功能说明                                                                                                                                | 适用场景                                                  | 注意事项                                      |
| ------------------------------------- | ----------- | ----------------------------------------------------------------------------------------------------------------------------------- | ----------------------------------------------------- | ----------------------------------------- |
| `FLAG_ACTIVITY_NEW_TASK`              | 1           | 新建一个任务栈（或复用同 `taskAffinity` 的现有任务栈），将目标 `Activity` 作为新栈顶。                                                                           | 启动一个独立的界面（如从通知栏、桌面快捷方式启动应用）。                          | 默认情况下，同一应用的 `Activity` 共享任务栈，此标志位可强制跨任务栈。 |
| `FLAG_ACTIVITY_CLEAR_TASK`            | 11          | 清除当前任务栈中所有已存在的 `Activity`，仅保留刚启动的目标 `Activity`。                                                                                     | 登录过期后跳转登录页（需清空之前的业务页面）。                               | 需配合 `FLAG_ACTIVITY_NEW_TASK` 使用，否则无效。     |
| `FLAG_ACTIVITY_TASK_ON_HOME`          | 21          | 目标 `Activity` 成为任务的根，且任务会被视为“从主屏幕启动”（类似用户手动按 Home 键后返回）。                                                                            | 需要模拟“从桌面启动”的任务行为（如某些深度链接场景）。                          | 影响任务在最近任务列表中的显示归属。                        |
| `FLAG_ACTIVITY_NEW_DOCUMENT`          | 21          | （文档模式）启动一个新的任务栈，用于展示独立文档（如 PDF、网页），用户可通过最近任务列表单独管理。                                                                                 | 需要支持多文档的应用（如文档阅读器），替代 `FLAG_ACTIVITY_NEW_TASK` 的部分场景。 | 需配合 `android:documentLaunchMode` 使用效果更佳。  |
| `FLAG_ACTIVITY_RETAIN_IN_RECENTS`     | 21          | 即使任务被重置（如用户长时间未使用），目标 `Activity` 仍保留在最近任务列表中。                                                                                       | 需要确保重要页面在最近任务中可见（如设置页、主界面）。                           | 默认情况下，长时间未使用的任务会被系统清理，此标志位可保留。            |
| `FLAG_ACTIVITY_CLEAR_WHEN_TASK_RESET` | 已弃用（API 21） | 原功能：当任务被重置（如用户通过“最近任务”重新启动任务）时，清除该 `Activity` 之上的所有页面。<br>替代方案：使用 `FLAG_ACTIVITY_RETAIN_IN_RECENTS` 或 `FLAG_ACTIVITY_NEW_DOCUMENT`。 | 已过时，不建议使用。                                            | —                                         |
#### 二、启动模式与生命周期类
影响 `Activity` 的启动模式（如是否复用现有实例、是否触发 `onNewIntent()`）或生命周期回调。

| 标志位                                  | API 级别      | 功能说明                                                                           | 适用场景                                      | 注意事项                                                    |
| ------------------------------------ | ----------- | ------------------------------------------------------------------------------ | ----------------------------------------- | ------------------------------------------------------- |
| `FLAG_ACTIVITY_SINGLE_TOP`           | 1           | 如果目标 `Activity` 已经是当前任务栈的栈顶，​**不重新创建**，而是调用其 `onNewIntent()`。                  | 需要避免重复创建栈顶 `Activity`（如新闻详情页，避免重复加载数据）。   | 若目标 `Activity` 不在栈顶（如被其他 `Activity` 覆盖），仍会新建实例。         |
| `FLAG_ACTIVITY_CLEAR_TOP`            | 1           | 如果目标 `Activity` 已存在于当前任务栈中，​**清除其上方的所有 `Activity`**，并将其复用（触发 `onNewIntent()`）。 | 需要返回到已存在的 `Activity` 并清理中间页面（如从详情页返回列表页）。 | 若目标 `Activity` 是栈顶，则直接调用 `onNewIntent()`，不会清理自身。        |
| `FLAG_ACTIVITY_FORWARD_RESULT`       | 1           | 标记 `Activity` 用于转发结果（需配合 `startActivityForResult()`）。                          | A 启动 B，B 启动 C，最终由 C 的结果返回给 A（B 作为中间页）。    | 需在 B 中调用 `setResult()` 并 finish，结果会传递给 A。               |
| `FLAG_ACTIVITY_NO_HISTORY`           | 1           | 目标 `Activity` 不会被保留在任务栈中，退出后无法通过返回键找回。                                         | 临时界面（如广告页、引导页），无需保留历史。                    | 退出后，该 `Activity` 会被系统销毁，无法恢复状态。                         |
| `FLAG_ACTIVITY_EXCLUDE_FROM_RECENTS` | 11          | 目标 `Activity` 不会出现在“最近任务”列表中。                                                  | 敏感操作界面（如支付页、登录页），避免用户误操作找回。               | 需配合 `android:excludeFromRecents="true"` 在清单文件中声明，或动态设置。 |
| `FLAG_ACTIVITY_PREVIOUS_IS_TOP`      | 已弃用（API 29） | 原功能：如果目标 `Activity` 已经是栈顶，不重复创建。<br>替代方案：使用 `FLAG_ACTIVITY_SINGLE_TOP`。        | 已过时，不建议使用。                                | —                                                       |
#### 三、历史记录与导航类
控制 `Activity` 在任务栈中的历史记录行为，影响返回键逻辑。

|标志位|API 级别|功能说明|适用场景|注意事项|
|---|---|---|---|---|
|`FLAG_ACTIVITY_BRING_TO_FRONT`|1|如果目标 `Activity` 已在运行，将其带到前台（不重新创建）；否则正常启动。|需要快速激活已存在的 `Activity`（如实时聊天界面，避免重复加载）。|行为类似 `FLAG_ACTIVITY_REORDER_TO_FRONT`，但不会调整栈中其他 `Activity` 顺序。|
|`FLAG_ACTIVITY_REORDER_TO_FRONT`|1|如果目标 `Activity` 已存在于任务栈中，将其移动到栈顶（不销毁其他 `Activity`）。|需要将已有 `Activity` 带到前台，同时保留中间页面（如从设置页返回之前的操作页）。|会改变任务栈中 `Activity` 的顺序，可能影响返回键逻辑。|
|`FLAG_ACTIVITY_NO_USER_ACTION`|1|标记 `Activity` 启动时**不触发用户的操作响应**​（如返回键）。|系统自动触发的 `Activity`（如崩溃恢复页），避免用户误操作。|仅在特殊场景使用，普通应用不建议滥用。|

#### 四、跨进程与多窗口类
用于控制 `Activity` 在多进程、多窗口模式下的启动行为。

|标志位|API 级别|功能说明|适用场景|注意事项|
|---|---|---|---|---|
|`FLAG_ACTIVITY_MULTIPROCESS`|1|允许 `Activity` 在另一个进程中运行（已弃用）。|早期多进程方案（如浏览器标签页），现推荐使用 `android:process` 属性声明。|已弃用（API 21），可能导致进程间通信复杂化，不建议使用。|
|`FLAG_ACTIVITY_LAUNCH_ADJACENT`|21|（多窗口模式）启动一个相邻的 `Activity`，与当前 `Activity` 并排显示（需系统支持多窗口）。|多窗口模式下需要分屏显示两个界面（如一边聊天一边看视频）。|需系统支持多窗口（如平板或折叠屏设备），手机端可能无效。|

#### 五、其他特殊用途标志位
|标志位|API 级别|功能说明|适用场景|注意事项|
|---|---|---|---|---|
|`FLAG_RECEIVER_REGISTERED_ONLY`|1|（广播专用）仅向已注册的 `BroadcastReceiver` 发送广播（静态注册的不接收）。|避免静态广播被恶意接收（如敏感操作通知）。|仅适用于 `sendBroadcast(Intent)` 发送广播的场景。|
|`FLAG_ACTIVITY_LAUNCHED_FROM_HISTORY`|系统内部|标记 `Activity` 是从历史记录中启动的（系统自动设置，开发者无需手动使用）。|系统内部管理历史记录，开发者无需干预。|—|
#### 六、常见标志位组合示例
实际开发中，标志位常组合使用以实现复杂逻辑：
##### 1. 登录过期跳转登录页（清空历史栈）
```java
// 组合 FLAG_ACTIVITY_NEW_TASK + FLAG_ACTIVITY_CLEAR_TASK
Intent loginIntent = new Intent(context, LoginActivity.class);
loginIntent.addFlags(Intent.FLAG_ACTIVITY_NEW_TASK | Intent.FLAG_ACTIVITY_CLEAR_TASK);
startActivity(loginIntent);
```
​**效果**​：新建任务栈并启动登录页，原任务栈中所有 `Activity` 被清除，用户按返回键退出应用。

##### 2.返回已存在的 `Activity` 并清理中间页
```java
// 组合 FLAG_ACTIVITY_CLEAR_TOP + FLAG_ACTIVITY_SINGLE_TOP（可选）
Intent backIntent = new Intent(context, MainActivity.class);
backIntent.addFlags(Intent.FLAG_ACTIVITY_CLEAR_TOP);
startActivity(backIntent);
```
**效果**​：若 `MainActivity` 在栈中，清除其上方的所有 `Activity` 并复用；若在栈顶，直接触发 `onNewIntent()`。

##### 3.启动临时广告页（不保留历史）
```java
Intent adIntent = new Intent(context, AdActivity.class);
adIntent.addFlags(Intent.FLAG_ACTIVITY_NO_HISTORY);
startActivity(adIntent);
```
**效果**​：广告页退出后，无法通过返回键回到广告页，直接回到之前的界面。