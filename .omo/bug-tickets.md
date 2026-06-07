# Bug 票 & 修改记录

## BUG-001: drawBombEffect 负半径导致白屏死机
**报告日期**: 当前
**状态**: 已修复

### 现象
- 角色死亡或放雷时屏幕白屏，无法操作，需按 F5 刷新
- 控制台报错: `Uncaught IndexSizeError: Failed to execute 'arc' on 'CanvasRenderingContext2D': The radius provided (-0.354008) is negative.`

### 根因
`drawBombEffect()` 中 `ctx.arc()` 的 radius 参数出现负值。由浮点精度问题导致。

### 修复方法
在 drawBombEffect 的 `ctx.arc()` 调用中添加 `Math.max(0, radius)` 保护。

---

## BUG-002: 灵梦副子弹从自身发射
**报告日期**: 当前
**状态**: 已修复

### 现象
灵梦的副子弹从玩家本身发射，而不是从辅助子机位置发射。

### 根因
`autoFire()` 中所有角色的副子弹都使用 `createBullet()`，该函数固定从 `game.player` 位置发射。

### 修复方法
修改 `autoFire()` 中灵梦分支，副子弹从 `game.companions[i].x/y` 位置发射。

---

## BUG-003: 早苗副子弹带跟踪
**报告日期**: 当前
**状态**: 已修复

### 现象
早苗的副子弹有跟踪效果，但设计要求是无跟踪的 V 字形扇形弹道。

### 根因
早苗副子弹代码引用了 `sortedEnemies` 进行角度计算。

### 修复方法
改回固定角度扇形弹道，不使用敌人位置计算角度。

---

## BUG-004: 敌人子弹颜色错误
**报告日期**: 当前
**状态**: 已修复

### 现象
敌人子弹颜色与玩家副子弹混淆。

### 修复方法
敌人子弹改为黄色 `#ffcc00`，SUB_COLORS 中移除黄色。

---

## CHANGE-001: Debug 按钮默认开启
**状态**: 已实现
**方法**: HTML 中 `<input>` 标签添加 `checked` 属性。

## CHANGE-002: Bomb 后 +1 秒无敌
**状态**: 已实现
**方法**: bomb 持续时间结束后额外给 1 秒无敌。

## CHANGE-003: ESC 暂停游戏
**状态**: 已实现
**方法**: 新增 PAUSED 状态，按 ESC 切换 PLAYING/PAUSED。

---

## CHANGE-004: Debug 默认 power=5
**状态**: 已实现
**方法**: enterPlaying 中 `game.power = game.debugMode ? 5 : 0`。启动时读取 checkbox 状态初始化 debugMode。

---

## CHANGE-005: 敌人子弹黄色，副子弹不含黄色
**状态**: 已实现
**方法**: 敌人子弹颜色改为 `#ffcc00`，SUB_COLORS 移除非蓝色系中的黄色。

---

## CHANGE-006: Bomb 持续时间内不能再次用 Bomb
**状态**: 已确认（原功能保留）
**方法**: useBomb 中已有 `if (game.isBombActive) return` 检查。

---

## CHANGE-007: 暂停时 PAUSED 叠加渲染
**状态**: 已实现
**方法**: gameLoop 渲染阶段绘制半透明遮罩 + "暂停" 文字。
