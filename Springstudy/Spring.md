# Spring的核心概念

IOC：

控制反转

DI：

依赖注入

AOP:

面向切面编程



核心模块：core container

Beans:bean工厂中bean的装配，创建对象的工作

Core:  IOC和DI最基本的实现

Context:IOC容器，SPRING容器

SpEL:Spring的表达式语言



数据访问层模块：

JDBC：封装

ORM：数据集成框架的封装

OXM：实现对象和XML互相转换的支持

JMS：生产者和消费者相互消息功能的实现

Transactions：事务管理





web模块：内容集成和内容聚合

面向web的功能

Websocket：套接字

Servlet：网络服务

Portlet：窗口



Instrumentation模块：检测器，Tomcat，JRM检测

Test模块：方便测试，Spring帮助处理的大部分；



## 装配bean的三种方式

在xml中进行显示的设置

#### 自动装配

（注解注释）

组建扫描：

@Component:表示这个类在应用程序中被创建

@ComponentScan：自动发现应用程序中创建的类

自动装配：

@Autowired：自动满足bean之间的依赖

自动配置类

@Configeration：表示当前类是一个配置类



Junit4 单元测试

引入junit4依赖

```java
<dependency>
    <groupId>junit</groupId>
    <artifactId>junit</artifactId>
    <version>4.12</version>
</dependency>
```

引入Springtest依赖

```java
<dependency>
    <groupId>org.springframework</groupId>
    <artifactId>spring-test</artifactId>
    <version>5.2.3.RELEASE</version>
</dependency>
```

```java
@RunWith(SpringJUnit4ClassRunner.class)
```

注解，自动帮助创建Spring的上下文环境

```java
@ContextConfiguration(classes = AppConfig.class)
```

注解：读取配置文件



#### @Autowired的使用场景



1，用在构造函数（效率最高，推荐）

```java
@Autowired
public CDplayer(CompactDisk cd) {
    this.cd = cd;
    System.out.println("CDplayer的有参构造函数");
}
```

2，用在成员变量(方便，效率低)

```java
@Autowired
    private CompactDisk cd;
@Autowired
    private Power power;
```

3，用在传统Setter方法上

```java
@Autowired

public void setCd(CompactDisk cd) {
    this.cd = cd;
    System.out.println("调用SetCd");
}

@Autowired
public void setPower(Power power) {
    this.power = power;
    System.out.println("调用SetPower");
}
```

4，用在任意方法

```java
@Autowired
    public void prepare(CompactDisk cd,Power power)
    {
        this.cd = cd;
        this.power = power;
        System.out.println("prepare方法被调用");
    }
```

required关键字：表示注入的对象是可选的

eg：@autowired（required = false）满足业务逻辑需要；





#### Spring中使用接口

创建一个接口

```java
public interface UserService {
    void add();
}
```

实现类继承接口，注意！component注解一定要写在实现类中，不能写在接口上

```java
@Component
public class UserServiceNormal implements UserService {
    public void add() {
        System.out.println("添加用户");
    }
}
```

在调用的时候声明接口类型的变量

```java
@Autowired
private UserService userService;
```



#### 自动装配的歧义性

在实例中，多个实现类继承同一个接口，可能导致重名无法创建Bean，可以用以下方法去解决

1，设置首选bean：在其中一个实现类添加@Primary(只能定义一个，不能设置多个，且需要预定义，有局限性)

```java
@Component
@Primary
public class UserServiceFestival implements UserService {
    public void add() {
        System.out.println("注册用户发送优惠券");
    }
}
```

2，使用限定符：

```java
@Component
@Qualifier("festival")    //限定符定义service为festival类型
public class UserServiceFestival implements UserService {
    public void add() {
        System.out.println("注册用户发送优惠券");
    }
}
```

测试，在装配的时候要使用@Qualifier的注解

```java
@RunWith(SpringJUnit4ClassRunner.class)
@ContextConfiguration(classes = Appconfig.class)
public class UserServiceTest {

    @Autowired
    @Qualifier("festival")
    private UserService userService;


   @Test
    public void TestAdd(){
        userService.add();
    }
}
```

3，在声明bean组件时在@Component后面声明属性

```java
@Component("normal")
//@Qualifier("normal")
```

也就是声明类的id

或者在@Qualifier里面写小写开头的类名（Spring默认的id）

```java
@Qualifier("userServiceNormal")
```



##### Java标准的解决方案

可以用@resource注解来解决（基于jdk核心包），但是我的jdk版本似乎不支持







#### web程序的基本架构

浏览器

web层(Controller)        注解：@Controller

业务层(Service)             			@Service

数据访问层(Dao)					 @Repository

数据库(Data)



#### 通过java配置文件进行组件扫描

```java
@Configuration
//@ComponentScan("com.gybedu.demo")
//@ComponentScan(basePackages {"com.gybedu.demo.web","com.gybedu.demo.dao","com.gybedu.demo.service"} )  //字符串储存，重构不安全
@ComponentScan(basePackageClasses = {UserService.class, UserController.class, UserDao.class})
public class Appconfig {
}
```

#### 通过配置XML配置文件也可以：

在resources包里面添加一个applicationContext.xml的文件，详细如下：

```java
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xmlns:context="http://www.springframework.org/schema/context"
       xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd http://www.springframework.org/schema/context https://www.springframework.org/schema/context/spring-context.xsd">

    <context:component-scan base-package="com.gybedu.demo"/>  //放的位置
</beans>
```

```java
@ContextConfiguration("classpath:applicationContext.xml")
```

在测试的时候就改成👆上面这样

****

### **Java装配**

在JavaConfig里装配bean对象

```java
@Configuration
public class Appconfig {

    @Bean   //
    public UserDao userDaoNormal(){
        System.out.println("创建UserDao对象");
        return new UserDaoNormal();
    }

}
```

#### 依赖注入的方法：

添加构造函数：

```java
public UserServiceNormal(UserDao userDao) {
    this.userDao = userDao;
}
```

```java
@Bean
public UserService userServiceNormal(){

    System.out.println("创建Userservice对象");

    UserDao userDao = userDaoNormal();      //不会反复调用

    return new UserServiceNormal(userDao);   //将UserDaoNormal和UserServiceNormal建立联系
}
```

​                    									这样写看起来更舒服

```java
@Bean
public UserService userServiceNormal(UserDao userDao){

    System.out.println("创建Userservice对象");

    return new UserServiceNormal(userDao);
}
```

用Setter方法依赖注入

```java
public void setUserDao(UserDao userDao) {
    this.userDao = userDao;
}
```

```java
@Bean
public UserService userServiceNormal(UserDao userDao){
    System.out.println("创建Userservice对象");
    UserServiceNormal userService = new UserServiceNormal(userDao);
    userService.setUserDao(userDao);
    return userService;
```

java装配的歧义性

```java
@Bean
@Qualifier("normal")  //限定符的方法
public UserDao userDaoNormal(){

    System.out.println("创建UserDao对象");
    return new UserDaoNormal();
}

@Bean("cache")    //id方法
public UserDao userDaoCache(){ 

    System.out.println("创建UserCache对象");
    return new UserDaoCache();
}

@Bean
@Primary        //首选bean
public UserDao userDaoRom(){
    System.out.println("创建UserRom对象");
    return new UserDaoRom();
}

@Bean
public UserService userServiceNormal(@Qualifier("normal")UserDao userDao){
    System.out.println("创建Userservice对象");
    UserServiceNormal userService = new UserServiceNormal(userDao);
    userService.setUserDao(userDao);
    return userService;
}
```



### Xml装配（麻烦）

xml作为描述的主要方式

```java
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd">

    <bean class="com.gybedu.demo.SoundSystem.CompatctDisc"/>  //装配的路径   
</beans>
```

调用

```java
public class ApplicationSpring {
    public static void main(String[] args) {
        System.out.println("Application is running......");
//读取配置文件，Spring进行初始化容器
        ClassPathXmlApplicationContext context = new ClassPathXmlApplicationContext("applicationContext.xml");
// 把获取到的bean对象给cd这个变量
        CompatctDisc cd = context.getBean(CompatctDisc.class);
        cd.play();
    }
}
```

```java
CompatctDisc cd = (CompatctDisc) context.getBean("compactDisc1");
```

```java
<bean id="compatctDisc1" class="com.gybedu.demo.SoundSystem.CompatctDisc"/>
```

也可以使用以上方法对bean对象进行id划分。。。。。（实际操作时可能存在定义多个bean对象）





#### 单元测试

```java
@RunWith(SpringJUnit4ClassRunner.class)
@ContextConfiguration("classpath:applicationContext.xml")
public class AppTest {

    @Autowired
    private CompatctDisc compactDisc1;
    @Autowired
    private CompatctDisc compactDisc2;
    @Autowired
    @Qualifier("compactDisc2")
    private CompatctDisc compactDisc3;

    @Test
    public void test01(){
        compactDisc1.play();
        compactDisc2.play();
        compactDisc3.play();
    }
}
```



#### 尤其注意id和name在spring容器中是唯一的！！

```java
<bean name="compactDisc1 name1 name2" class="com.gybedu.demo.SoundSystem.CompatctDisc"/>
```

但是按照上面的格式，可以取别名，方便调用；

id后面跟的是一整个字符串，所以不能取别名

```java
private CompatctDisc compactDisc1 name1 name2;
```

上面错误示范

#### xml中利用构造函数进行依赖注入

创建cdplayer类

```java
public class CDplayer {
    private CompatctDisc cd;

    public CDplayer() {
        super();
        System.out.println("CDplay的构造函数"+this.toString());
    }

    public CDplayer(CompatctDisc cd) {
        this.cd = cd;
        System.out.println("CDplay的有参构造函数"+this.toString());
    }
    public void play(){
        cd.play();
    }
}
```

```java
<bean id="cdplayer1" class="com.gybedu.demo.SoundSystem.CDplayer">
    <constructor-arg ref="compactDisc1" />
</bean>
```

spring容器启动时，创建cdplayer1对象，constructor告诉spring要将id=compactDisk1引用传递到cdplayer的构造函数中

```java
/**
 * @author gybin___NN
 * @data 2020 - 02 - ${DATA} - 23:13
 */
@RunWith(SpringJUnit4ClassRunner.class)
@ContextConfiguration("classpath:applicationContext.xml")
public class CDplayerTest {

    @Autowired
    private CDplayer cDplayer1;

    @Test
    public void Test(){
        cDplayer1.play();
    }
```

Constructor-arg作为构造函数的依赖注入方法，另外还有c名称注入的方法

#### c名称注入

```java
xmlns:c="http://www.springframework.org/schema/c"
```

配置文件里加上上面这句话

```java
<bean id="cdplayer2" class="com.gybedu.demo.SoundSystem.CDplayer" c:cd-ref="compactDisc2"> </bean>
```

使用格式：c:+构造函数参数的名称-+ref="需要引用的对象名"





```java
<bean name="compactDisc1" class="com.gybedu.demo.SoundSystem.CompatctDisc">
    <constructor-arg  name="title" value="I DO"/>
    <constructor-arg name="artist" value="莫文蔚"/>
</bean>
<bean id="compactDisc2" class="com.gybedu.demo.SoundSystem.CompatctDisc"
      c:title="周杰伦的床边故事"
      c:artist="周杰伦"/>
```

以上两种方法简单类型的依赖注入。







### list类型的注入

```java
<constructor-arg name="tracks">
    <list>
        <value>I Do 1</value>
        <value>I DO 2</value>
        <value>I DO 3</value>
    </list>
</constructor-arg>
```

```java
for (String track : this.tracks) {
    System.out.println("音乐："+track);
}
```

建立一个类的调用

```java
public class Music {

    private String title;
    private Integer duratino;

    public String getTitle() {
        return title;
    }

    public void setTitle(String title) {
        this.title = title;
    }

    public Integer getDuratino() {
        return duratino;
    }

    public void setDuratino(Integer duratino) {
        this.duratino = duratino;
    }

    public Music(String title, Integer duratino) {
        this.title = title;
        this.duratino = duratino;
    }
}
```

配置文件

```java
<bean id="music1" class="com.gybedu.demo.SoundSystem.Music">
    <constructor-arg value="I DO 1"/>
    <constructor-arg value="270"/>
</bean>

<bean id="music2" class="com.gybedu.demo.SoundSystem.Music">
<constructor-arg value="I DO 2"/>
<constructor-arg value="280"/>
</bean>

<bean id="music3" class="com.gybedu.demo.SoundSystem.Music">
    <constructor-arg value="I DO 3"/>
    <constructor-arg value="290"/>
</bean>
```

```java
<constructor-arg name="tracks">
    <list>
        <ref bean="music1"/>
        <ref bean="music2"/>
        <ref bean="music3"/>
    </list>
</constructor-arg>
```

#### 用ref的方法注入

```java
public void play(){
    System.out.println("播放CD音乐。。。。"+this.toString()+" "+this.title+"by"+this.artist);
    for (Music track : this.tracks) {
        System.out.println("音乐："+ track.getTitle() + "，时长" + track.getDuratino());
    }
}
```

调用的方法



set方法和list方法差不多一样，但set中重复的值会被忽略掉；

实现方式是linkedHashset

### 注入map类型

预定义一个map类型的变量

```java
private Map<String,Music> tracks;
```

```java
<constructor-arg name="tracks">
    <map>
        <entry key="m1" value-ref="music1"/>
        <entry key="m2" value-ref="music2"/>
        <entry key="m3" value-ref="music3"/>
    </map>
</constructor-arg>
```

```java
for (String key : this.tracks.keySet()) {
    System.out.println("key:"+key);
    Music music = this.tracks.get(key);
    System.out.println("音乐："+ music.getTitle() + "，时长" + music.getDuratino());
}
```

### 注入数组类型的参数

```java
private Music[] tracks;
```

用以下方式注入

```java
<constructor-arg name="tracks">
    <array>
        <!--<value>I Do</value>-->
        <ref bean="music1"/>
        <ref bean="music2"/>
        <ref bean="music3"/>
    </array>
</constructor-arg>
```



### 属性注入的方法

关键字<property />

配置文件

```java
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd">

    <bean id="music1" class="com.gybedu.demo.SoundSystem.Music">
        <property name="title" value="告白气球" />
        <property name="duratino" value="215" />
    </bean>

    <bean id="music2" class="com.gybedu.demo.SoundSystem.Music">
        <property name="title" value="爱情废柴" />
        <property name="duratino" value="305" />
    </bean>
</beans>
```

```java
public Music(){
    super();
    System.out.println("Music的构造函数"+this.toString());
}
```

走无参的构造函数

用property方法注入,数组，map，list类型都和构造函数注入方法一样，参考上面;

```java
<bean id="compactdisc1" class="com.gybedu.demo.SoundSystem.CompatctDisc">
    <property name="title" value="周杰伦的床边故事" />
    <property name="artist" value="周杰伦" />
    <property name="tracks"  >
        <array>
            <!--<value>I Do</value>-->
            <ref bean="music1"/>
            <ref bean="music2"/>

        </array>
    </property>
</bean>
```

p名称空间注入

```java
xmlns:p="http://www.springframework.org/schema/p"
```

引入

```java
<bean id="music2" class="com.gybedu.demo.SoundSystem.Music"
        p:title="爱情废材"
        p:duratino="305"/>
```

这样更简洁

```java
<bean id="CDplayer1" class="com.gybedu.demo.SoundSystem.CDplayer" p:cd-ref="compactdisc1">
</bean>
```

引用对象

集合和数组没有p名称的方法，所以还是得用propertiy的方法，还有util；

util名称空间：

创建一个列表bean

```java
<util:list> </util:list>
```

idea会自动引入头文件

```java
<util:list id="trackList">
    <ref bean="music1"/>
    <ref bean="music2"/>
</util:list>


<bean id="compactdisc1" class="com.gybedu.demo.SoundSystem.CompatctDisc" 
      p:title="周杰伦的床边故事" 
      p:artist="周杰伦" 
      p:tracks-ref="trackList">
</bean>
```

set和map方法同上，引入同样的util标签

```java
<util:map>
    
</util:map>
<util:set>
    
</util:set>
```

### spring的高级装配

#### bean的作用域

spring中的bean都是以单例的形式默认创立的

1.无论我们是否去主动获取或注入bean对象，Spring上下文一加载就会创建bean对象;

2.无论获取/注入多少次，拿到的都是同一个对象

