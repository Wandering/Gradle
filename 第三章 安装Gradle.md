# 第3章安装Gradle
**目录**
=====
[3.1.先决条件](#3.1.先决条件)\
[3.2.下载](#3.2.-下载)\
[3.3.开箱](#3.3.-开箱)\
[3.4.环境变量](#3.4.-环境变量)\
[3.5.运行和测试您的安装](#3.5.-运行和测试您的安装)\
[3.6.JVM选项](#3.6.-jvm选项)

##3.1.先决条件
Gradle需要安装Java JDK或JRE，版本7或更高（要检查，使用java -version）。Gradle自带Groovy库，因此Groovy不需要安装。Gradle将忽略任何现有的Groovy安装。

Gradle使用它在路径中找到的任何JDK。或者，您可以将JAVA_HOME环境变量设置为指向所需JDK的安装目录。

##3.2. 下载
您可以从[Gradle网站](https://gradle.org/install/)下载其中一个Gradle发行版。

##3.3. 开箱
Gradle发行版以ZIP格式打包。完整的发行版包含：

* Gradle二进制文件。

* 用户指南（HTML和PDF）。

* DSL参考指南。

* API文档（Javadoc）。

大量的示例，包括用户指南中引用的示例，以及一些完整和更复杂的构建，您可以使用它们作为自己构建的起点。

二进制来源。这仅供参考。如果您想构建Gradle，您需要下载源代码发布版或从源代码库签出源代码。有关详细信息，请参阅[Gradle网站](https://gradle.org/resources/)。

##3.4. 环境变量
为了运行Gradle，首先添加环境变量GRADLE_HOME。这应该指向Gradle网站的解压缩文件。接下来添加GRADLE_HOME/bin到你的PATH环境变量。通常，这足以运行Gradle。

##3.5. 运行和测试您的安装
你通过gradle命令运行Gradle 。要检查Gradle是否正确安装，只需键入gradle -v。输出显示了Gradle版本以及本地环境配置（Groovy，JVM版本，OS等）。显示的Gradle版本应与您下载的发行版相匹配。

##3.6. JVM选项
运行Gradle的JVM选项可以通过环境变量来设置。您可以使用GRADLE_OPTS或JAVA_OPTS，或两者兼而有之。JAVA_OPTS通常是由许多Java应用程序共享的环境变量。一个典型的用例是将HTTP代理放入JAVA_OPTS和内存选项中GRADLE_OPTS。这些变量也可以在开始时设置gradle或gradlew脚本。

请注意，目前无法在命令行上为Gradle设置JVM选项。