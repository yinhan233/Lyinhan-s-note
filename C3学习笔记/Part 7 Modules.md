## 如何定义模块
```c
// 文件：math_tools.c3
module math_tools;

// 下面写的函数都属于 math_tools 这个模块
fn int add(int a, int b) {
    return a + b;
}
```
> 一个模块可以拆分写在多个文件里。比如 `file_a.c3` 和 `file_b.c3` 的开头都写了 `module foo;`，那么它们都属于 `foo` 模块。
## 如何使用模块
如果想用其他模块写好的代码，需要先用 `import` 把它"导入"进来。C3 有一个非常清晰的"前缀规则"：
- **函数/变量**：必须带上模块名作为前缀（防止名字冲突）。
- **类型（如结构体）**：不需要带前缀，除非名字重复了。
```c
module my_game;

import math_tools; // 导入上面我们写的模块

fn void test() {
    // 调用函数：必须带上模块名和双冒号 (math_tools::)
    int result = math_tools::add(5, 3);

    // 假设 math_tools 里有一个结构体叫 Vector2
    // 使用类型：直接用，不需要写 math_tools::Vector2
    Vector2 position = {};
}
```
## 可见性设置
| 标签 | 谁能用这段代码？ | 举例 |
| --- | --- | --- |
| （默认） | 所有人（Public） | 任何 `import` 了你的模块的人都能用。 |
| `@private` | 仅同一个模块内部 | 别人不能用，但同属该模块的其他文件可以互调。 |
| `@local` | 仅当前这个文件 | 即使是同一个模块的其他文件也不能用，最严格。 |
```c
module foo;

fn void init() { ... }           // 大家都能用
fn void open() @private { ... }  // 只有 foo 模块内部能用
```
## 补充
文档中其实还提到了一些别的特性，后面再看吧！
- **动态加载 (`@dynamic`)**：用于根据不同版本加载动态链接库。
- **文本包含 (`$include` / `$exec`)**：允许你在编译时插入其他文件的文本，或者执行脚本生成代码。
- **非递归导入 (`@norecurse`)**：默认导入模块时会把它底下的子模块也带进来，用这个可以阻止这种行为。[文档链接](https://c3-lang.org/language-fundamentals/modules/#non-recursive-imports)