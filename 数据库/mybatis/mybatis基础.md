# mybatis需要配置哪些

1. 数据源dataSource的bean
   - 驱动类driver
   - 访问url
   - 用户名
   - 密码

2. SQLSession工厂bean，org.mybatis.spring.SqlSessionFactoryBean，需指定数据源bean
3. 扫描器bean，org.mybatis.spring.mapper.MapperScannerConfigurer，其中配置的是扫描哪个包下的dao接口
4. mybatis包，spring-mybatis包，jdbc包

# mybatis的作用

支持SQL查询，存储过程和高级映射的优秀持久层框架，消除了几乎所有的JDBC代码和参数的手工设置以及结果集的索引。使用简单的XML或者注解用于配置和原始映射，将java中的对象映射成数据库中的记录。