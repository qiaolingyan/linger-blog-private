vs code快捷键

ctrl+k  ctrl+0   快速折叠代码





serve .   启动http窗口





两个 script 标签，，会执行完第一个 script 标签里的同步代码和微任务才会去执行第二个 script 标签里的代码，，所以 script  相当于一个宏任务

script 标签1 的 同步代码 -》 script 标签1 的 微任务 -》script 标签2 的 同步代码 -》 script 标签2 的 微任务 -》script 标签1 与 script 标签2 的宏任务