#### flutter 内存泄露检测

1. 使用profile模式启动app，打开devTools

   链接类似如

   ```
   http://127.0.0.1:49728/#/?ide=Android-Studio&uri=http%253A%252F%252F127.0.0.1%253A55852%252F06aYEDuPk-Q%253D
   ```

   取出uri 参数，做两次urlDecode ,得到原始路径,类似

   ```
   http://127.0.0.1:55852/06aYEDuPk-Q=
   ```

在浏览器打开

点击底部的 "allocation profile"

记得点击GC后再查具体实例个数

在底部ctrl+F 查找具体类型，点击进去。
查找字符串 currently allocated	count
点开 strongly reachable会显示每个实例，点击其中任何一个。进入该实例详情。
进入后点击 Retaining path，就能查到引用路径了。


