# DSH-Desktop-EAC UI Skin Authoring Convention (Draft v0.11)

# DSH-Desktop-EAC皮肤创作公约（草案 v0.11）

> 状态：草案，开放讨论  
> 关键词：**必须（MUST）**、**应当（SHOULD）**、**可以（MAY）**、**禁止（MUST NOT）**  
> 适用对象：依赖DSH-Desktop-EAC的控件包、样式包、皮肤包、插件包的作者

---

> [!IMPORTANT]
>
> 本公约当前正处于草案阶段，不保证稳定性。
>
> 如果您愿意尝试基于本草案开发控件包、样式包、槽位包或是皮肤包，还请您做好草案修改可能引发破坏性修改的心理准备

## 1. 范围

本公约定义 **皮肤创作侧** 的契约。

任何依赖本 profile 的包 **必须** 满足本约定。  
不依赖本 profile 的实现不受约束。

本公约不定义主程序如何实现槽位、绑定表、拖放、浮窗等运行时机制。

---

## 2. 术语

| 术语          | 含义                                      |
| ------------- | ----------------------------------------- |
| Profile       | 本契约，`dsh-desktop-eac-ui-skin-profile` |
| 包 Package    | 控件包、样式包、皮肤包、插件包、槽位包    |
| 声明 Declare  | 包声明“我能提供某区块的控件/样式”         |
| 选择 Select   | 用户为某区块选中该包                      |
| 绑定 Bind     | 用户保存后，该包正式占用槽位              |
| 槽位 Slot     | 区块 × 类型（控件 / 样式）                |
| 区块名        | 短名，包内唯一，宿主自动加包名前缀        |
| 控件名        | 短名或完全限定名，区块内唯一              |
| 契约控件      | 由本体定义的公共控件名                    |
| 私有控件      | 非本体定义的控件，必须命名空间化          |
| 预定义状态    | Profile 定义的共享状态词汇                |
| 自定义状态    | 包私有的状态扩展                          |
| 预定义区块    | Profile 定义的标准区块                    |
| 自定义区块    | 槽位包声明的非标准区块                    |
| 槽位包        | 可选包类型，只声明槽位，不填充            |
| Binding Table | 记录槽位与绑定关系的表                    |
| 默认皮肤      | 内置包 `system.default`                   |
| Force-Enable  | 版本不匹配时强制启用                      |
| Toast         | 消息层，面向用户的通知                    |

---

## 3. 命名空间与标识符

### 3.1 命名空间

- 每个包 **必须** 声明全局唯一命名空间。
- 命名空间 **必须** 使用反向域名格式，例如 `com.example.cyber`。
- 禁止裸键，例如 `main`、`panel`、`toggle`。
- 系统保留前缀：`system.`、`surface.`、`dsh.`。

### 3.2 区块名

- 包作者只写 **短名**，例如 `left-sidebar`、`session`。
- 区块名 **必须** 在包内唯一。
- 宿主在读取声明时自动加上包名前缀，构造完整标识符。
- 完整标识符示例：

```text
com.example.cyber.left-sidebar
com.example.neon.session
```

- 完整标识符用于 Binding Table、审计、调试。
- 样式包 **不** 使用完整标识符。

### 3.3 控件名

控件名分为两类：

**契约控件**：

- 由本体定义的公共控件名，例如 `entry-list`、`toolbar`、`session-content`。
- 使用 **短名**。
- 在区块内唯一。
- 样式包可直接用短名匹配。
- 跨包语义一致，样式可复用。

**私有控件**：

- 非本体定义的控件。
- **必须** 使用完全限定名，命名空间化，例如 `com.example.cyber.overlay-button`。
- 在区块内唯一。
- 样式包需用完全限定名匹配。
- 不会与其它包的私有控件混淆。

规则：

- 契约控件名由本体保留。
- 包 **禁止** 将私有控件命名为契约控件名。
- 控件名 **不** 进入 Binding Table。
- 控件名 **不** 参与槽位唯一性。
- 契约控件名 **禁止** 命名空间化。
- 私有控件名 **必须** 命名空间化。

控件名的形态约束：

- 契约控件名 **必须** 为小写 kebab-case（`^[a-z][a-z0-9-]*$`），**禁止** 包含点号——点号是命名空间化的标志（§3.1）。
- 契约控件名是 **区块局部** 的：同名控件在不同区块中语义独立，样式包 **应当** 始终用区块与控件名联合定位（§7.4）。
- 本体自身实现使用、不承诺给样式包的私有节点，**可以** 使用保留前缀 `system.` 下的完全限定名，例如 `system.default.loading-spinner`。

日志中定位符 **建议** 使用 `包名 + 区块名 + 控件名`，或其它等价的唯一定位符，以便于 debug。

### 3.4 状态命名

状态分为 **预定义状态** 与 **自定义状态**。

**预定义状态** 由 profile 定义，是所有包共享的词汇：

| 状态          | 含义             |
| ------------- | ---------------- |
| `empty`       | 无内容或未初始化 |
| `idle`        | 空闲可交互       |
| `loading`     | 正在加载资源     |
| `running`     | 任务执行中       |
| `success`     | 最近操作成功     |
| `warning`     | 警告             |
| `error`       | 错误             |
| `disabled`    | 不可交互         |
| `collapsed`   | 已折叠           |
| `hidden`      | 隐藏但可恢复     |
| `docked`      | 停靠             |
| `floating`    | 浮窗             |
| `detached`    | 分离窗口         |
| `dragging`    | 正在拖动         |
| `drop-target` | 可作为放置目标   |
| `dangerous`   | 执行危险操作     |
| `animating`   | 动画中           |

规则：

- 预定义状态名由 profile 保留，**禁止** 命名空间化。
- 预定义状态 **不需要** 注册、绑定、撤销。
- 预定义状态 **只能** 由控件设置，样式包 **禁止** 设置状态。
- `.animating` 状态 **必须** 由控件设置。
- `.animating` 状态 **必须** 由控件在动画结束时移除。
- 控件 **应当** 监听 `animationend` 或 `transitionend` 事件来移除。
- 样式包 **禁止** 移除或设置 `.animating`。
- 宿主 **可以** 提供辅助 API，但不应自动移除。

**状态在 DOM 中的表示**：

- 状态通过 `data-state` 暴露（§7.4）。
- 一个元素 **可以** 同时处于多个预定义状态；`data-state` 取值为 **空格分隔** 的状态名列表，样式包 **应当** 用 `~=` 匹配，**禁止** 用 `=`：

```css
[data-region="session"] [data-control-name="composer"][data-state~="loading"] {
    /* ... */
}
```

- 交互态 **不是** 预定义状态：`hover`、`focus-visible`、`active`、`selected` 由伪类与 ARIA 属性承担，**禁止** 作为 `data-state` 的取值。

**自定义状态** 是包私有的状态扩展：

- 自定义状态名 **必须** 命名空间化，例如 `com.example.cyber.compiling`。
- 自定义状态 **不需要** 注册、绑定、撤销。
- 自定义状态 **只能** 由控件设置，样式包 **禁止** 设置状态。

---

## 4. 包类型

### 4.1 控件包 Control Package

控件包 **可以**：

- 对目标区块提供控件、功能、结构；
- 添加、删除、替换控件；
- 定义区块结构、排列、尺寸、断点、折叠、停靠策略；
- 声明预定义区块的控件；
- 通过 `dependencies` 引入槽位包，并填充其声明的位置（见 §4.5 / §6.3）；
- 添加快捷键声明；
- 通过 Toast 发布通知；
- 调用其它区块的内置 API。

控件包 **必须**：

- 声明命名空间、owner、版本、兼容范围；
- 声明每个区块的控件槽位；
- 为内部控件声明控件名；
- 契约控件用短名，私有控件用完全限定名；
- 提供 fallback 接口供自定义控件调用失败时使用。

### 4.2 样式包 Style Package

样式包 **只能**：

- 修改现有样式：CSS 变量、颜色、字体、边框、阴影、图标、动画；
- 使用更精确的选择器覆盖默认样式；
- 定义状态视觉；
- 针对区块级别声明样式；
- 针对控件级别写样式。

样式包 **禁止**：

- 新增、删除、替换结构、控件、功能；
- 改变控件功能；
- 设置状态；
- 注入功能逻辑。

### 4.3 皮肤包 Skin Package

皮肤包 **可以**：

- 整合控件包与样式包发布；
- 附带资源文件。

皮肤包 **必须**：

- 声明依赖的控件包、样式包与槽位包版本；
- 声明兼容的 profile 版本；
- 允许用户只替换其中一层。

### 4.4 插件包 Plugin Package

插件包 **可以**：

- 提供入口、面板、功能；
- 声明默认边栏（可选，未声明默认为 `left-sidebar`）；
- 声明允许停靠的区块。

插件包的完全环境权限属于插件公约，不属于本 profile。

### 4.5 槽位包 Slot Package（可选）

槽位包是 **可选** 包类型，只声明槽位，不填充。

槽位包 **可以**：

- 声明自定义区块；
- 声明区块的默认 Z-Index；
- 声明区块的 CSS 样式定义；
- 为自定义区块提供槽位，供其它包填充。

槽位包 **禁止**：

- 填充槽位；
- 提供控件或样式。

使用场景：

- 第三方定义槽位，让别的包填充；
- 需要将槽位定义与实现解耦时；
- 控件包需要自定义区块时——该区块 **必须** 由槽位包声明（见 §6.3），控件包可自行定义槽位包后作为依赖引入。

控件包与皮肤包若需使用槽位包声明的位置，**必须** 通过 `dependencies` 引入对应槽位包（§13.1 / §13.3）。控件包 **禁止** 自带 `customRegions`。

### 4.6 控件包是样式包的超集

- 控件包 **可以** 同时提供控件与样式；
- 控件槽位与样式槽位 **独立绑定**，互不影响；
- 用户可对同一区块选择同一包的控件与样式，也可分别选择不同包。

---

## 5. 三段式生命周期

### 5.1 声明 Declare

- 包安装时，向主程序声明“我能提供某区块的控件/样式”。
- 声明粒度 = 包 × 区块 × 类型，独立声明。
- 声明使用区块短名。
- 声明不等于生效。
- 声明不等于占用槽位。
- 自定义区块与预定义区块使用相同的声明流程。

### 5.2 选择 Select

- 用户在设置中为某区块选择该包。
- 选择阶段处于编辑态，未生效。
- 用户可取消选择。

### 5.3 绑定 Bind

- 用户点击保存修改时，选择转为绑定。
- 绑定后该包正式占用槽位并生效。
- 绑定原子切换：旧绑定撤销与新绑定建立同事务。
- 失败回滚到旧绑定。

---

## 6. 区块与绑定

### 6.1 区块定义

- 区块是界面上的大型布局区域。
- 每个区块拥有控件槽位与样式槽位。
- 槽位 = 区块 × 类型（控件 / 样式）。
- 每个槽位唯一绑定 1 个包。
- 绑定粒度为 **区块**，独立。
- 控件槽位与样式槽位独立。
- 区块分为预定义区块与自定义区块。

### 6.2 预定义区块

预定义区块由本 profile 定义，共 6 个：

`top-sidebar`、`bottom-sidebar`、`left-sidebar`、`right-sidebar`、`session`、`overlay`。

- 预定义区块的短名由 profile 保留，包 **禁止** 覆盖。
- 预定义区块 **应当** 声明其默认 Z-Index（§7.6）。
- 预定义区块各自拥有一个控件槽位与一个样式槽位，槽位名为 `<区块名>.control` 与 `<区块名>.style`（§6.1）。
- 预定义区块的绑定流程与普通区块一致。

**instance kind 不是区块**：`popup`、`dialog`、`floating-window` 是 **instance kind**（§8），由创建者自己的控件包与样式包渲染，**不** 进入 Binding Table。

### 6.3 自定义区块

自定义区块是槽位包声明的非标准区块。

- **槽位包** **可以** 声明自定义区块；控件包与皮肤包 **禁止** 声明，只能通过 `dependencies` 引入（§4.5 / §13.1 / §13.3）后填充。
- 自定义区块使用 **短名**，例如 `sidebar-overlay`。
- 短名 **必须** 在包内唯一。
- 宿主自动加包名前缀，构造完整标识符，例如 `com.example.slot.overlay.sidebar-overlay`。
- 自定义区块 **应当** 声明其默认 Z-Index。
- 自定义区块 **应当** 提供 CSS 样式定义。
- 自定义区块拥有自己的控件槽位与样式槽位。
- 自定义区块与预定义区块使用相同的声明、选择、绑定流程。
- 自定义区块冲突时，处理方式与正常方案一致（用户选择唯一生效者），但该冲突属于预期不会出现的行为。
- 自定义区块 **禁止** 覆盖预定义区块的短名。
- 自定义区块 **禁止** 与其它包的自定义区块完整名相同。

### 6.4 默认皮肤

- 默认皮肤为内置包 `system.default`。
- 默认绑定自动建立。
- 无皮肤时使用默认皮肤。
- 默认皮肤在用户界面中可见，便于切回。

### 6.5 绑定与禁用

- 禁用粒度 = 包 × 区块。
- 用户可只禁用某包的某区块，其余区块继续生效。
- 禁用整个包 = 撤销该包全部绑定。
- owner 消失 = 包卸载。
- 禁用后该区块回到默认。

### 6.6 重新启用

- 用户重新启用时，恢复到之前的绑定，除非该包已卸载。

---

## 7. 样式模型

### 7.1 混合规则

- 默认样式永远作为基底。
- 唯一绑定样式叠加其上。
- 混合结果由 CSS 覆盖规则决定。
- 每个区块样式槽位必定是“默认 + 唯一绑定”的混合产物。

### 7.2 完全替换

- 样式包想完全替换默认样式，需写更精确的选择器覆盖。
- 禁止通过非 CSS 手段替换结构。

### 7.3 加载顺序

包级顺序固定为：

1. 控件包布局样式
2. 样式包视觉 token
3. 样式包状态选择器

绑定级顺序：

- 默认样式先加载。
- 唯一绑定样式后加载。

宿主 **必须** 按此顺序装配，保证后加载者可覆盖先加载者，与 §7.1 / §7.2 的覆盖语义一致。

### 7.4 样式包查询模型

样式包按以下方式定位控件并应用状态样式：

```text
区块 控件名.状态
```

**契约控件** 使用短名：

```css
[data-region="session"] [data-control-name="session-content"][data-state="running"] {
    /* ... */
}
```

**私有控件** 使用完全限定名：

```css
[data-region="session"] [data-control-name="com.example.cyber.overlay-button"][data-state="running"] {
    /* ... */
}
```

规则：

- 区块定位使用 `data-region`。
- 控件定位使用 `data-control-name`。
- 状态定位使用 `data-state`。
- 契约控件用短名。
- 私有控件用完全限定名。
- 预定义状态直接使用共享词汇。
- 自定义状态使用命名空间化的状态名。
- 样式包 **禁止** 设置状态，只读取状态。
- 样式包 **不** 需要知道哪个控件包被绑定。
- 样式包只关心区块 + 控件名 + 状态是否匹配。

### 7.5 控件级别样式

- 样式包 **可以** 针对控件级别写样式。
- 契约控件级别选择器 **应当** 使用短名。
- 私有控件级别选择器 **必须** 使用完全限定名。
- 控件名 **应当** 在 DOM 中以稳定属性暴露，例如 `data-control-name`。
- 控件名 **不** 参与槽位绑定，仅作为样式与调试锚点。

### 7.6 自定义区块样式与 Z-Index

- 自定义区块 **应当** 使用 CSS 定义其样式。
- 自定义区块 **应当** 提供默认 Z-Index。
- 用户 **可以** 手动调整 Z-Index 并覆盖默认值。
- 用户覆盖 **必须** 持久化。
- Z-Index 覆盖 **不** 影响区块绑定。
- Z-Index 覆盖 **应当** 在设置界面中可编辑。
- 自定义区块 **应当** 建立独立堆叠上下文，避免与预定义区块相互干扰。

`customRegions` 属 **槽位包** 清单（§13.5），控件包与皮肤包不带该字段（§4.5 / §6.3）。示例：

```json
{
    "customRegions": [
        {
            "id": "sidebar-overlay",
            "type": "region",
            "zIndex": {
                "default": 1200
            },
            "style": {
                "css": "com.example.slot.overlay.sidebar-overlay"
            }
        }
    ]
}
```

---

## 8. 浮窗与弹窗

- 浮窗与弹窗是 **实例**，不是区块。
- 内部无区块。
- 控件/样式唯一。
- 实例基数 `0..n`。
- 实例不进入 Binding Table。

---

## 9. 快捷键

- 快捷键 **必须** 在入口文件中注册。
- 快捷键单独列表，不进入 Binding Table。
- 快捷键 ID **必须** 命名空间化。
- 快捷键冲突检测在绑定后。
- 冲突时禁用冲突的快捷键，不禁用功能。
- 不冲突后立即启用。
- 用户可以手动修改。
- 注册的快捷键是默认快捷键。

---

## 10. 消息层（Toast）

- Toast 就是消息层。
- 级别：`success` / `info` / `warning` / `failed`。
- 面向用户，不做控件间通信。
- 不需要频道、订阅、命名空间、owner、绑定。
- Toast **应当** 带 `source` 字段，便于排查。
- 通知 **应当** 使用 Toast，非必要不自定义通知 UI。

```js
surface.toast.publish({
    level: "success",
    title: "控件包已启用",
    body: "com.example.cyber",
    source: "com.example.cyber",
    timeout: 4000
});
```

---

## 11. 区块间调用与 Fallback

### 11.1 区块间调用

- 区块之间 **可以** 直接调用。
- 对内置控件 API **可以** 直接修改。
- 例：打开会话后，会话区加载历史会话。

### 11.2 Fallback

- 对自定义控件，**必须** 提供 fallback。
- 控件不存在时 **禁止** 导致程序崩溃。
- 调用失败时走 fallback。
- Fallback 采用声明式 + 运行时检测。
- Fallback 时需要添加审计日志，并明确通过 Toast 告知用户执行失败。
- 审计日志 **建议** 记录 `包名 + 区块名 + 控件名` 或等价的唯一定位符。
- 内置 API 由 profile 定义版本。

```json
{
    "fallback": {
        "onMissing": "system.default",
        "onError": "surface.toast.warning",
        "onTimeout": 3000
    }
}
```

---

## 12. 版本与兼容

### 12.1 版本命名

| 名称         | 字段                    | 示例                                   |
| ------------ | ----------------------- | -------------------------------------- |
| Profile 版本 | `compatibility.profile` | `dsh-desktop-eac-ui-skin-profile@^0.3` |
| Package 版本 | `version`               | `1.0.0`                                |
| 请求版本     | `requestedVersion`      | `^0.3`                                 |
| 实现版本     | `implementedVersion`    | `0.3.0`                                |

### 12.2 Force-Enable

- 版本不匹配时，用户 **可以** 强制启用。
- Force-Enable **必须** 记录 `{请求版本, 实现版本, 用户确认}`。
- 不添加降级模式。
- 强制启用时运行时不保证可用性。
- 流程：临时应用 → 用户检查界面 → 确认保存。
- 超时未确认：恢复原设置。

---

## 13. 包格式

### 13.1 控件包

```json
{
    "id": "com.example.cyber",
    "type": "control",
    "version": "1.0.0",
    "compatibility": {
        "profile": "dsh-desktop-eac-ui-skin-profile@^0.3"
    },
    "owner": "com.example.cyber",
    "dependencies": {
        "com.example.slot.overlay": "1.0.0"
    },
    "declarations": [
        {
            "region": "left-sidebar",
            "type": "control",
            "controls": [
                {
                    "name": "entry-list",
                    "fallback": {
                        "onMissing": "system.default"
                    }
                },
                {
                    "name": "com.example.cyber.overlay-button"
                }
            ]
        },
        {
            "region": "session",
            "type": "control",
            "controls": [
                {
                    "name": "session-content"
                }
            ]
        }
    ],
    "shortcuts": [
        {
            "id": "com.example.cyber.toggle-overlay",
            "default": "Ctrl+Shift+O",
            "scope": "global"
        }
    ]
}
```

字段说明：

- `dependencies`：**可以**。键为所依赖的包的 `id`，值为版本要求。控件包若需使用自定义区块，**必须** 依赖声明该区块的槽位包（§4.5 / §6.3）；控件包 **禁止** 自带 `customRegions`。

### 13.2 样式包

```json
{
    "id": "com.example.neon",
    "type": "style",
    "version": "1.0.0",
    "compatibility": {
        "profile": "dsh-desktop-eac-ui-skin-profile@^0.3"
    },
    "owner": "com.example.neon",
    "declarations": [
        {
            "region": "session",
            "type": "style",
            "tokens": {
                "--surface-bg": "#0b1020",
                "--surface-animation-duration": "180ms"
            },
            "controlSelectors": [
                {
                    "control": "session-content",
                    "states": {
                        "running": {
                            "--session-content-bg": "#111936"
                        },
                        "com.example.cyber.compiling": {
                            "--session-content-bg": "#2a1a3a"
                        }
                    }
                },
                {
                    "control": "com.example.cyber.overlay-button",
                    "states": {
                        "dangerous": {
                            "--overlay-button-bg": "#ff5c7a"
                        }
                    }
                }
            ]
        }
    ]
}
```

### 13.3 皮肤包

```json
{
    "id": "com.example.skin.cyber",
    "type": "skin",
    "version": "1.0.0",
    "compatibility": {
        "profile": "dsh-desktop-eac-ui-skin-profile@^0.3"
    },
    "owner": "com.example.skin.cyber",
    "control": "com.example.cyber",
    "style": "com.example.neon",
    "dependencies": {
        "com.example.cyber": "1.0.0",
        "com.example.neon": "1.0.0",
        "com.example.slot.overlay": "1.0.0"
    },
    "assets": []
}
```

字段说明：

- `control` / `style`：**必须**。分别指向所整合的控件包与样式包的 `id`。
- `dependencies`：**必须**。键为所依赖的包的 `id`，值为版本要求；**必须** 同时包含 `control` 与 `style` 指向的两个包（对应 §4.3 的义务）。皮肤包 **可以** 组合槽位包，同样通过本字段引入——槽位包 **不** 设顶层字段，因为 `control` / `style` 有顶层字段是出于 §7 / §8 的唯一绑定语义，而一个皮肤包可以依赖多个槽位包。
- `assets`：**可以**。随包分发的资源清单，元素为**包根目录下的相对文件名**；为空数组表示不含额外资源。

### 13.4 插件包

```json
{
    "id": "com.example.plugin.chat",
    "type": "plugin",
    "version": "1.0.0",
    "compatibility": {
        "profile": "dsh-desktop-eac-ui-skin-profile@^0.3"
    },
    "owner": "com.example.plugin.chat",
    "entry": {
        "id": "com.example.plugin.chat.entry.main",
        "label": "Chat",
        "defaultRegion": "left-sidebar",
        "allowedRegions": ["left-sidebar", "right-sidebar", "floating-window", "detached-window"],
        "element": "com.example.plugin.chat.root",
        "draggable": true
    }
}
```

插件 **不必** 声明默认边栏。未声明 `entry.defaultRegion` 时，默认放到 `left-sidebar`。

### 13.5 槽位包（可选）

```json
{
    "id": "com.example.slot.overlay",
    "type": "slot",
    "version": "1.0.0",
    "compatibility": {
        "profile": "dsh-desktop-eac-ui-skin-profile@^0.3"
    },
    "owner": "com.example.slot.overlay",
    "customRegions": [
        {
            "id": "sidebar-overlay",
            "type": "region",
            "zIndex": {
                "default": 1
            },
            "style": {
                "css": "com.example.slot.overlay.sidebar-overlay"
            }
        }
    ]
}
```

槽位包 **只声明，不填充**（§4.5）。控件包或皮肤包若要使用其中声明的位置，**必须** 通过各自的 `dependencies` 字段引入本包（§13.1 / §13.3）；反之，控件包 **禁止** 自带 `customRegions`。

---

## 14. 最小合规清单

- [ ] 包声明命名空间与 owner
- [ ] 所有键带命名空间，无裸键
- [ ] 区块名使用短名，包内唯一
- [ ] 契约控件使用短名
- [ ] 私有控件使用完全限定名
- [ ] 契约控件不命名空间化
- [ ] 私有控件必须命名空间化
- [ ] 契约控件名小写 kebab-case，不含点号
- [ ] 预定义状态禁止命名空间化
- [ ] 自定义状态必须命名空间化
- [ ] 状态以 `data-state` 暴露，多状态用空格分隔
- [ ] 声明粒度 = 包 × 区块 × 类型
- [ ] 控件包与样式包职责分离
- [ ] 控件包提供 fallback
- [ ] 槽位包可声明自定义区块；控件包与皮肤包须通过 `dependencies` 引入
- [ ] 自定义区块使用短名，宿主加包前缀
- [ ] 自定义区块声明默认 Z-Index
- [ ] 自定义区块提供 CSS 样式
- [ ] 自定义区块拥有独立控件槽位与样式槽位
- [ ] 自定义区块不覆盖预定义区块短名
- [ ] 样式包可针对区块级别与控件级别写样式
- [ ] 样式包按 `区块 控件名.状态` 查询
- [ ] 样式包对契约控件用短名
- [ ] 样式包对私有控件用完全限定名
- [ ] 快捷键 ID 命名空间化，注册在入口文件
- [ ] 通知使用 Toast
- [ ] 包声明 `compatibility.profile`
- [ ] Force-Enable 记录 `{请求版本, 实现版本, 用户确认}`
- [ ] 不添加降级模式
- [ ] 日志建议记录 `包名 + 区块名 + 控件名` 或等价唯一定位符

---

## 15. 附录：缺口状态

本文档末尾原列有 6 项「后续应补充」。截至 Task 1.4（内置 UI 皮肤包拆分，见 `docs/adr/0009`）落地，现状如下：

| 项 | 状态 | 期间的判据 | 说明 |
| --- | --- | --- | --- |
| 控件名 DOM 暴露规则 | **已定案** | §3.3 / §3.4 / §7.4 / §7.5 | Task 1.4 落地了 `data-region` / `data-control-name` / `data-state` 三属性契约 |
| 正式 JSON Schema | **部分可冻结** | 按 §13 各包型的字段表手工核对 | 4 种包型的 manifest 字段集已由实现与测试冻结；schema 文件本身仍待补 |
| fallback 接口详情 | **待补** | 按 §11.2 原条款 | Task 1.4 未涉及，§11.2 的控件调用降级接口细节仍待补 |
| Toast 结构详情 | **推迟** | 按 §10 现有示例与级别枚举实现 | 需外部包实际接入才能确定合理形态 |
| 快捷键冲突提示格式 | **推迟** | 按 §9 现有规则实现，冲突时禁用快捷键而非功能 | 同上 |
| 自定义区块 Z-Index 用户覆盖存储格式 | **推迟** | 暂不实现该能力 | 同上 |

**推迟不等于留白**：上表「期间的判据」列给出可照做的临时依据。推迟项的共性理由是——它们都需要 **外部包实际接入** 才能确定合理形态。

### 15.1 本次由实现定案的接口

以下接口此前属空白或与实现不符，Task 1.4 落地后已可写实，本次一并补入正文：

| 接口 | 正文位置 | 定案内容 |
| --- | --- | --- |
| 预定义区块清单 | §6.2 | 6 个区块；槽位命名 `<区块名>.control` / `<区块名>.style`；`popup` / `dialog` / `floating-window` 为 instance kind，不进 Binding Table |
| 控件名形态 | §3.3 | 契约控件名小写 kebab-case、无点号；区块局部作用域；实现私有节点用 `system.` 前缀 |
| 状态 DOM 表示 | §3.4 | `data-state` 空格分隔多值，用 `~=` 匹配；交互态由伪类与 ARIA 承担 |
| 包级加载顺序 | §7.3 | 控件包布局 → 样式包 token → 样式包状态选择器 |

### 15.2 与实现不一致、待裁定

| 项 | 实现现状 | 公约现状 |
| --- | --- | --- |
| `compatibility.forceable` | 4 种内置包 manifest 均含 `"forceable": false`，且由测试断言 | 正文从未定义该字段，§12.2 只描述「用户可强制启用」的 **行为** |
| 槽位包的位置声明字段 | 顶层 `regions[]`，元素为 `{ name, controlSlot, styleSlot, defaultZIndex }`；另有 `instanceKinds[]` | §13.5 写作 `customRegions[]`，元素为 `{ id, type, zIndex: { default }, style: { css } }` |

两处均需维护者裁定，本次不改动正文：

- `forceable` 的存废：或写入 §12.2 / §13 成为包侧契约，或从实现中移除。
- 槽位包位置声明的字段名与形状：§13.5 与实现两者字段名、嵌套结构均不同，**按 §13.5 编写的槽位包不会被现有实现解析**。需择一为准（§7.6 的示例同属这一段）。

---

> 本公约为皮肤创作侧契约。
>
> **关于主程序侧的实现契约**：皮肤创作侧只需知道「宿主如何暴露挂接点」。主程序侧的完整实现规范（区块运行时、绑定表、皮肤选择与切换、`/skin/` 路由实现等）不在本公约范围内。此前本文档以《本体开发公约》指称该侧文档，但该文档 **尚未创建**；在此明确其范围为「宿主实现侧」，不以未发布的文档作为本公约的引用依据。
