# 牛客网社区系统项目
## 项目介绍
本项目实现了牛客网社区系统的部分功能：讨论区。这个项目的整体结构来源于牛客网，
主要使用了Springboot、Mybatis、MySQL、Redis、Kafka等工具。主要实现了用户的注册、登录、发帖、点赞、系统通知、按热度排序、搜索等功能。
另外引入了redis数据库来提升网站的整体性能，实现了用户凭证的存取、点赞关注的功能。
基于 Kafka 实现了系统通知：当用户获得点赞、评论后得到通知。
利用定时任务定期计算帖子的分数，并在页面上展现热帖排行榜。

## 目录

1. [注册登录](#1)
2. [拦截器](#2)
3. [发帖](#3)
4. [Spring事务的管理方式](#4)
5. [添加评论](#5)
6. [处理异常](#6)
7. [点赞](#7)
8. [利用kafaka实现发送系统通知功能](#8)
9. [Spring ThreadPoolTaskSchedular实现定时任务](#9)
10. [热帖排行](#10)

<span id="1"></span>
***1.注册登录***

前端的注册信息->传入controller中->传给service判断数据是否合法(主要判断传入的账号、密码是否合理，数据库中是否已经存在、验证码是否正确，验证码从session中取出判断，邮箱是否存在)，合法就注册成功，返回map给controller->controller判断map，给model加响应的信息，返回给前端控制器去判断。然后返回相应的数据。

验证码的生成：使用Kaptcha，在Controller中设置生成的验证码的路径，然后通过配置类kaptchaProducer生成验证码，把生成的验证码文字存入session以供验证，然后生成验证码文字对应的图片输出给浏览器。

登录流程：验证登录的账号是否合理，如果合理则发放凭证给客户端，页面需要记住这个登陆凭证，下次进来要去找ticket，才能说明登陆成功。登录成功状态改为0，否则为1.

<span id="2"></span>
***2.拦截器***

目的：让未登录用户不能访问某些页面
原理：在方法前标注自定义注解，拦截所有的请求，只处理带有该注解的方法。

<span id="3"></span>
***3.发帖***

采用ajax请求，实现发布帖子的功能，jQuery文件用于页面的异步交互。
异步请求：不需要等待响应，随时发送下一次请求
发帖流程：首先获取当前的用户，然后将前端输入的帖子存到数据库中，使用异步请求返回发帖的结果。

<span id="4"></span>
***4.Spring事务的管理方式***

声明式事务：声明式事务是建立在AOP之上的。其本质是对方法前后进行拦截，然后在目标方法开始之前创建或者加入一个事务，在执行完目标方法之后根据执行情况提交或者回滚事务。声明式事务最大的优点就是不需要通过编程的方式管理事务，这样就不需要在业务逻辑代码中掺杂事务管理的代码，只需在配置文件中做相关的事务规则声明(或通过基于@Transactional注解的方式)，便可以将事务规则应用到业务逻辑中。
采用ajax请求，实现发布帖子的功能，jQuery文件用于页面的异步交互。
异步请求：
编程式事务：使用TransactionTemplate或者直接使用底层的PlatformTransactionManager。对于编程式事务管理，spring推荐使用TransactionTemplate。

和编程式事务相比，声明式事务唯一不足地方是，它的最细粒度只能作用到方法级别，无法做到像编程式事务那样可以作用到代码块级别。但是即便有这样的需求，也存在很多变通的方法，比如，可以将需要进行事务管理的代码块独立为方法等等。

<span id="5"></span>
***5.添加评论***

首先通过数据层添加sql语句，实现添加评论和更新评论数量。
在业务层进行事务操作，由于添加评论和修改帖子评论数量是两个事务，因此需要进行事务管理。事务操作成功后向数据库中添加评论数据，修改帖子的评论数量。表现层将结果返回给前端界面

<span id="6"></span>
***6.处理异常***

异常是一层一层往上抛的，所以只要处理表现层就能满足需求。
springboot自动处理，把要显示的错误网页放在templates/error，自动识别错误spring统一处理：@controlleradvice
//这个组件会扫描所有组件，要做限制->这个注解只去扫描带有controller注解的那些bean   注解为：@ControllerAdvice(annotations = Controller.class)

<span id="7"></span>
***7.点赞***

点击点赞按钮：如果还没点赞，那么就点赞，如果已经点赞了，那么就取消点赞

<span id="8"></span>
***8.利用kafaka实现发送系统通知功能***

触发通知的事件：评论，点赞，关注
拼一个对象，触发事件，处理事件。面向事件编程。
Entity实体类中编写消费者和生产者
创建event包，专门处理，里面的eventproducer发送消息，eventconsumer处理消息
Eventconsumer中，把从producer那边拿到的消息进行内容和格式的判断，之后发送站内通知，先设置发送者和接受者，为了获得发送的消息的具体内容，发送的消息的内容需要拼出，创这个hashmap（content）,从event里面获取内容装进去，构造好消息，最后用messageservice塞进去给响应的controller调用eventproducer，产生通知
步骤：先创建一个event实体类，然后分别定义事件生产者和消费者，每当出现评论点赞关注等操作后，根据主题不同选择发向不同的topic，然后消费者再从消息对立中消费数据

<span id="9"></span>
***9.Spring ThreadPoolTaskSchedular实现定时任务***

JDK线程池
（1）ExecutorService
（2）ScheduledExecutorService
Spring线程池
（1）ThreadPoolTaskExecutor
（2）ThreadPoolTaskSchedular
分布式定时任务
Spring Quartz
线程池有关内容整体了解：
JDK自带了线程池ExecutorService普通线程池，ScheduledExecutorService创建定时任务。
spring框架的线程池ThreadPoolTaskExecutor普通线程池，ThreadPoolTaskSchedular创建定时任务。
JDK的线程池在分布式环境下有问题（因为它数据是存在内存的），所以我们一般用Spring Quartz（官网http://www.quartz-schedular.org）
负载均衡工具：Nginx
JDK和spring是各自为战的，不能解决分布式的部署，而Quartz数据存在DB中的，多个Quartz可以在分布式下使用：

<span id="10"></span>
***10.热帖排行***

通常而言，启动一个定时任务进行分数计算，合理的方式：前面的热门帖子保持一定的时间不变，等过段时间定时任务算完在更新。这里为了方便测试，定时任务设置为5min更新一次。
结果的展现：排序的不同
定时计算没必要把所有的帖子都算一遍，把分数变化的帖子存进redis缓存中。
等到定时到了要计算的时候把缓存中的帖子拿出来计算，这样减轻服务器的负担。
![image](https://user-images.githubusercontent.com/52394792/116857008-a9ab9700-ac2e-11eb-8584-ec5665984e92.png)


## 该项目的用途
本项目用于夯实基础知识，旨在通过本项目加深对SSM框架的理解。同时，通过开发小小的社区网站，熟悉项目开发流程。最终目的--》秋招找一个好工作！！！
