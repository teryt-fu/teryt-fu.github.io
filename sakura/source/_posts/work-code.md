---
title: 工作笔记
date: 2019-12-30 13:09:29
tags:
- python
- robotframework
- java
- docker
- hexo
categories:
- work-code
---
# 1. **monitor平台**
   - 1.系统传参。
      在monitor平台中通过接口管理传递参数时，在robot测试用例中必须包含*** Variables ***字段，哪怕用例中实际用不到也得添加。传递其他参数时同样通过接口管理的param下的param里添加key-value的形式添加参数名和值，然后在测试计划的配置列表中添加param字段，在里面的value字段中添加key-value，同样测试用例中需有Variables，用例中使用${key}来使用此参数。
   - 2.基本操作。
      添加的测试用例必须符合格式，具体为[]括起，每句用例用""括起，除最后一句外，句尾加'，'号。句中注意特殊字符需要\转义。目前已知"和\两个需要转义。
   - 3.本地monitor平台。
      本地平台地址：`http://host:8080/result/prod/6`, 其中host根据丽姐本地url变化。访问时需手动两次更改localhost为实际url，登录名密码均为邮箱前缀。
# 2. **jenkins平台**
   - 1.python版本选择
      `ln -s /usr/bin/python3.6 /usr/bin/python` 通过这条命令，把需要jenkins使用的pythonX 变更为python
   - 2.环境依赖
      jenkins和本地环境不一致，可直接使用jenkins shell脚本在jenkins添加环境
# 3. **python相关**
   - 1、在 python2中，str 其实是 bytes，而不是 unicode，在代码中声明了编码方式为 utf-8，并将该参数存入到了 DB 中，导致下次请求传递的还是 DB 中的 utf-8 类型的 port，而不是 int 或者 string，port给int型
   - 2、配置文件ini中不得加上`coding: utf-8`。python函数获取当前文件名的方法是：`os.path.basename(sys.argv[0]).split('.')[0]`，但当用其他文件中方法调用此函数时不能返回当前文件名，此时需用：`__file__.split('/')[-1].split('.')[0]`。
   - 3、对于windows系统，python2会产生编码问题，此时需加上：`reload(sys)  \ sys.setdefaultencoding('utf-8')`来重置系统默认编码。
   - 4、对于配置文件，当调用时`config.read(filename)`，此时可用`os.path.dirname(os.path.realpath(__file__)) + filename`来获取绝对的文件路径。
   - 5、requests访问https时，如遇SSL或CA证书验证失败，可在发送请求时将verify参数设置为False，默认为True，开启证书验证。对于有登录要求的，可以抓包后将cookie放置在headers里，实例化requests.session()对象后通过session对象发请求。
   - 6、python调用java的jpype模块安装及使用注意：
      - 安装：`pip install jpype1`
      - 使用：
         这里的ext_classpath指的是.class文件的的引用路径之前的路径，如：Javatest.class文件的全路径是：`D:\code\H5\run\demo\src\com\Javatest.class`，Javatest类的包路径（看上面的目录结构）是com，所以此处`ext_classpath='D:\code\H5\run\demo\src'`；JClass的路径就是Javatest类的包路径：JClass('com.Javatest')
         ![解释1](/images/carbon11.png)
         ![解释2](/images/carbon12.png)
   - 7、python写入文件按照列表的方式读取， 每写入一条数据需写入一个换行符隔开，读取出来则是以列表的格式，filehandle.write('\n')
   - 8、thrift接口，客服端调用报错 ’‘TSocket’‘读取0字节，服务端报错 ''No handlens could be found for logger 'Thrift.server.TServer'''，只要是Handlens类报错，提示全都是这个，实际最后找到报错为返回类型和idl文件定义不一致，包括某些变量拼写错误也会出现这种情况
   - 9、flask-sqlalchemy对已生成的表字段做修改，在migrations/env.py文件，在run_migrations_online函数加入![图](/images/carbon18.png)
# 4. **MYSQL数据库**
   - 1、查询数据库中的状态
     >use databasename;
     >show processlist;  state显示lock或waiting的可以杀死
     >kill id;
# 5. **java相关**
   - 1、maven项目需将所有依赖的jar包打包到lib目录：`mvn dependency:copy-dependencies -DoutputDirectory=target/lib`
   - 2、java的 ‘==’和‘equals()’方法，== 对于基本类型来说是值比较，对于引用类型来说是比较的是引用；而 equals 默认情况下是引用比较，只是很多类重写了 equals 方法，比如 String、Integer 等把它变成了值比较，所以一般情况下 equals 比较的是值是否相等。
# 6. **docker相关**
   - docker基础操作
     1.查询镜像- 在docker hub中搜索，或者命令行`docker search imagename`
     2.下载镜像-`docker pull imagename`
     3.查询本地镜像-`docker images`
     4.启动镜像-`docker run -it REPOSITORY:TAG bash`;  显式运行镜像bash命令行;  -i 表示持续打开 STDIN(标准输入);-t 表示申请一个 tty 给这个 docker 使用，同样是为了能够交互，所以一般情况下都是一起使用; -d后台运行。-p port:port（前面是宿主机端口，后面是容器里监听端口，例：`docker run -d -p 40000:40000 REPOSITORY:TAG python3 /home/mi/test/app.py`，此时可通过宿主机ip:40000端口访问容器里的:40000端口的web服务）
     5.退出容器-`exit`或者ctrl+d
     6.查看容器-`docker ps`(运行中);  `docker ps -a`(本地存在的容器)
     7.启动容器/进入后台容器/停止容器-`docker start CONTAINER ID`;  `docker exec -it CONTAINER ID bash`;  `docker stop CONTAINER ID`;   注：使用start启动的容器会进入后台运行状态
     8.删除容器-`docker rm CONTAINER ID`
     9.删除镜像-`docker rmi REPOSITORY:TAG`
   - windows10系统加速源填写的格式错误会导致docker不可用，正确方式为在系统右下角托盘 Docker 图标内右键菜单选择 Settings，打开配置窗口后左侧导航菜单选择 Daemon。在 Registrymirrors 一栏中填写加速器地址 `https://registry.docker-cn.com` ，之后点击 Apply 保存后 Docker 就会重启并应用配置的镜像地址
ubuntu系统加速方式为，更换为国内的镜像作为加速器，首先打开配置文件,配置文件如果不存在则新建：`vi /etc/docker/daemon.json` ;加入如下内容：`{ "registry-mirrors": ["https://docker.mirrors.ustc.edu.cn"]}`;重启docker: `service docker restart`
   - ubuntu:16.04
     1、运行：docker run -it ubuntu:16.04 bash;    注：-it是前端交互，:16.04是tag，不写默认latest，会重新从docker hub拉取最新的ubuntu镜像，bash指开启bash。
     2、安装python3：apt-get update(更新库);  apt-get install python3;  apt-get install python3-pip;
         安装pip3 install mysql时可能会报'Command "python setup.py egg_info" failed with error code 1 in /tmp/pip-install-d27dt_9j/mysqlclient/' 此时先：'pip3 install --upgrade setuptools'，再：'apt install libmysqlclient-dev python-mysqldb'，如还报：'''Command "/usr/bin/python3 -u -c "import setuptools, tokenize;file='/tmp/pip-install-agcpvfop/mysqlclient/setup.py';f=getattr(tokenize, 'open', open)(file);code=f.read().replace('\r\n', '\n');f.close();exec(compile(code, file, 'exec'))" install --record /tmp/pip-record-cuzgf87x/install-record.txt --single-version-externally-managed --compile" failed with error code 1 in /tmp/pip-install-agcpvfop/mysqlclient/'''，则再：'apt-get install libpcap-dev libpq-dev'，就可成功安装
     3、安装mysql：apt-get install mysql-server mysql-client;   如报错:ERROR 2002 (HY000): Can't connect to local MySQL server through socket '/var/run/mysqld/mysqld.sock'，则：cd /etc/init.d;  service mysql stop;  service mysql start
     4、查询ip：ip address;
     5、保存容器：docker commit -m '解释语句' CONTAINER ID REPOSITORY:TAG;     例如：docker commit -m 'add python' bac0551903dd yourname/ubuntu:1.0.0 
         命令：-a :提交的镜像作者；-c :使用Dockerfile指令来创建镜像；-m :提交时的说明文字；-p :在commit时，将容器暂停
     6、推送到远程：注册账号，security里创建access token，命令行docker login --username yourname，提示输入password时贴入access token。docker push REPOSITORY:TAG。若repository不是以yourname命名，则需先docker tag REPOSITORY:TAG yourname/REPOSITORY:TAG
   - jenkins
     1、拉取jenkins镜像：  $docker pull jenkinsci/jenkins  该版本为最新版本
         完成后使用docker images查看是否拉取成功运行jenkins容器： $docker run -p 10086:8080 -p 50000:50000  -v loaclpath:contatinerPath jenkinsci/jenkins   (-p选项，前面是宿主机映射的端口，后面是jenkins容器默认端口，-v选项，前面是本地目录，后面是jenkins容器默认目录，可以将本地目录挂载到容器里)在网页输入localhost:8080打开jenkins,输入控制台提示的administrator password，设置admin用户名和密码，进入jenkins图形界面，成功！！！
     2、安装插件，如果失败，两种解决方案
         一、更新下载插件镜像地址，点击Manage Jenkins→Manager Plugin→Advanced下面的Update site框输入新的镜像地址，镜像地址查询：`http://mirrors.jenkins-ci.org/status.html`
         二、到`https://wiki.jenkins-ci.org/display/JENKINS/Plugins` 网站，手动下载需要的插件，然后在系统管理–管理插件–高级–上传插件即可，点击上传，然后它会自动上传及安装，待jenkins重启后插件即生效
     3、当前项目jenkins需要使用到的插件有：
           gitlab、Gitlab Authentication、Gitlab Hook Plugin、Groovy、Groovy Postbuild、Robot Framework（机器人日志）、Email Extension（邮件配置）、build timeout（构建超时）、（chinese可选）、Email Extension Template Plugin(邮件扩展模板插件)、HTML Publisher (发布HTML报告)、Post build task(该插件允许根据构建日志输出执行批处理/ shell任务)、Naginator（设置失败后重新构建或其他动作）、Startup Trigger（设置启动时构建）、Workspace Cleanup（删除项目工作区)、email-ext（邮件模板配置）、Matrix Authorization Strategy（设置项目矩阵授权策略）
     4、部分功能参考https://www.cnblogs.com/luchuangao/p/7748575.html#_label19
# 7. **hexo搭建gitpage**
   - 1、md文章导入图片方法：在/目录source文件夹下新建images文件夹，将图片放入，md引用时使用`/images/..`
   - 2、评论和自定义域名功能暂未实现。
   - 3、推送源码到github。
      使用hexo deploy推送到github的实现是hexo自动生成的public文件夹里的内容。如果想推送源码，可以先新建分支，然后再用git push提交。
          注：在根目录。
      ```
       1. git checkout -b source  # 切换到一个新分支'source'
       2. git add -A  # 添加源码
       3. git commit -m 'commit'  # 提交
       4. git push origin source  # 提交分支
      ```
      推送后在仓库的Settings中的Branches中更改默认分支显示即可。