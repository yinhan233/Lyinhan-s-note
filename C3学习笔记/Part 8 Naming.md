C3 直接把一部分命名规则写进了编译器里面，具体如下：

## 强制要求的规则
| 类别 | 强制命名规则 | 正确示范 | 错误示范及原因 |
| --- | --- | --- | --- |
| 类型 (Struct/Enum/Union) | 必须以大写字母开头，且必须包含至少一个小写字母 | `Player`, `Vec2`, `Http_req` | `HTTP` (全大写报错), `player` (小写开头报错) |
| 常量 / 枚举值 | 必须以大写字母开头 | `MAX_HP`, `VALUE_1` | `max_hp` (小写开头报错) |
| 变量 / 函数 / 宏 | 必须以小写字母开头 | `health`, `get_hp()`, `fooBar` | `Health`, `Get_hp()` (大写开头报错) |
| 模块名 (Module) | 只能是全小写字母、数字和下划线 | `math_utils`, `net` | `Math_Utils` (包含大写报错) |

## 官方推荐的代码风格
虽然上面说到编译器允许变量写成驼峰式（比如 `myHealth`），但 C3 官方标准库有一套推荐的格式：
- **类型 (Types)**：使用 `PascalCase`（大驼峰，如 `PlayerState`）。
- **常量 (Constants/Enums)**：使用 `SCREAMING_SNAKE_CASE`（全大写加下划线，如 `MAX_PLAYERS`）。
- **其他所有东西**（变量、函数、参数、结构体字段、宏）：一律使用 `snake_case`（全小写加下划线，如 `current_health`, `calculate_damage()`）。
- C3 标准库选择了 Allman 风格（大括号换行）：
