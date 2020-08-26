# mybatis内部运行步骤

转自[博客](https://blog.csdn.net/qq_21150865/article/details/84305338?utm_medium=distribute.pc_relevant.none-task-blog-BlogCommendFromMachineLearnPai2-1.channel_param&depth_1-utm_source=distribute.pc_relevant.none-task-blog-BlogCommendFromMachineLearnPai2-1.channel_param)

- 通过Reader对象读取Mybatis映射文件
- 通过SqlSessionFactoryBuilder对象创建SqlSessionFactory对象
- 获取当前线程的SQLSession
- 事务默认开启
- 通过SQLSession读取映射文件中的操作编号，从而读取SQL语句
- 提交事务
- 关闭资源

1. SqlSessionFactory的创建:回顾`SqlSessionFactoryUtil`的`initSqlSessionFactory`方法，首先用`InputStream`读取`mybatis-config.xml`配置文件，然后通过`SqlSessionFactoryBuilder`的`build`方法构造。

   ![SqlSessionFactory](https://img-blog.csdnimg.cn/20181120151409990.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzIxMTUwODY1,size_16,color_FFFFFF,t_70)

   进入`build`方法,可以看到通过`XMLConfigBuilder`解析inputStream中的配置参数，`parser.parse()`将配置数据存入Configuration类中。

   ```java
   public SqlSessionFactory build(InputStream inputStream, String environment, Properties properties) {
           SqlSessionFactory var5;
           try {
               XMLConfigBuilder parser = new XMLConfigBuilder(inputStream, environment, properties);
               var5 = this.build(parser.parse());
           } catch (Exception var14) {
               throw ExceptionFactory.wrapException("Error building SqlSession.", var14);
           } finally {
               ErrorContext.instance().reset();
   
               try {
                   inputStream.close();
               } catch (IOException var13) {
               }
   
           }
   
           return var5;
       }
   ```

   进入`parse()`方法中查看，`parseConfiguration（XNode root）`方法中包含了许多其他方法，每一个方法的作用都是将特定节点的数据存入到`configuration`对象中。`Configuration`是一个很关键的类，mybatis中所有配置信息基本都存在于此。

   ```java
   public Configuration parse() {
           if (this.parsed) {
               throw new BuilderException("Each XMLConfigBuilder can only be used once.");
           } else {
               this.parsed = true;
               this.parseConfiguration(this.parser.evalNode("/configuration"));
               return this.configuration;
           }
       }
   
       private void parseConfiguration(XNode root) {
           try {
               this.propertiesElement(root.evalNode("properties"));
               Properties settings = this.settingsAsProperties(root.evalNode("settings"));
               this.loadCustomVfs(settings);
               this.typeAliasesElement(root.evalNode("typeAliases"));
               this.pluginElement(root.evalNode("plugins"));
               this.objectFactoryElement(root.evalNode("objectFactory"));
               this.objectWrapperFactoryElement(root.evalNode("objectWrapperFactory"));
               this.reflectorFactoryElement(root.evalNode("reflectorFactory"));
               this.settingsElement(settings);
               this.environmentsElement(root.evalNode("environments"));
               this.databaseIdProviderElement(root.evalNode("databaseIdProvider"));
               this.typeHandlerElement(root.evalNode("typeHandlers"));
               this.mapperElement(root.evalNode("mappers"));
           } catch (Exception var3) {
               throw new BuilderException("Error parsing SQL Mapper Configuration. Cause: " + var3, var3);
           }
       }
   ```

   最后再调用重载的build（Configuration config）方法，创建一个DefaultSqlSessionFactory，此类是SqlSessionFactory的一个实现类。

   ```java
   public SqlSessionFactory build(Configuration config) {
       return new DefaultSqlSessionFactory(config);
   }
   ```

   

2. 成功创建完SqlSessionFactory后，接下来就是获取sqlSession了，`SqlSession`也是一个非常重要的类，通过它去和数据库打交道。我们来看如何获取sqlSession，进入`sqlSessionFactory.openSession()`方法。最终会来到如下方法。

   ```java
   private SqlSession openSessionFromDataSource(ExecutorType execType, TransactionIsolationLevel level, boolean autoCommit) {
           Transaction tx = null;
   
           DefaultSqlSession var8;
           try {
               Environment environment = this.configuration.getEnvironment();
               TransactionFactory transactionFactory = this.getTransactionFactoryFromEnvironment(environment);
               tx = transactionFactory.newTransaction(environment.getDataSource(), level, autoCommit);
               Executor executor = this.configuration.newExecutor(tx, execType);
               var8 = new DefaultSqlSession(this.configuration, executor, autoCommit);
           } catch (Exception var12) {
               this.closeTransaction(tx);
               throw ExceptionFactory.wrapException("Error opening session.  Cause: " + var12, var12);
           } finally {
               ErrorContext.instance().reset();
           }
   
           return var8;
       }
   ```

   可以看到，通过取出configuration对象中的数据源，创建一个执行器`Executor`，最后通过`new DefaultSqlSession(configuration, executor, autoCommit);`拿到SqlSession的实现。Executor执行器也是一个非常重要的接口，它真正地实现了与数据库的交互。
   到此为止，获取到sqlSession对象后，说明已经与数据库真正建立了连接关系，接下来就是实际的交互，发送sql了。
   在获取数据之前，我们再看一下源码中对SqlSession类的描述：

   The primary Java interface for working with MyBatis. 

   Through this interface you can execute commands, get mappers and manage transactions. 

   SqlSession类是mybatis运行的最核心的java接口，通过这个接口可以执行命令、获取Mapper类、管理事务。

3. 通过sqlSession拿到userMapper对象，接下来在执行`userMapper.getUserById("1")`这一行打断点。

   ![在这里插入图片描述](https://img-blog.csdnimg.cn/20181120160713350.png)

   进入方法内部，看到此处便使用了动态代理技术，这也是为什么mybatis只用接口便可以完成对数据库的操作。继续执行，程序来到`mapperMethod.execute(sqlSession, args);`

   ![img](https://img-blog.csdnimg.cn/20181120161010117.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzIxMTUwODY1,size_16,color_FFFFFF,t_70)

   进入execute方法中，看到此处使用了一个switch语句来判断当前执行的sql是哪种执行类型，我们这里是select，程序继续执行，来到`result = sqlSession.selectOne(command.getName(), param);`这一行，可以看到此处调用了sqlSession的selectOne方法，并且传入了方法名全路径和参数。

   ![在这里插入图片描述](https://img-blog.csdnimg.cn/2018112016155899.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzIxMTUwODY1,size_16,color_FFFFFF,t_70)

   进入selectOne方法内部，下图断点标识的一行便是去数据库中查询，然后返回数据。接下来看看是如何进行查询的。

   ![在这里插入图片描述](https://img-blog.csdnimg.cn/2018112016195974.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzIxMTUwODY1,size_16,color_FFFFFF,t_70)

   进入selectList（statement，parameter）方法内部，此时会调用一个重载方法，继续进入，最终会来到以下代码片段。可以看到最终还是通过executor来执行查询的。

   ![在这里插入图片描述](https://img-blog.csdnimg.cn/20181120162434928.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzIxMTUwODY1,size_16,color_FFFFFF,t_70)

   接下来继续进入query方法，看一下boundSql里面是什么，里面包含了三个重要的属性，sql、parameterMappings、parameterObject。

   ![在这里插入图片描述](https://img-blog.csdnimg.cn/20181120162918784.png)

   ![在这里插入图片描述](https://img-blog.csdnimg.cn/20181120163201961.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzIxMTUwODY1,size_16,color_FFFFFF,t_70)

   继续进入query方法，可以看到此处会去查一遍缓存，如果命中缓存则直接返回，否则继续查询。

   ![在这里插入图片描述](https://img-blog.csdnimg.cn/20181120163551151.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzIxMTUwODY1,size_16,color_FFFFFF,t_70)