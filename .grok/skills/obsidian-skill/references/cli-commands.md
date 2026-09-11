# Obsidian CLI 命令参考

当 Obsidian 正在运行时，可通过命令行直接操作库（vault），无需打开 GUI。
**优先使用 CLI，而不是直接读写 `.md` 文件**——CLI 会同步更新链接、索引与缓存。

## 基本约定

```bash
obsidian <command> [options]
```

| 项 | 说明 |
|---|---|
| `vault=<name>` | 全局选项，指定目标库；省略则用当前活动库 |
| `file=<name>` | **按名称解析**（类似 wiki 链接，自动匹配、可省略 `.md`） |
| `path=<path>` | **精确路径**（如 `01-总览/主页.md`），推荐使用，避免歧义 |
| 默认目标 | 多数命令省略 `file`/`path` 时作用于**当前活动文件** |
| 引号 | 含空格的值需加引号：`name="My Note"` |
| 转义 | 内容中的换行用 `\n`，制表符用 `\t` |
| 输出格式 | 多数命令支持 `format=json\|tsv\|csv\|yaml` |

> ⚠️ `file=` 与 `path=` 的区别是高频踩坑点：文件名重复时 `file=` 可能命中错误笔记，**脚本中一律用 `path=`**。

## 一、读取与检索

```bash
obsidian read path="01-总览/主页.md"            # 读文件内容
obsidian file path="01-总览/主页.md"            # 文件信息（大小/修改时间/wikilink 形式）
obsidian files                                  # 列出全库文件
obsidian files folder="03-数据结构"              # 按文件夹筛选
obsidian files ext=md                           # 按扩展名筛选
obsidian files total                            # 仅返回文件总数
obsidian folders                                # 列出所有文件夹
obsidian folders total                          # 仅返回文件夹总数
obsidian folder path="03-数据结构" info=files    # 文件夹信息（files|folders|size）
obsidian search query="Cache" limit=10          # 全文搜索（返回文件名列表）
obsidian search query="Cache" format=json       # JSON 输出，便于脚本处理
obsidian search query="Cache" total             # 仅返回匹配数量
obsidian search:context query="易错点" limit=5   # 带匹配行上下文
obsidian recents                                # 最近打开的文件
obsidian random:read folder="03-数据结构"        # 随机读一篇（做抽检复习很好用）
```

## 二、创建与修改

```bash
obsidian create path="03-数据结构/新笔记.md" content="# 标题\n\n正文"   # 新建
obsidian append path="某笔记.md" content="追加内容"                     # 末尾追加
obsidian append path="某笔记.md" content="续行" inline                  # 追加且不换行
obsidian prepend path="某笔记.md" content="置顶内容"                    # 开头插入
obsidian rename path="旧名.md" name="新名.md"                           # 重命名（自动更新反链）
obsidian move path="某笔记.md" to="目标文件夹/"                          # 移动
obsidian delete path="某笔记.md"                                        # 删除（移入回收站）
obsidian delete path="某笔记.md" permanent                              # 永久删除，不进回收站
```

> ⚠️ **`move` 不会自动创建目标文件夹**：目标目录不存在会直接报 `ENOENT`。
> 移动前先确认目录已存在，否则用 `obsidian folders` 核对，或先 `mkdir -p` 建目录。

## 三、属性（Frontmatter / Properties）

```bash
obsidian properties                                   # 列出全库属性（默认 yaml）
obsidian properties counts sort=count                 # 带出现次数并按次数排序
obsidian properties path="某笔记.md"                   # 单文件的所有属性
obsidian property:read path="某笔记.md" name="mastery" # 读单个属性值
obsidian property:set path="某笔记.md" name="mastery" value="一轮中"        # 写入
obsidian property:set path="某.md" name="tags" value="a,b" type=list        # 指定类型
obsidian property:remove path="某笔记.md" name="draft"                      # 删除属性
```

`type` 可选：`text` `list` `number` `checkbox` `date` `datetime`

> ✅ 实用场景：批量更新复习掌握度
> `obsidian property:set path="03-数据结构/数据结构_05_树与二叉树.md" name="mastery" value="已掌握"`

## 四、链接与图谱

```bash
obsidian links path="00-主页.md"          # 出链列表
obsidian links path="00-主页.md" total    # 出链数量
obsidian backlinks path="00-主页.md"      # 反链（谁引用了我）
obsidian backlinks path="00-主页.md" counts   # 反链 + 链接次数
obsidian unresolved                       # 未解析链接（死链 / 尚未创建的笔记）
obsidian unresolved total                 # 死链数量 ← 建库后必查
obsidian orphans                          # 孤立笔记（无入链）
obsidian orphans total                    # 孤立笔记数量
obsidian deadends                         # 死胡同笔记（无出链）
obsidian outline path="某笔记.md"          # 标题大纲（tree|md|json）
```

> 💡 建库收尾动作：`unresolved total` 应为 0；`orphans` 中的笔记应逐个挂进 MOC。

## 五、任务（Checkbox）

```bash
obsidian tasks                            # 列出全库任务
obsidian tasks total                      # 任务总数
obsidian tasks path="某笔记.md"            # 指定文件的任务
obsidian tasks verbose                    # 带**行号**输出
obsidian task path="某笔记.md" line=6 toggle   # 切换第 6 行任务状态
```

> ⚠️ **行号是含 frontmatter 的绝对行号**（第 1 行就是 `---`），不是正文相对行号。
> 正确姿势：先 `obsidian tasks path="..." verbose` 拿到行号，再拿去 `toggle`；直接猜行号会报 `Line N is not a task`。

## 六、日记与模板

```bash
obsidian daily:path        # 输出今日日记的路径（不会创建文件）
obsidian daily             # 打开/创建今日日记
obsidian daily:read        # 读今日日记
obsidian daily:append content="今晚复盘"    # 往日记追加
obsidian templates         # 列出模板文件
obsidian template path="90-模板/章节笔记模板.md"   # 应用模板
```

依赖 `.obsidian/daily-notes.json` 与 `.obsidian/templates.json` 的配置：
- 目录不存在或未配置 → `Error: Folder "..." not found` / `No template folder configured`
- 修复：把 `folder` 设为**实际存在的目录**，或置为空字符串 `""`（表示库根目录）

## 七、插件与高级功能

```bash
obsidian plugins                          # 列出已装插件
obsidian plugins filter=core              # 只看核心插件
obsidian plugins:enabled versions         # 已启用的插件 + 版本
obsidian plugin:enable id="daily-notes" filter=core
obsidian plugin:disable id="xxx" filter=community
obsidian plugin:install id="dataview" enable       # 安装社区插件并启用
obsidian plugins:restrict on              # 开启受限模式（禁用社区插件）
obsidian commands filter="editor:"        # 列出可用命令 ID
obsidian command id="editor:toggle-bold"  # 执行命令
obsidian hotkeys                          # 热键列表
obsidian bases                            # 列出 base 文件
obsidian base:query path="某.base" format=json     # 查询 base
obsidian bookmark file="某笔记.md" title="要点"     # 添加书签
obsidian bookmarks                        # 书签列表
obsidian aliases verbose                  # 全库别名
```

## 八、维护命令

```bash
obsidian reload            # 重载库（外部改动文件后用）
obsidian restart           # 重启 Obsidian
obsidian history path="某笔记.md"             # 文件历史版本
obsidian history:restore path="某.md" version=2   # 恢复历史版本
obsidian diff path="某.md"                    # 对比本地/同步版本
```

## 九、环境中已验证的坑（实测记录）

| 现象 | 原因 | 解决 |
|---|---|---|
| `move` 报 `ENOENT` | 目标文件夹不存在，CLI 不自动创建 | 先确认/创建目标目录 |
| `task line=N` 报 `Line N is not a task` | 行号未计入 frontmatter | 用 `tasks verbose` 获取真实行号 |
| `daily:path` 报 `Folder not found` | `daily-notes.json` 指向不存在的目录 | 修正配置或置空 `""` |
| `templates` 报 `No template folder configured` | `templates.json` 的 folder 不存在/为空 | 创建模板目录后再配置 |
| 删除/移动后立即查询结果不符 | 索引未刷新 | 执行 `obsidian reload` 后重试 |
| `obsidian files` 偶发 `Command not found` | 刚 `reload` 时索引重建中 | 等待 1~2 秒重试 |
| 命令无输出 | 库未在 Obsidian 中打开 | 先用 GUI 打开目标库 |

## 十、典型工作流片段

**建库后健康检查**
```bash
obsidian files total && obsidian folders total && obsidian unresolved total && obsidian orphans total
```

**批量建笔记（保持索引同步）**
```bash
obsidian create path="03-数据结构/数据结构_06_图.md" content="# 06 图\n\n## 考点清单\n"
obsidian property:set path="03-数据结构/数据结构_06_图.md" name="mastery" value="未开始"
```

**把新笔记挂上 MOC**
```bash
obsidian append path="03-数据结构/数据结构-MOC.md" content="\n- [[数据结构_06_图]]"
```

**每日复习巡检**
```bash
obsidian tasks verbose | head -30
obsidian random:read folder="03-数据结构"
```
