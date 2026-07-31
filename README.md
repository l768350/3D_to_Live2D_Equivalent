# 3D → Inochi2D OBJ Deformer

用一个真实的 3D 参考模型（OBJ），自动为 [Inochi2D](https://inochi2d.com/) 木偶文件（`.inx`）生成转头/歪头（Turn / Roll）用的 deformer 笼子和参数绑定——省去在 Inochi Creator 里逐姿态手工摆点、纯靠经验估算深度的过程。

> 本项目由个人使用 AI 辅助编程完成，用于解决 Live2D 式 2D 建模中"转头形变"这一步过于依赖手工经验的问题。

## 这是做什么的

传统 Live2D/Inochi2D 建模里，"转头看起来自然"这个效果通常靠美术手工在若干个角度上摆放变形关键帧，凭经验估算五官在转头时该往前后挪多少（模拟深度）。

这个工具反过来：**先有一个大致的 3D 模型作为"深度的真值来源"**，程序在模型表面采样、计算旋转后各点的真实投影坐标，直接把结果写成 Inochi2D 认识的 deformer 网格 + Turn/Roll 参数绑定。美术只需要专注贴图和精修，不用再猜深度关系怎么摆。

- 输入：一个 OBJ 格式的 3D 参考模型（不需要是游戏级精度，够用来算相对位置关系即可）
- 输出：一个 `.inx` 文件（新建，或往已有 `.inx` 里追加节点），可以直接在 Inochi Creator 里打开
- 贴图绘制、五官精细摆放仍然留给人在 Inochi Creator 里手动完成——本工具只负责生成"笼子"和转动数学，不负责最终画面精修

## 功能特点

- 三种笼子构筑方式（`--cage-method`）：
  - `cylinder`（默认）：绕 Y 轴柱面展开，规整网格，覆盖模型 360° 表面（含背面）
  - `full-topology`：直接复用模型自身顶点/三角面当笼子
  - `full-topology-uv`：复用模型顶点/三角面，笼子静止布局使用模型自带 UV，可直接搭配模型自身贴图
- 支持按 Blender 物体名 / 材质名 / 两者复合拆分模型为多个独立部件（`--group-by`），互相之间不要求互斥、允许重叠
- 支持往已有 `.inx` 增量插入新节点，不影响已有内容
- 提供 UV 连通性分析，用于诊断一个分组内部是不是碎片化的
- 两套本地 GUI（桌面版 tkinter / 网页版 Streamlit），也可以直接用命令行

## 项目结构

| 文件 | 作用 |
|---|---|
| `add_obj_deformer.py` | 核心 CLI 工具：读取 OBJ，向 `.inx` 插入 deformer 节点与 Turn/Roll 绑定 |
| `obj_head_model.py` | OBJ 解析、射线求交、坐标旋转等底层几何计算 |
| `obj_pipeline.py` | 笼子构建、参数（keyframe）生成的具体流程 |
| `node_builder.py` | Inochi2D 节点/UUID 相关的构建辅助 |
| `inx_container.py` | `.inx` 文件（JSON 容器）的读写 |
| `gui_core.py` | GUI 与核心逻辑之间的胶水层，不改动任何核心文件的行为 |
| `app_tk.py` | 桌面版 GUI（tkinter，推荐先试这个） |
| `app.py` | 网页版 GUI（Streamlit） |
| `anchors.py` / `mesh_gen.py` | 旧版 pipeline 遗留的占位 stub，仅满足 import 依赖，当前主流程不会用到 |

## 安装

```bash
git clone <本仓库地址>
cd 3D_to_Live2D_equivalent
pip install -r requirements.txt --break-system-packages   # 网页版需要；桌面版只需 numpy+pillow
```

依赖：`numpy`、`pillow`；网页版额外需要 `streamlit`。桌面版的 `tkinter` 通常随 Python 自带，Linux 上如果提示缺失，用系统包管理器装：`sudo apt install python3-tk`（Debian/Ubuntu，其他发行版参考对应包名）。

## 使用方法

### 桌面版 GUI（推荐）
```bash
python3 app_tk.py
```
一个紧凑的原生窗口，选文件用系统对话框。

### 网页版 GUI
```bash
streamlit run app.py
```

两套界面操作逻辑一致：导入 OBJ →（可选）导入已有 `.inx` 增量修改 →（可选）先修正模型初始朝向 → 逐个部件填名字/挂载点/参数设置 → 确认写入 → 保存 `.inx`。

### 命令行
```bash
python3 add_obj_deformer.py --obj model.obj --out result.inx
# 追加到已有文件、指定名字与父节点
python3 add_obj_deformer.py --obj model.obj --in existing.inx --out result.inx \
    --name "Head" --parent "Neck"
```
完整参数说明见 `add_obj_deformer.py --help`。

## 已知限制

- `full-topology-uv` 模式不生成 Roll 参数（需要在 Inochi Creator 里手动补）
- `full-topology`（正交投影）模式不支持 `--node-type part`
- 按 UV island 分组（`--group-by island`）尚未接入主流程，GUI 的"诊断"按钮可以看到拆分结果，但目前不能直接从某个 island 生成节点

## License

（在此选择并填写你希望使用的开源许可证，例如 MIT / Apache-2.0，见下方"开源前检查清单"）
