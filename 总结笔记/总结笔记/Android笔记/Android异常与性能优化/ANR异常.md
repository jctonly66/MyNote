##ANR
####一. 什么是ANR？
Application Not Responding，页面无响应的对话框

####二. 发生ANR的条件

首先，应用程序的响应性是由ActivityManager和WindowManager系统服务监视的，当ANR发生条件满足时，就会弹出ANR的对话框。在以下三种情况会发生ANR异常：

- Activity超过5秒无响应
- BroadcastReceiver超过10秒无响应
- Service超过20秒无响应

####三. 造成ANR的主要原因

- 主线程被IO操作阻塞
- 主线程存在耗时操作

在Android中，在主线程执行的主要有：

- Activity的所有生命周期回调都是执行在主线程的
- Service默认执行在主线程中
- BoardcastReceiver的回调onReceive()执行在主线程中
- AsyncTask的回调除了doInBackground，其他都是在主线程中
- 没有使用子线程Looper的Handler的handlerMessage，post(Runnable)都是执行在主线程中

####四. 如何解决ANR
耗时的操作放在子线程去执行：

- 使用AsyncTask处理耗时IO操作
- 使用Thread或HandlerThread并且降低优先级
- 使用Handler处理工作线程的耗时操作
- Activity的onCreate和onResume回调尽量避免耗时操作