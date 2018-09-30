## 打赏
### 网站演示  https://yf.abbeyok.com
## 需要环境

    推荐 CentOS7 + Python2.7
    一个域名
    注册有赞和有赞云个人开发者
    创建店铺并获取密钥信息

## 准备
1. 提前注册好 [有赞](https://www.youzan.com/)，注册好有赞之后，再注册 [有赞云](https://console.youzanyun.com/register) 个人开发者
2. 直接 [扫码](https://h5.youzan.com/v2/index/wxdpc) 安装有赞微小店，然后在手机上用手机注册店铺，支付会走这个店铺的订单系统的，店铺注册后这个微小店基本就不用管了。
3. 点击 创建有赞云应用，选择【自用式】，然后选择【有赞微商城】。
4. 应用授权: 创建完店铺后，再登录到有赞云控制台创建自用型应用并授权刚创建的店铺（是微小店哦）
5. 开启有赞推送消息

开启之后，有赞才会在支付成功之后回调信息到你的服务器

- 开启地址: https://console.youzanyun.com/application/msg_config
- 推送网址格式为：http://domain/order_msg
- 推送配置勾选：交易消息V3-交易支付

## 安装



### 安装 Nginx

```bash
yum -y install wget screen curl python git vim
wget http://mirrors.linuxeye.com/lnmp-full.tar.gz
tar xzf lnmp-full.tar.gz
cd lnmp
screen -S lnmp
./install.sh
```

### 安装 pip


```bash
curl https://bootstrap.pypa.io/get-pip.py -o get-pip.py
python get-pip.py

```

### 绑定域名

```bas
./vhost.sh 中添加即可
```

### 部署源码
```bash
git clone https://github.com/biezhi/yaofan
# 安装依赖
cd yaofan
pip install -r requirement.txt
```

### 修改配置
```bash
vim app/youzan/yz_config.py
```
将下面几个配置修改掉：进入 [有赞云后台](https://console.youzanyun.com/application/setting) 分别获取：

- client_id
- client_secret
- shopid

### 修改系统配置信息

修改 ```config.py``` 的``` SQLALCHEMY_DATABASE_URI```参数
 - 如果使用 MySQL 备注下面这行
``` 
SQLALCHEMY_DATABASE_URI = 'sqlite:///' + os.path.join(basedir, 'data.sqlite')  # sqlite3
```
 - 如果使用sqlite3，则不需要修改内容

### 初始化数据库

```bash
python run.py deploy
```

运行：```gunicorn -w4 -b 0:35000 run:app```然后访问你的ip:35000试试

### 绑定域名
- 首先域名绑定到你的服务器ip
- 修改 Nginx 配置
- 添加反向代理配置
```bash
location / {
​    root /root/yaofan;
​    proxy_pass http://127.0.0.1:35000;
​    proxy_read_timeout 300;
​    proxy_connect_timeout 300;
​    proxy_redirect     off;

    proxy_set_header   X-Forwarded-Proto $scheme;
    proxy_set_header   Host              $http_host;
    proxy_set_header   X-Real-IP         $remote_addr;
}
```
### 重新加载配置
```bash
service nginx reload
```
### 设置开机启动
- 修改源码目录的 supervisord.conf，主要修改源码目录和端口号
- 执行以下命令
```bash
pip install supervisor
echo 'supervisord -c /root/yaofan/supervisord.conf' >> /etc/rc.d/rc.local
chmod +x /etc/rc.d/rc.local
```
重启：```bash supervisorctl reload```

### 关闭防火墙
```bash
systemctl stop firewalld.service    #停止firewall
systemctl disable firewalld.service #禁止firewall开机启动
```
### 修改时区

如果你发现站点日志的时间更新有问题，检查下你服务器当地时间，使用命令date -R查看，再使用命令修改成上海时间即可。
```bash
cp /usr/share/zoneinfo/Asia/Shanghai /etc/localtime
```
## 教程来源 
- [biezhi](https://gist.github.com/biezhi/34b70840bb295920e1fb34d5701a1014#%E5%B0%8F%E5%93%A5%E5%93%A5%E4%B8%80%E8%B5%B7%E6%9D%A5%E8%A6%81%E9%A5%AD)
