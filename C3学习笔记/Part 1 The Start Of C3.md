## 写在前面

正如你所见，这是我的 C3 学习笔记。先介绍下背景吧——在写这本笔记时，本人只是一名普普通通的大三学生，就读于某个国内211的计算机科学与技术专业。  
自己做过的项目很水：Java 聊天室、一个智能盆栽相关的 agent（有一半是 AI）、一个用 Rust 写的数据库（有一半是 AI）、一个车辆调度软件（这个全是 AI 的）。    
所以自己基本上没有什么开发经验，是个菜鸡 😭，自己也很焦虑。笔记的参考价值不大，只是想记录一下自己的学习过程。最后的话可能想用 C3 手搓一些项目，路过的大佬轻喷。  
我的写作水准并不高，笔记大部分内容是基于AI辅助的对官方文档的翻译，建议可以看看[C to C3](https://c3-lang.org/c-to-c3/a-guide-for-c-programmers/#other-changes)。
## 为什么学习 C3
没有什么特别的理由，只是听说他是更现代的 C。我个人也非常喜欢 C 语言，所以比较想学习。  
## 环境搭建
本人使用系统为 CachyOS，编辑器使用 VS Code。这里介绍一下我的安装过程：  
如有需要，请访问  
[https://c3-lang.org/getting-started/prebuilt-binaries/]  
1. 安装编译器：  
   ```bash
   sudo pacman -S c3c
   ```
2. VS Code 安装以下插件：  
   - C3 Language Support for VSCode
   - c3fmt
   - lldb
1. 配置 C3 相关的路径  
`.vscode` 配置时包含 `launch.json` 和 `tasks.json`，这里给出我的配置。  
首先是 `launch.json`：    
```json
{
  "version": "0.2.0",
  "configurations": [
    {
      "type": "lldb",
      "request": "launch",
      "name": "Debug current C3 file (LLDB)",
      "program": "${fileDirname}/build/${fileBasenameNoExtension}",
      "args": [],
      "cwd": "${fileDirname}",
      "preLaunchTask": "c3c: compile current file"
    }
  ]
}
```
然后是 `tasks.json`：  
```json
{
  "version": "2.0.0",
  "tasks": [
    {
      "label": "c3c: compile current file",
      "type": "shell",
      "command": "mkdir -p \"${fileDirname}/build\" && c3c compile -g \"${file}\" -o \"${fileDirname}/build/${fileBasenameNoExtension}\"",
      "group": {
        "kind": "build",
        "isDefault": true
      },
      "problemMatcher": [],
      "presentation": {
        "reveal": "always",
        "panel": "shared"
      }
    }
  ]
}
```

后面的话，我基本上是按照教程一步一步来的，可能比较 ~~铸币~~