# Minecraft 混淆表提取数据 (mappings-extracted)

从 [takenaka](https://github.com/zlataovce/takenaka) 混淆表网站
(`D:\mappings-gh-pages\mappings-gh-pages`) 提取的 Minecraft Java 版各版本反混淆数据。
其中 **1.8.9** 不在该网站收录范围内, 由官方 MCP 数据转换而来(见下文说明)。

> **📖 全链路文档**：[`全链路文档.md`](全链路文档.md) —— 本数据如何流转到下游
> （生成器 gen_maps.py → MCCombatStatusJni.dll → 注入/上报 → 连点器消费），
> 含命名体系、真机验证矩阵、关键边界与全部踩坑记录。
>
> **下游生态**：
> - 生成器/DLL 源码：`D:\VibeCoding\MCCombatStatusJni\`（上游项目）
> - 全链路工具包（含文档）：`D:\VibeCoding\MCCombatStatusToolkit\`
> - 消费端（连点器）：`D:\VibeCoding\AutoClicker-main\`

- **版本范围**: 1.8.8 ~ 1.21.11,共 54 个版本
  (53 个来自网站 + 1.8.9 来自官方 MCP 数据)
- **数据来源**:
  - 53 个网站版本: 每个版本目录下的 `class-index.js`(类索引)与每个类一个的 HTML 页面
  - 1.8.9: 官方 MCP 文件(`joined.srg`/`joined.exc`/`fields.csv`/`methods.csv`/`params.csv`)
- **提取工具**:
  - `D:\mappings-gh-pages\extract_mappings.py`(网站版本)
  - `D:\mappings-gh-pages\convert_mcp_189.py`(1.8.9 MCP 转换)
- **提取时间**: 2026-08-12 ~ 2026-08-13
- **数据规模**: 约 29.9 万类 / 127 万字段 / 202 万方法 / 1081 万参数(合计约 1.7GB)

---

## 目录结构

```
mappings-extracted\
  README.md              ← 本文件
  _summary.json          ← 全部版本的统计汇总
  <版本>\
    mapping.json         ← 版本元数据(命名空间、颜色、统计)
    classes.csv          ← 类名映射(全部命名空间)
    fields.csv           ← 字段映射(含枚举常量)
    methods.csv          ← 方法/构造函数映射(含签名与 JVM 描述符)
    params.csv           ← 参数名(长格式)
    joined.srg           ← obf → Searge 的 CL/FD/MD 行(与 MCP 同格式)
    joined.exc           ← Searge 方法 → 真实参数名(尽力而为)
```

---

## 命名空间

每个版本可能包含以下 6 个命名空间(旧版本只有部分):

| 命名空间 | 含义 | 颜色(网站) | 覆盖版本 |
|---|---|---|---|
| `混淆的` | 类文件中的真实混淆名(如 `a`, `bH`, `bsr`) | `#581C87` | 全部 |
| `Searge` | MCP/Searge 映射(如 `field_96303_A`, `func_175744_a`) | `#B91C1C` | 全部 |
| `Spigot` | Spigot/CraftBukkit 映射(如 `getTrader`, `EnumChatFormat`) | `#CA8A04` | 全部 |
| `Mojang` | 官方 Mojang 映射(1.14.4+,如 `ChatFormatting`) | `#4D7C0F` | 1.14.4+ |
| `Yarn` | Fabric Yarn 映射(如 `class_124`, `field_1074`) | `#626262` | 1.14.4+ |
| `Intermediary` | Fabric Intermediary 映射(如 `C_4856_`, `m_179367_`) | `#0369A1` | 1.14.4+ |

> joined.srg 的左边(混淆名)对应 `混淆的` 列,右边(Searge 全限定名)对应
> `Searge` 列 —— 与官方 MCP joined 映射的方向一致,已与参考 1.8.9 的
> joined.srg 逐行核对,CL 行 100% 一致。

命名空间↔CSV 列的对应关系(**固定顺序**):

```
混淆的, Searge, Spigot, Mojang, Yarn, Intermediary
```

---

## 1.8.9 特殊说明

takenaka 网站不收录 1.8.9,因此该版本由**官方 MCP 数据**
(`C:\Users\11407\Desktop\1.8.9\`)转换而来,格式与网站版本保持一致,但有几点差异:

### 命名空间

| 命名空间 | 1.8.9 中的内容 |
|---|---|
| `混淆的` | SRG 左侧混淆名(如 `a`, `avh`) |
| `Searge` | MCP 全限定名 / `field_*` / `func_*` |
| `Spigot` | **实际存放 MCP 可读名**(1.8.9 无 CraftBukkit 数据) |
| `Mojang` / `Yarn` / `Intermediary` | 空 |
| `MCP` | 仅 params.csv 使用: 参数可读名来源命名空间 |

### 文件差异

- **fields.csv / methods.csv 末尾多两列**: `side`(0=客户端, 1=服务端, 2=双端)与
  `desc`(MCP 文档描述)。这是官方 MCP 独有的数据,网站版本没有。
- **methods.csv 的 `descriptor` 列**: 由 MCP 描述符转回混淆命名空间类型名
  (如 `func_175744_a` → `(I)La;`),与 joined.srg 的 obf 描述符一致。
- **构造函数**: `joined.srg` 不含构造函数,构造函数来自 `joined.exc` 的 `<init>`
  条目(共 1946 个),`signature` 为 `(p_i45394_1_, ...)` 形式。
- **params.csv**: 每个参数最多 3 行 —— `混淆的`(argN 占位)、`Searge`(p_xxx)、
  `MCP`(可读名,如 `key`/`brokenEntity`)。参数名来源优先级:
  1. `joined.exc` 精确匹配(键为 MCP 类/方法/描述符)
  2. `params.csv` 前缀启发式(`p_<func索引>_<i>_`,覆盖约 84% 的方法)
  3. `argN` 占位符(仅当 MCP 未命名该参数)
- **joined.srg / joined.exc**: 官方 MCP 文件**原样拷贝**(含 7 条 `PK:` 行),
  与参考文件逐字节一致。
- **mapping.json**: `stats` 含 `constructors` / `fields_with_name` /
  `methods_with_name` 附加字段,`namespaces` 标注了 `MCP` 来源。

### 与 1.8.8 的对比示例

```bash
# 1.8.9 (MCP): func_100015_a 有参数名
grep 'func_100015_a' mappings-extracted/1.8.9/methods.csv
# a,a(avb arg0),func_100015_a(KeyBinding p_100015_0_),isKeyDown(KeyBinding key),...

# 1.8.8 (网站): 同方法只有 argN 占位
grep 'func_100015_a' mappings-extracted/1.8.8/methods.csv
# avh,a(arg0),func_100015_a(arg0),,,,...
```

---

## 文件格式

### classes.csv

类名映射,每行一个类,列为 6 个命名空间(完整类路径,`/` 分隔,无 `.class` 后缀):

```
混淆的,Searge,Spigot,Mojang,Yarn,Intermediary
a,net/minecraft/src/C_4856_,net/minecraft/EnumChatFormat,net/minecraft/ChatFormatting,net/minecraft/util/Formatting,net/minecraft/class_124
```

- 内嵌类用 `$` 表示,如 `aap$b`
- 该版本不存在的命名空间留空

### fields.csv

字段与枚举常量映射:

```
owner_obf,混淆的,Searge,Spigot,Mojang,Yarn,Intermediary,kind,type
```

- `owner_obf`: 所属类的混淆名(与 classes.csv 的 `混淆的` 列对应)
- `kind`: `field`(字段)或 `enum`(枚举常量)
- `type`: 修饰符与类型显示文本(如 `private final EntityType<?>`),枚举常量通常为空

### methods.csv

方法/构造函数映射:

```
owner_obf,混淆的,Searge,Spigot,Mojang,Yarn,Intermediary,kind,signature,descriptor,return_type
```

- `kind`: `method` 或 `constructor`
- `signature`: 混淆命名空间下的显示签名(如 `a(int arg0, World arg1)`),构造函数为 `<init>`
- `descriptor`: **重建的 JVM 描述符**(混淆命名空间类型名),如 `(ILadm;)V`
  - 泛型参数按擦除处理(如 `<T>` → `Object`)
  - `...`(varargs)与 `[]` 按数组处理
  - 无法解析的类型(极少,如网站渲染缺失)该列为空
- `return_type`: 返回类型显示文本(构造函数为 `public` 等修饰符文本)

### params.csv

参数名(长格式,便于程序查询):

```
owner_obf,member_obf,signature,kind,param_index,namespace,name
```

- `member_obf`: 成员混淆名(方法名或 `<init>`)
- `signature`: 与方法签名区分同一方法的重载
- `param_index`: 从 0 开始的参数序号
- `namespace`: 该参数名所属命名空间
- `name`: 参数名(占位符为 `arg0`/`p_xxx` 等)

> 旧网站版本(≤1.13)几乎只有 `argN` 占位符;真实参数名主要来自
> Yarn(1.14.4+)与 Mojang(1.21+ 的部分方法)。**1.8.9** 额外提供
> `MCP` 命名空间行(官方 p_xxx 可读参数名,见上文特殊说明)。

### joined.srg

兼容官方 MCP joined 映射格式,三部分按字典序排列:

```
CL: <混淆类名> <Searge类名>
FD: <混淆类名>/<混淆字段> <Searge类名>/<Searge字段>
MD: <混淆类名>/<混淆方法> <混淆描述符> <Searge类名>/<Searge方法> <Searge描述符>
```

- 两侧描述符分别使用混淆命名空间与 Searge 命名空间的类型名
- 与参考的 1.8.9 joined.srg 对照: CL 行完全一致,FD/MD 行除两版本间真实差异外一致
- 网站版本不含 `PK:` 行(混淆名全部为扁平名,无包结构);
  **1.8.9 例外**: 该版本为官方 MCP 文件原样拷贝,含 7 条 `PK:` 行

### joined.exc

兼容 MCP 参数名文件(尽力而为):

```
# extracted from takenaka mappings (mappings-gh-pages)
# version: 1.8.8
net/minecraft/.../Class/member(Ldesc;)V=|name1,name2
```

- 方法名使用 Searge 名,描述符使用 Searge 命名空间类型名
- 只有存在真实参数名(非 `argN`)的方法/构造函数才会写入
- 旧版本(≤1.13)几乎为空,因为网站没有这些数据

### mapping.json

版本元数据:

```json
{
  "type": "unified",
  "version": "1.8.8",
  "namespaces": [
    {"name": "混淆的", "color": "#581C87"},
    ...
  ],
  "files": {
    "classes": "classes.csv", "fields": "fields.csv", "methods": "methods.csv",
    "params": "params.csv", "srg": "joined.srg", "exc": "joined.exc"
  },
  "stats": {
    "classes": 2451, "fields": 8255, "methods": 17991, "params": 48450,
    "srg_lines": 25937, "exc_lines": 0, "desc_fail": 0, "page_errors": 0
  }
}
```

---

## 查询示例

### 1. 查一个混淆方法对应什么名字(1.8.8)

```bash
grep '^MD: a/b ' mappings-extracted/1.8.8/joined.srg
# MD: a/b (Ljava/lang/String;)La; net/minecraft/util/EnumChatFormatting/func_96300_b (Ljava/lang/String;)Lnet/minecraft/util/EnumChatFormatting;
```

### 2. 查某个类的全部命名空间

```bash
grep ',net/minecraft/world/entity/Entity' mappings-extracted/1.21.11/classes.csv
# cgk,net/minecraft/src/C_507_,net/minecraft/world/entity/Entity,net/minecraft/world/entity/Entity,net/minecraft/entity/Entity,net/minecraft/class_1297
```

### 3. 用 CSV 查询(推荐,python)

```python
import csv
rows = list(csv.reader(open('mappings-extracted/1.21.11/methods.csv', encoding='utf-8-sig')))
# 找 Entity.setPose 的 JVM 描述符
for r in rows:
    if r[0] == 'cgk' and 'setPose' in r[4]:   # r[4] 是 Mojang 列
        print(r[9])  # (Lchx;)V
```

---

## 已知限制

1. **网站版本(除 1.8.9)无 MCP 风格参数名**: 网站本身不包含 MCP 风格的
   `p_XXXX_N_` 参数名,也没有 `side`/`desc` 列数据。旧网站版本(≤1.13)的参数
   全是 `argN` 占位符。**1.8.9 例外**: 该版本由官方 MCP 数据转换,含完整的
   p_xxx 参数名与可读名(见上文"1.8.9 特殊说明")。
2. **`access$` 合成方法缺失**: 1.8.9 参考 joined.srg 中有约 280 条 `access$N`
   合成方法(如 `PlayerProfileCache$ProfileEntry/access$200`),网站不收录这些,
   故提取结果中没有。这是源数据差异,不是解析错误。
3. **描述符重建失败率**: 现代版本约 0.02%(如 1.21.11 的 63305 个方法中 2 个),
   均为网站渲染缺陷(返回类型单元格为空)导致的桥接方法。失败方法的
   `descriptor` 列为空,SRG 中对应 MD 行缺失,`mapping.json` 的 `desc_fail`
   字段记录了数量。
4. **不同版本间差异**: 1.8.8 与 1.8.9 的 CL 行 100% 一致,FD/MD 行除上述
   已知差异外一致;其它版本之间同样只反映真实差异。

---

## 重新生成

```bash
python D:\mappings-gh-pages\extract_mappings.py            # 53 个网站版本
python D:\mappings-gh-pages\extract_mappings.py 1.8.8 1.21 # 只提取部分(前缀匹配)
python D:\mappings-gh-pages\convert_mcp_189.py             # 1.8.9(MCP 转换, 独立)
```
