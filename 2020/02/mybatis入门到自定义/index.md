# MyBatis入门到自定义


<!--more-->
第一个 MyBatis 程序（XML 配置）
----------------------

在上一篇中，简单总结了一下原生 JDBC 的一些局限性，同时引出了 MyBatis 这个框架，算较为详细的整理如何搭建 MyBatis 的工作环境

这一篇，我们在开篇，现在搭建好工作环境的基础上，开始我们的第一个例程，但是，简单的让程序跑起来以后，我们却要讲解如何自定义 MyBatis 框架，它的意义是什么呢？

虽然第一个例程虽然比较简单，但是其中有很多点却是容易引起疑惑的，整个执行过程也不是很清楚，通过自定义框架，可以让自己对于 MyBatis 的理解更加深刻，从而更好的应用这个框架

首先，我们想让我们的第一个程序运行起来

1、搭建好环境，在主配置文件 (SqlMapConfig.xml) 中指定映射配置文件的位置

```
<!-- 指定映射配置文件的位置 -->
<mappers>
    <mapper resource="cn/ideal/mapper/UserMapper.xml"/>
</mappers>
复制代码
```

2、在 test 文件夹下，创建一个如图结构测试类

![](https://user-gold-cdn.xitu.io/2020/2/2/170050f832ae9187?imageView2/0/w/1280/h/960/format/webp/ignore-error/1)

由于我们的 mapper 接口中写的方法是一个 查询所有信息的方法，所以我们直接如下图所写就行了，这就是第一个例程，后面我们会详细的讲到其中的点，先让自己的程序跑起来看看

```
public class MyBatisTest {
    public static void main(String[] args) throws Exception {
        //读取配置文件
        InputStream in = Resources.getResourceAsStream("SqlMapConfig.xml");
        //创建SqlSessionFactory工厂
        SqlSessionFactoryBuilder factoryBuilder = new SqlSessionFactoryBuilder();
        SqlSessionFactory factory = factoryBuilder.build(in);
        //使用工厂生产SqlSession对象
        SqlSession session = factory.openSession();
        //使用SqlSession创建Mapper接口的代理对象
        UserMapper userMapper = session.getMapper(UserMapper.class);
        //使用代理对象执行方法
        List<User> users = userMapper.findAllUserInfo();
        for (User user : users) {
            System.out.println(user);
        }
        //释放资源
        session.close();
        in.close();
    }
}
复制代码
```

### 第一个 MyBatis 程序（注解配置）

注解配置，自然我们刚才的 UserMapper.xml 就可以删除了掉了，同时我们需要在 mapper 接口方法中添加注解如图所示

```
public interface UserMapper {
    /**
     * 查询所有用户信息
     *
     * @return
     */
    @Select("select * from user")
    List<User> findAllUserInfo();
}
复制代码
```

然后在主配置文件中，使用 class 属性指定被注解的 mapper 全限定类名

```
<mappers>
	<mapper class="cn.ideal.mapper.UserMapper"/>
</mappers>
复制代码
```

两种方式执行结果都是一致的，如下图

![](https://user-gold-cdn.xitu.io/2020/2/2/170050f83516a6c2?imageView2/0/w/1280/h/960/format/webp/ignore-error/1)

自定义 MyBatis 框架 (首先使用 XML)
-------------------------

首先我们创建一个 Maven 工程，修改其 pom.xml 文件 增加一些必要依赖的坐标，由于我们使用 dom4j 的方式解析 xml 文件所以，需要引入 dom4j 和 jaxen

```
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>

    <groupId>cn.ideal</groupId>
    <artifactId>code_02_user_defined</artifactId>
    <version>1.0-SNAPSHOT</version>
    
    <dependencies>
        <dependency>
            <groupId>mysql</groupId>
            <artifactId>mysql-connector-java</artifactId>
            <version>5.1.6</version>
        </dependency>
        <dependency>
            <groupId>dom4j</groupId>
            <artifactId>dom4j</artifactId>
            <version>1.6.1</version>
        </dependency>
        <dependency>
            <groupId>jaxen</groupId>
            <artifactId>jaxen</artifactId>
            <version>1.1.6</version>
        </dependency>
    </dependencies>
    
</project>
复制代码
```

由于今天我们是要使用自定义 MyBatis 框架 进行一个简单的查询，即查询表中的所有用户信息，所以我们需要根据数据库内容，对应的创建出其 User 类实体

```
CREATE DATABASE ideal_mybatis; -- 创建数据库

CREATE TABLE USER (
  id		INT(11)NOT NULL AUTO_INCREMENT,
  username 	VARCHAR(32) NOT NULL COMMENT '用户名',
  telephone     VARCHAR(11) NOT NULL COMMENT '手机',
  birthday	DATETIME DEFAULT NULL COMMENT '生日',
  gender 	CHAR(1) DEFAULT NULL COMMENT '性别',
  address	VARCHAR(256) DEFAULT NULL COMMENT '地址',
  PRIMARY KEY  (`id`)
) ENGINE=INNODB DEFAULT CHARSET=utf8;
复制代码
```

创建出 UserMapper 接口，并书写相关的方法

```
package cn.ideal.mapper;

public interface UserMapper {
    /**
     * 查询所有用户信息
     */
    List<User> findAllUserInfo();
}
复制代码
```

设置好其主配置文件（SqlMapConfig.xml）需要说明的是：由于我们是要自己模拟设计一个 MyBatis 框架，所以不要习惯性的加上对应的 DTD 规范约束，以下是具体代码

```
<?xml version="1.0" encoding="UTF-8"?>
<!-- mybatis 主配置文件 -->
<configuration>
    <!-- 配置环境，和spring整合后 environments配置将会被废除 -->
    <environments default="development">
        <environment id="development">
            <!-- 使用JDBC事务管理 -->
            <transactionManager type="JDBC"></transactionManager>
            <!-- 数据库连接池 -->
            <dataSource type="POOLED">
                <property />
                <property />
                <property />
                <property />
            </dataSource>
        </environment>
    </environments>

    <!-- 指定映射配置文件的位置 -->
    <mappers>
        <mapper resource="cn/ideal/mapper/UserMapper.xml"/>
    </mappers>
</configuration>
复制代码
```

修改其 SQL 映射配置文件

```
<?xml version="1.0" encoding="UTF-8"?>
<mapper namespace="cn.ideal.mapper.UserMapper">
    <select id="findAllUserInfo" resultType="cn.ideal.domain.User">
        select * from user
    </select>
</mapper>
复制代码
```

测试类，我们的今天用的测试类，与直接使用 MyBatis 框架使用的测试类是一致的，只不过我们需要自己实现其中的一些类和方法

```
package cn.ideal.test;

import cn.ideal.domain.User;
import cn.ideal.mapper.UserMapper;
import cn.ideal.mybatis.io.Resources;
import cn.ideal.mybatis.sqlsession.SqlSession;
import cn.ideal.mybatis.sqlsession.SqlSessionFactory;
import cn.ideal.mybatis.sqlsession.SqlSessionFactoryBuilder;

import java.io.InputStream;
import java.util.List;

public class MyBatisTest {
    public static void main(String[] args) throws Exception {
        //读取配置文件
        InputStream in = Resources.getResourceAsStream("SqlMapConfig.xml");
        //创建SqlSessionFactory工厂
        SqlSessionFactoryBuilder factoryBuilder = new SqlSessionFactoryBuilder();
        SqlSessionFactory factory = factoryBuilder.build(in);
        //使用工厂生产SqlSession对象
        SqlSession session = factory.openSession();
        //使用SqlSession创建Mapper接口的代理对象
        UserMapper userMapper = session.getMapper(UserMapper.class);
        //使用代理对象执行方法
        List<User> users = userMapper.findAllUserInfo();
        for (User user : users) {
            System.out.println(user);
        }
        //释放资源
        session.close();
        in.close();
    }
}
复制代码
```

我们参考一下真正的 MyBatis 先规整一下，我们需要，Resources 以及 SqlSessionFactoryBuilder 类需要 SqlSessionFactory 、SqlSession 这样两个接口，然后我们还需要创建出其对应的实现类，其余的我们根据需要再进行创建，然后根据测试类中的需要，为创建出来的类加上**对应需要的方法**

###(一) Resources 类

首先先创建一个 Resources 类，并且在中增加一个，getResourceAsStream 方法，参数类型自然是一个字符串类型，根据测试类中的接收类型，判断出返回值类型为 InputStream，也因此可以写出具体的方法体了，如下图

```
package cn.ideal.mybatis.io;

import java.io.InputStream;

public class Resources {
    /**
     * 根据传入的参数，获取一个字节输入流
     * @param filePath
     * @return
     */
    public static InputStream getResourceAsStream(String filePath){
        // 获取当前类的字节码，获取字节码的类加载器，根据类加载器读取配置
        return Resources.class.getClassLoader().getResourceAsStream(filePath);
    }
}
复制代码
```

获取到主配置文件的字节输入流后，我们就需要将其中的 XML 解析出来，这里就用到了 Dom4j 的 XML 解析方式，在 Pom.xml 中我们已经引入了它的坐标，我们需要创建其工具类，当然我们可以找一份现成的工具类，也可以自己手动的去写一份，我们的重心还是在 MyBatis 的执行流程上

### (二) XMLConfigBuilder（解析 XML 工具类，理解即可）

由于我们首先使用的 XML 配置的方式，所以我们暂时先不管 XML 中的注解部分，所以注释掉的部分，暂时先不用理会

```
package cn.ideal.mybatis.utils;

import cn.ideal.mybatis.cfg.Configuration;
import cn.ideal.mybatis.cfg.Mapper;
import cn.ideal.mybatis.io.Resources;
//import com.itheima.mybatis.annotations.Select;

import org.dom4j.Attribute;
import org.dom4j.Document;
import org.dom4j.Element;
import org.dom4j.io.SAXReader;

import java.io.IOException;
import java.io.InputStream;
import java.lang.reflect.Method;
import java.lang.reflect.ParameterizedType;
import java.lang.reflect.Type;
import java.util.HashMap;
import java.util.List;
import java.util.Map;

/**
 *  用于解析配置文件
 */
public class XMLConfigBuilder {
    /**
     * 解析主配置文件，把里面的内容填充到DefaultSqlSession所需要的地方
     * 使用的技术：
     *      dom4j+xpath,所以需要导入dom4j和jaxen的坐标
     */
    public static Configuration loadConfiguration(InputStream config){
        try{
            //定义封装连接信息的配置对象（mybatis的配置对象）
            Configuration cfg = new Configuration();

            //1.获取SAXReader对象
            SAXReader reader = new SAXReader();
            //2.根据字节输入流获取Document对象
            Document document = reader.read(config);
            //3.获取根节点
            Element root = document.getRootElement();
            //4.使用xpath中选择指定节点的方式，获取所有property节点
            List<Element> propertyElements = root.selectNodes("//property");
            //5.遍历节点
            for(Element propertyElement : propertyElements){
                //判断节点是连接数据库的哪部分信息
                //取出name属性的值
                String name = propertyElement.attributeValue("name");
                if("driver".equals(name)){
                    //表示驱动
                    //获取property标签value属性的值
                    String driver = propertyElement.attributeValue("value");
                    cfg.setDriver(driver);
                }
                if("url".equals(name)){
                    //表示连接字符串
                    //获取property标签value属性的值
                    String url = propertyElement.attributeValue("value");
                    cfg.setUrl(url);
                }
                if("username".equals(name)){
                    //表示用户名
                    //获取property标签value属性的值
                    String username = propertyElement.attributeValue("value");
                    cfg.setUsername(username);
                }
                if("password".equals(name)){
                    //表示密码
                    //获取property标签value属性的值
                    String password = propertyElement.attributeValue("value");
                    cfg.setPassword(password);
                }
            }
            //取出mappers中的所有mapper标签，判断他们使用了resource还是class属性
            List<Element> mapperElements = root.selectNodes("//mappers/mapper");
            //遍历集合
            for(Element mapperElement : mapperElements){
                //判断mapperElement使用的是哪个属性
                Attribute attribute = mapperElement.attribute("resource");
                if(attribute != null){
                    System.out.println("XML方式");
                    //表示有resource属性，用的是XML
                    //取出属性的值
                    String mapperPath = attribute.getValue();//获取属性的值"com/itheima/dao/IUserDao.xml"
                    //把映射配置文件的内容获取出来，封装成一个map
                    Map<String, Mapper> mappers = loadMapperConfiguration(mapperPath);
                    //给configuration中的mappers赋值
                    cfg.setMappers(mappers);
                }else{
//                    System.out.println("注解方式");
//                    //表示没有resource属性，用的是注解
//                    //获取class属性的值
//                    String daoClassPath = mapperElement.attributeValue("class");
//                    //根据daoClassPath获取封装的必要信息
//                    Map<String,Mapper> mappers = loadMapperAnnotation(daoClassPath);
//                    //给configuration中的mappers赋值
//                    cfg.setMappers(mappers);
                }
            }
            //返回Configuration
            return cfg;
        }catch(Exception e){
            throw new RuntimeException(e);
        }finally{
            try {
                config.close();
            }catch(Exception e){
                e.printStackTrace();
            }
        }

    }

    /**
     * 根据传入的参数，解析XML，并且封装到Map中
     * @param mapperPath    映射配置文件的位置
     * @return  map中包含了获取的唯一标识（key是由dao的全限定类名和方法名组成）
     *          以及执行所需的必要信息（value是一个Mapper对象，里面存放的是执行的SQL语句和要封装的实体类全限定类名）
     */
    private static Map<String,Mapper> loadMapperConfiguration(String mapperPath)throws IOException {
        InputStream in = null;
        try{
            //定义返回值对象
            Map<String,Mapper> mappers = new HashMap<String,Mapper>();
            //1.根据路径获取字节输入流
            in = Resources.getResourceAsStream(mapperPath);
            //2.根据字节输入流获取Document对象
            SAXReader reader = new SAXReader();
            Document document = reader.read(in);
            //3.获取根节点
            Element root = document.getRootElement();
            //4.获取根节点的namespace属性取值
            String namespace = root.attributeValue("namespace");//是组成map中key的部分
            //5.获取所有的select节点
            List<Element> selectElements = root.selectNodes("//select");
            //6.遍历select节点集合
            for(Element selectElement : selectElements){
                //取出id属性的值      组成map中key的部分
                String id = selectElement.attributeValue("id");
                //取出resultType属性的值  组成map中value的部分
                String resultType = selectElement.attributeValue("resultType");
                //取出文本内容            组成map中value的部分
                String queryString = selectElement.getText();
                //创建Key
                String key = namespace+"."+id;
                //创建Value
                Mapper mapper = new Mapper();
                mapper.setQueryString(queryString);
                mapper.setResultType(resultType);
                //把key和value存入mappers中
                mappers.put(key,mapper);
            }
            return mappers;
        }catch(Exception e){
            throw new RuntimeException(e);
        }finally{
            in.close();
        }
    }

    /**
     * 根据传入的参数，得到dao中所有被select注解标注的方法。
     * 根据方法名称和类名，以及方法上注解value属性的值，组成Mapper的必要信息
     * @param daoClassPath
     * @return
     */
//    private static Map<String,Mapper> loadMapperAnnotation(String daoClassPath)throws Exception{
//        //定义返回值对象
//        Map<String,Mapper> mappers = new HashMap<String, Mapper>();
//
//        //1.得到dao接口的字节码对象
//        Class daoClass = Class.forName(daoClassPath);
//        //2.得到dao接口中的方法数组
//        Method[] methods = daoClass.getMethods();
//        //3.遍历Method数组
//        for(Method method : methods){
//            //取出每一个方法，判断是否有select注解
//            boolean isAnnotated = method.isAnnotationPresent(Select.class);
//            if(isAnnotated){
//                //创建Mapper对象
//                Mapper mapper = new Mapper();
//                //取出注解的value属性值
//                Select selectAnno = method.getAnnotation(Select.class);
//                String queryString = selectAnno.value();
//                mapper.setQueryString(queryString);
//                //获取当前方法的返回值，还要求必须带有泛型信息
//                Type type = method.getGenericReturnType();//List<User>
//                //判断type是不是参数化的类型
//                if(type instanceof ParameterizedType){
//                    //强转
//                    ParameterizedType ptype = (ParameterizedType)type;
//                    //得到参数化类型中的实际类型参数
//                    Type[] types = ptype.getActualTypeArguments();
//                    //取出第一个
//                    Class domainClass = (Class)types[0];
//                    //获取domainClass的类名
//                    String resultType = domainClass.getName();
//                    //给Mapper赋值
//                    mapper.setResultType(resultType);
//                }
//                //组装key的信息
//                //获取方法的名称
//                String methodName = method.getName();
//                String className = method.getDeclaringClass().getName();
//                String key = className+"."+methodName;
//                //给map赋值
//                mappers.put(key,mapper);
//            }
//        }
//        return mappers;
//    }
}

复制代码
```

我们可以大致看一下代码，首先**获取所有 property 节点，将驱动，用户名，密码等的值获取出来**，然后

**取出 mappers 中的所有 mapper 标签，判断他们使用了 resource 还是 class 属性**，这就判断出使用了 XML 还是注解的方式，例如本例中的 XML 方式，就获取到了主配置文件中 `<mapper resource="cn/ideal/mapper/UserMapper.xml"/>` 这一句中`cn/ideal/mapper/UserMapper.xml` 然后对这个 SQL 映射的配置文件进行解析，同样将其中一些必要的信息提取出来

但是我们如果想要使用这个工具类，可以看到还是有一些报错的地方，这就是因为我们缺少一些必要的类，我们需要自己，补充一下

首先我们需要创建一个 Configuration 实体类，用来传递我们获取到的 一些链接信息

```
public class Configuration {
    
    private String driver;
    private String url;
    private String username;
    private String password;
    private Map<String, Mapper> mappers = new HashMap<String, Mapper>();
    
 	补充其对应get set方法   
    //特别说明：setMappers 需要使用putALL的追加写入方式，不能直接赋值，不然旧的就会被新的覆盖掉
    public void setMappers(Map<String, Mapper> mappers) {
        this.mappers.putAll(mappers); // 追加的方式
    }
}    
复制代码
```

其次我们可以看到，对 SQL 映射文件进行解析的时候，我们将其封装成一个 Map 集合，其中 key 是由 mapper 的全限定类名和方法名组成的， 即通过分别获取两者的值，然后进行字符串拼接成 key，value 值是一个 Mapper 对象，它的 key 为 id ，而 value 值为 SQL 语句 以及 resultType

```
public class Mapper {

	private String queryString; //SQL
	private String resultType; //实体类的全限定类名
	
	补充其对应get set方法 
}
复制代码
```

### (三) SqlSessionFactoryBuilder 类

我们继续回到测试类中，我们需要创建一个 SqlSessionFactoryBuilder 类 ，根据测试类看到，我们将 Recourse 类中获取到的流文件，作为参数传递到了本类中的 build 方法中，这也就是说，我们在 build 方法中，需要对 XML 进行解析，然后使用刚才创建的 Configuration 类进行接收，但是根据测试类中，我们可以知道，测试中使用的是一个 SqlSessionFactory 类型来接收 build 的返回，而通过 真正 MyBatis 可知，SqlSessionFactory 是一个接口，这里使用了建造者设计模式，所以我们真正要返回的就是其对应的实现类，并且将 Configuration 的对象传进去，代码如下所示，

```
public class SqlSessionFactoryBuilder {
    public  SqlSessionFactory build(InputStream config){
        Configuration cfg = XMLConfigBuilder.loadConfiguration(config);
        return new DefaultSqlSessionFactory(cfg);
    }
}
复制代码
```

### (四) SqlSessionFactory 接口

```
public interface SqlSessionFactory {
    /**
     * 用来打开一个新的SqlSession对象
     * @return
     */
    SqlSession openSession();
}
复制代码
```

### (五) DefaultSqlSessionFactor 实现类

由于我们将配置传入了方法中，所以必要的，我们需要先创建一个 Configuration 成员，然后创建一个带参构造，同样根据测试类创建其 openSession 方法，用于创建操作数据的对象，这里是同样的思路，SqlSession 是一个接口，所以我们真正需要返回的还是其接口

```
public class DefaultSqlSessionFactory implements SqlSessionFactory {

    private Configuration cfg;
    public DefaultSqlSessionFactory(Configuration cfg) {
        this.cfg = cfg;
    }

    /**
     * 用于创建一个新的操作数据库对象
     * @return
     */
    public SqlSession openSession() {
        return new DefaultSqlSession(cfg);
    }
}
复制代码
```

### (六) SqlSession 接口

在这里我们需要使用 SqlSession 创建 Mapper 接口的代理对象

```
public interface SqlSession {
    /**
     * 根据参数创建一个代理对象
     * @param mapperInterfaceClass mapper的接口字节码
     * @param <T>
     * @return
     */
    <T> T getMapper(Class<T> mapperInterfaceClass);

    /**
     * 释放资源
     */
    void close();
}
复制代码
```

### (七) DefaultSqlSession 实现类

首先依旧是创建成员以及带参构造方法，我们需要创建一个工具类，用来创建数据源，起名为 DataSourceUtil，同时这个类中比较重要的是创建我们的 getMapper 方法，我们使用 Proxy.newProxyInstance 这个方法进行，其中的参数，

*   第一个参数即我们需要代理的类加载器，也就是代理谁，就用谁的类加载器
*   第二个参数就是动态代理类需要实现的接口
*   第三个参数 动态代理方法在执行时，会调用里面的方法去执行 （自创）
    *   其中的参数，就是将我们配置中的信息传入

```
public class DefaultSqlSession implements SqlSession {

    private Configuration cfg;
    private Connection connection;

    public DefaultSqlSession(Configuration cfg) {
        this.cfg = cfg;

        connection = DataSourceUtil.getConnection(cfg);
    }

    /**
     * 用于创建代理对象
     *
     * @param mapperInterfaceClass mapper的接口字节码
     * @param <T>
     * @return
     */
    public <T> T getMapper(Class<T> mapperInterfaceClass) {
        return (T) Proxy.newProxyInstance(mapperInterfaceClass.getClassLoader(),
                new Class[]{mapperInterfaceClass},new MapperProxy(cfg.getMappers(),connection));
    }

    /**
     * 用于释放资源
     */
    public void close() {
        if (connection != null) {
            try {
                connection.close();
            } catch (SQLException e) {
                e.printStackTrace();
            }
        }
    }
}

复制代码
```

### (八) DataSourceUtil 工具类

```
public class DataSourceUtil {
    public static Connection getConnection(Configuration cfg){
        try {
            Class.forName(cfg.getDriver());
            return DriverManager.getConnection(cfg.getUrl(),cfg.getUsername(),cfg.getPassword());
        } catch (Exception e) {
            throw new RuntimeException(e);
        }
    }
}
复制代码
```

### (九) MapperProxy

我们开始实现刚才自定义的一个 MapperProxy 类，常规的创建成员以及构造函数，其中传入 Connection 的原因是，为了最后执行 Executor() 的需要， 这个类还有一个重要的就是使用 invoke 对方法进行增强，将其获取并组合，

```
public class MapperProxy implements InvocationHandler {

    //map的key是全限定类名 + 方法名
    private Map<String, Mapper> mappers;
    private Connection connection;

    public MapperProxy(Map<String, Mapper> mappers,Connection connection) {
        this.mappers = mappers;
        this.connection = connection;
    }

    /**
     * 用于对方法进行增强的，这里的增强就是调用 selectList方法
     * @param proxy
     * @param method
     * @param args
     * @return
     * @throws Throwable
     */
    public Object invoke(Object proxy, Method method, Object[] args) throws Throwable {
        //获取方法名
        String methodName = method.getName();
        //获取方法所在类的名称
        String className = method.getDeclaringClass().getName();
        //组合key
        String key = className + "." + methodName;
        //获取mappers中的Mapper对象
        Mapper mapper = mappers.get(key);
        //判断是否有mapper
        if (mapper == null){
            throw  new IllegalArgumentException("传入的参数有误");
        }
        //调用工具类执行查询所有1
        return new Executor().selectList(mapper,connection);
    }
}

复制代码
```

### (十) Executor 工具类

接着就是使用一个现成的工具类进行执行我们的 SQL，因为 SQL 语句，以及 resultType 已经被封装在了 mapper 中被传递了进来

```
/**
 * 负责执行SQL语句，并且封装结果集
 */
public class Executor {

    public <E> List<E> selectList(Mapper mapper, Connection conn) {
        PreparedStatement pstm = null;
        ResultSet rs = null;
        try {
            //1.取出mapper中的数据
            String queryString = mapper.getQueryString();//select * from user
            String resultType = mapper.getResultType();//com.itheima.domain.User
            Class domainClass = Class.forName(resultType);
            //2.获取PreparedStatement对象
            pstm = conn.prepareStatement(queryString);
            //3.执行SQL语句，获取结果集
            rs = pstm.executeQuery();
            //4.封装结果集
            List<E> list = new ArrayList<E>();//定义返回值
            while(rs.next()) {
                //实例化要封装的实体类对象
                E obj = (E)domainClass.newInstance();

                //取出结果集的元信息：ResultSetMetaData
                ResultSetMetaData rsmd = rs.getMetaData();
                //取出总列数
                int columnCount = rsmd.getColumnCount();
                //遍历总列数
                for (int i = 1; i <= columnCount; i++) {
                    //获取每列的名称，列名的序号是从1开始的
                    String columnName = rsmd.getColumnName(i);
                    //根据得到列名，获取每列的值
                    Object columnValue = rs.getObject(columnName);
                    //给obj赋值：使用Java内省机制（借助PropertyDescriptor实现属性的封装）
                    PropertyDescriptor pd = new PropertyDescriptor(columnName,domainClass);//要求：实体类的属性和数据库表的列名保持一种
                    //获取它的写入方法
                    Method writeMethod = pd.getWriteMethod();
                    //把获取的列的值，给对象赋值
                    writeMethod.invoke(obj,columnValue);
                }
                //把赋好值的对象加入到集合中
                list.add(obj);
            }
            return list;
        } catch (Exception e) {
            throw new RuntimeException(e);
        } finally {
            release(pstm,rs);
        }
    }


    private void release(PreparedStatement pstm,ResultSet rs){
        if(rs != null){
            try {
                rs.close();
            }catch(Exception e){
                e.printStackTrace();
            }
        }

        if(pstm != null){
            try {
                pstm.close();
            }catch(Exception e){
                e.printStackTrace();
            }
        }
    }
}
复制代码
```

到这里我们就可以来测试一下了，结果是没有什么问题的

![](https://user-gold-cdn.xitu.io/2020/2/2/170050f8355d3393?imageView2/0/w/1280/h/960/format/webp/ignore-error/1)

如何使用注解方式 自定 MyBatis
-------------------

首先我们需要在主配置文件中修改为注解的形式，即

```
<mappers>
	<mapper class="cn.ideal.mapper.UserMapper"/>
</mappers>
复制代码
```

然后创建一个自定义的 Select 注解

```
@Retention(RetentionPolicy.RUNTIME)
@Target(ElementType.METHOD)
public @interface Select {
    /**
     * 配置SQL语句的
     * @return
     */
    String value();
}
复制代码
```

然后到我们的 UserMapper 接口方法上添加注解，将 SQL 语句写在其中

```
public interface UserMapper {
    /**
     * 查询所有用户信息
     *
     * @return
     */
    @Select("select * from user")
    List<User> findAllUserInfo();
}
复制代码
```

最后我们将 XMLConfigBuilder 工具类中 注释掉注解的部分取消注释就可以了

运行结果如下：

![](https://user-gold-cdn.xitu.io/2020/2/2/170050f835a1df1b?imageView2/0/w/1280/h/960/format/webp/ignore-error/1)

最后，给大家程序结构图，方便大家创建包，结构为 cn.ideal.xxxx

![](https://user-gold-cdn.xitu.io/2020/2/2/170050f8352269f1?imageView2/0/w/1280/h/960/format/webp/ignore-error/1)

总结
--

我重新捋一下，整个流程

*   MyBatis 相关的配置文件，例中的 SqlMapConfig.xml 被 Rosource 类中的方法读取，得到一个流文件
*   在 SqlSessionFactoryBuilder 中通过 XMLConfigBuilder 对配置文件进行解析，同时使用 Configuration 类，存取提取出来具体的一些配置信息
*   通过 SqlSessionFactory 创建新的操作数据库的对象
*   获取到 SqlSession
*   使用代理模式 MapperProxy 执行 SQL，本质是调用 Executor 执行器
*   测试运行

自制一个简单的流程图

![](https://user-gold-cdn.xitu.io/2020/2/2/1700514439aa8e9d?imageView2/0/w/1280/h/960/format/webp/ignore-error/1)

--




