## 一、字符串处理板块
### 0. C 标准库 `<string.h>` 常用函数速查
```c
#include <stdio.h>
#include <string.h>
int main() {
    char s1[100] = "Hello", s2[100] = "World";
    char s3[100], *p;
    int len, cmp;
    // 1. strlen：求字符串长度（不包含末尾'\0'）
    len = strlen(s1);  // len = 5
    // 2. strcpy：字符串复制（把s2复制到s1，会覆盖s1原内容，需保证s1空间足够）
    strcpy(s3, s1);    // s3 = "Hello"
    // 3. strncpy：安全复制（最多复制n个字符，若n>s2长度，剩余补'\0'）
    strncpy(s3, s2, 3); // s3前3个字符为"Wor"，需手动补'\0'（若需完整字符串）
    // 4. strcat：字符串拼接（把s2拼接到s1末尾，需保证s1空间足够）
    strcat(s1, s2);    // s1 = "HelloWorld"
    // 5. strncat：安全拼接（最多拼接n个字符）
    strncat(s1, s2, 2); // s1 = "HelloWorldWo"
    // 6. strcmp：字符串比较（按字典序，相等返回0，s1<s2返回负数，s1>s2返回正数）
    cmp = strcmp("Apple", "Banana"); // cmp < 0
    // 7. strncmp：比较前n个字符
    cmp = strncmp("Apple", "Apply", 4); // cmp = 0（前4个都是"Appl"）
    // 8. strchr：查找字符第一次出现的位置（返回指针，未找到返回NULL）
    p = strchr("Hello", 'l'); // p指向第一个'l'的位置
    // 9. strrchr：查找字符最后一次出现的位置
    p = strrchr("Hello", 'l'); // p指向第二个'l'的位置
    // 10. strstr：查找子串第一次出现的位置（返回指针，未找到返回NULL）
    p = strstr("HelloWorld", "World"); // p指向"World"的起始位置
    // 11. memset：内存批量设置（按字节赋值，常用于初始化数组）
    char buf[100];
    memset(buf, 0, sizeof(buf)); // 把buf全设为0（'\0'）
    memset(buf, 'a', 5);         // 把buf前5个字符设为'a'
    // 12. memcpy：内存复制（不限制类型，比strcpy更通用，需保证空间不重叠）
    int a[5] = {1,2,3,4,5}, b[5];
    memcpy(b, a, sizeof(a)); // 把a数组复制到b数组
    return 0;
}
```
---
### 1. KMP 模式匹配算法
**说明**：在文本串中快速查找模式串，时间复杂度 O(n+m)，避免暴力匹配的回溯。
```c
#include <string.h>
// 计算next数组（next[0]=-1，next[i]表示模式串前i个字符的最长相等前后缀长度）
void get_next(char *pattern, int *next) {
    int m = strlen(pattern);
    next[0] = -1;
    int i = 0, j = -1;
    while (i < m) {
        if (j == -1 || pattern[i] == pattern[j]) {
            i++, j++;
            next[i] = j;
        } else {
            j = next[j];
        }
    }
}
// KMP匹配，返回模式串在文本串中第一次出现的起始索引，未找到返回-1
int kmp_search(char *text, char *pattern, int *next) {
    int n = strlen(text), m = strlen(pattern);
    int i = 0, j = 0;
    while (i < n && j < m) {
        if (j == -1 || text[i] == pattern[j]) {
            i++, j++;
        } else {
            j = next[j];
        }
    }
    return (j == m) ? (i - j) : -1;
}
```
---
### 2. 字符串哈希（单哈希，自然溢出）
**说明**：将字符串映射为数字，快速比较子串是否相等，Base 常用 131/13331，自然溢出避免取模（速度快）。
```c
typedef unsigned long long ULL;
const int MAXN = 1e5 + 10;  // 根据题目调整大小
const ULL BASE = 131;
ULL hash_val[MAXN], power[MAXN];  // hash_val[i]是前i个字符的哈希值，power[i]是BASE^i
// 预处理哈希值和幂次
void init_hash(char *s) {
    int n = strlen(s);
    power[0] = 1;
    for (int i = 1; i <= n; i++) {
        power[i] = power[i-1] * BASE;
        hash_val[i] = hash_val[i-1] * BASE + s[i-1];  // 注意s是0起始，hash_val是1起始
    }
}
// 计算子串s[l..r]的哈希值（l和r是0起始，闭区间）
ULL get_hash(int l, int r) {
    return hash_val[r+1] - hash_val[l] * power[r-l+1];
}
```
---
### 3. Manacher 算法（最长回文子串）
**说明**：线性时间 O(n) 求最长回文子串，通过插入分隔符统一奇偶长度回文的处理。
```c
#include <string.h>
#define MAXN 200010  // 原串长度n，处理后长度2n+1，注意开两倍空间
char t[MAXN];   // 处理后的字符串
int p[MAXN];    // p[i]表示以t[i]为中心的最长回文半径（包含t[i]）
// 预处理原串s，插入分隔符（如'#'），开头加'^'结尾加'$'避免边界判断
void manacher_init(char *s) {
    int n = strlen(s), len = 0;
    t[len++] = '^';  // 开头哨兵
    for (int i = 0; i < n; i++) {
        t[len++] = '#';
        t[len++] = s[i];
    }
    t[len++] = '#';
    t[len++] = '$';  // 结尾哨兵
    t[len] = '\0';
}
// 计算p数组，返回最长回文子串的长度
int manacher() {
	memset(p, 0, sizeof(p));
    int len = strlen(t);
    int center = 0, right = 0, max_len = 0;
    for (int i = 1; i < len - 1; i++) {  // 跳过哨兵
        // 利用对称性初始化p[i]
        if (i < right) {
            int mirror = 2 * center - i;
            p[i] = (right - i) < p[mirror] ? (right - i) : p[mirror];
        } else {
            p[i] = 1;
        }
        // 扩展回文
        while (t[i + p[i]] == t[i - p[i]]) {
            p[i]++;
        }
        // 更新center和right
        if (i + p[i] > right) {
            center = i;
            right = i + p[i];
        }
        // 更新最长回文长度
        if (p[i] - 1 > max_len) {
            max_len = p[i] - 1;
        }
    }
    return max_len;
}
```
---
### 4. Trie 树（字典树，前缀匹配）
**说明**：存储多个字符串，快速查询前缀是否存在、字符串是否存在，适合字符串统计类题目。
```c
#include <stdio.h>
#include <stdlib.h>
#include <string.h>
#define MAXN 100010  // 根据题目调整节点数
#define CHARSET 26      // 假设只有小写字母
typedef struct TrieNode {
    int count;  // 以该节点为结尾的字符串数量
    struct TrieNode *next[CHARSET];
} TrieNode;
TrieNode *root;
// 创建新节点
TrieNode* new_node() {
    TrieNode *node = (TrieNode*)malloc(sizeof(TrieNode));
    node->count = 0;
    memset(node->next, 0, sizeof(node->next));
    return node;
}
// 初始化Trie树
void trie_init() {
    root = new_node();
}
// 插入字符串s
void trie_insert(char *s) {
    TrieNode *p = root;
    int n = strlen(s);
    for (int i = 0; i < n; i++) {
        int idx = s[i] - 'a';
		if (idx < 0 || idx >= CHARSET) continue;
        if (!p->next[idx]) {
            p->next[idx] = new_node();
        }
        p = p->next[idx];
    }
    p->count++;  // 标记字符串结尾
}
// 查询字符串s是否存在，返回出现次数
int trie_search(char *s) {
    TrieNode *p = root;
    int n = strlen(s);
    for (int i = 0; i < n; i++) {
        int idx = s[i] - 'a';
        if (!p->next[idx]) {
            return 0;
        }
        p = p->next[idx];
    }
    return p->count;
}
// 查询是否有字符串以s为前缀
int trie_starts_with(char *s) {
    TrieNode *p = root;
    int n = strlen(s);
    for (int i = 0; i < n; i++) {
        int idx = s[i] - 'a';
        if (!p->next[idx]) {
            return 0;
        }
        p = p->next[idx];
    }
    return 1;
}
```

---
## 二、排序 — `qsort()` 快速排序函数（C 标准库）
### 函数原型：
```c
#include <stdlib.h>
void qsort(void *base, size_t nitems, size_t size, int (*compar)(const void *, const void *));
```
### 参数说明：
1. `base`：待排序数组的首地址（直接传数组名即可）
2. `nitems`：数组元素个数
3. `size`：单个元素的大小（用 `sizeof(元素类型)` 获取）
4. `compar`：自定义比较函数指针（核心，决定排序规则）
### 比较函数规则：
- 若 a 应排在 b 前面，返回 **负数**
- 若 a 和 b 相等，返回 **0**
- 若 a 应排在 b 后面，返回 **正数**
---
### 常见排序场景代码示例
```c
#include <stdio.h>
#include <stdlib.h>
#include <string.h>
// 场景1：普通int数组升序/降序
int cmp_int_asc(const void *a, const void *b) {
    return *(int*)a - *(int*)b;  // 升序：a < b 返回负数
}
int cmp_int_desc(const void *a, const void *b) {
    return *(int*)b - *(int*)a;  // 降序：b < a 返回负数
}
// 场景2：结构体排序（重点！）
typedef struct Student {
    int id;         // 学号（int）
    char name[20];  // 姓名（字符串）
    double score;   // 成绩（double）
} Student;
// 2.1 按学号id升序排列
int cmp_student_id(const void *a, const void *b) {
    Student *s1 = (Student*)a;  // 先强制类型转换
    Student *s2 = (Student*)b;
    return s1->id - s2->id;     // 升序
}
// 2.2 按姓名name字典序排列（用strcmp）
int cmp_student_name(const void *a, const void *b) {
    Student *s1 = (Student*)a;
    Student *s2 = (Student*)b;
    return strcmp(s1->name, s2->name);  // strcmp天然符合比较函数规则
}
// 2.3 按成绩score降序排列（注意double不能直接相减，避免精度问题）
int cmp_student_score_desc(const void *a, const void *b) {
    Student *s1 = (Student*)a;
    Student *s2 = (Student*)b;
    if (s1->score > s2->score) return -1;  // s1分高，排前面（降序）
    if (s1->score < s2->score) return 1;
    return 0;
}
// 测试主函数
int main() {
    // 测试int数组
    int arr[] = {3, 1, 4, 1, 5};
    int n = sizeof(arr)/sizeof(arr[0]);
    qsort(arr, n, sizeof(int), cmp_int_asc);
    for (int i = 0; i < n; i++) printf("%d ", arr[i]);  // 输出：1 1 3 4 5
    printf("\n");
    // 测试结构体数组
    Student stu[] = {
        {102, "Bob", 85.5},
        {101, "Alice", 92.0},
        {103, "Charlie", 78.0}
    };
    int m = sizeof(stu)/sizeof(stu[0]);
    qsort(stu, m, sizeof(Student), cmp_student_id);  // 按学号排
    for (int i = 0; i < m; i++)
        printf("%d %s %.1f\n", stu[i].id, stu[i].name, stu[i].score);
    return 0;
}
```
### 关键提醒：
1. 比较函数的参数必须是 `const void*`，先强制转成目标类型指针再访问字段
2. 比较 `double / float` 时，**不要直接相减**（精度误差可能导致错误），用 `if` 判断大小返回 1/-1
3. 字符串比较直接用 `strcmp`，它的返回值正好符合 `qsort` 的要求
---
## 三、图论核心算法板块
**说明**：全部纯 C 实现，无 C++ 依赖，适配 CCF-CSP 常考场景，代码极简好记，标注考试避坑点，节点编号默认按考试习惯从 1 开始。
### 0. 图的基础存储（2 种考试常用实现）
#### （1）邻接矩阵（适合 n≤1000 的小数据，代码极简）
```c
#include <stdio.h>
#include <string.h>
#define MAXN 1005  // 最大节点数，按题目调整
#define INF 0x3f3f3f3f  // 通用无穷大（int型，避免加法溢出，约1e9）
int graph[MAXN][MAXN];  // graph[u][v] = u到v的边权，无边则为INF
int n, m;  // n个节点，m条边
// 初始化邻接矩阵
void graph_init() {
    memset(graph, 0x3f, sizeof(graph));
    for (int i = 1; i <= n; i++) {
        graph[i][i] = 0;  // 自己到自己的边权为0
    }
}
// 添加边（无向图需加双向，有向图只加单向）
void add_edge(int u, int v, int w) {
    graph[u][v] = w;
    // 无向图取消下面注释
    // graph[v][u] = w;
}
```
#### （2）前向星（数组模拟邻接表，适合 n≤1e5 的大数据，CSP 首选）
```c
#include <stdio.h>
#include <string.h>
#define MAXN 100005  // 最大节点数
#define MAXM 200005  // 最大边数（无向图必须开2倍）
#define INF 0x3f3f3f3f
typedef struct Edge {
    int to;     // 边的终点
    int w;      // 边权
    int next;   // 同起点的上一条边的索引
} Edge;
Edge edge[MAXM];
int head[MAXN];  // head[u] = 起点u的第一条边的索引
int edge_cnt;    // 边的计数器
int n, m;        // n个节点，m条边
// 初始化前向星
void graph_init() {
    memset(head, -1, sizeof(head));
    edge_cnt = 0;
}
// 添加边（无向图需加双向，有向图只加单向）
void add_edge(int u, int v, int w) {
    edge[edge_cnt].to = v;
    edge[edge_cnt].w = w;
    edge[edge_cnt].next = head[u];
    head[u] = edge_cnt++;
    // 无向图取消下面注释
    // add_edge(v, u, w);
}
```
---
### 1. DFS 深度优先搜索
**用途**：连通性判断、路径查找、回溯类问题、拓扑排序基础，递归实现代码极简，CSP 考试首选（无特殊情况不会栈溢出）
```c
#include <stdbool.h>
#define MAXN 1005
bool vis[MAXN];  // 访问标记数组，避免重复访问
// 邻接矩阵版DFS
void dfs_matrix(int u) {
    vis[u] = true;  // 标记已访问
    printf("%d ", u);  // 可替换为题目要求的逻辑
    // 遍历所有相邻未访问节点
    for (int v = 1; v <= n; v++) {
        if (graph[u][v] != INF && !vis[v]) {
            dfs_matrix(v);
        }
    }
}
// 前向星邻接表版DFS
void dfs_adj(int u) {
    vis[u] = true;
    printf("%d ", u);
    // 遍历当前节点所有出边
    for (int i = head[u]; i != -1; i = edge[i].next) {
        int v = edge[i].to;
        if (!vis[v]) {
            dfs_adj(v);
        }
    }
}
// 调用示例：
// memset(vis, 0, sizeof(vis));
// dfs_matrix(起点编号);
```
**考试提醒**：多组测试数据时，每次必须重置 `vis` 数组和图结构！

---
### 2. BFS 广度优先搜索
**用途**：无权图最短路径、最少步数、层序遍历、连通块统计，按层遍历无递归栈溢出风险，C 语言用数组模拟队列实现
```c
#include <stdbool.h>
#define MAXN 1005
bool vis[MAXN];
int queue[MAXN];  // 数组模拟队列
int front, rear;  // 队头、队尾指针（front=队首元素，rear=队尾下一个空位）
// 邻接矩阵版BFS
void bfs_matrix(int start) {
    memset(vis, 0, sizeof(vis));
    front = rear = 0;  // 初始化队列
    // 起点入队
    queue[rear++] = start;
    vis[start] = true;
    while (front < rear) {  // 队列非空
        int u = queue[front++];  // 出队
        printf("%d ", u);  // 可替换为题目逻辑
        // 遍历所有相邻节点
        for (int v = 1; v <= n; v++) {
            if (graph[u][v] != INF && !vis[v]) {
                vis[v] = true;  // 入队即标记，避免重复入队
                queue[rear++] = v;
            }
        }
    }
}
// 前向星邻接表版BFS
void bfs_adj(int start) {
    memset(vis, 0, sizeof(vis));
    front = rear = 0;
    queue[rear++] = start;
    vis[start] = true;
    while (front < rear) {
        int u = queue[front++];
        printf("%d ", u);
        // 遍历当前节点所有出边
        for (int i = head[u]; i != -1; i = edge[i].next) {
            int v = edge[i].to;
            if (!vis[v]) {
                vis[v] = true;
                queue[rear++] = v;
            }
        }
    }
}
```
**考试核心提醒**：BFS 求最短步数时，入队时必须立即标记访问，否则会出现重复入队、超时 / 答案错误！

---
### 3. Dijkstra 迪杰斯特拉算法（单源最短路径）
**用途**：求一个起点到其他所有节点的最短路径，仅适用于无负权边的图，CSP 最常考的最短路算法  
**时间复杂度**：朴素版 O(n²)，适合 n≤1000，代码极简，考试首选
```c
#include <stdbool.h>
#define MAXN 1005
#define INF 0x3f3f3f3f
int dist[MAXN];  // dist[u] = 起点到u的最短距离
bool vis[MAXN];  // 标记是否已确定最短路径
// 邻接矩阵版（考试首选，好写不出错）
void dijkstra_matrix(int start) {
    // 1. 初始化距离数组
    memset(dist, 0x3f, sizeof(dist));
    memset(vis, 0, sizeof(vis));
    dist[start] = 0;  // 起点到自身距离为0
    // 2. 遍历n次，每次确定一个节点的最短路径
    for (int i = 1; i <= n; i++) {
        // 找当前未访问的、距离最小的节点u
        int u = -1, min_dist = INF;
        for (int j = 1; j <= n; j++) {
            if (!vis[j] && dist[j] < min_dist) {
                min_dist = dist[j];
                u = j;
            }
        }
        if (u == -1) break;  // 剩余节点不可达，提前退出
        vis[u] = true;       // 标记u的最短路径已确定
        // 3. 用u更新相邻节点的距离
        for (int v = 1; v <= n; v++) {
            if (!vis[v] && graph[u][v] != INF && dist[v] > dist[u] + graph[u][v]) {
                dist[v] = dist[u] + graph[u][v];
            }
        }
    }
}
// 前向星邻接表版（适合n稍大的场景，逻辑完全一致）
void dijkstra_adj(int start) {
    memset(dist, 0x3f, sizeof(dist));
    memset(vis, 0, sizeof(vis));
    dist[start] = 0;
    for (int i = 1; i <= n; i++) {
        int u = -1, min_dist = INF;
        for (int j = 1; j <= n; j++) {
            if (!vis[j] && dist[j] < min_dist) {
                min_dist = dist[j];
                u = j;
            }
        }
        if (u == -1) break;
        vis[u] = true;
        // 遍历u的所有出边更新距离
        for (int j = head[u]; j != -1; j = edge[j].next) {
            int v = edge[j].to;
            int w = edge[j].w;
            if (!vis[v] && dist[v] > dist[u] + w) {
                dist[v] = dist[u] + w;
            }
        }
    }
}
```
**考试避坑点**：
- 绝对不能用于有负权边的图，会直接出错
- `INF` 固定用 `0x3f3f3f3f`，不要用 `0x7fffffff`，避免加法溢出
- 多组数据必须重置 `dist`、`vis` 数组和图结构
## 四、数学核心算法板块
### 0. 模运算核心规则（考试必背！90% 数学题都要用到）
CSP 几乎所有数学题都会要求结果取模，**一步错全错**，先记死规则：
#### 1. 常用模数：
- `MOD = 1000000007`（1e9+7，质数，考试最常用）
- 其次 `998244353`
#### 2. 核心取模公式：

|运算|公式|
|---|---|
|加法|`(a + b) % MOD = ((a % MOD) + (b % MOD)) % MOD`|
|减法|`(a - b) % MOD = ((a % MOD) - (b % MOD) + MOD) % MOD`（必须加 MOD 避免负数！）|
|乘法|`(a * b) % MOD = ((a % MOD) * (b % MOD)) % MOD`|
|除法|模意义下除法 = 乘以除数的**模逆元**（仅当 MOD 是质数时，可用费马小定理快速求）|

#### 3. 避坑铁律：
- 所有中间结果都要取模，尤其是循环里的累加 / 累乘，绝对不能等最后再取模，会直接溢出！
---
### 1. 快速幂（快速幂取模，最高频考点）
**用途**：O(log b) 时间计算 abmodp ，解决 b 极大（比如 1e9）的幂次计算，CSP 必考，适配大数幂、逆元、密码类题目
```c
#include <stdio.h>
typedef long long ll;  // 所有数值用long long，避免int溢出
// 核心快速幂：计算 (base^exponent) mod modu，exponent必须非负
ll quick_pow(ll base, ll exponent, ll modu) {
    ll result = 1;
    base = (base % modu + modu) % modu;  // 先处理底数，避免负数
    while (exponent > 0) {
        if (exponent % 2 == 1) {  // 指数为奇数，把当前底数乘到结果里
            result = (result * base) % modu;
        }
        base = (base * base) % modu;  // 底数平方
        exponent = exponent / 2;  // 指数折半
    }
    return result;
}
// 补充：快速乘（防爆乘法），计算(a*b) mod modu，避免两个1e9相乘爆long long
ll quick_mul(ll a, ll b, ll modu) {
    ll result = 0;
    a = a % modu;
    while (b > 0) {
        if (b % 2 == 1) {
            result = (result + a) % modu;
        }
        a = (a * 2) % modu;
        b = b / 2;
    }
    return result;
}
```
**考试提醒**：变量必须用 `long long`！`int` 在 MOD=1e9+7 时绝对会溢出，别省这个功夫。

---
### 2. 最大公约数 GCD / 最小公倍数 LCM（超高频基础）
**用途**：分数化简、周期问题、同余方程、线性筛，几乎所有数论题都可能用到，代码极简
```c
typedef long long ll;
// 欧几里得算法（辗转相除法）求GCD，O(log min(a,b))
ll gcd(ll a, ll b) {
    return b == 0 ? a : gcd(b, a % b);
}
// 求最小公倍数LCM，【必看】先除后乘，绝对避免溢出！
ll lcm(ll a, ll b) {
    if (a == 0 || b == 0) return 0;  // 边界处理
    return a / gcd(a, b) * b;  // 先除以gcd再乘，禁止先a*b！
}
```
**考试提醒**：
- `gcd(a,0)=a`
- `gcd(0,0)` 无意义，代码里要提前处理边界
- **LCM 先除后乘是铁律**，否则两个 1e9 的数相乘直接爆 `long long`
---
### 3. 素数相关算法（CSP 高频考点）
#### 3.1 单个素数判断（试除法）
**用途**：判断单个数字是否是素数，O(√n)，适合 n≤1e12 的单个判断
```c
#include <stdbool.h>
typedef long long ll;
bool is_prime(ll n) {
    if (n <= 1) return false;  // 1及以下不是素数
    if (n == 2) return true;   // 2是唯一的偶素数
    if (n % 2 == 0) return false;  // 偶数直接排除，提速一倍
    for (ll i = 3; i * i <= n; i += 2) {  // 只遍历奇数，到√n即可
        if (n % i == 0) return false;
    }
    return true;
}
```
#### 3.2 埃氏筛法（考试首选！）
**用途**：O(n log log n) 时间求 n 以内所有素数，代码极简、好写不出错，适合 n≤1e6（CSP99% 的场景够用）
```c
#include <stdbool.h>
#include <string.h>
#define MAXN 1000005  // 按题目调整，1e6足够应付绝大多数题
bool is_prime[MAXN];  // 标记是否是素数
int prime[MAXN];      // 可选：存储筛出来的所有素数
int prime_cnt;        // 可选：素数的个数
// 埃氏筛初始化
void sieve_eratosthenes(int n) {
    memset(is_prime, true, sizeof(is_prime));
    is_prime[0] = is_prime[1] = false;  // 0和1不是素数
    for (int i = 2; i * i <= n; i++) {
        if (is_prime[i]) {  // 若i是素数，标记它的所有倍数
            for (int j = i * i; j <= n; j += i) {
                is_prime[j] = false;
            }
        }
    }
    // 可选：把所有素数存入prime数组
    prime_cnt = 0;
    for (int i = 2; i <= n; i++) {
        if (is_prime[i]) {
            prime[prime_cnt++] = i;
        }
    }
}
```
#### 3.3 线性筛（欧拉筛，选考）
**用途**：O(n) 线性时间筛素数，无重复标记，适合 n≤1e7 的大数据，同时可拓展求欧拉函数等
```c
#include <stdbool.h>
#include <string.h>
#define MAXN 10000005  // 1e7以内可用
bool is_prime[MAXN];
int prime[MAXN];
int prime_cnt;
void sieve_euler(int n) {
    memset(is_prime, true, sizeof(is_prime));
    is_prime[0] = is_prime[1] = false;
    prime_cnt = 0;
    for (int i = 2; i <= n; i++) {
        if (is_prime[i]) {
            prime[prime_cnt++] = i;
        }
        // 核心：保证每个数只被它的最小质因子筛一次
        for (int j = 0; j < prime_cnt && i * prime[j] <= n; j++) {
            is_prime[i * prime[j]] = false;
            if (i % prime[j] == 0) break;
        }
    }
}
```
**考试提醒**：优先用埃氏筛！代码短、好记、不容易写错，CSP 的 n 基本不会超过 1e6，埃氏筛完全够用。

---
### 4. 质因数分解（试除法，高频考点）
**用途**：分解 n 的所有质因子，拓展求约数个数、约数和、欧拉函数，CSP 常考
```c
typedef long long ll;
#define MAX_FACTOR 100  // 足够用，1e12的数质因子不超过20个
// 分解n的质因子，存入factor数组，cnt数组存对应指数，返回质因子个数
int prime_factorize(ll n, ll factor[], int cnt[]) {
    int fact_cnt = 0;
    if (n <= 1) return 0;
    // 先处理2，单独提出来提速
    if (n % 2 == 0) {
        factor[fact_cnt] = 2;
        cnt[fact_cnt] = 0;
        while (n % 2 == 0) {
            cnt[fact_cnt]++;
            n /= 2;
        }
        fact_cnt++;
    }
    // 处理奇数因子
    for (ll i = 3; i * i <= n; i += 2) {
        if (n % i == 0) {
            factor[fact_cnt] = i;
            cnt[fact_cnt] = 0;
            while (n % i == 0) {
                cnt[fact_cnt]++;
                n /= i;
            }
            fact_cnt++;
        }
    }
    // 【必加】最后剩余的n>1，说明是一个质因子
    if (n > 1) {
        factor[fact_cnt] = n;
        cnt[fact_cnt] = 1;
        fact_cnt++;
    }
    return fact_cnt;
}
// 补充：求n的正约数个数（基于质因数分解）
ll get_divisor_count(ll n) {
    ll factor[MAX_FACTOR];
    int cnt[MAX_FACTOR];
    int fact_cnt = prime_factorize(n, factor, cnt);
    ll res = 1;
    for (int i = 0; i < fact_cnt; i++) {
        res *= (cnt[i] + 1);
    }
    return res;
}
```
---
### 5. 组合数计算（模意义下，CSP 常考路径计数、排列组合）
分两个版本，按题目数据范围选，都是适配 CSP 常用质数模数
```c
#include <stdio.h>
typedef long long ll;
#define MOD 1000000007  // 可替换为题目要求的质数模数
// ====================== 版本1：杨辉三角递推（考试首选！）======================
// 适合n≤1000的小范围，代码极简，不用考虑逆元，绝对不会写错
#define MAXN 1005
ll C[MAXN][MAXN];  // C[n][k] 就是组合数C(n,k)
void comb_init_yanghui(int max_n) {
    for (int i = 0; i <= max_n; i++) {
        C[i][0] = C[i][i] = 1;  // 边界：C(n,0)=C(n,n)=1
        for (int j = 1; j < i; j++) {
            C[i][j] = (C[i-1][j-1] + C[i-1][j]) % MOD;
        }
    }
}
// 调用示例：comb_init_yanghui(1000); 之后直接用 C[5][2] 得到10
// ====================== 版本2：阶乘+逆元版（大范围）======================
// 适合n≤1e6的场景，O(n)预处理，O(1)查询，仅当MOD是质数时可用
#define MAX_FACT 1000005
ll fact[MAX_FACT];     // fact[i] = i! mod MOD
ll inv_fact[MAX_FACT]; // inv_fact[i] = (i!)的模逆元 mod MOD
// 复用前面的快速幂函数
ll quick_pow(ll base, ll exponent, ll modu) {
    ll result = 1;
    base = (base % modu + modu) % modu;
    while (exponent > 0) {
        if (exponent % 2 == 1) {
            result = (result * base) % modu;
        }
        base = (base * base) % modu;
        exponent /= 2;
    }
    return result;
}
// 预处理阶乘和阶乘逆元
void comb_init_fact(int max_n) {
    fact[0] = 1;
    for (int i = 1; i <= max_n; i++) {
        fact[i] = fact[i-1] * i % MOD;
    }
    // 费马小定理：逆元 = a^(MOD-2) mod MOD（MOD是质数）
    inv_fact[max_n] = quick_pow(fact[max_n], MOD-2, MOD);
    for (int i = max_n-1; i >= 0; i--) {
        inv_fact[i] = inv_fact[i+1] * (i+1) % MOD;
    }
}
// 计算C(n,k) = n!/(k!*(n-k)!) mod MOD
ll comb(ll n, ll k) {
    if (k < 0 || k > n) return 0;  // 非法情况直接返回0
    return fact[n] * inv_fact[k] % MOD * inv_fact[n - k] % MOD;
}
```

**考试提醒**：n≤1000 直接用杨辉三角，不用想逆元，零出错，考试首选。

---
### 6. 扩展欧几里得算法（选考，基础版）
**用途**：解不定方程 ax+by=gcd(a,b)、求非质数模数的逆元、解线性同余方程，CSP 前 3 题基本不考，有余力再记
```c
typedef long long ll;
// 扩展欧几里得：求解ax + by = gcd(a,b)，返回gcd(a,b)，x和y为输出的解
ll exgcd(ll a, ll b, ll *x, ll *y) {
    if (b == 0) {
        *x = 1;
        *y = 0;
        return a;
    }
    ll g = exgcd(b, a % b, y, x);
    *y -= a / b * *x;
    return g;
}
// 补充：求a在mod m下的逆元，返回-1表示逆元不存在（a和m不互质）
ll mod_inverse(ll a, ll m) {
    ll x, y;
    ll g = exgcd(a, m, &x, &y);
    if (g != 1) return -1;
    return (x % m + m) % m; // 处理负数，返回正的逆元
}
```
---
## 五、ASCII 核心操作 & 进制转换快速板子
### 一、ASCII 核心操作（CSP 字符串题必用，一行搞定）
先记死 3 个核心基准值 ，考试不用查表，所有操作都基于这个，绝对不会写错：

|字符|十进制 ASCII 值|核心用途|
|---|---|---|
|`'0'`|48|数字字符↔数值转换|
|`'A'`|65|大写字母基准|
|`'a'`|97|小写字母基准|
|补充|`'a'-'A'=32`|大小写差值固定为32|

#### 常用操作代码（考试直接抄）
```c
#include <stdio.h>
#include <ctype.h> // 标准库字符判断函数，可选，更快捷
int main() {
    char c;
    int num;
    // ====================== 1. 数字字符 ↔ 整数数值（最高频）======================
    // 1.1 单个数字字符 → 整数（例：'5'→5）
    c = '5';
    num = c - '0'; // 核心公式！num=5，零出错
    // 1.2 整数 → 单个数字字符（例：3→'3'，仅限0-9）
    num = 3;
    c = num + '0'; // c='3'
    // ====================== 2. 大小写字母转换 ======================
    // 2.1 安全版：小写→大写（先判断，避免非字母出错，考试推荐）
    c = 'a';
    if (c >= 'a' && c <= 'z') c = c - 32; // 或 c -= ('a'-'A')，更直观
    // 2.2 安全版：大写→小写
    c = 'B';
    if (c >= 'A' && c <= 'Z') c = c + 32;
    // ====================== 3. 标准库快速判断函数（不用手写，直接用）======================
    c = '5';
    isdigit(c); // 判断是否是数字字符，是返回非0，否返回0
    isalpha(c); // 判断是否是字母（大小写都算）
    isupper(c); // 判断是否是大写字母
    islower(c); // 判断是否是小写字母
    isalnum(c); // 判断是否是字母/数字
    isspace(c); // 判断是否是空白字符（空格、换行、制表符等）
    return 0;
}
```

**考试避坑铁律**：
1. 绝对不要直接写 48/65/97，用 `'0'/'A'/'a'` 代替，代码直观且零出错
2. 大小写转换前优先判断字符类型，避免非字母字符转换出乱码
3. 多位数的字符串转整数，需要循环累加：`num = num * 10 + (c - '0')`
---
### 二、进制转换快速实现（CSP 必考，优先用标准库方案）
#### 方案 1：C 标准库函数（最快！一行搞定，考试首选，零手写错误）
适用场景：`long long` 整数范围内的进制转换，支持 2-36 进制，CSP 99% 的进制题都能用，GCC 评测环境完全支持。
##### 核心用 2 个标准函数：
- `sprintf`：10 进制整数 → 指定进制字符串
- `strtoll`：任意进制字符串 → 10 进制 `long long` 整数
```c
#include <stdio.h>
#include <stdlib.h>
#include <string.h>
int main() {
    char str[100]; // 存储转换后的字符串，开100足够覆盖long long的2进制长度
    long long num = 123;
    // ====================== 1. 10进制整数 → 其他进制字符串 ======================
    sprintf(str, "%b", num); // 10→2进制，输出：1111011（GCC扩展，CSP完全支持）
    sprintf(str, "%o", num); // 10→8进制，输出：173（标准C）
    sprintf(str, "%x", num); // 10→16进制小写，输出：7b
    sprintf(str, "%X", num); // 10→16进制大写，输出：7B
    sprintf(str, "%lld", num); // 10→10进制字符串，long long用%lld

    // ====================== 2. 任意进制字符串 → 10进制整数 ======================
    // 函数原型：strtoll(待转换字符串, NULL, 原字符串的进制)，支持2-36进制
    char bin_str[] = "1111011";
    char hex_str[] = "7B";
    char base36_str[] = "Z";
    long long res1 = strtoll(bin_str, NULL, 2);    // 2→10进制，结果123
    long long res2 = strtoll(hex_str, NULL, 16);   // 16→10进制，结果123
    long long res3 = strtoll(base36_str, NULL, 36); // 36→10进制，结果35
    return 0;
}
```
**考试避坑点**：
1. 大数必须用 `long long`，对应 `sprintf` 用 `%lld`，转换函数用 `strtoll`（不是 `strtol`，范围更大）
2. 转换后的字符串数组必须开足够大，`long long` 的 2 进制最多 64 位，开 100 绝对够用
3. `strtoll` 自动兼容字母大小写，不用手动处理大小写转换
#### 方案 2：手写万能进制转换板子（应对标准库搞不定的场景）
适用场景：超过 `long long` 范围的小数据、自定义进制逻辑、需要完整控制转换过程的题目，核心逻辑是 **除基取余、逆序排列**。
```c
#include <stdio.h>
#include <string.h>
#include <ctype.h>
#define MAX_LEN 1000
// 进制字符表，0-9对应0-9，10-35对应A-Z，支持2-36进制
const char BASE_CHAR[] = "0123456789ABCDEFGHIJKLMNOPQRSTUVWXYZ";
// ====================== 1. 任意进制字符串 → 10进制long long整数 ======================
long long any_to_dec(char *str, int base) {
    long long res = 0;
    int len = strlen(str);
    for (int i = 0; i < len; i++) {
        char c = toupper(str[i]); // 统一转大写，兼容小写
        int num = isdigit(c) ? (c - '0') : (c - 'A' + 10);
        res = res * base + num; // 核心递推公式
    }
    return res;
}
// ====================== 2. 10进制long long整数 → 任意进制字符串 ======================
void dec_to_any(long long num, int base, char *res) {
    if (num == 0) { // 必须特殊处理0，否则会输出空串
        strcpy(res, "0");
        return;
    }
    int idx = 0, is_negative = 0;
    char temp[MAX_LEN];
    // 处理负数
    if (num < 0) {
        is_negative = 1;
        num = -num;
    }
    // 核心：除基取余
    while (num > 0) {
        int remainder = num % base;
        temp[idx++] = BASE_CHAR[remainder];
        num = num / base;
    }
    if (is_negative) temp[idx++] = '-';
    // 逆序输出，补字符串结束符
    int res_idx = 0;
    for (int i = idx - 1; i >= 0; i--) {
        res[res_idx++] = temp[i];
    }
    res[res_idx] = '\0'; // 必须加，否则会出现乱码
}
// ====================== 3. 万能互转：任意进制A → 任意进制B ======================
void any_to_any(char *src, int src_base, char *dest, int dest_base) {
    long long dec = any_to_dec(src, src_base);
    dec_to_any(dec, dest_base, dest);
}
// 测试示例
int main() {
    char src[] = "1111011";
    char dest[100];
    any_to_any(src, 2, dest, 16); // 2进制转16进制，输出7B
    dec_to_any(123, 36, dest);    // 10进制转36进制，输出3F
    return 0;
}
```

**考试终极提醒**：明天遇到进制转换题，**优先用标准库的 `sprintf/strtoll`**，10 秒写完，零出错，除非题目明确考大数进制转换，再用手写板子。ASCII 的 `c-'0'` 是字符串题的核心公式，一定要记死！

---
## 六、前 3 题必拿分核心工具（最高优先级）
### 1. 前缀和与差分（CSP 前两题必考，送分题核心）
**用途**：O(1) 处理区间求和、区间修改问题，避免循环累加超时，一维用于数组区间题，二维用于矩阵 / 网格题，几乎每年都考。
```c
#include <stdio.h>
#include <string.h>
#define MAXN 100005
#define MAXM 1005
// ====================== 一维前缀和 ======================
int pre_sum[MAXN];  // 前缀和数组
int a[MAXN];        // 原数组
// 初始化一维前缀和
void init_pre_sum(int n) {
    pre_sum[0] = 0;
    for (int i = 1; i <= n; i++) {
        pre_sum[i] = pre_sum[i-1] + a[i];  // 原数组a从1开始，避免边界判断
    }
}
// 查询区间[l, r]的和（闭区间，1起始）
int query_sum(int l, int r) {
    return pre_sum[r] - pre_sum[l-1];
}
// ====================== 一维差分 ======================
int diff[MAXN];  // 差分数组
// 区间[l, r]加val（1起始）
void add_diff(int l, int r, int val) {
    diff[l] += val;
    diff[r+1] -= val;
}
// 差分数组还原为原数组
void restore_diff(int n) {
    a[0] = 0;
    for (int i = 1; i <= n; i++) {
        a[i] = a[i-1] + diff[i];
    }
}
// ====================== 二维前缀和（矩阵题用）======================
int pre2[MAXM][MAXM];
int a2[MAXM][MAXM];
// 初始化二维前缀和
void init_pre2(int n, int m) {
    memset(pre2, 0, sizeof(pre2));
    for (int i = 1; i <= n; i++) {
        for (int j = 1; j <= m; j++) {
            pre2[i][j] = pre2[i-1][j] + pre2[i][j-1] - pre2[i-1][j-1] + a2[i][j];
        }
    }
}
// 查询子矩阵(x1,y1)到(x2,y2)的和（左上角到右下角，1起始）
int query_sum2(int x1, int y1, int x2, int y2) {
    return pre2[x2][y2] - pre2[x1-1][y2] - pre2[x2][y1-1] + pre2[x1-1][y1-1];
}
```
**考试提醒**：原数组统一从 1 开始，不用处理 0 的边界问题，零出错。

---
### 2. 二分查找（CSP 第 2-4 题高频考点）
**用途**：O(log n) 查找有序数组元素，更常用的是**二分答案**（解决最大化最小值、最小化最大值类问题，CSP 超高频）
```c
#include <stdio.h>
#include <stdlib.h>

// ====================== 1. 标准库bsearch（有序数组查找，快速搞定）======================
// 比较函数，和qsort规则一致
int cmp_bsearch(const void *a, const void *b) {
    return *(int*)a - *(int*)b;
}
// 调用示例：
// int arr[] = {1,3,5,7,9}, key = 5;
// int *p = bsearch(&key, arr, 5, sizeof(int), cmp_bsearch);
// 找到返回对应元素指针，未找到返回NULL
// ====================== 2. 手写二分查找（有序数组升序，找目标值）======================
// 找到返回索引，未找到返回-1
int binary_search(int arr[], int n, int target) {
    int left = 0, right = n - 1;
    while (left <= right) {
        int mid = left + (right - left) / 2;  // 避免left+right溢出，别写(left+right)/2
        if (arr[mid] == target) {
            return mid;
        } else if (arr[mid] < target) {
            left = mid + 1;
        } else {
            right = mid - 1;
        }
    }
    return -1;
}
// ====================== 3. 二分答案通用模板（核心！CSP最常考）======================
// check函数：判断mid是否满足题目条件，自己根据题目写
bool check(int mid) {
    // 这里写题目逻辑，满足返回true，不满足返回false
    return true;
}
// 模板1：找满足条件的最小值
int find_min() {
    int left = 最小值下界, right = 最大值上界;
    int ans = right;
    while (left <= right) {
        int mid = left + (right - left) / 2;
        if (check(mid)) {
            ans = mid;
            right = mid - 1;  // 找最小值，满足就往左缩
        } else {
            left = mid + 1;
        }
    }
    return ans;
}
// 模板2：找满足条件的最大值
int find_max() {
    int left = 最小值下界, right = 最大值上界;
    int ans = left;
    while (left <= right) {
        int mid = left + (right - left) / 2;
        if (check(mid)) {
            ans = mid;
            left = mid + 1;  // 找最大值，满足就往右缩
        } else {
            right = mid - 1;
        }
    }
    return ans;
}
```
**考试避坑点**：`mid` 必须写 `left + (right - left)/2`，禁止 `(left+right)/2`，避免大数溢出。

---
### 3. 并查集完整板子（带路径压缩 + 按秩合并）
**用途**：解决集合合并、连通性判断问题（朋友圈、岛屿数量、连通块），不止用于克鲁斯卡尔，CSP 第 2-3 题常考
```c
#include <stdio.h>
#define MAXN 100005
int parent[MAXN];  // 父节点数组
int rank_[MAXN];   // 秩数组（按树的深度合并，避免树退化成链）
// 初始化并查集
void dsu_init(int n) {
    for (int i = 1; i <= n; i++) {
        parent[i] = i;
        rank_[i] = 1;  // 初始每个集合深度为1
    }
}
// 查找根节点（带路径压缩，核心）
int find(int x) {
    if (parent[x] != x) {
        parent[x] = find(parent[x]);  // 路径压缩，直接指向根节点
    }
    return parent[x];
}
// 合并两个集合（按秩合并，更高效），返回1=合并成功，0=已在同一集合
int unite(int x, int y) {
    x = find(x);
    y = find(y);
    if (x == y) return 0;
    // 小树合并到大树上，保持树的深度最小
    if (rank_[x] < rank_[y]) {
        parent[x] = y;
    } else {
        parent[y] = x;
        if (rank_[x] == rank_[y]) {
            rank_[x]++;
        }
    }
    return 1;
}
// 判断两个节点是否在同一集合
bool is_same(int x, int y) {
    return find(x) == find(y);
}
```

---
### 4. C 语言输入输出避坑大全（纯 C 选手必看，90% 的低级错误都在这）
```c
#include <stdio.h>
#include <string.h>
int main() {
    // ====================== 1. 格式符必记（溢出重灾区）======================
    int a; long long b; double c; char d;
    scanf("%d", &a);    // int用%d
    scanf("%lld", &b);  // long long必须用%lld！别用%d，直接溢出
    scanf("%lf", &c);   // double用%lf，float用%f
    scanf("%c", &d);    // char用%c，【大坑】会读入换行符/空格！
    // ====================== 2. 读入char的换行符残留问题（最高频翻车点）======================
    // 错误示例：先读int，再读char，会读入换行符
    int n; char ch;
    scanf("%d", &n);
    getchar(); // 吃掉换行符！
    scanf("%c", &ch);
    // ====================== 3. 读入带空格的字符串 =======================
    char str[100];
    // 正确1：用fgets读整行（推荐，安全）
    fgets(str, sizeof(str), stdin);
    str[strcspn(str, "\n")] = '\0'; // 去掉换行符
    // 正确2：scanf正则匹配，读入到换行符为止
    scanf("%[^\n]", str);
    getchar(); // 吃掉末尾的换行符
    // ====================== 4. 输出格式符 ======================
    printf("%d", a);    // int输出
    printf("%lld", b);  // long long输出必须用%lld
    printf("%.2f", c);  // double输出用%f，保留2位小数，不用%lf
    printf("%05d", a);  // 固定5位宽度，不足补前导0（比如123→00123，进制题常用）
    // ====================== 5. 多组数据输入处理 =======================
    // 方式1：已知数据组数t
    int t;
    scanf("%d", &t);
    while (t--) {
        // 处理每组数据，【必做】每组数据前重置数组/变量！
    }
    // 方式2：未知组数，读到文件末尾（CSP常用）
    int x;
    while (scanf("%d", &x) != EOF) {
        // 处理数据
    }
    return 0;
}
```
**考试铁律**：
- 只要用 `%c` 读入字符，前面必须检查是否有残留的换行符，用 `getchar()` 吃掉
- `long long` 的输入必须用 `%lld`，输出也必须用 `%lld`
- 多组数据，每组必须重置所有数组、变量、图结构，否则会用上次的数据出错
---
## 七、进阶高频考点
### 1. 栈（数组模拟，括号匹配 / 表达式求值必用）
```c
#include <stdio.h>
#include <stdbool.h>
#include <string.h>
#define MAXN 100005
int stack[MAXN];
int top = -1;
void push(int x) { stack[++top] = x; }
int pop() { return top == -1 ? -1 : stack[top--]; }
int get_top() { return top == -1 ? -1 : stack[top]; }
bool is_empty() { return top == -1; }
// ====================== 经典应用：括号匹配（CSP常考）======================
bool is_valid(char *s) {
    top = -1;
    int n = strlen(s);
    for (int i = 0; i < n; i++) {
        if (s[i] == '(' || s[i] == '[' || s[i] == '{') {
            push(s[i]);
        } else {
            if (is_empty()) return false;
            char top_c = get_top();
            if ((s[i] == ')' && top_c == '(') ||
                (s[i] == ']' && top_c == '[') ||
                (s[i] == '}' && top_c == '{')) {
                pop();
            } else {
                return false;
            }
        }
    }
    return is_empty();
}
```
---
### 2. 高精度算法（大数加减，超过 long long 用）
```c
#include <stdio.h>
#include <string.h>
#define MAX_LEN 1005
// ====================== 高精度加法 ======================
int big_add(char *a, char *b, int res[]) {
    int len_a = strlen(a), len_b = strlen(b);
    int num_a[MAX_LEN] = {0}, num_b[MAX_LEN] = {0};
    int res_len = 0, carry = 0;
    for (int i = 0; i < len_a; i++) num_a[i] = a[len_a - 1 - i] - '0';
    for (int i = 0; i < len_b; i++) num_b[i] = b[len_b - 1 - i] - '0';
    int max_len = len_a > len_b ? len_a : len_b;
    for (int i = 0; i < max_len; i++) {
        int sum = num_a[i] + num_b[i] + carry;
        res[res_len++] = sum % 10;
        carry = sum / 10;
    }
    if (carry > 0) res[res_len++] = carry;
    return res_len;
}
// ====================== 高精度减法（保证a>=b）======================
int big_sub(char *a, char *b, int res[]) {
    int len_a = strlen(a), len_b = strlen(b);
    int num_a[MAX_LEN] = {0}, num_b[MAX_LEN] = {0};
    int res_len = 0, borrow = 0;
    for (int i = 0; i < len_a; i++) num_a[i] = a[len_a - 1 - i] - '0';
    for (int i = 0; i < len_b; i++) num_b[i] = b[len_b - 1 - i] - '0';
    for (int i = 0; i < len_a; i++) {
        int sub = num_a[i] - num_b[i] - borrow;
        if (sub < 0) { sub += 10; borrow = 1; }
        else borrow = 0;
        res[res_len++] = sub;
    }
    while (res_len > 1 && res[res_len - 1] == 0) res_len--;
    return res_len;
}
```
✅ **推荐补充快读模板**（非必需，但对性能敏感题有用）：
```c
int read_int() {
    int x = 0, f = 1;
    char ch = getchar();
    while (ch < '0' || ch > '9') {
        if (ch == '-') f = -1;
        ch = getchar();
    }
    while (ch >= '0' && ch <= '9') {
        x = x * 10 + ch - '0';
        ch = getchar();
    }
    return x * f;
}
```
---
### 3. 贪心算法经典场景模板（CSP 前两题常考）
```c
#include <stdio.h>
#include <stdlib.h>
#define MAXN 100005
typedef struct Interval {
    int start, end;
} Interval;
Interval interval[MAXN];
int cmp_interval(const void *a, const void *b) {
    return ((Interval*)a)->end - ((Interval*)b)->end;
}
int max_non_overlap(int n) {
    qsort(interval, n, sizeof(Interval), cmp_interval);
    int count = 1, last_end = interval[0].end;
    for (int i = 1; i < n; i++) {
        if (interval[i].start >= last_end) {
            count++;
            last_end = interval[i].end;
        }
    }
    return count;
}
```