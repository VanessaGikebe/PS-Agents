# Exercise 3: Designing a Searching Agent
# GROUP MEMBERS
1. Atina kellie : 156976
2. Fadhili Njenga : 189979
3. Vanessa Njoki : 188972


Based on the class discussion on the self-driving trailer agent:

## Question 1

# PEAS Framework Design and Environmental Classification of a Self-Driving Trailer Agent

The PEAS framework breaks down an intelligent agent into four structural components—Performance measures, Environments, Actuators, and Sensors—which are required to map an abstract AI problem into a functional software and hardware architecture (Russell & Norvig, 2010). For a self-driving trailer agent operating as an autonomous or connected smart-trailer unit, the PEAS parameters are defined alongside its underlying task environment characteristics.

### 1. PEAS Framework Design

- Performance Measure (P): The system evaluates success based on safe navigation (minimizing collisions with vehicles, infrastructure, or pedestrians), cargo integrity and securement (preventing internal shifting, theft, or damage), route efficiency and punctuality (minimizing fuel or energy consumption and arrival delays), and regulatory compliance (adhering to speed limits, lane boundaries, and automated emergency stop protocols).
- Environment (E): The operational workspace consists of public highways, urban and rural thoroughfares, busy logistics hubs, tight loading docks, varying traffic densities, other human-driven or autonomous vehicles, pedestrians, and unpredictable meteorological events such as rain, snow, and crosswinds.
- Actuators (A): The mechanism by which the agent asserts physical control includes electronic steering and articulation systems, a dual-circuit braking system (service, parking, and emergency friction brakes), propulsion and acceleration overrides (if utilizing an integrated electric powertrain or communicating throttle adjustments to a lead tractor unit), and communication-based tools like automated signaling/hazard lights and fleet management telematics.
- Sensors (S): The perceptual inputs are captured through external surround-view and internal cargo-bay cameras, LiDAR and Radar arrays for three-dimensional proximity and hazard detection, GPS receivers and Inertial Measurement Units (IMUs) for tracking positioning, velocity, tilt, and articulation angles, alongside internal telematics and load sensors to monitor real-time tire pressure, axle weights, and securement statuses.

### 2. Task Environment Properties

The design of an intelligent agent’s program is directly dictated by the structural properties of its task environment. The self-driving trailer agent operates within an environment classified by the following parameters:

- Partially Observable: The trailer agent cannot perceive the global state of the roadway simultaneously; its sensor arrays are restricted by line-of-sight occlusions, heavy precipitation, blind spots along the trailer body, and hidden technical failures within a sealed cargo hull.
- Stochastic: The next state of the roadway environment is not completely determined by the agent's current state and execution of an action; random variables such as a sudden pedestrian crossing, an erratic human driver, or shifting traction on a wet surface introduce environmental uncertainty.
- Sequential: Every decision executed by the trailer agent—such as initiating a lane transition or modulating articulation braking—directly influences future states and navigation choices; the agent cannot treat its operations as isolated, independent episodes.
- Dynamic: The environment changes continuously while the agent is actively calculating its trajectory; surrounding traffic moves, crosswinds fluctuate, and the position of target objects shifts in real time regardless of the agent's computation speed.
- Continuous: The workspace operates over a non-discrete domain; physical positions, timestamps, steering metrics, velocity profiles, and incoming telemetry data consist of infinite, continuous real-valued ranges rather than countable steps.
- Multi-Agent: The agent operates alongside multiple competing and cooperative entities; it must predictively interact with other human-driven vehicles, fellow autonomous vehicles, dispatch systems, and pedestrians, which requires cooperative behaviors like zipper merging and defensive yielding.

## Question 2

# Kenya Highway Network State-Space Search Design

This design maps out the problem-solving environment for a self-driving trailer agent, structured using foundational artificial intelligence principles for **Problem Formulation**, **Path Cost Functions**, and **State-Space Abstraction** (Russell & Norvig, 2010).

---

## 1. Directed Graph Representation \(G = (V, E)\)

In state-space search, a task environment is formally modeled as a **directed network or graph** where the nodes represent states (cities/towns) and the links between them represent actions (highway travel paths). This graph charts all possible physical paths across the network before pruning or optimization is applied.

### A. Visual Network Diagram

```text
                                        [ CENTRAL HIGHLANDS REGION ]
                                           Nyeri
                                             ^
                                             | (B5: 25km)
  Nairobi ----A2----> Thika ----A2----> Karatina ----A2----> Nanyuki
  (Origin)                              
     |
     | (A8: 90km)
     v
  Naivasha ----A8----> Nakuru ----B1----> Kericho ----B1----> Kisumu [GOAL]
                       |                                         ^

                       | (A8: 150km)                             | (A1: 50km)
                       v                                         |
                    Eldoret -----------C39-----------> Kakamega --+
  
                     [ RIFT VALLEY & LAKE VICTORIA BASIN ]
```

### B. Formal Node Set (\(V\))

*   \(V\) = {Nairobi, Thika, Karatina, Nyeri, Nanyuki, Naivasha, Nakuru, Eldoret, Kericho, Kisumu, Kakamega}

### C. Directed Edge Weights Matrix (\(E\)) with Step Costs

A numeric step cost is assigned along the paths based on real-world kilometer distances between the nodes:

*   `Nairobi → Thika` via **Thika Superhighway / A2**: 45 km
*   `Thika → Karatina` via **A2 Highway**: 85 km
*   `Karatina → Nyeri` via **B5 Highway**: 25 km
*   `Karatina → Nanyuki` via **A2 Highway**: 60 km
*   `Nairobi → Naivasha` via **Nairobi–Malaba Highway / A8**: 90 km
*   `Naivasha → Nakuru` via **A8 Highway**: 70 km
*   `Nakuru → Eldoret` via **A8 Highway**: 150 km
*   `Nakuru → Kericho` via **Mau Summit–Kisumu Road / B1**: 107 km
*   `Kericho → Kisumu` via **B1 Highway**: 81 km
*   `Eldoret → Kakamega` via **Eldoret–Webuye Road / C39**: 95 km
*   `Kakamega → Kisumu` via **Kisumu–Kakamega Road / A1**: 50 km

---

## 2. Abstracted Search Tree

A search tree unrolls from the designated initial state (**Nairobi**) and branches outward via valid actions to evaluate distinct strategies for reaching the specified goal state (**Kisumu**). 

By definition, **Abstraction** involves systematically leaving out irrelevant information, meaning the agent should only consider states and actions that directly contribute to solving the search problem efficiently (Russell & Norvig, 2010). Because the goal state is **Kisumu**, the entire **Central Highlands** corridor (Thika, Karatina, Nyeri, Nanyuki) constitutes an irrelevant trajectory leading away from the target destination. This entire network cluster is therefore abstracted out of the active search tree expansion to conserve algorithmic memory.

Furthermore, following the principle that a rational agent should prune trajectories that accumulate excessive, suboptimal path costs, the tree branches are evaluated by their cumulative distance metrics:

```text
                                  [ ROOT NODE ]
                                     Nairobi
                                        |
                                        | (A8: 90km)
                                        v
                                    Naivasha
                                        |
                                        | (A8: 70km)
                                        v
                                     Nakuru
                                    /      \
                                   /        \
                    [ OPTIMAL BRANCH ]    [ SUBOPTIMAL BRANCH ]
                        Kericho                 Eldoret 

                           |                       |
                           | (B1: 81km)            | (C39: 95km)
                           v                       v
                        [ GOAL ]                Kakamega
                         Kisumu                    |
                      (Cost: 348km)                | (A1: 50km)
                                                   v
                                                [ GOAL ]
                                                 Kisumu
                                             (Cost: 455km)
```

### Cumulative Path Cost Calculations:

*   **Western Route A (via South Rift):** `Nairobi → Naivasha → Nakuru → Kericho → Kisumu`
    *   *Total Path Cost Calculation:* 90 km + 70 km + 107 km + 81 km = **348 km** *(Optimal Solution / Least Cost)*
*   **Western Route B (via North Rift):** `Nairobi → Naivasha → Nakuru → Eldoret → Kakamega → Kisumu`
    *   *Total Path Cost Calculation:* 90 km + 70 km + 150 km + 95 km + 50 km = **455 km** *(Suboptimal Solution)*

---

## 3. Application of the Abstraction Concept

This model strictly adheres to formal state-space formulation principles:

*   **State Abstraction:** The graph and tree omit thousands of real-world intermediate geographic locations, smaller trading centers (e.g., Limuru, Gilgil, Molo), and local intersections. Isolating only major regional logistics hubs ensures the search frontier queue remains clean and computationally light.
*   **Route and Target Abstraction:** Irrelevant sub-graphs that increase overall path length or move in opposition to the goal vector (such as the Central Highlands loops) are intentionally left out of the tree. This simplifies the state space into a straightforward directed tracking optimization problem.
*   **Action Abstraction:** High-level macro-actions like `Drive(Naivasha, Nakuru)` abstract away minute mechanical vehicle handling details (gear shifting, active lane keeping, braking for obstacles). This consolidates complex continuous physical movements into a single discrete branch link with a fixed numeric cost.

## Question 3

Using real-world driving data along the designated regional highways, the step costs (distances between adjacent cities/towns) are:

- Nairobi \(\rightarrow \) Naivasha: 90 km
- Naivasha \(\rightarrow \) Nakuru: 70 km
- Nakuru \(\rightarrow \) Kericho: 107 km
- Kericho \(\rightarrow \) Kisumu: 81 km
- Nakuru \(\rightarrow \) Eldoret: 150 km
- Eldoret \(\rightarrow \) Kakamega: 95 km
- Kakamega \(\rightarrow \) Kisumu: 50 km
- Nairobi \(\rightarrow \) Narok: 141 km
- Narok \(\rightarrow \) Bomet: 81 km
- Bomet \(\rightarrow \) Kisii: 83 km
- Kisii \(\rightarrow \) Kisumu: 106 km

Note on Abstraction the entire Central Highlands region (Thika, Karatina, Nyeri, Nanyuki) is abstracted out because it targets states leading completely away from the designated goal state of Kisumu.

### 2.Uniform-Cost Search Execution

Uniform-Cost Search manages a priority queue ordered by cumulative path cost, \(g(n)\). At each step, the node with the absolute lowest cost is popped from the frontier queue and expanded.

- Step 1: Expand [Nairobi] (Cost = 0)
    - Add to Frontier: Naivasha (90 km), Narok (141 km)
- Step 2: Pop [Naivasha] (Lowest cost: 90 km) and Expand
    - Add to Frontier: Nakuru (\(90 + 70 = 160\text{ km}\))
    - Frontier Queue State: [Narok: 141 km, Nakuru: 160 km]
- Step 3: Pop [Narok] (Lowest cost: 141 km) and Expand
    - Add to Frontier: Bomet (\(141 + 81 = 222\text{ km}\))
    - Frontier Queue State: [Nakuru: 160 km, Bomet: 222 km]
- Step 4: Pop [Nakuru] (Lowest cost: 160 km) and Expand
    - Add to Frontier: Kericho (\(160 + 107 = 267\text{ km}\)), Eldoret (\(160 + 150 = 310\text{ km}\))
    - Frontier Queue State: [Bomet: 222 km, Kericho: 267 km, Eldoret: 310 km]
- Step 5: Pop [Bomet] (Lowest cost: 222 km) and Expand
    - Add to Frontier: Kisii (\(222 + 83 = 305\text{ km}\))
    - Frontier Queue State: [Kericho: 267 km, Kisii: 305 km, Eldoret: 310 km]
- Step 6: Pop [Kericho] (Lowest cost: 267 km) and Expand
    - Add to Frontier: Kisumu (Goal Node A) (\(267 + 81 = \mathbf{348\text{ km}}\))
    - Frontier Queue State: [Kisii: 305 km, Eldoret: 310 km, Kisumu (A): 348 km]
- Step 7: Pop [Kisii] (Lowest cost: 305 km) and Expand
    - Add to Frontier: Kisumu (Goal Node B) (\(305 + 106 = \mathbf{411\text{ km}}\))
    - Frontier Queue State: [Eldoret: 310 km, Kisumu (A): 348 km, Kisumu (B): 411 km]
- Step 8: Pop [Eldoret] (Lowest cost: 310 km) and Expand
    - Add to Frontier: Kakamega (\(310 + 95 = 405\text{ km}\))
    - Frontier Queue State: [Kisumu (A): 348 km, Kakamega: 405 km, Kisumu (B): 411 km]
- Step 9: Pop [Kisumu (Goal Node A)] (Lowest cost: 348 km)
    - Goal Test Evaluation: Node matches target destination. Since UCS expands nodes in strict increasing order of optimal cost, this path is guaranteed to minimize distance. Search terminates successfully.

### 3. Answer

- Optimal Path: Nairobi \(\rightarrow \) Naivasha \(\rightarrow \) Nakuru \(\rightarrow \) Kericho \(\rightarrow \) Kisumu
- Minimum Total Path Cost: 348 km

---

## References

Russell, S., & Norvig, P. (2010). *Artificial intelligence: A modern approach* (3rd ed.). Prentice Hall.


# MEETING SCREENSHOT 
![Proof of discussion - Google meet screenshot](image.png)
