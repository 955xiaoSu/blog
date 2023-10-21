# 搭建个人网站



以下内容已按搭建网站的过程排序

#### 方法一: 服务器 + 自我配置

* 购买服务器与域名：
  * 具体购买请参照个人需求自行选择 可以前往[腾讯云](https://cloud.tencent.com/act/2022season?fromSource=gwzcw.5729214.5729214.5729214\&utm\_medium=cpc\&utm\_id=gwzcw.5729214.5729214.5729214\&gclid=EAIaIQobChMIqZOdq-qt9gIVsyCtBh2M4wWiEAAYASAAEgJ\_l\_D\_BwE)/[阿里云](https://cn.aliyun.com/?utm\_content=se\_1010304256\&gclid=EAIaIQobChMI8tfFg-ut9gIVuiCtBh19mg53EAAYASAAEgKGFvD\_BwE)购买完成后根据相关指导进行备案
* 进行域名的备案与解析：
  * 域名的备案可参考各服务平台的文件, 购买完服务器后可以获得公网IP，域名的解析使域名变成可以登录的网址。
  * IP地址是网络上标识站点的数字地址，为了方便记忆，采用域名来代替IP地址标识站点地址。域名解析就是域名到IP地址的转换过程。域名的解析工作由DNS服务器完成。域名解析也叫域名指向、服务器设置、域名配置以及反向IP登记等等。说得简单点就是将好记的域名解析成IP，服务由DNS服务器完成，是把域名解析到一个IP地址，然后在此IP地址的主机上将一个子目录与域名绑定。域名解析即把公网IP配置到域名中即可(可在相关服务商的云上完成操作)
  * 判断是否完成域名解析: 打开cmd->ping 域名(如sbk825.cn)->返回时间即解析成功
    * ping命令是发送4/5个ICMP数据包，根据返回的数量来确定网络的通断和延迟。ICMP数据包是一种网络通信数据包。ICMP是“Internet Control Message Protocol”（Internet控制消息协议）的缩写。它是TCP/IP的一个子协议，用于在IP主机、路由器之间传递控制消息。ICMP控制包是指用于探查网络通不通、主机是否可达、路由是否可用等网络问题的消息。
* 配置个人网站：
  * 如果想省事，也可以直接花money，借助[wordpress](https://wordpress.com/start/domains) / [appnode](https://www.appnode.com/)直接一键式建站，则可自动跳过以下所有步骤，如果并不愿意采取这种方式，请您移目下方。
  * 参考[LNMP](https://bkcat.cn/index.php/page/5/lnmp.org), LNMP指的是Linux系统下Nginx+MySQL+PHP这种网站服务器架构。
  *   安装完lnmp后，接下来的步骤可参考，readme文件/或如下步骤

      * 安装wget命令: 使用ssh命令远程登陆服务器, 执行apt -get install wget命令。确认是否成功安装的命令:

      dpkg -l | grep wget

      * 执行screen S -lnmp
      * 执行wget [http://soft.vpser.net/lnmp/lnmp1.8.tar.gz](http://soft.vpser.net/lnmp/lnmp1.8.tar.gz) -cO lnmp1.8.tar.gz && tar zxf lnmp1.8.tar.gz && cd lnmp1.8 && ./install.sh lnmp
  * 成功安装Nginx+MySQL+PHP后，根据喜好选择博客系统[typecho](https://typecho.org/)/[wordpress](https://wordpress.org/)
    * 以下对typecho的安装进行说明。
    * 进入[typecho](https://github.com/typecho/typecho/releases/tag/v1.2.0-beta.2)安装typecho.zip，而后解压。将解压好的文件通过scp -r 上传至网站的根目录。而后在浏览器上访问自己的网站，应该就可以开始typecho的配置啦！接着就一步一步跟着其指导往下做即可。
  * 如果不想采用LNMP部署的话，也可尝试docker部署，作者能力有限无法进行具体过程说明。可参考[docker](http://hub.docker/)。

#### 方法二：借助现有的平台，免费建站:

* [凡科建站](https://jz.fkw.com/?\_ta=9240&\_kw=258240)等一键式建站，可在知乎中查找到许多类似的网站。

\
