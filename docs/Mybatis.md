## Mybatis

### Mybatis概述

mybatis是一个持久层框架，用java编写的。

它封装了jdbc操作的很多底层细节，使开发者只需要关注sql语句本身，而无需关注注册驱动，创捷连接等繁杂过程。

它使用了ORM思想，实现了结果集的封装。

**ORM**：

object relational mapping 对象关系映射

简单的说：就是把数据库表和实体类及实体类的属性对应起来

​					让我们可以操作实体类就能实现操作数据库表。



步骤：

第一步：读取配置文件

第二步：创建SqlSessionFactory工厂

第三步：创建SqlSession

第四步：创建Dao接口的代理对象

第五步：执行dao中的方法

第六步：释放资源



注意事项：不要忘记在映射配置中告知mybatis从数据库中返回的结果要封装到哪个实体类中。

配置方式：指定实体类的全限定类名



### properties属性

可以在标签内部配置连接数据库的信息，也可以通过属性引用外部配置文件信息。

- resource属性
  - 用于指定配置文件的位置，是按照类路径的写法来写，并且必须存在于类路径下
- url属性
  - 是要求按照url的写法来写地址
  - url：uniform resource locator 统一资源定位符，他是可以唯一标识一个资源的位置。
  - 写法：协议  主机  端口  uri
    - http://localhoest:8080/spring
    - uri：uniform resource identifier 统一资源标识符，他是在应用中可以唯一定位一个资源的。



### typeAliases 配置别名

他只能写实体类的别名

- typeAlias
  - type 属性指定的是实体类的全限定类名
  - alias 属性指定别名，当指定了别名就不再区分大小写。

- package
  - 用于指定要配置别名的包，当指定之后，该包下的实体类都会注册别名，并且类名就是别名，不再区分大小写。





### Mybatis中连接池

**连接池可以减少我们获取连接所消耗的时间。**

连接池就是用于存储连接的一个容器

容器其实就是一个集合对象，该集合必须是线程安全的，不能两个线程拿到同一个连接。

该集合必须是心啊队列的特性，先进先出。



#### mybatis连接池

提供了3种方式的配置：

- 配置的位置：主配置文件SqlMapConfig.xml中的dataSource标签，type属性就是表示采用何种连接池方式。
- type属性取值
  - POOLED：采用传统的javax.sql.DataSource规范中的连接池，mybatis中有针对规范的是实现
  - UNPOOLED：采用传统的获取连接的方式，虽然也实现了javax.sql.DataSource接口，但是并没有使用池的思想。
  - JNDI：采用服务器提供的 JNDI 技术实现，来获取DataSource对象，不同的服务器所能拿到的DataSource是不一样的。
    - 注意不是web或者maven的war工程，是不能使用的。
    - 