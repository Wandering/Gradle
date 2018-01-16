# 第四章使用Gradle命令行
**目录**
===============
[4.1. 执行多个任务](#4.1.-执行多个任务)\
[4.2. 排除任务](#4.2.-排除任务)\
[4.3. 在发生故障时继续构建](#4.3.-在发生故障时继续构建)\
[4.4. 任务名称缩写](#4.4.-任务名称缩写)\
[4.5. 选择要执行的构建](#4.5.-选择要执行的构建)\
[4.6. 强制任务执行](#4.6.-强制任务执行)\
[4.7. 获取有关您的构建的信息](#4.7.-获取有关您的构建的信息)\
[4.8. Dry Run](#4.8.-dry-run)\
[4.9. 概要](#4.9.-概要)\
本章介绍Gradle命令行的基础知识。您可以使用gradle前面章节中已经看到的命令来运行构建。

####4.1. 执行多个任务
通过在命令行上列出每个任务，您可以在单个构建中执行多个任务。例如，该命令gradle compile test将执行compile和test任务。Gradle将按照它们在命令行中列出的顺序执行任务，并且还将执行每个任务的依赖关系。每个任务只被执行一次，无论它是如何包含在构建中的：它是在命令行中指定的，还是作为另一个任务的依赖项，或者两者兼而有之。我们来看一个例子。

以下四个任务被定义。双方dist并test取决于compile任务。运行gradle dist test此构建脚本会导致compile只执行一次任务。
![图4.1. 任务依赖关系](https://docs.gradle.org/4.4.1/userguide/img/commandLineTutorialTasks.png "图4.1. 任务依赖关系")


任务依赖关系\
例4.1. 执行多个任务

`build.gradle`
    
    task compile {
        doLast {
            println 'compiling source'
        }
    }
    
    task compileTest(dependsOn: compile) {
        doLast {
            println 'compiling unit tests'
        }
    }
    
    task test(dependsOn: [compile, compileTest]) {
        doLast {
            println 'running unit tests'
        }
    }
    
    task dist(dependsOn: [compile, test]) {
        doLast {
            println 'building the distribution'
        }
    }


输出 `gradle dist test`

    > gradle dist test
    :compile
    compiling source
    :compileTest
    compiling unit tests
    :test
    running unit tests
    :dist
    building the distribution

    BUILD SUCCESSFUL in 0s
    4 actionable tasks: 4 executed


每个任务只执行一次，所以`gradle test test`等效于`gradle test`。

####4.2. 排除任务
您可以使用`-x`命令行选项排除正在执行的任务，并提供要排除的任务的名称。让我们试试上面的示例构建文件。

例4.2. 排除任务

输出 `gradle dist -x test`

    > gradle dist -x test
    :compile
    compiling source
    :dist
    building the distribution

    BUILD SUCCESSFUL in 0s
    2 actionable tasks: 2 executed


你可以从这个例子的输出中看到，这个test任务并没有被执行，即使它是dist任务的一个依赖。你也会注意到这个test任务的依赖关系，比如`compileTest`也没有被执行。这些依赖关系test是另一个任务所要求的，比如compile仍然执行。

####4.3. 在发生故障时继续构建
默认情况下，一旦任何任务失败，Gradle将中止执行并使构建失败。这样可以尽快完成构建，但隐藏了可能发生的其他故障。为了在单个构建执行中发现尽可能多的失败，可以使用该--continue选项。

当执行时`--continue，Gradle`将执行每个要执行的任务，而该任务的所有依赖完成而不会失败，而不是在遇到第一个失败时立即停止。每个遇到的故障将在构建结束时报告。

如果一个任务失败，任何依赖它的后续任务将不会被执行，因为这样做是不安全的。例如，如果测试代码中存在编译失败，测试将不会运行; 因为测试任务将取决于编译任务（直接或间接）。

####4.4. 任务名称缩写
在命令行中指定任务时，不必提供任务的完整名称。您只需提供足够的任务名称即可唯一标识任务。例如，在上面的示例构建中，可以dist通过运行gradle d以下命令来执行任务：

例4.3. 缩写的任务名称

输出 `gradle di`

    > gradle di
    :compile
    compiling source
    :compileTest
    compiling unit tests
    :test
    running unit tests
    :dist
    building the distribution
    
    BUILD SUCCESSFUL in 0s
    4 actionable tasks: 4 executed
您也可以缩写骆驼案件任务名称中的每个单词。例如，您可以compileTest通过运行gradle compTest或甚至执行任务gradle cT

例4.4. 缩写骆驼案例任务名称

输出 `gradle cT`

    > gradle cT
    :compile
    compiling source
    :compileTest
    compiling unit tests
    
    BUILD SUCCESSFUL in 0s
    2 actionable tasks: 2 executed
您也可以使用这些缩写与-x命令行选项。

####4.5. 选择要执行的构建
当你运行该gradle命令时，它会在当前目录中查找一个构建文件。您可以使用该-b选项来选择另一个构建文件。例：

例4.5。使用构建文件选择项目

`subdir/myproject.gradle`

    task hello {
        doLast {
            println "using build file '$buildFile.name' in '$buildFile.parentFile.name'."
        }
    }
输出 `gradle -q -b subdir/myproject.gradle hello`

    > gradle -q -b subdir/myproject.gradle hello
    using build file 'myproject.gradle' in 'subdir'.
或者，您可以使用该-p选项来指定要使用的项目目录。对于多项目构建，您应该使用-p选项而不是-b选项。

例4.6. 使用项目目录选择项目

输出 `gradle -q -p subdir hello`

    > gradle -q -p subdir hello
    using build file 'build.gradle' in 'subdir'.
####4.6. 强制任务执行
许多任务，特别是Gradle自己提供的任务，都支持增量构建。这些任务可以根据自上次运行以来输入或输出是否发生变化来确定是否需要运行。当UP-TO-DATE构建运行期间，Gradle在其名称旁边显示文本时，您可以轻松识别参与增量构建的任务。

你有时可能会迫使Gradle运行所有的任务，忽略任何最新的检查。如果是这种情况，只需使用该--rerun-tasks选项。下面是运行一个没有和没有任务的输出--rerun-tasks：

例4.7。强制任务运行

输出 `gradle doIt`

    > gradle doIt
    :doIt UP-TO-DATE
输出 `gradle --rerun-tasks doIt`

    > gradle --rerun-tasks doIt
    :doIt
请注意，这将强制执行所有必需的任务，而不仅仅是在命令行上指定的任务。这有点像运行一个clean，但没有构建的生成的输出被删除。

####4.7. 获取有关您的构建的信息
Gradle提供了几个内置的任务，显示你的构建的特定细节。这对了解构建的结构和依赖关系以及调试问题很有用。

除了下面显示的内置任务之外，还可以使用项目报告插件将任务添加到将生成这些报告的项目中。

#####4.7.1. 列出项目
“正在运行” gradle projects会为您提供所选项目的子项目列表，并以层次结构显示。这里是一个例子：

例4.8。获取有关项目的信息

输出 `gradle -q projects`

    > gradle -q projects
    
    ------------------------------------------------------------
    Root project
    ------------------------------------------------------------
    
    Root project 'projectReports'
    +--- Project ':api' - The shared API for the application
    \--- Project ':webapp' - The Web application implementation
    
    To see a list of the tasks of a project, run gradle <project-path>:tasks
    For example, try running gradle :api:tasks
该报告显示每个项目的描述，如果指定的话。您可以通过设置description属性为项目提供说明：

例4.9. 提供一个项目的描述

`build.gradle`

    description = '应用程序的共享API'
4.7.2. 列出任务
运行gradle tasks会为您提供所选项目的主要任务清单。此报告显示项目的默认任务（如果有）以及每个任务的描述。下面是这个报告的一个例子：

例4.10。获取有关任务的信息

输出 `gradle -q tasks`

    > gradle -q tasks
    
    ------------------------------------------------------------
    All tasks runnable from root project
    ------------------------------------------------------------
    
    Default tasks: dists
    
    Build tasks
    -----------
    clean - Deletes the build directory (build)
    dists - Builds the distribution
    libs - Builds the JAR
    
    Build Setup tasks
    -----------------
    init - Initializes a new Gradle build.
    wrapper - Generates Gradle wrapper files.
    
    Help tasks
    ----------
    buildEnvironment - Displays all buildscript dependencies declared in root project 'projectReports'.
    components - Displays the components produced by root project 'projectReports'. [incubating]
    dependencies - Displays all dependencies declared in root project 'projectReports'.
    dependencyInsight - Displays the insight into a specific dependency in root project 'projectReports'.
    dependentComponents - Displays the dependent components of components in root project 'projectReports'. [incubating]
    help - Displays a help message.
    model - Displays the configuration model of root project 'projectReports'. [incubating]
    projects - Displays the sub-projects of root project 'projectReports'.
    properties - Displays the properties of root project 'projectReports'.
    tasks - Displays the tasks runnable from root project 'projectReports' (some of the displayed tasks may belong to subprojects).
    
    To see all tasks and more detail, run gradle tasks --all
    
    To see more detail about a task, run gradle help --task <task>
默认情况下，此报告仅显示已分配给任务组的所有任务，即所谓的可见任务。你可以通过设置group任务的属性来完成。您也可以设置description属性，以提供描述以包含在报告中。

例4.11. 更改任务报告的内容

`build.gradle`

    dists {
        description = '建立分配' 
        group = 'build'
    }
您可以使用该--all选项获取任务列表中的更多信息。使用此选项，任务报告列出项目中的所有任务，包括尚未分配给任务组的任务，即所谓的隐藏任务。这里是一个例子：

例4.12. 获取更多关于任务的信息

输出 `gradle -q tasks --all`

    > gradle -q tasks --all
    
    ------------------------------------------------------------
    All tasks runnable from root project
    ------------------------------------------------------------
    
    Default tasks: dists
    
    Build tasks
    -----------
    clean - Deletes the build directory (build)
    api:clean - Deletes the build directory (build)
    webapp:clean - Deletes the build directory (build)
    dists - Builds the distribution
    api:libs - Builds the JAR
    webapp:libs - Builds the JAR
    
    Build Setup tasks
    -----------------
    init - Initializes a new Gradle build.
    wrapper - Generates Gradle wrapper files.
    
    Help tasks
    ----------
    buildEnvironment - Displays all buildscript dependencies declared in root project 'projectReports'.
    api:buildEnvironment - Displays all buildscript dependencies declared in project ':api'.
    webapp:buildEnvironment - Displays all buildscript dependencies declared in project ':webapp'.
    components - Displays the components produced by root project 'projectReports'. [incubating]
    api:components - Displays the components produced by project ':api'. [incubating]
    webapp:components - Displays the components produced by project ':webapp'. [incubating]
    dependencies - Displays all dependencies declared in root project 'projectReports'.
    api:dependencies - Displays all dependencies declared in project ':api'.
    webapp:dependencies - Displays all dependencies declared in project ':webapp'.
    dependencyInsight - Displays the insight into a specific dependency in root project 'projectReports'.
    api:dependencyInsight - Displays the insight into a specific dependency in project ':api'.
    webapp:dependencyInsight - Displays the insight into a specific dependency in project ':webapp'.
    dependentComponents - Displays the dependent components of components in root project 'projectReports'. [incubating]
    api:dependentComponents - Displays the dependent components of components in project ':api'. [incubating]
    webapp:dependentComponents - Displays the dependent components of components in project ':webapp'. [incubating]
    help - Displays a help message.
    api:help - Displays a help message.
    webapp:help - Displays a help message.
    model - Displays the configuration model of root project 'projectReports'. [incubating]
    api:model - Displays the configuration model of project ':api'. [incubating]
    webapp:model - Displays the configuration model of project ':webapp'. [incubating]
    projects - Displays the sub-projects of root project 'projectReports'.
    api:projects - Displays the sub-projects of project ':api'.
    webapp:projects - Displays the sub-projects of project ':webapp'.
    properties - Displays the properties of root project 'projectReports'.
    api:properties - Displays the properties of project ':api'.
    webapp:properties - Displays the properties of project ':webapp'.
    tasks - Displays the tasks runnable from root project 'projectReports' (some of the displayed tasks may belong to subprojects).
    api:tasks - Displays the tasks runnable from project ':api'.
    webapp:tasks - Displays the tasks runnable from project ':webapp'.
    
    Other tasks
    -----------
    api:compile - Compiles the source files
    webapp:compile - Compiles the source files
    docs - Builds the documentation
#####4.7.3. 显示任务使用详情
运行`gradle help --task someTask`为您提供有关特定任务的详细信息，或者在多项目构建中与给定任务名称匹配的多个任务。以下是详细信息示例：

例4.13. 获得详细的帮助任务

输出 `gradle -q help --task libs`

    > gradle -q help --task libs
    Detailed task information for libs
    
    Paths
         :api:libs
         :webapp:libs
    
    Type
         Task (org.gradle.api.Task)
    
    Description
         Builds the JAR
    
    Group
         build
这些信息包括完整的任务路径，任务类型，可能的命令行选项和给定任务的描述。

#####4.7.4. 列出项目依赖关系
运行`gradle dependencies`会为您提供所选项目的依存关系列表，按配置细分。对于每个配置，该配置的直接和传递依赖关系显示在树中。下面是这个报告的一个例子：

例4.14. 获取有关依赖关系的信息

输出 `gradle -q dependencies api:dependencies webapp:dependencies`

    > gradle -q dependencies api:dependencies webapp:dependencies
    
    ------------------------------------------------------------
    Root project
    ------------------------------------------------------------
    
    No configurations
    
    ------------------------------------------------------------
    Project :api - The shared API for the application
    ------------------------------------------------------------
    
    compile
    \--- org.codehaus.groovy:groovy-all:2.4.10
    
    testCompile
    \--- junit:junit:4.12
         \--- org.hamcrest:hamcrest-core:1.3
    
    ------------------------------------------------------------
    Project :webapp - The Web application implementation
    ------------------------------------------------------------
    
    compile
    +--- project :api
    |    \--- org.codehaus.groovy:groovy-all:2.4.10
    \--- commons-io:commons-io:1.2
    
    testCompile
    No dependencies

没有依赖关系
由于依赖关系报告可能会变大，因此将报告限制为特定的配置会很有用。这是通过可选--configuration参数实现的：

例4.15. 通过配置过滤依赖关系报告

输出 `gradle -q api:dependencies --configuration testCompile`

    > gradle -q api:dependencies --configuration testCompile
    
    ------------------------------------------------------------
    Project :api - The shared API for the application
    ------------------------------------------------------------
    
    testCompile
    \--- junit:junit:4.12
         \--- org.hamcrest:hamcrest-core:1.3
#####4.7.5. 列出项目的buildscript依赖关系
运行`gradle buildEnvironment`可视化所选项目的构建脚本依赖关系，类似于`gradle dependencies`可视化正在构建的软件的依赖关系。

#####4.7.6. 深入了解特定的依赖关系
运行`gradle dependencyInsight`可让您深入了解与指定输入相匹配的特定依赖项（或依赖项）。下面是这个报告的一个例子：

例4.16. 深入了解特定的依赖关系

输出 `gradle -q webapp:dependencyInsight --dependency groovy --configuration compile`

    > gradle -q webapp:dependencyInsight --dependency groovy --configuration compile
    org.codehaus.groovy:groovy-all:2.4.10
    \--- project :api
         \--- compile
这个任务对于调查依赖关系解析，找出某些依赖关系来自何处以及为什么选择某些版本，是非常有用的。有关更多信息，请参阅DependencyInsightReportTaskAPI文档中的类。

内置的dependencyInsight任务是“帮助”任务组的一部分。该任务需要配置依赖和配置。该报告查找与指定配置中指定的依赖关系规范相匹配的依赖关系。如果应用了与Java相关的插件，dependencyInsight任务预先配置了“编译”配置，因为通常它是我们感兴趣的编译依赖项。您应该通过命令行“--dependency”选项指定您感兴趣的依赖项。如果您不喜欢默认值，您可以通过“--configuration”选项来选择配置。有关更多信息，请参阅DependencyInsightReportTaskAPI文档中的类。

#####4.7.7. 列出项目属性
正在运行`gradle properties`将为您提供所选项目的属性列表。这是输出的一个片段：

例4.17. 有关属性的信息

输出 `gradle -q api:properties`

    > gradle -q api:properties
    
    ------------------------------------------------------------
    Project :api - The shared API for the application
    ------------------------------------------------------------
    
    allprojects: [project ':api']
    ant: org.gradle.api.internal.project.DefaultAntBuilder@12345
    antBuilderFactory: org.gradle.api.internal.project.DefaultAntBuilderFactory@12345
    artifacts: org.gradle.api.internal.artifacts.dsl.DefaultArtifactHandler_Decorated@12345
    asDynamicObject: DynamicObject for project ':api'
    baseClassLoaderScope: org.gradle.api.internal.initialization.DefaultClassLoaderScope@12345
    buildDir: /home/user/gradle/samples/userguide/tutorial/projectReports/api/build
    buildFile: /home/user/gradle/samples/userguide/tutorial/projectReports/api/build.gradle
#####4.7.8. 分析构建
在`--profile`您的构建运行时命令行选项会记录一些有用的定时信息，并写入报告build/reports/profile目录。报告将使用运行构建的时间命名。

此报告列出配置阶段和任务执行的摘要时间和详细信息。配置和任务执行的时间先按最昂贵的操作排序。任务执行结果还指示是否跳过了任何任务（以及原因），还是未跳过的任务没有任何工作。

使用buildSrc目录的构建将在`buildSrc/build`目录中为`buildSrc`生成第二个配置文件报告。

![报告](https://docs.gradle.org/4.4.1/userguide/img/profile.png "报告")
####4.8. Dry Run
有时候您对哪些任务按命令行上指定的任务集执行顺序感兴趣，但不希望任务被执行。你可以使用这个-m选项。例如，如果运行"`gradle -m clean compile`"，您将看到作为`clean`和`compile`任务的一部分将执行的所有任务。这是对tasks任务的补充，它显示了可执行的任务。

####4.9. 概要
在本章中，您已经从命令行中看到了可以使用Gradle执行的一些操作。您可以gradle在[附录D Gradle Command Line](https://docs.gradle.org/4.4.1/userguide/gradle_command_line.html)中找到更多关于该命令的信息。