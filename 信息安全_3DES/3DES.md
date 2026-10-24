# 3DES：从视频原理到 C 代码实现

沿着原笔记的思路，先看 DES 的轮函数，再看轮密钥、16 轮迭代和 3DES 的组合，最后重点记录怎样把单块算法用于文件：读写、CBC、填充、末块判断和失败清理。

已经理解算法时，可以直接从第七节开始，重点看文件处理的完整代码和长度示例。

工程现在只保留一套简洁注释代码，合并为 **3 个 `.c` 和 2 个 `.h`**。单块算法放在一起，文件处理单独保留，入口负责组织调用：

| 文件 | 职责 |
| --- | --- |
| [des3.c](../../Study/信息安全/3DES实验/des3.c)、[des3.h](../../Study/信息安全/3DES实验/des3.h) | 字节转换、轮密钥、DES 与 3DES 单块运算 |
| [file_crypto.c](../../Study/信息安全/3DES实验/file_crypto.c)、[file_crypto.h](../../Study/信息安全/3DES实验/file_crypto.h) | 文件头、CBC、8 字节读写、末块填充 |
| [main.c](../../Study/信息安全/3DES实验/main.c) | 参数解析、IV、打开与关闭文件、错误提示和失败清理 |

下面代码摘自这套重构后的源码。算法部分保留核心函数；文件部分贴出完整的加解密流函数及入口中的文件操作。片段仍依赖工程中的头文件和固定表，编译时使用完整工程。

## 一、先分清这些位数

| 对象 | 长度 | 在代码中的表示 |
| --- | --- | --- |
| 一个 DES / 3DES 数据分组 | 64 位，即 8 字节 | `uint64_t block` |
| DES 左半部、右半部 | 各 32 位 | `uint32_t left`、`right` |
| DES 输入密钥 | 64 位，其中 8 位为奇偶校验位 | `uint64_t raw_key` |
| DES 实际参与运算的密钥位 | 56 位 | PC1 的输出 |
| 一轮使用的轮密钥 | 48 位 | `key->round[i]` 的低 48 位 |
| 三密钥 3DES 的输入密钥 | 3 × 8 = 24 字节 | `raw[KEY_SIZE]` |

**分组长度与密钥长度是两回事。** 3DES 虽然使用三段密钥，但每次处理的数据仍然只有 8 字节。

本文用 `k_i` 表示 DES 第 i 轮的轮密钥，用 `K1`、`K2`、`K3` 表示 3DES 的三段 DES 密钥，避免把两种“密钥”混在一起。

## 二、DES 的轮函数：32 位输入如何变回 32 位输出

原笔记记录的是：

> 原文（32 位）→ 扩散到 48 位 → 分组混淆 → 32 位 → 随机置换。

更准确的流程是：

```text
当前右半部 R（32 位）
    ↓ E 扩展置换
48 位
    ↓ 与当前轮密钥 k_i（48 位）异或
48 位
    ↓ 分成 8 组，每组 6 位，分别进入 8 个 S 盒
8 × 4 = 32 位
    ↓ P 置换
32 位轮函数结果 f(R, k_i)
```

这里要注意三点：

1. 输入是**当前轮的右半块**，不是独立的“32 位原文”。第一轮之后，它已经fp是前面运算产生的中间结果。
2. E 扩展与 S 盒之间还有一步：**和本轮的 48 位轮密钥异或**。
3. E、P、IP、FP 都使用算法规定的**固定表**，不会在每次加密时随机打乱位置。

### 2.1 对应代码：`des_f`

来源：[des3.c](../../Study/信息安全/3DES实验/des3.c)，函数 `des_f`。

```c
static uint32_t des_f(uint32_t right, uint64_t subkey)
{
    uint64_t mixed = permute(right, E, 48, 32) ^ subkey;
    uint32_t substituted = 0;
    for (int i = 0; i < 8; ++i) {
        unsigned six = (unsigned)((mixed >> (42 - 6 * i)) & 0x3fU);
        unsigned row = ((six & 0x20U) >> 4) | (six & 1U);
        unsigned col = (six >> 1) & 0x0fU;
        substituted = (substituted << 4) | SBOX[i][row][col];
    }
    return (uint32_t)permute(substituted, P, 32, 32);
}
```

对照上面的流程阅读：

| 代码 | 对应操作 |
| --- | --- |
| `permute(right, E, 48, 32)` | 从 32 位中按 E 表选取并重复部分位，得到 48 位 |
| `^ subkey` | 与本轮的 48 位轮密钥异或；C 中 `^` 是异或，不是乘方 |
| `mixed >> (42 - 6 * i)` | 从高位到低位，依次把第 i 个六位组移到低位 |
| `& 0x3fU` | 只保留低 6 位；`0x3f` 的二进制是 `111111` |
| `row`、`col` | 用首尾两位计算行号，中间四位计算列号 |
| `SBOX[i][row][col]` | 查第 i 个 S 盒，得到一个 0～15 的四位结果 |
| `(substituted << 4) \| ...` | 将 8 个四位结果依次拼成 32 位 |
| `permute(substituted, P, 32, 32)` | 按 P 表重新排列这 32 位 |

例如 S 盒输入为 `101011`：首尾组成 `11`，行号为 3；中间四位为 `0101`，列号为 5。代码数组下标从 0 开始，因此查询第一个 S 盒时是 `SBOX[0][3][5]`。

**轮函数不等于完整的一轮 DES。** `des_f` 只负责上面这条 32 位到 32 位的路径；一轮 DES 还要把它的结果与旧左半部异或，并更新左右半部，见第四节。

### 2.2 固定置换表如何在代码中执行：`permute`

来源：[des3.c](../../Study/信息安全/3DES实验/des3.c)，私有函数 `permute`。

```c
static uint64_t permute(uint64_t value, const int *table, int output_bits, int input_bits)
{
    uint64_t result = 0;
    for (int i = 0; i < output_bits; ++i) {
        result = (result << 1) | ((value >> (input_bits - table[i])) & UINT64_C(1));
    }
    return result;
}
```

DES 表中的位号从 1 开始，1 表示输入最高位；C 数组下标从 0 开始。`input_bits - table[i]` 将标准位号转换为需要右移的距离，右移后通过 `& 1` 取出这一位，再接到 `result` 末尾。

这个函数只做“按表取位”，实际效果由表决定：

| 表 | 输入 → 输出 | 用途 |
| --- | --- | --- |
| `IP` | 64 → 64 | 数据分组的初始置换 |
| `FP`，即 IP 的逆置换 | 64 → 64 | 数据分组的最终置换 |
| `E` | 32 → 48 | 右半部扩展，部分位会被重复选择 |
| `P` | 32 → 32 | S 盒输出的置换 |
| `PC1` | 64 → 56 | 去掉密钥校验位并重排 |
| `PC2` | 56 → 48 | 选出每一轮的轮密钥 |

完整表格和 8 个 S 盒都在 `des3.c` 开头，不需要自己随机生成。这些表以及 `permute`、`des_f` 都使用 `static`，只供 DES 模块内部调用。

## 三、DES 的密钥：从 64 位生成 16 个 48 位轮密钥

### 3.1 原始密钥、有效位数与轮密钥

原笔记中“8 位一组，每组最后一位为奇偶校验位，实际有效位数为 56 位”可以保留。

完整过程为：

```text
原始 DES 密钥（64 位）
    ↓ PC1：去掉 8 个奇偶校验位，并重新排列
56 位
    ↓ 拆成左右两半
C0（28 位）和 D0（28 位）
    ↓ 每轮分别循环左移 1 位或 2 位
Ci（28 位）和 Di（28 位）
    ↓ 拼接
56 位
    ↓ PC2 置换选择
第 i 轮的轮密钥 k_i（48 位）
```

**轮密钥是 48 位，是为了与 E 扩展后的 48 位结果异或。** 它不是与扩展前的 32 位直接对应。

代码中的 PC1 从原始 64 位密钥一次选出 56 位，不需要先手工删除校验位，再对已经变短的密钥使用 PC1。

### 3.2 每轮的循环左移次数

保留视频笔记中的截图：

![[Pasted image 20261021151615.png]]

对应 `des3.c` 中的 `SHIFTS` 表：

```c
static const int SHIFTS[16] = {
    1, 1, 2, 2, 2, 2, 2, 2,
    1, 2, 2, 2, 2, 2, 2, 1
};
```

第 1、2、9、16 轮左移 1 位，其余轮左移 2 位。**每轮在上一轮的 C、D 基础上继续旋转**，不是每轮都从 C0、D0 重新开始。

### 3.3 对应代码：`DesKey` 和 `des_key_init`

来源：[des3.h](../../Study/信息安全/3DES实验/des3.h)，保存 16 个轮密钥的结构体：

```c
typedef struct {
    uint64_t round[16];
} DesKey;
```

每个轮密钥只有 48 位，但 C 没有标准的 48 位整数类型，所以使用 `uint64_t` 保存，实际只用低 48 位。

来源：[des3.c](../../Study/信息安全/3DES实验/des3.c)，函数 `des_key_init`：

```c
void des_key_init(DesKey *key, uint64_t raw_key)
{
    uint64_t selected = permute(raw_key, PC1, 56, 64);
    uint32_t c = (uint32_t)(selected >> 28);
    uint32_t d = (uint32_t)(selected & UINT64_C(0x0fffffff));
    for (int i = 0; i < 16; ++i) {
        int shift = SHIFTS[i];
        c = ((c << shift) | (c >> (28 - shift))) & 0x0fffffffU;
        d = ((d << shift) | (d >> (28 - shift))) & 0x0fffffffU;
        key->round[i] = permute(((uint64_t)c << 28) | d, PC2, 48, 56);
    }
}
```

关键代码含义如下：

| 代码 | 含义 |
| --- | --- |
| `permute(raw_key, PC1, 56, 64)` | 从原始 64 位密钥取得 56 位有效排列 |
| `selected >> 28` | 取高 28 位，得到 C0 |
| `selected & 0x0fffffff` | 取低 28 位，得到 D0 |
| `SHIFTS[i]` | 查出本轮应移动 1 位还是 2 位 |
| `(c << shift) \| (c >> (28 - shift))` | 左移时把移出去的高位接回低位，形成循环左移 |
| `& 0x0fffffffU` | 限制结果只保留 28 位 |
| `((uint64_t)c << 28) \| d` | 将 C、D 拼接为 56 位；先转为 64 位，避免在 32 位类型内拼接 |
| `permute(..., PC2, 48, 56)` | 从这 56 位中选出本轮所需的 48 位 |

16 次循环结束后，`key->round[0]` 到 `key->round[15]` 就是 16 个轮密钥，后面处理各个数据分组时重复使用。

## 四、DES 的完整加密：64 位分组经过 16 轮

原笔记中对左右部分交替更新的描述，可以整理成两个公式：

```text
L_i = R_(i-1)
R_i = L_(i-1) XOR f(R_(i-1), k_i)
```

也就是：**新左半部 = 旧右半部；新右半部 = 旧左半部 XOR 轮函数结果。** 两个赋值都应基于这一轮开始时的旧值。

完整顺序是：

```text
64 位输入分组
    ↓ IP 初始置换
拆成 L0、R0，各 32 位
    ↓ 按公式执行 16 轮
得到 L16、R16
    ↓ 最后交换并拼接
R16 || L16
    ↓ FP 最终置换，即 IP 的逆置换
64 位输出分组
```

先进行 IP，再拆分左右半部。`R16 || L16` 里的 `||` 是“拼接”的数学记号，不是这里要执行的 C 逻辑或运算。

### 4.1 对应代码：`des_block`

来源：[des3.c](../../Study/信息安全/3DES实验/des3.c)，函数 `des_block`。传入 `decrypt = 0` 表示加密。

```c
uint64_t des_block(uint64_t block, const DesKey *key, int decrypt)
{
    uint64_t initial = permute(block, IP, 64, 64);
    uint32_t left = (uint32_t)(initial >> 32);
    uint32_t right = (uint32_t)initial;
    for (int i = 0; i < 16; ++i) {
        int index = decrypt ? 15 - i : i;
        uint32_t next = left ^ des_f(right, key->round[index]);
        left = right;
        right = next;
    }
    return permute(((uint64_t)right << 32) | left, FP, 64, 64);
}
```

这段代码与原理逐步对应：

1. `initial` 是整个 64 位分组经过 IP 后的结果。
2. `initial >> 32` 取出左半部；转成 `uint32_t` 取出低 32 位作为右半部。
3. 加密时 `index = i`，按正序取轮密钥。
4. `next` 先保存新右半部，再执行 `left = right`、`right = next`，避免更新变量时覆盖仍然需要的旧值。
5. 循环结束后，`right` 放到高 32 位、`left` 放到低 32 位，完成最终交换和拼接，再执行 FP。

## 五、DES 解密：结构相同，轮密钥顺序相反

原笔记写的是“DES 为对称加密，将密文重复上述步骤即可得到原文”。这里要补充：

> 对称加密表示加密和解密使用同一份秘密密钥，并不表示用完全相同的加密操作再处理一次就一定能恢复明文。DES 解密复用相同结构，但要倒序使用轮密钥。

```text
加密：k1 → k2 → … → k16
解密：k16 → k15 → … → k1
```

对应 `des_block` 中这一行：

```c
int index = decrypt ? 15 - i : i;
```

传入 `decrypt = 1` 时，`index` 依次为 15、14、…、0；不需要重新生成另一套轮密钥，也不需要再写一个逆 S 盒。

为什么这种结构可以逆推？从一轮的结果 `L_i`、`R_i` 可以得到：

```text
R_(i-1) = L_i
L_(i-1) = R_i XOR f(L_i, k_i)
```

因为 `A XOR B XOR B = A`，异或同一个值两次会抵消。因此 Feistel 结构的可逆性不要求轮函数本身可逆。

## 六、3DES：复用 DES 做三次运算

### 6.1 加密与解密顺序

原笔记中的顺序是正确的。我们的工程使用三密钥 3DES，记为 `K1`、`K2`、`K3`：

```text
加密：明文块 → DES 加密 K1 → DES 解密 K2 → DES 加密 K3 → 密文块
解密：密文块 → DES 解密 K3 → DES 加密 K2 → DES 解密 K1 → 明文块
```

用 `E(K, X)` 表示 DES 加密，`D(K, X)` 表示 DES 解密：

```text
3DES 加密：C = E(K3, D(K2, E(K1, P)))
3DES 解密：P = D(K1, E(K2, D(K3, C)))
```

嵌套表达式从最内层开始执行。解密需要倒过来撤销每一步，所以密钥顺序也必须反过来。

这里的“三次”指对**同一个 64 位分组**连续进行三次 DES 单块运算。三段输入密钥合计 24 字节，各段的 8 个校验位都不参与 DES 运算。

### 6.2 对应代码：三套轮密钥

来源：[des3.h](../../Study/信息安全/3DES实验/des3.h)，类型 `TripleKey`：

```c
typedef struct {
    DesKey keys[3];
} TripleKey;
```

其中 `keys[0]`、`keys[1]`、`keys[2]` 分别保存 K1、K2、K3 展开后的 16 个轮密钥。

来源：[des3.c](../../Study/信息安全/3DES实验/des3.c)，函数 `triple_key_init`：

```c
void triple_key_init(TripleKey *key, const uint8_t raw[KEY_SIZE])
{
    for (int i = 0; i < 3; ++i) {
        des_key_init(&key->keys[i], load_be64(raw + i * BLOCK_SIZE));
    }
}
```

`BLOCK_SIZE = 8`，因此 `raw + i * BLOCK_SIZE` 的偏移依次为 0、8、16 字节。每次读取一段 8 字节密钥，通过 `des_key_init` 生成对应的 16 个轮密钥。整个 3DES 共保存三套、总计 48 个轮密钥。

### 6.3 对应代码：`triple_block`

来源：[des3.c](../../Study/信息安全/3DES实验/des3.c)，函数 `triple_block`：

```c
uint64_t triple_block(uint64_t block, const TripleKey *key, int decrypt)
{
    if (decrypt) {
        block = des_block(block, &key->keys[2], 1);
        block = des_block(block, &key->keys[1], 0);
        return des_block(block, &key->keys[0], 1);
    }
    block = des_block(block, &key->keys[0], 0);
    block = des_block(block, &key->keys[1], 1);
    return des_block(block, &key->keys[2], 0);
}
```

调用 `des_block` 时，最后一个参数 `0` 表示 DES 加密，`1` 表示 DES 解密。所以加密路径是 `keys[0]/0 → keys[1]/1 → keys[2]/0`，解密路径是 `keys[2]/1 → keys[1]/0 → keys[0]/1`。

DES 和 3DES 现在都在 `des3.c` 中：`triple_block` 直接复用 `des_block`，不需要再跨一个独立模块，也不需要重复实现 IP、S 盒或 16 轮迭代。

## 七、重点补充：怎样加解密一个完整文件

`triple_block` 一次只能处理 8 字节，文件可能有 0 字节、3 字节或几百 MB。文件处理要解决的是：**按顺序读入字节、补齐末块、维护 CBC 链值，把结果写出，再正确关闭文件。**

```text
main：检查参数 → 准备密钥和 IV → 打开输入、输出文件
                               ↓
加密：写文件头 → 每次读取 8 字节 → 必要时填充 → CBC 加密 → 写出 8 字节
解密：读文件头 → 读取当前块并预读下一块 → CBC 解密 → 末块去填充 → 写出有效字节
                               ↓
main：检查处理结果 → 关闭两个文件 → 成功保留输出，失败清理输出
```

### 7.1 文件不是字符串：先理解 `FILE *`、打开方式和返回值

`FILE *` 指向 C 标准库管理的文件流，流里记录当前位置、缓冲区、错误和文件结束状态。它不是“整个文件内容的指针”。`fopen` 成功返回这个流指针，失败返回 `NULL`；每次读取后，位置自动向后移动。

入口使用的两种模式是：

| 调用 | 含义 |
| --- | --- |
| `fopen(argv[2], "rb")` | 按二进制读取已有输入文件 |
| `fopen(argv[3], "wbx")` | 按二进制新建输出文件；`x` 要求路径尚不存在，已有文件会导致打开失败 |

`b` 表示按字节处理，避免某些平台的文本模式转换换行。图片、压缩包、文本在这里都是字节序列；中文文本也不需要先解码。输出使用 `x`，因此误把输出路径写成输入路径时会失败，不会先清空原文件。

读取一块使用 `fread(block, 1, BLOCK_SIZE, input)`：

| 参数 | 在这里的含义 |
| --- | --- |
| `block` | 接收数据的数组 |
| `1` | 每个数据项的大小是 1 字节 |
| `BLOCK_SIZE` | 最多读 8 个数据项 |
| `input` | 从哪个文件流读 |
| 返回值 `count` | 实际读到多少个数据项；因为每项为 1 字节，所以也就是字节数 |

不要改成 `fread(block, BLOCK_SIZE, 1, input)` 后仍把返回值当作字节数：那样返回的是完整的“8 字节项”数量，无法直接表示末尾读到了 3 个字节。

写入同理：`fwrite(block, 1, BLOCK_SIZE, output)` 成功应返回 8；少于期望数量就返回写入错误。解密最后一块只写 `used` 个有效字节，所以比较的是 `fwrite(plain, 1, used, output) != used`。

这里有两个容易写错的地方：

- **不用 `strlen(block)` 计算文件数据长度。** 文件中可以有 `00`，它会让字符串长度计算提前结束；块数组也未必带字符串结束符，继续扫描还可能越界。文件长度依据 `fread` 的返回值。入口中的 `parse_hex` 可以对命令行字符串用 `strlen`，因为那才是以 `\0` 结尾的字符串。
- **不用 `while (!feof(input))` 预测还能否读到数据。** `feof` 反映已经发生的读取是否遇到文件尾，无法预知下一次读取。程序先执行 `fread`，再检查 `ferror(input)` 和实际数量；读取错误不能当作正常文件尾去填充。

整个过程只保留少量块数组，不把文件一次性装进内存，也不需要先计算文件总长度。

### 7.2 字节数组怎样交给 64 位算法

文件读到的是 `uint8_t[8]`，算法处理的是 `uint64_t`，所以需要明确转换顺序。这里统一用**大端顺序：文件中先出现的字节放在整数高位**。

来源：[des3.c](../../Study/信息安全/3DES实验/des3.c)，两个完整转换函数：

```c
uint64_t load_be64(const uint8_t bytes[8])
{
    uint64_t value = 0;
    for (int i = 0; i < 8; ++i) {
        value = (value << 8) | bytes[i];
    }
    return value;
}

void store_be64(uint64_t value, uint8_t bytes[8])
{
    for (int i = 7; i >= 0; --i) {
        bytes[i] = (uint8_t)(value & 0xffU);
        value >>= 8;
    }
}
```

例如 `01 23 45 67 89 AB CD EF` 被组装为 `0x0123456789ABCDEF`。读取时每次左移 8 位，再拼入一个字节；写回时不断取低 8 位，从数组末尾往前填。

显式转换使文件表示不依赖机器的内存字节序，也避免直接把字节数组强转为整数指针带来的对齐问题。**转换只改变表示形式，不是加密步骤。**

### 7.3 CBC 链值和 16 字节文件头

令 `P_i` 表示第 i 个明文块、`C_i` 表示第 i 个密文块，`C_0 = IV`：

```text
加密：C_i = 3DES加密(P_i XOR C_(i-1))
解密：P_i = 3DES解密(C_i) XOR C_(i-1)
```

代码中的 `previous` 就是 IV 或前一个密文块。加密时先异或再做 3DES；解密时先做 3DES 再异或。

为了让解密能够取得同一个 IV，密文文件按下面格式保存：

```text
偏移 0～7       偏移 8～15       偏移 16 开始
┌──────────────┬──────────────┬───────────────────────────┐
│ T3CBC001     │ 8 字节 IV    │ 8 字节 C1、8 字节 C2、……   │
└──────────────┴──────────────┴───────────────────────────┘
```

`T3CBC001` 是本实验的 8 字节格式标识，不包含字符串结束符；文件头既不参与分组加密，也不保存密钥。IV 不需要保密，解密时从头部读出，密钥仍由命令行提供。

`main.c` 的 `random_iv` 默认从 `/dev/urandom` 读取 8 字节 IV；演示时也可以传入固定 IV，便于比较结果。同样的文件和密钥，使用不同 IV 会得到不同的密文。日常运行省略这个可选参数即可。

### 7.4 ANSI X9.23：怎样补齐，怎样恢复真实长度

最后不足 8 字节时，补 `n = 8 - used` 字节：前 `n−1` 字节为 `00`，最后一个字节记录 `n`。我们的实现检查并使用这一零填充形式。

```text
原文 ABC：       41 42 43
填充后：        41 42 43 00 00 00 00 05
                └─原文─┘ └─补 5 字节──┘

空文件，或原文长度正好为 8 的倍数时，额外填充一整块：
00 00 00 00 00 00 00 08
```

即使长度恰好是 8 的倍数，也必须有一个专门的填充末块，否则解密端无法判断末字节是原文还是填充长度。文件头不另存原文长度，解密就靠末块填充恢复它。

来源：[file_crypto.c](../../Study/信息安全/3DES实验/file_crypto.c)，完整填充函数。它们只服务文件处理，因此合并到这个文件并用 `static` 限定为内部函数：

```c
static void add_padding(uint8_t block[BLOCK_SIZE], size_t used)
{
    size_t padding = BLOCK_SIZE - used;
    memset(block + used, 0, padding);
    block[BLOCK_SIZE - 1] = (uint8_t)padding;
}

static int remove_padding(const uint8_t block[BLOCK_SIZE], size_t *used)
{
    size_t padding = block[BLOCK_SIZE - 1];
    if (padding < 1 || padding > BLOCK_SIZE) {
        return 0;
    }
    for (size_t i = BLOCK_SIZE - padding; i < BLOCK_SIZE - 1; ++i) {
        if (block[i] != 0) {
            return 0;
        }
    }
    *used = BLOCK_SIZE - padding;
    return 1;
}
```

`add_padding` 的调用条件保证 `used` 为 0～7。它先把剩余空间清零，再在末字节记录填充长度。`remove_padding` 依次做三件事：检查末字节在 1～8 内，检查填充区其余字节都是零，把真实长度写到调用者传入的 `*used`。

不能直接“删除末尾所有零”：原始文件完全可以以零字节结尾。也不能在每一块上去填充，因为中间块的末字节只是普通数据。

### 7.5 加密：完整函数与逐步对应

来源：[file_crypto.c](../../Study/信息安全/3DES实验/file_crypto.c)，完整 `encrypt_stream`：

```c
FileCryptoResult encrypt_stream(FILE *input, FILE *output, const TripleKey *key, const uint8_t iv[BLOCK_SIZE])
{
    uint8_t block[BLOCK_SIZE];
    uint64_t previous = load_be64(iv);
    /* 先写 16 字节头部：8 字节标识 + 8 字节 IV，不保存密钥。 */
    if (fwrite(MAGIC, 1, sizeof MAGIC, output) != sizeof MAGIC) {
        return FILE_CRYPTO_IO_ERROR;
    }
    if (fwrite(iv, 1, BLOCK_SIZE, output) != BLOCK_SIZE) {
        return FILE_CRYPTO_IO_ERROR;
    }
    for (;;) {
        /* 每项 1 字节，所以返回值就是实际读到的字节数。 */
        size_t count = fread(block, 1, BLOCK_SIZE, input);
        if (ferror(input)) {
            return FILE_CRYPTO_IO_ERROR;
        }
        /* 空文件或整块文件末尾会读到 0，此时也要补一整块。 */
        int last = count < BLOCK_SIZE;
        if (last) {
            add_padding(block, count);
        }
        /* CBC 加密：先异或前一密文，再做 3DES；结果作为新的链值。 */
        previous = triple_block(load_be64(block) ^ previous, key, 0);
        store_be64(previous, block);
        if (fwrite(block, 1, BLOCK_SIZE, output) != BLOCK_SIZE) {
            return FILE_CRYPTO_IO_ERROR;
        }
        if (last) {
            return FILE_CRYPTO_OK;
        }
    }
}
```

这段循环可以按六步理解：

1. 把 IV 变成整数，放进 `previous`；先向输出写入标识和 IV，共 16 字节。
2. 每次最多读 8 字节，先检查有没有读取错误。
3. `count == 8` 时是完整数据块，直接处理；`count < 8` 时标记为 `last`，补齐后处理。
4. `load_be64(block) ^ previous` 完成 CBC 异或，再由 `triple_block(..., 0)` 加密。结果赋给 `previous`，供下一轮使用。
5. `store_be64` 把密文转回数组，`fwrite` 总是写出完整的 8 字节密文块。
6. 写完填充末块就成功返回，否则继续读取下一块。

为什么读到 0 也必须执行一次加密？看具体长度：

| 原文长度 | 各次 `fread` 返回值 | 最后一块如何形成 | 密文数据块数 | 含 16 字节头部的总长度 |
| --- | --- | --- | --- | --- |
| 0 字节 | `0` | 补一整块 | 1 | 24 字节 |
| 3 字节 | `3` | 3 字节原文 + 5 字节填充 | 1 | 24 字节 |
| 8 字节 | `8 → 0` | 第一个块是原文，再补一整块 | 2 | 32 字节 |
| 9 字节 | `8 → 1` | 末块为 1 字节原文 + 7 字节填充 | 2 | 32 字节 |
| 16 字节 | `8 → 8 → 0` | 两个原文块之后再补一整块 | 3 | 40 字节 |

当原文长度为 `N` 时，文件总长度是 `16 + 8 × (⌊N / 8⌋ + 1)`。程序不需要提前计算它；读到不足一块时完成填充，自然得到相同结果。

例如 `N = 8` 时，第一次正好读满并不能说明后面还有没有数据。下一次读到 0 后，才知道文件已经结束，并生成独立填充块。这也是先读、再判断实际数量的意义。

### 7.6 解密：为什么要保留 `current` 和预读 `next`

加密时只需知道本次读了多少字节；解密时，每个正常密文块都恰好是 8 字节，仅靠当前块长度看不出它是不是末块。要决定是否去填充，必须再往后读一次。

四个数组分别保存：`header` 是文件头，`current` 是正在处理的密文，`next` 是预读的密文，`plain` 是解密后的明文。先读入一个完整 `current`，然后在循环中读 `next`：

| 读取 `next` 的结果 | 意义 | 对 `current` 的处理 |
| --- | --- | --- |
| 无错误，读到 8 字节 | 后面还有一块 | 解密并写出全部 8 字节 |
| 无错误，读到 0 字节 | 当前就是最后一块 | 解密、验证填充，只写有效字节 |
| 无错误，读到 1～7 字节 | 密文被截断，末尾不成块 | 返回长度错误 |
| `ferror(input)` 非零 | 读取过程中出错 | 返回读写错误 |

来源：[file_crypto.c](../../Study/信息安全/3DES实验/file_crypto.c)，完整 `decrypt_stream`：

```c
FileCryptoResult decrypt_stream(FILE *input, FILE *output, const TripleKey *key)
{
    uint8_t header[16];
    uint8_t current[BLOCK_SIZE];
    uint8_t next[BLOCK_SIZE];
    uint8_t plain[BLOCK_SIZE];
    /* 校验文件头后恢复 IV；即使原文为空，也至少存在一个填充分组。 */
    if (fread(header, 1, sizeof header, input) != sizeof header || ferror(input)) {
        return ferror(input) ? FILE_CRYPTO_IO_ERROR : FILE_CRYPTO_BAD_HEADER;
    }
    if (memcmp(header, MAGIC, sizeof MAGIC) != 0) {
        return FILE_CRYPTO_BAD_HEADER;
    }
    uint64_t previous = load_be64(header + sizeof MAGIC);
    if (fread(current, 1, BLOCK_SIZE, input) != BLOCK_SIZE || ferror(input)) {
        return ferror(input) ? FILE_CRYPTO_IO_ERROR : FILE_CRYPTO_BAD_LENGTH;
    }
    for (;;) {
        /* 预读下一块：0 表示当前是末块，8 表示后面还有数据。 */
        size_t count = fread(next, 1, BLOCK_SIZE, input);
        if (ferror(input) || (count != 0 && count != BLOCK_SIZE)) {
            return ferror(input) ? FILE_CRYPTO_IO_ERROR : FILE_CRYPTO_BAD_LENGTH;
        }
        /* 先保留当前原始密文，下一轮需要用它作为 CBC 链值。 */
        uint64_t cipher = load_be64(current);
        uint64_t value = triple_block(cipher, key, 1) ^ previous;
        store_be64(value, plain);
        /* 只有最后一块去填充；普通块的 8 字节必须原样写出。 */
        size_t used = BLOCK_SIZE;
        if (count == 0 && !remove_padding(plain, &used)) {
            return FILE_CRYPTO_BAD_PADDING;
        }
        if (fwrite(plain, 1, used, output) != used) {
            return FILE_CRYPTO_IO_ERROR;
        }
        if (count == 0) {
            return FILE_CRYPTO_OK;
        }
        previous = cipher;
        memcpy(current, next, BLOCK_SIZE);
    }
}
```

按执行顺序看，关键点是：

1. **先验证格式。** 必须读满 16 字节头部，前 8 字节与 `MAGIC` 一致；`memcmp` 按指定长度比较字节，不依赖字符串结束符。
2. **恢复 IV，读取首块。** `header + sizeof MAGIC` 跳过标识，从第 9 个字节开始取 IV。首个密文块必须完整，只有文件头也算格式不完整；原文为空时仍应有一个填充块。
3. **先预读，再处理当前块。** `count` 是 `next` 的字节数。通过它判断 `current` 是普通块还是末块。
4. **保存原始密文。** `cipher = load_be64(current)` 保存的必须是解密前的值。当前明文是 `3DES解密(cipher) ^ previous`，下一轮的链值则必须是当前原始密文 `cipher`，不能用明文替代。
5. **决定写多少。** `used` 默认是 8；只有末块才调用 `remove_padding` 修改它。空文件的填充块去掉后 `used = 0`，`fwrite` 写 0 字节，恢复出空文件。
6. **推进到下一块。** 普通块写完后，执行 `previous = cipher`，再把 `next` 复制到 `current`。这样本轮预读到的块成为下一轮的处理对象；每个密文块只从文件中读一次。

以两个密文块为例：

```text
读头部并取 IV；current = C1
第 1 轮：读 next 得 C2 → P1 = D(C1) XOR IV → 写 8 字节
         previous = C1；current = C2
第 2 轮：读 next 得 0  → P2 = D(C2) XOR C1 → 去填充 → 写有效字节 → 结束
```

### 7.7 谁关闭文件，处理到一半失败怎么办

两个流函数只借用 `FILE *` 做读写，通过 `FileCryptoResult` 返回状态；**由打开文件的 `main` 统一关闭文件并处理输出结果。** 状态定义在 [file_crypto.h](../../Study/信息安全/3DES实验/file_crypto.h)：

| 返回值 | 含义 |
| --- | --- |
| `FILE_CRYPTO_OK` | 分组处理完成，入口还需检查关闭结果 |
| `FILE_CRYPTO_IO_ERROR` | 读写未成功 |
| `FILE_CRYPTO_BAD_HEADER` | 头部不完整或标识不匹配 |
| `FILE_CRYPTO_BAD_LENGTH` | 没有首个完整密文块，或后面的密文不足一块 |
| `FILE_CRYPTO_BAD_PADDING` | 末块填充长度或零字节检查失败 |

来源：[main.c](../../Study/信息安全/3DES实验/main.c)，`main` 中从密钥初始化到函数返回的完整文件操作片段。此前已经检查命令行、解析 `raw_key`，并在加密时准备好 `iv`：

```c
TripleKey key;
triple_key_init(&key, raw_key);
/* rb 按二进制读入；文件内容不作字符串或文本编码解释。 */
FILE *input = fopen(argv[2], "rb");
if (input == NULL) {
    perror("无法打开输入文件");
    return EXIT_FAILURE;
}
/* x 表示独占新建，已有输出（包括输入文件本身）不会被截断。 */
FILE *output = fopen(argv[3], "wbx");
if (output == NULL) {
    perror("无法新建输出文件（输出路径必须尚不存在）");
    fclose(input);
    return EXIT_FAILURE;
}
/* 具体分块处理交给文件模块，流的打开与关闭仍由 main 负责。 */
FileCryptoResult result;
if (encrypt) {
    result = encrypt_stream(input, output, &key, iv);
} else {
    result = decrypt_stream(input, output, &key);
}
int ok = result == FILE_CRYPTO_OK;
if (!ok) {
    fputs(crypto_error_message(result), stderr);
}
if (fclose(input) != 0) {
    ok = 0;
}
/* fwrite 可能只写入缓冲区，fclose 刷新时仍可能失败。 */
if (fclose(output) != 0) {
    ok = 0;
}
if (!ok) {
    fputs("操作失败，正在清理未完成的输出文件。\n", stderr);
    /* 只清理本次新建的不完整输出，不修改输入文件。 */
    if (remove(argv[3]) != 0) {
        perror("清理失败，请手动删除不完整的输出文件");
    }
    return EXIT_FAILURE;
}
puts(encrypt ? "加密完成。" : "解密完成。");
return EXIT_SUCCESS;
```

这里的处理顺序有明确原因：

- 输入打开失败就返回；输出新建失败时，先关闭已经打开的输入。此时没有本次新建的输出，不执行删除。
- 成功新建输出后，可能已经写过文件头或若干分组才遇到错误。因此流函数返回失败时，不能把那个不完整文件留成看起来正常的结果。
- `fwrite` 返回成功，可能只是写入了标准库缓冲区。`fclose(output)` 会刷新尚未写出的内容，刷新时也可能失败，所以关闭结果同样要检查。即使输入关闭失败，也继续关闭输出，不能提前返回而漏掉清理。
- 两个流关闭后，只有所有步骤都成功才保留输出、显示“完成”；否则调用 `remove(argv[3])` 清理本次创建的输出。若清理也失败，用 `perror` 告知用户。

填充检查能发现部分错误密钥或损坏，但这个文件格式没有认证标签，因此“解密成功”只表示这些检查通过，不能证明内容从未被改动。

## 八、阅读与演示顺序

理解单块算法后，可以直接按下面顺序阅读文件部分：

| 顺序 | 源码位置 | 要回答的问题 |
| --- | --- | --- |
| 1 | `main.c` 的 `main` | 参数怎样变成密钥、IV 和两个文件流？ |
| 2 | `file_crypto.c` 的 `encrypt_stream` | 怎样读一块、链接前块、补末块并写出？ |
| 3 | 同文件的 `add_padding`、`remove_padding` | 怎样区分原文和填充，恢复实际长度？ |
| 4 | 同文件的 `decrypt_stream` | 怎样预读识别末块，怎样保存前一密文？ |
| 5 | 回到 `main` 后半段 | 为什么流处理成功后还要检查关闭？失败时清理谁？ |

在工程根目录编译，生成的程序位于 `build/des3`：

```bash
cd /home/yinhan/Study/信息安全/3DES实验
make
```

下面用 9 字节文件演示：第一块完整，第二块只有 1 字节，需要补 7 字节。请在工程目录下执行；临时目录保证输出路径尚不存在，比较成功时终端不会输出差异：

```bash
demo_dir=$(mktemp -d)
KEY=0123456789ABCDEFFEDCBA98765432100011223344556677
IV=1234567890ABCDEF
printf '123456789' > "$demo_dir/plain.bin"
./build/des3 enc "$demo_dir/plain.bin" "$demo_dir/encrypted.t3c" "$KEY" "$IV"
./build/des3 dec "$demo_dir/encrypted.t3c" "$demo_dir/restored.bin" "$KEY"
cmp "$demo_dir/plain.bin" "$demo_dir/restored.bin"
wc -c "$demo_dir/plain.bin" "$demo_dir/encrypted.t3c" "$demo_dir/restored.bin"
```

预期三个文件大小依次是 **9、32、9 字节**。固定 IV 只用于这段可重复的演示；省略加密命令最后的 IV 参数就会自动生成。

结构说明见 [模块划分](模块划分.md)，文件处理详解见 [实现与流程](实现与流程.md)，讲解提纲见 [汇报讲解](汇报讲解.md)。
