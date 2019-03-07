<!-- GFM-TOC -->
* [cmake和makefile](#cmake和makefile)
* [cmake的执行过程](#cmake的执行过程)
* [常用指令](#常用指令)
<!-- GFM-TOC -->

####cmake和makefile
Makefile描述了整个工程的编译、连接等规则，Makefile 可以有效的减少编译和连接的程序，只编译和连接那些修改的文件。
Makefile的执行过程如下：
1. 在当前目录下寻找Makefile或makefile。
2. 找到第一个文件中的第一个目标文件，和目标文件依赖的.o文件。
3. 如果.o文件不存在，或是后面.o文件比target文件更新，那么它就会执行后面的语句来生成这个文件。
4.  最后makefile会根据.o文件依赖的.h和.c文件生成.o文件。

Cmke跨平台，可以更加简单的产生makefile，不用自己修改。

###cmake的执行过程
1. 编写CMake的配置文件CMakeLists.txt
2. 执行命令 cmake PATH 或者 ccmake PATH 生成 Makefile, PATH位CMakeLists.txt所在的目录。(ccmake有交互见面，cmake没有)
3. 使用make命令进行编译

###常用指令
####find指令
find pathname -options [-print -exec -ok]
-options: -name 按照文件名查找文件

         -perm 按文件权限查找文件

         -user 按文件属主查找文件

         -group  按照文件所属的组来查找文件。

         -type  查找某一类型的文件
命令参数：
        pathname: find命令所查找的目录路径。例如用.来表示当前目录，用/来表示系统根目录。
        -print： find命令将匹配的文件输出到标准输出。
        -exec： find命令对匹配的文件执行该参数所给出的shell命令。相应命令的形式为'command' {  } \;，注意{   }和\;之间的空格。
        -ok： 和-exec的作用相同，只不过以一种更为安全的模式来执行该参数所给出的shell命令，在执行每一个命令之前，都会给出提示，让用户来确定是否执行。
####rm命令
rm -option filename
-option: -i 删除前逐一询问
         -rf 不用一一确认
         --
#####删除当前文件夹下满足提交的所有文件
find / -name "test*"|xargs rm -rf
find / -name “test*” -exec rm -rf {} /;
rm -rf $(find / -name “test”)
