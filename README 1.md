# 小组成员与学号

| 成员名 | 学号            |
| --- | ------------- |
| 林静茂 | 2024306230213 |
| 徐姝妍 | 2024317220724 |
# 程序的功能简介和运行环境
## 功能简介
贪吃蛇控制：支持使用 WASD 或 方向键 控制蛇的移动方向
食物系统：在游戏区域内随机生成食物，蛇吃到食物后会变长
碰撞检测：实时检测蛇头与墙壁碰撞即游戏结束
游戏重新开始功能：游戏结束后会询问是否重新开始
增强信息显示：显示当前移动方向和显示食物位置坐标
暂停功能：按 P 键暂停游戏，按任意键继续
跨平台支持：支持 Windows 和 Linux 双系统运行
分数记录：游戏结束后自动将得分保存到文件
## 运行环境
Windows系统：Windows 7/8/10/11，需要C运行库支持
Linux系统：主流Linux发行版，需要GCC编译器和标准C库
双端兼容：程序已针对两个平台进行适配，使用条件编译技术
# 如何编译和运行程序(含windows乱码解决方案)
## 如何编译程序
考虑到需要编译多个文件，在编译的时候采用Makefile，在AI的指导下学习了交叉编译,成功编译了.exe的windows可执行文件  
对于新增小功能后的文件编译，没有重新下载编译器，主要通过在cmd中使用 Dev-C++ 自带的 TDM-GCC 64位编译器编译四个C源文件生成可执行文件
对于linux版  
编译步骤大体如下:  
首先,进行变量定义:
```
编译器和编译选项 CC = gcc CFLAGS = -Wall -Wextra -O2 -std=c99 -D_POSIX_C_SOURCE=200112L TARGET = snake_game 
源文件和头文件 SRCS = snake_game.c game_logic.c input.c display.c OBJS = $(SRCS:.c=.o) HDRS = snake_game.h game_logic.h input.h display.h
```
然后,链接生成可执行文件
```
$(TARGET): $(OBJS) $(CC) $(CFLAGS) -o $(TARGET) $(OBJS)
```
然后将.c转换为二进制文件.o
```
snake_game.o: snake_game.c $(HDRS) $(CC) $(CFLAGS) -c snake_game.c -o snake_game.o 
game_logic.o: game_logic.c game_logic.h snake_game.h $(CC) $(CFLAGS) -c game_logic.c -o game_logic.o 
input.o: input.c input.h snake_game.h $(CC) $(CFLAGS) -c input.c -o input.o display.o: display.c display.h snake_game.h $(CC) $(CFLAGS) -c display.c -o display.o
```
最后编译出对应目标  
具体执行的话,只要`Make -f Makefile`就可。  
对于windows版本的`MakeFile`
因为不会写,是让AI指导着写的  
在这个过程中,我发现可以直接用通配符来转换`.c`到`.o`...与linux不同点主要就在于CFlags和所用的编译器,大体上还是相同的。
## 如何运行程序
windows下,双击运行,如果乱码请往下看  
注意,因为windows自带的终端可能不支持UTF-8,所以可能需要进行如下操作:
```cmd（win+r后输入cmd）
首先,在cmd(win+x->powershell/命令提示行)中输入
chcp 65001
然后右键snake_game.exe->复制文件路径
在打开的cmd中右键粘贴,回车运行。
```
linux下  
`chmod +x snake_game`  
`./snake_game`  
即可
# 小组成员分工
林静茂:负责编写程序中地图生成,兼容linux+windows双系统,蛇的控制和移动，包括实现键盘输入、蛇头和蛇身的移动功能的实现,并且对程序进行了编译。  
徐姝妍:新增游戏重新开始功能，暂停/继续功能，增强信息显示（移动方向，食物坐标，部分文档补充与完善
贡献比例70%（林）-30%（徐）
# 项目总结
本项目成功实现了一个跨平台的贪吃蛇游戏，通过合理的模块化设计和条件编译技术，确保了在Windows和Linux系统上的良好运行。程序不仅具备完整的游戏功能，还提供了友好的用户界面和便捷的编译运行方式，展现了团队在C语言的编程、系统兼容性和工程实践方面的能力