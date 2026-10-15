AICOF Simulation Package V5.1 --- Complete Function Reference
1. Overview
This package is AICOF Simulation Package V5.1, a paper-lock
refinement of the V5 simulation framework for evaluating processor
interconnection-network topologies and routing strategies.
The package focuses on:
Structural analysis of processor interconnection networks
Traditional shortest-path routing
Static adaptive routing
MLP-style routing
GNN-style routing
RL-style routing
Integrated AICOF routing
AICOF component ablation
Scalability analysis
AICOF improvement/gain analysis
CSV result generation
IEEE-style LaTeX table generation
V5.1-specific refinement
The supplied `README_V5_1.txt` describes V5.1 as a paper-lock
refinement of V5 with:
stronger GNN ablation visibility,
mild topology-sensitive congestion/fault penalties,
preservation of AICOF > RL,
improved topology-wise gains.
The supplied source also contains a `v51_tuning` configuration intended
to control GNN, RL, MLP, congestion, and fault-related tuning.
---
2. Package Contents
The supplied archive contains the following files:
``` text
AICOF_simulation_package_V5_1_work/
│
├── config.py
├── topologies.py
├── traffic.py
├── metrics.py
├── routing.py
├── simulator.py
├── experiments.py
├── parallel.py
├── latex_export.py
├── utils.py
├── run_all.py
├── requirements.txt
├── README.md
└── README_V5_1.txt
```
No `results/` directory is included in the supplied archive. Results are
created when `run_all.py` is executed.
---
3. Overall Execution Architecture
The package follows this execution flow:
``` text
                         run_all.py
                             |
          +------------------+------------------+
          |                  |                  |
      config.py         experiments.py       utils.py
                             |
            +----------------+----------------+
            |                |                |
       topologies.py     simulator.py     parallel.py
            |                |
        network graph      routing.py
                             |
                       traffic.py
                       metrics.py
                             |
                             v
                       Result DataFrames
                             |
                             v
                      latex_export.py
                             |
                +------------+------------+
                |                         |
             *.csv                    *.tex
```
---
4. Installation
4.1 Requirements
The supplied `requirements.txt` contains:
``` text
networkx>=3.2
numpy>=1.24
pandas>=2.0
```
Install with:
``` bash
python -m pip install -r requirements.txt
```
or:
``` bash
pip install -r requirements.txt
```
4.2 Verify the environment
``` bash
python -c "import networkx, numpy, pandas; print('Environment OK')"
```
---
5. Important V5.1 Configuration Note
The supplied `config.py` defines:
``` python
CFG = ExperimentConfig()
```
and then contains:
``` python
CFG["v51_tuning"] = {
    ...
}
```
`ExperimentConfig` is a Python dataclass and does not implement
dictionary-style item assignment.
Therefore, the supplied source as archived contains a
configuration-code inconsistency that should be corrected before
execution.
A safe correction is to represent the V5.1 tuning configuration as a
normal attribute, for example:
``` python
CFG.v51_tuning = {
    "gnn_weight_boost": 1.18,
    "rl_weight_scale": 0.96,
    "mlp_weight_scale": 0.95,
    "congestion_topology_boost": {
        "torus": 1.08,
        "dragonfly": 1.08,
        "pdn": 1.08
    },
    "fault_topology_boost": {
        "torus": 1.06,
        "dragonfly": 1.08,
        "fat-tree": 1.10,
        "pdn": 1.05
    }
}
```
However, the supplied V5.1 `routing.py` does not currently read this
tuning dictionary. Therefore, changing the configuration to an attribute
alone does not make these values active in routing.
This README documents the actual supplied source behavior, rather
than assuming that the tuning dictionary is already applied.
---
6. Running the Package
6.1 Quick validation
``` bash
python run_all.py --quick --workers 2
```
Quick mode changes:
``` text
repetitions = 4
packets_per_run = 500
max_steps_per_packet = 32
```
The purpose is to check that the complete pipeline runs.
6.2 Full experiment
``` bash
python run_all.py --workers 4
```
Full mode uses the values defined in `config.py`:
``` text
node_sizes = [64, 128, 256, 512]
repetitions = 12
packets_per_run = 1800
max_steps_per_packet = 56
```
6.3 Single-worker execution
``` bash
python run_all.py --workers 1
```
This is recommended for debugging.
6.4 Parallel execution
``` bash
python run_all.py --workers 2
```
or:
``` bash
python run_all.py --workers 4
```
The `--workers` argument controls the number of multiprocessing workers.
---
7. `config.py`
7.1 `ExperimentConfig`
Defined as:
``` python
@dataclass
class ExperimentConfig:
    node_sizes: list = field(
        default_factory=lambda: [64, 128, 256, 512]
    )
    repetitions: int = 12
    packets_per_run: int = 1800
    max_steps_per_packet: int = 56
    traffic_pattern: str = "uniform"
    hotspot_fraction: float = 0.22
    failure_ratio_baseline: float = 0.05
    failure_ratio_scalability: float = 0.02
    failure_ratio_ai: float = 0.08
    random_seed: int = 42
```
Purpose
Stores the global simulation parameters.
Parameters
---
Parameter                     Meaning                                    Default
---
`node_sizes`                  Network sizes used              `[64,128,256,512]`
for scalability
`repetitions`                 Repeated simulation                           `12`
runs
`packets_per_run`             Packets generated per                       `1800`
simulation
`max_steps_per_packet`        Maximum routing steps                         `56`
for each packet
`traffic_pattern`             Traffic-generation                       `uniform`
pattern
`hotspot_fraction`            Fraction of nodes                           `0.22`
treated as hotspots
`failure_ratio_baseline`      Baseline failure                            `0.05`
ratio
`failure_ratio_scalability`   Scalability failure                         `0.02`
ratio
`failure_ratio_ai`            AI-comparison failure                       `0.08`
ratio
`random_seed`                 Base random seed                              `42`
---
8. `CFG`
``` python
CFG = ExperimentConfig()
```
Purpose
Creates the global configuration object.
It is imported by:
``` python
experiments.py
```
and used throughout the simulation.
---
9. V5.1 Paper-Lock Tuning
The supplied `config.py` contains the following intended tuning:
``` python
{
    "gnn_weight_boost": 1.18,
    "rl_weight_scale": 0.96,
    "mlp_weight_scale": 0.95,
    "congestion_topology_boost": {
        "torus": 1.08,
        "dragonfly": 1.08,
        "pdn": 1.08
    },
    "fault_topology_boost": {
        "torus": 1.06,
        "dragonfly": 1.08,
        "fat-tree": 1.10,
        "pdn": 1.05
    }
}
```
Intended interpretation
---
Parameter                           Intended purpose
---
`gnn_weight_boost`                  Increase GNN contribution
`rl_weight_scale`                   Scale RL contribution
`mlp_weight_scale`                  Scale MLP contribution
`congestion_topology_boost`         Topology-specific congestion
adjustment
`fault_topology_boost`              Topology-specific fault-awareness
adjustment
Current source status
The current `routing.py` does not retrieve these values from `cfg`.
Therefore they are declared tuning values, not active runtime
controls in the supplied source.
---
10. `topologies.py`
This module creates the five network topologies used by the experiments.
---
11. `build_hypercube(n)`
``` python
build_hypercube(n)
```
Purpose
Creates an `n`-node hypercube.
Internally:
``` python
dim = int(math.log2(n))
nx.hypercube_graph(dim)
```
Requirement
`n` must be a power of two.
Valid examples:
``` text
64
128
256
512
```
Invalid example:
``` text
100
```
The function raises:
``` text
ValueError: Hypercube requires power-of-2 size.
```
Used by
``` text
get_topology()
resolve_valid_size()
structural experiment
baseline experiment
scalability experiment
AI comparison
AICOF gain
ablation
```
---
12. `build_torus(n)`
``` python
build_torus(n)
```
Purpose
Creates a two-dimensional periodic grid.
The implementation uses:
``` python
nx.grid_2d_graph(
    side,
    side,
    periodic=True
)
```
Requirement
`n` must be a perfect square.
Examples:
``` text
64
256
```
The node count is therefore represented as:
``` text
side × side
```
---
13. `build_fat_tree(n)`
``` python
build_fat_tree(n)
```
Purpose
Creates a balanced-tree approximation of a fat-tree topology.
The supplied implementation uses:
``` python
nx.balanced_tree(r, depth)
```
with:
``` python
r = 4
```
The graph is truncated if it exceeds the requested node count.
Important note
This is a simplified balanced-tree approximation, not a full
conventional datacenter fat-tree implementation.
---
14. `build_dragonfly(n)`
``` python
build_dragonfly(n)
```
Purpose
Creates a simplified Dragonfly-like graph.
The implementation:
Determines a group size.
Creates groups.
Fully connects nodes inside each group.
Adds inter-group links.
This provides a graph abstraction of Dragonfly rather than a complete
hardware-level Dragonfly topology.
---
15. `build_pdn(n)`
``` python
build_pdn(n)
```
Purpose
Creates the PDN topology used by the simulation.
The implementation uses modular offsets:
``` python
offsets = [1, 3, 7]
```
and adds positive and negative offset connections.
---
16. `nearest_power_of_two(n)`
``` python
nearest_power_of_two(n)
```
Purpose
Finds the nearest supported power-of-two node count.
This is required because:
``` text
Hypercube → N = 2^k
```
---
17. `nearest_perfect_square(n)`
``` python
nearest_perfect_square(n)
```
Purpose
Finds a nearby perfect-square node count for the torus topology.
The implementation checks neighboring square values around:
``` python
sqrt(n)
```
---
18. `resolve_valid_size(topology, requested_n)`
``` python
resolve_valid_size(topology, requested_n)
```
Purpose
Converts a requested node count into a topology-valid node count.
Rules:
``` text
hypercube → nearest power of two
torus     → nearest perfect square
others    → requested value
```
Example
``` python
resolve_valid_size("hypercube", 100)
```
returns a nearby power of two.
---
19. `get_topology(name, n)`
``` python
get_topology(name, n)
```
Purpose
Acts as the central topology factory.
Supported topology names:
``` text
hypercube
torus
fat-tree
dragonfly
pdn
```
Example:
``` python
G = get_topology("hypercube", 64)
```
---
20. `traffic.py`
`sample_packet_pairs()`
``` python
sample_packet_pairs(
    nodes,
    count,
    pattern="uniform",
    hotspot_fraction=0.2,
    rng=None
)
```
Purpose
Generates source-destination pairs for packet routing.
---
Uniform traffic
``` python
sample_packet_pairs(
    nodes,
    1800,
    pattern="uniform"
)
```
Both source and destination are selected randomly.
---
Hotspot traffic
``` python
sample_packet_pairs(
    nodes,
    1800,
    pattern="hotspot",
    hotspot_fraction=0.22
)
```
The first `hotspot_fraction` of nodes are considered hotspots.
With probability:
``` text
0.6
```
the destination is selected from the hotspot set.
---
Output
Returns:
``` python
[
    (source1, destination1),
    (source2, destination2),
    ...
]
```
---
21. `metrics.py`
`safe_average_shortest_path_length(G)`
``` python
safe_average_shortest_path_length(G)
```
Purpose
Calculates average shortest-path length.
If the graph is disconnected, the function uses the largest connected
component.
This prevents an exception from disconnected graphs.
---
22. `structural_metrics(G)`
``` python
structural_metrics(G)
```
Purpose
Calculates structural characteristics of a topology.
Returns:
``` python
{
    "node_degree": ...,
    "diameter": ...,
    "avg_path_length": ...,
    "path_diversity": ...,
    "scalability": ...,
    "fault_tolerance": ...
}
```
---
22.1 Node Degree
Calculated as the average degree:
``` text
mean degree of all nodes
```
---
22.2 Diameter
Maximum shortest-path distance.
For disconnected graphs, the largest connected component is used.
---
22.3 Average Path Length
Uses:
``` python
safe_average_shortest_path_length()
```
---
22.4 Path Diversity
The supplied implementation calculates:
``` text
network density × average node degree
```
This is a simulation-defined proxy rather than a conventional
path-disjointness metric.
---
22.5 Scalability
The supplied implementation calculates:
``` text
1 / average path length
```
This is a normalized structural proxy.
---
22.6 Fault Tolerance
Calculated as:
``` text
(edge connectivity + node connectivity) / 2
```
---
23. `connectivity_ratio(G)`
``` python
connectivity_ratio(G)
```
Purpose
Calculates the fraction of nodes contained in the largest connected
component.
Formula:
``` text
Connectivity Ratio =
largest connected component size / total node count
```
Range:
``` text
0 → 1
```
Higher is better.
---
24. `routing.py`
This module contains all routing policies.
Supported policies:
``` text
Shortest Path
Static Adaptive
MLP
GNN
RL
AICOF
```
and the AICOF ablation variants.
---
25. `_v51_topology_multiplier()`
``` python
_v51_topology_multiplier(
    topology,
    table,
    default=1.0
)
```
Purpose
Retrieves a topology-specific multiplier from a dictionary.
Example:
``` python
_v51_topology_multiplier(
    "torus",
    {"torus": 1.08}
)
```
returns:
``` text
1.08
```
If the topology is not found, the default is returned.
Current use
The function is defined in `routing.py`, but the supplied V5.1 routing
implementation does not call it.
---
26. `_safe_norm()`
``` python
_safe_norm(x, scale)
```
Purpose
Safely normalizes a value.
Formula:
``` text
x / scale
```
If:
``` text
scale <= 0
```
it returns:
``` text
0.0
```
Used by
`MLPPolicy.score()`.
---
27. `RoutingPolicy`
``` python
RoutingPolicy(G, cfg, rng=None)
```
Purpose
Base class for all routing policies.
It stores:
``` text
Graph
Configuration
Random generator
Maximum degree
Betweenness centrality
Closeness centrality
```
For graphs up to 600 nodes:
``` python
nx.betweenness_centrality()
nx.closeness_centrality()
```
are calculated.
For larger graphs, the centrality values are set to zero to avoid
expensive computation.
---
28. `RoutingPolicy.dist()`
``` python
dist(u, v)
```
Purpose
Returns shortest-path distance.
If no path exists:
``` text
10^9
```
is returned.
This provides a large penalty to unreachable nodes.
---
29. `RoutingPolicy.local_fault_penalty()`
``` python
local_fault_penalty(node, failed_nodes)
```
Returns:
``` text
1.0 → node is failed
0.0 → node is not failed
```
Used in adaptive and AI-style policies.
---
30. `RoutingPolicy.neighbor_resilience()`
``` python
neighbor_resilience(node)
```
Purpose
Provides a normalized degree-based resilience estimate.
Formula:
``` text
degree(node) / maximum degree
```
capped at 1.
---
31. `RoutingPolicy.two_hop_congestion()`
``` python
two_hop_congestion(node, queue_len)
```
Purpose
Estimates local congestion.
It considers:
the current node queue,
one-hop neighbors,
two-hop neighbors.
Two-hop queue values receive a factor of:
``` text
0.5
```
---
32. `ShortestPathPolicy`
`choose_next_hop()`
``` python
choose_next_hop(
    current,
    dst,
    queue_len,
    link_load,
    failed_nodes=None,
    visited=None
)
```
Purpose
Selects the neighboring node having the minimum shortest-path distance
to the destination.
Conceptually:
``` text
current
   |
   +--> neighbor with smallest distance to dst
```
Used by
The traditional baseline and scalability experiments.
---
33. `StaticAdaptivePolicy`
`choose_next_hop()`
Selects the next hop using a weighted adaptive score.
The score includes:
destination progress,
remaining distance,
queue length,
link load,
node degree,
fault penalty,
visited-node penalty.
The supplied coefficients are:
``` text
progress        +5.4
distance        -0.22
queue           -0.22
link load       -0.15
degree          +0.18
fault           -1.2
visited         -1.8
```
Higher score is preferred.
---
34. `MLPPolicy`
`score()`
``` python
score(
    current,
    v,
    dst,
    queue_len,
    link_load,
    failed_nodes=None,
    visited=None
)
```
Purpose
Calculates a local next-hop score using features such as:
inverse destination distance,
normalized degree,
queue congestion,
link load,
fault status,
visited status.
The score is:
``` text
2.6 × inverse distance
+ 0.95 × normalized degree
- 1.25 × queue factor
- 0.95 × link-load factor
- 1.05 × fault penalty
- 0.55 × visited penalty
```
`choose_next_hop()`
Chooses the neighbor having the highest score.
Important scientific note
Despite the name `MLPPolicy`, the supplied V5.1 source does not
create or train an actual neural-network MLP.
There is no:
``` python
torch.nn
```
model in this package.
The implementation is a weighted mathematical scoring policy.
---
35. `GNNPolicy`
`score()`
Uses graph-derived features:
inverse destination distance,
betweenness centrality,
closeness centrality,
neighbor resilience,
two-hop congestion,
fault penalty,
visited penalty.
The supplied score is:
``` text
1.85 × inverse distance
+ 1.15 × betweenness
+ 1.05 × closeness
+ 0.80 × resilience
- 1.20 × two-hop congestion
- 1.15 × fault
- 0.45 × visited
```
`choose_next_hop()`
Chooses the highest-scoring neighbor.
Important scientific note
The supplied implementation does not use:
``` text
PyTorch
PyTorch Geometric
GraphConv
GCN
GAT
GraphSAGE
```
It is a graph-feature-based heuristic policy.
---
36. `RLPolicy`
`score()`
Uses:
progress toward destination,
remaining distance,
queue congestion,
link load,
fault risk,
neighbor resilience,
visited-node penalty.
The risk term is:
``` text
0.65 × fault penalty
+
0.35 × (1 - neighbor resilience)
```
The score includes:
``` text
3.0 × progress
- 0.20 × distance
- 0.75 × queue
- 0.58 × link load
- 1.0 × risk
- 0.42 × visited
```
`choose_next_hop()`
Selects the highest-scoring neighbor.
Important scientific note
The supplied V5.1 source does not train PPO, DQN, SAC, or another
reinforcement-learning agent.
There is no:
``` python
stable_baselines3
```
dependency.
Therefore this is an RL-style reward/score heuristic, not a trained RL
model.
---
37. `AICOFPolicy`
``` python
AICOFPolicy(
    G,
    cfg,
    rng=None,
    use_mlp=True,
    use_gnn=True,
    use_rl=True,
    use_congestion=True,
    use_fault=True
)
```
Purpose
Implements the integrated AICOF routing policy.
The framework combines:
``` text
MLP-style scoring
GNN-style graph scoring
RL-style adaptive scoring
Congestion awareness
Fault awareness
```
---
38. AICOF Component Flags
Flag               Function
---
`use_mlp`          Enables MLP-style component
`use_gnn`          Enables GNN-style component
`use_rl`           Enables RL-style component
`use_congestion`   Enables congestion penalties
`use_fault`        Enables fault-awareness
---
39. AICOF Fusion Score
For each candidate neighbor, the supplied implementation calculates:
``` text
AICOF score =
    0.75 × MLP score
  + 1.10 × GNN score
  + 0.85 × RL score
  + 0.45 × destination-distance contribution
  - congestion penalties
  - fault penalty
  + resilience contribution
  - visited penalty
```
The congestion contribution contains:
``` text
1.10 × normalized queue
0.85 × normalized link load
0.60 × normalized two-hop congestion
```
The fault component contains:
``` text
1.15 × fault penalty
0.80 × neighbor resilience
```
The highest-scoring neighbor is selected.
---
40. AICOF Ablation Variants
The package defines six variants.
Variant                       Disabled component
---
`AICOF-Full`                  None
`AICOF w/o RL`                RL
`AICOF w/o GNN`               GNN
`AICOF w/o MLP`               MLP
`AICOF w/o Congestion`        Congestion
`AICOF w/o Fault-Awareness`   Fault awareness
---
41. `build_policy()`
``` python
build_policy(name, G, cfg, rng=None)
```
Purpose
Creates the correct routing policy based on its name.
Supported values:
``` text
shortest_path
static_adaptive
mlp
gnn
rl
aicof
aicof_no_rl
aicof_no_gnn
aicof_no_mlp
aicof_no_congestion
aicof_no_fault
```
Example:
``` python
policy = build_policy(
    "aicof",
    G,
    CFG,
    rng
)
```
---
42. `simulator.py`
This module executes packet-level routing simulations.
---
43. `METHOD_LOAD`
The supplied source defines:
``` python
METHOD_LOAD = {
    "shortest_path": (1.00, 1.00),
    "static_adaptive": (1.04, 1.02),
    "mlp": (0.95, 0.94),
    "gnn": (0.92, 0.90),
    "rl": (0.88, 0.86),
    "aicof": (0.84, 0.82),
    ...
}
```
Each tuple represents:
``` text
(latency scale, energy scale)
```
These factors directly influence simulated latency and energy.
Important interpretation
These are manually specified simulation scaling parameters, not
learned model parameters.
---
44. `METHOD_FAIL_PENALTY`
This dictionary defines method-specific failure scaling.
Example:
``` python
"aicof": 0.82
```
Therefore:
``` text
effective failure ratio =
requested failure ratio × method failure factor
```
For AICOF:
``` text
effective ratio = requested ratio × 0.82
```
Important scientific implication
Different methods can therefore experience different effective numbers
of failed edges.
For a strict controlled comparison, all methods should ideally use the
same failure realization.
---
45. `apply_failures()`
``` python
apply_failures(
    G,
    failure_ratio,
    rng
)
```
Purpose
Injects random edge failures into the network.
Steps
Copies the original graph.
Calculates the number of failed edges.
Randomly samples edges.
Removes those edges.
Identifies isolated nodes.
Returns
``` python
H, failed_nodes
```
where:
``` text
H            → network after failures
failed_nodes → nodes with degree zero
```
---
46. `simulate_routing()`
``` python
simulate_routing(
    G,
    cfg,
    policy_name,
    failure_ratio=0.0,
    seed=42
)
```
Purpose
This is the main simulation function.
---
47. Simulation Processing Sequence
``` text
1. Initialize random generator
        ↓
2. Calculate effective failure ratio
        ↓
3. Apply edge failures
        ↓
4. Build routing policy
        ↓
5. Generate source/destination pairs
        ↓
6. Route each packet
        ↓
7. Track queues and link loads
        ↓
8. Accumulate latency and energy
        ↓
9. Calculate PDR
        ↓
10. Calculate throughput
        ↓
11. Calculate connectivity ratio
        ↓
12. Calculate fault recovery score
        ↓
13. Return metrics
```
---
48. `simulate_routing()` Output
Returns:
``` python
{
    "connectivity_ratio": ...,
    "throughput": ...,
    "latency": ...,
    "pdr": ...,
    "energy_per_packet": ...,
    "fault_recovery_score": ...
}
```
---
49. Connectivity Ratio
Calculated using:
``` python
connectivity_ratio(H)
```
It is:
``` text
largest connected component / total nodes
```
---
50. Throughput
The supplied implementation calculates:
``` text
throughput =
delivered packets / max_steps_per_packet
```
Important note
This is a simulation-defined throughput proxy.
It is not a physical measurement such as:
``` text
packets/second
Gbps
bits/second
```
unless the model is extended with explicit simulation time.
---
51. Latency
For every routed hop:
``` text
latency =
latency_scale ×
(1
 + 0.022 × queue length
 + 0.012 × link load)
```
The final metric is:
``` text
total latency / delivered packets
```
Lower is better.
---
52. Packet Delivery Ratio
The simulator calculates:
``` text
PDR =
delivered packets / generated packets
```
and reports it as:
``` text
PDR (%) =
100 × PDR
```
Higher is better.
---
53. Energy per Packet
For every hop:
``` text
energy =
energy_scale ×
(
    0.55
    + 0.008 × node degree
    + 0.004 × queue length
)
```
Final value:
``` text
total energy / delivered packets
```
Lower is better.
---
54. Fault Recovery Score
The supplied formula is:
``` text
FRS =
100 ×
(
    0.55 × PDR_fraction
    +
    0.45 × connectivity_ratio
)
```
Higher is better.
---
55. `experiments.py`
This module defines the experiments that produce the manuscript tables.
---
56. Experiment Constants
`TOPOLOGIES`
``` python
[
    "hypercube",
    "torus",
    "fat-tree",
    "dragonfly",
    "pdn"
]
```
`AI_METHODS`
``` python
[
    "shortest_path",
    "static_adaptive",
    "mlp",
    "gnn",
    "rl",
    "aicof"
]
```
`ABLATION_MAP`
``` python
[
    ("AICOF-Full", "aicof"),
    ("AICOF w/o RL", "aicof_no_rl"),
    ("AICOF w/o GNN", "aicof_no_gnn"),
    ("AICOF w/o MLP", "aicof_no_mlp"),
    ("AICOF w/o Congestion", "aicof_no_congestion"),
    ("AICOF w/o Fault-Awareness", "aicof_no_fault")
]
```
---
57. `avg_dict()`
``` python
avg_dict(dicts)
```
Purpose
Averages numerical metrics from repeated simulation runs.
Example:
``` python
[
    {"latency": 10.0, "pdr": 90.0},
    {"latency": 12.0, "pdr": 92.0}
]
```
produces approximately:
``` python
{
    "latency": 11.0,
    "pdr": 91.0
}
```
---
58. `_sim_task()`
``` python
_sim_task(task)
```
Purpose
Executes one simulation task.
Task structure:
``` python
(
    topology,
    requested_node_count,
    method,
    failure_ratio,
    repetition
)
```
The function:
Resolves valid node count.
Creates topology.
Calls `simulate_routing()`.
Uses a repetition-specific random seed.
---
59. `run_structural_table()`
``` python
run_structural_table(node_count=64)
```
Purpose
Generates structural topology metrics.
Topologies:
``` text
Hypercube
Torus
Fat-tree
Dragonfly
PDN
```
Metrics:
``` text
Node Degree
Diameter
Average Path Length
Path Diversity
Scalability
Fault Tolerance
```
Output
Pandas DataFrame.
---
60. `run_baseline_table()`
``` python
run_baseline_table(
    node_count=64,
    workers=1
)
```
Purpose
Evaluates traditional shortest-path routing.
Default failure ratio:
``` text
5%
```
from:
``` python
CFG.failure_ratio_baseline
```
Metrics:
``` text
Connectivity Ratio
Throughput
Average Latency
PDR
Energy / Packet
Fault Recovery Score
```
---
61. `run_scalability_tables()`
``` python
run_scalability_tables(workers=1)
```
Purpose
Studies performance as network size increases.
Default node sizes:
``` text
64
128
256
512
```
Default failure ratio:
``` text
2%
```
It returns:
``` python
latency_dataframe,
throughput_dataframe
```
---
62. `run_ai_vs_traditional()`
``` python
run_ai_vs_traditional(
    node_count=256,
    workers=1
)
```
Purpose
Compares:
``` text
Shortest Path
Static Adaptive
MLP
GNN
RL
AICOF
```
across all five topologies.
Default failure ratio:
``` text
8%
```
Metrics:
``` text
Connectivity Ratio
Throughput
Average Latency
PDR
Energy / Packet
Fault Recovery Score
```
---
63. `run_aicof_gain()`
``` python
run_aicof_gain(
    node_count=256,
    workers=1
)
```
Purpose
Calculates the percentage improvement of AICOF over shortest-path
routing for every topology.
For higher-is-better metrics:
``` text
gain =
(AICOF - baseline) / baseline × 100
```
For lower-is-better metrics:
``` text
gain =
(baseline - AICOF) / baseline × 100
```
The inverse formula is used for:
``` text
Latency
Energy
```
---
64. `run_ablation()`
``` python
run_ablation(
    node_count=256,
    workers=1
)
```
Purpose
Measures the contribution of individual AICOF components.
Variants:
``` text
AICOF-Full
AICOF w/o RL
AICOF w/o GNN
AICOF w/o MLP
AICOF w/o Congestion
AICOF w/o Fault-Awareness
```
This is the main experiment used to assess component contribution.
---
65. `parallel.py`
`run_parallel()`
``` python
run_parallel(
    func,
    tasks,
    workers=1
)
```
Purpose
Executes experiment tasks either sequentially or in parallel.
If:
``` text
workers <= 1
```
the function executes:
``` python
[func(t) for t in tasks]
```
If:
``` text
workers > 1
```
it creates a multiprocessing pool.
Example:
``` python
results = run_parallel(
    _sim_task,
    tasks,
    workers=4
)
```
---
66. `latex_export.py`
`df_to_latex_table()`
``` python
df_to_latex_table(
    df,
    caption,
    label,
    outpath
)
```
Purpose
Converts a pandas DataFrame into a LaTeX table.
The generated table has the structure:
``` latex
\begin{table*}[t]
\centering
\caption{...}
\label{...}
\begin{tabular}{...}
...
\end{tabular}
\end{table*}
```
Number formatting
Floating-point values are formatted to four decimal places and trailing
zeros are removed.
Missing values are written as:
``` text
--
```
---
67. `utils.py`
`log()`
``` python
log(msg)
```
Prints messages immediately:
``` python
log("[START] experiment")
```
---
`StageTimer`
``` python
with StageTimer("Experiment"):
    ...
```
Measures execution time.
Example output:
``` text
[START] Experiment
[DONE ] Experiment in 4.2 sec
```
If an exception occurs:
``` text
[FAIL ] Experiment after 4.2 sec: ...
```
---
`ensure_dir()`
``` python
ensure_dir(path)
```
Creates a directory if it does not exist.
Used by `run_all.py` to create:
``` text
results/
```
---
68. `run_all.py`
This is the main execution script.
Command:
``` bash
python run_all.py
```
---
69. Command-Line Arguments
`--quick`
Example:
``` bash
python run_all.py --quick
```
Changes:
``` text
repetitions = 4
packets_per_run = 500
max_steps_per_packet = 32
```
---
`--workers`
Example:
``` bash
python run_all.py --workers 4
```
Controls multiprocessing.
---
70. Main Six Experiment Stages
`run_all.py` executes:
``` text
1/6 Topology structural table
2/6 Baseline traditional-routing table
3/6 Scalability tables
4/6 AI vs traditional routing
5/6 AICOF gain table
6/6 Ablation table
```
---
71. Stage 1 --- Structural Table
``` python
run_structural_table(node_count=64)
```
Output:
``` text
results/topology_structural.csv
results/table_topology_structural.tex
```
---
72. Stage 2 --- Baseline Table
``` python
run_baseline_table(
    node_count=64,
    workers=args.workers
)
```
Output:
``` text
results/baseline_traditional.csv
results/table_baseline_traditional.tex
```
---
73. Stage 3 --- Scalability Tables
``` python
run_scalability_tables(
    workers=args.workers
)
```
Output:
``` text
results/scalability_latency.csv
results/table_scalability_latency.tex

results/scalability_throughput.csv
results/table_scalability_throughput.tex
```
---
74. Stage 4 --- AI vs Traditional
``` python
run_ai_vs_traditional(
    node_count=256,
    workers=args.workers
)
```
Output:
``` text
results/ai_vs_traditional.csv
results/table_ai_vs_traditional.tex
```
---
75. Stage 5 --- AICOF Gain
``` python
run_aicof_gain(
    node_count=256,
    workers=args.workers
)
```
Output:
``` text
results/aicof_gain.csv
results/table_aicof_gain.tex
```
---
76. Stage 6 --- Ablation
``` python
run_ablation(
    node_count=256,
    workers=args.workers
)
```
Output:
``` text
results/ablation.csv
results/table_ablation.tex
```
---
77. Complete Output Directory
After successful execution:
``` text
results/
│
├── topology_structural.csv
├── table_topology_structural.tex
│
├── baseline_traditional.csv
├── table_baseline_traditional.tex
│
├── scalability_latency.csv
├── table_scalability_latency.tex
│
├── scalability_throughput.csv
├── table_scalability_throughput.tex
│
├── ai_vs_traditional.csv
├── table_ai_vs_traditional.tex
│
├── aicof_gain.csv
├── table_aicof_gain.tex
│
├── ablation.csv
└── table_ablation.tex
```
---
78. Function-to-Module Summary
---
Function / Class                        Module                  Purpose
---
`ExperimentConfig`                      `config.py`             Global experiment
configuration
`CFG`                                   `config.py`             Configuration instance
`_v51_topology_multiplier()`            `routing.py`            Retrieve topology
multiplier
`_safe_norm()`                          `routing.py`            Safe normalization
`build_hypercube()`                     `topologies.py`         Generate hypercube
`build_torus()`                         `topologies.py`         Generate torus
`build_fat_tree()`                      `topologies.py`         Generate fat-tree
approximation
`build_dragonfly()`                     `topologies.py`         Generate Dragonfly-like
topology
`build_pdn()`                           `topologies.py`         Generate PDN
`nearest_power_of_two()`                `topologies.py`         Validate hypercube size
`nearest_perfect_square()`              `topologies.py`         Validate torus size
`resolve_valid_size()`                  `topologies.py`         Resolve
topology-compatible
size
`get_topology()`                        `topologies.py`         Topology factory
`sample_packet_pairs()`                 `traffic.py`            Generate traffic
`safe_average_shortest_path_length()`   `metrics.py`            Average shortest path
`structural_metrics()`                  `metrics.py`            Structural metrics
`connectivity_ratio()`                  `metrics.py`            Connectivity ratio
`RoutingPolicy`                         `routing.py`            Base routing class
`dist()`                                `routing.py`            Shortest-path distance
`local_fault_penalty()`                 `routing.py`            Fault penalty
`neighbor_resilience()`                 `routing.py`            Resilience feature
`two_hop_congestion()`                  `routing.py`            Congestion feature
`ShortestPathPolicy`                    `routing.py`            Traditional routing
`StaticAdaptivePolicy`                  `routing.py`            Adaptive routing
`MLPPolicy`                             `routing.py`            MLP-style routing
`GNNPolicy`                             `routing.py`            GNN-style routing
`RLPolicy`                              `routing.py`            RL-style routing
`AICOFPolicy`                           `routing.py`            Integrated AICOF
`build_policy()`                        `routing.py`            Policy factory
`apply_failures()`                      `simulator.py`          Inject edge failures
`simulate_routing()`                    `simulator.py`          Main packet simulation
`avg_dict()`                            `experiments.py`        Average repeated
results
`_sim_task()`                           `experiments.py`        Execute one simulation
task
`run_structural_table()`                `experiments.py`        Structural experiment
`run_baseline_table()`                  `experiments.py`        Baseline experiment
`run_scalability_tables()`              `experiments.py`        Scalability experiment
`run_ai_vs_traditional()`               `experiments.py`        AI comparison
`run_aicof_gain()`                      `experiments.py`        AICOF improvement
`run_ablation()`                        `experiments.py`        Ablation experiment
`run_parallel()`                        `parallel.py`           Multiprocessing
`df_to_latex_table()`                   `latex_export.py`       DataFrame → LaTeX
`log()`                                 `utils.py`              Logging
`StageTimer`                            `utils.py`              Stage timing
`ensure_dir()`                          `utils.py`              Directory creation
`save_with_latex()`                     `run_all.py`            Save CSV and LaTeX
`main()`                                `run_all.py`            Complete experiment
pipeline
---
79. Experiment-to-Paper Mapping
Paper experiment               Function                       Node size   Failure ratio
---
Structural topology analysis   `run_structural_table()`              64             N/A
Traditional baseline           `run_baseline_table()`                64              5%
Scalability                    `run_scalability_tables()`       64--512              2%
AI vs traditional              `run_ai_vs_traditional()`            256              8%
AICOF gain                     `run_aicof_gain()`                   256              8%
Ablation                       `run_ablation()`                     256              8%
---
80. Reproducibility and Random Seeds
The base seed is:
``` python
random_seed = 42
```
Each repetition uses:
``` python
CFG.random_seed + rep
```
Therefore repeated experiments use deterministic pseudo-random seeds
when the same source, configuration, Python version, and dependency
versions are retained.
For publication-quality reproducibility, preserve:
``` text
source code
configuration
requirements.txt
random seed
CSV results
LaTeX tables
figure-generation scripts
Python version
package versions
```
---
81. Important Scientific Interpretation
81.1 MLP
The current V5.1 source does not contain a trained MLP.
`MLPPolicy` is a weighted scoring model.
81.2 GNN
The current V5.1 source does not contain a trained GNN.
`GNNPolicy` uses graph-theoretic features such as:
``` text
betweenness
closeness
degree/resilience
two-hop congestion
distance
```
81.3 RL
The current V5.1 source does not contain PPO or another trained
reinforcement-learning agent.
`RLPolicy` is a reward-like weighted scoring function.
81.4 AICOF
The current AICOF implementation is an integrated mathematical routing
policy that combines these three score-based components plus congestion
and fault terms.
Therefore, the technically accurate description of this V5.1 package is:
> A simulation-based, AI-inspired/score-based AICOF routing framework
> rather than a framework containing independently trained MLP, GNN, and
> PPO models.
This distinction should be respected when describing the implementation
in a scientific manuscript.
---
82. Important Modeling Considerations
82.1 Method-specific failure ratio
The use of:
``` python
METHOD_FAIL_PENALTY
```
means that different routing methods can receive different effective
failure ratios.
This can make comparisons less controlled.
For a strict experiment, a stronger design is:
``` text
same topology
same failed edges
same traffic pairs
same random seeds
different routing policy
```
---
82.2 Method-specific latency/energy scaling
The use of:
``` python
METHOD_LOAD
```
means latency and energy are partly determined by manually assigned
method-specific scaling factors.
These factors should be clearly documented in the manuscript.
---
82.3 Throughput definition
The current throughput metric is:
``` text
delivered packets / maximum routing steps
```
This is a proxy rather than physical throughput.
---
82.4 Energy definition
Energy is calculated from a mathematical model involving:
``` text
method energy scale
node degree
queue length
```
It is not hardware-measured energy.
---
82.5 Structural path diversity
The current path-diversity value is:
``` text
network density × average node degree
```
This should not be interpreted as the number of edge-disjoint or
node-disjoint paths.
---
82.6 Scalability metric
The current structural scalability value is:
``` text
1 / average path length
```
It is a normalized structural indicator.
---
83. Recommended First Run
Because the supplied `config.py` contains the `CFG["v51_tuning"]` issue
described above, first correct that issue.
Then run:
``` bash
python run_all.py --quick --workers 1
```
If successful:
``` bash
python run_all.py --quick --workers 2
```
Then execute the full experiment:
``` bash
python run_all.py --workers 2
```
For a machine with sufficient CPU and memory:
``` bash
python run_all.py --workers 4
```
---
84. Manual Single-Topology Test
After fixing the configuration issue, a basic test is:
``` python
from config import CFG
from topologies import get_topology
from simulator import simulate_routing

G = get_topology("hypercube", 64)

result = simulate_routing(
    G,
    CFG,
    "aicof",
    failure_ratio=0.08,
    seed=42
)

print(result)
```
---
85. Manual Policy Comparison
``` python
methods = [
    "shortest_path",
    "static_adaptive",
    "mlp",
    "gnn",
    "rl",
    "aicof"
]

for method in methods:
    result = simulate_routing(
        G,
        CFG,
        method,
        failure_ratio=0.08,
        seed=42
    )
    print(method, result)
```
This is useful for validating individual routing policies before
executing the complete experiment suite.
---
86. Manual Ablation Test
``` python
methods = [
    "aicof",
    "aicof_no_rl",
    "aicof_no_gnn",
    "aicof_no_mlp",
    "aicof_no_congestion",
    "aicof_no_fault"
]

for method in methods:
    result = simulate_routing(
        G,
        CFG,
        method,
        failure_ratio=0.08,
        seed=42
    )
    print(method, result)
```
---
87. Troubleshooting
`TypeError` involving `CFG["v51_tuning"]`
Cause:
``` python
CFG = ExperimentConfig()
CFG["v51_tuning"] = ...
```
A dataclass is not a dictionary.
Fix by changing the tuning configuration to an attribute or by adding
the tuning values to the dataclass.
---
`ModuleNotFoundError`
Run:
``` bash
python -m pip install -r requirements.txt
```
---
Hypercube size error
Use a power of two:
``` text
64
128
256
512
1024
```
---
Torus size error
Use a perfect square:
``` text
64
100
144
196
256
324
400
484
576
```
---
Slow execution
Start with:
``` bash
python run_all.py --quick --workers 1
```
Then increase:
``` text
--workers 2
--workers 4
```
---
88. Recommended Publication Workflow
``` text
1. Correct/validate source
        ↓
2. Run quick validation
        ↓
3. Inspect generated CSV files
        ↓
4. Run full experiments
        ↓
5. Archive CSV files
        ↓
6. Generate figures from CSV results
        ↓
7. Generate LaTeX tables
        ↓
8. Update manuscript
        ↓
9. Verify every reported number
        ↓
10. Archive source + results + configuration
```
---
89. Result Validation Rule
Do not manually type previous manuscript values into the paper after
changing the simulation.
The preferred workflow is:
``` text
simulation
    ↓
CSV
    ↓
figure/table generation
    ↓
manuscript
```
The generated CSV should be treated as the numerical source of truth for
the final manuscript.
---
90. Final V5.1 File Map
``` text
config.py
    ExperimentConfig
    CFG
    V5.1 tuning configuration

topologies.py
    build_hypercube
    build_torus
    build_fat_tree
    build_dragonfly
    build_pdn
    nearest_power_of_two
    nearest_perfect_square
    resolve_valid_size
    get_topology

traffic.py
    sample_packet_pairs

metrics.py
    safe_average_shortest_path_length
    structural_metrics
    connectivity_ratio

routing.py
    _v51_topology_multiplier
    _safe_norm
    RoutingPolicy
    ShortestPathPolicy
    StaticAdaptivePolicy
    MLPPolicy
    GNNPolicy
    RLPolicy
    AICOFPolicy
    build_policy

simulator.py
    METHOD_LOAD
    METHOD_FAIL_PENALTY
    apply_failures
    simulate_routing

experiments.py
    TOPOLOGIES
    AI_METHODS
    ABLATION_MAP
    avg_dict
    _sim_task
    run_structural_table
    run_baseline_table
    run_scalability_tables
    run_ai_vs_traditional
    run_aicof_gain
    run_ablation

parallel.py
    run_parallel

latex_export.py
    df_to_latex_table

utils.py
    log
    StageTimer
    ensure_dir

run_all.py
    save_with_latex
    main
```
---
91. V5.1 Summary
The supplied V5.1 package is a compact NetworkX-based simulation
framework with six manuscript-level experiments.
Its major pipeline is:
``` text
Topology generation
        ↓
Traffic generation
        ↓
Failure injection
        ↓
Routing-policy selection
        ↓
Packet-level routing
        ↓
Metric calculation
        ↓
Repeated-run averaging
        ↓
CSV output
        ↓
LaTeX table output
```
The main scientific contribution represented by the source is the
integrated AICOF score-based routing policy, combining:
``` text
MLP-style local scoring
        +
GNN-style graph features
        +
RL-style adaptive scoring
        +
Congestion awareness
        +
Fault awareness
```
For publication, the distinction between score-based AI-inspired
components and genuinely trained MLP/GNN/PPO models should be made
explicit unless the implementation is subsequently upgraded to actual
trained models.
