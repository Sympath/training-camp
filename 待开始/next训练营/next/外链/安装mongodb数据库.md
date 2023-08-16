
## 安装mongo数据库
接下来我们使用 curl 命令来下载安装mongo（指定版本为4.0.9）

### 进入 /usr/local
```
cd /usr/local
```
### 下载
```
sudo curl -O https://fastdl.mongodb.org/osx/mongodb-osx-ssl-x86_64-4.0.9.tgz
```

### 解压
```
sudo tar -zxvf mongodb-osx-ssl-x86_64-4.0.9.tgz
```

### 重命名为 mongodb 目录

```
sudo mv mongodb-osx-x86_64-4.0.9/ mongodb
```
### 实现全局命令

安装完成后，我们可以把 MongoDB 的二进制命令文件目录（安装目录/bin）添加到 PATH 路径中，这样就能在任意位置都能执行`mongod`命令了。

配置到脚本文件中，可以参考[mac配置环境变量]()一文，下面是需要配置到配置文件的内容

```
export PATH=/usr/local/mongodb/bin:$PATH21
```
### 创建日志及数据存放的目录

##### 先新建目录

数据存放路径

```
sudo mkdir -p /usr/local/var/mongodb
```
日志文件路径

```
sudo mkdir -p /usr/local/var/log/mongodb
```
##### 再赋予文件权限

接下来要确保当前用户对以上两个目录有读写的权限，这里以上 wzyan 是我电脑上的用户，你这边需要根据你当前对用户名来修改（可以通过`whoami`查看）。

```
sudo chown wzyan /usr/local/var/mongodb
sudo chown wzyan /usr/local/var/log/mongodb
```

至此，我们就完成了mongodb的安装啦！这些是初次安装才需要做的动作，下面的操作是以后日常中也需要使用到的。

## 启动mongodb

接下来我们使用以下命令在后台启动 mongodb：

```
mongod --dbpath /usr/local/var/mongodb --logpath /usr/local/var/log/mongodb/mongo.log --fork
```
- --dbpath 设置数据存放目录
- --logpath 设置日志存放目录
- --fork 在后台运行
如果不想在后端运行，而是在控制台上查看运行过程可以直接设置配置文件启动：

```
mongod --config /usr/local/etc/mongod.conf
```
## 查看结果
查看 mongod 服务是否启动

```
ps aux | grep -v grep | grep mongod
```
使用以上命令如果看到有 mongod 的记录表示运行成功。

## 可视化工具 Navicat for mongoDB
### Navicat for mongoDB破解版资源获取

mongodb的可视化工具很多，这里推荐Navicat for mongoDB，可以自己去找找破解版，如果实在找不到，关注公众号【南方小菜的小屋】，发送【Navicat for mongoDB】获取
进行下载安装，然后一直next即可。

### 新建连接
可视化界面操作，没什么可讲的，默认端口27017，填写下连接名称就好；然后在里面新建自己的数据库，再将库名写在`config.js`中即可。
secret属于加盐字段（加密用的），随意填