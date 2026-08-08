## 依旧 Hello World
每一门编程语言的学习，一般都是从 Hello World 学起的，C3 也不例外：
```c
import std::io;

fn void main()
{
    io::printn("Hello, World!");
}
```
> 注：因为 C3 在 Obsidian 里面没有好看的高亮，我就用 C 来作为代码块标记了。  
## 详细拆分
可以看到，C3 采用 `import` 导入模块。本例中，导入了 `std` 中的 `io` 模块。  
然后定义了一个主函数。C3 的函数定义格式为：先以 `fn` 关键字开头，然后是返回类型，最后是函数名，括号内是参数列表：
```c
fn void main(){}
```
官方笔记里提到：`main` 为主函数，也可写成 `fn void main(String[] args)`。这里的 `args` 用空格连接不同参数，举个例子：
```
./chat_client 192.168.1.100 8080
```
- `args[0]` 就是 `"./chat_client"`
- `args[1]` 就是 `"192.168.1.100"`
- `args[2]` 就是 `"8080"`

`{}` 即为函数的作用域，其中包含了 `io::printn("Hello, World!");`，即调用 `std::io` 模块中的 `printn` 函数。

## 编译与运行
以本例为例：
- 编译：`c3c compile hello_world.c3`
- 运行：`./hello_world`
- 编译并运行：`c3c compile-run hello_world.c3`

## 其他构建命令
| 命令 | 核心作用 | 适用场景 |
| --- | --- | --- |
| `c3c compile` | 编译独立文件为可执行程序 | 单文件脚本、快速验证代码 |
| `c3c compile-run` | 编译并立即运行独立文件 | 新手学习、轻量级功能测试 |
| `c3c init` | 初始化标准项目结构 | 创建新项目（支持 exe/静态库/动态库） |
| `c3c build` | 按照配置构建整个项目 | 日常工程开发 |
| `c3c run` | 增量构建并运行项目 | 开发过程中的高频调试指令 |
| `c3c clean` | 清除之前的构建产物 | 解决奇怪的构建缓存问题 |
| `c3c test` | 运行所有单元测试 | 验证 `@test` 标记的代码逻辑 |
| `c3c benchmark` | 运行性能基准测试 | 性能剖析、代码优化 |
