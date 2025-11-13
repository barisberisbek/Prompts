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

--------------------------------------


The "Desktop GUI" Prompt (Tkinter + Threading)
Role: Act as a Senior Python Desktop Application Developer.

Objective: Convert the previously designed "Dynatrace Multi-Root Service Crawler" logic into a Standalone Desktop GUI Application.

Constraint: Do NOT use web frameworks like Streamlit or Flask. Use the standard tkinter library so the application is lightweight and requires minimal dependencies.

UI Layout Requirements:

Main Window: Title: "Dynatrace Topology Mapper v1.0".

Configuration Frame (Top or Left):

Tenant URL: Text entry (Default: https://{id}.live.dynatrace.com).

API Token: Text entry (Set show="*" to mask the token like a password field).

Max Depth: Spinbox or Entry (Default: 5).

Excluded Services: Entry (comma separated).

Input Area (Middle):

Label: "Root Services (Paste names, one per line)".

Widget: ScrolledText (Multi-line text box) to paste the list of L5/Entry services.

Action Area (Bottom):

"Start Mapping" Button: This triggers the process.

Progress Bar: To show activity (indeterminate mode is fine).

Log Console: A read-only ScrolledText area to display real-time logs (e.g., "Scanning: Mobile-Gateway...", "Edge added: A -> B").

Critical Technical Requirements (Must-Haves):

Threading (Anti-Freezing):

The crawling logic MUST run in a separate thread (threading.Thread).

The GUI must not freeze while the API calls are happening.

Thread-Safe Logging:

Since the crawling runs in a background thread, it cannot directly write to the GUI. Implement a thread-safe way (e.g., using a queue or root.after callbacks) to update the "Log Console" text widget from the background thread.

Error Handling:

If the API Token is invalid or the URL is wrong, show a messagebox.showerror popup.

Completion:

When finished, show a messagebox.showinfo popup saying "Graph Saved!" and enable the user to open the file location.

Deliverable: Provide the complete, single-file Python script (topology_gui.py) that includes the GUI setup, the threading logic, and the recursive crawler engine.