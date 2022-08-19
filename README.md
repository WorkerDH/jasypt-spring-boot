# 1jasypt-spring-boot

> 该插件对配置文件加解密一共有三种方式，第一种是写一个密文的Java生成类，这个并不推荐，自然也就不赘述了。
>
> 主要介绍的是通过环境变量的方式来和在运行时注入参数的这两种方式来解密，然后引入jasypt的插件来加密配置。

### 1.如何加密

首先我们需要引入必要的依赖和插件如下(请保持插件和依赖的版本一致，否则你可能需要自己去定义算法的参数，因为不同的版本的算法和参数有的是不同的)：

```xml
<!--依赖引入-->
<dependency>   
    <groupId>com.github.ulisesbocchio</groupId>    
    <artifactId>jasypt-spring-boot-starter</artifactId>
    <version>3.0.4</version>
</dependency>
```

```xml
<!--插件引入-->
<plugin>    
    <groupId>com.github.ulisesbocchio</groupId>    
    <artifactId>jasypt-maven-plugin</artifactId>    
    <version>3.0.4</version>
</plugin>
```

现在我们有一个没有明文的配置文件中的属性如下：

```properties
jasypt.encryptor.password=${secretKey:} 
spring.datasource.password=123456
```

我们将其密码包裹在DEC中，如下

```properties
spring.datasource.password=DEC(123456)
```

然后我们在该项目下的doc运行如下maven指令，将DEC中的内容加密(其中的"abcd_123"就是我们的密钥，一会儿我们还要用这个密钥来解密)：

```bash
mvn jasypt:encrypt -Djasypt.encryptor.password="abcd_123"
```

这个指令执行之后，会更改我们配置文件中所有被DEC包裹的密码，使其变成密文，同事DEC也会替换成ENC,如下：

```properties
spring.datasource.password=ENC(kjinUJsDRKH4Bbv96NpGqUsHF9STB7/g0/jukawHvOu7oj4RXQjja1vko5iC5A7i)
```

至此，配置的加密完成。

### 2.如何解密

这里的解密并不是将ENC中的密文在配置文件中更改成明文，那样密文也就无用了，而是在项目启动的时候，jasypt通过密钥解密，获取到解密之后的明文给需要这个配置的程序使用。所以我们需要在启动的时候传入一个密钥参数。现在介绍以下注入两种方式：

##### 1) 启动参数注入

这个方式很简单，我们只需要运行jar包的时候为其注入我们的密钥即可，注入方式如下：

```
java -jar target\jasyptspringboot-0.0.1-SNAPSHOT.jar --jasypt.encryptor.password=abcd_123
```

我们可以看到，“abcd_123”就是我们刚刚加密明文配置所用到的密钥，我们现在用它来解密了。

需要注意的是，采用这种方式的时候，你如果在配置文件中增加了jasypt.encryptor.passwor这个属性，请将它注释掉。

##### 2) 环境变量注入

与第一种方式不同的是，这次我们需要在配置环境去配置我们的密钥，新增的配置如下：

```properties
jasypt.encryptor.password=${secretKey:} 
```

上面的配置的意思是，从我们的系统环境变量中去找到secretKey这个值来赋给jasypt.encryptor.password；所以我们需要在系统环境变量中新增一个变量secretKey，值为我们刚刚用来加密的密钥“abcd_123”



