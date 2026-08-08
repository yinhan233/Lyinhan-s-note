## 带有标签的 break 和 continue
C3 允许给 `if`、`switch`、`while` 和 `do` 语句加上标签，这样你就可以直接跳出指定的外层作用域。
```c
fn void test(int i) {
    // FOO: 是给这个 if 块起的标签
    if FOO: (i > 0) {
        while (1) {
            io::printfn("%d", i);

            // 直接跳出最外层的 FOO (即 if 语句块)，而不是仅跳出 while
            if (i++ > 10) break FOO;
        }
    }
}
```

## 不带 while 的 do 语句
`do-while` 语句可以省略结尾的 `while`。在这种情况下，它的行为就等同于 `while(0)`。
```c
fn void test(int x) {
    do {
        // 如果 x 是 0，跳出 do 代码块，后面的 "Hello " 就不会被打印
        if (!x) break;
        io::printf("Hello ");
    };

    io::printf("World!\n");
}
```
## nextcase 和带标签的 nextcase
`nextcase` 用于在 `switch` 和 `if-catch` 中跳转到下一个条件分支。它支持接表达式、`default` 或外层标签，实现有结构的状态机跳转。  
在 C 语言中，`switch` 的 case 结尾如果不写 `break`，程序会自动"贯穿"到下一个 case，这非常容易因为漏写 `break` 而产生 bug。  
**C3 的规则是：默认每个 case 都是自动 break 的！** 如果你明确想要跳到下一个/指定的 case，就必须显式使用 `nextcase`：
```c
switch MAIN: (enum_var) {
    case FOO:
        switch (i) {
            case 1:
                doSomething();
                nextcase 3; // 直接跳转到 case 3
            case 2:
                doSomethingElse();
            case 3:
                nextcase rand(); // 动态跳转到随机的 case
            default:
                io::printn("Ended");
                nextcase MAIN: BAR; // 跳出当前 switch，跳到外层 MAIN 的 BAR 分支！
        }
    case BAR:
        io::printn("BAR");
    default:
        break;
}
```
- `nextcase;`：跳到物理位置上的下一个 case。
- `nextcase 3;`：直接跳到 `case 3:`（像一种安全的局部 `goto`）。
- `nextcase MAIN: BAR;`：甚至能跳到指定标签外层 `switch` 的某个分支，非常适合用来写状态机或解析器。
## 运行时求值的 Switch
`switch` 可以作为增强版的 `if-else` 链使用。如果省略 switch 后的条件，默认等同于 `switch (true)`。
```c
// 简写形式：省略条件，等同于 switch (true)
switch {
    case x < 0:
        xless();
    case x > 0:
        xgreater();
    default:
        xequals();
}
```
## 运行时 Switch 中的 nextcase
对于运行时求值的 switch，`nextcase` 依然跳到下一个 case。如果带着表达式（例如 `nextcase <expr>`），它会重新从 switch 的顶部开始再次进行按条件评估匹配。  
在一个 `switch (true)` 语句中：
- 使用普通的 `nextcase;`：直接按顺序跳到下一个 case 块。
- 使用 `nextcase 值;`：程序会从 switch 最顶部重新开始计算每个 case 的逻辑，寻找第一个匹配该"值"的分支跳转。
## 使用 `@jump` 的跳转表 Switch
只包含枚举或整数 case 的普通 switch 可以使用 `@jump` 属性。这会强制编译器将 switch 实现为"跳转表（Jump Table）"。
## 总结一下

| 控制流特性 | C 语言 | C3 语言 |
| --- | --- | --- |
| 多层 break | 不支持，需借助 `goto` | 支持标签，如 `break FOO;` |
| 单次作用域 | 需要写 `do { ... } while(0);` | 可直接写 `do { ... };` |
| Switch 贯穿 | 默认贯穿，易写出 bug | 默认隔绝，需要显式写 `nextcase` |
| Switch 条件 | 仅支持编译期常量 | 支持运行时动态表达式（代替 `if-else`） |
| 跳转表强制 | 依赖 GCC/Clang 专有扩展 | 语言原生内置 `@jump` 属性 |
