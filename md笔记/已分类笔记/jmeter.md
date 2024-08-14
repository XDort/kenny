#### 安装最新版Jmeter问题

已经配置好新版JDK8和环境变量

Error: VM option 'UseG1GC' is experimental and must be enabled via -XX:+UnlockExperimentalVMOptions.

Error: Could not create the Java Virtual Machine.

Error: A fatal exception has occurred. Program will exit.

按照提示，编辑jmeter文件，找到GC_ALGO:="

添加 -XX:+UnlockExperimentalVMOptions

然后可以正常启动GUI模式的jmeter