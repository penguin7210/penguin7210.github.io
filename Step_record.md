![](https://gimg2.baidu.com/image_search/src=http%3A%2F%2Fwww.zzvips.com%2Fuploads%2Fallimg%2F200408%2F202UaW2-2.png&refer=http%3A%2F%2Fwww.zzvips.com&app=2002&size=f9999,10000&q=a80&n=0&g=0n&fmt=auto?sec=1673159394&t=8cac6816b69e436e78e4668bb2f35d9c)
这是 **penguin7210** 的**Linux**大作业。
## 2 Linux大作业完成步骤
### 第一题
创建目录，将安装的软件存放在此目录下
```
mkdir test
cd test
```

下载git安装包
```
wget https://github.com/git/git/archive/refs/tags/v2.31.1.tar.gz
```
下载依赖项
```
yum install curl-devel expat-devel gettext-devel openssl-devel zlib-devel gcc perl-ExtUtils-MakeMaker
```
 
安装依赖完成

解压git安装包
```
tar -zxvf v2.31.1.tar.gz
```

查看目录可发现安装包已解压并且存在该目录下
```
ls
```
编译源码
```
make prefix=/usr/local/git all
```
 
安装git
```
make prefix=/usr/local/git install
```
安装git命令
```
yum install -y git
```
git安装成功
```
git --version
```
配置环境变量
```
#vim/etc/profile

Path=$Path:/usr/local/git/bin
export PATH
```
 ![]()
配置git：配置用户名、邮箱、Windows提交到Linux上是否自动转换换行符、字符集
```
git config --global user.name XX
git config --global user.email xx
git config --global core.autocrlf false
git config --global gui.encoding utf-8
```
创建SSH Key
```
ssh-keygen -t rsa -C "email"
```
生成密钥
添加Key至GitHub
测试是否与GitHub相连
```
ssh -T git@github.com
```
在GitHub上新建一个仓库，输入仓库名、描述信息、勾选公开仓库，创建README文件，点击Create repository创建仓库即可。
创建完成

克隆远程仓库到本地
```
git clone git@github.com:penguin7210/mygithub.git
```
创建一个文件，添加该文件到暂存区，查看文件位置状态，最后将文件推送到仓库。
```
vim test2.py
git add test.2py
git status
git commit -m "seconf file"
git push
```
### 第二题：

在一个终端里编写一个服务端的代码，创建服务端
 ```
vim server.c
```
 ```
#include <unistd.h>
#include <sys/types.h>         
#include <sys/socket.h>
#include <arpa/inet.h>
#include <string.h>
#include <stdio.h>

int main()
{
  //1.创建一个socket文件，也就是打开一个网络通讯端口,类型是IPV4（AF_INET）+TCP（SOCK_STREAM）
  int serv_sock = socket(AF_INET, SOCK_STREAM,0);
  
  //2.绑定服务器ip和端口到这个socket
  struct sockaddr_in serv_addr;//这里因为是ipv4，使用的结构体是ipv4的地址类型sockaddr_in
  memset(&serv_addr, 0, sizeof(serv_addr));//先清空一下初始的值，写上地址和端口号，可以用bzero(&serv_addr, sizeof(serv_addr));
  serv_addr.sin_family = AF_INET;
  serv_addr.sin_addr.s_addr = inet_addr("127.0.0.1");//本机ip环回地址，这里还可以使用inet_pton函数进行地址转换
  serv_addr.sin_port = htons(8899);//随意选了一个端口8899
  bind(serv_sock, (struct sockaddr*)&serv_addr, sizeof(serv_addr));
  
  //3.将socket设置为监听状态
  listen(serv_sock,128);//设置最大连接数为128
  
  //4.准备接收客户端的请求连接,这里的步骤可以重复进行，接收多个客户端的请求
  while(1){
    //接收客户端的请求连接后，返回一个新的socket（clnt_sock）用于和对应的客户端进行通信
    struct sockaddr_in clnt_addr;//作为一个传出参数
    socklen_t clnt_addr_size = sizeof(clnt_addr);//作为一个传入+传出参数
    int clnt_sock = accept(serv_sock, (struct sockaddr*)&clnt_addr, &clnt_addr_size);

    //5.读取客户端发送来的数据
    char recv_buf[256];
    char send_buf[512]="hello ";
    int len = read(clnt_sock,recv_buf,sizeof(recv_buf)-1);
    recv_buf[len] = '\0';//字符串以“\0”结尾
    
    //6.打印出客户端发来的消息
    printf("客户端发来的：%s\n",recv_buf);
    
    //7.加上hello处理后返回给客户端
    strcpy(send_buf+strlen("hello "),recv_buf);//注意这里不能用sizeof，要用strlen，不然包含了‘\0’，后面的jingjing就打印不出来了。
    write(clnt_sock,send_buf,sizeof(send_buf));
    
    //8.关闭客户端连接
    close(clnt_sock);
    break;

  }
  //9.关闭服务端监听的socket
  close(serv_sock);
  return 0;

}

```
在另一个终端编写一个客户端文件，编写客户端代码
```
vim client.c
```
```
#include <unistd.h>
#include <sys/types.h>         
#include <sys/socket.h>
#include <arpa/inet.h>
#include <string.h>
#include <stdio.h>

int main()
{
    //1.创建socket，用于和服务端通信
    int sock = socket(AF_INET, SOCK_STREAM, 0);

    //2.向服务端发起请求连接
    struct sockaddr_in serv_addr;//首先要指定一个服务端的ip地址+端口，表明是向哪个服务端发起请求
    memset(&serv_addr, 0, sizeof(serv_addr));
    serv_addr.sin_family = AF_INET;
    serv_addr.sin_addr.s_addr = inet_addr("127.0.0.1");//注意，这里是服务端的ip和端口
    serv_addr.sin_port = htons(8899);
    connect(sock, (struct sockaddr*)&serv_addr, sizeof(serv_addr));
    
    //3.向服务端发送消息
    char send_buf[256] = "jingjing";
    char recv_buf[512];
    write(sock,send_buf,sizeof(send_buf));
    
    //4.接收服务端发来的消息
    int len = read(sock,recv_buf,sizeof(recv_buf)-1);
    recv_buf[len] = '\0';
    printf("收到服务端的返回：%s\n",recv_buf);
    
    //5.关闭socket
    close(sock);
    return 0;

}

```
将server.c预处理、汇编、编译并链接形成可执行文件serv，开启服务端，等待客户端连接
 ```
gcc server.c -o serv
./serv
```
将client.c预处理、汇编、编译并链接形成可执行文件cln，编译客户端的代码，接收到服务端发来的信息
  ```
gcc client.c -o cln
./cln
```
同时服务端会受到客户端返回的信息
 

第三题：
安装hugo
```
wget https://github.com/gohugoio/hugo/releases/download/v0.108.0/hugo_0.108.0_Linux-64bit.tar.gz
```
解压hugo安装包
```
tar -zxf hugo_0.108.0_Linux-64bit.tar.gz -C hugo

ls
```
可查询到hugo版本，即安装成功
```
hugo version
```
在当前目录执行命令创建blog站点
 ```
hugo new site myblog
```
到hugo主题官网下载一个主题，找到一个需要的主题点击
 
 
git clone命令克隆主题到文件
```
git clone https://github.com/adityatelange/hugo-PaperMod themes/PaperMod --depth=1
cd myblog
```
修改config.toml配置文件，改名为config.yml
 ```
vim config.toml
mv config.toml config.yml
```
添加一篇markdown文章，文章目录在content/post
 ```
hugo new post/blog.md
```
配置hugo的本地访问
```
systemctl start httpd 
systemctl status httpd
firewall-cmd --add-service=http --permanent
firewall-cmd --add-port=1313/tcp --permanent
firewall-cmd --reload
firewall-cmd --list-ports

```
开启hugo后，命令启动时会默认将地址绑定在虚拟机的127.0.0.0上无法通过本机浏览器访问， 则应该写命令将地址修改为"0.0.0.0",并设置访问的url,这样即可正常使用
url为地址+端口：http: 192.168.56.102:1313/，启动的hugo项目
  ```
hugo server theme=hyde --buildDrafts --bind="0.0.0.0" --baseURL=http:Linux.address:1313
```
将hugo部署到github上

其中实验过程中鉴权失败的可能原因之一是访问token过期或者没有设置token。通过在setting -> Developer settings -> Personal access tokens 查看，而且在输入的密码是token的验证码而不是GitHub的登陆密码
  ```
    cd public
    git init		
    git add .		
    git commit -m "First_post"
	git remote add origin https://github.com/username/username.github.io.git
	git push -u origin master
```
 
在config.yml中设置网址，最后修改完后即可： 
部署到GitHub上:

**http://penguin7210.github.io**
 

