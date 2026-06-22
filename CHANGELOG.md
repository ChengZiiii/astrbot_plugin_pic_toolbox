# CHANGELOG

## [v1.1.3] — 2026-06-22

### ✨ 新增

- **二次元人脸检测优化**：`发射` 指令引入 LBP 动漫人脸级联分类器（`lbpcascade_animeface.xml`，来自 nagadomi/lbpcascade_animeface），专为日系二次元脸型优化训练。配合直方图均衡化（`equalizeHist`）改善线稿对比度，大幅提升动漫头像的检测准确率。
- **三级检测策略**：LBP 动漫级联（首选）→ Haar 正面级联（回退）→ Haar 侧脸级联（回退），每一层尝试多组参数（宽松 → 严格），自动择优取面积最大的检测框，确保动漫 + 真人头像均能正确检测。

### 🔧 影响范围

`meme/shoot.py` — `_detect_face_focal()` 完全重写，新增 `resource/lbpcascade_animeface.xml` 级联文件（~241KB）。其他指令不受影响。

### 🙏 鸣谢

- [nagadomi/lbpcascade_animeface](https://github.com/nagadomi/lbpcascade_animeface) — LBP 动漫人脸级联分类器

---

## [v1.1.2] — 2026-06-16

### 🐛 修复

- **GIF 调速后颜色偏移**：`save_rgba_gif()` 对所有操作重新量化生成新调色板，导致 GIF 主体颜色偏移。修复为：调速/翻转/对称等不改变像素颜色的操作直接复用原 GIF 调色板；仅反色等变色操作回退到重量化。
- **GIF 动画静止**：修正 GCE (Graphic Control Extension) 二进制写入的 struct 格式（`<BBHB` → `<BHBB>`）。原格式将 delay（2 字节）与 transparent index（1 字节）的字节数写反，导致 `trans_idx=255` 时每帧延迟 ~653 秒 → GIF 看似静止。

### 🔧 影响范围

`save_rgba_gif()` 新增 `source_palette` / `source_trans_idx` 可选参数，所有调用方（`gif_speed` / `flip` / `mirror`）在 `unfold_frames` 前捕获原调色板后传入。

---

## [v1.1.1] — 2026-06-16

### 🐛 修复

- **透明 GIF 背景变白**：修复 `unfold_frames()` 在 GIF 存在 `transparency` 索引但 `background` 索引不同时，`_get_background_rgba()` 错误返回不透明调色板颜色的问题。`disposal=2` 帧的背景从此正确使用透明色，后续帧不再丢失 alpha 通道。
- **`save_rgba_gif()` 透明索引**：改用 `colors=255` 量化策略，让 Pillow 自动分配透明索引（替代之前的手动 `colors=254` + 强制 `trans_idx=255`），调色板映射更可靠。
- **临时文件清理**：每次请求使用 `uuid` 生成唯一文件名，避免并发请求共享 PID 文件名导致的竞态条件。清理闭包显式捕获路径变量，确保 `asyncio.ensure_future` 延迟清理可靠执行。

### 📦 依赖

- `requirements.txt` 补充 `numpy>=1.24.0` 和 `opencv-python-headless>=4.8.0`（之前为隐式依赖）。

### 🔧 影响范围

透明 GIF 修复使以下指令受益：调速、反色、左右翻转、上下翻转、左/右/上/下对称。

---

## [v1.1.0] — 2026-06-15

### ✨ 新增

- **GIF 调速自动回退**：当调速倍率使帧率超过 50 FPS 上限时，自动计算并回退到不超过上限的最接近倍率（保留一位小数），同时以文字提示告知用户实际使用的倍率与回退原因。
- **GIF 调速丢帧模式**：新增配置项 `gif_speed_allow_frame_drop`，开启后调速超限时不再回退倍率，而是通过均匀丢帧（每隔 N 帧取 1 帧）保证单帧间隔 ≥ 20ms，在遵守 50 FPS 上限的同时实现目标倍率。

### 🔧 配置变更

| 配置项 | 类型 | 默认值 | 说明 |
| :--- | :--- | :--- | :--- |
| `gif_speed_allow_frame_drop` | `bool` | `false` | 调速超 50 FPS 时是否允许均匀丢帧实现目标倍率 |

### 📝 文档

- README 新增「⚡ GIF 调速详解」小节，包含两种策略对比表。
- 新增 `CHANGELOG.md`（本文件）。

---

## [v1.0.0] — 初始发布

- 反色、左右翻转、上下翻转
- 左/右/上/下对称
- GIF 调速（0.3× ~ 5.0×，50 FPS 硬上限）
- 摸头（Petpet）、发射（人脸检测）、撅人（双人头）、鞭笞（双人头）、砍头表情包
- `@用户` 取头像处理、精准匹配模式
- 统一 GIF 增量帧展开管道
- 启动残留文件自动清理
