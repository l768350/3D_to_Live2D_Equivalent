[English](README.en.md) | 简体中文

# 3D → Inochi2D OBJ Deformer

> 本项目由个人使用 AI 辅助编程完成

用一个真实的 3D 参考模型（OBJ），自动为 [Inochi2D](https://inochi2d.com/) 木偶文件（`.inx`）生成转头/歪头（Turn / Roll）用的 deformer 笼子和参数绑定，省去在 Inochi Creator 里逐姿态手工摆点、凭经验估算深度的过程。

传统 Live2D/Inochi2D 建模里，转头形变通常靠美术手工估算五官该往前后挪多少来模拟深度。这个工具反过来：先给一个大致的 3D 模型作为"深度真值"，程序在模型表面采样、计算旋转后各点的真实投影坐标，直接写成 Inochi2D 认识的 deformer 网格 + Turn/Roll 参数绑定。贴图和五官精修仍然留给人在 Inochi Creator 里手动完成。

## 使用方法

先下载本仓库的 3D_to_Live2D_equivalent.zip 并解压。

### 桌面版（推荐）
```bash
pip install numpy pillow --break-system-packages
python3 app_tk.py
```
Linux 上如果提示缺 `tkinter`：`sudo apt install python3-tk`（或对应发行版包名）。
如果 python3 命令不存在，换成 python app_tk.py试试

### 网页版
```bash
pip install -r requirements.txt --break-system-packages
streamlit run app.py
```

两套 GUI 操作逻辑一致：导入 OBJ →（可选）导入已有 `.inx` 增量修改 →（可选）先修正模型初始朝向 → 逐个部件填名字/挂载点/参数设置 → 确认写入 → 保存 `.inx`。

命令行直接用核心工具：
```bash
python3 add_obj_deformer.py --obj model.obj --out result.inx
# 追加到已有文件、指定名字与父节点
python3 add_obj_deformer.py --obj model.obj --in existing.inx --out result.inx --name "Head" --parent "Neck"
```
完整参数见 `add_obj_deformer.py --help`。

Linux 上桌面版如果提示缺 `tkinter`：`sudo apt install python3-tk`（或对应发行版包名）。

### 使用小贴士

- 模型面数太多/文件太大可能导致程序崩溃
- 结果比例看起来不对，试着调整 --scale（GUI 里对应的参数）
- 模型有 UV island 的话，可以在右上角导入贴图后跑 full-topology-uv，理论上能把 3D 模型直接"写入"到 2D 上；但一个 OBJ 里如果有多个距离较远、零散的 UV island，可能会跑不出来
- 想做侧面/背面角度，调最下面的"笼子朝向"
- 选好 OBJ 后记得点"导入"刷新组件列表；一个 .obj 文件可以包含多个组件
- cylinder 方式建议自己调整 cols/rows，点数太少网格形状可能不对
- 输出前一定要点"浏览"选另存为路径、输入文件名确认、再点"保存到这个路径"，改动才会真正写入电脑文件；在这之前的所有操作都只是暂存在内存里，没有落盘

## 功能特点

- 三种笼子构筑方式（`--cage-method`）：`cylinder`（默认，绕 Y 轴柱面展开，覆盖模型 360°）、`full-topology`（直接复用模型自身网格）、`full-topology-uv`（复用网格+模型自带 UV，可搭配模型自身贴图）
- 支持按物体名/材质名/两者复合拆分模型为多个独立部件，互相允许重叠
- 支持往已有 `.inx` 增量插入新节点，不影响已有内容
- 提供 UV 连通性分析，用于诊断分组是否碎片化
- 两套本地 GUI（桌面版 tkinter / 网页版 Streamlit），也可以直接用命令行

## 项目结构

| 文件 | 作用 |
|---|---|
| `add_obj_deformer.py` | 核心 CLI：读取 OBJ，向 `.inx` 插入 deformer 节点与 Turn/Roll 绑定 |
| `obj_head_model.py` | OBJ 解析、射线求交、坐标旋转等底层几何计算 |
| `obj_pipeline.py` | 笼子构建、参数（keyframe）生成流程 |
| `node_builder.py` | Inochi2D 节点/UUID 构建辅助 |
| `inx_container.py` | `.inx`（JSON 容器）读写 |
| `gui_core.py` | GUI 与核心逻辑之间的胶水层，不改动核心文件行为 |
| `app_tk.py` | 桌面版 GUI（推荐） |
| `app.py` | 网页版 GUI（Streamlit） |
| `anchors.py` / `mesh_gen.py` | 旧版 pipeline 遗留的占位 stub，主流程不会用到 |

## 已知限制

- `full-topology-uv` 模式不生成 Roll 参数，需在 Inochi Creator 里手动补
- `full-topology`（正交投影）模式不支持 `--node-type part`
- 按 UV island 分组（`--group-by island`）尚未接入主流程，GUI 的"诊断"按钮可查看拆分结果，但暂不能直接生成节点

### 使用许可

欢迎随便使用本项目，包括但不限于直接使用、修改代码。
