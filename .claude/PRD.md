# Product Requirements Document: Revit MCP Demo

## 1. Executive Summary

Revit MCP is a proof-of-concept integration that connects Large Language Models (LLMs) to Autodesk Revit through the Model Context Protocol (MCP). This enables architects, engineers, and BIM professionals to interact with their Revit models using natural language—querying model data, visualizing views, placing elements, and even executing custom automation scripts through conversational AI.

### How It Works (Simple Version)

```
  You speak to Claude  →  Claude calls tools  →  Tools talk to Revit  →  Results come back
       (natural language)      (MCP protocol)        (HTTP/localhost)       (data/images)
```

The system runs entirely on each workstation—no cloud servers needed (except Anthropic's LLM API which Claude Code already uses). This means **every Triviron workstation with Revit can have AI-assisted BIM capabilities** by simply cloning this repository.

The demo targets executive stakeholders (CEO Phenyo and board members) to showcase the transformative potential of AI-assisted BIM workflows. By demonstrating that an LLM can "see" and manipulate a Revit model in real-time, we establish Triviron's position at the forefront of AEC technology innovation.

**MVP Goal:** Deliver a compelling 5-10 minute screen-recorded demo showing Claude interacting with a live Revit model—querying information, generating views, placing families, and executing custom code—all through natural conversation.

---

## 2. Mission

**Mission Statement:** Democratize Revit automation by enabling anyone to control and query BIM models through natural language, eliminating the barrier of API expertise.

**Core Principles:**
1. **Conversational First** — Users describe what they want; the AI figures out how
2. **Visual Proof** — Show, don't tell; export views and images as evidence
3. **Safe Experimentation** — All modifications are transactional and reversible
4. **Extensible Foundation** — Easy to add new tools as needs evolve
5. **Local-First Architecture** — Revit data stays local; only prompts go to LLM cloud

---

## 3. Target Users

### Primary Persona: Executive Stakeholder (Demo Audience)
- **Who:** CEO, board members, potential investors
- **Technical Level:** Low to moderate; familiar with Revit conceptually but not API details
- **Needs:** Understand the business value and innovation potential
- **Pain Points:** Difficulty visualizing how AI integrates with existing tools

### Secondary Persona: BIM Professional (Future User)
- **Who:** Architects, engineers, BIM managers using Revit daily
- **Technical Level:** Expert in Revit, limited programming experience
- **Needs:** Automate repetitive tasks, query model data quickly, generate reports
- **Pain Points:** Writing Dynamo scripts or C# plugins is time-consuming

### Tertiary Persona: Developer (Contributor)
- **Who:** Python developers interested in AEC/BIM automation
- **Technical Level:** High; comfortable with APIs and MCP
- **Needs:** Extend functionality, integrate with other systems
- **Pain Points:** Revit API has steep learning curve

---

## 4. MVP Scope

### ✅ In Scope (Demo Features)

**Core Connectivity**
- ✅ Health check and status verification
- ✅ Model information retrieval (title, path, element counts)
- ✅ List all levels with elevations

**Visual Capabilities**
- ✅ List all exportable views by type (floor plans, sections, 3D, etc.)
- ✅ Export any view as PNG image
- ✅ Display exported images directly in Claude conversation

**Element Queries**
- ✅ List available family categories with counts
- ✅ Search families by name filter
- ✅ Get comprehensive family type listings

**Model Manipulation**
- ✅ Place family instances at specific coordinates
- ✅ Apply color overrides to element categories (visualization)
- ✅ Clear color overrides

**Advanced Capability**
- ✅ Execute arbitrary IronPython code in Revit context
- ✅ Automatic transaction handling for modifications
- ✅ Detailed error messages with IronPython hints

**Demo Infrastructure**
- ✅ Screen recording workflow documentation
- ✅ Sample Revit model with diverse elements
- ✅ Scripted demo scenarios

### ❌ Out of Scope (Future Phases)

**Element Operations**
- ❌ Create walls, floors, roofs from scratch
- ❌ Modify element parameters (dimensions, materials)
- ❌ Delete elements
- ❌ Copy/move elements

**Documentation**
- ❌ Sheet creation and management
- ❌ Schedule generation
- ❌ Annotation placement

**Export/Import**
- ❌ IFC export
- ❌ DWG export
- ❌ Point cloud integration

**Collaboration**
- ❌ Worksharing/BIM 360 integration
- ❌ Multi-user sessions
- ❌ Cloud hosting of MCP server

**Security**
- ❌ Authentication for Routes API
- ❌ Role-based access control
- ❌ Audit logging

---

## 5. User Stories

### Demo Stories (Board Presentation)

**US-1: Model Awareness**
> As a board member watching the demo, I want to see Claude describe the current Revit model, so that I understand the AI can "see" the project context.

*Example:* "Claude, what model do I have open and what's in it?"
*Response:* Lists project name, file path, counts of walls/doors/windows/rooms.

**US-2: Visual Export**
> As a viewer, I want Claude to show me a specific view from Revit, so that I can see the AI retrieving actual project visuals.

*Example:* "Show me the Level 1 floor plan."
*Response:* Exports and displays the PNG image inline.

**US-3: Spatial Understanding**
> As a stakeholder, I want Claude to list all levels and their elevations, so that I see it understands the building's vertical organization.

*Example:* "What are all the levels in this building?"
*Response:* Table of levels with names and elevations.

**US-4: Family Discovery**
> As a viewer, I want Claude to find specific families in the model, so that I see it can search the BIM database.

*Example:* "Find all door families available in this project."
*Response:* Lists door families and types with details.

**US-5: Element Placement**
> As a stakeholder, I want to see Claude place a furniture family in the model, so that I understand the AI can modify designs.

*Example:* "Place a desk at coordinates (10, 5, 0) on Level 1."
*Response:* Confirms placement, optionally shows updated view.

**US-6: Visual Highlighting**
> As a viewer, I want Claude to highlight all elements of a category with a color, so that I see visual feedback of AI actions.

*Example:* "Highlight all walls in red."
*Response:* Applies color override, exports view showing red walls.

**US-7: Custom Automation**
> As a technically-curious board member, I want to see Claude write and execute custom code, so that I understand the extensibility.

*Example:* "Count all rooms and calculate total area."
*Response:* Executes IronPython, returns room count and summed area.

**US-8: Error Recovery**
> As a stakeholder, I want to see Claude handle errors gracefully, so that I trust the system's robustness.

*Example:* Request an invalid operation; Claude explains the issue clearly.

---

## 6. Core Architecture & Patterns

### High-Level Architecture

```
┌─────────────────────────────────────────────────────────────────────┐
│                    EACH TRIVIRON WORKSTATION                        │
│                                                                     │
│  ┌──────────────┐        ┌──────────────┐        ┌──────────────┐  │
│  │ Claude Code  │ stdio  │   FastMCP    │  HTTP  │    Revit     │  │
│  │   Desktop    │◄──────►│   Server     │◄──────►│  + pyRevit   │  │
│  │              │        │  (main.py)   │        │   Routes     │  │
│  │              │        │              │        │              │  │
│  └──────────────┘        └──────────────┘        └──────────────┘  │
│         │                                                           │
│         │ API calls                                                 │
│         ▼                                                           │
│  ┌─────────────────────────────────────────────────────────────┐   │
│  │                   Anthropic Cloud                            │   │
│  │                   (Sonnet 4.5 LLM)                          │   │
│  └─────────────────────────────────────────────────────────────┘   │
└─────────────────────────────────────────────────────────────────────┘
```

### Component Breakdown

| Component | Location | Role |
|-----------|----------|------|
| **Claude Code Desktop** | Local workstation | User interface; sends prompts to Anthropic cloud |
| **Sonnet 4.5 (LLM)** | Anthropic cloud | Processes natural language; decides which tools to call |
| **FastMCP Server (main.py)** | Local workstation | Translates MCP tool calls → HTTP requests to Revit |
| **pyRevit Routes API** | Inside Revit (local) | Executes Revit API commands; returns results |

### Why Everything Runs Locally (Except the LLM)

1. **Revit is a desktop application** — requires Windows, GPU, and license
2. **pyRevit Routes only listens on localhost:48884** — not accessible from internet
3. **FastMCP server must be on same machine** — to reach Revit's local API
4. **Only the LLM is cloud-hosted** — Claude Code handles this connection automatically

### Decision: Local vs Cloud-Hosted MCP Server

**We considered** hosting the FastMCP server in the cloud (e.g., at `revit.triviron.com`) so workstations wouldn't need local setup.

**We abandoned this approach** because:

| Challenge | Why It's Impractical |
|-----------|---------------------|
| **Revit runs locally** | Revit is a desktop app — it cannot run in the cloud without expensive virtual desktop infrastructure (VDI) |
| **pyRevit Routes is localhost-only** | The Routes API binds to `localhost:48884` and cannot be accessed over the network without complex tunneling |
| **Security concerns** | Exposing Revit's API to the internet would require authentication, encryption, and audit logging |
| **Latency** | Round-trip to cloud and back would slow down every tool call |
| **No real benefit** | Each workstation already has Revit — adding the MCP server locally is trivial |

**Conclusion:** The local architecture is simpler, faster, more secure, and works perfectly for Triviron's use case. Cloud hosting would add complexity without meaningful benefit.

### Directory Structure

```
revit-mcp-triviron/
├── main.py                      # MCP server entry point
├── tools/                       # MCP tool definitions
│   ├── __init__.py             # Central registration
│   ├── utils.py                # Response formatting
│   ├── status_tools.py         # Health checks
│   ├── model_tools.py          # Model queries
│   ├── view_tools.py           # View export
│   ├── family_tools.py         # Family operations
│   ├── colors_tools.py         # Color overrides
│   └── code_execution_tools.py # Dynamic code execution
├── revit_mcp/                   # pyRevit extension (IronPython)
│   ├── __init__.py
│   ├── status.py, model_info.py, views.py
│   ├── placement.py, colors.py, code_execution.py
│   └── utils.py                # IronPython helpers
├── startup.py                   # pyRevit route registration
├── docs/
│   └── Revit-mcp-demo-setup.md # Setup instructions
└── PRD.md                       # This document
```

### Key Design Patterns

1. **Modular Tool Registration**
   - Each tool category is a separate module
   - Central `register_tools()` function aggregates all
   - Easy to add new tool categories

2. **Two-Language Bridge**
   - MCP Server: Python 3.11+ (modern async)
   - Revit Extension: IronPython 2.7 (Revit API access)
   - HTTP REST bridges the gap

3. **Safe Property Access**
   - IronPython quirks handled via utility functions
   - `get_element_name_safe()`, `get_family_name_safe()`

4. **Transaction Wrapping**
   - All model modifications auto-wrapped in transactions
   - Rollback on error ensures model integrity

---

## 7. Tools/Features Specification

### Tool Catalog

| Tool | Purpose | Key Parameters |
|------|---------|----------------|
| `get_revit_status` | Verify Revit connection | None |
| `get_revit_model_info` | Get model metadata | None |
| `list_levels` | List all levels with elevations | None |
| `list_revit_views` | List exportable views by type | None |
| `get_revit_view` | Export view as PNG | `view_name` |
| `list_family_categories` | List categories with counts | None |
| `list_families` | Search families | `contains`, `limit` |
| `place_family` | Place family instance | `family_name`, `type_name`, `x`, `y`, `z`, `level_name` |
| `splash_color` | Apply color to category | `category_name`, `color` (RGB) |
| `clear_colors` | Remove color overrides | `category_name` |
| `execute_revit_code` | Run IronPython code | `code`, `description` |

### Feature: Code Execution (Highlight)

The `execute_revit_code` tool is the "escape hatch" that makes this system infinitely extensible:

```python
# Example: Count rooms and calculate total area
code = '''
from pyrevit import revit, DB
doc = revit.doc

rooms = DB.FilteredElementCollector(doc) \
    .OfCategory(DB.BuiltInCategory.OST_Rooms) \
    .WhereElementIsNotElementType() \
    .ToElements()

total_area = 0
for room in rooms:
    area_param = room.get_Parameter(DB.BuiltInParameter.ROOM_AREA)
    if area_param:
        total_area += area_param.AsDouble()

# Convert from sq ft to sq m
total_area_sqm = total_area * 0.092903

print("Room count: {}".format(len(rooms)))
print("Total area: {:.2f} sq m".format(total_area_sqm))
'''
```

**Safety Features:**
- Auto-wrapped in transaction (rollback on error)
- Stdout capture returns print output
- Detailed error hints for common IronPython issues

---

## 8. Technology Stack

### MCP Server (Python 3.11+)

| Component | Package | Version |
|-----------|---------|---------|
| MCP Framework | `mcp[cli]` | ≥1.9.0 |
| HTTP Client | `httpx` | (bundled) |
| ASGI Server | `uvicorn` | (bundled) |
| Async Runtime | `anyio` | (bundled) |

### Revit Extension (IronPython 2.7)

| Component | Source |
|-----------|--------|
| pyRevit | Installed separately |
| Revit API | `Autodesk.Revit.DB` |
| Routes Module | `pyrevit.routes` |

### Development Tools

| Tool | Purpose |
|------|---------|
| UV | Python package management |
| Git | Version control |
| Claude Code | AI-assisted development |

---

## 9. Security & Configuration

### Current Security Model

**Scope:** Local-only operation
- FastMCP server communicates via stdio (no network port exposed)
- pyRevit Routes binds to `localhost:48884` (not accessible from network)
- LLM API calls go to Anthropic cloud (secured by API key)

**Configuration:**
```python
# main.py
REVIT_HOST = "localhost"
REVIT_PORT = 48884  # Default pyRevit Routes port
```

### Security Considerations

| Aspect | Current State | Future Consideration |
|--------|---------------|---------------------|
| Authentication | None (local only) | Add API keys for Routes |
| Authorization | Full access | Role-based tool restrictions |
| Code Execution | Unrestricted | Sandboxing, code review |
| Audit Trail | Logging only | Persistent audit log |

### Demo Security

For the demo, security is not a concern:
- Everything runs on presenter's local machine
- No network exposure beyond standard Anthropic API calls
- No sensitive data leaves the system (except prompts to LLM)

---

## 9.1 Triviron Deployment Guide

### Deployment Model

Each Triviron workstation runs the full stack locally. There is **no central server** — each machine is self-contained.

```
  Workstation A              Workstation B              Workstation C
┌─────────────────┐      ┌─────────────────┐      ┌─────────────────┐
│ Claude Desktop  │      │ Claude Desktop  │      │ Claude Desktop  │
│ FastMCP Server  │      │ FastMCP Server  │      │ FastMCP Server  │
│ Revit + pyRevit │      │ Revit + pyRevit │      │ Revit + pyRevit │
└────────┬────────┘      └────────┬────────┘      └────────┬────────┘
         │                        │                        │
         └────────────────────────┼────────────────────────┘
                                  │
                                  ▼
                    ┌─────────────────────────┐
                    │    Anthropic Cloud      │
                    │    (Shared LLM API)     │
                    └─────────────────────────┘
```

### Per-Workstation Setup

#### Prerequisites
- Windows 10/11
- Autodesk Revit 2023+ (licensed)
- pyRevit installed ([pyrevitlabs.io](https://pyrevitlabs.io))
- Python 3.11+ 
- Git

#### Installation Steps

```bash
# 1. Clone the repository
git clone https://github.com/triviron/revit-mcp-triviron.git
cd revit-mcp-triviron

# 2. Install Python dependencies
pip install uv
uv sync

# 3. Copy pyRevit extension to pyRevit extensions folder
# (or symlink for easier updates)
xcopy /E revit_mcp "%APPDATA%\pyRevit\Extensions\revit_mcp.extension\lib\revit_mcp"
copy startup.py "%APPDATA%\pyRevit\Extensions\revit_mcp.extension\startup.py"

# 4. Enable pyRevit Routes
# Open Revit → pyRevit tab → Settings → Routes → Enable Server (port 48884)

# 5. Restart Revit
```

#### Configure Claude Code Desktop

Add to Claude Code Desktop MCP settings (`%APPDATA%\Claude\claude_desktop_config.json`):

```json
{
  "mcpServers": {
    "revit": {
      "command": "uv",
      "args": ["run", "python", "C:/path/to/revit-mcp-triviron/main.py"],
      "env": {}
    }
  }
}
```

#### Verify Installation

1. Open Revit with a model
2. Open Claude Code Desktop
3. Ask: "Are you connected to Revit?"
4. Should return status with model info

### Deployment Options

| Option | Pros | Cons |
|--------|------|------|
| **Git clone per workstation** | Easy updates via `git pull` | Requires Git knowledge |
| **Shared network folder** | Single source of truth | Network dependency |
| **MSI installer** (future) | One-click install | Requires building installer |

### Keeping Workstations Updated

```bash
# On each workstation, pull latest changes
cd revit-mcp-triviron
git pull origin master
uv sync  # Update dependencies if needed
# Restart Revit and Claude Code Desktop
```

---

## 10. API Specification

### MCP Tools → Revit Routes Mapping

| MCP Tool | HTTP Method | Endpoint |
|----------|-------------|----------|
| `get_revit_status` | GET | `/revit_mcp/status/` |
| `get_revit_model_info` | GET | `/revit_mcp/model_info/` |
| `list_levels` | GET | `/revit_mcp/list_levels/` |
| `list_revit_views` | GET | `/revit_mcp/list_views/` |
| `get_revit_view` | GET | `/revit_mcp/get_view/<view_name>` |
| `list_family_categories` | GET | `/revit_mcp/list_family_categories/` |
| `list_families` | GET | `/revit_mcp/list_families/?contains=&limit=` |
| `place_family` | POST | `/revit_mcp/place_family/` |
| `splash_color` | POST | `/revit_mcp/splash_color/` |
| `clear_colors` | POST | `/revit_mcp/clear_colors/` |
| `execute_revit_code` | POST | `/revit_mcp/execute_code/` |

### Example: Place Family Request

```json
POST /revit_mcp/place_family/
Content-Type: application/json

{
  "family_name": "Desk",
  "type_name": "1525 x 762mm",
  "x": 10.0,
  "y": 5.0,
  "z": 0.0,
  "level_name": "Level 1",
  "rotation": 0,
  "properties": {}
}
```

### Example: Code Execution Request

```json
POST /revit_mcp/execute_code/
Content-Type: application/json

{
  "code": "print('Hello from Revit!')\nprint('Model: ' + doc.Title)",
  "description": "Hello world test"
}
```

---

## 11. Success Criteria

### Demo Success Definition

The demo is successful if board members:
1. **Understand** the core value proposition (AI + BIM)
2. **See** real-time interaction with a live Revit model
3. **Appreciate** the extensibility (code execution)
4. **Envision** future applications and business potential

### Functional Requirements

**Connectivity**
- ✅ MCP server starts without errors
- ✅ Successfully connects to Revit via pyRevit Routes
- ✅ Status check returns positive response

**Query Operations**
- ✅ Model info returns accurate data
- ✅ Level listing matches Revit UI
- ✅ View listing includes all exportable views
- ✅ Family search returns relevant results

**Visual Operations**
- ✅ View export produces valid PNG
- ✅ Image displays correctly in Claude
- ✅ Color splash visually changes model

**Modification Operations**
- ✅ Family placement creates element at correct location
- ✅ Code execution returns output
- ✅ Errors are handled gracefully with clear messages

### Quality Indicators

- Response time < 5 seconds for queries
- Response time < 15 seconds for view exports
- No crashes during demo
- Clear, conversational responses from Claude

---

## 12. Implementation Phases

### Phase 1: Demo Preparation (Current)
**Goal:** Ensure all existing features work reliably for demo

**Deliverables:**
- ✅ Verify all 11 tools function correctly
- ✅ Prepare sample Revit model with diverse content
- ✅ Write demo script with specific prompts
- ✅ Test full demo flow end-to-end
- ✅ Create setup documentation

**Validation:** Complete dry-run of demo without issues

---

### Phase 2: Demo Enhancement (If Time Permits)
**Goal:** Add impressive demo features

**Deliverables:**
- ⬜ Add `get_element_by_id` tool for detailed element info
- ⬜ Add `list_rooms` tool with area calculations
- ⬜ Add `create_3d_view` tool for dynamic 3D generation
- ⬜ Improve view export quality/resolution

**Validation:** New features work in demo flow

---

### Phase 3: Post-Demo Polish
**Goal:** Refine based on feedback

**Deliverables:**
- ⬜ Address feedback from board presentation
- ⬜ Improve error messages and edge cases
- ⬜ Add comprehensive documentation
- ⬜ Create video tutorial

**Validation:** Positive stakeholder feedback

---

### Phase 4: Production Readiness (Future)
**Goal:** Prepare for broader use

**Deliverables:**
- ⬜ Add authentication to Routes API
- ⬜ Implement audit logging
- ⬜ Create installer/setup script
- ⬜ Add more element manipulation tools
- ⬜ Performance optimization

**Validation:** Deployed to pilot users

---

## 13. Future Considerations

### Post-Demo Enhancements

**Element Operations**
- Create/modify/delete walls, floors, doors, windows
- Bulk parameter updates
- Element duplication and arrays

**Documentation Automation**
- Auto-generate sheets from templates
- Create schedules from natural language
- Place annotations and dimensions

**Analysis Integration**
- Energy analysis queries
- Clash detection summaries
- Quantity takeoffs

### Integration Opportunities

- **Dynamo:** Execute Dynamo scripts via MCP
- **Power BI:** Export data for dashboards
- **BIM 360:** Cloud model access (requires auth)
- **Other CAD:** Similar MCP servers for AutoCAD, Rhino

### Advanced Features

- Multi-model coordination
- Version comparison ("What changed since last week?")
- Natural language Revit macros ("Every time I open this model, list the warnings")

---

## 14. Risks & Mitigations

| Risk | Impact | Likelihood | Mitigation |
|------|--------|------------|------------|
| **Revit crashes during demo** | High | Low | Test extensively; have backup recording |
| **pyRevit Routes not responding** | High | Medium | Verify setup; restart Revit before demo |
| **Slow response times** | Medium | Medium | Use simple model; pre-warm connections |
| **Claude misunderstands request** | Low | Medium | Use scripted prompts; demo multiple attempts |
| **IronPython code errors** | Medium | Medium | Test all code snippets beforehand |

### Contingency Plan

If live demo fails:
1. Switch to pre-recorded backup video
2. Explain the technical issue briefly
3. Continue with slides showing capabilities
4. Offer follow-up live demo after troubleshooting

---

## 15. Appendix

### A. Demo Script Outline

1. **Introduction** (30 sec)
   - "Let me show you how AI can interact with Revit"

2. **Connection Check** (30 sec)
   - "Claude, are you connected to Revit?"
   - Show status response

3. **Model Overview** (1 min)
   - "What model do I have open?"
   - "List all the levels"

4. **Visual Export** (1 min)
   - "Show me the Level 1 floor plan"
   - Display exported image

5. **Family Search** (1 min)
   - "What furniture families are available?"
   - "Find all chair types"

6. **Element Placement** (1.5 min)
   - "Place a chair at coordinates (5, 5, 0) on Level 1"
   - Export view to show placement

7. **Visual Highlighting** (1 min)
   - "Highlight all doors in blue"
   - Export view showing color override

8. **Code Execution** (2 min)
   - "Count all rooms and their total area"
   - Show custom code execution

9. **Wrap-up** (1 min)
   - Summarize capabilities
   - Discuss future potential

### B. Setup Checklist

- [ ] Revit 2024 installed and licensed
- [ ] pyRevit installed with Routes enabled (port 48884)
- [ ] Python 3.11+ installed
- [ ] UV package manager installed
- [ ] Project dependencies installed (`uv sync`)
- [ ] Sample Revit model prepared
- [ ] Claude Code Desktop configured with MCP server
- [ ] Test all demo prompts
- [ ] Record backup video

### C. Related Documents

- `docs/Revit-mcp-demo-setup.md` — Detailed setup instructions
- `LLM.txt` — Technical context for AI assistants
- `README.md` — Project overview and quick start

### D. Repository

- **GitHub:** (Add repository URL)
- **Branch:** master
- **Latest Commit:** 1cd2819 (added claude code template)

---

*Document Version: 1.0*
*Created: 2026-02-02*
*Author: Claude Code + Human Collaboration*

