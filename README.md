1. Introduction

This report analyses the street network in King's Cross, London. The area covers a 2000m radius. The study explores how street centrality affects urban accessibility and retail distribution. The key research questions are:

How do different streets connect to each other, and how does this affect movement?

Is there a link between where shops are and how close they are to tube stations?

By looking at these, we can see how people move in the area and where shops might do well.

2. Dataset and Neighbourhood

The study focuses on King's Cross, a busy area with many shops, offices, and transport links. The data comes from OpenStreetMap (OSM) and was collected using OSMnx. The data includes street layouts, shop locations, and tube station positions. 

3. Analysis and Visualisation

3.1 Data Acquisition


```python
import osmnx as ox
import networkx as nx
import matplotlib.pyplot as plt
import os

# Choose King’s Cross as research area
address = "King's Cross Station, London, UK"
graphml_file = "kings_cross_graph.graphml"

# Check whether the file exists, if it does, load it directly, otherwise download it
if os.path.exists(graphml_file):
    G = ox.load_graphml(graphml_file)
else:
    G = ox.graph_from_address(address, dist=2000, network_type="walk")

    # Save as GraphML file
    ox.save_graphml(G, graphml_file)

# Plot the street network map
fig, ax = ox.plot_graph(G, node_size=10, edge_linewidth=0.5, bgcolor='black')
```


    
![png](output_3_0.png)
    


3.2 Closeness Centrality

Closeness centrality measures how easily a place connects to other places. If a street has a high score, it means people can get to many other streets quickly. The results show which areas are easy to reach. These places are likely to have a lot of foot traffic, making them good for businesses.


```python
import osmnx as ox
import networkx as nx
import matplotlib.pyplot as plt
import numpy as np
import pickle

# Define the path to the storage file
closeness_file = "closeness_centrality.pkl"

# Calculate & read Closeness Centrality
try:
    with open(closeness_file, "rb") as f:
        closeness_centrality = pickle.load(f)

except FileNotFoundError:
    closeness_centrality = nx.closeness_centrality(G)

    # Save the calculate result
    with open(closeness_file, "wb") as f:
        pickle.dump(closeness_centrality, f)

# Gets the colour of all nodes
nc_closeness = [closeness_centrality[node] for node in G.nodes()]
norm_closeness = plt.Normalize(vmin=min(nc_closeness), vmax=max(nc_closeness))
cmap_closeness = plt.colormaps.get_cmap("coolwarm")
node_colors_closeness = [cmap_closeness(norm_closeness(value)) for value in nc_closeness]

# Plot Closeness Centrality Map
fig, ax = ox.plot_graph(
    G, 
    node_color=node_colors_closeness, 
    node_size=20, 
    edge_linewidth=0.5, 
    bgcolor="white"
)
```


    
![png](output_5_0.png)
    


3.3 Betweenness Centrality

Betweenness centrality shows how often a street is used as a shortcut. If a street has a high score, it means many people pass through it. These streets act like main routes that help people move around the area. Knowing this helps identify which streets are the busiest and most important.


```python
import osmnx as ox
import networkx as nx
import matplotlib.pyplot as plt
import numpy as np
import pickle 

betweenness_file = "betweenness_centrality.pkl"

try:
    # Try to load the already calculated Betweenness Centrality
    with open(betweenness_file, "rb") as f:
        betweenness_centrality = pickle.load(f)

except FileNotFoundError:
    # Calculate Betweenness Centrality
    betweenness_centrality = nx.betweenness_centrality(G, weight="length", normalized=True)

    # Save the calculate result
    with open(betweenness_file, "wb") as f:
        pickle.dump(betweenness_centrality, f)

# Gets the colour of all nodes
nc = [betweenness_centrality[node] for node in G.nodes()]
norm = plt.Normalize(vmin=min(nc), vmax=max(nc)) 
cmap = plt.colormaps.get_cmap("inferno")  
node_colors = [cmap(norm(value)) for value in nc]

# Plot Betweenness Centrality Map
fig, ax = ox.plot_graph(
    G, 
    node_color=node_colors, 
    node_size=20, 
    edge_linewidth=0.5, 
    bgcolor="white"
)
```


    
![png](output_7_0.png)
    


3.4 Relationship Between Retail Locations and Tube Stations

The study also looks at where shops are and how close they are to tube stations. The results show that many shops are near stations. This suggests that businesses prefer to be close to public transport, where there are more people. The findings can help in planning where to open new shops.


```python
import os
import pickle
import osmnx as ox
import geopandas as gpd
import matplotlib.pyplot as plt
import matplotlib.lines as mlines

address = "King's Cross Station, London, UK"

# Load or get POI data
poi_file = "pois.pkl"

if os.path.exists(poi_file):
    with open(poi_file, "rb") as f:
        pois = pickle.load(f)
else:
    # Get tube station data
    tube_tags = {"public_transport": "station"}
    tube_stations = ox.features_from_address(address, tube_tags, dist=2000)

    # Get retail location data
    shop_tags = {"shop": True}
    shops = ox.features_from_address(address, shop_tags, dist=2000)

    # Save data
    pois = {"tube_stations": tube_stations, "shops": shops}
    with open(poi_file, "wb") as f:
        pickle.dump(pois, f)

# Unpack POI data
tube_stations, shops = pois["tube_stations"], pois["shops"]

# Plot POI data
fig, ax = plt.subplots(figsize=(10, 10))

# Plot street network
ox.plot_graph(G, ax=ax, node_size=10, edge_linewidth=0.5, bgcolor="white", show=False)

# Plot tube stations
if len(tube_stations) > 0:
    tube_stations.plot(ax=ax, color="blue", markersize=50)

# Plot retail locations
if len(shops) > 0:
    shops.plot(ax=ax, color="red", markersize=30)

# Create the legend
legend_elements = []
if len(tube_stations) > 0:
    legend_elements.append(mlines.Line2D([], [], color="blue", marker="o", linestyle="None", markersize=10, label="Tube Stations"))
if len(shops) > 0:
    legend_elements.append(mlines.Line2D([], [], color="red", marker="o", linestyle="None", markersize=8, label="Shops"))

if legend_elements:
    ax.legend(handles=legend_elements)

plt.title("King’s Cross: Shops & Tube Stations")
plt.show()
```


    
![png](output_9_0.png)
    


4. Discussion

Shops are more likely to be in places that are easy to reach and have a lot of people passing through. Streets with high centrality scores help people move quickly, making them attractive for businesses. The study also shows that shops are often close to tube stations. This is because these areas have more people walking by, increasing the chances of customers stopping in.

If someone wants to open a new shop and get the most foot traffic, they should choose a spot on a busy street with high centrality. The best places are near major intersections where many pedestrians pass. Being close to a tube station would also help bring in both commuters and visitors. These locations ensure that the shop is easy to find and attracts more customers.

5. Conclusion

This study looks at how street networks affect movement and shop locations in King’s Cross. The results show that the busiest streets and areas near tube stations are the best places for shops. These areas get the most foot traffic, which is important for businesses. Future studies could include real-world foot traffic data to confirm these findings and get a clearer picture of how people move in the area.

Reference

OpenStreetMap (OSM). (2024). OpenStreetMap Data. Retrieved from https://www.openstreetmap.org
