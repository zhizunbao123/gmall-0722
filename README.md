# gmall-0722
谷粒商城电商平台



day01：
	nacos：注册及配置中心 eureka
		nacos服务：阿里官方提供了nacos-server(bat  sh)
		注册中心：
			1. 引入依赖：discovery-starter
			2. application.yml：spring.cloud.nacos.discovery.server-addr=地址
			3. 注解@EnableDiscoveryClient
			
		配置中心
			1. 引入依赖：config-starter
			2. bootstrap.yml:
				spring.cloud.nacos.config.server-addr=地址
				spring.cloud.nacos.config.namespace=唯一标志uuid
				spring.cloud.nacos.config.group=组名
				spring.cloud.nacos.ext-config[0].data-id=配置名
				spring.cloud.nacos.ext-config[0].group=组名
				spring.cloud.nacos.ext-config[0].refresh=true
			3. 注解：@RefreshScope
			好处：
				1. 动态刷新配置，即使改动了配置，也不需要重启
				2. 统一管理配置文件
				3. 配置版本管理

	gateway：网关组件 zuul
		动态路由、负载均衡、身份认证、限流、路径重写、熔断降级、请求过滤
		spring.cloud.gateway.routes[0]
			id: 唯一标志
			uri：路由的路径
			predicates：断言（判断）
			filters：过滤器（拦截）
				重写请求路径的过滤器：RewritePath
				自定义了过滤器：Auth
		自定义过滤器：
			1. 编写class实现GatewayFilter接口（推荐继承AbstractGatewayFilter）
			2. 编写xxxGatewayFilterFactory类实现GatewayFilterFactory（xxx就是过滤器的名称）
			3. 通过xxx使用
				
day02：
	搭建环境
		逆向工程生成单表操作的增删改查
		
	mybatis-plus：在mybatis的基础上制作增强不做改变
		单表的增删改查，不需要自己实现
		1. 引入依赖：参照官网
		2. 配置
			mybatis-plus.mapper-locations=classpath:mappers/pms/**/*.xml
			mybatis-plus.global-config.db-config.id-type=auth(数据库自增，默认分布式id生成策略，input：指定id)
		3. 注解：@MapperScan(dao/mapper包路径)
			@TableName("表名") ：当实体类和表名不一致的情况下
			@TableId(type= IdType.INPUT)：指定表的主键
			@TableField("列名")：当字段名和列名不一致的情况下
		4. 分页插件：
			@Configuration
			public class MybatisPlusConfig {

				@Bean
				public PaginationInterceptor paginationInterceptor() {
					PaginationInterceptor paginationInterceptor = new PaginationInterceptor();
					return paginationInterceptor;
				}
			}
		5. Wrapper：QueryWrapper   UpdateWrapper
	
	cors解决跨域
		浏览器同源策略
		什么情况下会出现跨域？ 
			1. ajax请求
			2. 域名、端口、协议不一致
		解决方案：
			1. nginx反向代理为不跨域
			2. jsonp方式，只能解决get
			3. cors浏览器规范
				发送两次请求：预检请求（允许跨域的请求方法 允许那些域名跨域 是否允许携带cookie 允许携带的头信息） 真正的请求
		过滤器统一解决：
			springMVC：CorsFilter
			springWebFlux: CorsWebFilter
			
	阿里OSS: 浏览直传图片到阿里云服务器
		
day03：
	spu：标准商品单元，商品集合
	sku：库存量单元，具体商品
	规格参数
	表关系
	保存商品（pms sms wms）

day04：
	本地事务：
		@Trancactional
		事务：逻辑上的一组操作，组成这组操作的各个逻辑单元，要么都成功，要么都失败。
		ACID：原子性 一致性 隔离性 持久性
		
		隔离级别：
			read uncommitted：读未提交，脏读（不允许发生） 
			read committed：读已提交，不可重复读（可允许） oracle
			repeatable read：可重复读，幻读/虚读（可允许） mysql
			serializable：序列化读，性能最低
			
		传播行为：7中传播行为，
			REQUIRED：一个事务，要么成功，要么失败
			REQUIRES_NEW：两个不同事务，彼此之间没有关系。一个事务失败了不影响另一个事务
			
		回滚策略
			默认的回滚策略：编译时（受检异常）异常不回滚，运行时异常（不受检异常）都会回滚
			rollBackFor
			rollBackForClassName
			noRollBackFor
			noRollBackForClassName
			
		只读事务：不能做增删改操作
			readOnly=true
			
		超时事务：timeOut=3
	
	分布式事务：网络、服务器宕机、消息丢失
		场景：
			1. 不同的服务不同数据库
			2. 不同的服务相同数据库
			3. 相同的服务不同的数据库
		解决方案：
			1. 两阶段提交（XA：数据库支持  seata， 性能较低）
			2. TCC补偿机制（T：try  C：confirm  Cancel：取消）
			3. 消息队列最终一致性（性能最高）
		seata：
			seata-server服务
			每一个数据库都要有一个undo_log表
			1. 引入依赖：参照官方文档
			2. 配置：
				1. registry.conf
				2. file.conf
				3. java配置：配置了数据源代理
					@Primary
					@Bean("dataSource")
					public DataSource dataSource(@Value("${spring.datasource.url}") String jdbcUrl, @Value("${spring.datasource.username}") String username,
												 @Value("${spring.datasource.password}") String password, @Value("${spring.datasource.driver-class-name}") String driverClassName) {
						HikariDataSource hikariDataSource = new HikariDataSource();
						hikariDataSource.setJdbcUrl(jdbcUrl);
						hikariDataSource.setDriverClassName(driverClassName);
						hikariDataSource.setUsername(username);
						hikariDataSource.setPassword(password);
						return new DataSourceProxy(hikariDataSource);
					}
			3. 注解：@GlobalTransactional @Transactional
	
day05
	倒排索引：文档列表、倒排索引区
	全文检索：从海量数据中快速获取需要的信息
	lucene：底层api
	搜索产品：solr elasticsearch
	
	elasticsearch：
		安装：jvm.options  elasticsearch.yml
		kibana：
		ik分词器：ik_max_word ik_smart
			扩展词典（niginx配置，添加分词后不用重启） 停用词典
	
	DSL语法：
	java客户端：jest springData-elasticsearch(ElasticsearchRestTemplate ElasticsearchRepository(Repository)  HignLevelRestClient SearchSourceBuilder)
		springData-elasticsearch(ElasticsearchRestTemplate ElasticsearchRepository(Repository)  HignLevelRestClient SearchSourceBuilder)
		spring.elasticsearch.rest.uris=地址
		
	
day06
	数据模型的设计
		Document(indexName = "goods", type = "info", shards = 3, replicas = 2)作用在实体类上
		@Id：主键字段
		@Field(type = FieldType.Keyword/Text/Integer/Boolean/Long/Float/Nested, index = false, analyzer = "ik_max_word")
	
	数据导入功能
	DSL语句
	代码实现搜索功能
	
day07
	rabbitmq及数据同步
	MQ：message queue
	作用：解耦 异步 削峰
	实现：
		AMQP（rabbitmq  协议  五种消息模型  任何语言都可以实现） 
		JMS（activemq  java规范  提供了两种消息模型  必须是java实现）
	安装：5672（java） 15672（浏览器客户端） 25672（集群）
	五种消息模型：
		1.simple（简单模型）
		2.worker（工作模型）
		3.发布订阅之fanout（广播：消息发送之后，所有的队列都可以获取消息）
		4.发布订阅之direct（路由：定向发送消息）
		5.发布订阅之topic（通配符：*一个词  #一个或者多个词）
	ACK：消息确认机制
	能者多劳：channel.basicQos(1)
	持久化：交换机持久化  队列持久化  消息持久化
	springBoot整合rabbitMQ
		1. 引入依赖：amqp-starter
		2. 配置：rabbit链接信息
			spring.rabbitmq.host=地址
			spring.rabbitmq.post
			spring.rabbitmq.virtual-host=虚拟主机
			spring.rabbitmq.username=
			spring.rabbitmq.password=
		3. 注解：
			接受消息：
			@RabbitListener(bindings = @QueueBinding(
				value = @Queue(value = "队列名称", durable = "true"),
				exchange = @Exchange(value = "交换机名称", ignoreDeclareExchange = "true", type = ExchangeTypes.Topic),
				key = {"routingKey"}
			))
			
			发送消息：AmqpTemplate.converteAndSend()
			
day08
	三级分类的查询
	添加缓存：
		标准：1. 写的频率低  2. 读的频率高
	过程：
		1.查询缓存有没有  
		2.缓存中没有查询数据库 
		3.放入缓存
	缓存存在的问题：
		雪崩：给缓存的过期时间添加随机值
		穿透：即使数据库中的数据为null，也缓存
		击穿：分布式锁
	实现分布式锁：
		标准：
			1. 排他
			2. 防止死锁发生，设置有效时间
			3. 防止释放别人的锁
		实现：
			1. 获取锁（原子性）
				String uuid = UUID.randomUUID().toString();
				Boolean lock = this.redisTemplate.opsForValue().setIfAbsent("lock", uuid, 2, TimeUnit.SECONDS);
			2. 释放锁（原子性）
				String script = "if redis.call('get', KEYS[1]) == ARGV[1] then return redis.call('del', KEYS[1]) else return 0 end";
				this.redisTemplate.execute(new DefaultRedisScript<>(script), Arrays.asList("lock"), Arrays.asList(uuid));
			3. 重试（没有锁的请求）
				try {
					Thread.sleep(1000);
					testLock();
				} catch (InterruptedException e) {
					e.printStackTrace();
				}
	redisson：
		1. 引入依赖
		2. java配置
			@Configuration
			public class RedissonConfig {

				@Bean
				public RedissonClient redissonClient(){
					Config config = new Config();
					config.useSingleServer().setAddress("redis://172.16.116.100:6379");
					return Redisson.create(config);
				}
			}
		3. RedissonClient.getLock() lock.lock() lock.unlock()
			semaphore countdownlatch readWriteLock
	
	使用AOP结合分布式锁实现缓存的封装
		1. 自定义注解@GmallCache
		2. 编写切面实现缓存功能
			@Aspect：切面类
			@Around：环绕通知 @Around(@Annotation=注解的全路径)
				四个条件：1.方法的返回值必须是Object 2.方法的参数ProceedingJoinPoint  3.方法必须抛出Throwable异常  4.joinPoint.proceed(joinPoint.getArgs())
				(MethodSignature)joinPoint.getSignature();
		
day09
	商品详情页：大量的远程调用
	优化：页面静态化、缓存、异步编排
	异步编排：CompletableFuture
		线程初始化回顾：
			1.继承thread抽象类
			2.实现Runnable接口
			3.实现Callable接口 + FutureTask
			4.线程池
		异步任务初始化：
			runAsync()：没有返回值
			supplyAsync()：有返回值
		任务完成时方法：
			whenComplete((t,u) -> {正常或异常情况下开启另一个任务})
			whenCompleteAsync：异步
			exceptionally(t -> {异常情况下开启另一个任务})
		处理任务完成结果
			handle(t -> {处理任务完成时的结果})
		中间的串行方法：9个方法
			thenApply()：获取上一个任务的返回结果集，并返回自己的结果集
			thenAccept()：获取上一个任务的结果集，执行自己的任务，没有返回值
			thenRun()：上一个任务结束，执行自己的任务，不获取返回结果集，也没有自己的返回结果集
			thenApplyAsync()
			thenAcceptAsync()
			thenRunAsync()
		两个任务组合，都要完成：9个方法
			thenCombine：组合两个future，获取两个future的返回结果，并返回当前任务的返回值
			thenAcceptBoth：组合两个future，获取两个future任务的返回结果，然后处理任务，没有返回值。
			runAfterBoth：组合两个future，不需要获取future的结果，只需两个future处理完任务后，处理该任务。
		两任务组合，一个完成：9个方法
			applyToEither：两个任务有一个执行完成，获取它的返回值，处理任务并有新的返回值。
			acceptEither：两个任务有一个执行完成，获取它的返回值，处理任务，没有新的返回值。
			runAfterEither：两个任务有一个执行完成，不需要获取future的结果，处理任务，也没有返回值。
		多任务组合：
			allof()：所有任务完成，执行新的任务
			anyof()：任何一个任务完成，执行新的任务
			
day10
	注册功能：
		1. 校验数据是否可用：用户名 手机号 邮箱
		2. 发送短信验证码：生成验证码  发消息 并把短信验证码保存到redis中
		3. 用户注册功能（新增用户）
			1）校验验证码
			2）生成盐
			3）对密码加盐加密
			4）保存用户信息
			5）删除redis中的验证码
		4. 根据用户名和密码查询用户
			1）根据用户名查询用户信息
			2）判断用户是否存在
			3）对用户输入的密码加盐加密
			4）和数据库中的密码比较
	cookie：
		作用域：子可用访问父，父不可操作子。token的域使用一级域名
		作用路径：/
		过期时间：
	单点登录：
		无状态登录（jwt）
		有状态登录（session redis）
		jwt + rsa
		jwt：
			头部信息：token类型，编码方式
			载荷信息：用户具体信息
			签名信息：校验前两部信息是否合法 rsa加密
		加密方式：
			对称加密：base64
			不可逆加密：md5
			非对称加密：rsa（公钥 私钥）
	代码具体实现：参照课堂代码
		
day11
	购物车需求：
		新增购物车
		删除购物车
		查询购物车
		修改数量
		勾选购物车
		比价
	技术选型：
		未登录情况下：
			redis
			mysql
			cookie
			indexDB
			webSQL
			localStorage
			mongodb（NoSQL数据库，硬盘，写比较频繁）
			
		登录情况下：
			mysql + redis
			redis
			mongodb
	数据模型：hash（散列）
	业务流程
		新增：判断是否登录
			未登录：使用userKey放入redis中
			登录：使用userId放入redis中
			判断该商品是否在购物车中
				在：更新数量
				不在：新增一条记录
		查询：判断是否登录
			未登录：根据userKey查询，查询完成直接响应
			登录：先判断是否有未登录的购物车
				有：同步到登录状态的购物车，再查询
				无：直接查询已登录的购物车，直接响应
	比价功能：
		单独保存一份实时价格：{skuId:price}
		查询购物车，实时价格单独查询
		价格同步：使用消息队列
		
	获取用户登录信息的拦截器：LoginInterceptor
		ThreadLocal
		1.实现HandlerInterceptor接口或者继承HandlerInterceptorAdapter抽象类
		2.实现3个方法：
			1.preHandle：前置方法，返回值（true：放行  false：拦截）
				获取用户的登录状态（解析jwt类型的token）
				放入threadLocal
			2.postHandle：后置方法
			3.afterComplete：完成方法，视图渲染完成之后执行
				ThreadLocal.remove()释放线程局部变量，防止内存泄漏
		3.编写一个配置类，实现WebMvcConfigurer接口
			addInterceptors(InterceptorRegistry registry) {
				registry.addInterceptor(loginInterceptor).addPathPatterns("/**");
			}
		
		
day12
	订单
		订单确认页
			数据模型：orderToken防止表单重复 收货地址列表 配送方式  送货清单 积分信息
			IdWorker：雪花算法
		
		提交订单
			提交数据模型：orderToken、收货地址、配送方式、支付方式、送货清单、发票信息、积分信息、总价
			业务流程：
				1. 防重
				2. 验价
				3. 验库存并锁库存
				4. 下单
				5. 删除购物车
				
		定时关单
			1. 定时任务（juc   @Scheduled  @EnableScheduling）
			2. 延时队列（延时队列：设置消息的有效时间/设置死信路由/设置死信rountingKey，死信队列绑定到私信路由，消费者监听死信队列）
		
	支付
		内网穿透：花生壳 哲西云
		阿里沙箱：
		支付成功异步回调
	
	秒杀
		页面静态化、限流、异步、缓存
		页面限流
		nginx限流：漏斗算法 令牌桶算法
		网关限流：限流过滤器
		服务器内部限流：信号量
		用户查询订单时，使用闭锁
		
	
		
		


-------------------
02/07/2020 added
『课程目录』:
├─day01-2019年5月22日
│ │ 02-代码.rar
│ │ 03-笔记.rar
│ │
│ └─01-视频
│ 01 谷粒商城的简介
│ 02 谷粒商城的相关技术
│ 03 谷粒商城的工程浏览
│ 04 开发工具的介绍
│ 05 idea的简单配置
│ 06 idea和git的配置
│ 07 数据结构和设计工具的介绍
│ 08 user测试项目的搭建
│ 09 项目的启动和域名的配置
│ 10 controllerservice的测试代码
│ 11 bean映射类
│ 12 通用mapper的加入
│ 13 根据用户memberId查询用户收获地址信息集合
│ 14 补充一个通用mapper的问题
│

├─day02-2019年5月24日
│ │ 02-代码.rar
│ │ 03-笔记.rar
│ │
│ └─01-视频
│ 01 项目架构的简介
│ 02 项目parent技术框架的介绍
│ 03 项目api包的搭建
│ 04 项目util包的介绍
│ 05 项目common-util包的搭建
│ 06 项目service-util包的搭建
│ 07 项目web-util包的搭建
│ 08 关于dubbo的介绍
│ 09 关于dubbo监控中心的启动
│ 10 关于dubbo监控中心的启动
│ 11 启动zookeeper
│ 12 监控中心和zookeeper的开机自启动
│ 13 dubbo框架整合
│ 14 dubbo服务provider的启动
│ 15 dubbo服务consumer的启动
│
├─day03-2019年5月25日
│ │ 02-代码.rar
│ │ 03-笔记.rar
│ │
│ └─01-视频
│ 01 pms-商品概念的介绍
│ 02 pms-数据结构的划分
│ 03 pms-前后端的介绍
│ 04 pms-什么是前后端分离
│ 05 pms-前端项目的启动
│ 06 pms-分类和属性功能的介绍
│ 07 pms-分类和属性功能的实现
│ 08 pms-跨域问题
│ 09 pms-分类(类目)列表的查询
│ 10 pms-平台属性的查询
│ 11 pms-平台属性的保存
│
├─day04-2019年5月27日
│ │ 02-代码.rar
│ │ 03-笔记.rar
│ │
│ └─01-视频
│ 01 平台属性保存功能
│ 02 平台属性的修改操作
│ 03 平台属性的修改代码demo
│ 04 商品spu和sku的概念
│ 05 商品spu和sku的设计原则
│ 06 商品spu列表功能
│ 07 商品spu-关于图片上传的处理
│ 08 商品spu-商品的销售属性介绍
│ 09 商品spu-商品的保存提交的介绍
│ 10 商品spu-商品的保存的参数
│ 11 商品spu-商品的保存的参数
│ 12 图片存储服务-fastdfs的介绍
│ 13 图片存储服务-fastdfs的安装前的准备
│ 14 图片存储服务-libfastcommon类库安装
│ 15 图片存储服务-FDFS软件的安装和配置
│ 16 图片存储服务-FDFS的tracker的启动脚本的配置
│ 17 图片存储服务-FDFS的storage的启动脚本的配置
│ 18 图片存储服务-FDFS的启动
│ 19 图片存储服务-FDFS的linux上传测试
│
├─day05-2019年5月28日
│ │ 02-代码.rar
│ │ 03-笔记.rar
│ │
│ └─01-视频
│ 01 fastdfs-上传测试
│ 02 fastdfs-插件配置
│ 03 fastdfs-nginx配置
│ 04 fastdfs-关于集群
│ 05 fastdfs-客户端安装
│ 06 fastdfs-客户端整合
│ 07 spu-商品信息的保存代码
│ 08 spu-商品信息的保存测试
│ 09 sku-商品库存单元数据结构
│ 10 sku-商品平台属性和销售属性的区别
│ 11 sku-商品页面平台属性和销售属性列表
│ 12 sku-功能的完成
│
├─day06-2019年5月29日
│ │ 02-代码.rar
│ │ 03-笔记.rar
│ │
│ └─01-视频
│ 01 电商前台系统的介绍
│ 02 电商前台系统分布业务的介绍
│ 03 电商前台系统-item项目的创建和thymeleaf的引入
│ 04 电商前台系统-thymeleaf的扩展标签
│ 05 电商前台系统-thymeleaf的送校验和热部署
│ 06 电商前台系统-thymeleaf的标签
│ 07 电商前台系统-thymeleaf的js表达式处理参数
│ 08 电商前台系统-thymeleaf的页面引入
│ 09 item-静态页面引入
│ 10 item-sku数据展示
│ 11 item-sku根据销售属性动态切换
│ 12 item-sku根据销售属性的功能分析
│ 13 item-sku页面销售属性列表
│ 14 item-sku页面销售属性列表的选中sql处理
│
├─day07-2019年5月31日
│ │ 02-代码.rar
│ │ 03-笔记.rar
│ │
│ └─01-视频
│ 01 商品详情页面销售属性值功能复习
│ 02 商品详情页面销售属性值功能复习
│ 03 商品销售属性切换的hash表的分析
│ 04 商品销售属性切换-hash表的制作
│ 05 商品销售属性切换-hash表页面处理切换请求
│ 06 商品详情性能-页面高并发的优化方法介绍
│ 07 商品详情性能-缓存redis的使用方案
│ 08 商品详情性能-缓存redis的实现思路
│ 09 商品详情性能-缓存redis的整合
│ 10 商品详情性能-缓存redis的整合后测试
│ 11 商品详情性能-通过缓存和db查询商品的详情
│ 12 商品详情性能-通过缓存查询的代码测试
│ 13 商品详情性能-缓存的问题对比
│ 14 商品缓存击穿问题-如何解决
│ 15 商品缓存击穿问题-redis的nx分布式锁
│
├─day08-2019年6月01日
│ │ 02-代码.rar
│ │ 03-笔记.rar
│ │
│ └─01-视频
│ 01 商品详情redis锁-代码完成
│ 02 商品详情redis锁-手动压力测试
│ 03 商品详情redis锁-两种意外情况
│ 04 商品详情redis锁-两种意外情况总结
│ 05 商品详情redisson-redisson测试项目的整合
│ 06 商品详情redisson-防压力测试代码
│ 07 商品详情redisson-nginx负载均衡的配置
│ 08 商品详情redisson-安装和使用apache的ab压力测试工具
│ 09 商品详情redisson-压力测试redisson的分布锁
│ 10 商品搜索-搜索相关技术的介绍
│ 11 商品搜索-搜索相关技术的介绍
│ 12 商品搜索-es的安装和配置
│ 13 商品搜索-es的需要修改的配置文件
│
├─day09-2019年6月03日
│ │ 02-代码.rar
│ │ 03-笔记.rar
│ │
│ └─01-视频
│ 01 es的交互方式
│ 02 es的基本概念
│ 03 es的kibana和基本操作
│ 04 es的简单的增删改查
│ 05 es的查询关键字
│ 06 es的分词
│ 07 es的分词器的使用
│ 08 es的相关性算分
│ 09 es的集群-克隆一台虚拟机
│ 10 es的集群-配置参数说明
│ 11 es的集群-集群的启动
│ 12 es的集群-集群的工作原理
│ 13 es的集群-集群的工作原理实践
│ 14 谷粒搜索-功能模块简介
│ 15 谷粒搜索-功能模块功能分析
│ 16 谷粒搜索-数据结构的定义
│
├─day10-2019年6月04日
│ │ 02-代码.rar
│ │ 03-笔记.rar
│ │
│ └─01-视频
│ 01 搜索功能-es数据结构的搭建
│ 02 搜索功能-search项目的初始化
│ 03 搜索功能-es用的mysql数据转化
│ 04 搜索功能-es用的mysql数据导入
│ 05 搜索功能-es用的复杂查询
│ 06 搜索功能-es用的并集的处理
│ 07 搜索功能-es用jest执行查询语句
│ 08 搜索功能-es用dsl查询字符串工具
│ 09 搜索功能-前端静态资源文件的引入
│ 10 搜索功能-搜索服务的完成
│ 11 搜索功能-高亮
│
├─day11-2019年6月05日
│ │ 02-代码.rar
│ │ 03-笔记.rar
│ │
│ └─01-视频
│ 01 搜索功能-平台属性的聚合查询aggs
│ 02 搜索功能-平台属性的聚合查询方案B
│ 03 搜索功能-平台属性的聚合查询sql
│ 04 搜索功能-平台属性页面列表修改
│ 05 搜索功能-平台属性和面包屑请求的分析
│ 06 搜索功能-平台属性和面包屑请求的拼接
│ 07 搜索功能-平台属性参数的提交
│ 08 搜索功能-排除被选中的平台属性组
│ 09 搜索功能-面包屑怎么做
│ 10 搜索功能-面包屑怎么做
│ 11 搜索功能-面包屑怎么做bug调试
│ 12 搜索功能-谷粒搜索的总结
│ 13 搜索功能-系统测试
│ 14 购物车-提交页面引入
│ 15 购物车-功能介绍
│
├─day12-2019年6月06日
│ │ 02-代码.rar
│ │ 03-笔记.rar
│ │
│ └─01-视频
│ 01 购物车数据结构介绍
│ 02 购物车业务程序介绍
│ 03 购物车业务-cookie添加的实现
│ 04 购物车业务-cookie添加的测试
│ 05 购物车业务-db添加的代码
│ 06 购物车业务-购物车业务代码实现
│ 07 购物车业务-购物车业务代码测试
│ 08 购物车业务-购物车查询
│ 09 购物车业务-购物车异步修改选中状态
│ 10 购物车业务-购物车异步修改选中状态测试
│ 11 购物车业务-价格计算
│
├─day13-2019年6月10日
│ │ 02-代码.rar
│ │ 03-笔记.rar
│ │
│ └─01-视频
│ 01 购物车总价
│ 02 用户的登录认证功能介绍
│ 03 用户认证-认证模块的设计
│ 04 用户认证-认证passport建立和dubbo的校bug
│ 05 用户认证-整合认证passport
│ 06 用户认证-认证中心passport拦截器
│ 07 用户认证-拦截器的状态选择
│ 08 购物车的一个ui小bug
│ 09 用户认证-三种单点登录算法的对比
│ 10 用户认证-jwt的加密算法
│ 11 认证功能开发-主动登录
│ 12 认证功能开发-拦截器拦截token的四种情况处理
│ 13 认证功能开发-拦截器功能开发
│ 14 认证功能开发-登录拦截测试
│
├─day14-2019年6月11日
│ │ 02-代码.rar
│ │ 03-笔记.rar
│ │
│ └─01-视频
│ 01 认证流程的复习
│ 02 认证拦截器的两个小问题
│ 03 用户登录-服务的编写
│ 04 用户登录-服务的测试
│ 05 用户登录-jwt生成token测试
│ 06 用户登录-拦截器访问认证中心校验token
│ 07 用户登录-拦截器error全局异常的小问题
│ 08 用户登录-应用层获得拦截器传递的memberId
│ 09 用户登录-jwt的私人部分可以由base64直接解析
│ 10 用户登录-登录时是否同步购物车数据
│ 11 社交登录-oauth2协议介绍
│ 12 社交登录-四个步骤
│ 13 社交登录-社交登录的准备工作
│ 14 社交登录-社交登录的四个调用地址
│ 15 社交登录-第四步用户信息测试
│ 16 社交登录-四步用户信息测试
│
├─day15-2019年6月12日
│ │ 02-代码.rar
│ │ 03-笔记.rar
│ │
│ └─01-视频
│ 01 社交登录-未审核只能链接一个用户
│ 02 社交登录-四个地址的复习
│ 03 社交登录-地址一的整合
│ 04 社交登录-地址二的整合
│ 05 社交登录-地址三四的整合
│ 06 社交登录-社交用户数据保存
│ 07 社交登录-完成测试
│ 08 结算和订单-业务介绍
│ 09 结算和订单-order项目的创建
│ 10 结算和订单-结算页面
│
├─day16-2019年6月14日
│ │ 02-代码.rar
│ │ 03-笔记.rar
│ │
│ └─01-视频
│ 01 结算页面处理
│ 02 订单提交的业务介绍
│ 03 如何防止订单的重复提交
│ 04 通过交易码保证一个结算只能提交一个订单
│ 05 在并发情况下的交易码问题
│ 06 关于用户的两个bug
│ 07 lua脚本防止并发的情况下的交易码重复使用问题
│ 08 lua脚本防止并发的情况下的交易码重复使用代码测试
│ 09 订单提交-校验价格和库存
│ 10 订单提交-业务字段封装
│ 11 订单提交-业务字段保存测试
│ 12 订单提交-删除购物车的服务
│ 13 支付-支付业务的介绍
│
├─day17-2019年6月15日
│ │ 02-代码.rar
│ │ 03-笔记.rar
│ │
│ └─01-视频
│ 01 关于lua脚本的复习
│ 02 支付-流程图解释
│ 03 支付-支付服务的创建
│ 04 支付-支付服务的创建
│ 05 支付-支付对接步骤
│ 06 支付-将支付宝框架整合到项目中
│ 07 支付-支付宝的alipayClient的引入
│ 08 支付-生成对接表单
│ 09 支付-sdk的安装
│ 10 支付-交易创建的支付过程
│ 11 支付-公钥和私钥的使用过程
│ 12 支付-支付宝回调
│ 13 支付-支付成功
│
├─day18-2019年6月17日
│ │ 02-代码.rar
│ │ 03-笔记.rar
│ │
│ └─01-视频
│ 01 支付流程的复习
│ 02 分布式事务-概念
│ 03 分布式事务-解决方案介绍
│ 04 分布式事务-mq的实现框架的介绍
│ 05 分布式事务-mq的安装和启动
│ 06 分布式事务-mq整合
│ 07 分布式事务-mq的模式
│ 08 分布式事务-队列消息的消费
│ 09 分布式事务-话题消息的消费
│ 10 分布式事务-MQ整合
│ 11 分布式事务-gmall的消息队列的系统设计
│ 12 分布式事务-支付成功消息
│ 13 分布式事务-支付消息测试
│ 14 分布式事务-订单消费支付+测试
│
├─day19-2019年6月18日
│ │ 02-代码.rar
│ │ 03-笔记.rar
│ │
│ └─01-视频
│ 01 分布式事务-内容复习
│ 02 分布式事务-如何完善电商分布式系统
│ 03 延迟队列-提交支付发送延迟队列
│ 04 延迟队列-延迟队列的任务介绍
│ 05 延迟队列-延迟队列的循环逻辑
│ 06 延迟队列-延迟队列的循环逻辑测试
│ 07 延迟队列-延迟队列支付情况接口查询
│ 08 延迟队列-幂等性检查
│ 09 库存介绍-数据结构
│ 10 拆单介绍-库存和电商平台和物流的关系
│ 11 拆单介绍-库存和订单关系
│ 12 库存系统-库存整合和介绍
│ 13 库存系统-库存的测试
│ 14 一些补充知识
│
└─day20-2019年6月19日
│ 02-代码.rar
│ 03-笔记.rar
│
└─01-视频
01 秒杀-概念
02 秒杀-秒杀项目的创建
03 秒杀-秒杀程序
04 秒杀-redis秒杀程序
05 秒杀-redisson秒杀程序
06 限流-限流的概念
07 限流-限流的算法
08 其他技术-云容器的介绍
09 其他技术-金融名词介绍
10 项目介绍-项目是怎么设计出来的
11 项目介绍-关于硬件
12 项目介绍-关于项目流程
