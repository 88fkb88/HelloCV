# Linux终端的学习-Part 2
### 任务目标：
1. 掌握⽂件系统结构：
   - 根⽬录 / 、 /home 、 /etc 、 /var 等⽬录作⽤；
   - 绝对路径 vs 相对路径；
2. 掌握基础服务管理（ systemd ）与⽹络配置
3. 掌握常⽤命令⾏⼯具
   - vim 、 tmux 、 grep 、 awk 、 sed ；
4. 感悟
5. 鸣谢



### 一、掌握文件系统结构
笔者认为Linux虽然与Windows有许多不同之处，但是对于文件的存储应该是一致的，都是一种类似于树状结构的存储方法。笔者将会针对每一种命令分为“作用”和“好比（与笔者熟悉的 Windows 系统相比较）”来阐述。

#### **核心目录作用**

- `/` - 根目录

  - 作用：整个文件系统的起点，所有其他目录都挂载在它下面。
  - 好比：Windows 的 `C:\` 盘，但更纯粹。

- `/home` - 用户家目录

  - 作用：存储普通用户的个人文件和配置。每个用户都有一个以自己用户名命名的子目录（如 `/home/ubuntu`）。在这里可以自由创建、删除文件，安装个人程序等。
  - 好比：Windows 的 `C:\Users\YourName`。

- `/etc` - 配置文件目录

  - 作用：存放系统所有程序和服务的配置文件。
  - 重要文件举例：
    - `/etc/passwd`：用户账户信息
    - `/etc/hostname`：系统主机名
    - `/etc/network/interfaces` 或 `/etc/netplan/`：网络配置
  - 好比：Windows 的注册表和 `C:\Windows\System32\drivers\etc\` 的结合体。

- `/var` - 可变数据目录

  - 作用：存放经常变化的数据，如日志、缓存、邮件等。
  - 重要子目录：
    - `/var/log`：系统日志和应用程序日志都在这里。出问题了第一个要来看的地方！
    - `/var/www/html`：通常用于存放网站文件（Apache）。
    - `/var/cache`：程序缓存数据。
  - 好比：Windows 的 `C:\Windows\Logs` 和 `C:\Users\YourName\AppData` 的结合。

- **其他重要目录**：

  - `/bin` & `/sbin`：存放系统最基本的命令（如 `ls`, `cp`, `mv`）。`sbin` 通常是需要 root 权限的系统管理命令。
  - `/usr`：存放用户安装的应用程序和文件。（ "User System Resources"）。
  - `/tmp`：临时文件目录，所有用户都可读写，重启后可能会清空。
  - `/boot`：存放系统启动所需的文件（如内核、引导程序）。
  - `/root`：系统管理员（root）的家目录。
  - `/dev`：设备文件目录，Linux 将所有硬件和外设都抽象为文件（如 `/dev/sda` 代表硬盘）。

- #### **绝对路径 vs 相对路径**

  - 绝对路径：从根目录 `/` 开始写的完整路径。
    - 特点：总是以 `/` 开头。
    - 例子：`/home/yourname/Documents/report.pdf`
    - 优点：明确无误，无论当前在哪个目录，这个路径都指向同一个文件。
  - 相对路径：从当前工作目录开始写的路径。
    - 特点：不以 `/` 开头。
    - 依赖：它的意义取决于当前在哪个目录。
    - 特殊符号：
      - `.` 代表当前目录
      - `..` 代表上级目录
      - `~` 代表当前用户的家目录

- ![笔者将该图片上传至GitHub，如果不能正常显示，有可能是因为笔者操作失误。图片内容为操作过程截图。](https://github.com/88fkb88/Photos/blob/main/%E5%B1%8F%E5%B9%95%E6%88%AA%E5%9B%BE%202025-10-14%20125909.png?raw=true)

- ![笔者将该图片上传至GitHub，如果不能正常显示，有可能是因为笔者操作失误。图片内容为操作过程截图。](https://github.com/88fkb88/Photos/blob/main/%E5%B1%8F%E5%B9%95%E6%88%AA%E5%9B%BE%202025-10-14%20130532.png?raw=true)

### 二、掌握基础服务管理（ systemd ）与⽹络配置

笔者在前文的学习中已经初步接触到了**系统服务管理** （systemctl），详情请见“Linux终端的学习-Part 1”。

笔者查阅 [资料](https://zhuanlan.zhihu.com/p/648186810) 显示，systemd 是一个系统和服务管理器，于 2010 年首次推出，用于取代传统的 System V 初始化系统。它旨在提高启动速度并更有效地管理系统服务。如今，systemd 是许多流行 Linux 发行版的默认初始化系统，包括 Ubuntu、Fedora 和 Red Hat Enterprise Linux（RHEL）。之前提到的 `systemctl` 就是管理 systemd 的主要工具。

前文已经涉及了一些systemctl的基本用法：

![笔者将该图片上传至GitHub，如果不能正常显示，有可能是因为笔者操作失误。图片内容为操作过程截图。](https://github.com/88fkb88/Photos/blob/main/%E5%B1%8F%E5%B9%95%E6%88%AA%E5%9B%BE%202025-10-14%20131331.png?raw=true)

现在笔者决定进行更加深入的学习来找到其他命令：

除了基本启停，还有这些常用操作：

- `sudo systemctl enable <服务名>`：设置服务开机自启动
- `sudo systemctl disable <服务名>`：禁止服务开机自启动
- `sudo systemctl is-enabled <服务名>`：检查服务是否开机自启
- `sudo systemctl daemon-reload`：重新加载 systemd 配置（修改服务配置文件后需要执行）
- `sudo systemctl list-units --type=service`：列出所有已加载的服务

下面笔者以ssh服务为例展示部分命令：

![笔者将该图片上传至GitHub，如果不能正常显示，有可能是因为笔者操作失误。图片内容为操作过程截图。](https://github.com/88fkb88/Photos/blob/main/%E5%B1%8F%E5%B9%95%E6%88%AA%E5%9B%BE%202025-10-14%20132223.png?raw=true)

- 过程中，笔者遇到了问题就是在尝试用ssh练习命令的时候发现似乎并没有是先安装ssh，不过由于笔者已经学过了怎么在Linux里面安装自己想要的东西，很顺利地解决了问题。

- ``` shell
  sudo apt update
  sudo apt install openssh-server
  ```

下面顺利来到网络配置的学习：

查看网络信息

- `ip addr` 或 `ifconfig`：查看网络接口和 IP 地址
- `ping <host>`：测试网络连通性
- `ss -tuln` 或 `netstat -tuln`：查看端口监听情况

![笔者将该图片上传至GitHub，如果不能正常显示，有可能是因为笔者操作失误。图片内容为操作过程截图。](https://github.com/88fkb88/Photos/blob/main/%E5%B1%8F%E5%B9%95%E6%88%AA%E5%9B%BE%202025-10-14%20133126.png?raw=true)

~~（起初笔者在 ping minecraft.net 的时候发现没有输出还以为做错了，过了一会才知道是时间太长了，要好久才能得到输出。）~~

除了查看，我们当然还可以进行配置：

（由于笔者在网络上看到有人建议这一步不要进行操作，担心误打误撞更改一些重要的配置，于是笔者将笔者阅读的教程的截图附上）

![笔者将该图片上传至GitHub，如果不能正常显示，有可能是因为笔者操作失误。图片内容为操作过程截图。](https://github.com/88fkb88/Photos/blob/main/%E5%B1%8F%E5%B9%95%E6%88%AA%E5%9B%BE%202025-10-14%20133601.png?raw=true)

### 三、掌握常用命令行工具

任务里面提到的工具一共有5种，包括：vim 、 tmux 、 grep 、 awk 、 sed。

其中的vim已经在前文中详尽阐释，在此不再赘述。

##### 1.tmux

tmux，终端复用器，它的功能是可以让一个终端窗口包含多个会话、窗口、窗格。

**会话**：一个完整的终端工作环境

**窗口**：会话中的单个视图

**窗格**：窗口中的分割区域

笔者认为，这个强大的功能提供了一种多开的方法，比如我们可以一边在后台执行ping [luogu.com.cn](https://www.luogu.com.cn/) ，一边在另一个会话中操作我们的文件，这样就省去了另外开一个终端的麻烦。

![笔者将该图片上传至GitHub，如果不能正常显示，有可能是因为笔者操作失误。图片内容为操作过程截图。](https://raw.githubusercontent.com/88fkb88/Photos/ae0fb1d06f8876779fc56f86f1d2f477d38f0a6c/%E5%B1%8F%E5%B9%95%E6%88%AA%E5%9B%BE%202025-10-14%20161332.png)

![笔者将该图片上传至GitHub，如果不能正常显示，有可能是因为笔者操作失误。图片内容为操作过程截图。](https://raw.githubusercontent.com/88fkb88/Photos/ae0fb1d06f8876779fc56f86f1d2f477d38f0a6c/%E5%B1%8F%E5%B9%95%E6%88%AA%E5%9B%BE%202025-10-14%20161352.png)

![笔者将该图片上传至GitHub，如果不能正常显示，有可能是因为笔者操作失误。图片内容为操作过程截图。](https://github.com/88fkb88/Photos/blob/main/%E5%B1%8F%E5%B9%95%E6%88%AA%E5%9B%BE%202025-10-14%20162141.png?raw=true)

下面列出tmux的主要命令：

``` shell
# 启动新会话
tmux

# 启动命名会话
tmux new -s mysession

# 会话管理
Ctrl + b d     # 分离当前会话（会话在后台继续运行）
tmux ls        # 列出所有会话
tmux attach -t mysession  # 重新连接到会话

# 窗口管理
Ctrl + b c     # 创建新窗口
Ctrl + b n     # 切换到下一个窗口
Ctrl + b p     # 切换到上一个窗口
Ctrl + b 数字   # 切换到指定编号的窗口

# 窗格管理
Ctrl + b %     # 水平分割窗格
Ctrl + b "      # 垂直分割窗格
Ctrl + b 方向键 # 在窗格间移动
Ctrl + b x     # 关闭当前窗格
```

##### 2.grep

笔者通过之前对于vim的学习，已经有了在终端中编辑文本的方法，而grep可以在文本外部直接查找文本内部的文本内容：

``` shell
# 基本搜索
grep "error" logfile.txt          # 在文件中搜索 "error"
grep -i "error" logfile.txt       # 忽略大小写搜索
grep -r "function" /path/to/dir   # 递归搜索目录中的所有文件
grep -n "error" logfile.txt       # 显示匹配行的行号

# 高级用法
grep -v "success" logfile.txt     # 反向搜索，显示不匹配的行
grep -c "error" logfile.txt       # 统计匹配行数
grep -A 2 -B 2 "error" logfile.txt # 显示匹配行及前后各2行

# 正则表达式
grep "^start" file.txt           # 搜索以 "start" 开头的行
grep "end$" file.txt             # 搜索以 "end" 结尾的行
grep "[0-9]" file.txt            # 搜索包含数字的行
```

下面附实践图片：

![笔者将该图片上传至GitHub，如果不能正常显示，有可能是因为笔者操作失误。图片内容为操作过程截图。](https://github.com/88fkb88/Photos/blob/main/%E5%B1%8F%E5%B9%95%E6%88%AA%E5%9B%BE%202025-10-14%20164204.png?raw=true)

![笔者将该图片上传至GitHub，如果不能正常显示，有可能是因为笔者操作失误。图片内容为操作过程截图。](https://github.com/88fkb88/Photos/blob/main/%E5%B1%8F%E5%B9%95%E6%88%AA%E5%9B%BE%202025-10-14%20163711.png?raw=true)

笔者认为，有了grep的帮助，可以更加方便地在很多文件中搜索到自己想要的内容而不用打开它们再用vim一个个找。

##### 3.awk

awk是一个强大的文本处理语言，可以直接认为它就是一门编程语言。

下附常用语法：

``` shell
# 打印特定字段
awk '{print $1}' file.txt          # 打印每行的第一个字段
awk '{print $1, $3}' file.txt      # 打印第1和第3个字段

# 使用分隔符
awk -F: '{print $1}' /etc/passwd   # 使用冒号作为分隔符

# 条件过滤
awk '$3 > 1000' file.txt           # 打印第3个字段大于1000的行
awk '/error/' logfile.txt          # 打印包含 "error" 的行

# 内置变量
awk '{print NR, $0}' file.txt      # NR=行号, $0=整行内容
awk '{print NF, $1}' file.txt      # NF=字段数量
```

##### 4.sed

sed可以实现在文本外部对文本进行过滤与转换。

下附常用语法：

``` shell
# 打印特定字段
awk '{print $1}' file.txt          # 打印每行的第一个字段
awk '{print $1, $3}' file.txt      # 打印第1和第3个字段

# 使用分隔符
awk -F: '{print $1}' /etc/passwd   # 使用冒号作为分隔符

# 条件过滤
awk '$3 > 1000' file.txt           # 打印第3个字段大于1000的行
awk '/error/' logfile.txt          # 打印包含 "error" 的行

# 内置变量
awk '{print NR, $0}' file.txt      # NR=行号, $0=整行内容
awk '{print NF, $1}' file.txt      # NF=字段数量
```

##### 5.综合

依笔者之见，这些强大的文本处理工具可以很方便地在文本外部操作。

当我们要处理一个文本时，可以选择vim编辑器来进行查看该文本并且进行可视化操作。

然而，当我们面对一个十分庞大的文本（比如某软件的运行日志）时，使用vim似乎就不是那么方便了。这时候我们要是想要提取出所有报错“404”的行，grep就可以非常有用。如果我们想要把每一行的信息筛选一下，只显示其中几项，那么就可以试试awk：` awk '{print $1, $7}' access.log ` 。

所以，在vim与grep、awk、tmux、sed 等这些强大的工具的帮助下，可以极大地提升操作效率。

### 至此，Linux的第二部分学习结束

### 四、感悟

对Linux的更多学习让我意识到了Linux的强大，尤其是那些文本编辑工具是真的很有用。

### 五、鸣谢

~~(tips:笔者其实并不知道应不应该有这部分，但是笔者决定加上)~~

在此次学习过程中笔者主要要感谢群中学长拓展了笔者的视野。

其次要感谢deepseek以及网上各路大神的帖子帮助了我的学习。

# THE END





