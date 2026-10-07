# 样例案例：底线右手正手抽球（Forehand Groundstroke）

本案例是 `racket-video-lab` 的参考基准样本（Gold Standard Example），演示第一阶段 Pipeline 对单机位网球训练视频的分析产物与证据包结构。

---

## 1. 案例动态回放

![动作动图](swing.gif)

- **原始片段**：[`clip.mp4`](clip.mp4)（4 秒裁剪，H.264，约 320KB）
- **结构化元数据**：[`evidence.json`](evidence.json)

---

## 2. 五阶段逐帧证据切片

| 阶段 1：准备预判 (`phase1_ready.jpg`) | 阶段 2：侧身转肩 (`phase2_unit_turn.jpg`) | 阶段 3：下沉蓄力 (`phase3_drop_lag.jpg`) |
| :---: | :---: | :---: |
| ![准备](phase1_ready.jpg) | ![转肩](phase2_unit_turn.jpg) | ![下沉](phase3_drop_lag.jpg) |

| 阶段 4：触球瞬间 (`phase4_contact.jpg`) | 阶段 5：随挥收拍 (`phase5_finish.jpg`) |
| :---: | :---: |
| ![触球](phase4_contact.jpg) | ![随挥](phase5_finish.jpg) |

---

## 3. 诊断与力学数据示例

```json
{
  "shoulder_turn_angle_deg": 48.0,
  "optimal_shoulder_turn_range": [75.0, 90.0],
  "contact_point_rel_front_foot_cm": 0.0,
  "optimal_contact_point_range_cm": [20.0, 30.0],
  "head_stability_score": 0.95
}
```

### 核心诊断点：
1. **盯球稳定性极佳**：击球瞬间头颈锁定，视线专注。
2. **转肩不充分（48° vs 建议 75°~90°）**：动力链未贯通，呈手臂推球。
3. **击球点滞后（与前脚掌齐平）**：击球挤压，建议前移 20~30cm。
4. **随挥偏短**：缺乏向上刷球延伸，上旋弧线容错率低。
