---
name: cycling-power-calculator
description: 公路自行车功率↔速度换算。根据完整物理模型（风阻/重力/滚阻）计算骑行功率或速度，支持中英文。
metadata:
  openclaw:
    emoji: "🚴"
triggers:
  - pattern: "骑.*多快|多快.*骑|能骑|能跑多快"
    description: "询问骑行速度"
  - pattern: "需要.*功率|功率.*多少|几瓦|多少瓦|W.*骑|骑.*W"
    description: "询问骑行功率"
  - pattern: "功率.*速度|速度.*功率|功率换算|功率计算|骑行计算"
    description: "功率速度换算"
  - pattern: "cycling.*power|power.*cycling|watts.*km|km.*watts|watt.*speed|speed.*watt"
    description: "English cycling power query"
  - pattern: "CdA|FTP.*骑|骑.*FTP|爬坡.*功率|功率.*爬坡"
    description: "专业骑行参数"
  - pattern: "功率.*对照表|对照表.*功率|speed.*power.*table|power.*table"
    description: "生成功率速度对照表"
auto_invoke: true
examples:
  - "我以200W的功率能骑多快？5%上坡"
  - "骑30km/h平路需要多少功率？体重70kg"
  - "帮我生成一个功率-速度对照表，平路无风"
  - "What power do I need to ride 40 km/h into a 20 km/h headwind?"
  - "平路CdA 0.28，300W能跑多快"
  - "爬坡5%，FTP 250W，我能骑多快"
---

# 🚴 公路自行车功率-速度计算器

根据完整物理模型实时计算骑行功率或速度，结果含详细功率分解。

## 默认参数（用户未指定时使用）

| 参数 | 默认值 | 单位 | 说明 |
|------|--------|------|------|
| 人车合重 (m) | 66 | kg | 骑手 + 整车 |
| CdA | 0.300 | m² | 风阻系数 × 迎风面积 |
| 传动效率 (η) | 0.98 | — | 0–1 |
| 坡度 | 0.05 | — | 正数上坡，负数下坡（0.05 = 5%） |
| 风速 | 0 | km/h | 正数顶风，负数顺风 |
| 滚阻系数 Crr | 0.003 | — | |
| 气压 | 1020 | hPa | |
| 气温 | 30 | °C | |
| 相对湿度 | 0.80 | — | |

## 物理公式

### 空气密度

```
P_sat   = 6.1078 × 10^(7.5×T / (237.3+T)) × 10          [hPa]
P_water = RH × P_sat
P_dry   = P_atm − P_water
ρ       = (P_dry × 0.028964 + P_water × 0.018016)
          / (8.314 × (T + 273.15)) × 100                  [kg/m³]
```

### 功率分量（v 单位 m/s）

```
v_rel    = v + wind_speed_ms
P_aero   = 0.5 × ρ × CdA × v_rel² × v / η
P_gravity= m × 9.8067 × grade × v / η
P_rolling= Crr × m × 9.8067 × cos(arctan(grade)) × v / η
P_total  = P_aero + P_gravity + P_rolling
```

## 响应流程

### Step 1：解析请求

- **方向**：速度→功率，还是功率→速度
- **数值**：速度（km/h）或功率（W）
- **参数覆盖**：用户明确指定的参数（体重、CdA、坡度等）
- **语言**：与用户输入语言一致（中文或英文）

### Step 2：计算

**速度→功率**：速度 ÷ 3.6 转 m/s，代入公式直接计算。

**功率→速度**：P_total(v) 关于 v 单调递增，用二分法在 0–120 km/h 之间求解 P_total(v) = P_target。

### Step 3：输出格式

1. **主结果**加粗突出
2. **功率分解表**：
   - 风阻功率 P_aero：X W（XX%）
   - 重力功率 P_gravity：X W（XX%）
   - 滚阻功率 P_rolling：X W（XX%）
   - **合计**：X W
3. **使用参数**（标注哪些是覆盖值，哪些是默认值）
4. **空气密度** ρ = X kg/m³

### Step 4：补充背景（视情况）

- 坡度非零时，顺带给出平路同功率速度作对比
- 如用户提到 FTP，换算功率区间（Zone 1–5）
- 简短给出提升建议（减 CdA、减重、换低滚阻胎等）

## 场景比较

用户要求对比多场景时（平路 vs 爬坡、不同 CdA 等），输出对比表格，突出差值。

## 功率-速度对照表

用户要求生成表格时，默认输出 15–50 km/h 范围，步长 5 km/h（或按用户指定范围），列：速度 | P_total | P_aero | P_gravity | P_rolling。

## 注意事项

- 坡度默认 5% 上坡，若用户未指定需在结果中标注提醒
- 输出速度统一用 km/h，功率统一用 W（保留 1 位小数）
- 用户指定「人车合重」时直接用该值，**不要再额外加估算车重**
