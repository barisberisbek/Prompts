The "Master SRE" Prompt (Multi-Root Topology)
Role: Act as a Senior Site Reliability Engineer (SRE) and Python Automation Architect.

Project Objective: I need to map the downstream dependency chain ("Service Flow") of a critical banking application. The traffic enters through multiple entry points (L5 Gateways) and flows downstream, often converging on shared backend services and databases.

I need a Python script that performs a "Multi-Root Recursive Crawl" using the Dynatrace API v2.

Context & The Challenge:

Input: Instead of a single root, I will provide a list of Service Names (e.g., ROOT_SERVICES = ["Service-A", "Service-B", "Service-C"]).

Convergence: These distinct flows often call the same shared backend services (e.g., a core DB). The script must handle this gracefully. If "Service-A" and "Service-B" both call "Service-Core", "Service-Core" should appear as a single node in the graph with multiple incoming edges.

Output: A single, unified .graphml file visualizing this entire interconnected ecosystem.

Technical Requirements (The Algorithm):

Global Graph & State:

Initialize a single networkx.DiGraph.

Maintain a global visited_nodes set that persists across all root crawls. This is critical to ensure shared services are not crawled multiple times and are linked correctly.

Resolution Phase:

Iterate through the ROOT_SERVICES list.

Resolve each name to its Dynatrace entityId.

Mark these specific nodes with a distinct attribute (e.g., color="red", type="root") in the graph for easy identification.

Recursive "Deep Dive" Function:

Implement a function crawl_dependencies(service_id, current_depth):

API Call: Fetch toRelationships.calls and properties.displayName.

Logic: For each outgoing call (target):

Add the node and edge to the global graph.

Convergence Check: If the target is already in the global visited_nodes, DO NOT recurse (stop there, just add the edge). This handles the merging paths efficiently.

If not visited and current_depth < MAX_DEPTH, add to visited and recurse.

Configuration & Safety:

Inputs:

DT_TENANT_URL & DT_API_TOKEN (from Env Vars).

ROOT_SERVICES (List of strings).

MAX_DEPTH (Integer, e.g., 5).

EXCLUDED_PREFIXES (List of strings, e.g., ["Dynatrace", "Splunk"] to skip monitoring noise).

Rate Limiting: Include time.sleep to respect API limits.

Deliverable: Provide the complete, production-ready Python script. Include comments explaining how the "Convergence/Shared Service" logic works.