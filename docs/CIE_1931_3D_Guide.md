# 🚀 CIE 1931 三维色域实体模型构建与 Web 交互全栈指南

欢迎来到 **Open-Color-Science** 项目！本教程记录了一个从零构建 CIE 1931 三维色域实体模型的完整全栈探索过程。我们不仅推导了色彩学的底层数学逻辑，还真实记录了在 3D 建模、格式兼容、网页交互中踩过的经典大坑与修复方案，旨在打造一个开箱即用、通俗易懂的色彩科学入门宝库。

---

## 📖 目录
* [1. 色彩科学的生物与物理源头](#1-色彩科学的生物与物理源头)
* [2. 数学洗牌：从 RGB 到 XYZ 空间的诞生](#2-数学洗牌从-rgb-到-xyz-空间的诞生)
* [3. 降维打击与重新升维：真正的“三维陀螺”](#3-降维打击与重新升维真正的三维陀螺)
* [4. 实战血泪史：3D 渲染的四大经典“翻车”排查](#4-实战血泪史3d-渲染的四大经典翻车排查)
* [5. 核心源码：纯 Python 导出彩色 glTF 陀螺几何体](#5-核心源码纯-python-导出彩色-gltf-陀螺几何体)
* [6. 前端进阶：打造带调色与下载功能的高级交互控制台](#6-前端进阶打造带调色与下载功能的高级交互控制台)

---

## 1. 色彩科学的生物与物理源头

### 1.1 人眼是“RGB接收器”
自然界中本没有“颜色”，只有不同波长的电磁波。人类之所以能看到彩色，是因为人眼视网膜上存在 3 种感光视锥细胞：
* **L视锥细胞**：对长波长（红光附近）最敏感。
* **M视锥细胞**：对中波长（绿光附近）最敏感。
* **S视锥细胞**：对短波长（蓝光附近）最敏感。

人类大脑将这 3 种细胞接收到的信号组合编码，才形成了丰富的主观色彩世界。这就从生物学上决定了：**任何量化人眼视觉色彩的数学模型，其物理维度必须是“三维”的。**

### 1.2 1920年代的硬核测试：颜色匹配实验
为了将主观视觉转化为客观数学，科学家莱特（Wright）和吉尔德（Guild）通过精密的物理光学实验，让测试者旋转旋钮，用固定波长的**红（700nm）、绿（546.1nm）、蓝（435.8nm）**三种基色光，去拼命混合匹配另一侧的纯粹单波长光谱色。

**物理学界的“负数”危机**：实验发现，在匹配某些极度鲜艳的青绿色光时，无论怎么加大红绿蓝的亮度，肉眼的饱和度总是赶不上光谱色。科学家被迫想出了一个绝招：*把少量的红光移动到目标光谱色那一侧！*
* 数学方程等同于：`目标色 = - 红光 + 绿光 + 蓝光`。
这就导致在早期的 CIE RGB 颜色匹配函数中，红光的份额在某些波长段出现了**负数**，这给工程计算带来了极大的不便。

---

## 2. 数学洗牌：从 RGB 到 XYZ 空间的诞生

为了消灭工程计算中的负数，国际照明委员会（CIE）在 1931 年进行了一次人类色彩史上最重要的**数学坐标系旋转（线性变换）**。他们虚构了三个在物理上不存在的理想基色，命名为 **$X, Y, Z$（原色刺激值）**。

这次数学洗牌完美达成了三个目的：
1. **全面消灭负数**：新坐标系完美包裹了整个人眼可见光范围，所有变换数值全部为正。
2. **明度（亮度）独立**：特意调整矩阵系数，让 **$Y$ 函数正好等同于人眼的视见函数（明度响应曲线）**。这意味着，$Y$ 从此只负责描述物体的明暗亮度，而 $X$ 和 $Z$ 组合负责描述色彩。
3. **标准化归一**：当 $X=Y=Z$ 时，刚好对应没有任何色彩倾向的等能白点（白平衡原点）。

---

## 3. 降维打击与重新升维：真正的“三维陀螺”

### 3.1 降维：经典的平面马蹄图
为了在普通的二维白纸或屏幕上研究颜色，科学家对三维的 $X,Y,Z$ 进行了归一化降维处理：
$$x = \frac{X}{X+Y+Z}, \quad y = \frac{Y}{X+Y+Z}$$
因为 $x + y + z = 1$，知道 $x, y$ 就能推导出 $z$。将自然界 380nm 到 780nm 所有单波长光谱色的 $(x, y)$ 坐标在直角坐标系中连成线，它就自然而然地弯曲成了经典的**马蹄形（扇贝形）外廓**。

### 3.2 升维：xyY 空间里的“彩色陀螺”
如果我们把剔除出的亮度 $Y$ 作为垂直的高度轴加回来，就形成了真正的 **CIE xyY 三维色域实体模型**。它的形态是一个“两头尖、中间宽”的不规则双锥体（陀螺状）：
* **底部坍缩（黑点，$Y=0$）**：当没有光线时，无论 $(x,y)$ 坐标如何改变，绝对刺激值 $X,Y,Z$ 统统归零。物理上，这是绝对的黑暗（黑洞）。
* **中部最胖（中等亮度，$Y=0.5$ 左右）**：此时感光细胞处于最舒适响应区，显示器能发出相对最大的纯色能量，色彩饱和度达到极致，马蹄形截面展得最开。
* **顶部收缩（白点，$Y=1.0$）**：强光能量过载会导致所有色彩发生“泛白”现象。在最高明度下，只有等能白点 $(0.33, 0.33)$ 能够生存，因此模型顶部剧烈收缩汇聚于一个白点。

---

## 4. 实战血泪史：3D 渲染的四大经典“翻车”排查

在利用 Python 代码编写并导出这个三维模型的过程中，我们经历了数次经典的几何与工程报错，以下是完整的排查逻辑：

### 💥 大坑一：3D查看器打开后一片空白，什么也找不到
* **根本原因**：
  1. 原始的 CIE $(x, y)$ 坐标数值在 $0 \sim 1$ 之间，模型物理尺寸极其微小，被 3D 查看器的默认摄像机裁剪裁剪掉了。
  2. 轴向位置错误，模型可能被埋在了地平线以下。
* **修复方案**：在 Python 导出顶点时，将 $x, y, z$ 坐标等比例**放大 10 到 100 倍**。

### 💥 大坑二：模型可以加载，但“几何退化”被压扁成了一片纸
* **根本原因**：在构建切片层时，底面和顶面的顶点逻辑发生了重合，代码没有为不同的亮度 $Y$ 赋予真实的高度纵坐标，导致三维物体在高度轴上坍缩成了 2D 平面。
* **修复方案**：引入循环层机制，让纵坐标 $Z_{\text{mesh}} = \text{layer\_ratio} \times \text{total\_height}$，确保每一层切片在物理空间中有真正的高度差。

### 💥 大坑三：导出的点云（Point Cloud）模型无法加载或提示损坏
* **根本原因**：第一版代码直接导出了零散的顶点属性。然而 Windows 3D 查看器等主流渲染引擎的底层机制**默认只渲染由“面（Faces）”组成的密闭几何网格**。没有面，它就会判定文件是空的。
* **修复方案**：采用 **“切片叠层法”**。通过算法，用成百上千个小三角面（Faces）按顺时针/逆时针的拓扑顺序，将相邻两层之间的顶点严丝合缝地“编织”成密闭实体。

### 💥 大坑四：模型成功显示成了陀螺，但表面全是死气沉沉的灰色
* **根本原因**：
  1. 采用了 `.stl` 格式：该格式纯粹记录几何三角片，在物理标准上根本不支持颜色。
  2. 采用了 `.ply` 格式：虽然写入了顶点色，但 Windows 3D 查看器默认会强制套用其自带的灰色塑料默认材质，把顶点颜色冲刷掉了。
* **修复方案**：全面升级为现代 Web3D 工业标准格式 —— **`.gltf` / `.glb`**。在代码中显式声明 `"COLOR_0"` 顶点属性通道，并显式配置 PBR 材质数据，强制渲染引擎读取色彩。

---

## 5. 核心源码：纯 Python 导出彩色 glTF 陀螺几何体

这段 Python 脚本不需要你安装任何复杂的 3D 图形库，纯靠底层数学计算和 Base64 编码，运行后会直接在当前目录下生成标准的 `CIE_True_3D_Gamut.gltf` 文件。

```python
import numpy as np
import json
import base64

def xy_to_rgb(x, y, brightness):
    """
    色彩科学核心公式：将 CIE xy 坐标与亮度结合，转换为标准的 sRGB 颜色
    """
    z = 1.0 - x - y
    z = max(0.0, z)
    
    # 计算绝对刺激值 XYZ
    X = x * brightness
    Y = brightness
    Z = z * brightness
    
    # XYZ -> sRGB 线性转换矩阵
    r =  3.2406 * X - 1.5372 * Y - 0.4986 * Z
    g = -0.9689 * X + 1.8758 * Y + 0.0415 * Z
    b = -0.0557 * X - 0.2040 * Y + 1.0570 * Z
    
    # 范围裁剪与 Gamma 2.2 校正
    return [max(0.0, min(1.0, c)) ** (1/2.2) for c in [r, g, b]]

def generate_cie_3d_gltf():
    # 基础的 CIE 1931 马蹄形边界采样点 (x, y)
    boundary_points = [
        (0.17, 0.01), (0.15, 0.06), (0.12, 0.13), (0.08, 0.23), (0.05, 0.35),
        (0.03, 0.49), (0.07, 0.60), (0.12, 0.70), (0.21, 0.77), (0.30, 0.80),
        (0.40, 0.77), (0.50, 0.70), (0.60, 0.60), (0.67, 0.50), (0.71, 0.40),
        (0.73, 0.30), (0.70, 0.20), (0.60, 0.10), (0.40, 0.03), (0.25, 0.01)
    ]
    num_pts = len(boundary_points)
    
    scale = 10.0          # 放大倍数
    total_height = 12.0   # 模型总高度
    layers = 20           # 纵向切片层数

    vertices = []
    colors = []
    faces = []
    center_x, center_y = 0.33, 0.33 # E点白平衡中心轴

    print("正在计算三维立体色域几何与色彩网格...")

    # 逐层生成顶点（实现双锥体陀螺形态）
    for layer in range(layers + 1):
        h_ratio = layer / layers  
        z_val = h_ratio * total_height
        
        # 核心变形算法：下半部分展开，上半部分收缩
        if h_ratio < 0.5:
            layer_scale = h_ratio * 2.0
        else:
            layer_scale = (1.0 - h_ratio) * 2.0
        layer_scale = max(0.02, layer_scale) 

        # 写入中心轴点
        vertices.append([center_x * scale, center_y * scale, z_val])
        colors.append(xy_to_rgb(center_x, center_y, h_ratio) + [1.0]) 
        
        # 写入马蹄形外壳边界点
        for pt in boundary_points:
            x_shrunk = center_x + (pt[0] - center_x) * layer_scale
            y_shrunk = center_y + (pt[1] - center_y) * layer_scale
            vertices.append([x_shrunk * scale, y_shrunk * scale, z_val])
            colors.append(xy_to_rgb(pt[0], pt[1], h_ratio) + [1.0])

    # 构建三角面（编织侧面网格）
    points_per_layer = num_pts + 1
    for layer in range(layers):
        start_curr = layer * points_per_layer
        start_next = (layer + 1) * points_per_layer
        
        for i in range(1, num_pts):
            b1, b2 = start_curr + i, start_curr + i + 1
            t1, t2 = start_next + i, start_next + i + 1
            faces.extend([b1, b2, t2, b1, t2, t1])
            
        # 闭合最后一扇缝合面
        b1, b2 = start_curr + num_pts, start_curr + 1
        t1, t2 = start_next + num_pts, start_next + 1
        faces.extend([b1, b2, t2, b1, t2, t1])

    # 封闭底面盖子
    for i in range(1, num_pts):
        faces.extend([0, i + 1, i])
    faces.extend([0, 1, num_pts])
    
    # 封闭顶面盖子
    top_center_idx = layers * points_per_layer
    for i in range(1, num_pts):
        faces.extend([top_center_idx, top_center_idx + i, top_center_idx + i + 1])
    faces.extend([top_center_idx, top_center_idx + num_pts, top_center_idx + 1])

    # 数据封装与 Base64 编码
    v_np = np.array(vertices, dtype=np.float32)
    c_np = np.array(colors, dtype=np.float32)
    f_np = np.array(faces, dtype=np.uint16)

    v_bytes, c_bytes, f_bytes = v_np.tobytes(), c_np.tobytes(), f_np.tobytes()
    def align_bytes(b): return b + b'\x00' * ((4 - len(b) % 4) % 4)
    v_bytes, c_bytes, f_bytes = align_bytes(v_bytes), align_bytes(c_bytes), align_bytes(f_bytes)

    total_bytes = f_bytes + v_bytes + c_bytes
    v_offset = len(f_bytes)
    c_offset = v_offset + len(v_bytes)

    b64_uri = "data:application/octet-stream;base64," + base64.b64encode(total_bytes).decode('utf-8')

    # 组装符合 glTF 2.0 规范的 JSON
    gltf = {
        "asset": {"version": "2.0"},
        "scenes": [{"nodes": [0]}],
        "nodes": [{"mesh": 0, "rotation": [-0.707, 0, 0, 0.707]}], # 自动扶正模型
        "meshes": [{
            "primitives": [{
                "attributes": {"POSITION": 1, "COLOR_0": 2}, # 核心绑定
                "indices": 0,
                "material": 0
            }]
        }],
        "materials": [{
            "pbrMetallicRoughness": {"baseColorFactor": [1, 1, 1, 1], "metallicFactor": 0.0, "roughnessFactor": 0.5},
            "doubleSided": True
        }],
        "buffers": [{"byteLength": len(total_bytes), "uri": b64_uri}],
        "bufferViews": [
            {"buffer": 0, "byteOffset": 0, "byteLength": len(f_bytes), "target": 34963},
            {"buffer": 0, "byteOffset": v_offset, "byteLength": len(v_bytes), "target": 34962},
            {"buffer": 0, "byteOffset": c_offset, "byteLength": len(c_bytes), "target": 34962}
        ],
        "accessors": [
            {"bufferView": 0, "byteOffset": 0, "componentType": 5123, "count": len(faces), "type": "SCALAR"},
            {"bufferView": 1, "byteOffset": 0, "componentType": 5126, "count": len(vertices), "type": "VEC3", "max": v_np.max(axis=0).tolist(), "min": v_np.min(axis=0).tolist()},
            {"bufferView": 2, "byteOffset": 0, "componentType": 5126, "count": len(colors), "type": "VEC4"}
        ]
    }

    with open("CIE_True_3D_Gamut.gltf", 'w') as f:
        json.dump(gltf, f, indent=2)
    print("\n[成功] 立体『彩色陀螺』模型已成功导出为: CIE_True_3D_Gamut.gltf")

if __name__ == "__main__":
    generate_cie_3d_gltf()
