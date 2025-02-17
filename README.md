1. Introduction
This study analyzes the street network of King's Cross, London, using a 2000m radius. The research explores how street centrality affects urban accessibility and retail distribution. The key questions addressed are:

How do closeness and betweenness centrality reflect the urban structure of King’s Cross?

What is the relationship between retail locations and tube stations?
These analyses provide insights into optimal retail locations and pedestrian movement in the study area.

2. Dataset and Neighbourhood

Study Area: King's Cross, London, chosen for its mixed land use and high connectivity.

Data Source: OpenStreetMap (OSM) data accessed via OSMnx.

Processing: Data retrieved and analyzed using Python, NetworkX, and GeoPandas.

3. Analysis and Visualisation

3.1 Data Acquisition
The street network data for King's Cross was obtained using OSMnx, allowing for the extraction of road networks within a 2000m radius. Retail locations and tube station data were sourced from OpenStreetMap to ensure comprehensive spatial coverage. The data underwent processing, which included filtering, cleaning, and structuring to maintain accuracy and relevance for subsequent analysis.


```python
import osmnx as ox
import networkx as nx
import matplotlib.pyplot as plt
import os

# 选择 King’s Cross 作为研究区域
address = "King's Cross Station, London, UK"
graphml_file = "kings_cross_graph.graphml"  # 文件名

# 检查文件是否存在，如果存在则直接加载，否则下载
if os.path.exists(graphml_file):
    G = ox.load_graphml(graphml_file)
else:
    G = ox.graph_from_address(address, dist=2000, network_type="walk")

    # 保存为 GraphML 文件，避免下次重新下载
    ox.save_graphml(G, graphml_file)

# 绘制街道网络
fig, ax = ox.plot_graph(G, node_size=10, edge_linewidth=0.5, bgcolor='black')

```


    
![png](formative%20assessment_files/formative%20assessment_3_0.png)
    


3.2 Closeness Centrality
Closeness centrality measures the ease of reaching all other nodes in the network. Higher values indicate more accessible locations within the street network, making these areas more pedestrian-friendly. By visualizing closeness centrality, key zones that facilitate efficient movement can be identified, providing insights into pedestrian accessibility and potential areas for commercial development.


```python
import osmnx as ox
import networkx as nx
import matplotlib.pyplot as plt
import numpy as np
import pickle  # 用于存储计算结果

# 定义存储文件路径
closeness_file = "closeness_centrality.pkl"

# 计算 & 读取 Closeness Centrality**
try:
    with open(closeness_file, "rb") as f:
        closeness_centrality = pickle.load(f)

except FileNotFoundError:
    closeness_centrality = nx.closeness_centrality(G)

    # 保存计算结果
    with open(closeness_file, "wb") as f:
        pickle.dump(closeness_centrality, f)

# 获取所有节点的颜色（根据 closeness 值进行颜色映射）
nc_closeness = [closeness_centrality[node] for node in G.nodes()]
norm_closeness = plt.Normalize(vmin=min(nc_closeness), vmax=max(nc_closeness))
cmap_closeness = plt.colormaps.get_cmap("coolwarm")
node_colors_closeness = [cmap_closeness(norm_closeness(value)) for value in nc_closeness]

# 绘制 Closeness Centrality 可视化
fig, ax = ox.plot_graph(
    G, 
    node_color=node_colors_closeness, 
    node_size=20, 
    edge_linewidth=0.5, 
    bgcolor="white"
)

```


    
![png](formative%20assessment_files/formative%20assessment_5_0.png)
    


3.3 Betweenness Centrality
Betweenness centrality measures how frequently a street segment is used as part of the shortest path between multiple locations. Streets with high betweenness act as primary traffic corridors, channeling significant pedestrian and vehicular movement. Identifying these key streets helps in understanding movement patterns and highlights critical infrastructure that supports urban mobility in King's Cross.


```python
import osmnx as ox
import networkx as nx
import matplotlib.pyplot as plt
import numpy as np
import pickle  # 用于保存 & 读取计算结果

# 定义文件路径
betweenness_file = "betweenness_centrality.pkl"

try:
    # 尝试加载已经计算好的 Betweenness Centrality
    with open(betweenness_file, "rb") as f:
        betweenness_centrality = pickle.load(f)

except FileNotFoundError:
    # 计算 Betweenness Centrality（可能需要几分钟）
    betweenness_centrality = nx.betweenness_centrality(G, weight="length", normalized=True)

    # 保存计算结果
    with open(betweenness_file, "wb") as f:
        pickle.dump(betweenness_centrality, f)

# 获取所有节点的颜色（根据 betweenness 值进行颜色映射）
nc = [betweenness_centrality[node] for node in G.nodes()]
norm = plt.Normalize(vmin=min(nc), vmax=max(nc))  # 归一化
cmap = plt.colormaps.get_cmap("inferno")  # Matplotlib 3.7+ 兼容
node_colors = [cmap(norm(value)) for value in nc]

# 绘制 Betweenness Centrality 可视化
fig, ax = ox.plot_graph(
    G, 
    node_color=node_colors,  # 颜色
    node_size=20, 
    edge_linewidth=0.5, 
    bgcolor="white"
)

```


    
![png](formative%20assessment_files/formative%20assessment_7_0.png)
    


3.4 Relationship Between Retail Locations and Tube Stations
The spatial distribution of retail locations in King's Cross was analyzed in relation to tube stations to assess whether proximity to high-accessibility transit points influences retail density. The findings indicate that retail establishments tend to cluster near tube stations, suggesting that public transport accessibility plays a key role in shaping commercial activity. These insights contribute to urban planning strategies aimed at optimizing commercial development in high-footfall areas.


```python
import os
import pickle
import osmnx as ox
import geopandas as gpd
import matplotlib.pyplot as plt
import matplotlib.lines as mlines

# 研究区域
address = "King's Cross Station, London, UK"

# **加载或获取 POI 数据**
poi_file = "pois.pkl"

if os.path.exists(poi_file):
    with open(poi_file, "rb") as f:
        pois = pickle.load(f)
    print("✅ 读取已有的 POI 数据")
else:
    # 🚇 获取地铁站数据
    tube_tags = {"public_transport": "station"}
    tube_stations = ox.features_from_address(address, tube_tags, dist=2000)

    # 🏪 获取商店数据
    shop_tags = {"shop": True}
    shops = ox.features_from_address(address, shop_tags, dist=2000)

    # 存储数据
    pois = {"tube_stations": tube_stations, "shops": shops}
    with open(poi_file, "wb") as f:
        pickle.dump(pois, f)
    print("✅ POI 数据获取完成并保存")

# 解包 POI 数据
tube_stations, shops = pois["tube_stations"], pois["shops"]

# **绘制 POI 数据**
fig, ax = plt.subplots(figsize=(10, 10))

# 画出街道网络
ox.plot_graph(G, ax=ax, node_size=10, edge_linewidth=0.5, bgcolor="white", show=False)

# 画出地铁站（确保不为空）
if len(tube_stations) > 0:
    tube_stations.plot(ax=ax, color="blue", markersize=50)

# 画出商店（确保不为空）
if len(shops) > 0:
    shops.plot(ax=ax, color="red", markersize=30)

# 手动创建图例
legend_elements = []
if len(tube_stations) > 0:
    legend_elements.append(mlines.Line2D([], [], color="blue", marker="o", linestyle="None", markersize=10, label="Tube Stations"))
if len(shops) > 0:
    legend_elements.append(mlines.Line2D([], [], color="red", marker="o", linestyle="None", markersize=8, label="Shops"))

# 添加图例
if legend_elements:
    ax.legend(handles=legend_elements)

plt.title("King’s Cross: Shops & Tube Stations")
plt.show()

```

    ✅ 读取已有的 POI 数据
    


    
![png](formative%20assessment_files/formative%20assessment_9_1.png)
    


4. Discussion

The analysis reveals a strong correlation between retail locations and high closeness and betweenness centrality. Retail establishments are more likely to be found in areas that are highly accessible within the street network, as these locations facilitate greater pedestrian movement. Additionally, proximity to tube stations appears to be a significant factor in retail distribution patterns, with businesses clustering around key transit points to maximize foot traffic. The use of network centrality measures provides valuable insights into the spatial structure of King's Cross, highlighting how urban accessibility shapes commercial activity.

If asked to place a shop to maximize footfall, the optimal location would be in areas with high betweenness and closeness centrality, particularly near major intersections of pedestrian pathways and close to tube stations. These locations naturally experience a high volume of foot traffic, ensuring maximum visibility and accessibility for potential customers. Additionally, placing a shop near transit hubs would cater to both local commuters and visitors, further enhancing business opportunities.
The analysis reveals a strong correlation between retail locations and high closeness and betweenness centrality. Retail establishments are more likely to be found in areas that are highly accessible within the street network, as these locations facilitate greater pedestrian movement. Additionally, proximity to tube stations appears to be a significant factor in retail distribution patterns, with businesses clustering around key transit points to maximize foot traffic. The use of network centrality measures provides valuable insights into the spatial structure of King's Cross, highlighting how urban accessibility shapes commercial activity.

5. Conclusion

This study provides a spatial analysis of King’s Cross, demonstrating how network centrality and transit access influence retail location choices. The findings suggest that areas with high accessibility and proximity to tube stations attract a greater concentration of commercial activity. These insights contribute to a broader understanding of urban planning and pedestrian movement. Future research could integrate foot traffic data to validate the findings further and explore additional factors influencing retail location dynamics.
This study provides a spatial analysis of King’s Cross, demonstrating how network centrality and transit access influence retail location choices. Future work could integrate foot traffic data to validate findings further.


```python

```
