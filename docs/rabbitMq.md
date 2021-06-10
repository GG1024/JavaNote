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

> rabbitmq为什么是给予channel处理而不是连接？？

> 面试题：可以存在没有交换机的队列吗？ 不可能，虽然没有指定交换机但是一定会存在默认的交换机

> 核心组成部分

![image-20210603230632650](C:\Users\欧阳小广\AppData\Roaming\Typora\typora-user-images\image-20210603230632650.png)

1.server：又称broker，接受客户端的连接，实现AMQP实体服务，rabbitmq-server

2.Connection：连接，应用程序与broker的网络连接，TCP/IP 三次握手四次挥手

3.Channel：网络信道，几乎所有的操作都会在Channel中进行，Channel是进行消息读写的通道，客户端可以建立对各Channel，每个Channel代表一个会话任务

4.Message：消息，服务与应用程序之间传送的数据，由Properties和Body组成，Properties可对消息进行修饰，比如消息的优先级，延迟等高级特性，body则是消息的内容

5.Virtual Hosts：虚拟地址，用于进行逻辑隔离，最上层的消息路由，一个虚拟主机理论可以有若干个Exchange和Queue，同一个虚拟机中不能有相同名字的Exchange

6.Bindings：Exchange和Queue之间的虚拟连接，binding中可以保护多个routing key

7.Routing Key：是一个路由规则，虚拟机可以用它来确定如何路由一个消息

8.Queue：队列，成为MessageQueue 消息队列，保存消息并转发给消费者

> 架构图

![img](https://gimg2.baidu.com/image_search/src=http%3A%2F%2Fimage.mamicode.com%2Finfo%2F201912%2F20191221134658031491.png&refer=http%3A%2F%2Fimage.mamicode.com&app=2002&size=f9999,10000&q=a80&n=0&g=0n&fmt=jpeg?sec=1625325968&t=68881e5b1a8b529105e6f6254ea42271)

> 运行流程

![img](https://gimg2.baidu.com/image_search/src=http%3A%2F%2Fupload-images.jianshu.io%2Fupload_images%2F17014808-7c597bc1a05007e6.png&refer=http%3A%2F%2Fupload-images.jianshu.io&app=2002&size=f9999,10000&q=a80&n=0&g=0n&fmt=jpeg?sec=1625326014&t=9499430c8f6b0d93099fd96330253fdb)

## fanout模式

发布订阅模式，是一种广播机制，它没有路由key的模式

![img](https://gimg2.baidu.com/image_search/src=http%3A%2F%2Fsegmentfault.com%2Fimg%2FbVcSmOh&refer=http%3A%2F%2Fsegmentfault.com&app=2002&size=f9999,10000&q=a80&n=0&g=0n&fmt=jpeg?sec=1625407706&t=db2cd91b36bccda0550cb8bae45281c4)

生产者

```java
    public static void main(String[] args) {
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
            // 4.准备消息内容
            String message = "Hello xuexi MQ fanout";
            // 5.准备交换机
            String exchangeName="my-fanout";
            // 6.定义路由key
            String routingKey = "";
            // 7.定义交换机的类型
            String type = "fanout";

            channel.basicPublish(exchangeName, routingKey, null, message.getBytes());
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
    }
```

消费者

```java
private static Runnable runnable = new Runnable() {
        @Override
        public void run() {
            // 1.创建连接工厂
            ConnectionFactory connectionFactory = new ConnectionFactory();
            connectionFactory.setHost("192.168.72.129");
            connectionFactory.setPort(5672);
            connectionFactory.setUsername("admin");
            connectionFactory.setPassword("admin");
            connectionFactory.setVirtualHost("my_vhost");
            // 获取队列的名称
            final String queueName = Thread.currentThread().getName();
            Connection connection = null;
            Channel channel = null;
            try {
                // 2.创建连接connection
                connection = connectionFactory.newConnection("生产者");
                // 3.通过连接获取通道 channel
                channel = connection.createChannel();
                // 4.通过通道创建交换机，声明队列，绑定关系，路由key,发送消息，接受消息
                Channel finalChannel = channel;
                finalChannel.basicConsume(queueName, true, new DeliverCallback() {
                    @Override
                    public void handle(String s, Delivery message) throws IOException {
                        System.out.println(message.getEnvelope().getDeliveryTag());
                        System.out.println("收到消息是：" + new String(message.getBody(), "UTF-8"));
                    }
                }, new CancelCallback() {
                    @Override
                    public void handle(String s) throws IOException {
                        System.out.println("接受失败.....");
                    }
                });
                System.out.println(queueName + "开始接受消息....");
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
        }
    };

    public static void main(String[] args) {
        new Thread(runnable, "queue1").start();
        new Thread(runnable, "queue2").start();
        new Thread(runnable, "queue3").start();
    }
```

## direct模式

![img](https://gimg2.baidu.com/image_search/src=http%3A%2F%2Fimg2018.cnblogs.com%2Fblog%2F1496926%2F201907%2F1496926-20190708125508230-1549906538.png&refer=http%3A%2F%2Fimg2018.cnblogs.com&app=2002&size=f9999,10000&q=a80&n=0&g=0n&fmt=jpeg?sec=1625408958&t=99e5c9ed80a87f9c10f2877b138796af)

生产者

```java
public static void main(String[] args) {
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
            // 4.准备消息内容
            String message = "Hello xuexi MQ direct";
            // 5.准备交换机
            String exchangeName="my-direct";
            // 6.定义路由key
            String routingKey = "email";
            // 7.定义交换机的类型
            String type = "direct";

            channel.basicPublish(exchangeName, routingKey, null, message.getBytes());
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
    }
```

消费者

```java
private static Runnable runnable = new Runnable() {
        @Override
        public void run() {
            // 1.创建连接工厂
            ConnectionFactory connectionFactory = new ConnectionFactory();
            connectionFactory.setHost("192.168.72.129");
            connectionFactory.setPort(5672);
            connectionFactory.setUsername("admin");
            connectionFactory.setPassword("admin");
            connectionFactory.setVirtualHost("my_vhost");
            // 获取队列的名称
            final String queueName = Thread.currentThread().getName();
            Connection connection = null;
            Channel channel = null;
            try {
                // 2.创建连接connection
                connection = connectionFactory.newConnection("生产者");
                // 3.通过连接获取通道 channel
                channel = connection.createChannel();
                // 4.通过通道创建交换机，声明队列，绑定关系，路由key,发送消息，接受消息
                Channel finalChannel = channel;
                finalChannel.basicConsume(queueName, true, new DeliverCallback() {
                    @Override
                    public void handle(String s, Delivery message) throws IOException {
                        System.out.println(message.getEnvelope().getDeliveryTag());
                        System.out.println("收到消息是：" + new String(message.getBody(), "UTF-8"));
                    }
                }, new CancelCallback() {
                    @Override
                    public void handle(String s) throws IOException {
                        System.out.println("接受失败.....");
                    }
                });
                System.out.println(queueName + "开始接受消息....");
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
        }
    };

    public static void main(String[] args) {
        new Thread(runnable, "queue1").start();
        new Thread(runnable, "queue2").start();
        new Thread(runnable, "queue3").start();
    }
```

## topic模式

![img](https://gimg2.baidu.com/image_search/src=http%3A%2F%2Fimg2020.cnblogs.com%2Fi-beta%2F1883867%2F202003%2F1883867-20200307213938650-1888156121.png&refer=http%3A%2F%2Fimg2020.cnblogs.com&app=2002&size=f9999,10000&q=a80&n=0&g=0n&fmt=jpeg?sec=1625409328&t=bfa4bdf1d340c9ab79073bfe309d5a0a)

生产者

```java
public static void main(String[] args) {
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
            // 4.准备消息内容
            String message = "Hello xuexi MQ topic";
            // 5.准备交换机
            String exchangeName="my-topic";
            // 6.定义路由key
            String routingKey = "com.order.user.test";
            // 7.定义交换机的类型
            String type = "topic";

            channel.basicPublish(exchangeName, routingKey, null, message.getBytes());
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
    }
```

消费者

```
private static Runnable runnable = new Runnable() {
        @Override
        public void run() {
            // 1.创建连接工厂
            ConnectionFactory connectionFactory = new ConnectionFactory();
            connectionFactory.setHost("192.168.72.129");
            connectionFactory.setPort(5672);
            connectionFactory.setUsername("admin");
            connectionFactory.setPassword("admin");
            connectionFactory.setVirtualHost("my_vhost");
            // 获取队列的名称
            final String queueName = Thread.currentThread().getName();
            Connection connection = null;
            Channel channel = null;
            try {
                // 2.创建连接connection
                connection = connectionFactory.newConnection("生产者");
                // 3.通过连接获取通道 channel
                channel = connection.createChannel();
                // 4.通过通道创建交换机，声明队列，绑定关系，路由key,发送消息，接受消息
                Channel finalChannel = channel;
                finalChannel.basicConsume(queueName, true, new DeliverCallback() {
                    @Override
                    public void handle(String s, Delivery message) throws IOException {
                        System.out.println(message.getEnvelope().getDeliveryTag());
                        System.out.println("收到消息是：" + new String(message.getBody(), "UTF-8"));
                    }
                }, new CancelCallback() {
                    @Override
                    public void handle(String s) throws IOException {
                        System.out.println("接受失败.....");
                    }
                });
                System.out.println(queueName + "开始接受消息....");
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
        }
    };

    public static void main(String[] args) {
        new Thread(runnable, "queue1").start();
        new Thread(runnable, "queue2").start();
        new Thread(runnable, "queue3").start();
        new Thread(runnable, "queue4").start();
    }
```

## 完整的代码声明创建方式

生产者

```java
public static void main(String[] args) {
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
            // 4.准备消息内容
            String message = "Hello xuexi  mq java coding params";
            // 5.准备交换机
            String exchangeName = "direct_message_exchange";
            // 6.定义交换机的类型
            String type = "direct";
            // 7.定义路由key
            String routingKey = "course";

            // 注册交换机
            /**
             * @params1 交换机名称
             * @params2 交换机类型
             * @params3 是否持久化，交换机会不会随着服务的重启造成丢失，true代表不丢失，false重启就会丢失
             */
            channel.exchangeDeclare(exchangeName, type, true);

            //声明队列
            channel.queueDeclare("queue5", true, false, false, null);
            channel.queueDeclare("queue6", true, false, false, null);
            channel.queueDeclare("queue7", true, false, false, null);
            // 绑定队列和交换机的关系
            channel.queueBind("queue5",exchangeName,"order");
            channel.queueBind("queue6",exchangeName,"order");
            channel.queueBind("queue7",exchangeName,"course");

            channel.basicPublish(exchangeName, routingKey, null, message.getBytes());
            System.out.printf("消息发送成功");
        } catch (Exception e) {
            e.printStackTrace();
        } finally {
            // 7.关闭通道
            if (channel != null && channel.isOpen()) {
                try {
                    channel.close();
                } catch (Exception e) {
                    e.printStackTrace();
                }
            }

            // 8.关闭连接
            if (connection != null && connection.isOpen()) {
                try {
                    connection.close();
                } catch (Exception e) {
                    e.printStackTrace();
                }
            }

        }
    }
```

消费者

```java
 private static Runnable runnable = new Runnable() {
        @Override
        public void run() {
            // 1.创建连接工厂
            ConnectionFactory connectionFactory = new ConnectionFactory();
            connectionFactory.setHost("192.168.72.129");
            connectionFactory.setPort(5672);
            connectionFactory.setUsername("admin");
            connectionFactory.setPassword("admin");
            connectionFactory.setVirtualHost("my_vhost");
            // 获取队列的名称
            final String queueName = Thread.currentThread().getName();
            Connection connection = null;
            Channel channel = null;
            try {
                // 2.创建连接connection
                connection = connectionFactory.newConnection("生产者");
                // 3.通过连接获取通道 channel
                channel = connection.createChannel();
                // 4.通过通道创建交换机，声明队列，绑定关系，路由key,发送消息，接受消息
                Channel finalChannel = channel;
                finalChannel.basicConsume(queueName, true, new DeliverCallback() {
                    @Override
                    public void handle(String s, Delivery message) throws IOException {
                        System.out.println(queueName + "收到消息是：" + new String(message.getBody(), "UTF-8"));
                    }
                }, new CancelCallback() {
                    @Override
                    public void handle(String s) throws IOException {
                        System.out.println("接受失败.....");
                    }
                });
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
        }
    };

    public static void main(String[] args) {
        new Thread(runnable, "queue5").start();
        new Thread(runnable, "queue6").start();
        new Thread(runnable, "queue7").start();

    }
```

## Work模式轮询模式（Round-Robin

![img](https://gimg2.baidu.com/image_search/src=http%3A%2F%2Fpic2.zhimg.com%2Fv2-ca38bd6e2412c69988601d8b47e81951_r.jpg&refer=http%3A%2F%2Fpic2.zhimg.com&app=2002&size=f9999,10000&q=a80&n=0&g=0n&fmt=jpeg?sec=1625411760&t=1b2efb44a1fabfed8f744ab7b92583a3)

轮询模式：一个消费者一条，按均分配

公平分发：根据消费者的消费能力进行公平分发，处理快的处理多，处理慢的处理少，按劳分配。

> 轮询模式：该模式接收消息当有多个消费者接入时，一个消费者一条，按均分配直至消费完成。

生产者

```java
public static void main(String[] args) {
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
            // 4.准备消息内容
            //============================end topic 模式=============================
            for (int i = 1; i <=20 ; i++) {
                String message = "Hello xuexi  mq java coding lunxun" +i;
                channel.basicPublish("", "queue1", null, message.getBytes());
                Thread.sleep(1000);
            }
            System.out.printf("消息发送成功");
        } catch (Exception e) {
            e.printStackTrace();
        } finally {
            // 7.关闭通道
            if (channel != null && channel.isOpen()) {
                try {
                    channel.close();
                } catch (Exception e) {
                    e.printStackTrace();
                }
            }

            // 8.关闭连接
            if (connection != null && connection.isOpen()) {
                try {
                    connection.close();
                } catch (Exception e) {
                    e.printStackTrace();
                }
            }

        }
    }
```

Work1

```java
public static void main(String[] args) {
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
            connection = connectionFactory.newConnection("消费者-Work1");
            // 3.通过连接获取通道 channel
            channel = connection.createChannel();
            Channel finalChannel = channel;
            finalChannel.basicConsume("queue1", true, new DeliverCallback() {
                @Override
                public void handle(String s, Delivery message) throws IOException {
                    System.out.println("Work-1收到消息是：" + new String(message.getBody(), "UTF-8"));
                    try {
                        Thread.sleep(1000);
                    } catch (Exception e) {
                        e.printStackTrace();
                    }
                }
            }, new CancelCallback() {
                @Override
                public void handle(String s) throws IOException {
                    System.out.println("接受失败.....");
                }
            });
            System.out.println("Work-1开始接受消息....");
            System.in.read();
        } catch (Exception e) {
            e.printStackTrace();
        } finally {
            // 7.关闭通道
            if (channel != null && channel.isOpen()) {
                try {
                    channel.close();
                } catch (Exception e) {
                    e.printStackTrace();
                }
            }
            // 8.关闭连接
            if (connection != null && connection.isOpen()) {
                try {
                    connection.close();
                } catch (Exception e) {
                    e.printStackTrace();
                }
            }
        }
    }
```

Work2

```java
public static void main(String[] args) {
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
            connection = connectionFactory.newConnection("消费者-Work2");
            // 3.通过连接获取通道 channel
            channel = connection.createChannel();
           Channel finalChannel = channel;
            finalChannel.basicConsume("queue1", true, new DeliverCallback() {
                @Override
                public void handle(String s, Delivery message) throws IOException {
                    System.out.println("Work-2收到消息是：" + new String(message.getBody(), "UTF-8"));
                    try {
                        Thread.sleep(2000);
                    } catch (Exception e) {
                        e.printStackTrace();
                    }
                }
            }, new CancelCallback() {
                @Override
                public void handle(String s) throws IOException {
                    System.out.println("接受失败.....");
                }
            });
            System.out.println("Work-2开始接受消息....");
            System.in.read();
        } catch (Exception e) {
            e.printStackTrace();
        } finally {
            // 7.关闭通道
            if (channel != null && channel.isOpen()) {
                try {
                    channel.close();
                } catch (Exception e) {
                    e.printStackTrace();
                }
            }
            // 8.关闭连接
            if (connection != null && connection.isOpen()) {
                try {
                    connection.close();
                } catch (Exception e) {
                    e.printStackTrace();
                }
            }
        }
    }
```

> 公平分发，一定要设置qos,手动确认

生产者

```java
public static void main(String[] args) {
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
            // 4.准备消息内容
            //============================end topic 模式=============================
            for (int i = 1; i <=20 ; i++) {
                String message = "Hello xuexi  mq java coding lunxun" +i;
                channel.basicPublish("", "queue1", null, message.getBytes());
            }
            System.out.printf("消息发送成功");
        } catch (Exception e) {
            e.printStackTrace();
        } finally {
            // 7.关闭通道
            if (channel != null && channel.isOpen()) {
                try {
                    channel.close();
                } catch (Exception e) {
                    e.printStackTrace();
                }
            }

            // 8.关闭连接
            if (connection != null && connection.isOpen()) {
                try {
                    connection.close();
                } catch (Exception e) {
                    e.printStackTrace();
                }
            }

        }
    }
```

Work-1

```java
public static void main(String[] args) {
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
            connection = connectionFactory.newConnection("消费者-Work1");
            // 3.通过连接获取通道 channel
            channel = connection.createChannel();
            Channel finalChannel = channel;
            finalChannel.basicQos(1);
            finalChannel.basicConsume("queue1", false, new DeliverCallback() {
                @Override
                public void handle(String s, Delivery message) throws IOException {
                    try {
                        System.out.println("Work-1收到消息是：" + new String(message.getBody(), "UTF-8"));
                        Thread.sleep(200);
                        finalChannel.basicAck(message.getEnvelope().getDeliveryTag(), false);
                    } catch (Exception e) {
                        e.printStackTrace();
                    }
                }
            }, new CancelCallback() {
                @Override
                public void handle(String s) throws IOException {
                    System.out.println("接受失败.....");
                }
            });
            System.out.println("Work-1开始接受消息....");
            System.in.read();
        } catch (Exception e) {
            e.printStackTrace();
        } finally {
            // 7.关闭通道
            if (channel != null && channel.isOpen()) {
                try {
                    channel.close();
                } catch (Exception e) {
                    e.printStackTrace();
                }
            }
            // 8.关闭连接
            if (connection != null && connection.isOpen()) {
                try {
                    connection.close();
                } catch (Exception e) {
                    e.printStackTrace();
                }
            }
        }
    }
```

Work-2

```java
public static void main(String[] args) {
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
            connection = connectionFactory.newConnection("消费者-Work2");
            // 3.通过连接获取通道 channel
            channel = connection.createChannel();
            Channel finalChannel = channel;
            finalChannel.basicQos(1);
            finalChannel.basicConsume("queue1", false, new DeliverCallback() {
                @Override
                public void handle(String s, Delivery message) throws IOException {
                    try {
                        System.out.println("Work-2收到消息是：" + new String(message.getBody(), "UTF-8"));
                        finalChannel.basicAck(message.getEnvelope().getDeliveryTag(), false);
                        Thread.sleep(1000);
                    } catch (Exception e) {
                        e.printStackTrace();
                    }
                }
            }, new CancelCallback() {
                @Override
                public void handle(String s) throws IOException {
                    System.out.println("接受失败.....");
                }
            });
            System.out.println("Work-2开始接受消息....");
            System.in.read();
        } catch (Exception e) {
            e.printStackTrace();
        } finally {
            // 7.关闭通道
            if (channel != null && channel.isOpen()) {
                try {
                    channel.close();
                } catch (Exception e) {
                    e.printStackTrace();
                }
            }
            // 8.关闭连接
            if (connection != null && connection.isOpen()) {
                try {
                    connection.close();
                } catch (Exception e) {
                    e.printStackTrace();
                }
            }
        }
    }
```

# SpringBoot整合

## fanout 模式

通用pom

```java
		<dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-amqp</artifactId>
        </dependency>
```

生产者配置

```java
@Configuration
public class RabbitMqConfiguration implements Serializable {

    //1.声明注册fanout模式的交换机
    @Bean
    public FanoutExchange fanoutExchange() {
        return new FanoutExchange("fanout_order_exchange", true, false);
    }

    //2.声明队列 sms.fanout.queue email.fanout.queue duanxin.fanout.queue
    @Bean
    public Queue smsQueue() {
        return new Queue("sms.fanout.queue", true);
    }

    @Bean
    public Queue emailQueue() {
        return new Queue("email.fanout.queue", true);
    }

    @Bean
    public Queue duanxinQueue() {
        return new Queue("duanxin.fanout.queue", true);
    }


    //3.完成交换机与队列的绑定关系
    @Bean
    public Binding bindingSms() {
        return BindingBuilder.bind(smsQueue()).to(fanoutExchange());
    }

    @Bean
    public Binding bindingEmail() {
        return BindingBuilder.bind(emailQueue()).to(fanoutExchange());
    }

    @Bean
    public Binding bindingDuanxin() {
        return BindingBuilder.bind(duanxinQueue()).to(fanoutExchange());
    }

}

```

生产者发送消息

```
 	/**
     * 模拟下单
     */
public void makeOrder(String userId, String productId, int num) {

        //1:根据商品id查询库存是否充足
        //2:保存订单
        String orderId = UUID.randomUUID().toString();
        System.out.println("订单生成成功：" + orderId);

        //3.通过MQ来完成消息的分发
        String exchangeName = "fanout_order_exchange";
        String routingKey = "";
        /**
         * @param1 交换机
         * @param2 路由Key/queue
         * @param3 消息内容
         */
        rabbitTemplate.convertAndSend(exchangeName,routingKey,orderId);
    }
```

消费者其一

```java
@Component
@RabbitListener(queues = {"duanxin.fanout.queue"})
public class DuanxinConsumer {

    @RabbitHandler
    public void reviceMessage(String message) {
        System.out.println("duanxin fanout ------" + message);
    }
}
```

## direct模式

生产者配置

```java
@Configuration
public class DirectConfiguration implements Serializable {

    //1.声明注册directt模式的交换机
    @Bean
    public DirectExchange directtExchange() {
        return new DirectExchange("direct_order_exchange", true, false);
    }

    //2.声明队列 sms.direct.queue email.direct.queue duanxin.direct.queue
    @Bean
    public Queue smsDirectQueue() {
        return new Queue("sms.direct.queue", true);
    }

    @Bean
    public Queue emailDirectQueue() {
        return new Queue("email.direct.queue", true);
    }

    @Bean
    public Queue duanxinDirectQueue() {
        return new Queue("duanxin.direct.queue", true);
    }


    //3.完成交换机与队列的绑定关系
    @Bean
    public Binding bindingSmsDirect() {
        return BindingBuilder.bind(smsDirectQueue()).to(directtExchange()).with("sms");
    }

    @Bean
    public Binding bindingEmailDirect() {
        return BindingBuilder.bind(emailDirectQueue()).to(directtExchange()).with("email");
    }

    @Bean
    public Binding bindingDuanxinDirect() {
        return BindingBuilder.bind(duanxinDirectQueue()).to(directtExchange()).with("duanxin");
    }

}
```

消费者其一

```java
@Component
@RabbitListener(queues = {"duanxin.direct.queue"})
public class DuanxinDirectConsumer {

    @RabbitHandler
    public void reviceMessage(String message) {
        System.out.println("duanxin direct ------" + message);
    }
}
```

## topic模式

生产者

```java
public void makeOrderTopic(String userId, String productId, int num) {

        //1:根据商品id查询库存是否充足
        //2:保存订单
        String orderId = UUID.randomUUID().toString();
        System.out.println("订单生成成功：" + orderId);

        //3.通过MQ来完成消息的分发
        String exchangeName = "topic_order_exchange";
        String routingKey = "com.email.duanxin.xxx";
        /**
         * #.duanxin.#
         * *.email.#
         * com.#
         */
        rabbitTemplate.convertAndSend(exchangeName,routingKey,orderId);
}
```

消费者其一

```java
@Component
@RabbitListener(bindings = @QueueBinding(
        value = @Queue(value = "email.topic.queue", declare = "true", autoDelete = "false"),
        exchange = @Exchange(value = "topic_order_exchange", type = ExchangeTypes.TOPIC),
        key = "*.email.#"
)
)
public class EmailTopicConsumer {

    @RabbitHandler
    public void reviceMessage(String message) {
        System.out.println("email topic ------" + message);
    }

}
```



# 高级特性

## 过期时间TTL

过期时间TTL表示可以对消息设置预期的时间，在这个时间内都可以被消费者接收获取，过了之后消息将被自动删除，RabbitMq可以对消息队列设置TTL。

1.通过队列属性设置，队列中所有消息都有相同的过期的时间

2.对消息单独设置，每条消息TTL可以不同

上面两种方法同时使用，则消息的过期时间，以两者之间TTL较小的那个数值为准，消息在队列的生存时间一旦超过设置的TTL值，就称为dead message被投递到死信队列。消费者无法再消费到该消息。

![image-20210606152645305](C:\Users\欧阳小广\AppData\Roaming\Typora\typora-user-images\image-20210606152645305.png)

> 队列设置TTL

```java
@Configuration
public class TtlDirectConfiguration implements Serializable {

    //1.声明注册directt模式的交换机
    @Bean
    public DirectExchange ttlDirecttExchange() {
        return new DirectExchange("ttl_direct_exchange", true, false);
    }
    //2.声明队列 队列的过期时间
    @Bean
    public Queue ttlDirectQueue() {
        Map<String, Object> args = new HashMap<>();
        args.put("x-message-ttl", 5000);
        return new Queue("ttl.direct.queue", true,false,false,args);
    }
    //3.完成交换机与队列的绑定关系
    @Bean
    public Binding bindingTTLDirect() {
        return BindingBuilder.bind(ttlDirectQueue()).to(ttlDirecttExchange()).with("ttl");
    }
}


//生产者
public void makeOrderTTl(String userId, String productId, int num) {

        //1:根据商品id查询库存是否充足
        //2:保存订单
        String orderId = UUID.randomUUID().toString();
        System.out.println("订单生成成功：" + orderId);

        //3.通过MQ来完成消息的分发
        String exchangeName = "ttl_direct_exchange";
        String routingKey = "ttl";
        /**
         * #.duanxin.#
         * *.email.#
         * com.#
         */
        rabbitTemplate.convertAndSend(exchangeName, routingKey, orderId);
}
```

> 对消息进行单独设置

```java
 	@Bean
    public Queue ttlMessageDirectQueue() {
        return new Queue("ttl.message.direct.queue", true);
    }

    @Bean
    public Binding bindingTTLMessageDirect() {
        return BindingBuilder.bind(ttlMessageDirectQueue()).to(ttlDirecttExchange()).with("ttlmessage");
    }

//生产者
public void makeOrderTTlMassage(String userId, String productId, int num) {

        //1:根据商品id查询库存是否充足
        //2:保存订单
        String orderId = UUID.randomUUID().toString();
        System.out.println("订单生成成功：" + orderId);

        //3.通过MQ来完成消息的分发
        String exchangeName = "ttl_direct_exchange";
        String routingKey = "ttlmessage";
        //给消息设置过期时间
        MessagePostProcessor messagePostProcessor = new MessagePostProcessor() {
            @Override
            public Message postProcessMessage(Message message) throws AmqpException {
                message.getMessageProperties().setExpiration("5000");
                message.getMessageProperties().setContentEncoding("UTF-8");
                return message;
            }
        };
        rabbitTemplate.convertAndSend(exchangeName, routingKey, orderId,messagePostProcessor);
    }
```

## 死信队列

DLX，死信交换机，当消息在一个队列中变成死信之后，它能被重新发送到另外一个交换机中，这个交换机就是DLX，绑定DLX的队列就称之为死信队列。消息变成死信以下原因造成：

1.消息被拒绝

2.消息过期

3.队列达到最大长度

死信队列配置

```java
@Configuration
public class DeadRabbitMqConfiguration implements Serializable {

    @Bean
    public DirectExchange deadDirect() {
        return new DirectExchange("dead_direct_exchange", true, false);
    }

    @Bean
    public Queue deadQueue() {
        return new Queue("dead.direct.queue", true);
    }
    @Bean
    public Binding deadBinding() {
        return BindingBuilder.bind(deadQueue()).to(deadDirect()).with("dead");
    }

}
```

关联配置

```java
    //1.声明注册directt模式的交换机
    @Bean
    public DirectExchange ttlDirecttExchange() {
        return new DirectExchange("ttl_direct_exchange", true, false);
    }

    //2.声明队列 队列的过期时间
    @Bean
    public Queue ttlDirectQueue() {
        Map<String, Object> args = new HashMap<>();
        //队列过期时间
        args.put("x-message-ttl", 5000);
        //队列最大长度
        args.put("x-max-length", 5);
        //死信队列
        args.put("x-dead-letter-exchange", "dead_direct_exchange");
        args.put("x-dead-letter-routing-key", "dead");
        return new Queue("ttl.direct.queue", true, false, false, args);
    }

    //3.完成交换机与队列的绑定关系
    @Bean
    public Binding bindingTTLDirect() {
        return BindingBuilder.bind(ttlDirectQueue()).to(ttlDirecttExchange()).with("ttl");
    }
```

# 内存磁盘的监控

![image-20210606161357717](C:\Users\欧阳小广\AppData\Roaming\Typora\typora-user-images\image-20210606161357717.png)

当内存使用超过配置的阈值或者磁盘空间剩余空间对于配置阈值时，RabbitMq会暂时阻塞客户端的连接，并且停止接收从客户端发来的消息，一次避免服务器的崩溃，客户端与服务端的心态检测机制也会失效。

# Docker 安装RabbitMq集群

# 分布式事务