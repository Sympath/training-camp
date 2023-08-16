## 理论篇

### 加载顺序

Mac系统的环境变量，加载顺序为：

-  /etc/profile
- /etc/paths
- ~/.bash_profile
- ~/.bash_login
- ~/.profile
- ~/.bashrc或~/.zshrc

### 作用域解读

- /etc/profile和/etc/paths是系统级别的，系统启动就会加载
- 后面几个是当前用户级的环境变量。后面3个按照从前往后的顺序读取
- 如果/.bash_profile文件存在，则后面的几个文件就会被忽略不读了
- 如果/.bash_profile文件不存在，才会以此类推读取后面的文件
- ~/.bashrc没有上述规则，它是bash shell打开的时候载入的。

### PATH语法

```ruby
#中间用冒号隔开
export PATH=$PATH:<PATH 1>:<PATH 2>:<PATH 3>:------:<PATH N>
```

### 上述文件的科普

- /etc/paths （全局建议修改这个文件 ）
   编辑 paths，将环境变量添加到 paths文件中 ，一行一个路径
   Hint：输入环境变量时，不用一个一个地输入，只要拖动文件夹到 Terminal 里就可以了。
- /etc/profile （建议不修改这个文件 ）
   全局（公有）配置，不管是哪个用户，登录时都会读取该文件。
- /etc/bashrc （一般在这个文件中添加系统级环境变量）
   全局（公有）配置，bash shell执行时，不管是何种方式，都会读取此文件
- .profile 文件为系统的每个用户设置环境信息,当用户第一次登录时,该文件被执行.并从/etc/profile.d目录的配置文件中搜集shell的设置
   **使用注意**：如果你有对/etc/profile有修改的话必须得重启你的修改才会生效，此修改对每个用户都生效。
- ./bashrc 每一个运行bash shell的用户执行此文件.当bash shell被打开时,该文件被读取.
   **使用注意** 对所有的使用bash的用户修改某个配置并在以后打开的bash都生效的话可以修改这个文件，修改这个文件不用重启，重新打开一个bash即可生效。
- ./bash_profile 该文件包含专用于你的bash shell的bash信息,当登录时以及每次打开新的shell时,该文件被读取.（每个用户都有一个.bashrc文件，在用户目录下）
   **使用注意** 需要需要重启才会生效，/etc/profile对所有用户生效，~/.bash_profile只对当前用户生效。

### 重点注意

##### 触发加载

修改完配置文件一定要触发生效，比如我们常见的`.zshrc`文件修改，就需要执行如下命令触发重新加载

```
source ./.zshrc
```



## 操作篇

### 全局设置：以mysql为例

创建对应文件

```undefined
sudo touch /etc/paths.d/mysql
```

用 vim 打开这个文件

```undefined
sudo vim /etc/paths.d/mysql
```

编辑该文件，键入路径并保存，关闭该 Terminal 窗口并重新打开一个，就能使用 mysql 命令了

```bash
/usr/local/mysql/bin
```

1. $ source 相应的文件 生效配置环境

### 单个用户设置

打开用户级别的环境变量配置文件，以zshrc为例，如果是bashrc，修改`.bashrc`即可

```bash
vim ~/.zshrc 
```

把上述代码添加到~/.bash_profile上。

```
export PATH=/opt/local/bin:/opt/local/sbin:$PATH
```

生效新的配置

```
source ~/.zshrc
```



### 查看当前已经生效的环境变量

```bash
echo $PATH
```

