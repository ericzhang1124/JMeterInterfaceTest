# JMeterInterfaceTest
使用JMeter实现接口自动化测试的一个工程示例

# 1 环境配置
## 1.1 下载JMeter并安装



# 2 测试工程目录
> 目录说明

```
jmx                         // 目录，存放jmx测试脚本
resource                    // 目录， 存放测试需要的资源文件
result                      // 目录， 存放测试数据和测试报告
    report                  // 目录， 存放测试报告,生成的html文件
    test_data               // 目录， 存放测试结果数据，测试报告使用该数据生成测试报告
con                         // 目录， 存放项目的公共配置，目前内部只有一个project.properties文件
    project.properties      // 文件， 存放公共配置，例如本项目的 github.host, user.account

build.xml                   // 文件， Ant 执行的脚本文件
```

# 3 使用JMeter编写测试脚本

使用JMeter编写测试脚本即可

特别注意：
- 因为我们要使用一个公共的配置文件project.properties，所以需要修改`JMeter_HOME/bin/jmeter.properties`文件，将user.properties的值改为客户机（运行脚本的设备）的 `...\conf\project.properties`文件路径,例如我的配置是`user.properties=/Users/ericzhang/gitRes/JMeterInterfaceTest/conf/project.properties`!!!这样就可以保证我们在测试脚本里使用`${__property(key)}`获取到配置文件里的变量了！！！


---


# 4 Ant环境搭建
因为我们脚本自动化执行使用的是ant工具包，所以需要下载并配置ant

1. 下载ant
2. 将ant配置到环境变量
3. 将`JMeter_HOME/extras/ant-jmeter-1.1.1.jar` 复制一份到`ANT_HOME/lib`目录下

# 5 配置ANT的build.xml文件

从`JMeter_HOME/extras/`拷贝`build.xml`到工程根目录进行，然后进行对自己项目适配修改

```xml
<?xml version="1.0"?>
<!--
   Licensed to the Apache Software Foundation (ASF) under one or more
   contributor license agreements.  See the NOTICE file distributed with
   this work for additional information regarding copyright ownership.
   The ASF licenses this file to You under the Apache License, Version 2.0
   (the "License"); you may not use this file except in compliance with
   the License.  You may obtain a copy of the License at
    
       http://www.apache.org/licenses/LICENSE-2.0
    
   Unless required by applicable law or agreed to in writing, software
   distributed under the License is distributed on an "AS IS" BASIS,
   WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
   See the License for the specific language governing permissions and
   limitations under the License.
-->
<project name="ant-jmeter" default="all">
    <description>

        Sample build file for use with ant-jmeter.jar
        See http://www.programmerplanet.org/pages/projects/jmeter-ant-task.php

    To run a test and create the output report:
        ant -Dtest=script

    To run a test only:
        ant -Dtest=script run

    To run report on existing test output
        ant -Dtest=script report

    The "script" parameter is the name of the script without the .jmx suffix.

    Additional options:
        -Dshow-data=y - include response data in Failure Details
        -Dtestpath=xyz - path to test file(s) (default user.dir).
                         N.B. Ant interprets relative paths against the build file
        -Djmeter.home=.. - path to JMeter home directory (defaults to parent of this build file)
        -Dreport.title="My Report" - title for html report (default is 'Load Test Results')
    </description>
    
    <!--默认的用户目录，也就是build.xml所在的目录-->
    <property name="testpath" value="${user.dir}"/>
    <!--设置为客户机的JMeter根目录-->
    <property name="jmeter.home" value="/Users/ericzhang/Applications/apache-jmeter-3.2"/>
    <!--测试项目的title，可以任意设置value-->
    <property name="report.title" value="GITHUB INTERFACE TEST"/>
    
    <!-- Name of test (without .jmx) -->
    <property name="test" value="GitHubInterfaceTest"/>
    
    <!-- Should report include response data for failures? -->
    <!--如果希望在测试报告输出失败用例的响应数据，可以将该值设置为"y", 针对请求本身的失败-->
    <property name="show-data" value="y"/>
    
    <property name="format" value="2.1"/>
    
    <condition property="style_version" value="_21">
        <equals arg1="${format}" arg2="2.1"/>
    </condition>

    <condition property="funcMode">
        <equals arg1="${show-data}" arg2="y"/>
    </condition>
    
    <condition property="funcMode" value="false">
      <not>
        <equals arg1="${show-data}" arg2="y"/>
      </not>
    </condition>

    <!-- Allow jar to be picked up locally -->
    <path id="jmeter.classpath">
        <fileset dir="${basedir}">
          <include name="ant-jmeter*.jar"/>
        </fileset>
    </path>

    <taskdef
        name="jmeter"
        classpathref="jmeter.classpath"
        classname="org.programmerplanet.ant.taskdefs.jmeter.JMeterTask"/>
    
    <!--项目从这里开始执行， all任务依赖 run 和 report 两个任务-->
    <target name="all" depends="run,report"/>
    
    <!--这是run任务-->
    <target name="run">
        <!--控制台输出-->
        <echo>funcMode = ${funcMode}</echo>
        <!--执行前删除已有测试报告的操作，因为我们后面设置的报告名称和这里不一致，所以该语句无效-->
        <delete file="${testpath}/${test}.html"/>
        <!--因我的项目希望每次执行的时候将原来的JMeterResults.jtl测试数据删除重新生成，所以在执行测试前删除该文件-->
        <delete file="${testpath}/result/test_data/${test}.jtl"/>
        <!-- 因为我的项目希望执行的是工程目录jmx下的所有脚本，所以要使用testpalans， 删除testplan ="${testpath}/${test}.jmx"这行，替换为\<testplans dir="${basedir}/jmx" includes="*.jmx"/\>, 这里的脚本根目录需要根据自己的项目结构设置，resultlog就是测试产生的数据了-->
        <!--修改测试数据结果保存的路径resultlog-->
        <jmeter
            jmeterhome="${jmeter.home}"
            resultlog="${testpath}/result/test_data/${test}.jtl">
            <testplans dir="${basedir}/jmx" includes="*.jmx"/>
            
        <!--
            <jvmarg value="-Xincgc"/>
            <jvmarg value="-Xmx128m"/>
            <jvmarg value="-Dproperty=value"/>
            <jmeterarg value="-qextra.properties"/>
        -->
            <!-- Force suitable defaults -->
            <property name="jmeter.save.saveservice.output_format" value="xml"/>
            <property name="jmeter.save.saveservice.assertion_results" value="all"/>
            <property name="jmeter.save.saveservice.bytes" value="true"/>
            <property name="file_format.testlog" value="${format}"/>
            <property name="jmeter.save.saveservice.response_data.on_error" value="${funcMode}"/>
        </jmeter>
    </target>

    <property name="lib.dir" value="${jmeter.home}/lib"/>

    <!-- Use xalan copy from JMeter lib directory to ensure consistent processing with Java 1.4+ -->
    <path id="xslt.classpath">
        <fileset dir="${lib.dir}" includes="xalan*.jar"/>
        <fileset dir="${lib.dir}" includes="serializer*.jar"/>
    </path>
    <!--生成测试报告的任务-->
    <!--不清楚copy-images任务是做什么的，因此这里先将该任务删除（copy-images）-->
    <target name="report" depends="xslt-report">
        <echo>Report generated at ${report.datestamp}</echo>
    </target>

    <target name="xslt-report" depends="_message_xalan">
        <!--应为希望生成的测试报告带有时间戳，所以这里改变也一下时间的格式-->
        <tstamp><format property="report.datestamp" pattern="yyyyMMddHHmmss"/></tstamp>
        <!--修改in的值为自己生成测试数据的文件值-->
        <!--修改out为放置测试报告的目录,并为生成报告的文件名添加上时间戳-->
        <!--修改style路径-->
        <xslt
            classpathref="xslt.classpath"
            force="true"
            in="${testpath}/result/test_data/${test}.jtl"
            out="${testpath}/result/report/${test}${report.datestamp}.html"
            style="${jmeter.home}/extras/jmeter-results-detail-report${style_version}.xsl">
            <param name="showData" expression="${show-data}"/>
            <param name="titleReport" expression="${report.title}"/>
            <param name="dateReport" expression="${report.datestamp}"/>
        </xslt>
    </target>

    <!-- Copy report images if needed -->
    <!--不清楚该任务，这里先将其注释掉了
    <target name="copy-images" depends="verify-images" unless="samepath">
        <copy file="${basedir}/expand.png" tofile="${testpath}/expand.png"/>
        <copy file="${basedir}/collapse.png" tofile="${testpath}/collapse.png"/>
    </target>

    <target name="verify-images">
        <condition property="samepath">
                <equals arg1="${testpath}" arg2="${basedir}" />
        </condition>
    </target>
    -->
    
    <!-- Check that the xalan libraries are present -->
    <condition property="xalan.present">
          <and>
              <!-- No need to check all jars; just check a few -->
            <available classpathref="xslt.classpath" classname="org.apache.xalan.processor.TransformerFactoryImpl"/>
            <available classpathref="xslt.classpath" classname="org.apache.xml.serializer.ExtendedContentHandler"/>
          </and>
    </condition>

    <target name="_message_xalan" unless="xalan.present">
          <echo>Cannot find all xalan and/or serialiser jars</echo>
        <echo>The XSLT formatting may not work correctly.</echo>
        <echo>Check you have xalan and serializer jars in ${lib.dir}</echo>
    </target>


</project>

```

# 6 执行ant命令生成测试报告
