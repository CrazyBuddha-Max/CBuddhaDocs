# <span style='color:blue'>原创：JAVA网络编程技术</span> :smile:
***
### 1.网络编程的<span style='color:orange'>定义</span>
- 可以让设备中的程序与网络上其他设备中的程序进行数据交互，也就是实现网络通信的技术。
***
### 2.<span style='color:orange'>java</span>为网络编程提供了解决方案
- Java\.net.*包。
***
### 3.网络通信的基本架构
- #### <span style='color:red'>CS和BS：</span>
    - （client客户端/server服务端）CS架构、(browser浏览器/server服务器) BS架构。
- #### CS和BS区别
    - CS需要开发客户端，用户需要安装客户端才能使用。服务器更新后客户端也要更新，用户必须更新客户端程序才能正常使用。BS不需要开发浏览器，用户只需要安装浏览器就可以使用。服务器更新后浏览器不需要更新，用户不需要更新就可以使用。
  ***
- #### <span style='color:red'>注意：</span>
> 通信架构都需要依赖网络编程技术。
***
# 4.网络通信的三要素
- ### IP地址
  - 定义
  - IP地址是Internet上每台计算机的唯一标识，用来在网络中唯一标识一台计算机。
  - 分类
    - IPv4：32位二进制数，通常被分为4段，每段8位，共32位。
    - IPv6：128位二进制数，通常被分为8段，每段16位，共128位。
  - 域名
    - 定义
    - 域名是IP地址的别称，方便记忆。
    - 作用
      - 用来代替IP地址，方便记忆。
      - 用来代替IP地址，提高网络的灵活性。
  - 域名解析
    - 定义
    - 域名解析是将域名解析为IP地址的过程。
    - 作用
      - 方便记忆。
      - 提高网络的灵活性。
  - 端口号
    - 定义
    - 端口号是计算机上应用程序的唯一标识。
    - 作用
      - 用来区分计算机上不同的应用程序。
  - 端口号分类
    - 公有端口号：0~1023。
    - 私有端口号：1024~65535。
  - 端口号占用
    - 0~1023端口号已经被系统占用，用户不能使用。
    - 1024~65535端口号可以由用户自由使用。
  - 端口号的占用情况
    - 0~1023端口号已经被系统占用，用户不能使用。
    - 1024~65535端口号可以由用户自由使用。
  - 端口号的占用情况
    - 0~1023端口号已经被系统占用，用户不能使用。
    - 1024~65535端口号可以由用户自由使用。
***
### 5.网络通信协议
- #### <span style='color:red'>网络通信协议：</span>
  - 定义
  - 网络通信协议是通信双方必须共同遵守的规则，通信双方必须同时遵守才能完成数据交换。
  - 常见的网络通信协议
  - TCP/IP协议(<span style='color:orange'>java提供java\.net.Socket类来实现TCP通信</span>)
    - 定义
    - TCP/IP协议是Internet最基本的协议，由网络层的IP协议和传输层的TCP协议组成。
    - 作用
      - 规定了计算机在网络中如何传输数据。
    - 特点
      - 可靠，稳定。
      - 速度较慢。
  - UDP/IP协议(<span style='color:orange'>java提供java\.net.DatagramSocket类来实现UDP通信</span>)
    - 定义
    - UDP/IP协议是Internet最基本的协议，由网络层的IP协议和传输层的UDP协议组成。
    - 作用
      - 规定了计算机在网络中如何传输数据。
    - 特点
      - 不可靠，不稳定。
      - 速度较快。
      - 每次发送的数据最大不超过64kb。
***
### 6.Web服务器：Tomcat教程
- #### <span style='color:red'>Tomcat：</span>
  - 定义
  - Tomcat是Apache软件基金会（Apache Software Foundation）的Jakarta项目中的一个核心项目，由Apache、Sun和其他一些公司及个人共同开发而成。
  - 作用
    - 运行在服务器端，用于接收和处理客户端的请求，并返回响应给客户端。
  - 安装
    - 下载Tomcat
    - 解压Tomcat
    - 启动Tomcat
    - 访问Tomcat
  - 目录结构
    - bin：存放Tomcat的启动和关闭脚本。
    - conf：存放Tomcat的配置文件。
    - lib：存放Tomcat的依赖jar包。
    - logs：存放Tomcat的日志文件。
    - temp：存放Tomcat的临时文件。
    - webapps：存放Tomcat的Web应用。
    - work：存放Tomcat的工作目录。
  - 配置文件
    - server.xml：Tomcat的配置文件。
    - tomcat-users.xml：Tomcat的用户配置文件。
    - context.xml：Tomcat的上下文配置文件。
    - logging.properties：Tomcat的日志配置文件。
  - 启动和关闭Tomcat
    - 启动Tomcat
      - 进入Tomcat的bin目录，双击startup.bat文件。
    - 关闭Tomcat
      - 进入Tomcat的bin目录，双击shutdown.bat文件。
  - 访问Tomcat
    - 访问Tomcat的默认页面
      - 访问http://localhost:8080/
    - 访问Tomcat的Web应用
      - 访问http://localhost:8080/hello/index.jsp
  - 配置Tomcat
    - 配置Tomcat的端口号
      - 修改server.xml文件，修改Connector的端口号。
    - 配置Tomcat的用户
      - 修改tomcat-users.xml文件，添加用户。
    - 配置Tomcat的上下文
      - 修改context.xml文件，添加上下文。
    - 配置Tomcat的日志
      - 修改logging.properties文件，修改日志级别。
  - 部署Web应用
    - 创建Web应用
      - 创建一个名为hello的文件夹，并将其中的index.jsp文件修改为"Hello, World!"。
    - 部署Web应用
      - 将hello文件夹复制到Tomcat的webapps目录下。
    - 访问Web应用
      - 访问http://localhost:8080/hello/index.jsp
  - 配置Tomcat的虚拟主机
    - 创建虚拟主机
    - 修改server.xml文件，添加虚拟主机。
    - 重启Tomcat
  - 配置Tomcat的集群
    - 创建集群
    - 修改server.xml文件，添加集群。
    - 重启Tomcat
***

 