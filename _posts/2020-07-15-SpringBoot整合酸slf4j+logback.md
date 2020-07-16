---
layout:     post
title:      "Spring boot配置slf4j+logback日志框架"
header-img: "img/root/home-bg-o.jpg"
date:       2020-07-15
author:     "xxq"
tags:
    - SpringBoot
---

# Spring boot配置slf4j+logback日志框架

## 前言
对于一个web项目来说，日志框架是必不可少的，日志的记录可以帮助我们在开发以及维护过程中快速的定位错误。相信很多人听说过slf4j,log4j,logback,JDK Logging等跟日志框架有关的词语，所以这里也简单介绍下他们之间的关系。

### 关系
首先slf4j可以理解为规则的制定者，是一个抽象层，定义了日志相关的接口。log4j,logback,JDK Logging都是slf4j的实现层，只是出处不同，当然使用起来也就各有千秋，这里放一张网上的图更明了的解释了他们之间的关系：
![日志](https://gitee.com/dahongcha/image/raw/master/img/20200716185009.png)
可以看到logback是直接实现的slf4j，而其他的中间还有一个适配层，至于原因也很简单，因为logback和slf4j的作者是一个人。
## 在Spring boot中使用slf4j+logback日志框架
### 添加配置文件

在Spring boot使用是非常方便的，不需要我们有什么额外的配置，因为Spring boot默认支持的就是slf4j+logback的日志框架，想要灵活的定制日志策略，只需要我们在src/main/resources下添加配置文件即可，只是默认情况下配置文件的命名需要符合以下规则：
1. logback.xml
2. logback-spring.xml
> 其中logback-spring.xml是官方推荐的，并且只有使用这种命名规则，才可以配置不同环境使用不同的日志策略这一功能。

### 配置文件详解

首先介绍配置文件的关键节点：
#### 框架介绍

`<configuration>`：根节点,有三个属性：
1. scan：当配置文件发生修改时，是否重新加载该配置文件，两个可选值true or false，默认为true。
2. scanPeriod：检测配置文件是否修改的时间周期，当没有给出时间单位时默认单位为毫秒，默认值为一分钟,需要注意的是这个属性只有在scan属性值为true时才生效。
3. debug：是否打印loback内部日志信息，两个可选值true or false，默认为false。

根节点`<configuration>`有三个重要的子节点,正是这三个子节点的不同组合构成配置文件的基本框架，使得logback.xml配置文件具备很强的灵活性:
* `<appender>`：定义日志策略的节点，一个日志策略对应一个`<appender>`，一个配置文件中可以有零个或者多该节点，但一个配置文件如果没有定义至少一个`<appender>`，虽然程序不会报错，但就不会有任何的日志信息输出，也失去了意义，该节点有两个必要的属性：
    1. name：指定该节点的名称，方便之后的引用。
    2. class：指定该节点的全限定名，所谓的全限定名就是定义该节点为哪种类型的日志策略，比如我们需要将日志输出到控制台，就需要指定class的值为ch.qos.logback.core.ConsoleAppender;需要将日志输出到文件，则class的值为ch.qos.logback.core.FileAppender等，想要了解所有的appender类型，可以查阅官方文档
    
* `<logger>`：用来设置某个包或者类的日志打印级别，并且可以引用`<appender>`绑定日志策略，有三个属性：
    1. name：用来指定受此`<logger>`约束的包或者类。
    2. level：可选属性，用来指定日志的输出级别，如果不设置，那么当前`<logger>`会继承上级的级别。
    3. additivity：是否向上级传递输出信息，两个可选值true or false，默认为true。
> 在该节点内可以添加子节点`<appender-ref>`,该节点有一个必填的属性ref,值为我们定义的`<appender>`节点的name属性的值。

* `<root>`：根`<logger>`一个特殊的`<logger>`，即默认name属性为root的`<logger>`，因为是根`<logger>`，所以不存在向上传递一说，故没有additivity属性,所以该节点只有一个level属性。

介绍了根节点的三个主要的子节点，下面再介绍两个不那么重要但可以了解的子节点：
* `<contextName>`：设置上下文名称，每个`<logger>`都关联到`<logger>`上下文，默认上下文名称为default,但可以使用设置成其他名字，用于区分不同应用程序的记录,一旦设置，不能修改,可以通过 %contextName 来打印日志上下文名称，一般来说我们不用这个属性，可有可无。
* `<property>`：用来定义变量的节点，定义变量后，可以使${}来使用变量,两个属性,当定义了多个`<appender>`的时候还是很有用的：
    1. name：变量名
    2. value：变量值

好了，介绍了上边的节点我们就已经可以搭建一个简单的配置文件框架了，如下：

```xml
<?xml version="1.0" encoding="UTF-8"?>
    <!-- 一般根节点不需要写属性了，使用默认的就好 -->
    <configuration>
    
        <contextName>demo</contextName>
        
        <!-- 该变量代表日志文件存放的目录名 -->
        <property name="log.dir" value="logs"/>
        <!-- 该变量代表日志文件名 -->
        <property name="log.appname" value="eran"/>
        
        <!--定义一个将日志输出到控制台的appender，名称为STDOUT -->
        <appender name="STDOUT" class="ch.qos.logback.core.ConsoleAppender">
            <!-- 内容待定 -->
        </appender>
        
        <!--定义一个将日志输出到文件的appender，名称为FILE_LOG -->
        <appender name="FILE_LOG" class="ch.qos.logback.core.FileAppender">
            <!-- 内容待定 -->
        </appender>
      
        <!-- 指定com.demo包下的日志打印级别为INFO，但是由于没有引用appender，所以该logger不会打印日志信息，日志信息向上传递 -->
        <logger name="com.demo" level="INFO"/>
      
        <!-- 指定最基础的日志输出级别为DEBUG，并且绑定了名为STDOUT的appender，表示将日志信息输出到控制台 -->
        <root level="debug">
            <appender-ref ref="STDOUT" />
        </root>
    </configuration>
```

上面搭建了框架，定义了一个输出到控制台的ConsoleAppender以及输出到文件的FileAppender，下面来细说这两个最基本的日志策略，并介绍最常用的滚动文件策略的RollingFileAppender，这三种类型的日志策略足够我们的日常使用。
输出到控制台的ConsoleAppender的介绍：

先给出一个demo：

```xml
    <!--定义一个将日志输出到控制台的appender，名称为STDOUT -->
    <appender name="STDOUT" class="ch.qos.logback.core.ConsoleAppender">
        <encoder>
            <pattern>[Eran]%date [%thread %line] %level >> %msg >> %logger{10}%n</pattern>
        </encoder>
    </appender>
```

ConsoleAppender的功能是将日志输出到控制台，有一个`<encoder>`节点用来指定日志的输出格式，在较早以前的版本还有一个`<layout>`节点也是相同的作用，但是官方推荐使用encoder节点，所以这里我们介绍encoder节点即可。
`<encoder>`节点介绍

该节点主要做两件事：
* 把日志信息转换成字节数组
* 将字节数组写到输出流

该节点的子节点`<pattern>`作用就是定义日志的格式，即定义一条日志信息包含哪些内容，例如当前时间，在代码中的行数线程名等。需要哪些内容由我们自己定义，按照%+转换符的格式定义，下面列出常用的转换符：
* %date{}:输出时间，可以在花括号内指定时间格式，例如-%data{yyyy-MM-dd HH:mm:ss}，格式语法和java.text.SimpleDateFormat一样，可以简写为%d{}的形式，使用默认的格式时可以省略{}。
* %logger{}：日志的logger名称，可以简写为%c{},%lo{}的形式，使用默认的参数时可以省略{}，可以定义一个整形的参数来控制输出名称的长度，有下面三种情况：
    1. 不输入表示输出完整的`<logger>`名称
    2. 输入0表示只输出`<logger>`最右边点号之后的字符串
    3. 输入其他数字表示输出小数点最后边点号之前的字符数量
* %thread：产生日志的线程名，可简写为%t
* %line:当前打印日志的语句在程序中的行号，可简写为%L
* %level：日志级别,可简写为%le,%p
* %message：程序员定义的日志打印内容,可简写为%msg,%m
* %n：换行,即一条日志信息占一行
介绍了常用的转换符，我们再看看上边的例子中我们定义的格式：

```xml
<pattern>[Eran]%date [%thread %line] %level >> %msg >> %logger{10}%n</pattern>
```

日志的格式一目了然，可以看出我们在最前面加了[eran]的字符串，这里是我个人的使用习惯，一般将项目名统一展现在日志前边，而且在每个转换符之间加了空格，这更便于我们查看日志，并且使用了>>字符串来将%msg分割开来，更便于我们找到日志信息中我们关注的内容，这些东西大家可以自己按照自己的喜好来。

#### 输出到文件的FileAppender

先给出一个demo：

```xml
<!--定义一个将日志输出到文件的appender，名称为FILE_LOG -->
<appender name="FILE_LOG" class="ch.qos.logback.core.FileAppender">  
    <file>D:/test.log</file>
    <append>true</append>  
    <encoder>  
        <pattern>[Eran]%date [%thread %line] %level >> %msg >> %logger{10}%n</pattern>
    </encoder>
</appender>
```

FileAppender表示将日志输出到文件，常用几个子节点：
* `<file>`：定义文件名和路径，可以是相对路径 , 也可以是绝对路径 , 如果路径不存在则会自动创建
* `<append>`：两个值true和false，默认为true，表示每次日志输出到文件走追加在原来文件的结尾，false则表示清空现存文件
* `<encoder>`：和ConsoleAppender一样
显而易见，样例中我们的日志策略表示，每次将日志信息追加到D:/test.log的文件中。
滚动文件策略RollingFileAppender介绍

#### 按时间滚动TimeBasedRollingPolicy

demo如下：

```xml
    <appender name="ROL-FILE-LOG" class="ch.qos.logback.core.rolling.RollingFileAppender"> 
      <!--滚动策略，按照时间滚动 TimeBasedRollingPolicy-->
      <rollingPolicy class="ch.qos.logback.core.rolling.TimeBasedRollingPolicy">
          <fileNamePattern>D:/logs/test.%d{yyyy-MM-dd}.log</fileNamePattern>
          <!-- 只保留近七天的日志 -->
          <maxHistory>7</maxHistory>
          <!-- 用来指定日志文件的上限大小，那么到了这个值，就会删除旧的日志 -->
          <totalSizeCap>1GB</totalSizeCap>
      </rollingPolicy> 
      
      <encoder>
        <pattern>[Eran]%date [%thread %line] %level >> %msg >> %logger{10}%n</pattern>
      </encoder>
    </appender>
```

RollingFileAppender是非常常用的一种日志类型，表示滚动纪录文件，先将日志记录到指定文件，当符合某种条件时，将日志记录到其他文件,常用的子节点：
* `<rollingPolicy>`：滚动策略，通过属性class来指定使用什么滚动策略，最常用是按时间滚动TimeBasedRollingPolicy,即负责滚动也负责触发滚动，有以下常用子节点：
    1. `<fileNamePattern>`：指定日志的路径以及日志文件名的命名规则，一般根据日志文件名+%d{}.log来命名，这边日期的格式默认为yyyy-MM-dd表示每天生成一个文件，即按天滚动yyyy-MM，表示每个月生成一个文件，即按月滚动
    2. `<maxHistory>`：可选节点，控制保存的日志文件的最大数量，超出数量就删除旧文件，比如设置每天滚动，且`<maxHistory>` 是7，则只保存最近7天的文件，删除之前的旧文件
    3. `<encoder>`:同上
    4. `<totalSizeCap>`:这个节点表示设置所有的日志文件最多占的内存大小，当超过我们设置的值时，logback就会删除最早创建的那一个日志文件。
以上就是关于RollingFileAppender的常用介绍，上面的demo的配置也基本满足了我们按照时间滚动TimeBasedRollingPolicy生成日志的要求，下面再介绍一种常用的滚动类型SizeAndTimeBasedRollingPolicy,即按照时间和大小来滚动。
按时间和大小滚动SizeAndTimeBasedRollingPolicy

demo如下：
```xml
<appender name="ROL-SIZE-FILE-LOG" class="ch.qos.logback.core.rolling.RollingFileAppender">
    <rollingPolicy class="ch.qos.logback.core.rolling.SizeAndTimeBasedRollingPolicy">
        <fileNamePattern>D:/logs/test.%d{yyyy-MM-dd}.%i.log</fileNamePattern>
        <!-- 单个文件的最大内存 -->
        <maxFileSize>100MB</maxFileSize>
        <!-- 只保留近七天的日志 -->
        <maxHistory>7</maxHistory>
        <!-- 用来指定日志文件的上限大小，那么到了这个值，就会删除旧的日志 -->
        <totalSizeCap>1GB</totalSizeCap>
    </rollingPolicy>
    
    <encoder>
        <pattern>[Eran]%date [%thread %line] %level >> %msg >> %logger{10}%n</pattern>
    </encoder>
</appender>
```

仔细观察上边demo中的`<fileNamePattern>`会发现比TimeBasedRollingPolicy中定义的`<fileNamePattern>`多了.%i的字符，这个很关键，在SizeAndTimeBasedRollingPolicy中是必不可少的。
上边的demo中多了一个`<maxFileSize>`节点，这里介绍下，其他的节点上边已经解释过，这里就不再赘述。
`<maxFileSize>`：表示单个文件占用的最大内存大小，当某个文件超过这个值，就会触发滚动策略，产生一个新的日志文件。

### 日志过滤

#### 级别介绍

在说级别过滤之前，先介绍一下日志的级别信息：
* TRACE
* DEBUG
* INFO
* WARN
* ERROR

上述级别从上到下由低到高，我们开发测试一般输出DEBUG级别的日志，生产环境配置只输出INFO级别甚至只输出ERROR级别的日志，这个根据情况而定，很灵活。

#### 过滤节点`<filter>`介绍

过滤器通常配置在Appender中，一个Appender可以配置一个或者多个过滤器，有多个过滤器时按照配置顺序依次执行，当然也可以不配置，其实大多数情况下我们都不需要配置，但是有的情况下又必须配置，所以这里也介绍下常用的也是笔者曾经使用过的两种过率机制：级别过滤器LevelFilter和临界值过滤器ThresholdFilter。
在此之前先说下`<filter>`的概念，首先一个过滤器`<filter>`的所有返回值有三个，每个过滤器都只返回下面中的某一个值：
* DENY:日志将被过滤掉，并且不经过下一个过滤器
* NEUTRAL:日志将会到下一个过滤器继续过滤
* ACCEPT：日志被立即处理，不再进入下一个过滤器

#### 级别过滤器LevelFilter

过滤条件：只处理INFO级别的日志，格式如下：
```xml
<filter class="ch.qos.logback.classic.filter.LevelFilter">   
    <level>INFO</level>   
    <onMatch>ACCEPT</onMatch>   
    <onMismatch>DENY</onMismatch>   
</filter>
```

* `<level>`:日志级别
* `<onMatch>`：配置满足过滤条件的处理方式
* `<onMismatch>`:配置不满足过滤条件的处理方式

就如上边的demo中的配置一样，设置了级别为INFO,满足的日志返回ACCEPT即立即处理，不满足条件的日志则返回DENY即丢弃掉，这样经过这一个过滤器就只有INFO级别的日志会被打印出输出。

#### 临界值过滤器ThresholdFilter

过滤条件：只处理INFO级别之上的日志，格式如下：
```xml
<filter class="ch.qos.logback.classic.filter.ThresholdFilter">  
    <level>INFO</level>   
</filter>
```
当日志级别等于或高于临界值时，过滤器返回NEUTRAL,当日志级别低于临界值时，返回DENY。

带过滤器的`<Appender>`

下面给出一个带过滤器的`<Appender>`:
```xml
<appender name="ROL-SIZE-FILE-LOG" class="ch.qos.logback.core.rolling.RollingFileAppender">
    <rollingPolicy class="ch.qos.logback.core.rolling.SizeAndTimeBasedRollingPolicy">
        <fileNamePattern>D:/logs/test.%d{yyyy-MM-dd}.%i.log</fileNamePattern>
        <!-- 单个文件的最大内存 -->
        <maxFileSize>100MB</maxFileSize>
        <!-- 只保留近七天的日志 -->
        <maxHistory>7</maxHistory>
        <!-- 用来指定日志文件的上限大小，那么到了这个值，就会删除旧的日志 -->
        <totalSizeCap>1GB</totalSizeCap>
    </rollingPolicy>
    
    <encoder>
        <pattern>[Eran]%date [%thread %line] %level >> %msg >> %logger{10}%n</pattern>
    </encoder>
    
    <!-- 只处理INFO级别以及之上的日志 -->
    <filter class="ch.qos.logback.classic.filter.ThresholdFilter">   
        <level>INFO</level>   
    </filter>
    
    <!-- 只处理INFO级别的日志 -->
    <filter class="ch.qos.logback.classic.filter.LevelFilter">   
        <level>INFO</level>   
        <onMatch>ACCEPT</onMatch>   
        <onMismatch>DENY</onMismatch>   
    </filter>
</appender>
```

上边的demo中,我们给按时间和大小滚动SizeAndTimeBasedRollingPolicy的滚动类型加上了过滤条件。

#### 异步写入日志AsyncAppender

都知道，我们的日志语句是嵌入在程序内部，如果写入日志以及程序执行的处于一个串行的状态，那么日志的记录就必然会阻碍程序的执行，加长程序的响应时间，无疑是一种极为损耗效率的方式，所以实际的项目中我们的日志记录一般都用异步的方式来记录，这样就和主程序形成一种并行的状态，不会影响我们程序的运行，这也是我们性能调优需要注意的一个点。

> AsyncAppender并不处理日志，只是将日志缓冲到一个BlockingQueue里面去，并在内部创建一个工作线程从队列头部获取日志，之后将获取的日志循环记录到附加的其他appender上去，从而达到不阻塞主线程的效果。因此AsynAppender仅仅充当事件转发器，必须引用另一个appender来写日志。

```xml
<appender name ="ASYNC" class= "ch.qos.logback.classic.AsyncAppender">  
    <!-- 不丢失日志.默认的,如果队列的80%已满,则会丢弃TRACT、DEBUG、INFO级别的日志 -->  
    <discardingThreshold >0</discardingThreshold>  
    <!-- 更改默认的队列的深度,该值会影响性能.默认值为256 -->  
    <queueSize>512</queueSize>  
    <!-- 添加附加的appender,最多只能添加一个 -->  
    <appender-ref ref ="FILE_LOG"/>
</appender>
```

常用节点：
* `<discardingThreshold>`:默认情况下，当BlockingQueue还有20%容量，他将丢弃TRACE、DEBUG和INFO级别的日志，只保留WARN和ERROR级别的日志。为了保持所有的日志，设置该值为0。
* `<queueSize>`:BlockingQueue的最大容量，默认情况下，大小为256。
* `<appender-ref>`:添加附加的`<appender>`,最多只能添加一个

#### `<logger>`和`<root>`节点介绍

上边花费了很长的篇幅介绍了`<appender>`的相关内容，现在来详细介绍下`<logger>`节点以及`<root>`节点的相关内容。
上文已经简单介绍了`<logger>`节点的属性以及子节点，这里我们就举例来说明在logback-spring.xml文件中，该节点到底扮演怎样的角色，以及他的运行原理，看下边的demo：

首先在这里给出项目结构：

![](https://gitee.com/dahongcha/image/raw/master/img/20200716214554.png)

下面定义两个`<logger>`以及`<root>`：

```xml
<!-- logger1 -->
<logger name="com.example" level="ERROR">
    <appender-ref ref="STDOUT" />
</logger>
```

```xml
<!-- logger2 -->
<logger name="com.example.demo.controller" level="debug">
    <appender-ref ref="STDOUT" />
</logger>
```

```xml
<!-- 指定最基础的日志输出级别为DEBUG，并且绑定了名为STDOUT的appender，表示将日志信息输出到控制台 -->
<root level="INFO">
    <appender-ref ref="STDOUT" />
</root>
```

当存在多个`<logger>`时，会有父级子级的概念，日志的处理流程是先子级再父级，当然`<root>`是最高级别，怎样区分级别大小呢，根据name属性指定的包名来判断，包名级别越高则`<logger>`的级别越高，跟我们定义`<logger>`的顺序无关。
上边我们定义了logger1和logger2，很明显看出logger1是logger2的父级，以本例给出多个`<logger>`与`<root>`之间的执行流程图：

流程图看着一目了然，这里就不再赘述，只是在实际的项目中我们一般都不让`<logger>`输出日志，统一放在`<root>`节点中输出，所以一般不给`<logger>`节点添加`<appender>`，当然这个按实际需要可以灵活配置。

#### 配置profile

profile即根据不同的环境使用不同的日志策略，这里举例开发和生产环境：
```xml
    <!-- 开发环境输出到控制台 -->
    <springProfile  name="dev">
        <root level="INFO">
            <appender-ref ref="STDOUT" />
        </root>
    </springProfile>
```
```xml
    <!-- 生产环境输出到文件 -->
    <springProfile  name="prod">
        <root level="INFO">
            <appender-ref ref="FILE_LOG" />
        </root>
    </springProfile>
```

可以看到我们只需要在`<root>`节点的外边再套一层`<springProfile>`就可以了，并且指定name属性的值，在配置文件里边配置好之后,怎么启用，这里介绍两种方式：
1. 执行jar包时添加参数：
xmljava -jar xxx.jar --spring.profiles.active=prod


2. 在项目的application.properties配置文件中添加：
xmlspring.profiles.active=prod

### 整合

最后将所有的模块整合在一起形成一个完整的配置文件：
```xml
<?xml version="1.0" encoding="UTF-8"?>
<configuration>    
    <!--定义一个将日志输出到控制台的appender，名称为STDOUT -->
    <appender name="STDOUT" class="ch.qos.logback.core.ConsoleAppender">
        <encoder>
            <pattern>[%contextName]%date [%thread %line] %level >> %msg >> %logger{10}%n</pattern>
        </encoder>
    </appender> 
    
    <!--定义一个将日志输出到文件的appender，名称为FILE_LOG -->
    <appender name="FILE_LOG" class="ch.qos.logback.core.FileAppender">  
        <file>D:/test.log</file>
        <append>true</append>  
        <encoder>  
            <pattern>[Eran]%date [%thread %line] %level >> %msg >> %logger{10}%n</pattern>
        </encoder>
    </appender>  
    
    <!--  按时间滚动产生日志文件 -->
    <appender name="ROL-FILE-LOG" class="ch.qos.logback.core.rolling.RollingFileAppender"> 
      <!--滚动策略，按照时间滚动 TimeBasedRollingPolicy-->
      <rollingPolicy class="ch.qos.logback.core.rolling.TimeBasedRollingPolicy">
          <fileNamePattern>D:/logs/test.%d{yyyy-MM-dd}.log</fileNamePattern>
          <!-- 只保留近七天的日志 -->
          <maxHistory>7</maxHistory>
          <!-- 用来指定日志文件的上限大小，那么到了这个值，就会删除旧的日志 -->
          <totalSizeCap>1GB</totalSizeCap>
      </rollingPolicy> 
      
      <encoder>
        <pattern>[Eran]%date [%thread %line] %level >> %msg >> %logger{10}%n</pattern>
      </encoder>
    </appender>
    
    <!-- 按时间和文件大小滚动产生日志文件 -->
    <appender name="ROL-SIZE-FILE-LOG" class="ch.qos.logback.core.rolling.RollingFileAppender">
        <rollingPolicy class="ch.qos.logback.core.rolling.SizeAndTimeBasedRollingPolicy">
            <fileNamePattern>D:/logs/test.%d{yyyy-MM-dd}.%i.log</fileNamePattern>
            <!-- 单个文件的最大内存 -->
            <maxFileSize>100MB</maxFileSize>
            <!-- 只保留近七天的日志 -->
            <maxHistory>7</maxHistory>
            <!-- 用来指定日志文件的上限大小，那么到了这个值，就会删除旧的日志 -->
            <totalSizeCap>1GB</totalSizeCap>
        </rollingPolicy>
        
        <encoder>
            <pattern>[Eran]%date [%thread %line] %level >> %msg >> %logger{10}%n</pattern>
        </encoder>
        
        <!-- 只处理INFO级别以及之上的日志 -->
        <filter class="ch.qos.logback.classic.filter.ThresholdFilter">   
            <level>INFO</level>   
        </filter>
        
        <!-- 只处理INFO级别的日志 -->
        <filter class="ch.qos.logback.classic.filter.LevelFilter">   
            <level>INFO</level>   
            <onMatch>ACCEPT</onMatch>   
            <onMismatch>DENY</onMismatch>   
        </filter>
    </appender>
    
    <!-- 异步写入日志 -->
    <appender name ="ASYNC" class= "ch.qos.logback.classic.AsyncAppender">  
        <!-- 不丢失日志.默认的,如果队列的80%已满,则会丢弃TRACT、DEBUG、INFO级别的日志 -->  
        <discardingThreshold >0</discardingThreshold>  
        <!-- 更改默认的队列的深度,该值会影响性能.默认值为256 -->  
        <queueSize>512</queueSize>  
        <!-- 添加附加的appender,最多只能添加一个 -->  
        <appender-ref ref ="FILE_LOG"/>
    </appender>
    
    
    <!-- 指定com.demo包下的日志打印级别为DEBUG，但是由于没有引用appender，所以该logger不会打印日志信息，日志信息向上传递 -->
    <logger name="com.example" level="DEBUG"></logger>
    <!-- 这里的logger根据需要自己灵活配置 ，我这里只是给出一个demo-->
    
    <!-- 指定开发环境基础的日志输出级别为DEBUG，并且绑定了名为STDOUT的appender，表示将日志信息输出到控制台 -->
    <springProfile  name="dev">
        <root level="DEBUG">
            <appender-ref ref="STDOUT" />
        </root>
    </springProfile>
    
    <!-- 指定生产环境基础的日志输出级别为INFO，并且绑定了名为ASYNC的appender，表示将日志信息异步输出到文件 -->
    <springProfile  name="prod">
        <root level="INFO">
            <appender-ref ref="ASYNC" />
        </root>
    </springProfile>
</configuration>
```

### 代码中使用

终于到最后一步了，上边介绍了怎么配置logback-spring.xml配置文件，下面介绍怎么在项目中引入日志对象，以及怎么使用它输出日志，直接上代码：
```java

import org.slf4j.Logger;
import org.slf4j.LoggerFactory;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RequestMethod;
import org.springframework.web.bind.annotation.RestController;


@RestController
public class TestLog {

    private final static Logger log = LoggerFactory.getLogger(TestLog.class);
    
    
    @RequestMapping(value="/log",method=RequestMethod.GET)
    public void testLog() {
        log.trace("trace级别的日志");
        log.debug("debug级别日志");
        log.info("info级别日志");
        log.warn("warn级别的日志");
        log.error("error级别日志");
    }
}
```

在每一个需要使用日志对象的方法里边都要定义一次private final static Logger log = LoggerFactory.getLogger(xxx.class);其中xxx代指当前类名，如果觉得这样很麻烦，也可以通过@Slf4j注解的方式注入，但是这种方式需要添加pom依赖并且需要安装lombok插件，这里就不概述了，需要了解的朋友可以自己google。