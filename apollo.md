#apollo分布式配置分离部署文档：
文档参考自：https://github.com/ctripcorp/apollo/wiki/%E5%88%86%E5%B8%83%E5%BC%8F%E9%83%A8%E7%BD%B2%E6%8C%87%E5%8D%97

##1.环境要求
Java 1.8+
mysql 5.6.5+

目前仅支持4种环境
* DEV  开发环境
* FAT  测试环境，相当于alpha环境(功能测试)
* UAT  集成环境，相当于beta环境（回归测试）
* PRO  生产环境

##2.部署步骤
共三步：
####1.创建数据库
Apollo服务端依赖于MySQL数据库，所以需要事先创建并完成初始化
导入apolloportaldb.sql、apolloconfigdb.sql文件到数据库创建数据库，其中apolloportaldb.sql为管理台数据库，只需要部署一套就行，apolloconfigdb.sql为每种环境的配置数据库，有几套环境就要装几套。比如现在有DEV,PRO两套环境，就要创建两套apolloconfigdb，数据库名称可以自己定义，建议叫ApolloConfigDB_DEV和ApolloConfigDB_PRO方便区分。

配置ApolloPortalDB管理台数据库，通过ApolloPortalDB.ServerConfig表中的apollo.portal.envs可以添加环境，注意目前只支持上面说的四种环境，配置不区分大小写

配置ApolloConfigDB_DEV和ApolloConfigDB_PRO配置数据库，修改ApolloConfigDB_DEV.ServerConfig表中的eureka.service.url为相应环境上的IP地址或域名，多个用逗号隔开，如http://1.1.1.1:8080/eureka/,http://2.2.2.2:8080/eureka/

####2.获取安装包
Apollo服务端安装包共有3个：apollo-configservice, apollo-adminservice, aollo-portal 可以直接下载我们事先打好的安装包，也可以自己通过源码构建，这里使用安装包的形式。

首先在DEV,PRO环境安装客户端程序 apollo-configservice, apollo-adminservice，
解压两个文件后分别打开config/application-github.properties文件修改数据库连接信息
要选择对应环境的数据库，如DEV环境本地数据库：
spring.datasource.url = jdbc:mysql://localhost:3306/ApolloConfigDB_DEV?characterEncoding=utf8
spring.datasource.username = someuser
spring.datasource.password = somepwd

然后安装管理台服务apollo-portal
解压后打开config/application-github.properties修改数据库连接信息，注意这里数据库要选择ApolloportalDB。

最后修改配置数据库属于哪个环境，打开config/apollo-env.properties文件,配置如下
dev.meta=http://1.1.1.1:8080,http://1.1.1.2:8080
pro.meta=http://apollo.xxx.com
一个环境多个主机用逗号隔开


####3.部署Apollo服务端
获取安装包后就可以部署到公司的测试和生产环境了

部署apollo-portal，此程序为管理台服务，只用在生产机器上部署一次就行了，将apollo-portal安装包上传到服务器上，此程序需要占用8070端口，注意防火墙是否开放此端口，修改端口请修改startup.sh脚本的SERVER_PORT配置，执行scripts/startup.sh即可。如需停止服务，执行scripts/shutdown.sh,注意修改startup.sh脚本文件的可执行权限，chmod 0755 startup.sh,如启动失败稍后重试。

部署apollo-adminservice，此程序为接受管理台配置修改信息服务，在所有的环境机器上都要部署，将apollo-adminservice安装包上传到服务器，此程序需要占用8090端口，修改端口请修改startup.sh脚本的SERVER_PORT配置,执行scripts/startup.sh即可。如需停止服务，执行scripts/shutdown.sh

部署apollo-configservice，此程序某环境下的配置信息服务，在所有的环境机器上都要部署，将apollo-configservice安装包上传到服务器，此程序需要占用8080端口，修改端口请修改startup.sh脚本的SERVER_PORT配置,执行scripts/startup.sh即可。如需停止服务，执行scripts/shutdown.sh

##4.客户端使用步骤
以laravel框架为例，把demo上传到服务器主机，修改server，appid,namespaces三个参数，以命令行形式运行 php apollo.php，开启客户端，管理台修改对应的环境配置，客户端主机就可以接收到配置变化并重写配置了。php apollo.php命令只能运行1分钟，所以需要配置定时器去跑这个脚本。如在crontab中添加：
* * * * * /usr/local/php/bin/php /usr/local/apollp/demo/apollo.php，也可以用laravel自带的定时任务。

需要配置下.evn_tpl.php文件，此文件为最后写入.evn文件的模板。详情见demo。