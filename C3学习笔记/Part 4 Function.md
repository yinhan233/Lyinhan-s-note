C3 既有常规函数，也有方法。方法是使用类型名称进行命名空间划分的函数，允许使用点语法进行调用。
## 常规函数
平平无奇：
```c
fn void test(int times) {
    for (int i = 0; i < times; i++) {
        io::printfn("Hello %d", i);
    }
}
```
## 函数参数
C3 允许使用默认参数以及命名参数。需要注意的是，任何未命名的参数必须出现在任何命名参数之前。
```c
fn int test_with_default(int foo = 1) {
    return foo;
}
fn void test() {
    test_with_default();  // 返回 1
    test_with_default(100);  // 返回 100
}
```
```c
fn void test_named(int times, double data) {
    for (int i = 0; i < times; i++) {
        io::printf("Hello %d\n", i + data);
    }
}
fn void test() {
    // 仅使用命名参数
    test_named(times: 1, data: 3.0);
    // 仅使用未命名参数
    test_named(3, 4.0);
    // 混合使用
    test_named(15, data: 3.141592);
    // 用命名参数覆盖已传递的未命名参数是错误的：
    // test_named_default(2, times: 3); 错误！
    // 未命名参数不能跟在命名参数之后：
    // test_named_default(times: 3, 4.0); 错误！
}
```
## 可变参数
有四种类型的可变参数：
1. **单一类型** (single typed)
2. **显式类型 any** (explicitly typed any)：将非 any 类型的参数作为引用传递
3. **隐式类型 any** (implicitly typed any)：参数被隐式转换为引用（需谨慎使用）
4. **无类型的 C 风格** (untyped C-style)
```c
fn void va_singletyped(int... args) { /* args 的类型是 int[] */ }
fn void va_variants_explicit(any... args) { /* args 的类型是 any[] */ }
fn void va_variants_implicit(args...) { /* args 的类型是 any[] */ }
extern fn void va_untyped(...); // 仅用于外部 C 函数

fn void test() {
    va_singletyped(1, 2, 3);
    int x = 1;
    any v = &x;
    va_variants_explicit(&&1, &x, v); // 将非 any 参数作为引用传递
    va_variants_implicit(1, x, "foo"); // 参数隐式转换为 any 类型
    va_untyped(1, x, "foo"); // 外部 C 函数
}
```
对于带类型的可变参数，我们可以通过展开操作符 `...` 来传递一个切片而不是逐个传递参数：
```c
fn void test_splat() {
    int[] x = { 1, 2, 3 };
    va_singletyped(...x);
}
```
### 展开操作符
- 展开 `...` 未知大小的切片，仅限用于带类型的可变参数槽。
- 展开 `...` 任意数组，可用于任何地方。
- 展开 `...` 已知大小的切片，可用于任何地方。
- 展开 `...` 结构体，可用于任何地方。被展开的结构体的字段必须与函数参数的顺序相同（名称不需要匹配，但类型必须匹配）。
### 命名参数与可变参数
通常，位于可变参数之后的参数是永远无法被赋值的：
```c
fn void testme(int a, double... x, double rate = 1.0) { /* ... */ }
fn void test() {
    // x 是 { 2.0, 5.0, 6.0 }，rate 会使用默认值 1.0
    testme(3, 2.0, 5.0, 6.0);
}
```
然而，可以使用命名参数来显式设置这个值：
```c
fn void test() {
    // x 是 { 2.0, 5.0 }，rate 会被设置为 6.0
    testme(3, 2.0, 5.0, rate: 6.0);
}
```
### 函数与可选返回值
函数返回值可以是"可选的"，由 `Type?` 表示。说明该函数可能返回一个带有结果的 Optional，或者返回一个带有借口（相当于错误/异常）的 Optional。  
如果传递了一个或多个 Optional 参数给函数调用，只有当所有 Optional 值都包含结果时，函数才会执行；否则，将直接返回第一个遇到的 Excuse。使用 `??` 可以在遇到 Excuse 时设置默认值，而 `if (try x)` 可以用于安全的条件解包并实现链式调用。
### 方法
方法看起来和普通函数完全一样，但是以类型名称作为前缀，并且（通常）在类型的实例上使用点（`.`）语法来调用。目标对象可以通过值传递，也可以通过指针传递。你甚至可以向所有运行时类型（包括内置的基础类型，如 `int`）添加方法。
```c
struct Point { int x; int y; }
fn void Point.add(Point* p, int x) { p.x += x; }

fn void example() {
    Point p = { 1, 2 };
    p.add(10); // 使用结构体方法
    Point.add(&p, 10); // 也可以这样等价调用
}
```
### 隐式首个参数
由于第一个参数的类型是已知的，因此可以省略类型。要指示非空指针，使用 `&`。
```c
fn int Foo.test(&self) { /* ... */ } // 等同于 fn int Foo.test(Foo* self) { /* ... */ }
```
### 契约
C3 的错误处理不是用来验证无效数据或检查不变量的。相反，C3 的做法是向函数添加注解，这些注解会有条件地编译为断言（asserts）。
```c
<*
 @param foo `foos的数量`
 @require foo > 0, foo < 1000
 @return `foos的数量乘10`
 @ensure return < 10000, return > 0
*>
fn int test_foo(int foo) {
    return foo * 10;
}
```
在 Debug 构建中，上述前置条件（`@require`）和后置条件（`@ensure`）会自动编译为 `assert()` 断言以确保安全。同时，编译器可以利用这些契约进行智能的死代码消除与优化。
### 简短函数声明语法
对于非常简短的函数，C3 提供了使用 `=>` 的"简短声明"语法：
```c
fn int square_short(int x) => x * x;
```
### Lambda 表达式
可以使用常规的 `fn` 语法创建匿名函数。匿名函数与常规函数相同，不会捕获其周围作用域的变量。
### 静态初始化器与清理器
带有 `@init` 和 `@finalizer` 注解的常规函数，将分别在程序启动和关闭时运行。你可以通过为注解提供参数来改变执行的优先级（例如 `@init(3000)`），数值越大执行越晚。建议使用 1024 或更高的优先级。
### 参数访问约束
你可以使用契约注解来限制参数的读写权限：
- `@param [in] value`：只读，修改被标记为 `in` 的参数会导致编译错误。
- `@param [out] buffer`：只写，读取被标记为 `out` 的参数会导致编译错误。