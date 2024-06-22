Sure! Let's create a more advanced chapter that builds on the basics. In this chapter, we'll explore more advanced topics such as:

1. Adding attributes to nodes and edges.
2. Finding paths and cycles in the graph.
3. Visualizing the graph with attributes.
4. Analyzing the graph's connectivity and centrality.

## Chapter 2: Advanced Network Analysis with NetworkX

### 1. Adding Attributes to Nodes and Edges

We can add custom attributes to nodes and edges to represent additional information about the tables and relationships:

```python
# Add attributes to nodes
G.nodes["Table_A"]["type"] = "source"
G.nodes["Table_B"]["type"] = "intermediate"
G.nodes["Table_C"]["type"] = "intermediate"
G.nodes["Table_D"]["type"] = "intermediate"
G.nodes["Table_E"]["type"] = "sink"

# Add attributes to edges
G.edges["Table_A", "Table_B"]["weight"] = 1.5
G.edges["Table_A", "Table_C"]["weight"] = 2.0
G.edges["Table_B", "Table_D"]["weight"] = 1.0
G.edges["Table_C", "Table_D"]["weight"] = 0.5
G.edges["Table_D", "Table_E"]["weight"] = 1.2

# Print node and edge attributes
print("Node attributes:", dict(G.nodes(data=True)))
print("Edge attributes:", dict(G.edges(data=True)))
```

### 2. Finding Paths and Cycles

NetworkX provides functions to find all paths and detect cycles in the graph:

```python
# Find all paths from source to sink
paths = list(nx.all_simple_paths(G, source="Table_A", target="Table_E"))
print("All paths from Table_A to Table_E:", paths)

# Detect cycles in the graph (if any)
cycles = list(nx.simple_cycles(G))
print("Cycles in the graph:", cycles)
```

### 3. Visualizing the Graph with Attributes

Enhance the graph visualization by including node and edge attributes:

```python
# Draw the graph with attributes
plt.figure(figsize=(10, 7))
pos = nx.spring_layout(G)
nx.draw(G, pos, with_labels=True, node_size=3000, node_color="skyblue", font_size=15, font_weight="bold", arrowsize=20)

# Draw node labels for attributes
node_labels = nx.get_node_attributes(G, 'type')
nx.draw_networkx_labels(G, pos, labels=node_labels, font_size=12, font_color="black")

# Draw edge labels for attributes
edge_labels = nx.get_edge_attributes(G, 'weight')
nx.draw_networkx_edge_labels(G, pos, edge_labels=edge_labels, font_size=12)

plt.title("Data Pipeline Graph with Attributes")
plt.show()
```

### 4. Analyzing Connectivity and Centrality

Analyze the graph's connectivity and centrality to understand its structure and importance of nodes:

```python
# Check if the graph is strongly connected
is_strongly_connected = nx.is_strongly_connected(G)
print("Is the graph strongly connected?", is_strongly_connected)

# Check if the graph is weakly connected
is_weakly_connected = nx.is_weakly_connected(G)
print("Is the graph weakly connected?", is_weakly_connected)

# Calculate centrality measures
in_degree_centrality = nx.in_degree_centrality(G)
out_degree_centrality = nx.out_degree_centrality(G)
betweenness_centrality = nx.betweenness_centrality(G)

# Convert centrality measures to DataFrame
centrality_df = pd.DataFrame({
    "Node": list(G.nodes()),
    "In-Degree Centrality": in_degree_centrality.values(),
    "Out-Degree Centrality": out_degree_centrality.values(),
    "Betweenness Centrality": betweenness_centrality.values()
})
print(centrality_df)
```

### Conclusion

In this chapter, we've added attributes to nodes and edges, explored paths and cycles, enhanced graph visualization with attributes, and analyzed the graph's connectivity and centrality. These advanced techniques provide deeper insights into your data pipeline's structure and behavior.

By continuing to experiment and build upon these concepts, you can create a robust simulation of complex data pipelines and gain valuable insights from your network analysis.

Happy coding!.