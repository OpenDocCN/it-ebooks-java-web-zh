# X.附录

### X.附录

# 附录 A. 常见应用属性

### 附录 A. 常见应用属性

你可以在`application.properties/application.yml`文件内部或通过命令行开关来指定各种属性。本章节提供了一个常见 Spring Boot 属性的列表及使用这些属性的底层类的引用。

**注**：属性可以来自 classpath 下的其他 jar 文件中，所以你不应该把它当成详尽的列表。定义你自己的属性也是相当合法的。

**注**：示例文件只是一个指导。不要拷贝/粘贴整个内容到你的应用，而是只提取你需要的属性。

```java
# ===================================================================
# COMMON SPRING BOOT PROPERTIES
#
# This sample file is provided as a guideline. Do NOT copy it in its
# entirety to your own application.               ^^^
# ===================================================================

# ----------------------------------------
# CORE PROPERTIES
# ----------------------------------------

# SPRING CONFIG (ConfigFileApplicationListener)
spring.config.name= # config file name (default to 'application')
spring.config.location= # location of config file

# PROFILES
spring.profiles.active= # comma list of active profiles
spring.profiles.include= # unconditionally activate the specified comma separated profiles

# APPLICATION SETTINGS (SpringApplication)
spring.main.sources=
spring.main.web-environment= # detect by default
spring.main.show-banner=true
spring.main....= # see class for all properties

# LOGGING
logging.path=/var/logs
logging.file=myapp.log
logging.config= # location of config file (default classpath:logback.xml for logback)
logging.level.*= # levels for loggers, e.g. "logging.level.org.springframework=DEBUG" (TRACE, DEBUG, INFO, WARN, ERROR, FATAL, OFF)

# IDENTITY (ContextIdApplicationContextInitializer)
spring.application.name=
spring.application.index=

# EMBEDDED SERVER CONFIGURATION (ServerProperties)
server.port=8080
server.address= # bind to a specific NIC
server.session-timeout= # session timeout in seconds
server.context-parameters.*= # Servlet context init parameters, e.g. server.context-parameters.a=alpha
server.context-path= # the context path, defaults to '/'
server.servlet-path= # the servlet path, defaults to '/'
server.ssl.enabled=true # if SSL support is enabled
server.ssl.client-auth= # want or need
server.ssl.key-alias=
server.ssl.ciphers= # supported SSL ciphers
server.ssl.key-password=
server.ssl.key-store=
server.ssl.key-store-password=
server.ssl.key-store-provider=
server.ssl.key-store-type=
server.ssl.protocol=TLS
server.ssl.trust-store=
server.ssl.trust-store-password=
server.ssl.trust-store-provider=
server.ssl.trust-store-type=
server.tomcat.access-log-pattern= # log pattern of the access log
server.tomcat.access-log-enabled=false # is access logging enabled
server.tomcat.compression=off # is compression enabled (off, on, or an integer content length limit)
server.tomcat.compressable-mime-types=text/html,text/xml,text/plain # comma-separated list of mime types that Tomcat will compress
server.tomcat.internal-proxies=10\\.\\d{1,3}\\.\\d{1,3}\\.\\d{1,3}|\\
        192\\.168\\.\\d{1,3}\\.\\d{1,3}|\\
        169\\.254\\.\\d{1,3}\\.\\d{1,3}|\\
        127\\.\\d{1,3}\\.\\d{1,3}\\.\\d{1,3} # regular expression matching trusted IP addresses
server.tomcat.protocol-header=x-forwarded-proto # front end proxy forward header
server.tomcat.port-header= # front end proxy port header
server.tomcat.remote-ip-header=x-forwarded-for
server.tomcat.basedir=/tmp # base dir (usually not needed, defaults to tmp)
server.tomcat.background-processor-delay=30; # in seconds
server.tomcat.max-http-header-size= # maximum size in bytes of the HTTP message header
server.tomcat.max-threads = 0 # number of threads in protocol handler
server.tomcat.uri-encoding = UTF-8 # character encoding to use for URL decoding

# SPRING MVC (WebMvcProperties)
spring.mvc.locale= # set fixed locale, e.g. en_UK
spring.mvc.date-format= # set fixed date format, e.g. dd/MM/yyyy
spring.mvc.favicon.enabled=true
spring.mvc.message-codes-resolver-format= # PREFIX_ERROR_CODE / POSTFIX_ERROR_CODE
spring.mvc.ignore-default-model-on-redirect=true # If the the content of the "default" model should be ignored redirects
spring.view.prefix= # MVC view prefix
spring.view.suffix= # ... and suffix

# SPRING RESOURCES HANDLING (ResourceProperties)
spring.resources.cache-period= # cache timeouts in headers sent to browser
spring.resources.add-mappings=true # if default mappings should be added

# MULTIPART (MultipartProperties)
multipart.enabled=true
multipart.file-size-threshold=0 # Threshold after which files will be written to disk.
multipart.location= # Intermediate location of uploaded files.
multipart.max-file-size=1Mb # Max file size.
multipart.max-request-size=10Mb # Max request size.

# SPRING HATEOAS (HateoasProperties)
spring.hateoas.apply-to-primary-object-mapper=true # if the primary mapper should also be configured

# HTTP encoding (HttpEncodingProperties)
spring.http.encoding.charset=UTF-8 # the encoding of HTTP requests/responses
spring.http.encoding.enabled=true # enable http encoding support
spring.http.encoding.force=true # force the configured encoding

# HTTP message conversion
spring.http.converters.preferred-json-mapper= # the preferred JSON mapper to use for HTTP message conversion. Set to "gson" to force the use of Gson when both it and Jackson are on the classpath.

# HTTP response compression (GzipFilterProperties)
spring.http.gzip.buffer-size= # size of the output buffer in bytes
spring.http.gzip.deflate-compression-level= # the level used for deflate compression (0-9)
spring.http.gzip.deflate-no-wrap= # noWrap setting for deflate compression (true or false)
spring.http.gzip.enabled=true # enable gzip filter support
spring.http.gzip.excluded-agents= # comma-separated list of user agents to exclude from compression
spring.http.gzip.excluded-agent-patterns= # comma-separated list of regular expression patterns to control user agents excluded from compression
spring.http.gzip.excluded-paths= # comma-separated list of paths to exclude from compression
spring.http.gzip.excluded-path-patterns= # comma-separated list of regular expression patterns to control the paths that are excluded from compression
spring.http.gzip.methods= # comma-separated list of HTTP methods for which compression is enabled
spring.http.gzip.mime-types= # comma-separated list of MIME types which should be compressed
spring.http.gzip.min-gzip-size= # minimum content length required for compression to occur
spring.http.gzip.vary= # Vary header to be sent on responses that may be compressed

# JACKSON (JacksonProperties)
spring.jackson.date-format= # Date format string (e.g. yyyy-MM-dd HH:mm:ss), or a fully-qualified date format class name (e.g. com.fasterxml.jackson.databind.util.ISO8601DateFormat)
spring.jackson.property-naming-strategy= # One of the constants on Jackson's PropertyNamingStrategy (e.g. CAMEL_CASE_TO_LOWER_CASE_WITH_UNDERSCORES) or the fully-qualified class name of a PropertyNamingStrategy subclass
spring.jackson.deserialization.*= # see Jackson's DeserializationFeature
spring.jackson.generator.*= # see Jackson's JsonGenerator.Feature
spring.jackson.mapper.*= # see Jackson's MapperFeature
spring.jackson.parser.*= # see Jackson's JsonParser.Feature
spring.jackson.serialization.*= # see Jackson's SerializationFeature
spring.jackson.serialization-inclusion= # Controls the inclusion of properties during serialization (see Jackson's JsonInclude.Include)

# THYMELEAF (ThymeleafAutoConfiguration)
spring.thymeleaf.check-template-location=true
spring.thymeleaf.prefix=classpath:/templates/
spring.thymeleaf.excluded-view-names= # comma-separated list of view names that should be excluded from resolution
spring.thymeleaf.view-names= # comma-separated list of view names that can be resolved
spring.thymeleaf.suffix=.html
spring.thymeleaf.mode=HTML5
spring.thymeleaf.encoding=UTF-8
spring.thymeleaf.content-type=text/html # ;charset= <encoding class="hljs-pi">is added
spring.thymeleaf.cache=true # set to false for hot refresh

# FREEMARKER (FreeMarkerAutoConfiguration)
spring.freemarker.allow-request-override=false
spring.freemarker.cache=true
spring.freemarker.check-template-location=true
spring.freemarker.charset=UTF-8
spring.freemarker.content-type=text/html
spring.freemarker.expose-request-attributes=false
spring.freemarker.expose-session-attributes=false
spring.freemarker.expose-spring-macro-helpers=false
spring.freemarker.prefix=
spring.freemarker.request-context-attribute=
spring.freemarker.settings.*=
spring.freemarker.suffix=.ftl
spring.freemarker.template-loader-path=classpath:/templates/ # comma-separated list
spring.freemarker.view-names= # whitelist of view names that can be resolved

# GROOVY TEMPLATES (GroovyTemplateAutoConfiguration)
spring.groovy.template.cache=true
spring.groovy.template.charset=UTF-8
spring.groovy.template.configuration.*= # See Groovy's TemplateConfiguration
spring.groovy.template.content-type=text/html
spring.groovy.template.prefix=classpath:/templates/
spring.groovy.template.suffix=.tpl
spring.groovy.template.view-names= # whitelist of view names that can be resolved

# VELOCITY TEMPLATES (VelocityAutoConfiguration)
spring.velocity.allow-request-override=false
spring.velocity.cache=true
spring.velocity.check-template-location=true
spring.velocity.charset=UTF-8
spring.velocity.content-type=text/html
spring.velocity.date-tool-attribute=
spring.velocity.expose-request-attributes=false
spring.velocity.expose-session-attributes=false
spring.velocity.expose-spring-macro-helpers=false
spring.velocity.number-tool-attribute=
spring.velocity.prefer-file-system-access=true # prefer file system access for template loading
spring.velocity.prefix=
spring.velocity.properties.*=
spring.velocity.request-context-attribute=
spring.velocity.resource-loader-path=classpath:/templates/
spring.velocity.suffix=.vm
spring.velocity.toolbox-config-location= # velocity Toolbox config location, for example "/WEB-INF/toolbox.xml"
spring.velocity.view-names= # whitelist of view names that can be resolved

# JERSEY (JerseyProperties)
spring.jersey.type=servlet # servlet or filter
spring.jersey.init= # init params
spring.jersey.filter.order=

# INTERNATIONALIZATION (MessageSourceAutoConfiguration)
spring.messages.basename=messages
spring.messages.cache-seconds=-1
spring.messages.encoding=UTF-8

# SECURITY (SecurityProperties)
security.user.name=user # login username
security.user.password= # login password
security.user.role=USER # role assigned to the user
security.require-ssl=false # advanced settings ...
security.enable-csrf=false
security.basic.enabled=true
security.basic.realm=Spring
security.basic.path= # /**
security.basic.authorize-mode= # ROLE, AUTHENTICATED, NONE
security.filter-order=0
security.headers.xss=false
security.headers.cache=false
security.headers.frame=false
security.headers.content-type=false
security.headers.hsts=all # none / domain / all
security.sessions=stateless # always / never / if_required / stateless
security.ignored= # Comma-separated list of paths to exclude from the default secured paths

# DATASOURCE (DataSourceAutoConfiguration & DataSourceProperties)
spring.datasource.name= # name of the data source
spring.datasource.initialize=true # populate using data.sql
spring.datasource.schema= # a schema (DDL) script resource reference
spring.datasource.data= # a data (DML) script resource reference
spring.datasource.sql-script-encoding= # a charset for reading SQL scripts
spring.datasource.platform= # the platform to use in the schema resource (schema-${platform}.sql)
spring.datasource.continue-on-error=false # continue even if can't be initialized
spring.datasource.separator=; # statement separator in SQL initialization scripts
spring.datasource.driver-class-name= # JDBC Settings...
spring.datasource.url=
spring.datasource.username=
spring.datasource.password=
spring.datasource.jndi-name= # For JNDI lookup (class, url, username & password are ignored when set)
spring.datasource.max-active=100 # Advanced configuration...
spring.datasource.max-idle=8
spring.datasource.min-idle=8
spring.datasource.initial-size=10
spring.datasource.validation-query=
spring.datasource.test-on-borrow=false
spring.datasource.test-on-return=false
spring.datasource.test-while-idle=
spring.datasource.time-between-eviction-runs-millis=
spring.datasource.min-evictable-idle-time-millis=
spring.datasource.max-wait=
spring.datasource.jmx-enabled=false # Export JMX MBeans (if supported)

# DAO (PersistenceExceptionTranslationAutoConfiguration)
spring.dao.exceptiontranslation.enabled=true

# MONGODB (MongoProperties)
spring.data.mongodb.host= # the db host
spring.data.mongodb.port=27017 # the connection port (defaults to 27107)
spring.data.mongodb.uri=mongodb://localhost/test # connection URL
spring.data.mongodb.database=
spring.data.mongodb.authentication-database=
spring.data.mongodb.grid-fs-database=
spring.data.mongodb.username=
spring.data.mongodb.password=
spring.data.mongodb.repositories.enabled=true # if spring data repository support is enabled

# JPA (JpaBaseConfiguration, HibernateJpaAutoConfiguration)
spring.jpa.properties.*= # properties to set on the JPA connection
spring.jpa.open-in-view=true
spring.jpa.show-sql=true
spring.jpa.database-platform=
spring.jpa.database=
spring.jpa.generate-ddl=false # ignored by Hibernate, might be useful for other vendors
spring.jpa.hibernate.naming-strategy= # naming classname
spring.jpa.hibernate.ddl-auto= # defaults to create-drop for embedded dbs
spring.data.jpa.repositories.enabled=true # if spring data repository support is enabled

# JTA (JtaAutoConfiguration)
spring.jta.log-dir= # transaction log dir
spring.jta.*= # technology specific configuration

# ATOMIKOS
spring.jta.atomikos.connectionfactory.borrow-connection-timeout=30 # Timeout, in seconds, for borrowing connections from the pool
spring.jta.atomikos.connectionfactory.ignore-session-transacted-flag=true # Whether or not to ignore the transacted flag when creating session
spring.jta.atomikos.connectionfactory.local-transaction-mode=false # Whether or not local transactions are desired
spring.jta.atomikos.connectionfactory.maintenance-interval=60 # The time, in seconds, between runs of the pool's maintenance thread
spring.jta.atomikos.connectionfactory.max-idle-time=60 # The time, in seconds, after which connections are cleaned up from the pool
spring.jta.atomikos.connectionfactory.max-lifetime=0 # The time, in seconds, that a connection can be pooled for before being destroyed. 0 denotes no limit.
spring.jta.atomikos.connectionfactory.max-pool-size=1 # The maximum size of the pool
spring.jta.atomikos.connectionfactory.min-pool-size=1 # The minimum size of the pool
spring.jta.atomikos.connectionfactory.reap-timeout=0 # The reap timeout, in seconds, for borrowed connections. 0 denotes no limit.
spring.jta.atomikos.connectionfactory.unique-resource-name=jmsConnectionFactory # The unique name used to identify the resource during recovery
spring.jta.atomikos.datasource.borrow-connection-timeout=30 # Timeout, in seconds, for borrowing connections from the pool
spring.jta.atomikos.datasource.default-isolation-level= # Default isolation level of connections provided by the pool
spring.jta.atomikos.datasource.login-timeout= # Timeout, in seconds, for establishing a database connection
spring.jta.atomikos.datasource.maintenance-interval=60 # The time, in seconds, between runs of the pool's maintenance thread
spring.jta.atomikos.datasource.max-idle-time=60 # The time, in seconds, after which connections are cleaned up from the pool
spring.jta.atomikos.datasource.max-lifetime=0 # The time, in seconds, that a connection can be pooled for before being destroyed. 0 denotes no limit.
spring.jta.atomikos.datasource.max-pool-size=1 # The maximum size of the pool
spring.jta.atomikos.datasource.min-pool-size=1 # The minimum size of the pool
spring.jta.atomikos.datasource.reap-timeout=0 # The reap timeout, in seconds, for borrowed connections. 0 denotes no limit.
spring.jta.atomikos.datasource.test-query= # SQL query or statement used to validate a connection before returning it
spring.jta.atomikos.datasource.unique-resource-name=dataSource # The unique name used to identify the resource during recovery

# BITRONIX
spring.jta.bitronix.connectionfactory.acquire-increment=1 # Number of connections to create when growing the pool
spring.jta.bitronix.connectionfactory.acquisition-interval=1 # Time, in seconds, to wait before trying to acquire a connection again after an invalid connection was acquired
spring.jta.bitronix.connectionfactory.acquisition-timeout=30 # Timeout, in seconds, for acquiring connections from the pool
spring.jta.bitronix.connectionfactory.allow-local-transactions=true # Whether or not the transaction manager should allow mixing XA and non-XA transactions
spring.jta.bitronix.connectionfactory.apply-transaction-timeout=false # Whether or not the transaction timeout should be set on the XAResource when it is enlisted
spring.jta.bitronix.connectionfactory.automatic-enlisting-enabled=true # Whether or not resources should be enlisted and delisted automatically
spring.jta.bitronix.connectionfactory.cache-producers-consumers=true # Whether or not produces and consumers should be cached
spring.jta.bitronix.connectionfactory.defer-connection-release=true # Whether or not the provider can run many transactions on the same connection and supports transaction interleaving
spring.jta.bitronix.connectionfactory.ignore-recovery-failures=false # Whether or not recovery failures should be ignored
spring.jta.bitronix.connectionfactory.max-idle-time=60 # The time, in seconds, after which connections are cleaned up from the pool
spring.jta.bitronix.connectionfactory.max-pool-size=10 # The maximum size of the pool. 0 denotes no limit
spring.jta.bitronix.connectionfactory.min-pool-size=0 # The minimum size of the pool
spring.jta.bitronix.connectionfactory.password= # The password to use to connect to the JMS provider
spring.jta.bitronix.connectionfactory.share-transaction-connections=false #  Whether or not connections in the ACCESSIBLE state can be shared within the context of a transaction
spring.jta.bitronix.connectionfactory.test-connections=true # Whether or not connections should be tested when acquired from the pool
spring.jta.bitronix.connectionfactory.two-pc-ordering-position=1 # The postion that this resource should take during two-phase commit (always first is Integer.MIN_VALUE, always last is Integer.MAX_VALUE)
spring.jta.bitronix.connectionfactory.unique-name=jmsConnectionFactory # The unique name used to identify the resource during recovery
spring.jta.bitronix.connectionfactory.use-tm-join=true Whether or not TMJOIN should be used when starting XAResources
spring.jta.bitronix.connectionfactory.user= # The user to use to connect to the JMS provider
spring.jta.bitronix.datasource.acquire-increment=1 # Number of connections to create when growing the pool
spring.jta.bitronix.datasource.acquisition-interval=1 # Time, in seconds, to wait before trying to acquire a connection again after an invalid connection was acquired
spring.jta.bitronix.datasource.acquisition-timeout=30 # Timeout, in seconds, for acquiring connections from the pool
spring.jta.bitronix.datasource.allow-local-transactions=true # Whether or not the transaction manager should allow mixing XA and non-XA transactions
spring.jta.bitronix.datasource.apply-transaction-timeout=false # Whether or not the transaction timeout should be set on the XAResource when it is enlisted
spring.jta.bitronix.datasource.automatic-enlisting-enabled=true # Whether or not resources should be enlisted and delisted automatically
spring.jta.bitronix.datasource.cursor-holdability= # The default cursor holdability for connections
spring.jta.bitronix.datasource.defer-connection-release=true # Whether or not the database can run many transactions on the same connection and supports transaction interleaving
spring.jta.bitronix.datasource.enable-jdbc4-connection-test # Whether or not Connection.isValid() is called when acquiring a connection from the pool
spring.jta.bitronix.datasource.ignore-recovery-failures=false # Whether or not recovery failures should be ignored
spring.jta.bitronix.datasource.isolation-level= # The default isolation level for connections
spring.jta.bitronix.datasource.local-auto-commit # The default auto-commit mode for local transactions
spring.jta.bitronix.datasource.login-timeout= # Timeout, in seconds, for establishing a database connection
spring.jta.bitronix.datasource.max-idle-time=60 # The time, in seconds, after which connections are cleaned up from the pool
spring.jta.bitronix.datasource.max-pool-size=10 # The maximum size of the pool. 0 denotes no limit
spring.jta.bitronix.datasource.min-pool-size=0 # The minimum size of the pool
spring.jta.bitronix.datasource.prepared-statement-cache-size=0 # The target size of the prepared statement cache. 0 disables the cache
spring.jta.bitronix.datasource.share-transaction-connections=false #  Whether or not connections in the ACCESSIBLE state can be shared within the context of a transaction
spring.jta.bitronix.datasource.test-query # SQL query or statement used to validate a connection before returning it
spring.jta.bitronix.datasource.two-pc-ordering-position=1 # The position that this resource should take during two-phase commit (always first is Integer.MIN_VALUE, always last is Integer.MAX_VALUE)
spring.jta.bitronix.datasource.unique-name=dataSource # The unique name used to identify the resource during recovery
spring.jta.bitronix.datasource.use-tm-join=true Whether or not TMJOIN should be used when starting XAResources

# SOLR (SolrProperties)
spring.data.solr.host=http://127.0.0.1:8983/solr
spring.data.solr.zk-host=
spring.data.solr.repositories.enabled=true # if spring data repository support is enabled

# ELASTICSEARCH (ElasticsearchProperties)
spring.data.elasticsearch.cluster-name= # The cluster name (defaults to elasticsearch)
spring.data.elasticsearch.cluster-nodes= # The address(es) of the server node (comma-separated; if not specified starts a client node)
spring.data.elasticsearch.properties.*= # Additional properties used to configure the client
spring.data.elasticsearch.repositories.enabled=true # if spring data repository support is enabled

# DATA REST (RepositoryRestConfiguration)
spring.data.rest.base-uri= # base URI against which the exporter should calculate its links

# FLYWAY (FlywayProperties)
flyway.*= # Any public property available on the auto-configured `Flyway` object
flyway.check-location=false # check that migration scripts location exists
flyway.locations=classpath:db/migration # locations of migrations scripts
flyway.schemas= # schemas to update
flyway.init-version= 1 # version to start migration
flyway.init-sqls= # SQL statements to execute to initialize a connection immediately after obtaining it
flyway.sql-migration-prefix=V
flyway.sql-migration-suffix=.sql
flyway.enabled=true
flyway.url= # JDBC url if you want Flyway to create its own DataSource
flyway.user= # JDBC username if you want Flyway to create its own DataSource
flyway.password= # JDBC password if you want Flyway to create its own DataSource

# LIQUIBASE (LiquibaseProperties)
liquibase.change-log=classpath:/db/changelog/db.changelog-master.yaml
liquibase.check-change-log-location=true # check the change log location exists
liquibase.contexts= # runtime contexts to use
liquibase.default-schema= # default database schema to use
liquibase.drop-first=false
liquibase.enabled=true
liquibase.url= # specific JDBC url (if not set the default datasource is used)
liquibase.user= # user name for liquibase.url
liquibase.password= # password for liquibase.url

# JMX
spring.jmx.enabled=true # Expose MBeans from Spring

# RABBIT (RabbitProperties)
spring.rabbitmq.host= # connection host
spring.rabbitmq.port= # connection port
spring.rabbitmq.addresses= # connection addresses (e.g. myhost:9999,otherhost:1111)
spring.rabbitmq.username= # login user
spring.rabbitmq.password= # login password
spring.rabbitmq.virtual-host=
spring.rabbitmq.dynamic=

# REDIS (RedisProperties)
spring.redis.database= # database name
spring.redis.host=localhost # server host
spring.redis.password= # server password
spring.redis.port=6379 # connection port
spring.redis.pool.max-idle=8 # pool settings ...
spring.redis.pool.min-idle=0
spring.redis.pool.max-active=8
spring.redis.pool.max-wait=-1
spring.redis.sentinel.master= # name of Redis server
spring.redis.sentinel.nodes= # comma-separated list of host:port pairs

# ACTIVEMQ (ActiveMQProperties)
spring.activemq.broker-url=tcp://localhost:61616 # connection URL
spring.activemq.user=
spring.activemq.password=
spring.activemq.in-memory=true # broker kind to create if no broker-url is specified
spring.activemq.pooled=false

# HornetQ (HornetQProperties)
spring.hornetq.mode= # connection mode (native, embedded)
spring.hornetq.host=localhost # hornetQ host (native mode)
spring.hornetq.port=5445 # hornetQ port (native mode)
spring.hornetq.embedded.enabled=true # if the embedded server is enabled (needs hornetq-jms-server.jar)
spring.hornetq.embedded.server-id= # auto-generated id of the embedded server (integer)
spring.hornetq.embedded.persistent=false # message persistence
spring.hornetq.embedded.data-directory= # location of data content (when persistence is enabled)
spring.hornetq.embedded.queues= # comma-separated queues to create on startup
spring.hornetq.embedded.topics= # comma-separated topics to create on startup
spring.hornetq.embedded.cluster-password= # customer password (randomly generated by default)

# JMS (JmsProperties)
spring.jms.jndi-name= # JNDI location of a JMS ConnectionFactory
spring.jms.pub-sub-domain= # false for queue (default), true for topic

# Email (MailProperties)
spring.mail.host=smtp.acme.org # mail server host
spring.mail.port= # mail server port
spring.mail.username=
spring.mail.password=
spring.mail.default-encoding=UTF-8 # encoding to use for MimeMessages
spring.mail.properties.*= # properties to set on the JavaMail session

# SPRING BATCH (BatchDatabaseInitializer)
spring.batch.job.names=job1,job2
spring.batch.job.enabled=true
spring.batch.initializer.enabled=true
spring.batch.schema= # batch schema to load

# SPRING CACHE (CacheProperties)
spring.cache.type= # generic, ehcache, hazelcast, jcache, redis, guava, simple, none
spring.cache.config= #
spring.cache.cache-names= # cache names to create on startup
spring.cache.jcache.provider= # fully qualified name of the CachingProvider implementation to use
spring.cache.guava.spec= # guava specs

# AOP
spring.aop.auto=
spring.aop.proxy-target-class=

# FILE ENCODING (FileEncodingApplicationListener)
spring.mandatory-file-encoding= # Expected character encoding the application must use

# SPRING SOCIAL (SocialWebAutoConfiguration)
spring.social.auto-connection-views=true # Set to true for default connection views or false if you provide your own

# SPRING SOCIAL FACEBOOK (FacebookAutoConfiguration)
spring.social.facebook.app-id= # your application's Facebook App ID
spring.social.facebook.app-secret= # your application's Facebook App Secret

# SPRING SOCIAL LINKEDIN (LinkedInAutoConfiguration)
spring.social.linkedin.app-id= # your application's LinkedIn App ID
spring.social.linkedin.app-secret= # your application's LinkedIn App Secret

# SPRING SOCIAL TWITTER (TwitterAutoConfiguration)
spring.social.twitter.app-id= # your application's Twitter App ID
spring.social.twitter.app-secret= # your application's Twitter App Secret

# SPRING MOBILE SITE PREFERENCE (SitePreferenceAutoConfiguration)
spring.mobile.sitepreference.enabled=true # enabled by default

# SPRING MOBILE DEVICE VIEWS (DeviceDelegatingViewResolverAutoConfiguration)
spring.mobile.devicedelegatingviewresolver.enabled=true # disabled by default
spring.mobile.devicedelegatingviewresolver.normal-prefix=
spring.mobile.devicedelegatingviewresolver.normal-suffix=
spring.mobile.devicedelegatingviewresolver.mobile-prefix=mobile/
spring.mobile.devicedelegatingviewresolver.mobile-suffix=
spring.mobile.devicedelegatingviewresolver.tablet-prefix=tablet/
spring.mobile.devicedelegatingviewresolver.tablet-suffix=

# ----------------------------------------
# ACTUATOR PROPERTIES
# ----------------------------------------

# MANAGEMENT HTTP SERVER (ManagementServerProperties)
management.port= # defaults to 'server.port'
management.address= # bind to a specific NIC
management.context-path= # default to '/'
management.add-application-context-header= # default to true
management.security.enabled=true # enable security
management.security.role=ADMIN # role required to access the management endpoint
management.security.sessions=stateless # session creating policy to use (always, never, if_required, stateless)

# PID FILE (ApplicationPidFileWriter)
spring.pidfile= # Location of the PID file to write

# ENDPOINTS (AbstractEndpoint subclasses)
endpoints.autoconfig.id=autoconfig
endpoints.autoconfig.sensitive=true
endpoints.autoconfig.enabled=true
endpoints.beans.id=beans
endpoints.beans.sensitive=true
endpoints.beans.enabled=true
endpoints.configprops.id=configprops
endpoints.configprops.sensitive=true
endpoints.configprops.enabled=true
endpoints.configprops.keys-to-sanitize=password,secret,key # suffix or regex
endpoints.dump.id=dump
endpoints.dump.sensitive=true
endpoints.dump.enabled=true
endpoints.env.id=env
endpoints.env.sensitive=true
endpoints.env.enabled=true
endpoints.env.keys-to-sanitize=password,secret,key # suffix or regex
endpoints.health.id=health
endpoints.health.sensitive=true
endpoints.health.enabled=true
endpoints.health.mapping.*= # mapping of health statuses to HttpStatus codes
endpoints.health.time-to-live=1000
endpoints.info.id=info
endpoints.info.sensitive=false
endpoints.info.enabled=true
endpoints.mappings.enabled=true
endpoints.mappings.id=mappings
endpoints.mappings.sensitive=true
endpoints.metrics.id=metrics
endpoints.metrics.sensitive=true
endpoints.metrics.enabled=true
endpoints.shutdown.id=shutdown
endpoints.shutdown.sensitive=true
endpoints.shutdown.enabled=false
endpoints.trace.id=trace
endpoints.trace.sensitive=true
endpoints.trace.enabled=true

# HEALTH INDICATORS (previously health.*)
management.health.db.enabled=true
management.health.elasticsearch.enabled=true
management.health.elasticsearch.response-timeout=100 # the time, in milliseconds, to wait for a response from the cluster
management.health.diskspace.enabled=true
management.health.diskspace.path=.
management.health.diskspace.threshold=10485760
management.health.mongo.enabled=true
management.health.rabbit.enabled=true
management.health.redis.enabled=true
management.health.solr.enabled=true
management.health.status.order=DOWN, OUT_OF_SERVICE, UNKNOWN, UP

# MVC ONLY ENDPOINTS
endpoints.jolokia.path=jolokia
endpoints.jolokia.sensitive=true
endpoints.jolokia.enabled=true # when using Jolokia

# JMX ENDPOINT (EndpointMBeanExportProperties)
endpoints.jmx.enabled=true
endpoints.jmx.domain= # the JMX domain, defaults to 'org.springboot'
endpoints.jmx.unique-names=false
endpoints.jmx.static-names=

# JOLOKIA (JolokiaProperties)
jolokia.config.*= # See Jolokia manual

# REMOTE SHELL
shell.auth=simple # jaas, key, simple, spring
shell.command-refresh-interval=-1
shell.command-path-patterns= # classpath*:/commands/**, classpath*:/crash/commands/**
shell.config-path-patterns= # classpath*:/crash/*
shell.disabled-commands=jpa*,jdbc*,jndi* # comma-separated list of commands to disable
shell.disabled-plugins=false # don't expose plugins
shell.ssh.enabled= # ssh settings ...
shell.ssh.key-path=
shell.ssh.port=
shell.telnet.enabled= # telnet settings ...
shell.telnet.port=
shell.auth.jaas.domain= # authentication settings ...
shell.auth.key.path=
shell.auth.simple.user.name=
shell.auth.simple.user.password=
shell.auth.spring.roles=

# SENDGRID (SendGridAutoConfiguration)
spring.sendgrid.username= # SendGrid account username
spring.sendgrid.password= # SendGrid account password
spring.sendgrid.proxy.host= # SendGrid proxy host
spring.sendgrid.proxy.port= # SendGrid proxy port

# GIT INFO
spring.git.properties= # resource ref to generated git info properties file</encoding> 
```

# 附录 B. 配置元数据

### 附录 B. 配置元数据

Spring Boot jars 包含元数据文件，它们提供了所有支持的配置属性详情。这些文件设计用于让 IDE 开发者能够为使用 application.properties 或 application.yml 文件的用户提供上下文帮助及代码完成功能。

主要的元数据文件是在编译器通过处理所有被`@ConfigurationProperties`注解的节点来自动生成的。

# 附录 B.1\. 元数据格式

### 附录 B.1\. 元数据格式

配置元数据位于 jars 文件中的`META-INF/spring-configuration-metadata.json`，它们使用一个具有"groups"或"properties"分类节点的简单 JSON 格式：

```java
{"groups": [
    {
        "name": "server",
        "type": "org.springframework.boot.autoconfigure.web.ServerProperties",
        "sourceType": "org.springframework.boot.autoconfigure.web.ServerProperties"
    }
    ...
],"properties": [
    {
        "name": "server.port",
        "type": "java.lang.Integer",
        "sourceType": "org.springframework.boot.autoconfigure.web.ServerProperties"
    },
    {
        "name": "server.servlet-path",
        "type": "java.lang.String",
        "sourceType": "org.springframework.boot.autoconfigure.web.ServerProperties"
        "defaultValue": "/"
    }
    ...
]} 
```

每个"property"是一个配置节点，用户可以使用特定的值指定它。例如，`server.port`和`server.servlet-path`可能在`application.properties`中如以下定义：

```java
server.port=9090
server.servlet-path=/home 
```

"groups"是高级别的节点，它们本身不指定一个值，但为 properties 提供一个有上下文关联的分组。例如，`server.port`和`server.servlet-path`属性是`server`组的一部分。

**注**：不需要每个"property"都有一个"group"，一些属性可以以自己的形式存在。

# 附录 B.1.1\. Group 属性

### 附录 B.1.1\. Group 属性

`groups`数组包含的 JSON 对象可以由以下属性组成：

| 名称 | 类型 | 目的 |
| --- | --- | --- |
| name | String | group 的全名，该属性是强制性的 |
| type | String | group 数据类型的类名。例如，如果 group 是基于一个被`@ConfigurationProperties`注解的类，该属性将包含该类的全限定名。如果基于一个`@Bean`方法，它将是该方法的返回类型。如果该类型未知，则该属性将被忽略 |
| description | String | 一个简短的 group 描述，用于展示给用户。如果没有可用描述，该属性将被忽略。推荐使用一个简短的段落描述，第一行提供一个简洁的总结，最后一行以句号结尾 |
| sourceType | String | 贡献该组的来源类名。例如，如果组基于一个被`@ConfigurationProperties`注解的`@Bean`方法，该属性将包含`@Configuration`类的全限定名，该类包含此方法。如果来源类型未知，则该属性将被忽略 |
| sourceMethod | String | 贡献该组的方法的全名（包含括号及参数类型）。例如，被`@ConfigurationProperties`注解的`@Bean`方法名。如果源方法未知，该属性将被忽略 |

# 附录 B.1.2\. Property 属性

### 附录 B.1.2\. Property 属性

`properties`数组中包含的 JSON 对象可由以下属性构成：

| 名称 | 类型 | 目的 |
| --- | --- | --- |
| name | String | property 的全名，格式为小写虚线分割的形式（比如`server.servlet-path`）。该属性是强制性的 |
| type | String | property 数据类型的类名。例如`java.lang.String`。该属性可以用来指导用户他们可以输入值的类型。为了保持一致，原生类型使用它们的包装类代替，比如`boolean`变成了`java.lang.Boolean`。注意，这个类可能是个从一个字符串转换而来的复杂类型。如果类型未知则该属性会被忽略 |
| description | String | 一个简短的组的描述，用于展示给用户。如果没有描述可用则该属性会被忽略。推荐使用一个简短的段落描述，开头提供一个简洁的总结，最后一行以句号结束 |
| sourceType | String | 贡献 property 的来源类名。例如，如果 property 来自一个被`@ConfigurationProperties`注解的类，该属性将包括该类的全限定名。如果来源类型未知则该属性会被忽略 |
| defaultValue | Object | 当 property 没有定义时使用的默认值。如果 property 类型是个数组则该属性也可以是个数组。如果默认值未知则该属性会被忽略 |
| deprecated | boolean | 指定该 property 是否过期。如果该字段没有过期或该信息未知则该属性会被忽略 |

# 附录 B.1.3\. 可重复的元数据节点

### 附录 B.1.3\. 可重复的元数据节点

在同一个元数据文件中出现多次相同名称的"property"和"group"对象是可以接受的。例如，Spring Boot 将`spring.datasource`属性绑定到 Hikari，Tomcat 和 DBCP 类，并且每个都潜在的提供了重复的属性名。这些元数据的消费者需要确保他们支持这样的场景。

# 附录 B.2\. 使用注解处理器产生自己的元数据

### 附录 B.2\. 使用注解处理器产生自己的元数据

通过使用`spring-boot-configuration-processor` jar， 你可以从被`@ConfigurationProperties`注解的节点轻松的产生自己的配置元数据文件。该 jar 包含一个在你的项目编译时会被调用的 Java 注解处理器。想要使用该处理器，你只需简单添加`spring-boot-configuration-processor`依赖，例如使用 Maven 你需要添加：

```java
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-configuration-processor</artifactId>
    <optional>true</optional>
</dependency> 
```

使用 Gradle 时，你可以使用[propdeps-plugin](https://github.com/spring-projects/gradle-plugins/tree/master/propdeps-plugin)并指定：

```java
dependencies {
        optional "org.springframework.boot:spring-boot-configuration-processor"
    }

    compileJava.dependsOn(processResources)
} 
```

**注**：你需要将`compileJava.dependsOn(processResources)`添加到构建中，以确保资源在代码编译之前处理。如果没有该指令，任何`additional-spring-configuration-metadata.json`文件都不会被处理。

该处理器会处理被`@ConfigurationProperties`注解的类和方法，description 属性用于产生配置类字段值的 Javadoc 说明。

**注**：你应该使用简单的文本来设置`@ConfigurationProperties`字段的 Javadoc，因为在没有被添加到 JSON 之前它们是不被处理的。

属性是通过判断是否存在标准的 getters 和 setters 来发现的，对于集合类型有特殊处理（即使只出现一个 getter）。该注解处理器也支持使用 lombok 的`@Data`, `@Getter`和`@Setter`注解。

# 附录 B.2.1\. 内嵌属性

### 附录 B.2.1\. 内嵌属性

该注解处理器自动将内部类当做内嵌属性处理。例如，下面的类：

```java
@ConfigurationProperties(prefix="server")
public class ServerProperties {

    private String name;

    private Host host;

    // ... getter and setters

    private static class Host {

        private String ip;

        private int port;

        // ... getter and setters

    }

} 
```

# 附录 B.2.2\. 添加其他的元数据

### 附录 B.2.2\. 添加其他的元数据

# 附录 C. 自动配置类

### 附录 C. 自动配置类

这里有一个 Spring Boot 提供的所有自动配置类的文档链接和源码列表。也要记着看一下你的应用都开启了哪些自动配置（使用`--debug`或`-Debug`启动应用，或在一个 Actuator 应用中使用`autoconfig`端点）。

# 附录 C.1\. 来自 spring-boot-autoconfigure 模块

### 附录 C.1 来自`spring-boot-autoconfigure`模块

下面的自动配置类来自`spring-boot-autoconfigure`模块：

| 配置类 | 链接 |
| --- | --- |
| [ActiveMQAutoConfiguration](http://github.com/spring-projects/spring-boot/tree/master/spring-boot-autoconfigure/src/main/java/org/springframework/boot/autoconfigure/jms/activemq/ActiveMQAutoConfiguration.java) | [javadoc](http://docs.spring.io/spring-boot/docs/1.3.0.BUILD-SNAPSHOT/api/org/springframework/boot/autoconfigure/jms/activemq/ActiveMQAutoConfiguration.html) |
| [AopAutoConfiguration](http://github.com/spring-projects/spring-boot/tree/master/spring-boot-autoconfigure/src/main/java/org/springframework/boot/autoconfigure/aop/AopAutoConfiguration.java) | [javadoc](http://docs.spring.io/spring-boot/docs/1.3.0.BUILD-SNAPSHOT/api/org/springframework/boot/autoconfigure/aop/AopAutoConfiguration.html) |
| [BatchAutoConfiguration](http://github.com/spring-projects/spring-boot/tree/master/spring-boot-autoconfigure/src/main/java/org/springframework/boot/autoconfigure/batch/BatchAutoConfiguration.java) | [javadoc](http://docs.spring.io/spring-boot/docs/1.3.0.BUILD-SNAPSHOT/api/org/springframework/boot/autoconfigure/batch/BatchAutoConfiguration.html) |
| [CacheAutoConfiguration](http://github.com/spring-projects/spring-boot/tree/master/spring-boot-autoconfigure/src/main/java/org/springframework/boot/autoconfigure/cache/CacheAutoConfiguration.java) | [javadoc](http://docs.spring.io/spring-boot/docs/1.3.0.BUILD-SNAPSHOT/api/org/springframework/boot/autoconfigure/cache/CacheAutoConfiguration.html) |
| [CloudAutoConfiguration](http://github.com/spring-projects/spring-boot/tree/master/spring-boot-autoconfigure/src/main/java/org/springframework/boot/autoconfigure/cloud/CloudAutoConfiguration.java) | [javadoc](http://docs.spring.io/spring-boot/docs/1.3.0.BUILD-SNAPSHOT/api/org/springframework/boot/autoconfigure/cloud/CloudAutoConfiguration.html) |
| [DataSourceAutoConfiguration](http://github.com/spring-projects/spring-boot/tree/master/spring-boot-autoconfigure/src/main/java/org/springframework/boot/autoconfigure/jdbc/DataSourceAutoConfiguration.java) | [javadoc](http://docs.spring.io/spring-boot/docs/1.3.0.BUILD-SNAPSHOT/api/org/springframework/boot/autoconfigure/jdbc/DataSourceAutoConfiguration.html) |
| [DataSourceTransactionManagerAutoConfiguration](http://github.com/spring-projects/spring-boot/tree/master/spring-boot-autoconfigure/src/main/java/org/springframework/boot/autoconfigure/jdbc/DataSourceTransactionManagerAutoConfiguration.java) | [javadoc](http://docs.spring.io/spring-boot/docs/1.3.0.BUILD-SNAPSHOT/api/org/springframework/boot/autoconfigure/jdbc/DataSourceTransactionManagerAutoConfiguration.html) |
| [DeviceDelegatingViewResolverAutoConfiguration](http://github.com/spring-projects/spring-boot/tree/master/spring-boot-autoconfigure/src/main/java/org/springframework/boot/autoconfigure/mobile/DeviceDelegatingViewResolverAutoConfiguration.java) | [javadoc](http://docs.spring.io/spring-boot/docs/1.3.0.BUILD-SNAPSHOT/api/org/springframework/boot/autoconfigure/mobile/DeviceDelegatingViewResolverAutoConfiguration.html) |
| [DeviceResolverAutoConfiguration](http://github.com/spring-projects/spring-boot/tree/master/spring-boot-autoconfigure/src/main/java/org/springframework/boot/autoconfigure/mobile/DeviceResolverAutoConfiguration.java) | [javadoc](http://docs.spring.io/spring-boot/docs/1.3.0.BUILD-SNAPSHOT/api/org/springframework/boot/autoconfigure/mobile/DeviceResolverAutoConfiguration.html) |
| [DispatcherServletAutoConfiguration](http://github.com/spring-projects/spring-boot/tree/master/spring-boot-autoconfigure/src/main/java/org/springframework/boot/autoconfigure/web/DispatcherServletAutoConfiguration.java) | [javadoc](http://docs.spring.io/spring-boot/docs/1.3.0.BUILD-SNAPSHOT/api/org/springframework/boot/autoconfigure/web/DispatcherServletAutoConfiguration.html) |

# 附录 C.2\. 来自 spring-boot-actuator 模块

### 附录 C.2 来自`spring-boot-actuator`模块

下列的自动配置类来自于`spring-boot-actuator`模块：

# 附录 D. 可执行 jar 格式

### 附录 D. 可执行 jar 格式

`spring-boot-loader`模块允许 Spring Boot 对可执行 jar 和 war 文件的支持。如果你正在使用 Maven 或 Gradle 插件，可执行 jar 会被自动产生，通常你不需要了解它是如何工作的。

如果你需要从一个不同的构建系统创建可执行 jars，或你只是对底层技术好奇，本章节将提供一些背景资料。

# 附录 D.1\. 内嵌 JARs

### 附录 D.1\. 内嵌 JARs

Java 没有提供任何标准的方式来加载内嵌的 jar 文件（也就是 jar 文件自身包含到一个 jar 中）。如果你正分发一个在不解压缩的情况下可以从命令行运行的自包含应用，那这将是个问题。

为了解决这个问题，很多开发者使用"影子" jars。一个影子 jar 只是简单的将所有 jars 的类打包进一个单独的"超级 jar"。使用影子 jars 的问题是它很难分辨在你的应用中实际可以使用的库。在多个 jars 中存在相同的文件名（内容不同）也是一个问题。Spring Boot 另劈稀径，让你能够直接嵌套 jars。

# 附录 D.1.1\. 可执行 jar 文件结构

### 附录 D.1.1 可执行 jar 文件结构

Spring Boot Loader 兼容的 jar 文件应该遵循以下结构：

```java
example.jar
 |
 +-META-INF
 |  +-MANIFEST.MF
 +-org
 |  +-springframework
 |     +-boot
 |        +-loader
 |           +-<spring boot loader classes>
 +-com
 |  +-mycompany
 |     + project
 |        +-YouClasses.class
 +-lib
    +-dependency1.jar
    +-dependency2.jar 
```

依赖需要放到内部的 lib 目录下。

# 附录 D.1.2\. 可执行 war 文件结构

### 附录 D.1.2\. 可执行 war 文件结构

Spring Boot Loader 兼容的 war 文件应该遵循以下结构：

```java
example.jar
 |
 +-META-INF
 |  +-MANIFEST.MF
 +-org
 |  +-springframework
 |     +-boot
 |        +-loader
 |           +-<spring boot loader classes>
 +-WEB-INF
    +-classes
    |  +-com
    |     +-mycompany
    |        +-project
    |           +-YouClasses.class
    +-lib
    |  +-dependency1.jar
    |  +-dependency2.jar
    +-lib-provided
       +-servlet-api.jar
       +-dependency3.jar 
```

依赖需要放到内嵌的`WEB-INF/lib`目录下。任何运行时需要但部署到普通 web 容器不需要的依赖应该放到`WEB-INF/lib-provided`目录下。

# 附录 D.2\. Spring Boot 的"JarFile"类

### 附录 D.2\. Spring Boot 的"JarFile"类

Spring Boot 用于支持加载内嵌 jars 的核心类是`org.springframework.boot.loader.jar.JarFile`。它允许你从一个标准的 jar 文件或内嵌的子 jar 数据中加载 jar 内容。当首次加载的时候，每个 JarEntry 的位置被映射到一个偏移于外部 jar 的物理文件：

```java
myapp.jar
+---------+---------------------+
|         | /lib/mylib.jar      |
| A.class |+---------+---------+|
|         || B.class | B.class ||
|         |+---------+---------+|
+---------+---------------------+
^          ^          ^
0063       3452       3980 
```

上面的示例展示了如何在 myapp.jar 的 0063 处找到 A.class。来自于内嵌 jar 的 B.class 实际可以在 myapp.jar 的 3452 处找到，B.class 可以在 3980 处找到（图有问题？）。

有了这些信息，我们就可以通过简单的寻找外部 jar 的合适部分来加载指定的内嵌实体。我们不需要解压存档，也不需要将所有实体读取到内存中。

# 附录 D.2.1\. 对标准 Java "JarFile"的兼容性

### 附录 D.2.1 对标准 Java "JarFile"的兼容性

Spring Boot Loader 努力保持对已有代码和库的兼容。`org.springframework.boot.loader.jar.JarFile`继承自`java.util.jar.JarFile`，可以作为降级替换。

# 附录 D.3\. 启动可执行 jars

### 附录 D.3\. 启动可执行 jars

`org.springframework.boot.loader.Launcher`类是个特殊的启动类，用于一个可执行 jars 的主要入口。它实际上就是你 jar 文件的`Main-Class`，并用来设置一个合适的`URLClassLoader`，最后调用你的`main()`方法。

这里有３个启动器子类，JarLauncher，WarLauncher 和 PropertiesLauncher。它们的作用是从嵌套的 jar 或 war 文件目录中（相对于显示的从 classpath）加载资源（.class 文件等）。在`[Jar|War]Launcher`情况下，嵌套路径是固定的（`lib/*.jar`和 war 的`lib-provided/*.jar`），所以如果你需要很多其他 jars 只需添加到那些位置即可。PropertiesLauncher 默认查找你应用存档的`lib/`目录，但你可以通过设置环境变量`LOADER_PATH`或 application.properties 中的`loader.path`来添加其他的位置（逗号分割的目录或存档列表）。

# 附录 D.3.1 Launcher manifest

### 附录 D.3.1 Launcher manifest

你需要指定一个合适的 Launcher 作为`META-INF/MANIFEST.MF`的`Main-Class`属性。你实际想要启动的类（也就是你编写的包含 main 方法的类）需要在`Start-Class`属性中定义。

例如，这里有个典型的可执行 jar 文件的 MANIFEST.MF：

```java
Main-Class: org.springframework.boot.loader.JarLauncher
Start-Class: com.mycompany.project.MyApplication 
```

对于一个 war 文件，它可能是这样的：

```java
Main-Class: org.springframework.boot.loader.WarLauncher
Start-Class: com.mycompany.project.MyApplication 
```

**注**：你不需要在 manifest 文件中指定`Class-Path`实体，classpath 会从嵌套的 jars 中被推导出来。

# 附录 D.3.2\. 暴露的存档

### 附录 D.3.2\. 暴露的存档

一些 PaaS 实现可能选择在运行前先解压存档。例如，Cloud Foundry 就是这样操作的。你可以运行一个解压的存档，只需简单的启动合适的启动器：

```java
$ unzip -q myapp.jar
$ java org.springframework.boot.loader.JarLaunche 
```

# 附录 D.4\. PropertiesLauncher 特性

### 附录 D.4\. PropertiesLauncher 特性

PropertiesLauncher 有一些特殊的性质，它们可以通过外部属性来启用（系统属性，环境变量，manifest 实体或 application.properties）。

| Key | 作用 |
| --- | --- |
| loader.path | 逗号分割的 classpath，比如`lib:${HOME}/app/lib` |
| loader.home | 其他属性文件的位置，比如/opt/app（默认为`${user.dir}`） |
| loader.args | main 方法的默认参数（以空格分割） |
| loader.main | 要启动的 main 类名称，比如`com.app.Application` |
| loader.config.name | 属性文件名，比如 loader（默认为 application） |
| loader.config.location | 属性文件路径，比如`classpath:loader.properties`（默认为 application.properties） |
| loader.system | 布尔标识，表明所有的属性都应该添加到系统属性中（默认为 false） |

Manifest 实体 keys 通过大写单词首字母及将分隔符从"."改为"-"（比如`Loader-Path`）来进行格式化。`loader.main`是个特例，它是通过查找 manifest 的`Start-Class`，这样也兼容 JarLauncher。

环境变量可以大写字母并且用下划线代替句号。

*   `loader.home`是其他属性文件（覆盖默认）的目录位置，只要没有指定`loader.config.location`。
*   `loader.path`可以包含目录（递归地扫描 jar 和 zip 文件），存档路径或通配符模式（针对默认的 JVM 行为）。
*   占位符在使用前会被系统和环境变量加上属性文件本身的值替换掉。

# 附录 D.5\. 可执行 jar 的限制

### 附录 D.5\. 可执行 jar 的限制

当使用 Spring Boot Loader 打包的应用时有一些你需要考虑的限制。

# 附录 D.5.1\. Zip 实体压缩

### 附录 D.5.1 Zip 实体压缩

对于一个嵌套 jar 的 ZipEntry 必须使用`ZipEntry.STORED`方法保存。这是需要的，这样我们可以直接查找嵌套 jar 中的个别内容。嵌套 jar 的内容本身可以仍旧被压缩，正如外部 jar 的其他任何实体。

# 附录 D.5.2\. 系统 ClassLoader

### 附录 D.5.2\. 系统 ClassLoader

启动的应用在加载类时应该使用`Thread.getContextClassLoader()`（多数库和框架都默认这样做）。尝试通过`ClassLoader.getSystemClassLoader()`加载嵌套的类将失败。请注意`java.util.Logging`总是使用系统类加载器，由于这个原因你需要考虑一个不同的日志实现。

# 附录 D.6\. 可替代的单一 jar 解决方案

### 附录 D.6\. 可替代的单一 jar 解决方案

如果以上限制造成你不能使用 Spring Boot Loader，那可以考虑以下的替代方案：

*   [Maven Shade Plugin](http://maven.apache.org/plugins/maven-shade-plugin/)
*   [JarClassLoader](http://www.jdotsoft.com/JarClassLoader.php)
*   [OneJar](http://one-jar.sourceforge.net/)

# 附录 E. 依赖版本

### 附录 E. 依赖版本

下面的表格提供了详尽的依赖版本信息，这些版本由 Spring Boot 自身的 CLI，Maven 的依赖管理（dependency management）和 Gradle 插件提供。当你声明一个对以下 artifacts 的依赖而没有声明版本时，将使用下面表格中列出的版本。