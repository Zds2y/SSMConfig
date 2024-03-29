

## Spring的xml文件配置

### 1、数据源的配置

#### 读取properties配置文件：

```xml
<context:property-placeholder location="classpath:jdbc.properties"/>
```

#### **druid：**

```java
<bean id="dataSource-xml" class="com.alibaba.druid.pool.DruidDataSource">
    <!-- 给数据源中的连接属性赋值 -->
    <property name="driverClassName" value="${jdbc.driver}"/>
    <property name="url" value="${jdbc.url}"/>
    <property name="username" value="${jdbc.username}"/>
    <property name="password" value="${jdbc.password}"/>
</bean>
```

#### **c3p0：**

```java
<bean id="dataSource-xml" class="com.mchange.v2.c3p0.ComboPooledDataSource">
    <!-- 给数据源中的连接属性赋值 -->
    <property name="driverClassName" value="${jdbc.driver}"/>
    <property name="url" value="${jdbc.url}"/>
    <property name="username" value="${jdbc.username}"/>
    <property name="password" value="${jdbc.password}"/>
</bean>
```

#### **jdbTemplate：**

```java
<bean id="jdbcTemplate-xml" class="org.springframework.jdbc.core.JdbcTemplate">
    <property name="dataSource" ref="dataSource-xml"/>
</bean>
```

#### **注解组件扫描**

```java
<context:component-scan base-package="com.moilion_zds.aop"/>
```

### **2、spring AOP**

导包：aspectjweaver包是第三方AOPjar包

```java
<!-- 导入第三方aop依赖 -->
<dependency>
    <groupId>org.aspectj</groupId>
    <artifactId>aspectjweaver</artifactId>
    <version>1.8.4</version>
</dependency>
```

#### xml方式配置：

定义一个切面类：

```java
public class MyAspect {
    //前置增强
    public void before(){
        System.out.println("前置增强");
    }
    //后置增强
    public void afterReturning(){
        System.out.println("后置增强");
    }
    //环绕增强  要传入一个切点ProceedingJoinPoint
    public Object around(ProceedingJoinPoint joinPoint) throws Throwable {
        System.out.println("环绕前增强");
        Object obj = joinPoint.proceed();
        System.out.println("环绕后增强");
        //如果目标方法有返回值 这里就必须要返回 否则方法取不到返回值
        return obj;
    }
    //异常抛出增强  当被增强的方法中出现异常的时候 执行
    public void afterThrowing(){
        System.out.println("异常抛出增强");
    }
    //最终增强  不管是否出现异常 都会执行 类似于finally
    public void after(){
        System.out.println("最终增强");
    }
}
```

```java
<!-- xml配置aop -->
<bean id="target" class="com.moilion_zds.aop_xml.Target"/>
<bean id="myAspect" class="com.moilion_zds.aop_xml.MyAspect"/>

<aop:config>
    <!-- 抽取切点表达式 这里的表达式意思是 在com.moilion_zds.aop_xml包下的任意类和子类中的任意方法，方法参数个数不限，返回值类型不限-->
    <aop:pointcut id="myPointcut" expression="execution(* com.moilion_zds.aop_xml..*.*(..))"/>
    <!-- 声明切面类 -->
    <aop:aspect ref="myAspect">
        <aop:before method="before" pointcut-ref="myPointcut"/>
        <aop:around method="around" pointcut-ref="myPointcut"/>
        <aop:after-returning method="afterReturning" pointcut-ref="myPointcut"/>
        <aop:after-throwing method="afterThrowing" pointcut-ref="myPointcut"/>
        <aop:after method="after" pointcut-ref="myPointcut"/>
    </aop:aspect>
</aop:config>
```

#### 注解配置AOP

定义一个切面类，将方法使用注解标注

```java
@Component("myAspect")
@Aspect  //标志着该类为切片类 相当于xml配置中的 <aop:config>
public class MyAspect {
    //前置增强
    @Before("myPointcut()")   //两种方法引入
    //@Before("MyAspect.myPointcut()")
    public void before(){
        System.out.println("前置增强");
    }
    //后置增强
    @AfterReturning("myPointcut()")
    public void afterReturning(){
        System.out.println("后置增强");
    }
    //环绕增强  要传入一个切点
    @Around("myPointcut()")
    public Object around(ProceedingJoinPoint joinPoint) throws Throwable {
        System.out.println("环绕前增强");
        Object obj = joinPoint.proceed();
        System.out.println("环绕后增强");
        //如果目标方法有返回值 这里就必须要返回 否则方法取不到返回值
        return obj;
    }
    //异常抛出增强  当被增强的方法中出现异常的时候 执行
    @AfterThrowing("myPointcut()")
    public void afterThrowing(){
        System.out.println("异常抛出增强");
    }
    //最终增强  不管是否出现异常 都会执行 类似于finally
    @After("myPointcut()")
    public void after(){
        System.out.println("最终增强");
    }

    //抽取切点表达式  
    //这个表达式意思是：
    @Pointcut("execution(* com.moilion_zds.app_anno..*.*(..))")
    public void myPointcut(){}
}
```

```java
<!-- 注解配置AOP -->
<!-- 扫描页面上的注解 -->
<context:component-scan base-package="com.moilion_zds.app_anno"/>
<!-- 配置aop自动代理 -->
<aop:aspectj-autoproxy/>
```

### 3、声明式事务管理

#### xml方式配置：

平台事务管理器：

```java
<!-- 平台事务管理器 -->
<bean id="dataSourceTransactionManager" class="org.springframework.jdbc.datasource.DataSourceTransactionManager">
    <property name="dataSource" ref="druidDataSource"/>
</bean>
```

增强：

```java
<!-- 增强 -->
<tx:advice id="txAdvice" transaction-manager="dataSourceTransactionManager">
    <tx:attributes>
    	<!-- 配置增强的方法名 -->
        <tx:method name="*"/>
    </tx:attributes>
</tx:advice>
```

织入：

```java
<!-- 织入 -->
<aop:config>
    <!-- 声明式事务 导入增强方法 -->
    <aop:advisor advice-ref="txAdvice" 
    pointcut="execution(* com.moilion_zds.service..*.*(..))"/>
</aop:config>
```

#### 注解方式配置：

在service业务层中，在需要事务管理的方法上添加注解@Transactional() 括号中可以配置参数

```java
@Service("userService")
public class UserServiceImpl implements UserService {

    @Resource(name = "userDao")
    private UserDao userDao;

    /**
     * @param outPeople  转出的人
     * @param inPeople   收钱的人
     * @param money      转账金额
     */
    @Override
    @Transactional()
    public void Transfer(String outPeople, String inPeople, Double money) {
        //转出
        userDao.out(outPeople,money);
        //int i=1/0;
        //收钱
        userDao.in(inPeople,money);
    }
}
```

```xml
<!--创建事务管理器  实际上就是通知/增强方法 -->
<bean id="transaction" class="org.springframework.jdbc.datasource.DataSourceTransactionManager">
    <property name="dataSource" ref="druidDataSource"/>
</bean>
<!-- 使用注解配置事务 -->
<!-- 事务注解驱动 -->
<tx:annotation-driven transaction-manager="transaction"/>

<!-- 开启aop自动代理 -->
<aop:aspectj-autoproxy/>
```

## Spring的三大核心

**1、IOC控制反转：** 就是把对象实例化的工作交给spring容器来完成，spring帮我们负责销毁对象，掌控对象的生命周期，在需要使用对象的时候直接向spring申请即可。通过反射创建对象的实例。

**2、DI依赖注入：**spring 使用 javaBean 对象的 set 方法或者带参数的构造方法为我们在创建所需对象时将其属性自动设置所需要的值的过程，即：将值通过配置的方式为变量初始化。和控制反转是同一个概念的不同角度的描述

有三种注入方式：构造器注入、setter方法注入、根据注解注入

**3、AOP面向切面：**

实现原理：动态代理（利用反射和动态编译将代理模式变成动态的）

​		**JDK的动态代理：**JDK返回动态代理的类，是基于接口生成的代理对象，代理对象实现了被代理对象的所有接口，代理对象和被代理对象之间是兄弟关系。

​		**cglib动态代理：**CGLibProxy返回的动态代理的类，是基于继承生成的代理对象，代理对象继承了被代理对象，代理对象和被代理对象之间是父子关系。

## SpringMVC

### SpringMVC的执行流程

![Snipaste_2019-08-13_13-50-21](C:\Users\Administrator\Desktop\截图\Snipaste_2019-08-13_13-50-21.png)

![Snipaste_2019-08-13_13-47-45](C:\Users\Administrator\Desktop\截图\Snipaste_2019-08-13_13-47-45.png)

![Snipaste_2019-08-13_13-47-13](C:\Users\Administrator\Desktop\截图\Snipaste_2019-08-13_13-47-13.png)

### SpringMVC中的常用注解：

1、**@Controller**：

​		@Controller 用于标记在一个类上，使用它标记的类就是一个SpringMVC Controller 对象。分发处理器将会扫描使用了该注解的类的方法，并检测该方法是否使用了@RequestMapping 注解。@Controller 只是定义了一个控制器类，而使用@RequestMapping 注解的方法才是真正处理请求的处理器。

2、**@RequestMapping**：

​		RequestMapping是一个用来处理请求地址映射的注解，可用于类或方法上。用于类上，表示类中的所有响应请求的方法都是以该地址作为父路径。

3、**@Resource和@Autowired**：

​		@Resource和@Autowired都是做bean的注入时使用，其实@Resource并不是Spring的注解，

​		它的包是javax.annotation.Resource，是JDK提供的注解，需要导入，但是Spring支持该注解的注入。

​		@Resource只能按照类型来注入对象，如果有多个同一类型的bean，那么就会出错，这是就需要配合@

4、**@ResponseBody**：可以设置方法返回类型为参数

5、**@RequestBody**：将引用类型数据，封装到方法的参数中

6、 **@PathVariable ：**匹配restful风格url中的数据

## SpringMVC的xml文件配置

### 1、扫包(只扫contorller包)：

```java
<!-- 扫包 只扫controller包 -->
<context:component-scan base-package="com.moilion_zds.controller"/>
```

### 2、内部资源视图解析器：

```java
<!-- 内部资源视图解析器 -->
<bean id="viewResolver" class="org.springframework.web.servlet.view.InternalResourceViewResolver">
    <!-- 前缀 -->
    <property name="prefix" value="/"/>
    <!-- 后缀 -->
    <property name="suffix" value=".jsp"/>
</bean>
```

### 3、注解驱动：

(可以将响应的实体类或集合转换成json字符串)

```java
<!-- 注解驱动 -->
<mvc:annotation-driven/>
```

```java
<!-- 使用请求映射处理器适配器来将响应的实体类转换成json字符串 -->
<!--<bean id="mappingHandlerAdapter" class="org.springframework.web.servlet.mvc.method.annotation.RequestMappingHandlerAdapter">
    <property name="messageConverters">
        <list>
            <bean class="org.springframework.http.converter.json.MappingJackson2HttpMessageConverter"/>
        </list>
    </property>
</bean>-->
```

### 4、静态资源访问：

(因为前端控制器会拦截除了jsp以外的所有请求，导致访问不到静态资源，可以使用默认的servlet处理器)

```java
<!-- 开启静态资源访问 -->
<mvc:default-servlet-handler/>
```

```java
<!-- 方法二：开启静态资源访问 -->
<!--<mvc:resources mapping="/js/**" location="/js/"/>-->
```

### 5、异常解析器：

springMVC提供的简单映射异常解析器：

```java
<!-- 简单映射异常解析器 -->
<!--<bean id="simpleMappingExceptionResolver" class="org.springframework.web.servlet.handler.SimpleMappingExceptionResolver">
    &lt;!&ndash; 默认错误页面 &ndash;&gt;
    <property name="defaultErrorView" value="error"/>
</bean>-->
```

自定义异常解析器

```java
/**
 *  自定义异常解析器
 *  需要在spring-mvc配置文件中配置
 */
public class MyExceptionResolver implements HandlerExceptionResolver {
    public ModelAndView resolveException(HttpServletRequest httpServletRequest, HttpServletResponse httpServletResponse, Object o, Exception e) {
        ModelAndView modelAndView=new ModelAndView();
        if (e instanceof NullPointerException){
            modelAndView.addObject("message","空指针异常");
        }else if (e instanceof ClassCastException){
            modelAndView.addObject("message","类型转换异常");
        }else if (e instanceof ClassNotFoundException){
            modelAndView.addObject("message","类找不到异常");
        }
        modelAndView.setViewName("error");
        return modelAndView;
    }
}
```

```java
<!--配置自定义异常解析器-->
<bean id="myExceptionResolver" class="com.moilion_zds.exceptionresolver.MyExceptionResolver"/>
```

### 6、自定义拦截器：

```java
/**
 * 自定义拦截器 实现HandlerInterceptor接口
 * 配置在spring-mvc配置文件中
 */
public class UserInterceptor implements HandlerInterceptor {
    //被拦截方法执行器 执行
    public boolean preHandle(HttpServletRequest request, HttpServletResponse response, Object handler) throws Exception {
        System.out.println("被拦截方法执行前拦截");
        //返回true代表放行
        return true;
    }
    //被拦截方法执行后 但未返回视图时执行
    public void postHandle(HttpServletRequest request, HttpServletResponse response, Object handler, ModelAndView modelAndView) throws Exception {
        System.out.println("被拦截方法执行后，但未返回视图拦截");
    }
    //被拦截方法全部执行完毕后执行
    public void afterCompletion(HttpServletRequest request, HttpServletResponse response, Object handler, Exception ex) throws Exception {
        System.out.println("被拦截方法执行完毕");
    }
}
```

```java
<!--配置自定义拦截器 -->
<mvc:interceptors>
    <mvc:interceptor>
        <!-- 拦截路径 表示拦截所有 -->
        <mvc:mapping path="/**"/>
        <!-- 不拦截的路径 -->
        <mvc:exclude-mapping path="/index.jsp"/>
        <bean class="com.moilion_zds.interceptor.UserInterceptor"/>
    </mvc:interceptor>
</mvc:interceptors>
```

### 7、文件上传解析器：

1、表单的提交类型必须是多部分表单  enctype="multipart/form-data"

```html
<!-- 上传单个文件 -->
<form action="${pageContext.request.contextPath}/upload/file01" method="post" enctype="multipart/form-data">
    账号：<input type="text" name="username"><br>
    文件：<input type="file" name="UpLoadFile"><br>
    <input type="submit" value="提交">
</form>
<!-- 上传多个文件 -->
<form action="${pageContext.request.contextPath}/upload/file02" method="post" enctype="multipart/form-data">
    账号：<input type="text" name="username"><br>
    文件1：<input type="file" name="UpLoadFiles"><br>
    文件2：<input type="file" name="UpLoadFiles"><br>
    <input type="submit" value="提交">
</form>
```

2、配置文件上传解析器

```java
<!-- 配置上传文件解析器 -->
<!-- 这个类的id必须是 multipartResolver -->
<bean id="multipartResolver" class="org.springframework.web.multipart.commons.CommonsMultipartResolver">
	<!-- 设置文件的最大大小 -->
    <property name="maxUploadSize" value="50000000"/>
    <!-- 设置默认字符编码 -->
    <property name="defaultEncoding" value="UTF-8"/>
</bean>
```

3、contorller层解析文件

```java
@Controller
@RequestMapping("/upload")
public class UpLoadController {

    //单个文件上传 使用MultipartFile对象
    @RequestMapping("/file01")
    //这里的MultipartFile UpLoadFile 的变量名必须和表单中<input>标签中文件的name名称相同
    public String upload01(String username, MultipartFile UpLoadFile) throws IOException {
        //定义一个状态
        String status=null;
        //判断用户名或文件是否为空
        if (username!=null&&username.length()!=0&&!UpLoadFile.isEmpty()){
            System.out.println(username);
            System.out.println(UpLoadFile.getOriginalFilename());
            String filename=System.currentTimeMillis()+UpLoadFile.getOriginalFilename();
            //System.out.println(UpLoadFile.getOriginalFilename().endsWith(".jpg"));
            File path=null;
            //判断文件是否以 .jpg和.png 结尾的
            if (UpLoadFile.getOriginalFilename().endsWith(".jpg")||UpLoadFile.getOriginalFilename().endsWith(".png")){
                //根据用户名来创建子文件夹
                path = new File("D:\\UpLoad\\image\\"+username+"\\"+filename);
                if (!path.exists()){
                    path.mkdirs();
                }
            }else {
                path = new File("D:\\UpLoad\\file\\"+username+"\\"+filename);
                if (!path.exists()){
                    path.mkdirs();
                }
            }
            UpLoadFile.transferTo(path);
            status="success";
        }else {
            status="error";
        }
        return status;
    }

    //多个文件上传  使用MultipartFile[]数组对象
    @RequestMapping("/file02")
    public ModelAndView upload02(ModelAndView modelAndView, String username, MultipartFile[] UpLoadFiles) throws IOException {
        if (username!=null&&username.length()!=0&&UpLoadFiles!=null){
            for (MultipartFile UpLoadFile : UpLoadFiles) {
                //使用时间戳来创建文件名
                String filename=System.currentTimeMillis()+UpLoadFile.getOriginalFilename();
                //根据用户名来创建子文件夹
                File path = new File("D:\\UpLoad\\"+username+"\\"+filename);
                if (!path.exists()){
                    path.mkdirs();
                }

                UpLoadFile.transferTo(path);
            }
            modelAndView.addObject("message","上传成功");
            modelAndView.setViewName("success");
        }else {
            modelAndView.addObject("message","上传失败");
            modelAndView.setViewName("error");
        }
        return modelAndView;
    }
```

### 8、自定义转换器：

```java
/** 第一步：
 * 自定义的转换器 要实现Converter接口
 * 泛型中的值是 要把什么类型的数据，转换成什么类型的数据
 * String --> Date
 * 第二步：
 * 在spring-mvc的配置文件中声明转换器
 * 第三步：
 * 在加载注解驱动时加载转换器
 */
public class DateConverter implements Converter<String, Date> {
    public Date convert(String dateStr) {
        SimpleDateFormat format=new SimpleDateFormat("yyyy-MM-dd");
        Date date=null;
        try {
            date = format.parse(dateStr);
        } catch (ParseException e) {
            e.printStackTrace();
        }
        return date;
    }
}
```

```java
<!-- 第二步： -->
<!-- 声明转换器 -->
<bean id="conversionServiceFactoryBean" class="org.springframework.context.support.ConversionServiceFactoryBean">
    <property name="converters">
        <list>
            <bean class="com.moilion_zds.converter.DateConverter"/>
        </list>
    </property>
</bean>
```

```java
<!-- 注解驱动加载自定义转换器 -->
<mvc:annotation-driven conversion-service="conversionServiceFactoryBean"/>
```

## MyBatis的xml文件配置

### MyBatis原始配置：

#### 1、MyBatis的核心配置文件

```xml-dtd
<!-- 核心配置文件头文件 -->
<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE configuration PUBLIC "-//mybatis.org//DTD Config 3.0//EN" "http://mybatis.org/dtd/mybatis-3-config.dtd">
<!-- 核心文件的配置 -->
<configuration>
	<!-- 加载数据源的配置文件 -->
    <properties resource="jdbc.properties"></properties>
    
    <!-- 设置别名 -->
    <typeAliases>
       <!-- 通过扫包来设置别名 mybatis会自动将别名设置为 类名或类名的首字母小写 比如：User或user -->
        <package name="com.moilion_zds.domain"/>
    </typeAliases>
    
    <!-- 类型处理器 -->
    <!--<typeHandlers>
    	<!-- 加载自定义的类型处理器 -->
        <typeHandler handler="com.moilion_zds.typehandler.DateTypeHandler"/>
    </typeHandlers>-->
    
    <!-- 使用插件 -->
    <plugins>
        <!-- 使用分页插件 -->
        <plugin interceptor="com.github.pagehelper.PageInterceptor">
            <!-- 在低版本的pagehelper中需要指定方言 新版本中不需要-->
            <!--<property name="dialect" value="mysql"/>-->
        </plugin>
    </plugins>
    
    <!-- 配置环境 数据源 -->
    <environments default="development">
        <environment id="development">
            <!-- 设置事务管理类型 -->
            <transactionManager type="JDBC"></transactionManager>
            <!-- 数据源的技术 -->
            <dataSource type="POOLED">
                <property name="driver" value="${jdbc.driver}"/>
                <property name="url" value="${jdbc.url}"/>
                <property name="username" value="${jdbc.username}"/>
                <property name="password" value="${jdbc.password}"/>
            </dataSource>
        </environment>
    </environments>
    
    <!-- 使用扫包 加载mapper映射文件 -->
    <mappers>
        <package name="com.moilion_zds.mapper"/>
    </mappers>
</configuration>
```

自定的类型处理器：

```java
/**
 *  可以继承BaseTypeHandler抽象类或者实现TypeHandler接口
 */
public class DateTypeHandler extends BaseTypeHandler<Date> {

    //将存入数据库的数据进行类型转换  将date类型数据转为long类型存入数据库
    public void setNonNullParameter(PreparedStatement preparedStatement, int i, Date date, JdbcType jdbcType) throws SQLException {
        long time = date.getTime();
        preparedStatement.setLong(i,time);
    }

    //将从数据库取出的数据进行类型转换  取出来转换为date类型
    public Date getNullableResult(ResultSet resultSet, String columnName) throws SQLException {
        long time = resultSet.getLong(columnName);
        Date date = new Date(time);
        return date;
    }

    //将从数据库取出的数据进行类型转换
    public Date getNullableResult(ResultSet resultSet, int columnIndex) throws SQLException {
        long time = resultSet.getLong(columnIndex);
        Date date = new Date(time);
        return date;
    }

    //将从数据库取出的数据进行类型转换
    public Date getNullableResult(CallableStatement callableStatement, int columnIndex) throws SQLException {
        long time = callableStatement.getLong(columnIndex);
        Date date = new Date(time);
        return date;
    }
}
```

#### 2、mapper映射配置文件：

```xml-dtd
<!-- 映射文件头文件 -->
<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE mapper PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN" "http://mybatis.org/dtd/mybatis-3-mapper.dtd">
<!-- mapper映射文件的配置 -->
<mapper namespace="com.moilion_zds.mapper.UserMapper">
    <!-- 抽取重复的sql片段 -->
    <sql id="sql">
        select * from user
    </sql>

    <!-- 手动配置映射关系 -->
    <resultMap id="userMap" type="user">
        <id property="id" column="id"/>
        <result property="username" column="username"/>
        <result property="password" column="password"/>
        <result property="birthday" column="birthday"/>
    </resultMap>

    <select id="selectAll" resultMap="userMap">
        <!-- 引入sql片段 -->
        <include refid="sql"></include>
    </select>
</mapper>
```

### MyBatis整合Spring的xml配置：

#### 1、MyBatis核心配置文件：

```xml-dtd
<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE configuration PUBLIC "-//mybatis.org//DTD Config 3.0//EN" "http://mybatis.org/dtd/mybatis-3-config.dtd">

<configuration>
    <!-- 设置别名 -->
    <typeAliases>
       <!-- 通过扫包来设置别名 mybatis会自动将别名设置为 类名或类名的首字母小写 比如：User或user -->
        <package name="com.moilion_zds.domain"/>
    </typeAliases>

    <!-- 类型处理器 -->
    <!--<typeHandlers>
        <typeHandler handler="com.moilion_zds.typehandler.DateTypeHandler"/>
    </typeHandlers>-->

    <!-- 使用插件 -->
    <plugins>
        <!-- 使用分页插件 -->
        <plugin interceptor="com.github.pagehelper.PageInterceptor">
            <!-- 在低版本的pagehelper中需要指定方言 新版本中不需要-->
            <!--<property name="dialect" value="mysql"/>-->
        </plugin>
    </plugins>

</configuration>
```

#### 2、mapper映射文件：

```xml-dtd
<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE mapper PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN" "http://mybatis.org/dtd/mybatis-3-mapper.dtd">

<mapper namespace="com.moilion_zds.mapper.UserMapper">
    <!-- 抽取重复的sql片段 -->
    <sql id="sql">
        select * from user
    </sql>

    <!-- 手动配置映射关系 -->
    <resultMap id="userMap" type="user">
        <id property="id" column="id"/>
        <result property="username" column="username"/>
        <result property="password" column="password"/>
        <result property="birthday" column="birthday"/>
    </resultMap>

    <select id="selectAll" resultMap="userMap">
        <!-- 引入sql片段 -->
        <include refid="sql"></include>
    </select>

</mapper>
```

#### 3、Spring的xml文件中配置：

```java
<!-- spring整合mybatis -->
    
<!-- 创建SqlSession工厂对象 将工厂对象交给spring容器管理 -->
<bean id="sqlSessionFactory" class="org.mybatis.spring.SqlSessionFactoryBean">
    <!-- 向mybatis中注入数据源 -->
    <property name="dataSource" ref="druidDataSource"/>
    <!-- 加载mybatis核心配置文件 -->
    <property name="configLocation" value="classpath:sqlMapConfig.xml"/>
</bean>

<!-- 扫描mapper映射文件 创建这些接口的实现类 交由spring容器管理 -->
<bean id="scannerConfigurer" class="org.mybatis.spring.mapper.MapperScannerConfigurer">
	<!-- 扫包 -->
    <property name="basePackage" value="com.moilion_zds.mapper"/>
</bean>
```

### 注意事项：

```java
映射文件的路径和持久层的路径必须一样

映射文件名和接口名必须一样

比如：

	接口全限定名：com.moilion_zds.mapper.UserMapper

	映射文件全限定名：com.moilion_zds.mapper.UserMapper.xml
```

## SSM中需要的jar包

```xml
<dependencies>

    <!-- spring相关的jar包 -->
    <!-- spring的上下文jar包 -->
    <dependency>
        <groupId>org.springframework</groupId>
        <artifactId>spring-context</artifactId>
        <version>5.0.5.RELEASE</version>
    </dependency>
    <!-- springMVC的web jar包 -->
    <dependency>
        <groupId>org.springframework</groupId>
        <artifactId>spring-web</artifactId>
        <version>5.0.5.RELEASE</version>
    </dependency>
    <!-- springMVC的webmvc jar包 -->
    <dependency>
        <groupId>org.springframework</groupId>
        <artifactId>spring-webmvc</artifactId>
        <version>5.0.5.RELEASE</version>
    </dependency>
    <!-- spring的事务jar包 -->
    <dependency>
        <groupId>org.springframework</groupId>
        <artifactId>spring-tx</artifactId>
        <version>5.0.5.RELEASE</version>
    </dependency>
    <!-- spring整合junit测试的jar包 -->
    <dependency>
            <groupId>org.springframework</groupId>
            <artifactId>spring-test</artifactId>
            <version>5.0.5.RELEASE</version>
    </dependency>
    <!-- junit测试的jar包 整合junit测试需要-->
    <dependency>
            <groupId>junit</groupId>
            <artifactId>junit</artifactId>
            <version>4.12</version>
    </dependency>
    
    <!-- jdbc模板依赖 在使用事务时需要用上 -->
    <dependency>
        <groupId>org.springframework</groupId>
        <artifactId>spring-jdbc</artifactId>
        <version>5.0.5.RELEASE</version>
    </dependency>
  
    <!-- aspectj依赖 第三方aop依赖 -->
    <!-- 导入第三方aop依赖 -->
    <dependency>
        <groupId>org.aspectj</groupId>
        <artifactId>aspectjweaver</artifactId>
        <version>1.8.4</version>
    </dependency>
    
   <!-- 数据库连接驱动jar包 -->
    <dependency>
        <groupId>mysql</groupId>
        <artifactId>mysql-connector-java</artifactId>
        <version>8.0.16</version>
    </dependency>
    <!-- mybatis jar包 -->
    <dependency>
        <groupId>org.mybatis</groupId>
        <artifactId>mybatis</artifactId>
        <version>3.4.5</version>
    </dependency>
    
    <!-- spring提供的整合mybatis的jar包 -->
    <dependency>
        <groupId>org.mybatis</groupId>
        <artifactId>mybatis-spring</artifactId>
        <version>1.3.1</version>
    </dependency>
    
    <!-- servlet和jsp jar包 -->
    <dependency>
        <groupId>javax.servlet</groupId>
        <artifactId>servlet-api</artifactId>
        <version>2.5</version>
    </dependency>
    <!-- jsp的jar包下面两个任选其一 -->
    <dependency>
        <groupId>javax.servlet.jsp</groupId>
        <artifactId>jsp-api</artifactId>
        <version>2.0</version>
    </dependency>
    <dependency>
        <groupId>javax.servlet.jsp</groupId>
        <artifactId>javax.servlet.jsp-api</artifactId>
        <version>2.2.1</version>
    </dependency>
    
    <!-- 数据源 -->
    <dependency>
        <groupId>com.alibaba</groupId>
        <artifactId>druid</artifactId>
        <version>1.0.9</version>
    </dependency>
    <dependency>
        <groupId>c3p0</groupId>
        <artifactId>c3p0</artifactId>
        <version>0.9.1.2</version>
    </dependency>
    
    <!-- 分页助手jar包 -->
    <dependency>
        <groupId>com.github.pagehelper</groupId>
        <artifactId>pagehelper</artifactId>
        <version>5.1.2</version>
    </dependency>
    <!-- 解析sql的jar包 -->
    <dependency>
        <groupId>com.github.jsqlparser</groupId>
        <artifactId>jsqlparser</artifactId>
        <version>1.0</version>
    </dependency>
    
    <!-- 导入标准标签库的jar包 -->
    <dependency>
        <groupId>jstl</groupId>
        <artifactId>jstl</artifactId>
        <version>1.2</version>
    </dependency>
    
    <!-- 文件上传需要的jar包 -->
    <dependency>
        <groupId>commons-fileupload</groupId>
        <artifactId>commons-fileupload</artifactId>
        <version>1.3.1</version>
    </dependency>
    <dependency>
        <groupId>commons-io</groupId>
        <artifactId>commons-io</artifactId>
        <version>2.6</version>
    </dependency>
    <dependency>
        <groupId>commons-logging</groupId>
        <artifactId>commons-logging</artifactId>
        <version>1.2</version>
    </dependency>
    
    <!-- json转换需要的jackson jar包 -->
    <dependency>
        <groupId>com.fasterxml.jackson.core</groupId>
        <artifactId>jackson-core</artifactId>
        <version>2.9.0</version>
    </dependency>
    <dependency>
        <groupId>com.fasterxml.jackson.core</groupId>
        <artifactId>jackson-databind</artifactId>
        <version>2.9.0</version>
    </dependency>
    <dependency>
        <groupId>com.fasterxml.jackson.core</groupId>
        <artifactId>jackson-annotations</artifactId>
        <version>2.9.0</version>
    </dependency>
    
    <!-- 日志jar包 -->
    <dependency>
        <groupId>log4j</groupId>
        <artifactId>log4j</artifactId>
        <version>1.2.17</version>
    </dependency>
    
</dependencies>
```

## 需要的properties配置文件

jdbc.properties:

```properties
#Oracle数据库连接信息
jdbc.oracle.driver=oracle.jdbc.driver.OracleDriver
jdbc.oracle.url=jdbc:oracle:thin:@192.168.80.88:1521:orcl
jdbc.oracle.username=moilion_zds
jdbc.oracle.password=123456
#MySQL数据库连接信息
jdbc.mysql.driver=com.mysql.cj.jdbc.Driver
jdbc.mysql.url=jdbc:mysql://localhost:3306/mybatis?serverTimezone = UTC
jdbc.mysql.username=root
jdbc.mysql.password=123456
```

log4j.properties:

```properties
### direct log messages to stdout ###
log4j.appender.stdout=org.apache.log4j.ConsoleAppender
log4j.appender.stdout.Target=System.out
log4j.appender.stdout.layout=org.apache.log4j.PatternLayout
log4j.appender.stdout.layout.ConversionPattern=%d{ABSOLUTE} %5p %c{1}:%L - %m%n

### direct messages to file mylog.log ###
log4j.appender.file=org.apache.log4j.FileAppender
log4j.appender.file.File=c:/mylog.log
log4j.appender.file.layout=org.apache.log4j.PatternLayout
log4j.appender.file.layout.ConversionPattern=%d{ABSOLUTE} %5p %c{1}:%L - %m%n

### set log levels - for more verbose logging change 'info' to 'debug' ###

log4j.rootLogger=debug, stdout
```

## SpringSecurity(安全认证框架)

### 核心配置文件

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:security="http://www.springframework.org/schema/security"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xsi:schemaLocation="http://www.springframework.org/schema/beans
    http://www.springframework.org/schema/beans/spring-beans.xsd
    http://www.springframework.org/schema/security
    http://www.springframework.org/schema/security/spring-security.xsd">

<!--指定放行资源-->
<security:http pattern="/login.jsp" security="none"></security:http>

    <!--对http请求进行权限认证
    	auto-config : 是否开启自动配置
    	use-expressions : 是否使用表达式
    -->
<security:http auto-config="true" use-expressions="false">
    
    <!--配置拦截路径
        pattern : 拦截路径   /* 拦截一级目录   /** 拦截目录以及其子目录下所有路径   /user/findAll
        access  : 路径需要什么样的角色可以访问 ,必须以ROLE_开头  ROLE_*
    -->
    <security:intercept-url pattern="/**" access="ROLE_USER,ROLE_ADMIN"></security:intercept-url>
        <!--配置使用自定义页面
            login-page : 指定登录页面
            login-processing-url : 指定登录路径  默认 :  /login
            username-parameter : 指定登录用户名参数名称  默认: username
            password-parameter : 指定登录密码参数名称  默认 : password
            default-target-url : 指定登录成功默认跳转页面
            authentication-success-forward-url : 认证成功之后的跳转路径
            authentication-failure-forward-url : 认证失败之后的跳转路径
        -->
        <security:form-login
                login-page="/login.jsp"
                login-processing-url="/login"
                username-parameter="username"
                password-parameter="password"
                default-target-url="/main.jsp"
                authentication-success-forward-url="/main.jsp"
                authentication-failure-forward-url="/login.jsp">
        </security:form-login>
		
        <!--禁用跨服务器资源拦截-->
        <security:csrf disabled="true"></security:csrf>

        <!--配置退出登录
            logout-url : 配置退出登录的路径  默认 : logout
            logout-success-url : 退出登录成功跳转的地址
            invalidate-session  : 是否销毁session
        -->
        <security:logout 
               logout-url="/logout" 
               logout-success-url="/login.jsp" 
               invalidate-session="true">
        </security:logout>
	</security:http>

<!--配置认证管理器-->
<security:authentication-manager>
    <!-- 这里的user-service-ref 引用的是自定义实现了UserDetailsService接口的类 -->
    <security:authentication-provider user-service-ref="userService">
        <!--配置密码加密器-->
       <security:password-encoder ref="passwordEncoder"></security:password-encoder>
    </security:authentication-provider>
</security:authentication-manager>
    
<!--密码加密器  这个NoOpPasswordEncoder表示的是不加密使用明文-->
<bean id="passwordEncoder" 		      class="org.springframework.security.crypto.password.NoOpPasswordEncoder">
</bean>
</beans>
```

### 需要实现类

```java

/**
 * 接口
 * 如果要使用SpringSecurity的安全 认证框架  那么自定义的接口就要继承UserDetailsService接口
 */
public interface UserService extends UserDetailsService {
    
}
```

```java
/**
* 实现类
*/
@Service("userService")
public class UserServiceImpl implements UserService {
	//从容器中取出
    @Autowired
    private UserInfoMapper userInfoMapper;

    @Override
    public UserDetails loadUserByUsername(String username) throws UsernameNotFoundException {
        //通过用户名查询出userInfo用户信息对象
        UserInfo userInfo = userInfoMapper.selectByUsername(username);

        //健壮性判断
        if (userInfo==null){
            return null;
        }

        //使用springSecurity安全认证框架提供的user对象  
        //将userInfo对象封装成一个UserDetails对象 ，然后返回
        /**
         *  方法参数解析：
         *  username  用户名
         *  password  密码
         *  boolean enabled  是否开启  true 可用  false 不可用
         *  accountNonExpired 账户是否过期   true 未过期   false 已过期
         *  credentialsNonExpired 账户认证是否过期    true 未过期    false 已过期
         *  accountNonLocked    账户是否被锁定   true 未锁定    false 已锁定
         *  authorities  用户拥有的权限列表
         */
        User user=new User(userInfo.getUsername(),userInfo.getPassword(),(userInfo.getStatus()==1?true:false),true,true,true,getAuto(userInfo.getRoles()));
        return user;
    }

    /**
     * 根据用户角色列表,获取用户权限列表
     */
    public List<SimpleGrantedAuthority> getAuto(List<Role>roles){
        ArrayList<SimpleGrantedAuthority> list = new ArrayList<>();
        for (Role role : roles) {
            list.add(new SimpleGrantedAuthority("ROLE_"+role.getRoleName()));
        }
        return list;
    }
}
```

### SpringSecurity的核心功能：

- 认证（你是谁）
- 授权（你能干什么）
- 攻击防护（防止伪造身份）

## SpringBoot：

### springboot原理：