---
title: "Jenkins Pipeline集成Sonar进行代码质量检测"
date: 2019-03-07T9:14:39+08:00
draft: false
type: blog
banner: "http://wx4.sinaimg.cn/large/ad5fbf65ly1g0u28qlkvrj21pj15o4ji.jpg"
authors: ["guoxudong"]
authorlink: "https://github.com/sunny0826"
summary: "在devops理念中，CI/CD毫无疑问是最重要的一环，而代码质量检查则是CI中必不可少的一步。在敏捷开发的思想下，代码的迭代周期变短，交付速度提升，这个时候代码的质量就很难保证，测试只能保证功能完整与可用，而代码的质量纯靠review的话效率又很低，这个时候sonar..."
tags: ["devops","jenkins","sonar"]
categories: ["devops"]
keywords: ["devops","jenkins","sonar"]
image:
  url: "https://tva1.sinaimg.cn/large/ad5fbf65ly1ge3jciuxh3j21pj15o4ji.jpg"
  caption: "Image from: [**Pexels**](https://www.pexels.com)"
  focal_point: ""
  preview_only: false
---
## 简介
image:
  caption: "Image from: [**Pexels**](https://www.pexels.com)"
  focal_point: ""
  preview_only: false
---

### jenkins pipeline

Jenkins Pipeline (或简称为 "Pipeline" )是一套插件，将持续交付的实现和实施集成到 Jenkins 中。

持续交付Pipeline自动化的表达了这样一种流程：将基于版本控制管理的软件持续的交付到您的用户和消费者手中。

Jenkins Pipeline 提供了一套可扩展的工具，用于将“简单到复杂”的交付流程实现为“持续交付即代码”。 Jenkins Pipeline 的定义通常被写入到一个文本文件（称为 ```Jenkinsfile``` ）中，该文件可以被检入到项目的源代码控制库中。

摘自[Jenkins官方文档](https://jenkins.io/zh/)

### SonarQube
SonarQube is an open source platform to perform automatic reviews with static analysis of code to detect bugs, code smells and security vulnerabilities on 25+ programming languages including Java, C#, JavaScript, TypeScript, C/C++, COBOL and more. 

SonarQube是一个开源的平台，以执行与代码的静态分析，自动审查，可以检测在25+的编程语言如Java，C＃，JavaScript，TypeScript，C/C++，COBOL等的代码缺陷和安全漏洞。

### OWASP
OWASP，全称是：Open Web Application Security Project，翻译为中文就是：开放式Web应用程序安全项目，是一个非营利组织，不附属于任何企业或财团，这也是该组织可以不受商业控制地进行安全开发及安全普及的重要原因，[详细介绍](https://en.wikipedia.org/wiki/OWASP/)。OWASP Dependency-Check，它识别项目依赖关系，并检查是否存在任何已知的、公开的、漏洞，基于OWASP Top 10 2013。

## 场景
在devops理念中，CI/CD毫无疑问是最重要的一环，而代码质量检查则是CI中必不可少的一步。在敏捷开发的思想下，代码的迭代周期变短，交付速度提升，这个时候代码的质量就很难保证，测试只能保证功能完整与可用，而代码的质量纯靠review的话效率又很低，这个时候sonar就可以很好的帮助开发自动化检测代码质量，降低bug数量，也可以根据扫描结果养成良好的编程习惯，同时也可以减少测试的工作量，真正提升整个团队效率，实现devops理念。

## 前提
jenkins、sonarqube服务已经搭建完成，jenkins安装sonar插件```SonarQube Scanner for Jenkins```，jenkins、sonarqube安装Dependency-Check插件```OWASP Dependency-Check Plugin```

版本：jenkins2.166，sonarqube6.7.6

## 配置

1. 下载安装jenkins插件

    **[系统管理]**-**[插件管理]**-**[可选插件]**-**[SonarQube Scanner for Jenkins]**

    ![image](http://wx4.sinaimg.cn/large/ad5fbf65ly1g0u4q3ae1bj20t90233yt.jpg)

2. SonarQube生成token，**这个token不会显示第二次，所以一定要记住**

    ![image](https://tva2.sinaimg.cn/mw690/ad5fbf65ly1g0u5902q6nj213f0hgwgn.jpg)

3. SonarQube配置Dependency-Check

    **[配置]**-**[Dependency-Check]**

    **[注意：]**这里去掉 ```${WORKSPACE}/```，否则将报
    ```Java stack trace
    [INFO] Dependency-Check XML report does not exists. Please check property sonar.dependencyCheck.reportPath:/data/jenkinsHome/workspace/xxx/${WORKSPACE}/dependency-check-report.xml
    ```

    ![image](http://wx4.sinaimg.cn/large/ad5fbf65ly1g0yvjjcvdaj211b0jhgod.jpg)

4. 在pom.xml文件中添加

    ```xml
    <plugin>
        <groupId>org.sonarsource.scanner.maven</groupId>
        <artifactId>sonar-maven-plugin</artifactId>
        <version>3.6.0.1398</version>
    </plugin>
    ```

5. 配置jenkins

    **[系统管理]**-**[系统设置]**-**[SonarQube servers]**

    ![image](http://wx3.sinaimg.cn/large/ad5fbf65ly1g0u50l8q4lj215o0b3myw.jpg)

6. sonar添加webhook

    在代码扫描成功后，扫描结果需要回调jenkins，添加的Jenkins的webhook结构为：http://[jenkins_url]/sonarqube-webhook/

    **[配置]**-**[web回调接口]**-**[URL]**

    ![image](http://wx4.sinaimg.cn/large/ad5fbf65ly1g0v4m590vhj212k0pw0vo.jpg)

7. 编辑jenkins pipeline

    在jenkinsfile文件中添加配置

    ```Groovy
    stage('依赖安全检查') {
        steps{
            dependencyCheckAnalyzer datadir: '', hintsFile: '', includeCsvReports: false, includeHtmlReports: true, includeJsonReports: false, includeVulnReports: true, isAutoupdateDisabled: false, outdir: '', scanpath: '', skipOnScmChange: false, skipOnUpstreamChange: false, suppressionFile: '', zipExtensions: ''
        }
    }

    stage('静态代码检查') {
        steps {
            echo "starting codeAnalyze with SonarQube......"
            withSonarQubeEnv('sonar') {
                //注意这里withSonarQubeEnv()中的参数要与之前SonarQube servers中Name的配置相同
                withMaven(maven: 'M3') {
                    sh "mvn clean package -Dmaven.test.skip=true sonar:sonar -Dsonar.projectKey={项目key} -Dsonar.projectName={项目名称} -Dsonar.projectVersion={项目版本} -Dsonar.sourceEncoding=UTF-8 -Dsonar.exclusions=src/test/** -Dsonar.sources=src/ -Dsonar.java.binaries=target/classes -Dsonar.host.url={SonarQube地址} -Dsonar.login={SonarQube的token}"
                }
            }
            script {
                timeout(1) {
                    //这里设置超时时间1分钟，不会出现一直卡在检查状态
                    //利用sonar webhook功能通知pipeline代码检测结果，未通过质量阈，pipeline将会fail
                    def qg = waitForQualityGate('sonar')
                    //注意：这里waitForQualityGate()中的参数也要与之前SonarQube servers中Name的配置相同
                    if (qg.status != 'OK') {
                        error "未通过Sonarqube的代码质量阈检查，请及时修改！failure: ${qg.status}"
                    }
                }
            }
        }
    }
    ```

    **参数解释：**

    - sonar.projectKey：项目key (必填项)
    - sonar.projectName：项目名称（必填项）
    - sonar.projectVersion：项目版本（必填项）
    - sonar.sources：源码位置(相对路径）
    - sonar.java.binaries：编译后的class位置（必填项，相对路径同上）
    - sonar.exclusions：排除的扫描的文件路径
    - sonar.host.url：SonarQube地址
    - sonar.login：SonarQube生成的token

## 运行
执行jenkins构建，构建成功后会显示如下，则证明sonar代码扫描成功且通过代码质量阈检查
![image](https://tva2.sinaimg.cn/mw690/ad5fbf65ly1g0u6qrh8qrj21fu0q2dmw.jpg)

查看sonar报告，这里有两种方式

- 可直接登录SonarQube查看报告
![image](https://tva2.sinaimg.cn/mw690/ad5fbf65ly1g0u6vbspv5j21260myadw.jpg)

- 也可直接在jenkins页面点击SonarQube图标进入，点击以下标记均可进去
![image](https://ws2.sinaimg.cn/mw690/ad5fbf65ly1g0u6xzcryhj21fn0q7wkm.jpg)

## 其他

### 问题一：无法扫描代码，错误提示

```Java stack trace
hudson.remoting.ProxyException: hudson.AbortException: SonarQube installation defined in this job (sonar) does not match any configured installation. Number of installations that can be configured: 1.
If you want to reassign jobs to a different SonarQube installation, check the documentation under https://redirect.sonarsource.com/plugins/jenkins.html
    at hudson.plugins.sonar.SonarInstallation.checkValid(SonarInstallation.java:94)
    at hudson.plugins.sonar.SonarBuildWrapper.setUp(SonarBuildWrapper.java:67)
    at org.jenkinsci.plugins.workflow.steps.CoreWrapperStep$Execution.start(CoreWrapperStep.java:80)
    at org.jenkinsci.plugins.workflow.cps.DSL.invokeStep(DSL.java:268)
Caused: hudson.remoting.ProxyException: org.codehaus.groovy.runtime.InvokerInvocationException: hudson.AbortException: SonarQube installation defined in this job (sonar) does not match any configured installation. Number of installations that can be configured: 1.
If you want to reassign jobs to a different SonarQube installation, check the documentation under https://redirect.sonarsource.com/plugins/jenkins.html
    at org.jenkinsci.plugins.workflow.cps.CpsStepContext.replay(CpsStepContext.java:499)
    at org.jenkinsci.plugins.workflow.cps.DSL.invokeStep(DSL.java:295)
    at org.jenkinsci.plugins.workflow.cps.DSL.invokeStep(DSL.java:207)
    at org.jenkinsci.plugins.workflow.cps.DSL.invokeDescribable(DSL.java:395)
    at org.jenkinsci.plugins.workflow.cps.DSL.invokeMethod(DSL.java:179)
    at org.jenkinsci.plugins.workflow.cps.CpsScript.invokeMethod(CpsScript.java:122)
    at sun.reflect.GeneratedMethodAccessor1200.invoke(Unknown Source)
    at sun.reflect.DelegatingMethodAccessorImpl.invoke(DelegatingMethodAccessorImpl.java:43)
    at java.lang.reflect.Method.invoke(Method.java:498)
    at org.codehaus.groovy.reflection.CachedMethod.invoke(CachedMethod.java:93)
    at groovy.lang.MetaMethod.doMethodInvoke(MetaMethod.java:325)
    at groovy.lang.MetaClassImpl.invokeMethod(MetaClassImpl.java:1213)
    at groovy.lang.MetaClassImpl.invokeMethod(MetaClassImpl.java:1022)
    at org.codehaus.groovy.runtime.callsite.PogoMetaClassSite.call(PogoMetaClassSite.java:42)
    at org.codehaus.groovy.runtime.callsite.CallSiteArray.defaultCall(CallSiteArray.java:48)
    at org.codehaus.groovy.runtime.callsite.AbstractCallSite.call(AbstractCallSite.java:113)
    at org.kohsuke.groovy.sandbox.impl.Checker$1.call(Checker.java:157)
    at org.kohsuke.groovy.sandbox.GroovyInterceptor.onMethodCall(GroovyInterceptor.java:23)
    at org.jenkinsci.plugins.scriptsecurity.sandbox.groovy.SandboxInterceptor.onMethodCall(SandboxInterceptor.java:155)
    at org.kohsuke.groovy.sandbox.impl.Checker$1.call(Checker.java:155)
    at org.kohsuke.groovy.sandbox.impl.Checker.checkedCall(Checker.java:159)
    at org.kohsuke.groovy.sandbox.impl.Checker.checkedCall(Checker.java:129)
    at org.kohsuke.groovy.sandbox.impl.Checker.checkedCall(Checker.java:129)
    at org.kohsuke.groovy.sandbox.impl.Checker.checkedCall(Checker.java:129)
    at com.cloudbees.groovy.cps.sandbox.SandboxInvoker.methodCall(SandboxInvoker.java:17)
Caused: hudson.remoting.ProxyException: java.lang.IllegalArgumentException: Failed to prepare withSonarQubeEnv step
    at org.jenkinsci.plugins.workflow.cps.DSL.invokeDescribable(DSL.java:397)
    at org.jenkinsci.plugins.workflow.cps.DSL.invokeMethod(DSL.java:179)
    at org.jenkinsci.plugins.workflow.cps.CpsScript.invokeMethod(CpsScript.java:122)
    at sun.reflect.GeneratedMethodAccessor1200.invoke(Unknown Source)
    at sun.reflect.DelegatingMethodAccessorImpl.invoke(DelegatingMethodAccessorImpl.java:43)
    at java.lang.reflect.Method.invoke(Method.java:498)
    at org.codehaus.groovy.reflection.CachedMethod.invoke(CachedMethod.java:93)
    at groovy.lang.MetaMethod.doMethodInvoke(MetaMethod.java:325)
    at groovy.lang.MetaClassImpl.invokeMethod(MetaClassImpl.java:1213)
    at groovy.lang.MetaClassImpl.invokeMethod(MetaClassImpl.java:1022)
    at org.codehaus.groovy.runtime.callsite.PogoMetaClassSite.call(PogoMetaClassSite.java:42)
    at org.codehaus.groovy.runtime.callsite.CallSiteArray.defaultCall(CallSiteArray.java:48)
    at org.codehaus.groovy.runtime.callsite.AbstractCallSite.call(AbstractCallSite.java:113)
    at org.kohsuke.groovy.sandbox.impl.Checker$1.call(Checker.java:157)
    at org.kohsuke.groovy.sandbox.GroovyInterceptor.onMethodCall(GroovyInterceptor.java:23)
    at org.jenkinsci.plugins.scriptsecurity.sandbox.groovy.SandboxInterceptor.onMethodCall(SandboxInterceptor.java:155)
    at org.kohsuke.groovy.sandbox.impl.Checker$1.call(Checker.java:155)
    at org.kohsuke.groovy.sandbox.impl.Checker.checkedCall(Checker.java:159)
    at org.kohsuke.groovy.sandbox.impl.Checker.checkedCall(Checker.java:129)
    at org.kohsuke.groovy.sandbox.impl.Checker.checkedCall(Checker.java:129)
    at org.kohsuke.groovy.sandbox.impl.Checker.checkedCall(Checker.java:129)
    at com.cloudbees.groovy.cps.sandbox.SandboxInvoker.methodCall(SandboxInvoker.java:17)
    at WorkflowScript.run(WorkflowScript:27)
    at ___cps.transform___(Native Method)
    at com.cloudbees.groovy.cps.impl.ContinuationGroup.methodCall(ContinuationGroup.java:57)
    at com.cloudbees.groovy.cps.impl.FunctionCallBlock$ContinuationImpl.dispatchOrArg(FunctionCallBlock.java:109)
    at com.cloudbees.groovy.cps.impl.FunctionCallBlock$ContinuationImpl.fixArg(FunctionCallBlock.java:82)
    at sun.reflect.GeneratedMethodAccessor249.invoke(Unknown Source)
    at sun.reflect.DelegatingMethodAccessorImpl.invoke(DelegatingMethodAccessorImpl.java:43)
    at java.lang.reflect.Method.invoke(Method.java:498)
    at com.cloudbees.groovy.cps.impl.ContinuationPtr$ContinuationImpl.receive(ContinuationPtr.java:72)
    at com.cloudbees.groovy.cps.impl.ClosureBlock.eval(ClosureBlock.java:46)
    at com.cloudbees.groovy.cps.Next.step(Next.java:83)
    at com.cloudbees.groovy.cps.Continuable$1.call(Continuable.java:174)
    at com.cloudbees.groovy.cps.Continuable$1.call(Continuable.java:163)
    at org.codehaus.groovy.runtime.GroovyCategorySupport$ThreadCategoryInfo.use(GroovyCategorySupport.java:122)
    at org.codehaus.groovy.runtime.GroovyCategorySupport.use(GroovyCategorySupport.java:261)
    at com.cloudbees.groovy.cps.Continuable.run0(Continuable.java:163)
    at org.jenkinsci.plugins.workflow.cps.SandboxContinuable.access$101(SandboxContinuable.java:34)
    at org.jenkinsci.plugins.workflow.cps.SandboxContinuable.lambda$run0$0(SandboxContinuable.java:59)
    at org.jenkinsci.plugins.scriptsecurity.sandbox.groovy.GroovySandbox.runInSandbox(GroovySandbox.java:121)
    at org.jenkinsci.plugins.workflow.cps.SandboxContinuable.run0(SandboxContinuable.java:58)
    at org.jenkinsci.plugins.workflow.cps.CpsThread.runNextChunk(CpsThread.java:182)
    at org.jenkinsci.plugins.workflow.cps.CpsThreadGroup.run(CpsThreadGroup.java:332)
    at org.jenkinsci.plugins.workflow.cps.CpsThreadGroup.access$200(CpsThreadGroup.java:83)
    at org.jenkinsci.plugins.workflow.cps.CpsThreadGroup$2.call(CpsThreadGroup.java:244)
    at org.jenkinsci.plugins.workflow.cps.CpsThreadGroup$2.call(CpsThreadGroup.java:232)
    at org.jenkinsci.plugins.workflow.cps.CpsVmExecutorService$2.call(CpsVmExecutorService.java:64)
    at java.util.concurrent.FutureTask.run(FutureTask.java:266)
    at hudson.remoting.SingleLaneExecutorService$1.run(SingleLaneExecutorService.java:131)
    at jenkins.util.ContextResettingExecutorService$1.run(ContextResettingExecutorService.java:28)
    at jenkins.security.ImpersonatingExecutorService$1.run(ImpersonatingExecutorService.java:59)
    at java.util.concurrent.Executors$RunnableAdapter.call(Executors.java:511)
    at java.util.concurrent.FutureTask.run(FutureTask.java:266)
    at java.util.concurrent.ThreadPoolExecutor.runWorker(ThreadPoolExecutor.java:1149)
    at java.util.concurrent.ThreadPoolExecutor$Worker.run(ThreadPoolExecutor.java:624)
    at java.lang.Thread.run(Thread.java:748)
Finished: FAILURE
```

原因：withSonarQubeEnv()中的参数与之前SonarQube servers中Name的配置不同，导致没有找到找到SonarQube

### 问题二：SonarQube的token配置不对，导致无法连接sonar

```Java stack trace
[ERROR] Failed to execute goal org.sonarsource.scanner.maven:sonar-maven-plugin:3.6.0.1398:sonar (default-cli) on project callcenter: Not authorized. Please check the properties sonar.login and sonar.password. -> [Help 1]
[ERROR] 
[ERROR] To see the full stack trace of the errors, re-run Maven with the -e switch.
[ERROR] Re-run Maven using the -X switch to enable full debug logging.
[ERROR] 
[ERROR] For more information about the errors and possible solutions, please read the following articles:
[ERROR] [Help 1] http://cwiki.apache.org/confluence/display/MAVEN/MojoExecutionException
[Pipeline] }
[withMaven] artifactsPublisher - Archive artifact pom.xml under cn/keking/callcenter/callcenter/0.0.1-SNAPSHOT/callcenter-0.0.1-SNAPSHOT.pom
[withMaven] artifactsPublisher - Archive artifact target/callcenter-0.0.1-SNAPSHOT.jar under cn/keking/callcenter/callcenter/0.0.1-SNAPSHOT/callcenter-0.0.1-SNAPSHOT.jar
[withMaven] artifactsPublisher - Archive artifact target/callcenter-0.0.1-SNAPSHOT-api.jar under cn/keking/callcenter/callcenter/0.0.1-SNAPSHOT/callcenter-0.0.1-SNAPSHOT-api.jar
[withMaven] junitPublisher - Archive test results for Maven artifact cn.keking.callcenter:callcenter:jar:0.0.1-SNAPSHOT generated by maven-surefire-plugin:test (default-test): target/surefire-reports/*.xml
[withMaven] junitPublisher - Jenkins JUnit Attachments Plugin not found, can't publish test attachments.Recording test results
None of the test reports contained any result
[withMaven] Jenkins Task Scanner Plugin not found, don't display results of source code scanning for 'TODO' and 'FIXME' in pipeline screen.
[withMaven] Publishers: Pipeline Graph Publisher: 1 ms, Generated Artifacts Publisher: 891 ms, Junit Publisher: 4 ms, Dependencies Fingerprint Publisher: 5 ms
[Pipeline] // withMaven
[Pipeline] }
WARN: Unable to locate 'report-task.txt' in the workspace. Did the SonarScanner succedeed?
```

原因：sonar.login的token配置不正确或者没有配置

### 问题三：jenkins pipeline在SonarQube回调时显示超时

```Java stack trace
[Pipeline] waitForQualityGate
Checking status of SonarQube task 'AWlX97LSgWqXn-z33SO5' on server 'sonar'
SonarQube task 'AWlX97LSgWqXn-z33SO5' status is 'IN_PROGRESS'
Cancelling nested steps due to timeout
```

原因：SonarQube没有配置webhook回调，导致请求超时，按照步骤4配置webhook即可解决

### 问题四：sonar找不到Dependency-Check XML

```Java stack trace
[INFO] Sensor Dependency-Check [dependencycheck]
[INFO] Process Dependency-Check report
[INFO] Dependency-Check XML report does not exists. Please check property sonar.dependencyCheck.reportPath:/data/jenkinsHome/workspace/xxx/${WORKSPACE}/dependency-check-report.xml
[INFO] Analysis skipped/aborted due to missing report file
[INFO] Dependency-Check HTML report does not exists. Please check property sonar.dependencyCheck.htmlReportPath:/data/jenkinsHome/workspace/xxx/${WORKSPACE}/dependency-check-report.html
[INFO] HTML-Dependency-Check report does not exist.
```
原因：SonarQube配置Dependency-Check插件有误，按照上文配置即可

## 结语

sonar与jenkins集成的方式还有很多，不止pipeline+maven这一种，还有配置在jenkins构建任务中、直接使用sonar脚本等方法。采用这样方法，一方面是配置相对简单，不需要每个构建任务都进行配置，只需要将jenkinsfile中拷入相应代码并修改几个参数即可。同时可以在静态代码扫描期间完整maven打包，减少持续集成的时间。