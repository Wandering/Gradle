# 第五章Gradle控制台
**目录**
=======
[5.1. 概观](#5.1.-概观)\
[5.2. 命令行反馈](#5.2.-命令行反馈)\
[5.3. 在非交互式环境中的外观和感觉](#5.3.-在非交互式环境中的外观和感觉)\
##5.1. 概观
几乎每个Gradle用户都会在某个时候体验命令行界面。Gradle的控制台输出针对渲染性能和可用性进行了优化，只显示相关信息并提供视觉吸引力的反馈。

####图5.1.Gradle命令行正在运行
![Gradle命令行正在运行](https://docs.gradle.org/4.4.1/userguide/img/console-animation.gif "Gradle命令行正在运行")


##5.2. 命令行反馈
Gradle在构建运行时显示信息，以便您可以专注于最重要的项目。Gradle的控制台输出的每个部分都有助于回答特定的问题。

* [有什么我应该知道我的构建现在，例如测试失败，并发出警告？](#图5.2.-构建gradle命令行的输出部分)

* [我的构建何时完成？](#图5.3.-构建gradle命令行的进度条部分)

* [Gradle现在在做什么？](#图5.4.-gradle命令行的正在进行的部分)

* [还有其他有趣的结果，例如被跳过或最新的任务吗？](#图5.5.-构建gradle命令行的进度条部分)

##5.2.1. 建立输出
构建脚本日志消息，任务，分叉进程，测试输出和编译警告的输出显示在构建进度栏上方。

####图5.2. 构建Gradle命令行的输出部分
![构建Gradle命令行的输出部分](https://docs.gradle.org/4.4.1/userguide/img/console-build-output.jpg "构建Gradle命令行的输出部分")

构建Gradle命令行的输出部分
从Gradle 4.0开始，命令行控制台输出的数量已经减少。每个任务的开始和结束不再显示或任务的结果（例如`UP-TO-DATE`）。只有在任务执行期间发出一些输出时才显示任务的名称。Gradle也将源自特定上下文的输出分组在一起，例如来自编译任务的所有警告，测试执行或分支进程。分组输出对并行任务执行特别有用，因为它可以防止没有明确指出其来源的交错消息（请参见[第26.8节“并行项目执行”](https://docs.gradle.org/4.4.1/userguide/multi_project_builds.html#sec:parallel_execution)）。

>Note:分组控制台输出和减少的控制台输出仅在交互式和丰富的控制台命令行中发生。持续集成服务器和构建使用`--console=plain`将会看到类似于Gradle 4.0之前的控制台输出。您也可以通过`org.gradle.console`属性设置该选项，请参见[第12.1节 通过gradle.properties配置构建环境](https://docs.gradle.org/4.4.1/userguide/build_environment.html#sec:gradle_configuration_properties)。

以下控制台输出显示配置阶段和任务的分组输出`:compileJava:`

    > Configure project ':library'
    Configuring project version for project ':library'
    
    > Configure project ':consumer'
    Configuring project version for project ':consumer'
    
    > Task :compileJava
    Note: Some input files use unchecked or unsafe operations.
    Note: Recompile with -Xlint:unchecked for details.
在显示输出之前，Gradle不会等待一个工作单元完成。Gradle会在短时间内将输出刷新到控制台，以确保相关信息尽快可用。并行构建时，长时间运行的任务的输出可以被其他任务分解。每个控制台输出块将清楚地指明它属于哪个任务。
    
    > Task :compileJava
    Note: Some input files use unchecked or unsafe operations.
    Note: Recompile with -Xlint:unchecked for details.
    
    > Task :generateCode
    Generating JAXB classed from XSD files.
    
    > Task :compileJava
    Note: Some input files use or override a deprecated API.
    Note: Recompile with -Xlint:deprecation for details.
##5.2.2. 建立进度条
构建进度条为您提供了一个非常快速的方法，知道构建是否即将完成。在构建执行工作时，进度栏将从左到右填充。在任何时候，构建进度条还会呈现构建生命周期的当前阶段（请参见第22.1节“构建阶段”）以及构建期间的总体时间。

####图5.3. 构建Gradle命令行的进度条部分
![构建Gradle命令行的进度条部分](https://docs.gradle.org/4.4.1/userguide/img/console-build-progress-bar.jpg "构建Gradle命令行的进度条部分")
构建Gradle命令行的进度条部分
以下示例显示了构建生命周期的初始化，配置和执行阶段的进度栏：

    <-------------> 0% INITIALIZING [2s]
    <==-----------> 25% CONFIGURING [4s]
    <=========----> 64% EXECUTING [17s]
###5.2.3. 正在进行显示
Gradle提供了直接在构建进度条下执行的实际工作的细粒度视图。每行代表一个线程或进程，可以并行执行工作 - 解决依赖关系，执行任务和运行测试。如果一个可用的工人没有被使用，那么它被标记IDLE。可用工作者的数量默认为执行构建的机器上的处理器数量。

####图5.4. Gradle命令行的正在进行的部分
![Gradle命令行的正在进行的部分](https://docs.gradle.org/4.4.1/userguide/img/console-work-in-progress.jpg "Gradle命令行的正在进行的部分")

Gradle命令行的正在进行的部分
并行测试执行只显示在Gradle核心（例如JUnit和TestNG）支持的基于JVM的测试中。未来的Gradle版本可能支持其他测试工具和框架。

控制台输出的以下部分显示了8个并发工作人员正在进行的工作：

    <==========---> 77% EXECUTING [10s]
    > :codeQuality:classpathManifest > Resolve dependencies :codeQuality:runtimeClasspath
    > :ivy:classpathManifest > Resolve dependencies :ivy:runtimeClasspath
    > IDLE
    > :antlr:classpathManifest > Resolve dependencies :antlr:runtimeClasspath
    > :scala:compileJava > Resolve dependencies :scala:compileClasspath
    > :buildInit:classpathManifest > Resolve dependencies :buildInit:runtimeClasspath
    > :jacoco:classpathManifest > Resolve dependencies :jacoco:runtimeClasspath
    > IDLE
###5.2.4. 建立结果
在构建结束时，Gradle将显示构建的结果（成功或失败）以及执行工作和避免工作的任务数量。构建结果还会显示执行构建所花费的总体时间。执行任务的数量提供了构建过时或繁忙的指示。

####图5.5. 构建Gradle命令行的进度条部分
![构建Gradle命令行的进度条部分](https://docs.gradle.org/4.4.1/userguide/img/console-build-result.jpg "构建Gradle命令行的进度条部分")

构建Gradle命令行的进度条部分
以下构建结果表示构建的成功以及包括其状态在内的任务数量：

    BUILD SUCCESSFUL in 2m 10s
    411 actionable tasks: 381 executed, 30 up-to-date
“Actionable”(可执行)任务至少有一个动作。生命周期任务`build`（也称为聚合任务）不声明任何操作，因此不可操作。

##5.3. 在非交互式环境中的外观和感觉
默认情况下，Gradle尝试通过检测构建运行的控制台的类型来启用丰富的控制台输出。这使颜色和其他控制台输出格式。非交互式环境回退到使用纯控制台输出。普通输出格式不支持输出分组。任务和结果始终打印为与Gradle 3.x版本一致。

>Note:从IDE（例如Buildship和IntelliJ）或持续集成产品（例如Jenkins和TeamCity）执行的Gradle构建默认使用纯控制台输出。

以下输出演示使用纯控制台：

    :compileJava
    Note: Some input files use unchecked or unsafe operations.
    Note: Recompile with -Xlint:unchecked for details.
    :processResources
    :classes
    :jar
    :assemble
    :compileTestJava NO-SOURCE
    :processTestResources NO-SOURCE
    :testClasses UP-TO-DATE
    :test NO-SOURCE
    :check UP-TO-DATE
    :build
    
    BUILD SUCCESSFUL in 6s
    11 actionable tasks: 6 executed, 5 up-to-date