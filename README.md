# 这是一个面向RISC-V初学者的文档
在这篇文档里，我们会以RISC-V软件开发者的角度来理解RISC-V，并适当涉足其硬件实现（Verilog），参考的范例是PicoRV32，参考的硬件开发板为SparkRoad Micro。
* PicoRV32<https://github.com/cliffordwolf/picorv32>
* SparkRoad Micro<https://github.com/verimake-team/SparkRoad-FPGA>

## 第一章 搭建RISC-V编译环境
### 本地虚拟机还是云主机？
到目前为止，主流的方式还是在Linux下构建RISC-V编译环境（如果你看到有Win下的案例，可以与我联系）。        
因此我们首先需要有一个Linux环境，在绝大多数用户都是Windows用户的情况下，传统的方式是在Windows操作系统上安装一个虚拟机软件（比如Virtual Box），在虚拟机里再安装一个Linux的发行版（比如Ubuntu），但是虚拟机系统需要占用大量的磁盘空间，而且会快速消耗你的笔记本电池。    
本文档建议的方式是使用云主机的方式来搭建，相对于传统方式，云主机可以实现24小时运行而不用过于担心电池，借助Putty这类轻巧的SSH客户端（大概1MB不到）几乎可以实现随时随地的使用，只要你手上有一台可以连上网络的电脑即可（甚至紧急情况下，用手机或平板电脑也可以轻松访问）。另外，由于绝大多数云主机都提供独立的公网IP和一定的网络带宽，你可以获得的额外好处就是可以搭建一个网络服务器，方便地实现网络应用。    
而你要付出的额外代码就是云主机的费用，好在这个费用是可以承受的，以阿里云为例，学生用户可以享受大概10元每月的优惠价格。即使是非学生用户，在服务商开展一些优惠活动时，大概每年的费用在人民币200元多一些，相对获得的便利，这个成本完全值得。

### 搭建Linux环境
本文档以阿里云（其它厂商比如Amazon、华为云大同小异）主机为例来搭建Linux环境。
首先你需要注册阿里云的账号，并在<https://www.aliyun.com/activity#/>活动列表中找到最优惠的活动，并通过活动链接购买主机。在相关的选项中，注意以下几点：
* 操作系统选择ubuntu，建议选择最新的版本
* 注意设置的root密码
* 如果经济条件允许，可以选择高一些的配置，尤其是网络带宽和CPU、内存    
本文档基于阿里云最低的主机配置：1核1G，40G硬件，1M带宽

#### 连接云主机
使用putty或者win10自带的ssh客户端（CMD命令行界面下输入ssh root@YouIP即可）    
本文档默认全部在root用户下操作（虽然这是一个不建议的方式）    

#### 安装git
<pre>
apt-get update
apt-get install git
</pre>

#### 下载源文件
<pre>
git clone --recursive https://github.com/riscv/riscv-gnu-toolchain    
</pre>
加上--recursive 参数，不仅会下载代码库本身，一些子模块也会被下载    
由于网络环境限制，这个下载可能会比较漫长，你可能需要具备一些特殊技能来加速这个下载过程，如果你还不具备（这个技能是必备技能...），那么可以通过以下链接从百度网盘下载源码包，并用scp命令传输到你的服务器（下载链接稍后放出）。

#### 安装必要工具
<pre>
$ apt-get install autoconf automake autotools-dev curl libmpc-dev libmpfr-dev libgmp-dev gawk build-essential bison flex texinfo gperf libtool patchutils bc zlib1g-dev libexpat-dev
</pre>
#### 配置并编译
<pre>
./configure --prefix=/opt/riscv    
make    
</pre>
如果在编译的过程中报错，可能的原因有两个：    
1、源码下载不完整，在riscv-gnu-toolchain目录下使用以下两个命令确保下载完成    
<pre>
git pull    
git submodule update --init --recursive    
</pre>
2、工具安装不完整，用apt-get install 命令再逐一检查一下    

#### 测试工具可用性
编译成功后，会生成/opt/riscv/目录，并可以在该目录下找到相应的工具    
输入命令    
<pre>
/opt/riscv/bin/riscv64-unknown-elf-gcc -v    
</pre>
可以看到相应的信息输出，表示已经成功编译得到相应的开发工具    
<pre>
Using built-in specs.
COLLECT_GCC=/opt/riscv/bin/riscv64-unknown-elf-gcc
COLLECT_LTO_WRAPPER=/opt/riscv/libexec/gcc/riscv64-unknown-elf/9.2.0/lto-wrapper
Target: riscv64-unknown-elf
Configured with: /root/rv/riscv-gnu-toolchain/riscv-gcc/configure --target=riscv64-unknown-elf --prefix=/opt/riscv --disable-shared --disable-threads --enable-languages=c,c++ --with-system-zlib --enable-tls --with-newlib --with-sysroot=/opt/riscv/riscv64-unknown-elf --with-native-system-header-dir=/include --disable-libmudflap --disable-libssp --disable-libquadmath --disable-libgomp --disable-nls --disable-tm-clone-registry --src=.././riscv-gcc --enable-checking=yes --disable-multilib --with-abi=lp64d --with-arch=rv64imafdc --with-tune=rocket 'CFLAGS_FOR_TARGET=-Os  -mcmodel=medlow' 'CXXFLAGS_FOR_TARGET=-Os  -mcmodel=medlow'
Thread model: single
gcc version 9.2.0 (GCC) 
</pre>
#### 将工具路径加入PATH环境变量
如果不想每次都输入/opt/riscv/bin/这么长的绝对路径，而只想输入riscv64-unknown-elf-gcc就可以使用工具，就需要将工具的路径加入到PATH环境变量，Linux会到相应路径下去查找相应的可执行程序    
先看一下PATH的当前值，输入命令    
<pre>
echo $PATH    
</pre>
可以看到类似的值：    
<pre>
/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin:/usr/games:/usr/local/games    
</pre>
你可以采取export的方式临时将工具所在的路径添加进PATH，但更建议修改~/.bashrc文件来永久实现,输入命令    
<pre>
vim ~/.bashrc    
</pre>
（这里假设你是一个Linux小白），用方向键将光标移动到文件的最后一行的最后一个字符后面，按一下i，最下面会显示-- INSERT --表示进入插入模式，按一下回车键新起一行，输入    
<pre>
export PATH=$PATH:/opt/riscv/bin    
</pre>
然后按一下ESC键（不放心就多按几次），可以看到最下面的-- INSERT--消失，表示进入命令模式，在命令模式下输入    
<pre>
:wq    
</pre>
保存修改并退出vim，回到Linux命令提示符下，输入以下命令使刚才的修改生效    
<pre>
source ~/.bashrc    
</pre>
输入命令riscv64-unknown-elf-gcc -v（在输入的过程中按Tab键试试）看一下效果    
如果可以看到与前面输入绝对路径一样的输出，表示配置正确，否则再次用vim打开.bashrc文件检查修改是否正确    








