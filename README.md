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

### 1.1 两类皮肤形态

本公约覆盖 **两类形态** 的皮肤。二者共用同一 profile 名，但形态与适用条款不同：

| 形态 | 服务对象 | 适用章节 |
| --- | --- | --- |
| **客户端皮肤包** | dsh Web UI 的区块、控件与状态 | §3–§12 |
| **壳层皮肤包** | 宿主壳层自有 UI（启动页、退出提示、错误页、资源缺失降级页等） | §3.1、§8、§12、**§15** |

判定规则：清单中 `kind` 字段为 `"shell-skin"` 者为 **壳层皮肤包**，适用 §15；其余为 **客户端皮肤包**，适用 §3–§12。

两类形态的身份 **必须** 单一——同一个包不得同时声明为两类，也不得混放于同一目录。

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
| 自定义区块    | 控件包或槽位包声明的非标准区块            |
| 槽位包        | 可选包类型，只声明槽位，不填充            |
| Binding Table | 记录槽位与绑定关系的表                    |
| 默认皮肤      | 内置包 `system.default`                   |
| Force-Enable  | 版本不匹配时强制启用                      |
| Toast         | 消息层，面向用户的通知                    |
| 壳层皮肤包    | 服务宿主壳层自有 UI 的皮肤包，见 §15      |
| 客户端皮肤包  | 服务 dsh Web UI 的皮肤包，见 §3–§12       |
| 控件调用降级  | 控件不存在或调用失败时转入替代行为，见 §11.2 |
| 取值兜底      | CSS 变量缺失时回退到字面量，见 §15.5      |

> **两处易混术语**：
>
> - **「控件调用降级」与「取值兜底」是两种不同机制**。前者是 §11.2 的 `fallback`——控件调用失败时的行为替代，需要审计日志与用户提示；后者是 §15.5 的 CSS 变量回退——取值缺失时回退到字面量，静默生效。二者可同时存在，互不替代。
> - **「皮肤包」是统称**，涵盖客户端皮肤包与壳层皮肤包两种形态，以清单的 `kind` 字段区分（见 §1.1）。

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
- 声明自定义区块；
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

- 声明依赖的控件包与样式包版本；
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
- 需要将槽位定义与实现解耦时。

若不使用槽位包，控件包 **可以** 自行声明并填充自定义区块。

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

- 预定义区块由本 profile 定义，例如 `left-sidebar`、`session`、`topbar`。
- 预定义区块的短名由 profile 保留，包 **禁止** 覆盖。
- 预定义区块的绑定流程与普通区块一致。

### 6.3 自定义区块

自定义区块是控件包或槽位包声明的非标准区块。

- 控件包或槽位包 **可以** 声明自定义区块。
- 自定义区块使用 **短名**，例如 `sidebar-overlay`。
- 短名 **必须** 在包内唯一。
- 宿主自动加包名前缀，构造完整标识符，例如 `com.example.cyber.sidebar-overlay`。
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

- 默认样式先加载。
- 唯一绑定样式后加载。

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
                "css": "com.example.cyber.sidebar-overlay"
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

**关于壳层皮肤包的取值**：壳层皮肤包 **必须** 声明 `compatibility.forceable` 为 `false`。理由见 §15.7——壳层 UI 属宿主进程内的既有界面，版本不匹配时强制启用无法保证可用性；且壳层皮肤缺失时已有 §15.5 的兜底契约保证可读，无需 Force-Enable 路径。

§13.1–§13.5 各示例中的 `forceable: true` 为客户端形态的取值，不适用于壳层皮肤。

---

## 13. 包格式

### 13.1 控件包

```json
{
    "id": "com.example.cyber",
    "type": "control",
    "version": "1.0.0",
    "compatibility": {
        "profile": "dsh-desktop-eac-ui-skin-profile@^0.3",
        "forceable": true
    },
    "owner": "com.example.cyber",
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
    "customRegions": [
        {
            "id": "sidebar-overlay",
            "type": "region",
            "zIndex": {
                "default": 1200
            },
            "style": {
                "css": "com.example.cyber.sidebar-overlay"
            }
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

### 13.2 样式包

```json
{
    "id": "com.example.neon",
    "type": "style",
    "version": "1.0.0",
    "compatibility": {
        "profile": "dsh-desktop-eac-ui-skin-profile@^0.3",
        "forceable": true
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
        "profile": "dsh-desktop-eac-ui-skin-profile@^0.3",
        "forceable": true
    },
    "owner": "com.example.skin.cyber",
    "control": "com.example.cyber",
    "style": "com.example.neon",
    "dependencies": {
        "com.example.cyber": "1.0.0",
        "com.example.neon": "1.0.0"
    },
    "assets": []
}
```

字段说明：

- `control` / `style`：**必须**。分别指向所整合的控件包与样式包的 `id`。
- `dependencies`：**必须**。键为所依赖的包的 `id`，值为版本要求；**必须** 同时包含 `control` 与 `style` 指向的两个包（对应 §4.3 的义务）。
- `assets`：**可以**。随包分发的资源清单，元素为**包根目录下的相对文件名**；为空数组表示不含额外资源。

> **与壳层皮肤包的区分**：本节 `control` / `style` 指向 **包的 id**；壳层皮肤包（§13.6 / §15）中的同名字段指向 **层标识**，语义不同。判定时先看 `kind` 字段——`kind: "shell-skin"` 者适用 §15。

### 13.4 插件包

```json
{
    "id": "com.example.plugin.chat",
    "type": "plugin",
    "version": "1.0.0",
    "compatibility": {
        "profile": "dsh-desktop-eac-ui-skin-profile@^0.3",
        "forceable": true
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
        "profile": "dsh-desktop-eac-ui-skin-profile@^0.3",
        "forceable": true
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

### 13.6 壳层皮肤包（`kind: "shell-skin"`）

壳层皮肤包 **不** 属于 §13.1–13.5 的任一类型。它服务宿主程序的 **壳层自有 UI**，不参与区块、控件、状态与 Binding Table 模型——完整规范见 §15。

```json
{
    "id": "system.default",
    "name": "EAC 本体默认皮肤",
    "nameEn": "EAC Shell Default",
    "type": "skin",
    "kind": "shell-skin",
    "version": "1.0.0",
    "compatibility": {
        "profile": "dsh-desktop-eac-ui-skin-profile@^0.3",
        "forceable": false
    },
    "owner": "io.github.dsh-eac",
    "control": "system.shell-controls",
    "style": "system.shell-style",
    "dependencies": {
        "system.shell-controls": "1.0.0",
        "system.shell-style": "1.0.0"
    },
    "description": "壳层控件与样式皮肤包。tokens.css 定义全部视觉 token，controls.css 只消费 token。",
    "consumers": [
        "tauri-shell/src/main.rs",
        "tauri-shell/src/exit-overlay.js"
    ],
    "assets": ["tokens.css", "controls.css"]
}
```

字段语义、目录结构与降级契约见 §15。**注意**：本节 `control` / `style` 的语义是 **层标识**，与 §13.3 皮肤包中指向包 `id` 的同名字段不同，见 §15.3。

---

## 14. 最小合规清单

> **适用范围**：本清单默认面向 §3–§12 定义的 **客户端皮肤与各包型**。判定 **壳层皮肤包** 时，只适用以下 4 条：
>
> - 包声明命名空间与 owner
> - 所有键带命名空间，无裸键
> - 包声明 `compatibility.profile`
> - 不添加降级模式
>
> 其余条目所涉及的「区块」「控件名」「状态」「Binding Table」「快捷键」「Toast」等概念在壳层皮肤中 **不存在**，判定时应记为 **不适用（N/A）**，不得记为未通过。壳层皮肤包的完整清单见 §16.1。

- [ ] 包声明命名空间与 owner
- [ ] 所有键带命名空间，无裸键
- [ ] 区块名使用短名，包内唯一
- [ ] 契约控件使用短名
- [ ] 私有控件使用完全限定名
- [ ] 契约控件不命名空间化
- [ ] 私有控件必须命名空间化
- [ ] 预定义状态禁止命名空间化
- [ ] 自定义状态必须命名空间化
- [ ] 声明粒度 = 包 × 区块 × 类型
- [ ] 控件包与样式包职责分离
- [ ] 控件包提供 fallback
- [ ] 控件包可声明自定义区块
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

## 15. 壳层皮肤包

### 15.1 适用对象与定位

本节定义 **壳层皮肤包** 的契约。

壳层皮肤服务宿主程序的 **壳层自有 UI**——启动页、退出确认、错误页、资源缺失降级页等。它 **不** 包含：

- dsh Web UI 的客户端区域（那由 §3–§12 约束）；
- 以客户端插件形式分发的皮肤（那是另一类分发物，身份必须单一，不与本类混放）。

壳层 UI 在宿主进程内渲染，不进入 Web UI 的区块体系。因此本节明确：§5 三段式生命周期、§6 区块与 Binding Table、§7.4 的 `区块 控件名.状态` 查询模型 **均不适用**于壳层皮肤；§3.1 命名空间、§8 浮窗与弹窗、§12 版本与兼容 **适用**。

### 15.2 包目录结构

一个目录即一个包，包含四个文件：

| 文件 | 必要性 | 用途 |
| --- | --- | --- |
| `skin.json` | **必须** | 包清单，见 §15.3 |
| `tokens.css` | **必须** | 视觉 token 的单一事实源，见 §15.4 |
| `controls.css` | **必须** | 壳层控件类样式，只消费 token，见 §15.4 |
| `README.md` | 应当 | 包说明，含所遵循的 profile 版本与保留 ID 声明 |

Skin、Control、Style 三层在 `skin.json` 中以独立标识与版本依赖表达，**但物理上共存于同一目录**，不拆分为多个包。

目录名 **应当** 与包身份一致，便于人工识别；包身份以 `skin.json` 字段为准，不以目录名为准。

### 15.3 `skin.json` 清单

| 字段 | 必要性 | 类型 | 语义 |
| --- | --- | --- | --- |
| `id` | **必须** | string | 包标识。默认皮肤使用公约保留 ID `system.default`（§3.1 系统保留前缀）；其余包使用反向域名 |
| `type` | **必须** | string | 固定为 `"skin"` |
| `kind` | **必须** | string | 固定为 `"shell-skin"`，表示扩展分类，不替代 `type` |
| `version` | **必须** | string | 包版本，见 §12.1 |
| `compatibility` | **必须** | object | 见下 |
| `compatibility.profile` | **必须** | string | 兼容的 profile 版本，见 §12.1 |
| `compatibility.forceable` | **必须** | boolean | 是否允许 Force-Enable；壳层皮肤 **必须** 为 `false`，见 §15.7 |
| `owner` | **必须** | string | 命名空间所有者，反向域名，见 §3.1 |
| `control` | **必须** | string | **控件层标识**，见下方注意 |
| `style` | **必须** | string | **样式层标识**，见下方注意 |
| `dependencies` | **必须** | object | 层标识 → 版本要求；**必须** 同时包含 `control` 与 `style` 两条（对应 §4.3 的义务） |
| `assets` | **必须** | string[] | 包根目录下的 **相对文件名** 清单，即随包分发的资源 |
| `consumers` | **必须** | string[] | 消费方路径清单——宿主侧读取本包 token 的文件，供变更影响分析使用 |
| `name` / `nameEn` | 可以 | string | 显示名（中文 / 英文） |
| `description` | 可以 | string | 包描述 |

> **注意 `control` / `style` 的语义差异**：在 §13.3 皮肤包中，`control` / `style` 指向 **包的 id**；在壳层皮肤包中，它们指向 **层标识**。两者同名不同义。壳层皮肤把三层收在同一目录内，用层标识而非包引用来表达可替换性，`dependencies` 记录各层的版本要求。

### 15.4 视觉 token 与样式层

**`tokens.css` — 单一事实源**

- **必须** 在统一的命名空间下定义全部视觉 token。壳层皮肤 **禁止** 定义或覆盖 dsh 内核的 token 命名空间。
- 同 profile 下的不同壳层皮肤包，**应当** 实现 **相同的 token 名集合**——只替换取值，不增删 token 名。这样宿主才能在不改消费代码的前提下更换皮肤包。
- token **必须** 只在 `tokens.css` 中定义；消费处 **禁止** 重复定义。

**`controls.css` — 控件类样式**

- **只能** 消费 `tokens.css` 定义的 token，**禁止** 出现裸色值等未 token 化的视觉字面量。
- **必须** 使用统一的包作用域类名限定选择器，不污染宿主页面的其它区域。
- **禁止** 包含结构、事件或业务逻辑——样式包只负责外观。

> **与 §7.4 的关系**：§7.4 的 `data-region` / `data-control-name` / `data-state` 查询模型面向 Web UI 的区块与控件。壳层 UI 无区块与控件名概念，改用 **包作用域类名 + token 消费** 表达样式作用范围。两者不冲突，各自适用于各自的形态。

### 15.5 降级契约

壳层皮肤包可能缺失或加载失败（资源未随包、直接打开文件、早期降级路径等）。此时页面 **必须** 仍可读，**禁止** 因皮肤缺失而崩溃或白屏。

契约：

- 每一处 token 消费 **必须** 写成 `var(<token>, <兜底字面量>)` 形式；
- 兜底字面量 **必须** 与拆分前的取值一致，保证皮肤包缺失时视觉降级到既定基线；
- token 定义处 **禁止** 依赖运行时的皮肤包存在性判断。

> **术语说明**：本节的「兜底」指 **CSS 变量取值失败时的字面量回退**，与 §11.2 的 `fallback`（**控件调用失败时的降级**）是两种不同机制。§2 术语表已将二者分列。

### 15.6 与 Binding Table 的关系

壳层皮肤包 **不进入** Binding Table。

壳层页与退出确认属 §8 定义的 **实例**（独立窗口或弹层），按 §8 不进入区块 Binding Table。壳层皮肤包整体同理——它不存在「槽位」「绑定」概念，也不参与用户选择。

### 15.7 版本与兼容

- 包 **必须** 声明所兼容的 `compatibility.profile`（§12.1）。
- **`forceable` 取值为 `false`**。壳层 UI 属宿主进程内的既有界面，版本不匹配时强制启用无法保证可用性；且壳层皮肤缺失时已有 §15.5 的兜底契约保证可读，无需 Force-Enable 路径。
- 壳层皮肤的版本不匹配处理方式 **待定**：宿主当前固定加载单一目录，不匹配时的行为尚无实践依据。此项见 §16.3 的缺口状态。

---

## 16. 附录：合规判定与缺口状态

### 16.1 壳层皮肤包的合规清单

壳层皮肤包的合规要求见 §15，摘要如下：

- [ ] `skin.json` 含 §15.3 全部「必须」字段
- [ ] `type` 为 `"skin"`，`kind` 为 `"shell-skin"`
- [ ] `dependencies` 同时包含 `control` 与 `style` 两条
- [ ] 四件套齐备（`skin.json` / `tokens.css` / `controls.css` / `README.md`）
- [ ] `tokens.css` 定义全部 token，且命名空间不与内核冲突
- [ ] 同 profile 的不同壳层皮肤包，token 名集合一致
- [ ] `controls.css` 只消费 token，无裸视觉字面量
- [ ] `controls.css` 选择器限定在包作用域类名下
- [ ] 每处 token 消费都有兜底字面量
- [ ] `forceable` 为 `false`
- [ ] 不定义或覆盖内核 token 命名空间

### 16.2 判定权威性

当本节与宿主实现（含其自动化门禁）出现偏离时：

- **壳层皮肤形态**：判据以 **宿主实现与其门禁测试** 为准。壳层形态已有落地实现与自动化门禁，它们是权威事实源；本公约与之偏离时应修正公约，而非要求实现让步。
- **客户端皮肤形态**：目前尚无落地实现，本公约为唯一依据；实现落地时如出现偏离，由维护者裁定并记录。

无论哪种情况，**判定时应记录所依据的版本与来源**。

### 16.3 缺口状态

本文档末尾原列有 6 项「后续应补充」。现状如下：

| 项 | 状态 | 期间的判据 | 说明 |
| --- | --- | --- | --- |
| 正式 JSON Schema | **待补** | 按 §13 各包型的字段表手工核对 | 字段集需先冻结 |
| fallback 接口详情 | **部分已补** | §15.5 兜底契约 + §11.2 原条款 | §11.2 的控件调用降级接口细节仍待补 |
| Toast 结构详情 | **推迟** | 按 §10 现有示例与级别枚举实现 | 需外部包实际接入才能确定合理形态 |
| 快捷键冲突提示格式 | **推迟** | 按 §9 现有规则实现，冲突时禁用快捷键而非功能 | 同上 |
| 自定义区块 Z-Index 用户覆盖存储格式 | **推迟** | 暂不实现该能力；区块按声明的默认 Z-Index | 同上 |
| 控件名 DOM 暴露规则 | **推迟** | 客户端形态按 §7.4 / §7.5 现行写法 | 同上 |

**推迟不等于留白**：上表「期间的判据」列给出可照做的临时依据。推迟项的共性理由是——它们都需要 **外部包实际接入** 才能确定合理形态。

---

> 本公约为皮肤创作侧契约。
>
> **关于主程序侧的实现契约**：皮肤创作侧只需知道「宿主如何暴露挂接点」。主程序侧的完整实现规范（区块运行时、绑定表、皮肤选择与切换、`/skin/` 路由实现等）不在本公约范围内。此前本文档以《本体开发公约》指称该侧文档，但该文档 **尚未创建**；在此明确其范围为「宿主实现侧」，不以未发布的文档作为本公约的引用依据。
