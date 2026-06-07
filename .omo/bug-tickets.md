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
**状态**: 未修复 → 已修复 (2026-06-07)

### 现象
早苗的副子弹有跟踪效果，但设计要求是无跟踪的 V 字形扇形弹道。

### 根因（双重）
1. **首次修复不完整** (v10): `autoFire()` 中早苗分支已改为 V 字形固定角度发射，但 `updateBullets()` 对**所有**副子弹（`!b.isMain`）每帧覆盖 vx/vy 追踪最近敌人，V 字形弹道被立刻擦除。
2. 早苗子弹没有标记来区分"不可追踪"。

### 修复方法
1. 早苗子弹创建时添加 `noTracking: true` 字段
2. `updateBullets()` 中追踪逻辑跳过 `b.noTracking === true` 的子弹

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

## BUG-005: 大 boss 不出现
**状态**: 已修复

### 现象
分数到 2000 了 boss 还没出现。

### 根因
`big_boss` 类型在 ENEMY_TYPES 中 `prob: 0`，且之前的强制出场代码被移除了。随机生成永远选不到 boss。

### 修复方法
在 gameLoop 中添加强制出场逻辑：`score >= 1800` 时在屏幕正上方生成 boss。

---

## CHANGE-008: 敌人只在上半屏活动
**状态**: 已实现
**方法**: updateEnemies 改为横向巡逻 + 不追玩家。createEnemy 生成位置限制在上半屏。

## CHANGE-009: 毛玉和蝴蝶子弹只往下
**状态**: 已实现
**方法**: updateEnemies 中 fuzzy/butterfly 的射击角度固定为 `Math.PI/2`（向下）。

## CHANGE-010: 灵梦 bomb 圆形范围伤害
**状态**: 已实现
**方法**: updateBombEffect 中 reimu 分支改为自身周围半径 200 圆形范围，每秒 1 伤害，持续 7 秒。

## CHANGE-011: Bomb 清除敌人子弹
**状态**: 已实现
**方法**: useBomb 启动时 + updateBombEffect 每秒持续清除范围内的 enemyBullets。

## CHANGE-012: 玩家出生到底部
**状态**: 已实现
**方法**: createPlayer 中 y 改为 `CANVAS_H - 60`。

## CHANGE-013: 取消 power 特效
**状态**: 已实现
**方法**: 删除 onEnemyKilled 中的 `createExplosion('#ffcc44', 6)`。

---

## BUG-006: 敌人从左右侧生成后停留在屏幕外
**报告日期**: 2026-06-07
**状态**: 已修复

### 现象
大部分敌人没有出现在屏幕里，而是在屏幕外面。只有从上方生成的敌人可见。

### 根因
`createEnemy()` 从左右两侧生成时 x 坐标在屏幕外（`x = -radius-5` 或 `x = CANVAS_W+radius+5`），y 在上半屏内。`updateEnemies()` 中下落阶段（y < upperBound）只做 `e.y += e.speed`，不移动 x。敌人在整个下落过程（数秒）中 x 始终在屏幕外，完全不可见。

此外，`updateEnemies()` 中有重复的 `e.y += e.speed`（第 919 行和第 934 行），导致下落速度翻倍。

### 修复方法
1. 下落阶段也朝屏幕中心横向移动：`e.x += (canvasCenterX - e.x) * 0.02 * e.speed`
2. 修复重复的 `e.y += e.speed`，合并为单一下落逻辑

---

## BUG-007: 小 boss (mini_boss) 永不出场
**报告日期**: 2026-06-07
**状态**: 已修复

### 现象
分数超过 1500 后小 boss 不出现。

### 根因
`ENEMY_TYPES.mini_boss.prob = 0`，随机生成永远选不到小 boss。大 boss 有独立的强制出场逻辑（score>=1800），小 boss 没有。

### 修复方法
`mini_boss.prob` 从 `0` 改为 `0.05`。

---

## CHANGE-014: 移除 +0.1 POWER 浮动提示
**状态**: 已实现
**方法**: `onEnemyKilled()` 中删除 `bonusMessages.push({ text: '+0.1 POWER' })`，保留 power 增加和音效。

## CHANGE-015: Bomb 视觉改善 — 移除全屏白屏 + 灵梦显示范围圈
**状态**: 已实现
**方法**:
- 移除 `drawBombEffect()` 中 300ms 全屏白色遮罩
- 移除无意义的扩散光环
- 灵梦 bomb 显示脉冲圆圈（半径 200），半透明填充 + 橙色描边
- 魔理沙、早苗范围显示不变
- BOMB! 文字颜色改为金色

## CHANGE-016: 新增难度选择器 (Easy / Normal / Hard / Lunatic)
**状态**: 已实现
**方法**:
- HTML 新增 `<select id="diffSelect">` 四种选项
- `game.difficulty` 字段，默认 `'normal'`
- `getCurrentSpawnInterval()` 按难度系数调整：Easy×2.0 / Normal×1.0 / Hard×0.6 / Lunatic×0.4

## CHANGE-017: 重构敌人射击逻辑（按类型不同）
**状态**: 已实现
**方法**:
- 毛玉(fuzzy): 120帧间隔，1弹，只向下
- 蝴蝶(butterfly): 120帧间隔，1弹，50%向下/50%瞄准玩家
- 乌鸦(crow): 80帧间隔（1.5×毛玉），1弹，瞄准玩家附近带±0.25随机偏移
- 小boss(mini_boss): 40帧间隔（3×毛玉），1弹，瞄准玩家附近带随机偏移
- 大boss(big_boss): 80帧间隔，同上

---

## BUG-008: 灵梦副子弹半路消失
**报告日期**: 2026-06-07
**状态**: 已修复

### 现象
灵梦副子弹从底部发射后，打不到上半屏的敌人就在半路消失。早苗的副子弹没有这个问题。

### 根因
灵梦副子弹速度 2.5px/帧（SUB_BULLET_SPEED），life 120 帧 → 最大射程 2.5 × 120 = 300px。玩家在 y≈540，子弹到 y=240 就消失，而上半屏敌人巡逻区在 y=0~300。
早苗子弹速度 5px/帧（CONFIG.BULLET_SPEED）且 noTracking 不降速 → 射程 5×120=600px，足够穿过全屏。

### 修复方法
灵梦副子弹 `life` 从 `CONFIG.BULLET_LIFE`(120) 改为 `CONFIG.BULLET_LIFE * 2`(240)，射程 2.5×240=600px。

## CHANGE-018: 大boss射击与小boss区分
**状态**: 已实现
**方法**: 大boss 改为 3 发扇形弹幕（相邻间隔 0.25 弧度），小boss 保持 1 发。

## CHANGE-019: 敌人下落时分散到随机位置
**状态**: 已实现
**方法**: `createEnemy()` 新增 `targetX` 字段（randInt(60, W-60)），`updateEnemies()` 下落阶段趋近 `targetX` 而非屏幕中心，避免敌人全聚集中间区域导致弹幕密集。

## CHANGE-020: Bomb 伤害频率改为 0.2s（0.2hp/次）+ 弹幕清除同步
**状态**: 已实现（频率从 0.1s 调整为 0.2s）
**方法**:
- `updateBombEffect` 中 `bombDamageTimer` 间隔从 1000ms → 100ms → 200ms
- 每次伤害从 `e.hp -= 1` → `e.hp -= 0.1` → `e.hp -= 0.2`
- 敌人子弹清除也从每 1s 改为每 0.1s（灵梦/魔理沙范围内，早苗全屏）

## CHANGE-021: 魔理沙激光保持常亮 + 伤害修正
**状态**: 已实现
**方法**:
- `drawLasers()` alpha 改为常量 0.8（不再随 life 渐变，解决闪烁）
- 碰撞检测从每帧 60 点逐点判定改为每敌人一次点-线段距离判定，避免叠加
- 每次伤害从 `damage * 0.1` 改为 `damage * 0.05`（约 3 DPS，不再秒杀 boss）
- 激光 life 从 30 → 70

## CHANGE-023: 大boss 1000分出现 + 速度与毛玉相同 + 得分≥2000也通关
**状态**: 已实现
**方法**:
- `getAvailableEnemyTypes()` 和强制出场：分数阈值 1800 → 1000
- `ENEMY_TYPES.big_boss.speedMult`: 0.2 → 1.0
- `updateStateMachine()`: 新增 `score >= 2000` → CLEAR（与杀boss并存）

## CHANGE-022: 灵梦 bomb 期间子弹不再翻倍
**状态**: 已实现
**方法**: 移除 `autoFire()` 中灵梦 bomb 期间副子弹 ×3 和额外 2 发主子弹的逻辑。
