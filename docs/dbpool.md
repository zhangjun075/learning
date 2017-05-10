# 连接池配置
正确配置DBCP防止数据库重启引起的访问错误

在java web 应用中使用dbcp做为连接池，当数据库重启或数据库连接超过设置的最大timemout时间，数据库会强行断开已有的链接，此时当web程序访问数据库时就会出现错误，大致的错误信息java.io.EOFException: Can not read response from server. Expected to read 4 bytes, read 0 bytes before connection was unexpectedly lost，原因是数据库这边已有的连接强行断开，而连接池中不知道已经断开，还是从连接池取出数据库连接交给程序去执行数据库操作，所以导致出错。

mysql的默认最大timeout时间是8小时，对空闲超过8小时的数据库连接会强行断开。timeout有两种，一个是非交互式的最大等待时间wait_timeout，另一个是交互式的最大等待时间interactive_time，交互连接如mysql gui tool中的连接。一般情况下interactive_timeout的设置将要对你的web 应用没有多大的影响。wait_timeout的时间设置太小话会导致连接关闭很快，从而使一些持久连接不起作用，反之设置过大，容易造成连接打开时间过长，在show processlist时，能看到太多的sleep状态的连接，从而造成too many connections错误。修改wait_timeout可以在my.cnf的mysqld段中设置。

可以通过dbcp的配置来解决上述的报错。可以用两种方式。

方式一：通过设置validationQuery，例如:

	<property name="validationQuery">
		<value>select 1</value>
	</property>

使用上述配置，连接池在返回数据库连接给申请者时会多执行一条sql语句来确保该连接的有效性。如果数据库方已经关闭了，连接池会重新建立连接并返回给申请者。通过测试似乎跟testWhileIdle没有关系，不管其是true或false都正常工作。

方式二：通过配置timeBetweenEvictionRunsMillis和minEvictableIdleTimeMillis，例如：

	<property name="minEvictableIdleTimeMillis">
		<value>60000</value>
	</property>
	<property name="timeBetweenEvictionRunsMillis">
		<value>10000</value>
	</property>

在构造GenericObjectPool [BasicDataSource在其createDataSource () 方法中也会使用GenericObjectPool]时，会生成一个内嵌类Evictor，实现自Runnable接口。如果timeBetweenEvictionRunsMillis大于0，每过 timeBetweenEvictionRunsMillis毫秒Evictor会调用evict()方法，检查连接池中的连接的闲置时间是否大于 minEvictableIdleTimeMillis毫秒（_minEvictableIdleTimeMillis小于等于0时则忽略，默认为30分钟），是则销毁此对象，然后调用ensureMinIdle方法检查确保池中对象个数不小于_minIdle。如果连接池的连接数小于最小空闲连接数，则创建数据库连接，同时检查连接池的连接是否小于maxIdle，是则把刚创建的连接放入连接池中，否则销毁此对象。

上述方式一能确保不出现本文开头提到的错误，但是不好的方面是每次执行sql时都会额外执行一条提供的validationQuery sql；第二种方式在数据库重启后minEvictableIdleTimeMillis毫秒前访问web应用，连接数据库使用的还是连接池中老的连接，所以还会出现上述的错误，timeBetweenEvictionRunsMillis和minEvictableIdleTimeMillis也不宜设置过小，会加重系统开销。根据具体情况来考虑使用哪种方式。对于数据库可能会经常重启，web应用和数据库机器的网络连接不稳定，可以采取第一种方式，否则使用第二种。由于mysql的默认最大空闲时间8小时，所以只要把minEvictableIdleTimeMillis设置小于此值即可。例如配置每十分钟检查超过空闲一个小时的连接

	<property name="minEvictableIdleTimeMillis">
		<value>3600000</value>
	</property>
	<property name="timeBetweenEvictionRunsMillis">
		<value>600000</value>
	</property>