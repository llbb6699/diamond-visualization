# diamond-visualization
import numpy as np
import plotly.graph_objects as go

# 生成金刚石晶胞原子坐标（含超胞扩展）
def generate_diamond_structure(scale=2):
    a = 3.567  # 金刚石晶格常数(Å)
    base = np.array([[0,0,0], [0.5,0.5,0], [0.5,0,0.5], [0,0.5,0.5]])
    offset = np.array([0.25, 0.25, 0.25])
    atoms = np.vstack([base, base + offset]) * a
    
    # 扩展为2x2x2超胞
    if scale > 1:
        offsets = [np.array([i,j,k])*a for i in range(scale) 
                                for j in range(scale) 
                                for k in range(scale)]
        expanded = []
        for o in offsets:
            expanded.append(atoms + o)
        atoms = np.vstack(expanded)
    
    return atoms

# 生成键连接
def find_neighbors(atoms, a, cutoff=1.6):
    from scipy.spatial import cKDTree
    tree = cKDTree(atoms)
    pairs = tree.query_pairs(r=cutoff, output_type='ndarray')
    return atoms[pairs[:,0]], atoms[pairs[:,1]]]

# 创建3D可视化场景
atoms = generate_diamond_structure(scale=1)  # 单胞
bonds_start, bonds_end = find_neighbors(atoms, a=3.567)

fig = go.Figure()

# 添加原子（不同颜色区分两类原子）
fig.add_trace(go.Scatter3d(
    x=atoms[:4,0], y=atoms[:4,1], z=atoms[:4,2],
    mode='markers',
    marker=dict(
        size=8,
        color='rgb(255, 0, 0)',  # 红色原子
        opacity=0.8
    ),
    name='Base Atoms'
))

fig.add_trace(go.Scatter3d(
    x=atoms[4:,0], y=atoms[4:,1], z=atoms[4:,2],
    mode='markers',
    marker=dict(
        size=8,
        color='rgb(0, 0, 255)',  # 蓝色原子
        opacity=0.8
    ),
    name='Offset Atoms'
))

# 添加化学键
for start, end in zip(bonds_start, bonds_end):
    fig.add_trace(go.Scatter3d(
        x=[start[0], end[0]],
        y=[start[1], end[1]],
        z=[start[2], end[2]],
        mode='lines',
        line=dict(
            color='rgb(150, 150, 150)',
            width=6
        ),
        showlegend=False
    ))

# 设置场景布局
fig.update_layout(
    scene=dict(
        xaxis=dict(title='X (Å)', range=[0, 3.567]),
        yaxis=dict(title='Y (Å)', range=[0, 3.567]),
        zaxis=dict(title='Z (Å)', range=[0, 3.567]),
        aspectmode='cube',
        camera=dict(
            eye=dict(x=1.5, y=1.5, z=1.5)  # 初始视角
        )
    ),
    margin=dict(l=0, r=0, b=0, t=0),
    legend=dict(
        yanchor="top",
        y=0.99,
        xanchor="left",
        x=0.01
    )
)

# 保存为独立HTML文件（可离线查看）
fig.write_html("diamond_unit_cell.html")

# 在Jupyter中直接显示（可选）
# fig.show()
