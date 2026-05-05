# 小组成员与学号

| 成员名 | 学号            |
| --- | ------------- |
| 林静茂 | 2024306230213 |
| 徐姝妍 | 2024317220724 |
# 程序的功能简介和运行环境
## 功能简介
实现了贪吃蛇的游戏界面,能在终端下用WASD进行操作,在棋盘内能随机刷新食物,在蛇头与墙壁碰撞时,自动结束。并且实现多系统可用。
## 运行环境
windows+linux 双端可用。
# 如何编译和运行程序(windows乱码请看)
## 如何编译程序
考虑到需要编译多个文件，在编译的时候采用Makefile，在AI的指导下学习了交叉编译,成功编译了.exe的windows可执行文件  
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
```cmd
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
徐姝妍:

贡献比例50%-50%