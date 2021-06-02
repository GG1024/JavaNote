# Docker安装rabbitMq

## 安装rabbitMq

1、获取rabbitMq镜像

```shell
docker pull rabbitmq:management
```

2、创建并运行容器

```shell
docker run -di --name=rabbitmq001 -p 15672:15672 rabiitmq:management
```

运行时设置用户名密码

```shell
### -d 后台运行容器；
### --name 指定容器名；
### -p 指定服务运行的端口（5672：应用访问端口；15672：控制台Web端口号）；
### -v 映射目录或文件；
### --hostname 主机名（RabbitMQ的一个重要注意事项是它根据所谓的 “节点名称” 存储数据，默认为主机名）；
### -e 指定环境变量；（RABBITMQ_DEFAULT_VHOST：默认虚拟机名；RABBITMQ_DEFAULT_USER：默认的用户名；RABBITMQ_DEFAULT_PASS：默认用户名的密码）

docker run -d --name=rabbitmq001 -v /mydata/rabbitmq:/var/lib/rabbitmq \
--hostname myRabbitmq -e RABBITMQ_DEFAULT_VHOST=my_vhost \
-e RABBITMQ_DEFAULT_USER=admin -e RABBITMQ_DEFAULT_PASS=admin \
-p 15672:15672 -p 5672:5672 -p 25672:25672 -p 61613:61613 -p 1883:1883 rabiitmq:management
```

查看运行日志

```shell
docker logs -f [容器名称]
```

```shell
开启防火墙15672端口
firewall-cmd --zone=public --add-port=15672/tcp --permanent;　　　　　　　
firewall-cmd --reload;

firewall-cmd --zone=public --add-port=5672/tcp --permanent;　　　　　　　
firewall-cmd --reload;
```

# RabbitMq入门

## RabbitMq的角色分类

> none

不能访问management plugin

> management：查看自己的节点信息

1.列出自己可以通过AMQP登入的虚拟机

2.查看自己的虚拟机节点，virtual hosts的queues，exchanges，和bindings信息

3.查看和关闭自己的channels，和connections

4.查看有关自己的虚拟机节点virtual hosts的统计信息，包括其他用户在这个节点virtual hosts中活动信息

> policymaker

1.包含management所有权限

2.查看和创建和删除自己的virtual hosts所属的policies和parameters

> Monitoring

1.包含management所有权限

2.罗列出所有的virtual hosts，包括不能登录的virtual hosts

3.查看其他用户的connections和channels信息

4.查看节点级别的数据如clustering和memory

5.查看所有的virtual hosts全局统计信息

> Administrator

1.最高权限

2.创建删除virtual hosts

3.查看创建删除users

4.查看创建permission

5.关闭所有用户的connections

## 入门案例 simple模式

> 步骤

```shell
### 1.maven工程
### 2.rabbitmq的maven的依赖
### 3.启动rabbitmq-server服务
### 4.定义生产者
### 5.定义消费者
### 6.观察消息在rabbitmq-server服务中的工程
```

1.maven工程

![image-20210602225812762](C:\Users\欧阳小广\AppData\Roaming\Typora\typora-user-images\image-20210602225812762.png)

2.导入rabbitmq的maven的依赖

```shell
		<!--java 原生依赖-->
        <dependency>
            <groupId>com.rabbitmq</groupId>
            <artifactId>amqp-client</artifactId>
            <version>5.10.0</version>
        </dependency>
```

3.定义生产者

```java
 		// 1.创建连接工厂
        ConnectionFactory connectionFactory = new ConnectionFactory();
        connectionFactory.setHost("192.168.72.129");
        connectionFactory.setPort(5672);
        connectionFactory.setUsername("admin");
        connectionFactory.setPassword("admin");
        connectionFactory.setVirtualHost("my_vhost");
        Connection connection = null;
        Channel channel = null;
        try {
            // 2.创建连接connection
            connection = connectionFactory.newConnection("生产者");
            // 3.通过连接获取通道 channel
            channel = connection.createChannel();
            // 4.通过通道创建交换机，声明队列，绑定关系，路由key,发送消息，接受消息
            String queueName = "queue001";
            /**
             * @params1 队列的名称
             * @params2 是否要持久化 Declare=false,持久化消息是否会存盘，非持久化=fakse,持久化=true ,非持久化会存盘,但是会随着服务器的重启会丢失。
             * @params3 排他性，是否独占队列
             * @params4 是否自动删除，随着最后一个消费者完毕消息以后是否把队列自动删除
             * @params5 携带一些附加参数
             */
            channel.queueDeclare(queueName, false, false, false, null);
            // 5.准备消息内容
            String message = "Hello xuexi MQ simple";
            // 6.发送消息给队列queue
            channel.basicPublish("", queueName, null, message.getBytes());
            System.out.printf("消息发送成功");
        } catch (Exception e) {
            e.printStackTrace();
        } finally {
            // 7.关闭连接
            if (connection != null && connection.isOpen()) {
                try {
                    connection.close();
                } catch (Exception e) {
                    e.printStackTrace();
                }
            }
            // 8.关闭通道
            if (channel != null && channel.isOpen()) {
                try {
                    channel.close();
                } catch (Exception e) {
                    e.printStackTrace();
                }
            }
        }
```

4.定义消费者

```java
		// 1.创建连接工厂
        ConnectionFactory connectionFactory = new ConnectionFactory();
        connectionFactory.setHost("192.168.72.129");
        connectionFactory.setPort(5672);
        connectionFactory.setUsername("admin");
        connectionFactory.setPassword("admin");
        connectionFactory.setVirtualHost("my_vhost");
        Connection connection = null;
        Channel channel = null;
        try {
            // 2.创建连接connection
            connection = connectionFactory.newConnection("生产者");
            // 3.通过连接获取通道 channel
            channel = connection.createChannel();
            // 4.通过通道创建交换机，声明队列，绑定关系，路由key,发送消息，接受消息
            String queueName = "queue001";
            channel.basicConsume(queueName, true, new DeliverCallback() {
                @Override
                public void handle(String s, Delivery message) throws IOException {
                    System.out.println("收到消息是：" + new String(message.getBody(), "UTF-8"));
                }
            }, new CancelCallback() {
                @Override
                public void handle(String s) throws IOException {
                    System.out.println("接受失败.....");
                }
            });
            System.out.println("开始接受消息....");
            System.in.read();
        } catch (Exception e) {
            e.printStackTrace();
        } finally {
            // 7.关闭连接
            if (connection != null && connection.isOpen()) {
                try {
                    connection.close();
                } catch (Exception e) {
                    e.printStackTrace();
                }
            }
            // 8.关闭通道
            if (channel != null && channel.isOpen()) {
                try {
                    channel.close();
                } catch (Exception e) {
                    e.printStackTrace();
                }
            }
        }
```

