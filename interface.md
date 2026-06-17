
## 阶段

```rs
enum Stage {
	Day,
	Night,
}

impl Stage {
	fn switch(&mut self) -> &Self {
		todo!();
	}
}
```

## 角色

```rs
struct Role<> {
    name: &str,          // 名字
    health: usize,       // 生命值
    life_steal: usize,   // 生命窃取
    damage: usize,       // 伤害
    attack_speed: usize, // 攻速
    crit_rate: usize,    // 暴击率（百分比概率）
    crit_damage: usize,  // 暴击伤害（百分比加成）
    armor: usize,        // 护甲
    dodge: usize,        // 闪避（百分比概率）
    speed: usize,        // 移速
    luck: usize,         // 幸运
}
```

```rs
trait Vampire {
	fn stamina(&self) -> usize; // 精力
	fn set_stamina(&self, stamina: usize) -> &Self;
}
```

```rs
trait Human {
}
```

```rs
trait Monster {
}
```

```rs
// 逃跑，逃跑血量阈值等在 `config` 配置
trait Flee {
	fn flee(&self, config: FleeConfig);
	fn is_flee(&self) -> bool;
}
```

## 弹幕

```rs
enum State {
	Created,
	Spawned,
	Updating,
	Deleted,
}
```

```rs
trait Bullet {
	fn on_created、on_spawned、on_updating、on_deleted;
	fn self_field_getter_and_setter;
	fn move(&mut self, impl MoveCtx, t, x, y, z) -> ...;
	fn effect(&mut self, &mut impl EffectEntity)-> ...;
}
```

```rs
// maybe
struct {
	init_pos: Vec,
	shape: Shape,
	t_warning: Duration,
	t_life: Duration,
	state: State,
}
```

以下无序列表代表顺序无关，有序列表代表按照顺序执行

- 当 Bullet 处于 `State::Created` 时
	0. 调用函数/触发事件 `on_created` (once)
	1. Bullet 状态转移为 `State::Spawned`
- 当 Bullet 处于 `State::Spawned` 时
	0. 调用函数/触发事件 `on_spawned` (once)
	1. 在位置 init_pos 出现 Bullet 刷新预警（提示玩家），timer_warning 计时
	2. 经过时间 t_warning 后，Bullet 在位置 init_pos 生成
	3. Bullet 状态转移为 `State::updating`，timer_life 计时
- 当 Bullet 处于 `State::Updating` 时
	- 调用函数/触发事件 `on_updating` (every tick)
	- Bullet 按照规律 move 增量移动
	- Bullet 碰撞 Entity 后造成效果 effect
	- Bullet 持续时间 t_life 后，状态转移为 `State::Deleted`
- 当 Bullet 处于 `State::Deleted` 时
	1. 调用函数/触发事件 `on_deleted` (once)
	2. Bullet 在当前位置消失（在实现上进入对象池回收或销毁）

Bullet 形状：点状、条状、...

t_warning 为 0 即代表 Bullet 无预警出现
规律 move 恒定为 0 代表 Bullet 永远在位置 init_pos 静止

effect(&mut self, &mut 墙体) 可以将 self 状态转移为 `State::Deleted`，即碰墙消失
effect(&mut self, &mut !Self) 可以将 self 状态转移为 `State::Deleted`，即单体伤害
effect(&mut self, &mut !Self) 可以将 self 状态转移为 `State::Updating`，即穿透伤害
effect(&mut self, &mut !Self) 应当对 !Self 造成伤害 damage
effect(&mut self, &mut Self) 可以对 Self 造成伤害 damage，即友伤

武器的 Bullet，t_warning 应当恒为 0 其中：

近战武器的 Bullet

- 形状为不定状（枪弹，冰锥，...）
- init_pos 恒为所有者位置（些许偏移）
- t_life 恒为无穷大

远程武器的 Bullet

- 形状为不定状（枪弹，冰锥，...）
- init_pos 恒为所有者位置（些许偏移）
- 根据敌人位置的 ctx 应用 move

光环武器的 Bullet

- 形状为大型点状
- init_pos 恒为所有者位置（些许偏移）
- t_life 恒为无穷大

轨道武器的 Bullet

- 形状为不定状（镰刀，法球，...）
- init_pos 恒为所有者位置（些许偏移）距离半径处

轨道武器的 Bullet 在实现上，可以 every tick 回收再利用，或者根据武器所有者位置的 ctx 应用 move，此时 t_life 可以恒为无穷大
