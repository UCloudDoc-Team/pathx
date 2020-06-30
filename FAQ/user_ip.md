# 获取访问者IP

## 如何获取访问者真实IP？

由于经过加速，在日志中看到的访问者IP全部变为PathX的出口IP。 如果需要获取真实的客户端IP,
可以在您的源站服务器上加载UCloud专有的内核模块，让应用直接获取到源IP，这时候，再去查看日志，就是访问者的真实IP了。  

**Linux系统**  
64位的linux系统可运行"modprobe toa"尝试加载模块，成功后无需其他操作。  
如提示未找到该模块，可按如下步骤进行手工编译与加载：

1.查看当前内核版本号，确认依赖"kernel -devel、kernel -headers"是否安装以及版本号是否与内核一致('uname
-r && rpm -qa |egrep 'kernel -devel|kernel -headers')：  
若一致，跳过步骤2，进行toa模块的编译安装；若不一致，如下图：  
![](/images/toa_201810301429.png) 需要卸载后进行步骤2操作(rpm -e
--nodeps kernel-devel kernel-headers)  
若未安装依赖，如下图： ![](/images/toa_201810301432.png)
  
2.yum搜索是否有与当前内核版本对应的kernel-devel、kernel-headers
若有，则安装对用版本（yum install pkgname\-version\.x86\_64）  
若无，如下图  
![](/images/toa_201810301443.png)  
则打开网站 http://rpm.pbone.net 点击左侧SEARCH标签，填入包名+版本号（如：kernel-devel-3.10.0-693.11.6.el7.x86_64），选择对应的系统发行版本（此处为CentOS7），点击搜索  
![](/images/toa_201810301447.png) 搜索结果：
![](/images/toa_201810301449.png) 或使用谷歌用关键字“rpm.pbone.net
kernel-devel-3.10.0-693.11.6.el7.x86_64”搜索
![](/images/toa_201810301450.png) 下载后rpm方式安装，kernel-headers的安装同理
![](/images/toa_201810301452.png) 确认安装结果（'uname -r && rpm -qa
|egrep 'kernel-devel|kernel-headers'），如下图：
![](/images/toa_201810301453.png)

3.下载linux通用版的源码包，该版本支持Centos 6.9和Centos 7、ubuntu
14.04等绝大多数的linux发行版：  

国内：  
```wget http://pathx.ufile.ucloud.com.cn/linux_toa.tar.gz```

国外：  
```wget http://toa.ufile.ucloud.com.cn/linux_toa.tar.gz```

  
4.编译加载  
```
yum install -y gcc
tar -zxvf linux_toa.tar.gz
cd linux_toa
make
mv toa.ko /lib/modules/`uname -r`/kernel/net/netfilter/ipvs/toa.ko
insmod /lib/modules/`uname -r`/kernel/net/netfilter/ipvs/toa.ko

```

toa模块安装验证如下（lsmod |grep toa）：

![](/images/toa_201810301534.png)

  
5.添加开机模块自动加载  

```

echo "insmod /lib/modules/`uname -r`/kernel/net/netfilter/ipvs/toa.ko"
>> /etc/rc.local

```

**nginx 环境下**，直接在nginx 日志中查看真实访问者地址 日志路径： /var/log/nginx/access.log

![](/images/nginx_真实地址.png)

**apache环境下**，直接在apache日志中查看真实访问者地址  

日志路径：/etc/httpd/logs/access_log 

![](/images/apache获取真实地址.png)

  - 其他web配置环境， 采用同样方法在相关web 日志文件中检查即可  

## 安装了toa 仍然无法查看真实客户端IP

toa原理是从tcp包中取出option字段，解析出真实客户端IP，最后通过内核钩子函数完成替换。如果转发过程中出现了tcp连接截断的情况，如在rs前使用七层负载均衡产品，就会导致安装toa 也获取不到真实客户端IP：

1）client -------> pathx 4层转发 --------- tcp packet (option字段包含：客户端IP ) --------> 7层负载均衡 ----------tcp packet (option字段不再包含 客户端IP) --------> 源站RS（安装toa 不能获取客户端IP）

2）client -------> pathx 4层转发 --------- tcp packet (option字段包含：客户端IP ) --------> 源站RS（安装toa 可以获取客户端IP）

3）client--------> pathx 4层转发 --------- tcp packet (option字段包含：客户端IP ) --------> 4层负载均衡（ulb4开启报文转发 或 AWS NLB开启保留源IP） ----------tcp packet (option字段包含 客户端IP) --------> 源站RS（安装toa 可以获取客户端IP）


如果使用http协议场景，七层转发 不需要安装toa模块就可以获取真实客户端IP：

4）client--------> pathx 7层转发 ------X-Forwarded-For---------> 各类LB --------> 源站RS（从http header获取客户端IP）

5）client--------> pathx 7层转发 ------X-Forwarded-For---------> 源站RS（从http header获取客户端IP）
