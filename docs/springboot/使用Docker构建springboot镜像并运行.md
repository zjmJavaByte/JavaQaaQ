# 使用Docker构建springboot镜像并运行

## 构建前准备

- 创建文件夹

```sh
#存储日志
mkdir -p /home/docker-project/civil-works-project/logs
#上传文件
mkdir -p /home/docker-project/civil-works-project/upload
#xml配置文件
mkdir -p /home/docker-project/civil-works-project/xml
#yml配置文件
mkdir -p /home/docker-project/civil-works-project/yml
```

- 将准备的yml文件和xml文件放入相应的文件夹中

application-prod.yml

```yml
server:
  port: 9000
  tomcat:
    max-swallow-size: -1
  error:
    include-exception: true
    include-stacktrace: ALWAYS
    include-message: ALWAYS
  servlet:
    context-path: /civilWorks
    #设置请求响应字符编码
    encoding:
      charset: utf-8
      force: true
      enabled: true
  compression:
    enabled: true #是否开启压缩,默认为false,Spring Boot默认没有启用Http包压缩功能，但是压缩对减少带宽和加快页面加载非常有用。
    min-response-size: 1024 #执行压缩的阈值,默认为2048
    mime-types: application/javascript,application/json,application/xml,text/html,text/xml,text/plain,text/css,image/* #指定要压缩的mime-type,多个以逗号分隔.
spring:
  application:
    name: civil-works
  autoconfigure:
    exclude: com.alibaba.druid.spring.boot.autoconfigure.DruidDataSourceAutoConfigure
  datasource:
    dynamic:
      druid: # 全局druid参数，绝大部分值和默认保持一致。(现已支持的参数如下,不清楚含义不要乱设置)
        # 连接池的配置信息
        # 初始化大小，最小，最大
        initial-size: 5
        min-idle: 5
        maxActive: 20
        # async-init是1.1.4中新增加的配置，如果有initialSize数量较多时，打开会加快应用启动时间
        # async-init: true
        # # 获取连接时最大等待时间，单位毫秒。配置了maxWait之后，缺省启用公平锁，并发效率会有所下降，如果需要可以通过配置useUnfairLock属性为true使用非公平锁。
        maxWait: 60000
        # 配置间隔多久才进行一次检测，检测需要关闭的空闲连接，单位是毫秒
        timeBetweenEvictionRunsMillis: 60000
        # 配置一个连接在池中最小生存的时间，单位是毫秒
        minEvictableIdleTimeMillis: 300000
        # 用来检测连接是否有效的sql，要求是一个查询语句，常用select 'x'。如果validationQuery为null，testOnBorrow、testOnReturn、testWhileIdle都不会起作用。
        validationQuery: SELECT 1
        #单位：秒，检测连接是否有效的超时时间。底层调用jdbc Statement对象的void setQueryTimeout(int seconds)方法
        validation-query-timeout: 6
        #建议配置为true，不影响性能，并且保证安全性。申请连接的时候检测，如果空闲时间大于timeBetweenEvictionRunsMillis，执行validationQuery检测连接是否有效。
        testWhileIdle: true
        #申请连接时执行validationQuery检测连接是否有效，做了这个配置会降低性能。
        testOnBorrow: false
        #归还连接时执行validationQuery检测连接是否有效，做了这个配置会降低性能。
        testOnReturn: false
        # 打开PSCache，并且指定每个连接上PSCache的大小
        # 是否缓存preparedStatement，也就是PSCache。PSCache对支持游标的数据库性能提升巨大，比如说oracle。在mysql下建议关闭。
        poolPreparedStatements: false
          #要启用PSCache，必须配置大于0，当大于0时，poolPreparedStatements自动触发修改为true。在Druid中，不会存在Oracle下PSCache占用内存过多的问题，可以把这个数值配置大一些，比如说100
          #maxPoolPreparedStatementPerConnectionSize: 20

          # 配置监控统计拦截的filters，去掉后监控界面sql无法统计，'wall'用于防火墙
          # 通过别名的方式配置扩展插件，多个英文逗号分隔，常用的插件有：
        # 监控统计用的filter:stat
        # 日志用的filter:log4j
        # 防御sql注入的filter:wall
        # filters: stat,wall,slf4j
        filters: stat,slf4j #开发环境去掉 wall 原因请查看： https://blog.csdn.net/u011818862/article/details/108838719
        # 通过connectProperties属性来打开mergeSql功能；慢SQL记录
        connectionProperties: druid.stat.mergeSql\=true;druid.stat.slowSqlMillis\=5000
      datasource:
        master:
          url: jdbc:mysql://localhost:3306/civil_works?characterEncoding=UTF-8&useUnicode=true&useSSL=false&tinyInt1isBit=false&allowPublicKeyRetrieval=true&serverTimezone=Asia/Shanghai
          username: 你的账号
          password: '你的密码'
          driver-class-name: com.mysql.cj.jdbc.Driver
          # SpringBoot 2.0 以上默认使用com.zaxxer.hikari.HikariDataSource数据源，但可以通过spring.datasource.type指定数据源
          #type: com.alibaba.druid.pool.DruidDataSource #自定义数据源
          # 多数据源配置
          #multi-datasource1:
          #url: jdbc:mysql://localhost:3306/jeecg-boot2?useUnicode=true&characterEncoding=utf8&autoReconnect=true&zeroDateTimeBehavior=convertToNull&transformedBitIsBoolean=true&allowPublicKeyRetrieval=true&serverTimezone=Asia/Shanghai
          #username: root
          #password: root
          #driver-class-name: com.mysql.cj.jdbc.Driver
  servlet:
    multipart:
      enabled: true #是否启用http上传处理
      max-request-size: 100MB #最大请求文件的大小
      max-file-size: 20MB #设置单个文件最大长度
      file-size-threshold: 20MB #当文件达到多少时进行磁盘写入
  web:
    resources:
      static-locations: classpath:/static/,classpath:/public/ #设置静态资源路径
  redis:
    host: 112.65.125.190
    database: 8
    lettuce:
      pool:
        max-active: 8   #最大连接数据库连接数,设 -1 为没有限制
        max-idle: 8     #最大等待连接中的数量,设 0 为没有限制
        max-wait: -1ms  #最大建立连接等待时间。如果超过此时间将接到异常。设为-1表示无限制。
        min-idle: 0     #最小等待连接中的数量,设 0 为没有限制
      shutdown-timeout: 100ms
    port: 26379
  cache:
    redis:
      time-to-live: 3600000
      #key-prefix: CACHE_
      #use-key-prefix: true
      cache-null-values: true
    type: redis
  mvc:
    static-path-pattern: /**
    #Spring Boot 2.6+后映射匹配的默认策略已从AntPathMatcher更改为PathPatternParser,需要手动指定为ant-path-matcher
    pathmatch:
      matching-strategy: ant_path_matcher
  quartz:
    job-store-type: jdbc
    #定时任务启动开关，true-开  false-关
    auto-startup: false
    #启动时更新己存在的Job
    overwrite-existing-jobs: true
    wait-for-jobs-to-complete-on-shutdown: true # 关闭时等待任务完成
    properties:
      org:
        quartz:
          scheduler:
            instanceName: MyScheduler
            instanceId: AUTO
          jobStore:
            class: org.springframework.scheduling.quartz.LocalDataSourceJobStore
            driverDelegateClass: org.quartz.impl.jdbcjobstore.StdJDBCDelegate
            tablePrefix: QRTZ_
            isClustered: true #为 true 表示开启集群功能，默认为 false
            misfireThreshold: 60000
            clusterCheckinInterval: 10000
          threadPool:
            class: org.quartz.simpl.SimpleThreadPool
            threadCount: 10
            threadPriority: 5
            threadsInheritContextClassLoaderOfInitializingThread: true
    jdbc:
      initialize-schema: embedded
  #json 时间戳统一转换
  jackson:
    date-format: yyyy-MM-dd HH:mm:ss
    time-zone: GMT+8
mybatis-plus:
  #mybatis.mapper-locations在SpringBoot配置文件中使用，作用是扫描Mapper接口对应的XML文
  mapper-locations: classpath*:/mapper/*.xml
  global-config:
    # 关闭MP3.0自带的banner
    banner: false
    db-config:
      #主键类型  0:"数据库ID自增",1:"该类型为未设置主键类型", 2:"用户输入ID",3:"全局唯一ID (数字类型唯一ID)", 4:"全局唯一ID UUID",5:"字符串全局唯一ID (idWorker 的字符串表示)";
      id-type: ASSIGN_ID
      # 默认数据库表下划线命名
      table-underline: true
      logic-delete-value: 1 # 逻辑已删除值(默认为 1)
      logic-not-delete-value: 0 # 逻辑未删除值(默认为 0)
  configuration:
    # 这个配置会将执行的sql打印出来，在开发或测试的时候可以用
    #log-impl: org.apache.ibatis.logging.stdout.StdOutImpl
    # 返回类型为Map,显示null对应的字段
    call-setters-on-nulls: true
cw:
  path:
    #文件上传根目录 设置
    upload: /civil-work/upload
    #文件上传根目录 设置
    download: /download/**
    #webapp文件路径
    webapp: /opt/webapp
    #日志文件路径
    logs: /civil-work/logs
  # 本地：local\Minio：minio\阿里云：alioss
  uploadType: alioss
  #阿里云oss存储和大鱼短信秘钥配置
  oss:
    accessKey: ?
    secretKey: ?
    endpoint: ?
    bucketName: ?
  # minio文件上传
  minio:
    minio_url: http://minio.jeecg.com
    minio_name: ??
    minio_pass: ??
  # 是否启用安全模式
  safeMode: true
  # 签名密钥串(前后端要一致，正式发布请自行修改)
  signatureSecret: dd05f1c54d63749eda95f9fa6d49v442a
  shiro:
    excludeUrls: /test/jeecgDemo/demo3,/test/jeecgDemo/redisDemo/**,/category/**,/visual/**,/map/**,/jmreport/bigscreen2/**
    bucketName: otatest
#swagger
knife4j:
  #开启增强配置
  enable: true
  #开启生产环境屏蔽
  production: false
  basic:
    enable: true
    username: admin
    password: admin
# 开启 Actuator 监控
management:
  endpoint:
    shutdown:
      enabled: true #用来关闭服务，开启远程关闭功能
  endpoints:
    web:
      exposure:
        include: '*' #开启所有端点
    jmx:
      exposure:
        include: '*'
  metrics:
    tags:
      application: ${spring.application.name}

logging:
  config: classpath:logback-spring-prod.xml
```

application.yml

```yml
spring:
  application:
    name: day-by-day
  profiles:
    active: '@profile.name@'
```

logback-spring-prod.xml

```sh
<?xml version="1.0" encoding="UTF-8"?>
<!--
scan：当此属性设置为true时,配置文件如果发生改变,将会被重新加载,默认值为true。
scanPeriod：设置监测配置文件是否有修改的时间间隔,如果没有给出时间单位,默认单位是毫秒当scan为true时,此属性生效。默认的时间间隔为1分钟。
debug：当此属性设置为true时,将打印出logback内部日志信息,实时查看logback运行状态。默认值为false。
-->
<configuration scan="true" scanPeriod="60 seconds" debug="false">
	<springProperty scope="context" name="logPath" source="cw.path.logs" defaultValue="/civil-work/logs"/>

	<!-- 定义日志的根目录为项目的根目录,前面不要加"/",加了会默认会认为是根目录,提示 classnotfond -->
	<property name="LOG_HOME" value="${logPath}" />
	<!--<property name="LOG_HOME" value="/logs" />-->

	<!-- 定义日志文件名称 -->
	<property name="appName" value="logbackBootText"/>

	<!-- ConsoleAppender 用于控制台输出 -->
	<!-- 控制台输出 -->
	<appender name="STDOUT" class="ch.qos.logback.core.ConsoleAppender">
		<encoder class="ch.qos.logback.classic.encoder.PatternLayoutEncoder">
			<!--格式化输出：%d表示日期，%thread表示线程名，%-5level：级别从左显示5个字符宽度%msg：日志消息，%n是换行符
			<pattern>%d{yyyy-MM-dd HH:mm:ss.SSS} [%thread] %-5level %logger{50}:%L - %msg%n</pattern>-->
			<pattern>%d{yyyy-MM-dd HH:mm:ss.SSS,CTT} [%thread] %highlight(%-5level) %cyan(%logger{50}:%L) - %msg%n</pattern>
		</encoder>
	</appender>

	<!-- 按照每天生成日志文件 -->
	<appender name="FILE" class="ch.qos.logback.core.rolling.RollingFileAppender">
		<rollingPolicy class="ch.qos.logback.core.rolling.SizeAndTimeBasedRollingPolicy">
			<!--日志文件输出的文件名 -->
			<FileNamePattern>${LOG_HOME}/civilWorks-%d{yyyy-MM-dd,CTT}.%i.log</FileNamePattern>
			<!--日志文件保留天数 -->
			<MaxHistory>30</MaxHistory>
			<maxFileSize>5MB</maxFileSize>
		</rollingPolicy>
		<encoder class="ch.qos.logback.classic.encoder.PatternLayoutEncoder">
			<!--格式化输出：%d表示日期，%thread表示线程名，%-5level：级别从左显示5个字符宽度%msg：日志消息，%n是换行符 -->
			<pattern>%d{yyyy-MM-dd HH:mm:ss.SSS,CTT} [%thread] %-5level %logger{50}:%L - %msg%n</pattern>
		</encoder>
	</appender>

	<!-- 生成 error html格式日志开始 -->
	<appender name="HTML" class="ch.qos.logback.core.FileAppender">
		<filter class="ch.qos.logback.classic.filter.ThresholdFilter">
			<!--设置日志级别,过滤掉info日志,只输入error日志-->
			<level>ERROR</level>
		</filter>
		<encoder class="ch.qos.logback.core.encoder.LayoutWrappingEncoder">
			<layout class="ch.qos.logback.classic.html.HTMLLayout">
				<pattern>%p%d%msg%M%F{32}%L</pattern>
			</layout>
		</encoder>
		<file>${LOG_HOME}/error-log.html</file>
	</appender>
	<!-- 生成 error html格式日志结束 -->

	<!-- 每天生成一个html格式的日志开始 -->
	<appender name="FILE_HTML" class="ch.qos.logback.core.rolling.RollingFileAppender">
		<rollingPolicy class="ch.qos.logback.core.rolling.SizeAndTimeBasedRollingPolicy">
			<!--日志文件输出的文件名 -->
			<FileNamePattern>${LOG_HOME}/civilWorks-%d{yyyy-MM-dd,CTT}.%i.html</FileNamePattern>
			<!--日志文件保留天数 -->
			<MaxHistory>30</MaxHistory>
			<MaxFileSize>5MB</MaxFileSize>
		</rollingPolicy>
		<encoder class="ch.qos.logback.core.encoder.LayoutWrappingEncoder">
			<layout class="ch.qos.logback.classic.html.HTMLLayout">
				<pattern>%p%d%msg%M%F{32}%L</pattern>
			</layout>
		</encoder>
	</appender>
	<!-- 每天生成一个html格式的日志结束 -->


	<!--输出到logstash的appender 结束-->
	<appender name="LOGSTASH" class="net.logstash.logback.appender.LogstashTcpSocketAppender">
		<!--可以访问的logstash日志收集端口-->
		<destination>172.28.128.216:4560</destination>
		<encoder charset="UTF-8" class="net.logstash.logback.encoder.LogstashEncoder"/>
	</appender>
	<!--输出到logstash的appender  结束-->
	<!--myibatis log configure -->
	<logger name="com.apache.ibatis" level="TRACE" />
	<logger name="java.sql.Connection" level="DEBUG" />
	<logger name="java.sql.Statement" level="DEBUG" />
	<logger name="java.sql.PreparedStatement" level="DEBUG" />

	<!-- 日志输出级别 -->
	<root level="INFO">
		<appender-ref ref="STDOUT" />
		<appender-ref ref="FILE" />
		<appender-ref ref="HTML" />
		<appender-ref ref="FILE_HTML" />
		<appender-ref ref="LOGSTASH"/>
	</root>

</configuration>
```

## 使用maven打包应用

![image-20220608174240377](https://zjm-picgo.oss-cn-shanghai.aliyuncs.com/202206081938019.png)

- 将打包好的jar上传到`linux`服务器的`/home/docker-project/civil-works-project`文件夹下

## 编写Dockerfile文件

```dockerfile
# 该镜像需要依赖的基础镜像
FROM java:8
# 将当前目录下的jar包复制到docker容器的/目录下
ADD civil-works.jar /civil-works.jar
# 运行过程中创建一个civil-works.jar文件
RUN bash -c 'touch /civil-works.jar'
#挂载当前系统的时区
RUN /bin/cp /usr/share/zoneinfo/Asia/Shanghai /etc/localtime
RUN echo "Asia/Shanghai" > /etc/timezone
# 声明服务运行在8080端口
EXPOSE 8988
# 指定docker容器启动时运行jar包   --spring.config.location=指定配置文件的位置
ENTRYPOINT ["java","-Xms256m","-Xmx256m","-jar","/civil-works.jar","--spring.config.location=/civil-work/yml/"]
# 指定维护者的名字
MAINTAINER zjm
```

- 将应用`Dockerfile`文件上传到`linux`服务器的`/home/docker-project/civil-works-project`文件夹下

## 构建镜像

在`Dockerfile`所在目录执行以下命令

```sh
#-t 表示指定镜像仓库名称/镜像名称:镜像标签 `.`表示使用当前目录下的Dockerfile
docker build -t civil-worksy/civil-works:0.0.1-SNAPSHOT .
```

![image-20220608174658006](https://zjm-picgo.oss-cn-shanghai.aliyuncs.com/202206081938063.png)

![image-20220608174731677](https://zjm-picgo.oss-cn-shanghai.aliyuncs.com/202206081938101.png)

- 运行项目

```sh
docker run -p 9000:9000 --name civil-works \
-v /home/docker-project/civil-works-project/upload:/civil-work/upload \
-v /home/docker-project/civil-works-project/logs:/civil-work/logs \
-v /home/docker-project/civil-works-project/xml:/civil-work/xml \
-v /home/docker-project/civil-works-project/yml:/civil-work/yml \
-v /etc/localtime:/etc/localtime \
-d civil-worksy/civil-works:0.0.1-SNAPSHOT
```