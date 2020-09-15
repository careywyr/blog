前一篇博客中介绍了spring boot的基础知识以及如何搭建最简单的spring boot项目，现在我们来讲一下spring boot的配置文件使用方法。
我们新建完spring boot项目后会发现在resources目录下会有一个application.properties的配置文件，spring boot启动时会默认读取这个配置文件里面的内容，因此文件名时不可以随便乱改的。但这里我推荐大家使用yml格式的配置文件，使用起来比较简洁方便。因此这里我就把原来的.properties文件换成了.yml的配置文件，但要注意yml文件是有严格的格式规范的，要注意空格和缩进的使用。
![image](https://leafw-blog-pic.oss-cn-hangzhou.aliyuncs.com/b1.png)
## 一、多环境配置
由于平常项目开发会有多个环境，每个环境的配置一般来说都是不一样的，在这种情况下，我们就需要提供多个配置文件。现在我们来建立三个配置文件：application-dev.yml,application-test.yml,application-prod.yml。在这三个配置文件里我们分别写上不同的配置：
dev:
```yaml
    server:
        port: 8080
```

test:

    server:
        port: 8081
        
prod:
    
    server:
        port: 8082

此配置指的就是项目启动时候的端口号，不写的话默认是8080，我们这里三个环境设置了不同的端口号。那现在问题来了，我们如何读取不同的配置文件呢。
很简单，我们在application.yml中加上：

    spring:
      profiles:
        active: prod
即可，这样读取的就是prod的配置，改成test则是test的配置。现在我们启动一下试试。然后我们访问localhost:8082，请求成功。
![image](https://leafw-blog-pic.oss-cn-hangzhou.aliyuncs.com/b2.png)

但这样又会有一个疑问，我们各个环境的配置文件是区分了，但我们难道每个环境的application.yml文件还得不一样么？这里就要说到spring boot的另一个启动方式了，spring boot除了我们目前使用的通过main方法启动之外，还可以通过jar包的方式去启动。首先我们将项目打jar包。在项目根目录下执行maven打包命令，这里我们跳过测试：

    mvn install -DskipTests

然后在target目录下能看到我们打的jar包，在这个目录下我们执行：

    java -jar XXX.jar --spring.profiles.active=test

然后我们再访问localhost:8081，同样请求成功。
![image](https://leafw-blog-pic.oss-cn-hangzhou.aliyuncs.com/b3.png)

注意我们这里的文件命名规范，都是application-xxx.yml，否则会读取不到对应的配置文件。

当然有人可能觉得我们可以通过命令行来修改运行参数不安全，那这里可以下面这句话就可以屏蔽命令行里面传入的参数：

    SpringApplication.setAddCommandLineProperties(false)。

## 二、自定义属性
我们开发的时候会出现需要一些自定义的属性的情况，这时候我们可以将配置的属性也写进我们的配置文件里，这里我们配置上如下属性：

    leafw:
        user: 1001
        sex: 18

那我们怎样获取属性呢，只需要在属性上面加上注解@Value("${属性名}"):

    @Value("${user.code}")
    private String code;
    @Value("${user.age}")
    private String age;
    

然后加一个请求的方法：

    @GetMapping("/")
    public String index(){
    	return "Spring Boot";
    }
    
访问http://localhost:8080/getUser，如图：

![image](https://leafw-blog-pic.oss-cn-hangzhou.aliyuncs.com/b4.png)

同时这里的参数之间是可以互相引用的，如

    user:
      code: 1001
      age: 18
      desc: ${user.code}的年龄是${user.age}

然后再上述方法中补充下属性配置，请求结果如下：
![image](https://leafw-blog-pic.oss-cn-hangzhou.aliyuncs.com/b5.png)

待续。。。