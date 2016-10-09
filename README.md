# DOL学习笔记
@(DOL笔记)[DOL, Git, Markdown]

###DOL简介
>**Distributed Operation Layer** : 
The distributed operation layer (**DOL**) is a software development framework to program parallel applications. The DOL allows to specify applications based on the Kahn process network model of computation and features a simulation engine based on SystemC. Moreover, the DOL provides an XML-based specification format to describe the implementation of a parallel application on a multi-processor systems, including binding and mapping.

***

###安装配置DOL
实验需要在linux下完成，因而需要在虚拟机上安装Linux
- **虚拟机**选择[VMware][1],下载安装[这里][3]
- **系统**选择[Ubuntu][2],下载安装[这里][4]

我虚拟机选择的是*VMware-workstation-full-12.1.0-3272444*，系统选择的是*ubuntu-14.04.3-desktop-amd64*
虚拟机及Ubuntu系统安装完成后，开始安装配置DOL实验环境：
- 首先是安装一些必要的工具**g++**，**ant**， **unzip**
``` bash
$ sudo apt-get update
$ sudo apt-get g++          
$ sudo apt-get install ant
$ sudo apt-get install unzip
```
- 还有**jdk**工具，将jdk压缩包jdk-8u101-linux-x64.gz放在当前用户主文件夹下，在/usr/lib/目录下创建java目录，执行以下命令，将压缩包解压到/usr/lib/java
``` bash
$ sudo tar -zxvf jdk-8u101-linux-x64.gz -C /usr/lib/java
```
- 配置**JAVA_HOME**，**JRE_HOME**，**CLASSPATH**变量
``` bash
（当前用户；root用户为：vim /.bashrc）
$ vim ~/.bashrc       
在打开的文档最后加入以下内容：
    export JAVA_HOME=/usr/lib/java/jdk1_8
    export JRE_HOME=${JAVA_HOME}/jre
    export CLASSPATH=${JAVA_HOME}/lib:${JRE_HOME}/lib
    export PATH=${JAVA_HOME}/bin:$PATH
执行
$ source ~/.bashrc  
```
- 完成上述工作后，开始进行DOL环境的配置，首先创建一个新目录embeded_system用来存储*dol*和*systemc-2.3.1*
``` bash
$ cd ~
$ mkdir embeded_system
```
- 将压缩包*dol_ethz.zip*和*systemc-2.3.1.tgz*放在*embeded_system*文件夹中，执行
``` bash
$ cd embeded_system
$ mkdir dol
$ unzip dol_ethz.zip -d dol
$ tar -zxvf systemc-2.3.1.tgz
```
- 此时，在embeded_system中有两个文件夹：*dol*和*systemc-2.3.1*，接下来编译*systemc*
```bash
$ cd systemc-2.3.1
$ mkdir objdir
$ cd objdir
$ ../configure CXX=g++ --disable-async-updates
$ sudo make install
$ cd ..
$ pwd       （显示当前目录，下面会用到）
```
- 到此，systemc编译完成，接下来编译dol
```bash
$ cd ../dol
```
- 进入dol文件夹，找到文件build_zip.xml文件，打开，找到下面这段话
```xml
<property name="systemc.inc" value="XXX/include" />
<property name="systemc.lib" value="XXX/lib_linux/libsystemc.a" />
```
- 将*XXX*改为上面执行pwd后显示的目录，我的系统是64位，故上面的**lib-linux**要改为**lib-linux64**，进行编译
```bash
$ ant -f build_zip.xml all
```
- 若成功则显示build successful，接下来可以试着运行第一个例子
```bash
$ cd build/bin/main
$ sudo ant -f runexample.xml -Dnumber=1
```
- 成功的话会显示BUILD SUCCESSFUL，若build failed，则可能是中文系统的问题，需要修改在dol/build/bin/main 下 的 runexample.xml文件，找到
```xml
 <tstamp>
      <format property="touch.time"
              pattern="MM/dd/yyyy hh:mm aa"
              offset="-5" unit="second"/>
 </tstamp>
 <touch datetime="${touch.time}">
      <fileset dir="example${number}"/>
 </touch>
```
- 将其注释
```xml
<!--     
<tstamp>
   <format property="touch.time"
              pattern="MM/dd/yyyy hh:mm aa"
              offset="-5" unit="second"/>
   </tstamp>
<touch datetime="${touch.time}">
     <fileset dir="example${number}"/>
</touch> -->
```

- 也可以写个一小段代码测试一下:
hello.h
```cpp
#ifndef _HELLO_H
#define _HELLO_H
#include "systemc.h"
#include <iostream>
using namespace std;
SC_MODULE(hello){
	SC_CTOR(hello){
		cout<<"Hello,SystemC!"<<endl;
	}
};
#endif
```
- hello.cpp
```cpp
#if 1
#include "hello.h"
#else
#include "systemc.h"
class hello : public sc_module{
public:
	hello(sc_module_name name) : sc_module(name){
	        cout<<"Hello,SystemC!"<<endl;
	}
};
#endif
int sc_main(int argc,char** argv){
	hello h("hello");
	return 0;
}
```
- Makefile文件
```makefile
LIB_DIR=-L /home/embedded-system/embeded_system/systemc-2.3.1/lib-linux64
INC_DIR=-I /home/embedded-system/embeded_system/systemc-2.3.1/include
LIB=-l systemc
APP=hello
all:
	g++ -o $(APP) $(APP).cpp $(LIB_DIR) $(INC_DIR) $(LIB)
clean:
	rm -rf $(APP)
```
- 将以上三个文件放在同一目录test下
```bash
$ cd test
$ make
$ ./hello
```
- 会看到输出
```bash
		SystemC 2.3.1-Accellera --- Mar 26 2015 11:03:07
		Copyright (c) 1996-2014 by all Contributors,
		ALL RIGHTS RESERVED
	Hello,SystemC!
```
- 若运行出现以下问题：
```bash
运行出现如下错误：
	./hello: error while loading shared libraries: libsystemc-2.3.1.so: cannot open shared object file: No such file or directory
```
- 需要增添链接器ld加载路径
```bash
$ su   
$ gedit usr-libs.conf
在其中加入(systemc路径下的lib-linux64)
/home/embedded-system/embeded_system/systemc-2.3.1/lib-linux64
退出后让配置生效
$ ldconfig
```
- 至此，DOL的实验环境配置完成。
***

###实验心得与体会
学习一门新的课程总免不了需要配置实验环境，在此次的实验中，我也遇到了不少问题，但总能从TA给的资料中找到答案，或者自己在网上查找到相关知识，但这仍然不够，linux系统在很多门课程中或多或少都会涉及到，因而linux的基本操作仍需要熟悉。


***

- 学号：14353024
- 邮箱：<431485077@qq.com>  


感谢阅读这份文档。

[1]:http://www.vmware.com/cn.html
[2]:https://www.ubuntu.com
[3]:http://rj.baidu.com/soft/detail/13808.html?ald
[4]:http://cn.ubuntu.com/download/