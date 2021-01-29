---
title: 工作笔记
date: 2019-12-30 13:09:29
# password: xiaomi191203
# abstract: 工作中常见或遇到过的问题汇总
# message:  输入密码，查看文章
tags:
- python
- robotframework
- java
- docker
- hexo
categories:
- work-code
---
# 简要介绍
   工作中常见或遇到过的问题汇总
<!-- more -->
# 1. **monitor平台zookeeper平台kibana平台**
   - 1. 系统传参。
      在monitor平台中通过接口管理传递参数时，在robot测试用例中必须包含*** Variables ***字段，哪怕用例中实际用不到也得添加。传递其他参数时同样通过接口管理的param下的param里添加key-value的形式添加参数名和值，然后在测试计划的配置列表中添加param字段，在里面的value字段中添加key-value，同样测试用例中需有Variables，用例中使用${key}来使用此参数。
   - 2. 基本操作。
      添加的测试用例必须符合格式，具体为[]括起，每句用例用""括起，除最后一句外，句尾加'，'号。句中注意特殊字符需要\转义。目前已知"和\两个需要转义。
   - 3. 本地monitor平台。
      本地平台地址：`http://host:8080/result/prod/6`, 其中host根据丽姐本地url变化。访问时需手动两次更改localhost为实际url，登录名密码均为邮箱前缀。
   - 4. zk获取各环境路径。
   ```
     product:c4/services下搜thrift文件中定义的service名字
     staging:staging/services下搜thrift文件中定义的service名字
     preview/prod_c3:c3/services下搜thrift文件中定义的service名字，点进去的Pool中各个host详情里的server.service.level字段=1是preview，server.service.level=10是prod_c3
     prod_c6:c6cloudsrv/services下搜thrift文件中定义的service名字
   ```
   - 5. Elasticsearch深度分页解决：scroll游标。包括初始化和遍历两部分。
     > 初始化。
     ```
      POST ip:port/my_index/my_type/_search?scroll=1m
      {
         "query": { "match_all": {}}
      }
     ```
     > 遍历。
     ```
      POST /_search?scroll=1m
      {
         "scroll_id":"XXXXXXXXXXXXXXXXXXXXXXX I am scroll id XXXXXXXXXXXXXXX"
      }
     ```
   说明：初始化时请求体中数据照常，只需在search后加上`?scroll=1m`，初始化返回一个_scroll_id，_scroll_id 用来下次取数据用。遍历里的的 scroll_id 即 上一次遍历取回的 _scroll_id 或者是初始化返回的 _scroll_id，同样的，需要带 scroll 参数。 重复这一步骤，直到返回的数据为空，即遍历完成。注意，每次都要传参数 scroll，刷新搜索结果的缓存时间。设置scroll的时候，需要使搜索结果缓存到下一次遍历完成，同时，也不能太长，毕竟空间有限。

# 2. **jenkins平台**
   - 1. python版本选择
      `ln -s /usr/bin/python3.6 /usr/bin/python` 通过这条命令，把需要jenkins使用的pythonX 变更为python
   - 2. 环境依赖
      jenkins和本地环境不一致，可直接使用jenkins shell脚本在jenkins添加环境
   - 3. 全局配置
      1. 安装java：`sudo apt-get install openjdk-8-jdk`，下载war包，运行`java -jar jenkins.war`。
      2. 安装完下面提示的插件后，在系统配置中需要配置git和mail相关。其中如是gitlab，则先在gitlab中获取访问令牌，再添加ssh-key。在mail配置中需要选择smtp，填写邮箱用户名密码，再在计划的设置中的mail配置中设置相关选项才能正确发送邮件。
   - 4. 邮件模板配置
      将run.template模板文件放在`$JENKINS_HOME/email-templates`目录下，如不存在则创建此目录。在job配置的`Editable Email Notification`中，配置`Content Type`为`HTML(text/html)`，`Default Subject`为`XX测试报告：${BUILD_STATUS} - ${PROJECT_NAME} - Build # ${BUILD_NUMBER}!`，`Default Content`为`${SCRIPT, template="run.template"}`。
                注：如不行，则将jenkins配置中的`Default Subject`和`Default Content`值也改为如上。
         附:run.template
         ```
            <%
            import java.text.DateFormat
            import java.text.SimpleDateFormat
            %>
            <STYLE>
            BODY, TABLE, TD, TH, P {
            font-family:Verdana,Helvetica,sans serif;
            font-size:11px;
            color:black;
            }
            h1 { color:black; }
            h2 { color:black; }
            h3 { color:black; }
            TD.bg1 { color:white; background-color:#0000C0; font-size:120% }
            TD.bg2 { color:white; background-color:#4040FF; font-size:110% }
            TD.bg3 { color:white; background-color:#8080FF; }
            TD.test_passed { color:blue; }
            TD.test_failed { color:red; }
            TD.console { font-family:Courier New; }
            </STYLE>
            <BODY>

            <TABLE>
            <TR><TD align="right"><IMG SRC="${rooturl}<%= build.result == hudson.model.Result.SUCCESS  ? "static/e59dfe28/images/32x32/blue.gif" : "static/e59dfe28/images/32x32/red.gif" %>" />
            </TD><TD valign="center"><B style="font-size: 150%;"><%= build.result == hudson.model.Result.SUCCESS ? "TESTSRUN ${build.result}" : "TESTRUN ${build.result}" %></B></TD></TR>
            <TR><TD>项目名称:</TD><TD>${project.name}</TD></TR>
            <TR><TD>运行日期:</TD><TD>${it.timestampString}</TD></TR>
            <TR><TD>测试时间:</TD><TD>${build.durationString}</TD></TR>
            <TR><TD>构建URL:</TD><TD><A href="http://10.234.30.24:8080/${build.url}">http://10.234.30.24:8080/${build.url}</A></TD></TR>
            <TR><TD>测试报告:</TD><TD><A href="http://10.234.30.24:8080/${build.url}robot/report/report.html">打开 report.html</A></TD></TR>

            <BR/>

            <!-- 自动化测试汇总报告 -->
            <%
            def robotResults = false
            def actions = build.actions // List<hudson.model.Action>
            actions.each() { action ->
            if( action.class.simpleName.equals("RobotBuildAction") ) { // hudson.plugins.robot.RobotBuildAction
               robotResults = true %>
            <p><h4>Robot Framework Results</h4></p>
            <table cellspacing="0" cellpadding="4" border="1" align="left">
                  <thead>
                     <tr bgcolor="#87CEFA">
                        <td><b>类型</b></td>
                        <td><b>用例总数</b></td>
                        <td><b>通过</b></td>
                        <td><b>不通过</b></td>
                        <td><b>通过率</b></td>
                     </tr>

                  </thead>

                  <tbody>

                     <tr><td><b>所有测试</b></td>
                        <td><%= action.result.overallTotal %></td>
                        <td><b><span style="color:#66CC00"><%= action.result.overallPassed %></span></b></td>
                        <td><b><span style="color:#FF3333"><%= action.result.overallFailed %></span></b></td>
                  <td><%= action.overallPassPercentage %>%</td>
                     </tr>

                     <tr><td><b>关键测试</b></td>
                        <td><%= action.result.criticalTotal %></td>
                        <td><b><span style="color:#66CC00"><%= action.result.overallPassed %></span></b></td>
                        <td><b><span style="color:#FF3333"><%= action.result.overallFailed %></span></b></td>
                  <td><%= action.overallPassPercentage %>%</td>
                     </tr>

                  </tbody>
                  </table>
            <br />
            <br />
            <br />
            <br />
            <br />
            <br />
            <table cellspacing="0" cellpadding="4" border="1" align="left">
            <thead>
            <tr bgcolor="#87CEFA">
            <td colspan="2"><b>测试名称</b></td>
            <td><b>状态</b></td>
            <td><b>执行时间</b></td>
            </tr>
            </thead>
            <tbody>
            <%  def suites = action.result.allSuites
               suites.each() { suite -> 
                  def currSuite = suite
                  def suiteName = currSuite.displayName
                  // ignore top 2 elements in the structure as they are placeholders
                  while (currSuite.parent != null && currSuite.parent.parent != null ) {
                  currSuite = currSuite.parent
                  suiteName = currSuite.displayName + "." + suiteName
                  } %>
            <tr><td colspan="3"><b><%= suiteName %></b></td></tr>
            <%    DateFormat format = new SimpleDateFormat("yyyyMMdd HH:mm:ss.SS")
                  def execDateTcPairs = []
                  suite.caseResults.each() { tc ->
                  Date execDate = format.parse(tc.starttime)
                  execDateTcPairs << [execDate, tc]
                  }
                  // primary sort execDate, secondary displayName
               execDateTcPairs = execDateTcPairs.sort{ a,b -> a[1].displayName <=> b[1].displayName }
               execDateTcPairs = execDateTcPairs.sort{ a,b -> a[0] <=> b[0] }
               execDateTcPairs.each() {
                  def execDate = it[0]
                  def tc = it[1]  %>

                                       <tr>
                                          <td colspan="2"><%= tc.displayName %></td>
                                          <td><b><span style="color:<%= tc.isPassed() ? "#66CC00" : "#FF3333" %>"><%= tc.isPassed() ? "PASS" : "FAIL" %></span></b></td>
                                          <td><%= tc.getDuration().intdiv(60000)+"分"+(tc.getDuration()-tc.getDuration().intdiv(60000)*60000).intdiv(1000)+"秒" %></td>
                                       </tr>
                                       <% if(tc.errorMsg != null) {%>
                                       <tr>
                                          <td ><b><span style="font-size:10px;color:#FF3333">错误描述：</span></b></td>
                                          <td colspan="3"><span style="font-size:10px"><%= tc.errorMsg%></span></td>

                                       </tr>
                                       <%
                                       }%>
            <%  } // tests
               } // suites %>
                                    </tbody>
                              </table>


                                 <p style="color:#AE0000;clear:both">*这个是通过Jenkins自动构建得出的报告，仅供参考。</p>
                              </div>
                              <%
            } // robot results
            }
            if (!robotResults) { %>
                  <p>No Robot Framework test results found.</p>
            <%
            } %>
            o
         ```

   - 5. robotframework工作环境
      thrift安装：
         安装thrift依赖：
         ```
           sudo apt-get install libboost-dev libboost-test-dev libboost-program-options-dev libevent-dev automake libtool flex bison pkg-config g++ libssl-dev
         ```
         解压编译：
         ```
            tar -zxvf thrift-0.11.0.tar.gz
            cd thrift-0.11.0
            ./configure --with-cpp --with-boost --with-python --without-csharp --with-java --without-erlang --without-perl --with-php --without-php_extension --without-ruby --without-haskell  --without-go
            make
            make install
         ```
      ```
       $ pip isntall thrift                # 如有thrift接口
       $ pip install robotframework
       $ pip install robotframework-requests
       $ pip install robotframework-jsonschemalibrary
       $ pip install robotframework-jsonvalidator
      ```
             注：若有报错，可能是more-itertools版本过高，网上有人说python2.7最高支持5.0.0
   - 6. robotframework运行报错:`UnicodeEncodeError: 'ascii' codec can't encode characters in position 0-1: ordinal not in range(128)`，一般是代码在windows平台运行过，此时需在py文件中加入：
   ```
      # 解决中文编码报错
      default_encoding = 'utf-8'
      if sys.getdefaultencoding() != default_encoding:
         reload(sys)
         sys.setdefaultencoding(default_encoding)
   ```
   才能在robotframework中传递中文参数。
            另注：rf的报告中只会打印rf直接调用的函数里的Print语句的内容，故如需调试应在client.py文件中添加print，ms.py中添加的无效。
   - 7. robotframework中tag用法。加在`*** Settings ***`下的`Default Tags    jokesAPI`，或加在case用例中的`[Tags]    fw`，运行robot时的参数：
   > -i --include tag *    Select test cases to run by tag
   > -e --exclude tag *    Select test cases not to run by tag
   如`robot -i dog -e cat animals.robot`则执行animals.robot文件中tag为dog但不执行tag为cat的测试用例。
   - 8. robotframework中Run Keyword If判断时，当变量值为字符串时，需用引号将变量引起来，或者使用变量时不加{}，与变量对比的字符常量也需用引号引起来。例：
   ```
      Run Keyword IF    '${resp[5]}'=='morning'
      Run Keyword IF    $resp[5]=='evening'
   ```
   当使用了Run Keyword If关键字时，后面的ELSE IF必须大写，且每个ELSE IF语句判断后面需跟动作，例如Log或Fail，不然会报`'Else If' is a reserved keyword. It must be in uppercase (ELSE IF) when used as a marker with 'Run Keyword If'.`
   - 9. robotframework导入python库。
      ```
      # 如果类的 __init__ 初始化方法需要传参，则在导入库后面跟对应的参数列表
      Library path/python_class_name.py    argument
      # 用路径法导入Python模块需要有文件后缀，且用 / 来表示目录下
           # 重点：使用路径法，只能导入和模块名相同的类名！
      ```
      1. python扩展库搜索规则：
        > 先根据 robot 文件自身当前目录下查找库文件
        > 如果没有找到则再根据 `--pythonpath` 和 `-P` 提供的搜索路径进行搜索
        > 最后找 Python 安装的路径
      2. python库引入了其他模块：
        > 当 robot 文件导入的 Python 测试库引入了其他模块时，确保导入的模块路径和RF导入的模块起始路径统一。
      3. python库中的class存在继承：
        > 当 robot 文件导入 Python 测试库的类继承了另一个类，确保导入的模块路径和RF导入的模块起始路径统一，使用的时候 RF 文件只需导入子类即可。
   - 10. robotframework跳过执行关键字、for循环嵌套if判断。
        > BuiltIn_ 关键字 `Pass Execution` 和 `Pass Execution If` 以PASS状态结束执行, 同时跳过剩下的关键字。
           ```
           Run Keyword IF    ${num1}<${num2}    Pass Execution    测试通过
           ...    ELSE IF    ${num1}==${num2}    Fail    读取历史数据对比全部失败
           ```
        > 循环嵌套判断，只要保证循环体内语句缩进两个空格以上。
           ```
           FOR    ${m}    IN    @{msg}
               ${num2}=    Evaluate    ${num2} + 1
               ${low}    Evaluate    float(${m * 0.2})
               ${high}    Evaluate    float(${m * 3.9})
               Run Keyword IF    ${low}<=${current}<=${high}    Log    失败次数:${current}在配置文件同比数据:${m}的0.2到3.9倍范围内
               ...    ELSE IF    ${-500}<=${current}-${m}<=${500}    Log    小数值差值范围在500以内
               ...    ELSE    AddNum
           END
           ```
                  1.如果  IN 后面跟的是一个 List 变量，必须用 @{list} 的格式哦！
                  2.循环体内的语句需要缩进两个空格以上
                  3.如果 IN 后面接的值太多，可以换行，需要通过 ... 来表示接着上一行的内容
                  4.注意：  FOR  和  IN 都不能小写哦

   - 11. jenkins+gitlab配置webhook
      首先确认`Gitlab Hook Plugin`和`Build Authorization Token Root Plugin`插件已安装。然后在job配置中勾选`Build when a change is pushed to GitLab. GitLab webhook URL: http://10.234.30.24:8080/project/test_suite`选项，保存GitLab webhook URL待用。在`Enabled GitLab triggers`中勾选第三个`Accepted Merge Request Events`，在高级选项中点`Secret token`后的`Genrate`会生成token，保存待用。在gitlab项目中选settings->Intergrations(集成)，粘贴保存的URL和Secret Token，点击Add webhook，点击Test测试连接即可。
   - 12. jenkins托管flask服务的shell脚本
   ```
      #!/bin/bash
      pwd
      source /home/mi/abc_issues_wsgi/venv/bin/activate
      cd /home/mi/.jenkins/workspace/apidata/
      a=$(ps -aux|grep uwsgi)
      str=$"\n"
      echo ${a}
      if [ ${#a} -lt 350 ]
      then
         nohup uwsgi config.ini &
         sstr=$(echo -e $str)
         echo $sstr
      else
         b=${a:9:5}
         s=`echo ${b}`
         echo ${s}
         kill -9 ${s}
         sleep 3
         nohup uwsgi config.ini &
         sstr=$(echo -e $str)
         echo $sstr
      fi
      jobs -l
   ```
   更新版本，防止ps查询的命令的进程id在首位：
   ```
   #!/bin/bash
   pwd
   source /home/mi/abc_issues_wsgi/venv/bin/activate
   cd /home/mi/.jenkins/workspace/apidata/
   a=$(ps -aux|grep uwsgi)
   str=$"\n"
   echo ${a}
   arr=(${a})
   if [ ${#a} -lt 350 ]
   then
      nohup uwsgi config.ini &
      sstr=$(echo -e $str)
      echo $sstr
   else
      if [ ${arr[7]} = "S+" ]
      then
         s=${arr[13]}
      else
         s=${arr[1]}
      fi
      #b=${a:9:5}
      #s=`echo ${b}`
      echo ${s}
      kill -9 ${s}
      sleep 3
      nohup uwsgi config.ini &
      sstr=$(echo -e $str)
      echo $sstr
   fi
   jobs -l
   ```
   再次更新脚本，发现当kill值为'Sl'那栏的id时只会杀死单个进程，不会停止整个服务。
   ```
   #!/bin/bash
   pwd
   source /home/mi/abc_issues_wsgi/venv/bin/activate
   cd /home/mi/.jenkins/workspace/apidata/
   a=$(ps -aux|grep uwsgi)
   str=$"\n"
   echo ${a}
   arr=(${a})
   if [ ${#a} -lt 350 ]
   then
      nohup uwsgi config.ini &
      sstr=$(echo -e $str)
      echo $sstr
   else
      index=7
      until [ "${arr[index]}" = "S" ]
      do
         echo ${arr[index]}
         index=$(($index + 12))
      done
      echo $index
      indexId=$(($index - 6))
      echo $indexId
      s=${arr[$indexId]}
      echo ${s}
      kill -9 ${s}
      sleep 3
      nohup uwsgi config.ini &
      sstr=$(echo -e $str)
      echo $sstr
   fi
   jobs -l
   ```
   - 13. jenkins添加用户及配置权限
      前提是已创建管理员账户，在管理中选择`Manage Users`，可以新建用户。
      再在管理中选择`Configure Global Security`，启用安全，安全域为`Jenkins own user database`，在授权策略中选择`项目矩阵授权策略`，添加用户，配置读权限。再在各job设置中启用项目安全，添加用户，配置各项权限。

# 3. **python相关**
   - 1. 在 python2中，str 其实是 bytes，而不是 unicode，在代码中声明了编码方式为 utf-8，并将该参数存入到了 DB 中，导致下次请求传递的还是 DB 中的 utf-8 类型的 port，而不是 int 或者 string，port给int型
   - 2. 配置文件ini中不得加上`coding: utf-8`。python函数获取当前文件名的方法是：`os.path.basename(sys.argv[0]).split('.')[0]`，但当用其他文件中方法调用此函数时不能返回当前文件名，此时需用：`__file__.split('/')[-1].split('.')[0]`。
   - 3. 对于windows系统，python2会产生编码问题，此时需加上：`reload(sys)  \ sys.setdefaultencoding('utf-8')`来重置系统默认编码。
   - 4. 对于配置文件，当调用时`config.read(filename)`，此时可用`os.path.dirname(os.path.realpath(__file__)) + filename`来获取绝对的文件路径。
   - 5. requests访问https时，如遇SSL或CA证书验证失败，可在发送请求时将verify参数设置为False，默认为True，开启证书验证。对于有登录要求的，可以抓包后将cookie放置在headers里，实例化requests.session()对象后通过session对象发请求。
   - 6. python调用java的jpype模块安装及使用注意：
      - 安装：`pip install jpype1`
               注意:当使用python2时,需使用`pip install jpype1==0.7.1`,这是最后一个支持py2的版本!
      - 使用：
         这里的ext_classpath指的是.class文件的的引用路径之前的路径，如：Javatest.class文件的全路径是：`D:\code\H5\run\demo\src\com\Javatest.class`，Javatest类的包路径（看上面的目录结构）是com，所以此处`ext_classpath='D:\code\H5\run\demo\src'`；JClass的路径就是Javatest类的包路径：JClass('com.Javatest')
         ![解释1](/images/carbon11.png)
         ![解释2](/images/carbon12.png)
   - 7. python写入文件按照列表的方式读取， 每写入一条数据需写入一个换行符隔开，读取出来则是以列表的格式，filehandle.write('\n')
   - 8. thrift接口，客服端调用报错 ’‘TSocket’‘读取0字节，服务端报错 ''No handlens could be found for logger 'Thrift.server.TServer'''，只要是Handlens类报错，提示全都是这个，实际最后找到报错为返回类型和idl文件定义不一致，包括某些变量拼写错误也会出现这种情况
   - 9. flask-sqlalchemy对已生成的表字段做修改，在migrations/env.py文件，在run_migrations_online函数加入![图](/images/carbon18.png)
   flask-sqlalchemy中不常用的类型，Text(16777216)在mysql中属于longtext`变长字符串，max32M`，Text(65536)属于mediumtext`变长字符串，max16M`，Text属于tinytext`变长字符串，64K`。
          数据迁移migrate的`flask db migrate -m "Datetime some change"`迁移命令带上-m选项后可用来添加迁移备注，在项目的`migrations/versions/`文件夹下的迁移记录名会带上备注，方便查询迁移顺序等。
   - 10. flask_sqlalchemy使用query.filter()查询数据库/filter()和filter_by()
      - 使用query.filter().all()返回列表，列表中为数据对象，使用`object.字段名`来取值
      - 使用query.filter().first()返回数据对象，同样使用`object.字段名`来取值
      - filter_by用于查询简单的列名，不支持比较运算符。语法：`column = expression`传入参数的写法，只需要用：`（不带类名的）列名 单个等号` 就可以判断。
      - filter中，语法更加贴近于，类似于，Python的语法。比filter_by的功能更强大，且更复杂的查询的语法，比如and()，or()等多个条件的查询，只支持filter。语法：`column == expression`传入参数的写法，要用：`类名.列名 两个等号` 去判断
      - 字段模糊查询，`filter(类名.字段名.like('%' + 精准字段值 + '%'))`
   - 11. robot对python2的thrift接口返回response做To Json处理时报`UnicodeDecodeError: 'utf8' codec can't decode byte 0xb7 in position 0: invalid start byte`，此时可对被解析对象重编码.encode('utf-8')，如仍报`UnicodeDecodeError: 'ascii' codec can't decode byte 0xb7 in position 263: ordinal not in range(128)`，则用`.decode('gbk').encode('utf-8')`重编码，python2的代码示例如下：
   ```
      import json
      s = '{"_srcdata":"{\\"updateDate\\":\\"20200103\\",\\"keywords\\":[\\"\u5927\u536b\u65af\u7279\u6069\\",\\"mvp\\",\\"nba\\",\\"NBA\\",\\"\u9ec4\u91d1\\",\\"\u62c9\u91cc\\"],\\"cpId\\":\\"20200103A07HEN00\\",\\"updateTime\\":1578019149000,\\"title\\":\\"\u4f17\u7403\u661f\u7f05\u6000\u5927\u536b\xb7\u65af\u7279\u6069\uff0c\u4ed6\u662fNBA\u9ec4\u91d130\u5e74\u91cc\u771f\u6b63\u7684MVP\\",\\"cp\\":\\"qqnews\\",\\"playLength\\":67,\\"playUrl\\":\\"http://files.ai.xiaomi.com/aiservice/aiservice/qqnews/276102adb35b72723a1abe0a0a39a941.mp3\\",\\"audio_id\\":337697176311300121,\\"id\\":\\"ai-audio-news_4628730706182661528\\",\\"categories\\":[\\"\u4f53\u80b2\\"],\\"category\\":\\"\u4f53\u80b2\\",\\"isHot\\":\\"0\\",\\"simHash\\":[\\"t-9197614281938041715\\",\\"b-2886507832045461559\\",\\"b383678740477312137\\",\\"t3500282104762336264\\",\\"b-7491421025233353511\\",\\"b-8069685134151215735\\",\\"t2310628634316906393\\",\\"b-2285480801161917239\\"],\\"status\\":1}","updateTime":1578019149000,"id":"ai-audio-news_4628730706182661528"}'
      sj = json.loads(s.decode('gbk').encode('utf-8'))
      print(sj['_srcdata'])
   ```
   - 12. thrift文件生成后运行时报`SyntaxError: Non-ASCII character '\xe8' in file gen-py/ai_course/ttypes.py on line 23, but no encoding declared`，是因为thrift文件中的中文注释用了`/**/`，将所有注释改为`//`再重新生成即可。
   - 13. flask使用Bokeh保存图片和plotly保存图片。
   此两种绘图库将所绘制图形保存为图片所依赖的库不同，分述如下：
      - Bokeh
         依赖PhantomJS，需用`conda install phantomjs`或`npm install -g phantomjs-prebuilt`命令。实测用的npm，如报错则执行命令`sudo apt install nodejs-legacy`
         ```
         Bokeh启用WebGL加速：
         p = figure(output_backend="webgl")  # for the plotting API
         p = Plot(output_backend="webgl")  # for the glyph API
         ```
         > Bokeh将生成内容嵌入html：
         ```
         from bokeh.embed import components
         ...
         p_script, p_div = components(p)
         p_html = p_script + p_div
         return render_template('example.html', p_html=p_html)
         # 或者
         # return jsonify({'status': 200, 'p_html': p_html})
         ```
         > Bokeh画图自适应宽度：在figure中添加`sizing_mode='scale_width'`参数，同时给出`plot_height=300`参数，不然高度也会自适应屏幕。
      - plotly
         依赖orca等。需执行`conda install -c plotly plotly-orca`或`npm install -g electron@6.1.4 orca`，此npm建议nodejs版本>=6.0。实测npm一直报错找不到文件及权限等，即使在命令后加上`--unsafe-perm=true --allow-root`安装成功后依然无法保存plotly绘制的图形，最后安装miniconda使用conda命令安装后可使用。
         plotly将生成内容嵌入到html：
         ```
         import plotly
         ...
         div = plotly.offline.plot(fig, auto_open=False, output_type='div')
         return render_template('example.html', plotly_div=div)
         ```
      - jinja2
         当html内容字符串作为数据传入到jinja2中时，如果要模板自动渲染，需要使用{{ div|safe }}或者
         ```
         {% autoescape false %}
         {{ div }}
         {% endautoescape %}
         ```
         来自动渲染成html内容。
   - 14. plotly、Bokeh保存路径支持相对路径，但openpyxl保存excel表格的路径只能用绝对路径，且send_from_directory的第二个参数为excel文件名，获取当前绝对路径：`os.path.abspath(os.path.join(os.path.dirname(__file__), os.path.pardir))`
   - 15. celery运行配置了redis，但实际尝试还是得装rabbitmq，不然会报连接不上，不知具体原因。安装rabbitmq前先得装erlang，具体可网上找。rabbitmq网上说的没成功，最后在官网下载的deb安装包，通过`sudo dpkg -i download_file\?file_path\=pool%2Frabbitmq-server%2Frabbitmq-server_3.7.23-1_all.deb`安装，如报依赖于socat，然而未安装，直接`apt-get install socat`，再运行上面命令安装rabbitmq，装好后运行`systemctl status rabbitmq-server`查看运行状态。重启`supervisorctl reload`后celery任务即可成功运行。
   - 16. flask中post传递的data，只需`req = request.form`后即可用`req.get('key')`取得表单中对应key的value，而无需`req = request.form()`，此时会报`'ImmutableMultiDict' object is not callable`。
   - 17. python异常捕获try...except语句捕获的异常打印出来信息太少，不利于调试，此时可用traceback模块，`traceback.print_exc()`直接打印出错误，`traceback.format_exc()`将返回字符串。例：
   ```
   try:
       pass
   except Exception as e:
        traceback.print_exc()
        return render_template('errorpage.html')
   ```
   因flask中uwsgi会将stdout标准输出的内容写入到日志中，故在flask项目中直接用`traceback.print_exc()`方法。
   附try...except语句能调用的属性和方法：
   ```
   try:
       1/0
   except Exception as e:
       print(e.args)  # 返回异常的错误编号和描述字符串
       print(str(e))  # 返回异常信息，但不包括异常信息的类型
       print(repr(e))  # 返回较全的异常信息，包括异常信息的类型

   输出结果：
   ('division by zero',)
   division by zero
   ZeroDivisionError('division by zero',)
   ```
   - 18. python2安装mysql官方提供的mysql-connector-python时`pip install mysql-connector-python`，如报setuptools版本错误，是版本必须小于45,此时可先卸载setuptools，再`pip install "setuptools<45"`安装特定条件版本，或者在建虚拟环境时`py -2.7-32 -m virtualenv --no-setuptools venv`或`virtualenv --no-setuptools venv --python=python2`
              注意：mysql-connector-python只有8.0的版本支持mysql8.0
   - 19. python线程池的使用。通过`thread_count = multiprocessing.cpu_count()`获取cpu核心数作为线程池中线程个数，示例代码：
   ```
   def multiThread(Ids=[]):
       thread_count = multiprocessing.cpu_count() * 2
       with ThreadPoolExecutor(max_workers=thread_count) as t:
           obj_list = []
           begin = time.time()
           # all_task = [t.submit(exportResult, i) for i in Ids]
           # wait(all_task,)
           for i in Ids:
               obj = t.submit(exportResult, i)
               obj_list.append(obj)
           for future in as_completed(obj_list):
               data = future.result()
               print(data)
           times = time.time() - begin
           print(f'总耗时：{times}')
       return f'{Ids}全部下载完成'
   ```
             注：其中exportResult为启动线程所需要运行的函数，i为该函数的入参。
   - 20. python与js中获取当前ip。
      > python
      ```
      import socket
      def getIp():
          s = socket.socket(socket.AF_INET, socket.SOCK_DGRAM)
          s.connect(('8.8.8.8', 80))
          ip = s.getsockname()[0]
          s.close()
          return ip
      ```
      > js
      ```
      $(document).ready(function () {
          URI = window.location.host;
          console.log(URI)
      });
      // window.location.host 获取ip:port(不带http://，赋值给a标签的href或ajax的url时需手动带上)
      // 其余方法另查
      ```
   - 21. python导出&安装依赖。
      1. 导出`pip freeze > requirements.txt`。
      2. 安装`pip install -r requirements.txt`。
      3. conda安装`conda install --yes --file requirements.txt`，如果requirements.txt中的包不可用，则会抛出“无包错误”，此时可用`while read requirement; do conda install --yes $requirement; done < requirements.txt`。
      4. 在conda命令无效时使用pip命令来代替`while read requirement; do conda install --yes $requirement || pip install $requirement; done < requirements.txt`。
      5. conda使用yaml文件，先激活环境，再导出`conda env export > py37.yaml`，使用`conda env create -f py37.yaml`。
                注意：以上只会导出conda命令直接安装的包。

# 4. **MYSQL数据库**
   - 1. 查询数据库中的状态
     >use databasename;
     >show processlist;  state显示lock或waiting的可以杀死
     >kill id;
   - 2. 导出数据表
     > mysql -uroot -ppasswd -e "select * from apipassrate" apidata > ~/GitHub/apipassrate.xlsx
             导出的xlsx文件不能直接被openpyxl模块识别，可通过excel或wps重新另存为。其中的时间字段如需还原成逗号分隔的文字，先把字段类型改为日期，语言选en-us或英国，在格式码处改为`YYYY,MM,DD,HH,MM,SS`，再复制到外部文本编辑器中，表格中另加一列，将外部数据复制进新加列，改字段类型为文本即可。
   - 3. 对查询出的数据二次查询报错-ERROR 1248 (42000): Every derived table must have its own alias。原因是在多级查询时，需要给表一个别名。
   错误示例：
             select avg(num_value) from (select distinct create_time,num_value from kibanadata where create_time like "%00:30:00" and aiservice_type='cpresource' and cp_name='xiaowei');
   正确示例：
             select avg(num_value) from (select distinct create_time,num_value from kibanadata where create_time like "%00:30:00" and aiservice_type='cpresource' and cp_name='xiaowei') as num;
   - 4. mysql对json对象操作的json_extract函数
     > 语法：select json_extract(data, '$.name') from table_json;
     > 注释：data为字段名，data字段里的数据为json对象，通过$表示该对象，.name表示键值对的key，得到的为键值对的value。
   - 5. mysql cast函数
     > 语法：cast(expression as TYPE)
     > 注释：cast()函数将任何类型的值转换为具有指定类型的值。目标类型可以是以下类型之一：BINARY、CHAR、DATE、DATETIME、TIME、DECIMAL、SIGNED、UNSIGNED。
     > 例子：mysql中比较字符型的数字时，'999'是大于'1099'的，故此时需将此字段的类型转换为DECIMAL类型后再做比较或者直接max()取最大值。
   - 6. mysql各种连接查询
     > 内连接：`select * from a_table (as) a inner join b_table (as) b on a.a_id=b.b_id;`
     > 左(外)连接：`select * from a_table (as) a left (outer) join b_table (as) b on a.a_id=b.b_id;`
     > 右(外)连接：`select * from a_table (as) a right (outer) join b_table (as) b on a.a_id=b.b_id;`
     > 全(外)连接：mysql不支持全外连接，full join，需要左连，右连后再union (all)。
   - 7. mysql备份及恢复
     > 备份：`mysqldump -hHost -uroot -ppasswd -Pport 数据库名 > test.sql`或`mysqldump -hHost -uroot -ppasswd -Pport 数据库名 数据表名 > test.sql`
     > 恢复：`mysql -hHost -uroot -ppasswd -Pport 数据库名 < test.sql`，或者在需要恢复的机器上进mysql后`source test.sql`，或者运行`mysqldump -h10.224.104.124 -uroot -pXIAOMI apidata coveragedata | mysql -h10.38.154.14 -udb_monitor_staging -pdb_monitor_staging db_monitor`将124机器上apidata数据库中coveragedata表导入到14机器上的db_monitor数据库中，db_monitor数据库需存在
   - 8. ubuntu命令行安装mysql时未提示输入密码，则可以在`/etc/mysql/debian.cnf`文件中找到用户名和密码，用此用户名密码登录mysql后，可重置密码，或添加一个root用户。成功后重启mysql服务即可。
   - 9. mysql将查询结果以逗号分隔一行打印，使用`group_concat()`函数，例：`select group_concat(cpname) from (select distinct(cpname) from kibanawow where aiservice_type=406 and value!=0 group by cpname) as name;`。
   - 10. mysql在linux环境自动备份脚本及自动任务。
     > 备份脚本`dump_mysql.sh`
     ```
       #!/bin/zsh
       #保存备份个数，备份7天数据
       number=7
       #备份保存路径
       # backup_dir=/home/fuyu/GitHub/work-code/shell/mysql
       backup_dir=$(dirname $(readlink -f $0))/mysql
       echo $backup_dir
       #日期
       dd=`date +%Y-%m-%d-%H-%M-%S`
       #备份工具
       tool=mysqldump
       #用户名
       username=root
       #密码
       password=tarena
       #将要备份的数据库
       database_name=apidata

       #如果文件夹不存在则创建
       if [ ! -d $backup_dir ]; 
       then     
          mkdir -p $backup_dir; 
       fi

       #简单写法  mysqldump -u root -p123456 users > /root/mysqlbackup/users-$filename.sql
       $tool -u $username -p$password $database_name > $backup_dir/$database_name-$dd.sql

       #写创建备份日志
       echo "create $backup_dir/$database_name-$dd.dupm" >> $backup_dir/log.txt

       #找出需要删除的备份
       delfile=`ls -l -crt  $backup_dir/*.sql | awk '{print $9 }' | head -1`

       #判断现在的备份数量是否大于$number
       count=`ls -l -crt  $backup_dir/*.sql | awk '{print $9 }' | wc -l`

       if [ $count -gt $number ]
       then
       #删除最早生成的备份，只保留number数量的备份
       rm $delfile
       #写删除文件日志
       echo "delete $delfile" >> $backup_dir/log.txt
       fi

     ```
     > 自动任务`dump_mysql.cron`
     ```
       0 0 * * * /home/fuyu/GitHub/work-code/shell/dump_mysql.sh
 
     ```
     > 启动任务
         1. 添加脚本执行权限：`chmod +x dump_mysql.sh`
         2. 启动crontab任务：`crontab dump_mysql.cron`
         3. 检查任务是否创建：`crontab -l`
     > 注意：cron文件中末尾必须有空行，否则报错
   - 11. mysql不支持`123<=id<=125`这类判断操作，在删除中带此类条件会清空数据表！！！使用`123<=id and id<=125`语句来判断。
   - 12. SQLite数据库
     > 安装：ubuntu自带
     > 使用：
         > `sqlite3`进入操作界面，`sqlite3 filename.db`打开特定数据库，直接执行sql语句。
           如想格式化数据展示，则通过三步：
           ```
           .header on
           .mode column
           .timer on
           ```
                  `.help`可打开帮助介绍。
         > python操作sqlite：
         ```
         import sqlite3
         db = sqlite3.connect('path/filename.db')
         cur = db.cursor()
         cur.execute('SQL语句')
         # 如是查询语句，则可获取查询到的内容
         print(cur.fetchall())
         cur.close()
         # 如是增加、修改操作，则需提交更改
         db.commit()
         db.close()
         ```

# 5. **java、scala、安卓相关**
   - 1. maven项目需将所有依赖的jar包打包到lib目录：`mvn dependency:copy-dependencies -DoutputDirectory=target/lib`
   - 2. java的 ‘==’和‘equals()’方法，== 对于基本类型来说是值比较，对于引用类型来说是比较的是引用；而 equals 默认情况下是引用比较，只是很多类重写了 equals 方法，比如 String、Integer 等把它变成了值比较，所以一般情况下 equals 比较的是值是否相等。
   - 3. Android Retrofit网络请求
   初始化Retrofit:
   ```
   String BASE_URL = "http://102.10.10.132/api/";
   Retrofit retrofit = new Retrofit.Builder()
            .baseUrl(BASE_URL)
            .build();
   ```
   GET请求:
   ```
   @GET("News/{newsId}")
   Call<NewsBean> getItem(@Path("newsId") String newsId);
   ```
               则url：http://102.10.10.132/api/News/newsId
   参数在?之后：
   ```
   @GET("News")
   Call<NewsBean> getItem(@Query("newsId") String newsId);
   ```
               则url：http://102.10.10.132/api/News?newsId=newsId
   附录：
     > @Path：网址中的参数，?前
     > @Query：?后的参数
     > @QueryMap：相当于多个@Query
     > @Field：Post提交单个数据
     > @Body：相当于多个@Field，以对象的形式提交
   - 4. java项目打包成jar包，用到maven依赖的，需填写`pom.xml`依赖文件，其中需要定义包的结构和入口文件，`target/lib`文件夹下的依赖jar包在`~/.m2/repository`下找，可通过项目中pom.xml同级的`xxx.iml`文件所列路径和包名查找。也可以通过maven命令导出到自定义文件夹`mvn dependency:copy-dependencies -DoutputDirectory=target/lib`
      ```
      <modelVersion>4.0.0</modelVersion>
      <groupId>com.xiaomi</groupId>
      <artifactId>apus-server-example</artifactId>
      <version>1.0-SNAPSHOT</version>

      <properties>
        <project.build.sourceEncoding>UTF-8</project.build.sourceEncoding>
      </properties>

      <build>
        <plugins>
            <plugin>
                <groupId>org.apache.maven.plugins</groupId>
                <artifactId>maven-jar-plugin</artifactId>
                <version>2.4</version>
                <configuration>
                    <archive>
                        <manifest>
                            <addClasspath>true</addClasspath>
                            <classpathPrefix>lib/</classpathPrefix>  <!--依赖jar包的存放目录，在target/lib下-->
                            <addDefaultImplementationEntries>true</addDefaultImplementationEntries>
                            <mainClass>client.Client</mainClass>  <!--入口文件名，在src/main/java下，从java目录下文件开始，即主文件的package路径加上.文件名-->
                        </manifest>
                        <manifestEntries>
                            <Class-Path>lib/apus-server-example-1.0-SNAPSHOT.jar</Class-Path>
                        </manifestEntries>
                    </archive>
                </configuration>
            </plugin>
        </plugins>
      </build>
      ```
   - 5. spring相关。
        1. velocity，后缀为.vm，是java-spring框架的模板文件，类似flask的jinja2模板，语法需详查。

# 6. **docker相关**
   - docker基础操作
     1. 查询镜像- 在docker hub中搜索，或者命令行`docker search imagename`
     2. 下载镜像-`docker pull imagename`
     3. 查询本地镜像-`docker images`
     4. 启动镜像-`docker run -it REPOSITORY:TAG bash`;  显式运行镜像bash命令行;  -i 表示持续打开 STDIN(标准输入);-t 表示申请一个 tty 给这个 docker 使用，同样是为了能够交互，所以一般情况下都是一起使用; -d后台运行。-p port:port（前面是宿主机端口，后面是容器里监听端口，例：`docker run -d -p 40000:40000 REPOSITORY:TAG python3 /home/mi/test/app.py`，此时可通过宿主机ip:40000端口访问容器里的:40000端口的web服务）
     5. 退出容器-`exit`或者ctrl+d
     6. 查看容器-`docker ps`(运行中);  `docker ps -a`(本地存在的容器)
     7. 启动容器/进入后台容器/停止容器-`docker start CONTAINER ID`;  `docker exec -it CONTAINER ID bash`;  `docker stop CONTAINER ID`;   注：使用start启动的容器会进入后台运行状态
     8. 删除容器-`docker rm CONTAINER ID`
     9. 删除镜像-`docker rmi REPOSITORY:TAG`
   - windows10系统加速源填写的格式错误会导致docker不可用，正确方式为在系统右下角托盘 Docker 图标内右键菜单选择 Settings，打开配置窗口后左侧导航菜单选择 Daemon。在 Registrymirrors 一栏中填写加速器地址 `https://registry.docker-cn.com` ，之后点击 Apply 保存后 Docker 就会重启并应用配置的镜像地址
ubuntu系统加速方式为，更换为国内的镜像作为加速器，首先打开配置文件,配置文件如果不存在则新建：`vi /etc/docker/daemon.json` ;加入如下内容：`{ "registry-mirrors": ["https://docker.mirrors.ustc.edu.cn"]}`;重启docker: `service docker restart`
   - ubuntu:16.04
     1. 运行：docker run -it ubuntu:16.04 bash;    注：-it是前端交互，:16.04是tag，不写默认latest，会重新从docker hub拉取最新的ubuntu镜像，bash指开启bash。
     2. 安装python3：apt-get update(更新库);  apt-get install python3;  apt-get install python3-pip;
         安装pip3 install mysql时可能会报'Command "python setup.py egg_info" failed with error code 1 in /tmp/pip-install-d27dt_9j/mysqlclient/' 此时先：'pip3 install --upgrade setuptools'，再：'apt install libmysqlclient-dev python-mysqldb'，如还报：'''Command "/usr/bin/python3 -u -c "import setuptools, tokenize;file='/tmp/pip-install-agcpvfop/mysqlclient/setup.py';f=getattr(tokenize, 'open', open)(file);code=f.read().replace('\r\n', '\n');f.close();exec(compile(code, file, 'exec'))" install --record /tmp/pip-record-cuzgf87x/install-record.txt --single-version-externally-managed --compile" failed with error code 1 in /tmp/pip-install-agcpvfop/mysqlclient/'''，则再：'apt-get install libpcap-dev libpq-dev'，就可成功安装
     3. 安装mysql：apt-get install mysql-server mysql-client;   如报错:ERROR 2002 (HY000): Can't connect to local MySQL server through socket '/var/run/mysqld/mysqld.sock'，则：cd /etc/init.d;  service mysql stop;  service mysql start
     4. 查询ip：ip address;
     5. 保存容器：docker commit -m '解释语句' CONTAINER ID REPOSITORY:TAG;     例如：docker commit -m 'add python' bac0551903dd yourname/ubuntu:1.0.0 
         命令：-a :提交的镜像作者；-c :使用Dockerfile指令来创建镜像；-m :提交时的说明文字；-p :在commit时，将容器暂停
     6. 推送到远程：注册账号，security里创建access token，命令行docker login --username yourname，提示输入password时贴入access token。docker push REPOSITORY:TAG。若repository不是以yourname命名，则需先docker tag REPOSITORY:TAG yourname/REPOSITORY:TAG
   - jenkins
     1. 拉取jenkins镜像：  $docker pull jenkinsci/jenkins  该版本为最新版本
         完成后使用docker images查看是否拉取成功运行jenkins容器： $docker run -p 10086:8080 -p 50000:50000  -v loaclpath:contatinerPath jenkinsci/jenkins   (-p选项，前面是宿主机映射的端口，后面是jenkins容器默认端口，-v选项，前面是本地目录，后面是jenkins容器默认目录，可以将本地目录挂载到容器里)在网页输入localhost:8080打开jenkins,输入控制台提示的administrator password，设置admin用户名和密码，进入jenkins图形界面，成功！！！
     2. 安装插件，如果失败，两种解决方案
         一、更新下载插件镜像地址，点击Manage Jenkins→Manager Plugin→Advanced下面的Update site框输入新的镜像地址，镜像地址查询：`http://mirrors.jenkins-ci.org/status.html`
         二、到`https://wiki.jenkins-ci.org/display/JENKINS/Plugins` 网站，手动下载需要的插件，然后在系统管理–管理插件–高级–上传插件即可，点击上传，然后它会自动上传及安装，待jenkins重启后插件即生效
     3. 当前项目jenkins需要使用到的插件有：
           gitlab、Gitlab Authentication、Gitlab Hook Plugin、Groovy、Groovy Postbuild、Robot Framework（机器人日志）、Email Extension（邮件配置）、build timeout（构建超时）、（chinese可选）、Email Extension Template Plugin(邮件扩展模板插件)、HTML Publisher (发布HTML报告)、Post build task(该插件允许根据构建日志输出执行批处理/ shell任务)、Naginator（设置失败后重新构建或其他动作）、Startup Trigger（设置启动时构建）、Workspace Cleanup（删除项目工作区)、email-ext（邮件模板配置）、Matrix Authorization Strategy（设置项目矩阵授权策略）
     4. 部分功能参考https://www.cnblogs.com/luchuangao/p/7748575.html#_label19

# 7. **hexo搭建gitpage**
   - 1. md文章导入图片方法：在/目录source文件夹下新建images文件夹，将图片放入，md引用时使用`/images/..`
   - 2. 评论和自定义域名功能暂未实现。
   - 3. 推送源码到github。
      使用hexo deploy推送到github的实现是hexo自动生成的public文件夹里的内容。如果想推送源码，可以先新建分支，然后再用git push提交。
          注：在根目录。
      ```
       1. git checkout -b source  # 切换到一个新分支'source'
       2. git add -A  # 添加源码
       3. git commit -m 'commit'  # 提交
       4. git push origin source  # 提交分支
      ```
      推送后在仓库的Settings中的Branches中更改默认分支显示即可。
   - 4. 新电脑创建已有环境。
       1. 安装node，下载gitpage的main分支。
       2. 进入项目主目录运行`npm install hexo-cli -g`及`npm install`命令安装博客环境。
       3. 运行`hexo generate`。
       4. 检查`themes/next`文件夹下是否有数据，此文件文件夹是项目主题，从其他已有环境中复制此文件夹下内容。
       5. 运行`hexo server`即可打开本地服务。

# 8. **ubuntu(linux)相关**
   - 1. ubuntu16.04系统，设置点击启动栏图标后应用最小化功能：`gsettings set org.compiz.unityshell:/org/compiz/profiles/unity/plugins/unityshell/ launcher-minimize-window true`，此方法已经过验证，如不行，则可以尝试`gsettings set org.gnome.shell.extensions.dash-to-dock click-action 'minimize'`，如果要预览是否打开了相同应用程序的多个窗口，请改用以下命令：`gsettings set org.gnome.shell.extensions.dash-to-dock click-action 'minimize-or-overview'`，如果想还原则使用：`gsettings reset org.gnome.shell.extensions.dash-to-dock click-action`。
   - 2. ubuntu16.04系统显示隐藏文件方式为`ctrl + H`，如想永远显示则需另外设置。
   - 3. ubuntu16.04系统开启ssh远程登录。先查看是否安装服务：`apt-cache policy openssh-client openssh-server`。ubuntu默认安装了openssh-client，openssh-server需手动安装：`apt-get install openssh-server`，查看ssh服务开启状况：`ps -e|grep ssh`，如出现sshd则说明服务开启，没有则执行`/etc/init.d/ssh start`开启。
            远程访问方法：`ssh username@host`
   将远程的文件/文件夹保存到本地，使用scp命令：`scp username@host:/home/username/somefile.xlsx /home/localusername/`；如将本地文件/文件夹上传到远程则反过来：`scp /home/localusername/somefile.xlsx username@host:/home/username/(文件保存路径)`。如需复制文件夹则添加`-r`参数以递归方式复制目录。
   - 4. ubuntu16.04安装supervisor。
         - 安装。`sudo apt install supervisor`
         - 配置网页端访问supervisor。在`/etc/supervisor/supervisord.conf`中添加如下：
         ```
           [inet_http_server]
           port=10.234.30.24:9001
           username=user
           password=123
         ```
         并确保该文件中包含`[include]files = /etc/supervisor/conf.d/*.conf`
         - 创建supervisor任务。在`/etc/supervisor/conf.d`中创建myflask.conf，内容如下：
         ```
           [program:myflask]
           command=/home/mi/myflask/venv/bin/uwsgi config.ini
           directory=/home/mi/myflask
           user=mi  # 注：此处的user是ubuntu系统的用户名
           autostart=true
           autorestart=true
           stopasgroup=true
           killasgroup=true
         ```
         如有celery任务，则创建celery.conf，内容如下：
         ```
           [program:celery]
           command=/home/mi/myflask/venv/bin/celery -A app.celery worker -B -l info
           directory=/home/mi/myflask
           user=mi
           autostart=true
           autorestart=true
           stopasgroup=true
           killasgroup=true
         ```
         - 重载supervisor服务：`sudo supervisorctl reload`
                >如报：unix:///var/run/supervisor.sock no such file
                    则：sudo touch /var/run/supervisor.sock
                       sudo chmod 777 /var/run/supervisor.sock
                       sudo service supervisor restart
                >如报：unix:///var/run/supervisor.sock refused connection
                    则：sudo supervisord -c /etc/supervisor/supervisord.conf
         - 查询supervisor开机自启：`systemctl is-enabled supervisord`
         - 设置supervisor开机自启：`sudo systemctl enable supervisor`
   - 5. linux系统的nohup和&后台运行。nohup，不挂断地运行命令。`nohup Command [Arg..] [ &]`输出会附加到当前目录的nohup.out文件中，例`nohup java -jar jenkins.war &`。查看运行的后台进程：`jobs -l`
           注：jobs命令只看当前终端生效的，关闭终端后，在另一个终端jobs无法看到，此时利用ps(进程查看命令)`ps -aux|grep jenkins.war`
   终止后台运行的进程：`kill -9 进程号`。运行nohup命令后，再按下回车，退回命令提示符输入后再退出终端。
   - 6. ubuntu16.04下运行shell脚本(脚本中有字符串截取)报`Bad substitution`，网上解释：
   ```
   从 ubuntu 6.10 开始，ubuntu 就将先前默认的bash shell 更换成了dash shell；其表现为 /bin/sh 链接倒了/bin/dash而不是传统的/bin/bash。
   ubuntu edgy是第一个将dash作为默认shell来发行的版本，这似乎是受了debian的影响。wiki 里面有官方的解释，https://wiki.ubuntu.com/DashAsBinSh，主要原因是dash更小，运行更快，还与POSIX兼容。
   但目前存在的问题是，由于shell的更换，致使很多脚本出错，毕竟现在的很多脚本不是100%POSIX兼容。
   在wiki里面也说到，如何将默认的shell改回bash，方法就是
   在终端执行
   sudo dpkg-reconfigure dash
   然后选 择 no
   ```
   也可以将运行命令改为`bash test.sh`或`zsh test.sh`，或者直接更改test.sh脚本文件为可执行，直接`./test.sh`运行。
   - 7. git相关
      1. git submodule 子模块
         - 为项目添加子模块：`git submodule add https://github.com/somename/`
         - 克隆带子模块的项目：`git clone --recurse-submodule https://github.com/somename/`
                 如果给 git clone 命令传递 --recurse-submodules 选项，它就会自动初始化并更新仓库中的每一个子模块， 包括可能存在的嵌套子模块。
         - 更新子模块：`git submodule update --init --recursive`
         - 拉取项目并更新子模块：`git pull --recurse-submodules`
   - 8. ssh相关
      1. `ssh-keygen -t rsa`命令用于生成密码，`-t`参数指定加密算法为`rsa`。另有`-t dsa`可用。
      2. 自动上传公钥。
         `ssh-copy-id -i ~/.ssh/id_rsa user@host`命令会自动将本地公钥上传到服务器上的`~/.ssh/authorized_keys`文件中，然后使用`ssh user@host`命令登录远程服务器再不需要密码。

# 9. **vue**
   - 1. `vue-cli create`创建的项目，当运行`npm run serve`时报
   ```
      Error from chokidar (/home/teryt/gitlab/work-code/fuyu/VUE/uos_test_vue/abc_quality_backend/node_modules/webpack-dev-server/client/utils): Error: ENOSPC: System limit for number of file watchers reached, watch '/home/teryt/gitlab/work-code/fuyu/VUE/uos_test_vue/abc_quality_backend/node_modules/webpack-dev-server/client/utils/createSocketUrl.js'
      Error from chokidar (/home/teryt/gitlab/work-code/fuyu/VUE/uos_test_vue/abc_quality_backend/node_modules/webpack-dev-server/client/utils): Error: ENOSPC: System limit for number of file watchers reached, watch '/home/teryt/gitlab/work-code/fuyu/VUE/uos_test_vue/abc_quality_backend/node_modules/webpack-dev-server/client/utils/getCurrentScriptSource.js'
      Error from chokidar (/home/teryt/gitlab/work-code/fuyu/VUE/uos_test_vue/abc_quality_backend/node_modules/webpack-dev-server/client/utils): Error: ENOSPC: System limit for number of file watchers reached, watch '/home/teryt/gitlab/work-code/fuyu/VUE/uos_test_vue/abc_quality_backend/node_modules/webpack-dev-server/client/utils/log.js'
   ```
   类似的错误时，需在终端运行`echo fs.inotify.max_user_watches=524288 | sudo tee -a /etc/sysctl.conf && sudo sysctl -p`即可，原因还需查询。