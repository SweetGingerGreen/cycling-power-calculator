# 🚴 Cycling Power Calculator — Claude Skill

一个基于完整物理模型的公路自行车**功率↔速度换算** Claude Skill，支持中英文输入。

> 灵感来源于本人调校的 Excel 功率计算模型，参数经过实际骑行数据验证。

---

## 功能

- **速度 → 功率**：计算在特定速度下需要输出多少瓦
- **功率 → 速度**：计算在特定功率下能骑多快
- **功率分解**：风阻 / 重力（爬坡）/ 滚阻 各占多少
- **场景对比**：平路 vs 爬坡、不同 CdA、不同体重等
- **功率-速度对照表**：一键生成全区间表格
- **精确空气密度**：根据气温、气压、湿度实时计算（不是简单用 1.225）

### 默认参数（可全部覆盖）

| 参数 | 默认值 | 说明 |
|------|--------|------|
| 人车合重 | 66 kg | 骑手 + 整车 |
| CdA | 0.300 m² | 风阻系数 × 迎风面积 |
| 传动效率 | 98% | 链条传动损耗 |
| 坡度 | 5%（上坡） | 正数上坡，负数下坡 |
| 风速 | 0 km/h | 正数顶风，负数顺风 |
| 滚阻系数 Crr | 0.003 | 公路胎典型值 |
| 气压 | 1020 hPa | |
| 气温 | 30 °C | |
| 相对湿度 | 80% | |

---

## 示例对话

```
你：我以200W的功率能骑多快？5%上坡，体重66kg

Claude：以 200W 功率在 5% 上坡骑行，速度约为 18.2 km/h
  风阻功率：22.9 W（11.5%）
  重力功率：166.9 W（83.6%）  ← 爬坡主导
  滚阻功率：10.0 W（5.0%）
  空气密度：ρ = 1.157 kg/m³（30°C/1020hPa/80%RH）
  对比：同功率平路可达 ~36.5 km/h
```

```
你：帮我生成一个功率-速度对照表，平路无风，体重75kg，CdA 0.32

Claude：
  速度 (km/h) | 总功率 (W) | 风阻 (W) | 滚阻 (W)
  25          |   78.9    |   63.3  |  15.6
  30          |  128.2    |  109.5  |  18.8
  35          |  195.9    |  174.0  |  21.9
  40          |  284.4    |  259.4  |  25.0
  45          |  397.4    |  369.3  |  28.1
```

---

## 安装方法

### 方法一：Claude.ai（网页版 / 手机 App）

1. 下载 [`cycling-power-calculator.skill`](https://github.com/SweetGingerGreen/cycling-power-calculator/releases) 文件
2. 打开 [Claude.ai](https://claude.ai)
3. 在对话界面点击 **附件图标** → 选择 `.skill` 文件上传
4. 上传后即可直接使用，对话中提到骑行功率/速度相关问题会自动触发

### 方法二：Claude Code（命令行）

**前提**：已安装 [Claude Code](https://docs.anthropic.com/claude-code)

```bash
# 克隆本仓库
git clone https://github.com/SweetGingerGreen/cycling-power-calculator.git

# 安装 skill（将 SKILL.md 复制到 Claude Code 的 skills 目录）
claude skill install ./cycling-power-calculator/SKILL.md
```

或者直接使用 `.skill` 文件：

```bash
claude skill install cycling-power-calculator.skill
```

安装后，在 Claude Code 的任意对话中提到骑行功率计算，Skill 会自动被调用。

---

## 物理模型

### 空气密度（考虑温湿度）

```
P_sat   = 6.1078 × 10^(7.5T / (237.3+T)) × 10   [hPa]
P_water = RH × P_sat
P_dry   = P_atm − P_water
ρ       = (P_dry × 0.028964 + P_water × 0.018016) / (8.314 × (T+273.15)) × 100
```

### 功率分量

```
P_aero    = 0.5 × ρ × CdA × (v + v_wind)² × v / η
P_gravity = m × 9.8067 × grade × v / η
P_rolling = Crr × m × 9.8067 × cos(arctan(grade)) × v / η
P_total   = P_aero + P_gravity + P_rolling
```

功率→速度方向使用二分法数值求解。

---

## License

MIT
