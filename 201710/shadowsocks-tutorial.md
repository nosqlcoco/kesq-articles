### Digital Shadowsocks搭建VPS科学上网

#### 前言
６月22号国内知名VPN团队宣布永久关停GreenVPN，再到10月份lantern长达几个星期不能使用，日常工作习惯了用google，不能翻墙对工作效率影响挺大，决心自己搭建一个VPN Server, 使用digitalocean主机，shadowsocks作为VPN Server。

#### 申请digitalocean主机
搭建VPN Server 只能使用非大陆区域的主机，我选择digitalocean，最便宜的主机５美刀/月，相对国内某云主机便宜多了。但是５美刀也是钱，查找了一些资料，github有一个针对学生群体的优惠计划[Get the Student Developer Pack](https://education.github.com/pack), 绑定@edu教育邮箱，并身份认证后可以获得digitalocean50美刀优惠券(还有别的几十项其他优惠券)。

问题是怎么弄到一个edu教育邮箱，后来在某宝上搜索“教育邮箱”，还真有人做github学生认证，花了39块钱通过了github学生认证，拿到了兑换码。

在[digitalocean](https://m.do.co/c/ba3c78cc4aca)注册一个账号，注册成功后:
- 绑定信用卡(必须)
- 找到“setting”>"Billing">"Promo code"，输入兑换码点击"Apply Code"即可获得50美刀优惠券。

创建主机(Droplet)
- 选择Ubuntu16操作系统
- 选择５美刀/月套餐，512MB/1 CPU, 20G SSD, 1000GB 流量，这个配置够用。
- 选择数据中心，我选择“San Francisco 2”，知乎上看人评论，北美网速稳定，欧洲次之，新加坡最差。
- 添加SSH keys，使用SSH登录droplet,必须要配置SSH key，并且只能在创建之前配置，创建后配置不起作用，报“Permission denied (publickey)”错误，这是个坑。
- 最后自定义hostname

这样就成功创建了droplet。

##### 安装shadowsocks server
使用ssh登录droplet, 安装shadowsocks步骤如下：
```
sudo apt-get install python-pip
pip install shadowsocks
```
安装成功后创建/etc/shadowsocks.json配置文件
```json
{
    "server":"my_server_ip",
    "server_port":8388,
    "local_address": "127.0.0.1",
    "local_port":1080,
    "password":"mypassword",
    "timeout":300,
    "method":"aes-256-cfb",
    "fast_open": false
}
```
my_server_ip是服务器ip地址，mypassword是vpn连接时的密码。

配置完后直接启动shadowsocks server
```
ssserver -c /etc/shadowsocks.json
```

有了服务器ip和vpn连接密码，接下来使用shadowsocks client测试搭建的vpn。

#### Ubuntu shadowsocks client：
安装shadowsocks-qt5连接
```
sudo add-apt-repository ppa:hzwhuang/ss-qt5
sudo apt-get update
sudo apt-get install shadowsocks-qt5
```
安装成功后打开shadowsocks-qt5
![Alt text](https://github.com/nosqlcoco/public-images/blob/master/blog_nosqlcoco/-ssclient1.png?raw=true)
填写服务器地址，密钥，加密方式改为AES-256-CFB，点击ok。
打开电脑“系统设置”>"网络">"网络代理"，代理方法选择"手动"，在socks主机一栏填写127.0.0.1，端口号为1080，这样就配置好了。
![Alt text](https://github.com/nosqlcoco/public-images/blob/master/blog_nosqlcoco/-ssclient2.png?raw=true)
![Alt text](https://github.com/nosqlcoco/public-images/blob/master/blog_nosqlcoco/-ssclient3.png?raw=true)

##### Android shadowsocks 客户端连接
在github上下载ss客户端apk（https://github.com/shadowsocks/shadowsocks-android/releases）在手机安装完后打开新增一个配置文件(点击app顶部加号>手动配置)
![Alt text](https://github.com/nosqlcoco/public-images/blob/master/blog_nosqlcoco/-webwxgetmsgimg%207.jpg?raw=true)
配置服务器地址和密码并保存。
最后点击连接vpn
![Alt text](https://github.com/nosqlcoco/public-images/blob/master/blog_nosqlcoco/-ssclient5.jpg?raw=true)

个人测试看YouTube网速过得去，日常Google没问题，比一年200来块钱的第三方VPN便宜不少，50美刀可以免费用10个月，只花费了39元github学生认证，还拿到了github私有仓库无限制使用权限.
