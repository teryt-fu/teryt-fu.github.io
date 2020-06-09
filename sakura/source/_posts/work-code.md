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
# 1. **monitor平台zookeeper平台**
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
   - 9. jenkins+gitlab配置webhook
      首先确认`Gitlab Hook Plugin`和`Build Authorization Token Root Plugin`插件已安装。然后在job配置中勾选`Build when a change is pushed to GitLab. GitLab webhook URL: http://10.234.30.24:8080/project/test_suite`选项，保存GitLab webhook URL待用。在`Enabled GitLab triggers`中勾选第三个`Accepted Merge Request Events`，在高级选项中点`Secret token`后的`Genrate`会生成token，保存待用。在gitlab项目中选settings->Intergrations(集成)，粘贴保存的URL和Secret Token，点击Add webhook，点击Test测试连接即可。
   - 10. jenkins托管flask服务的shell脚本
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
   - 11. jenkins添加用户及配置权限
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
      - 使用：
         这里的ext_classpath指的是.class文件的的引用路径之前的路径，如：Javatest.class文件的全路径是：`D:\code\H5\run\demo\src\com\Javatest.class`，Javatest类的包路径（看上面的目录结构）是com，所以此处`ext_classpath='D:\code\H5\run\demo\src'`；JClass的路径就是Javatest类的包路径：JClass('com.Javatest')
         ![解释1](/images/carbon11.png)
         ![解释2](/images/carbon12.png)
   - 7. python写入文件按照列表的方式读取， 每写入一条数据需写入一个换行符隔开，读取出来则是以列表的格式，filehandle.write('\n')
   - 8. thrift接口，客服端调用报错 ’‘TSocket’‘读取0字节，服务端报错 ''No handlens could be found for logger 'Thrift.server.TServer'''，只要是Handlens类报错，提示全都是这个，实际最后找到报错为返回类型和idl文件定义不一致，包括某些变量拼写错误也会出现这种情况
   - 9. flask-sqlalchemy对已生成的表字段做修改，在migrations/env.py文件，在run_migrations_online函数加入![图](/images/carbon18.png)
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
      - plotly
         依赖orca等。需执行`conda install -c plotly plotly-orca`或`npm install -g electron@6.1.4 orca`，此npm建议nodejs版本>=6.0。实测npm一直报错找不到文件及权限等，即使在命令后加上`--unsafe-perm=true --allow-root`安装成功后依然无法保存plotly绘制的图形，最后安装miniconda使用conda命令安装后可使用。
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
   - 5. mysql各种连接查询
     > 内连接：`select * from a_table (as) a inner join b_table (as) b on a.a_id=b.b_id;`
     > 左(外)连接：`select * from a_table (as) a left (outer) join b_table (as) b on a.a_id=b.b_id;`
     > 右(外)连接：`select * from a_table (as) a right (outer) join b_table (as) b on a.a_id=b.b_id;`
     > 全(外)连接：mysql不支持全外连接，full join，需要左连，右连后再union (all)。

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

# 8. **ubuntu(linux)相关**
   - 1. ubuntu16.04系统，设置点击启动栏图标后应用最小化功能：`gsettings set org.compiz.unityshell:/org/compiz/profiles/unity/plugins/unityshell/ launcher-minimize-window true`，此方法已经过验证，如不行，则可以尝试`gsettings set org.gnome.shell.extensions.dash-to-dock click-action 'minimize'`，如果要预览是否打开了相同应用程序的多个窗口，请改用以下命令：`gsettings set org.gnome.shell.extensions.dash-to-dock click-action 'minimize-or-overview'`，如果想还原则使用：`gsettings reset org.gnome.shell.extensions.dash-to-dock click-action`。
   - 2. ubuntu16.04系统显示隐藏文件方式为`ctrl + H`，如想永远显示则需另外设置。
   - 3. ubuntu16.04系统开启ssh远程登录。先查看是否安装服务：`apt-cache policy openssh-client openssh-server`。ubuntu默认安装了openssh-client，openssh-server需手动安装：`apt-get install openssh-server`，查看ssh服务开启状况：`ps -e|grep ssh`，如出现sshd则说明服务开启，没有则执行`/etc/init.d/ssh start`开启。
            远程访问方法：`ssh username@host`
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
   - 5. linux系统的nohup和&后台运行。nohup，不挂断地运行命令。`nohup Command [Arg..] [ &]`输出会附加到当前目录的nohup.out文件中。查看运行的后台进程：`jobs -l`
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