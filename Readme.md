# Role: Senior Python Developer & SRE Tooling Expert

# Objective
I need a full-fledged Desktop GUI Application aimed at Site Reliability Engineers (SREs). The goal is to visualize service topologies automatically by fetching data from the Dynatrace API v2.

# Tech Stack Requirements
- **Language:** Python 3.10+
- **GUI Framework:** PyQt6 (Must use robust threading/QThread to prevent freezing during API calls).
- **Styling:** `qt_material` (Use 'dark_teal.xml' or similar modern dark theme).
- **Graph Visualization:** `pyvis` embedded inside a `QWebEngineView` widget.
- **Data Handling:** `requests` for API, `networkx` for graph logic (merging, pathfinding).

# Core Features & Logic

## 1. Multi-Root Logic with Intelligent Merging
- The user can input **multiple** starting service IDs (Root Nodes) separated by commas or new lines.
- **Scenario:** If User provides `RootA` and `RootB`.
    - If `RootB` is actually a downstream dependency of `RootA` (e.g., found at depth 3 of RootA), the graph should not duplicate nodes. It should merge them into a single connected graph.
    - If they are disjoint, show two separate clusters in the same view.
- Use a `NetworkX` DiGraph object backend to build the graph before rendering it to PyVis. This ensures mathematical correctness of nodes/edges.

## 2. Deep Recursion (Handling up to 20+ Hops)
- The depth slider/input must allow values up to **20**.
- **Crucial:** Implement a `visited` set logic to prevent infinite loops (circular dependencies where Svc A -> Svc B -> Svc A).
- **Performance:** Since 20 hops can fetch thousands of nodes, use a `Worker` thread (QThread) for the fetching process so the GUI doesn't freeze. Update a progress bar in the GUI as nodes are fetched.

## 3. Dynatrace API v2 Implementation
- Use `GET /api/v2/entities`
- Parameters to use:
    - `entitySelector`: `entityId("...")`
    - `fields`: `toRelationships.calls,fromRelationships.calls,properties.entityId,properties.displayName,properties.serviceType`
- Logic: Fetch the root, identify targets, and recursively fetch them up to the user-defined `max_depth`.

## 4. GUI Layout Specification
- **Sidebar (Left):**
    - **Connection:** Inputs for `Dynatrace URL` and `API Token`.
    - **Roots:** A multi-line text area (`QTextEdit`) for "Root Service IDs" (one per line).
    - **Controls:** - A slider or SpinBox for "Depth" (Range: 1-20).
        - A dropdown for "Direction" (Outgoing, Incoming, Both).
    - **Actions:** A large "Analyze Topology" button and an "Export to Excel" button.
    - **Status:** A Log/Console window at the bottom of the sidebar showing "Fetching Layer 1...", "Found 50 nodes..." etc.
- **Main Area (Right):**
    - The `QWebEngineView` displaying the PyVis interactive graph.
    - The graph should use physics for layout but offer a button to "Stop Physics" (stabilize) if it gets too wobbly with 500+ nodes.

# Edge Cases to Handle
1. **Circular Dependencies:** Do not crash if Service A calls Service B and Service B calls Service A. Just draw the edge and stop recursing that path.
2. **Duplicate Roots:** If the user inputs the same ID twice or inputs an ID that is discovered during traversal, handle it gracefully (Idempotency).
3. **Missing Permissions:** If API returns 401/403, show a clear popup alert.

# Deliverable
Please provide the complete, runnable Python code in a single file (or structured classes if too long). Include comments explaining the recursion logic and the NetworkX merging strategy.
