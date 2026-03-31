┌──────────────────────────────────────────────────────────────┐
│ START: python src/main.py report.pdf                         │
└──────────────────────────────────────────────────────────────┘
                         ↓
┌──────────────────────────────────────────────────────────────┐
│ main.py Line 101: run_analysis()                             │
│ - Validates file exists                                       │
│ - Creates workflow graph                                      │
│ - Prepares initial_state = {                                 │
│     "input": "...",                                           │
│     "inspection_report_path": "report.pdf",                   │
│     "messages": []                                            │
│   }                                                           │
└──────────────────────────────────────────────────────────────┘
                         ↓
┌──────────────────────────────────────────────────────────────┐
│ workflow.py: create_inspection_workflow()                     │
│ - Creates StateGraph(InspectionState)                         │
│ - Adds 3 nodes: router, analysis, policy_pull                │
│ - Sets entry_point = "router"                                │
│ - Adds conditional_edges from router                          │
│ - Adds direct_edges: analysis→router, policy_pull→router      │
│ - Compiles graph                                              │
└──────────────────────────────────────────────────────────────┘
                         ↓
┌──────────────────────────────────────────────────────────────┐
│ main.py Line 44: app.invoke(initial_state)                   │
│ LangGraph takes over!                                         │
└──────────────────────────────────────────────────────────────┘
                         ↓
╔══════════════════════════════════════════════════════════════╗
║ 🟢 NODE 1: router_node() - FIRST TIME                       ║
║ router.py Line 22                                            ║
║                                                               ║
║ INPUT: {"inspection_report_path": "report.pdf", ...}        ║
║                                                               ║
║ EXECUTION:                                                    ║
║ 1. Creates Claude LLM (Line 38)                             ║
║ 2. Builds prompt: "Analysis already completed: NO" (Line 41)║
║ 3. Claude responds: "analysis" (Line 48)                     ║
║ 4. Sets state["next_agent"] = "analysis" (Line 55)          ║
║                                                               ║
║ OUTPUT: {"next_agent": "analysis", ...}                      ║
╚══════════════════════════════════════════════════════════════╝
                         ↓
┌──────────────────────────────────────────────────────────────┐
│ ⚡ CONDITIONAL EDGE: route_decision()                        │
│ router.py Line 64                                             │
│                                                               │
│ INPUT: {"next_agent": "analysis"}                            │
│ EXECUTION: next = state.get("next_agent", "end")  # "analysis"│
│            return "analysis"                                  │
│ OUTPUT: "analysis"                                            │
│                                                               │
│ LangGraph: mapping["analysis"] → Go to "analysis" node       │
└──────────────────────────────────────────────────────────────┘
                         ↓
╔══════════════════════════════════════════════════════════════╗
║ 🔵 NODE 2: analysis_node()                                   ║
║ analysis.py Line 54                                          ║
║                                                               ║
║ INPUT: {"inspection_report_path": "report.pdf", ...}        ║
║                                                               ║
║ EXECUTION:                                                    ║
║ 1. Creates ReAct agent with 4 tools (Line 84)               ║
║ 2. Agent autonomously runs ReAct loop:                       ║
║    → Calls extract_inspection_report()                       ║
║    → Calls extract_ratings_variables()                       ║
║    → Calls compare_and_verify_ratings()                      ║
║    → Writes final analysis                                   ║
║ 3. Sets state["final_analysis"] = "..." (Line 139)          ║
║ 4. Sets state["next_agent"] = None (Line 140)               ║
║                                                               ║
║ OUTPUT: {"final_analysis": "...", "next_agent": None}       ║
╚══════════════════════════════════════════════════════════════╝
                         ↓
┌──────────────────────────────────────────────────────────────┐
│ ➡️  DIRECT EDGE: analysis → router                           │
│ workflow.py Line 45                                           │
│                                                               │
│ No decision needed, ALWAYS go to router                       │
└──────────────────────────────────────────────────────────────┘
                         ↓
╔══════════════════════════════════════════════════════════════╗
║ 🟢 NODE 3: router_node() - SECOND TIME                      ║
║ router.py Line 22                                            ║
║                                                               ║
║ INPUT: {"final_analysis": "...", "next_agent": None, ...}   ║
║                                                               ║
║ EXECUTION:                                                    ║
║ 1. Builds prompt: "Analysis already completed: YES" (Line 41)║
║ 2. Claude responds: "end" (Line 48)                          ║
║ 3. Sets state["next_agent"] = "end" (Line 55)               ║
║                                                               ║
║ OUTPUT: {"next_agent": "end", ...}                           ║
╚══════════════════════════════════════════════════════════════╝
                         ↓
┌──────────────────────────────────────────────────────────────┐
│ ⚡ CONDITIONAL EDGE: route_decision()                        │
│ router.py Line 64                                             │
│                                                               │
│ INPUT: {"next_agent": "end"}                                 │
│ EXECUTION: next = state.get("next_agent", "end")  # "end"   │
│            return "end"                                       │
│ OUTPUT: "end"                                                 │
│                                                               │
│ LangGraph: mapping["end"] = END → Terminate workflow         │
└──────────────────────────────────────────────────────────────┘
                         ↓
╔══════════════════════════════════════════════════════════════╗
║ 🛑 WORKFLOW ENDS                                             ║
║                                                               ║
║ Final state returned to main.py                              ║
╚══════════════════════════════════════════════════════════════╝
                         ↓
┌──────────────────────────────────────────────────────────────┐
│ main.py Line 49-64: Display results                          │
│ - Extracts final_analysis from result                        │
│ - Prints to console                                           │
└──────────────────────────────────────────────────────────────┘
                         ↓
┌──────────────────────────────────────────────────────────────┐
│ END: User sees analysis results                               │
└──────────────────────────────────────────────────────────────┘