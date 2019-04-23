# windows下安卓环境的搭建
> 1. 安装node(版本高于8.3)
> 2. 全局安装react-native命令行工具(npm install react-native-cli -g)
> 3. 安装python2(注意版本必须是2.x)
> 4. 安装[jdk](https://www.oracle.com/technetwork/java/javase/downloads/jdk8-downloads-2133151.html),jdk的版本必须是1.8
> 5. 安装[android sdk](https://reactnative.cn/docs/getting-started.html)
> 6. 安装夜神模拟器
# 项目的启动
> 1. 启动夜神模拟器，[模拟器端口号](https://blog.csdn.net/u013383596/article/details/84561036)
> 2. 在命令行中输入: adb connect 127.0.0.1:62001 用以连接模拟器， 输入adb devices可以检测是否成功连接
> 3. 进入项目根目录，进入命令行输入: react-native run-android