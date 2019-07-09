# FAQ

{{indexmenu_n>10}}

## 加速配置和加速线路的关系

1、带宽共享功能：一个加速线路可以被多个加速配置绑定，这些加速配置共享加速线路的带宽；  
2、一个加速配置可以绑定多个加速线路。  
3、删除加速配置不会影响加速线路，加速线路仍存在；  
4、若加速线路上绑定了加速配置，该加速线路不能删除；若加速线路上无任何加速配置，该加速线路可以删除。  

## 为什么看到大量固定的IP地址在访问源站服务器？

这些地址是加速集群地址，分为两个作用：  
1、传输业务流量（可通过toa模块获取访问者真实IP）  
2、对源站服务器做健康检查  
对源站服务器做健康检查是通过TCP三次握手的原理来探测源站可用性，为短连接。

## 如何获取访问者真实IP？

由于经过加速，在日志中看到的访问者IP全部变为PathX的出口IP。 如果需要获取真实的客户端IP,
可以在您的源站服务器上加载UCloud专有的内核模块，让应用直接获取到源IP，这时候，再去查看日志，就是访问者的真实IP了。  
**Linux系统**  
64位的linux系统可运行"modprobe toa"尝试加载模块，成功后无需其他操作。  
如提示未找到该模块，可按如下步骤进行手工编译与加载：

1\.
查看当前内核版本号，确认依赖"kernel\\-devel、kernel\\-headers"是否安装以及版本号是否与内核一致('uname
\\-r && rpm \\-qa |egrep 'kernel\\-devel|kernel\\-headers')：  
\\- 若一致，跳过步骤2，进行toa模块的编译安装  
\\- 若不一致，如下图：  
![](/images/toa_201810301429.png) 需要卸载后进行步骤2操作(rpm \\-e
\\-\\-nodeps kernel\\-devel kernel\\-headers)  
\\- 若未安装依赖，如下图： ![](/network/pathx/toa_201810301432.png)

  
2\. yum搜索是否有与当前内核版本对应的‘kernel-devel、kernel-headers’包  
\\- 若有，则安装对用版本（yum install pkgname-version.x86\\\_64）  
\\- 若无，如下图  
![](/images/toa_201810301443.png)  
则打开网站http://rpm.pbone.net，点击左侧SEARCH标签，填入包名+版本号（如：kernel-devel-3.10.0-693.11.6.el7.x86\\\_64），选择对应的系统发行版本（此处为CentOS7），点击搜索  
![](/images/toa_201810301447.png) 搜索结果：
![](/images/toa_201810301449.png) 或使用谷歌用关键字“rpm.pbone.net
kernel-devel-3.10.0-693.11.6.el7.x86\\\_64”搜索
![](/images/toa_201810301450.png) 下载后rpm方式安装，kernel-headers的安装同理
![](/images/toa_201810301452.png) 确认安装结果（'uname -r && rpm -qa
|egrep 'kernel-devel|kernel-headers'），如下图：
![](/images/toa_201810301453.png)

  
3\. 下载linux通用版的源码包，该版本支持Centos 6.9和Centos 7、ubuntu
14.04等绝大多数的linux发行版：  
国内：  
`wget http://pathx.ufile.ucloud.com.cn/linux_toa.tar.gz` 国外：  
`wget http://toa.ufile.ucloud.com.cn/linux_toa.tar.gz`

  
4\. 编译加载  
``yum install -y gcc
tar -zxvf linux_toa.tar.gz
cd linux_toa
make
mv toa.ko /lib/modules/`uname -r`/kernel/net/netfilter/ipvs/toa.ko
insmod /lib/modules/`uname -r`/kernel/net/netfilter/ipvs/toa.ko
`` toa模块安装验证如下（lsmod |grep toa）：
![](/images/toa_201810301534.png)

  
5\. 添加开机模块自动加载  
``echo "insmod /lib/modules/`uname -r`/kernel/net/netfilter/ipvs/toa.ko"
>> /etc/rc.local
``  
\* **nginx 环境下**，直接在nginx 日志中查看真实访问者地址 日志路径： /var/log/nginx/access.log

![](/images/nginx_真实地址.png)

  - **apache环境下**，直接在apache日志中查看真实访问者地址  

日志路径：/etc/httpd/logs/access\_log ![](/network/pathx/apache获取真实地址.png)

  - 其他web配置环境， 采用同样方法在相关web 日志文件中检查即可  

## 如何查看PathX的出口IP？

请在线路详情页的基本信息模块中查看

## 非UCloud服务器是否可以使用全球动态加速？

可以，只要是公网路由可达的服务器即可。

## 网站是否需要备案？

若加速区域为国内，那么最终对外提供加速服务的域名需要在中华人民共和国工业和信息化部完成相关备案。

## 全球动态加速和CDN加速有区别吗？

CDN是以资源缓存的方式加速，一般为静态资源。全球动态加速偏重于链路优化。  
全球加速使用UCloud专线环路，您只需选择需要加速的区域，和业务服务器所在区域，我们会智能调度出较优的路由完成加速。

## https网站是否可以使用？

可以使用，全球动态加速是以tcp透传的方式。在控制台配置 TCP 443端口，证书仍然部署在您的业务服务器上，不需要在控制台做其他设置。

## 什么是多地接入？

以中国(多地)到香港的加速为例，创建加速后，华北、华东、华南的用户访问同一个加速域名，但解析出的IP不同，这些IP分别对应pathx在三个地区的入口，也即，不同区域的用户访问pathx加速域名会解析出离用户最近的加速入口IP。

## 源站可以正常访问，加速cname无法正常访问

用curl测试一般会报"curl: (56) Failure when receiving data from the peer"  
1\. 检查源站是否有安全策略设置，如阿里云的安骑士，若有，可以咨询售后获取pathx ip加入白名单  
2\. 检查系统参数设置

    net.ipv4.tcp_timestamps = 1
    net.ipv4.tcp_tw_recycle = 0
    net.ipv4.tcp_tw_reuse = 1

开启tw\_resuse足够进行TCP连接的回收，tw
recycle由于设计的时间较为早期，并没有考虑NAT技术在如今公网已经普及，会导致经过NAT（诸如网吧、4G、WIFI）用户部分的连接失败。当前这个参数已经基本废弃

## 源站是否可以修改？

源站不支持修改，可以新建加速配置，添加新的源站，切换DNS解析到新的cname后删除旧的加速配置

## pathX欠费回收后是否可以找回？

回收后无法找回

## pathX是否可配置带宽告警？

可在加速线路详情页"告警模板"处配置

## pathX的限流措施

pathX按购买带宽限流，当实时带宽（出向带宽与入向带宽的最大值）超过购买带宽时会触发限流，限流会引起丢包率上升，连接中断等现象发生，当实时带宽低于购买带宽时限流会自动解除
