# Linux终端的学习-Part 1

### 任务目标：

1. 引言
2. ⾃主安装并配置 Linux 系统
3. 对终端命令熟练掌握
   - 软件安装（ apt , snap 等）
   - ⽂件管理（ cp , mv , rm , ls , touch 等）
   - 权限管理（ chmod , chown , ls -l 理解权限位）
   - 进程控制（ ps , kill , top , systemctl ）
4. 感悟
5. 鸣谢

### 一、引言

书接上回，笔者已经装配好了自己的Linux虚拟机，现在要继续来学习对Linux终端的操作了。

### 二、⾃主安装并配置 Linux 系统

该部分笔者已于上篇文档中详细讲解，在此不再赘述，欲查看请到笔者的库中寻找 “实践任务.md”。

### 三、对终端命令熟练掌握

##### 1、软件安装

对于软件的安装，笔者在安装VScode的过程中已经意识到Linux与Windows的不同之处了——Linux需要用apt , snap 等命令来安装软件。

apt：

- `sudo apt install <软件包名>`： 安装软件。
- `sudo apt remove <软件包名>`： 卸载软件（但可能保留配置文件）。
- `sudo apt purge <软件包名>`： 彻底卸载软件（同时删除配置文件）。
- `sudo apt update`： 更新软件源列表。（笔者理解为进行应用商店的更新来获取最新的软件）
- `sudo apt upgrade`： 升级所有已安装的软件包。

在此处笔者以安装VScode的过程做实例：

``` shell
sudo apt update
sudo apt install code 
```

snap:

- `sudo snap install <软件包名>`： 安装 snap 软件。
- `sudo snap remove <软件包名>`： 卸载 snap 软件。

在此处笔者以安装Spotify为例：

``` shell
sudo snap install spotify
```

此外，笔者在查阅资料的过程中注意到[有的人建议](https://zhuanlan.zhihu.com/p/671969411)尽量使用apt而不是snap，除非官方建议使用snap。

##### 2.文件管理

> 这些是你每天都会用到的命令，相当于在 Windows 资源管理器里用鼠标进行的操作。

1. 基础语法：

   - `ls`： 列出当前目录下的文件和文件夹。
     - `ls -l`： 以详细列表形式显示（能看到权限、所有者、大小、修改时间）。
     - `ls -a`： 显示所有文件（包括隐藏文件，隐藏文件以 `. `开头）。
   - `pwd`： 显示你当前所在目录的绝对路径。（注意到此处提到绝对路径，将会以后阐明）
   - `cd <路径>`： 切换目录。
     - `cd ..`： 返回上一级目录。
     - `cd ~` 或 `cd`： 回到家目录（/home/Ubuntu）。
     - `cd -`： 回到上一个所在的目录。
   - `touch <文件名>`： 创建一个新的空文件，或者更新一个已存在文件的修改时间。
   - `mkdir <目录名>`： 创建新目录。
     - `mkdir -p parent/child`： 递归创建多级目录。

   ![笔者将该图片上传至GitHub，如果不能正常显示，有可能是因为笔者操作失误。图片内容为操作过程截图。](https://raw.githubusercontent.com/88fkb88/Photos/1213823dce823507058ea114033cea6ef845ef88/%E5%B1%8F%E5%B9%95%E6%88%AA%E5%9B%BE%202025-10-14%20082227.png)

   ![笔者将该图片上传至GitHub，如果不能正常显示，有可能是因为笔者操作失误。图片内容为操作过程截图。](https://raw.githubusercontent.com/88fkb88/Photos/1213823dce823507058ea114033cea6ef845ef88/%E5%B1%8F%E5%B9%95%E6%88%AA%E5%9B%BE%202025-10-14%20082633.png)

2. 复制、移动、删除 ：     ~~（笔者在一开始的时候总是习惯性地用Ctrl-C和Ctrl-V，被报错了好几次……）~~

   - `cp <源文件> <目标路径>`： 复制文件或目录。

     - `cp file1.txt /home/user/Documents/`： 复制 file1.txt 到 Documents 目录。
     - `cp -r dir1 dir2`： 递归复制目录及其内部所有内容。

   - `mv <源> <目标>`： 移动文件或目录，也可以用来重命名。

     - `mv oldname.txt newname.txt`： 重命名文件。
     - `mv file1.txt ~/Documents/`： 移动文件到 Documents 目录。

   - `rm <文件>`： 删除文件。

     - `rm -r <目录>`： 递归删除目录及其内部所有内容。

     - `rm -f <文件>`： 强制删除，不询问。

       > ⚠️ 警告： `rm -rf /` 是极其危险的命令，会强制删除根目录下的所有文件！永远不要执行！

   ![笔者将该图片上传至GitHub，如果不能正常显示，有可能是因为笔者操作失误。图片内容为操作过程截图。](https://github.com/88fkb88/Photos/blob/main/%E5%B1%8F%E5%B9%95%E6%88%AA%E5%9B%BE%202025-10-14%20084338.png?raw=true)

##### 3.权限管理：

> Linux 是一个多用户系统，所以权限很重要。

``` shell
ls -l
```

笔者得到如下输出：

``` shell
drwxr-xr-x 2 ubuntu ubuntu 4096 10月 11 19:30 公共的
```

其中第一个字符串 `drwxr-xr-x`：

- 第1位：文件类型
  - `d`： 目录
  - `-`： 普通文件
  - `l`： 链接文件
- 第2-4位：文件所有者（user） 的权限
- 第5-7位：所属用户组（group） 的权限
- 第8-10位：其他用户（others） 的权限

权限字符：

- `r`： 读权限（数字表示为 4）
- `w`： 写权限（数字表示为 2）
- `x`： 执行权限（数字表示为 1）
- `-`： 无此权限

所以 `rwxr-xr-x` 的意思是：

- 所有者：可读、可写、可执行 (rwx = 4+2+1 = 7)
- 用户组：可读、可执行 (r-x = 4+0+1 = 5)
- 其他用户：可读、可执行 (r-x = 4+0+1 = 5)

此外，使用 ` chmod ` 来进行权限的改变：

``` shell
chmod 644 my_file.txt
# 6 (所有者 rw-)，4 (用户组 r--)，4 (其他用户 r--)

# u：用户，g：组，o：其他，a：所有
# +：添加权限，-：移除权限，=：设置权限
chmod u+x my_script.sh  # 给所有者添加执行权限
chmod o-w secret.txt    # 移除其他用户的写权限
```

还可以使用` chown` 来修改文件所有者和所属组：

``` shell
sudo chown newuser:newgroup myfile.txt
sudo chown newuser myfile.txt # 只改所有者不改所属组
```

##### 4.进程控制

查看进程

- `ps`： 显示当前终端下的进程。
  - `ps aux`： 显示系统所有运行中的进程（最常用）。可以配合 `grep` 过滤，如 `ps aux | grep nginx`。
- `top` 或 `htop`： 动态、实时地显示进程状态（像 Windows 的任务管理器）。按 `q` 退出。

结束进程

- `kill <PID>`： 通过进程ID（PID）来结束进程。PID 可以从 `ps` 或 `top` 命令中看到。
  - `kill -9 <PID>`： 强制结束一个不响应的进程。
- `pkill <进程名>`： 通过进程名来结束进程。（很像 Windows 里面某些东西卡住之后用任务管理器强杀后台）
  - `pkill firefox`

![笔者将该图片上传至GitHub，如果不能正常显示，有可能是因为笔者操作失误。图片内容为操作过程截图。](https://github.com/88fkb88/Photos/blob/main/%E5%B1%8F%E5%B9%95%E6%88%AA%E5%9B%BE%202025-10-14%20090307.png?raw=true)

系统服务管理 - `systemctl`

- `sudo systemctl start <服务名>`： 启动服务。
- `sudo systemctl stop <服务名>`： 停止服务。
- `sudo systemctl restart <服务名>`： 重启服务。
- `sudo systemctl status <服务名>`： 查看服务状态。
- `sudo systemctl enable <服务名>`： 设置开机自启。
- `sudo systemctl disable <服务名>`： 禁止开机自启。

``` shell 
sudo systemctl status ssh  # 查看 SSH 服务状态
sudo systemctl restart ssh # 重启 SSH 服务
```

##### 5.另外，可以通过--help来即时获取帮助文档

### 至此，Linux重要的终端命令已经学习并掌握完毕。

### 四、感悟

在命令行的学习中，笔者深切感受到了 Linux 与 Windows 的区别，意识到终端操作在 Linux 系统中的重要地位。

### 五、鸣谢

~~(tips:笔者其实并不知道应不应该有这部分，但是笔者决定加上)~~

在此次学习过程中笔者主要要感谢群中学长拓展了笔者的视野。

其次要感谢deepseek以及网上各路大神的帖子帮助了我的学习。

# THE END















