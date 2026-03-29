# GSoC 2026 Proposal: Improve Getting Started Documentation

**Project:** #5 — Improve Getting Started Documentation  
**Organization:** dora-rs  
**Mentors:** Ning Huang, Yu Huang  
**Applicant:** Mrugaja  
**Size:** 175 hours | **Difficulty:** Medium  

---

## Table of Contents

1. [Abstract](#1-abstract)
2. [The Problem — Why This Project Matters](#2-the-problem--why-this-project-matters)
3. [Documentation Audit — What I Found](#3-documentation-audit--what-i-found)
4. [Proposed Solution — What I Will Build](#4-proposed-solution--what-i-will-build)
5. [Deliverables — Concrete Outputs](#5-deliverables--concrete-outputs)
6. [Sample Deliverable — Python Hello World Tutorial](#6-sample-deliverable--python-hello-world-tutorial)
7. [Sample Deliverable — Core Concepts Page](#7-sample-deliverable--core-concepts-page)
8. [Implementation Timeline — Week by Week](#8-implementation-timeline--week-by-week)
9. [Technical Approach](#9-technical-approach)
10. [About Me](#10-about-me)
11. [Why I Am the Right Fit](#11-why-i-am-the-right-fit)
12. [Availability and Communication](#12-availability-and-communication)

---

## 1. Abstract

dora-rs has excellent engineering — 10-17x faster than ROS2, zero-copy shared memory, a growing hub of pre-built nodes, multi-language support. But its documentation doesn't reflect this. A beginner landing on the docs today will hit broken code blocks, obsolete CLI commands, undefined concepts, and two contradictory documentation sites with no cross-links.

This project fixes that. In 175 hours over 12 weeks, I will:

1. **Audit** every onboarding surface and catalog every bug, gap, and dead-end.
2. **Fix** all critical blockers — broken commands, formatting bugs, obsolete instructions.
3. **Write** a complete Python-first onboarding path: install → hello world → real demo → next steps.
4. **Create** a Core Concepts page that defines every term the docs assume you already know.
5. **Build** a FAQ/Troubleshooting page from real user questions.
6. **Produce** a Rust quickstart guide (secondary priority, per maintainer guidance).
7. **Restructure** navigation to eliminate dead-ends and create a linear learning path.

The goal: a beginner goes from `pip install dora-rs-cli` to a running multi-node dataflow in under 15 minutes, with understanding of what they just did.

---

## 2. The Problem — Why This Project Matters

### 2.1 The Current State

dora-rs has two documentation sites. Neither works well for beginners.

| Surface | URL | State |
|---|---|---|
| **Old site (mdBook)** | `dora-rs.ai/dora/` | All code blocks render on a single line. Run command is obsolete (`dora-daemon`). 100% Rust — zero Python. Install commands have `<version>` placeholders. |
| **New site (Docusaurus)** | `dora-rs.ai/docs/guides/` | Better structure. But: commands concatenated on one line, no "what is dora?" preamble, no concept definitions, no next-steps links. |
| **GitHub README** | `github.com/dora-rs/dora` | Best overview of features. Links to new site. But the README is not a tutorial — it's a showcase. |

**Neither site links to the other.** A beginner landing on the old site (which Google still indexes highly) will follow broken instructions and give up. A beginner landing on the new site will follow steps without understanding what they're doing.

### 2.2 What This Costs the Project

| Impact | Evidence |
|---|---|
| **Lost users** | A Python developer who lands on the old Getting Started page sees Cargo.toml, `cargo new`, Rust match statements. They close the tab. |
| **Lost contributors** | GSoC applicants (like me) struggle to build dora from docs alone. Every applicant in the 2026 GitHub Discussions asked basic architectural questions that should be answered on a Concepts page. |
| **Support burden** | Questions that belong in documentation end up in Discord and GitHub Discussions — "what's the difference between a node and an operator?", "how do I run a Python dataflow?", "what does dora/timer/millis/100 mean?" |

### 2.3 The Opportunity

The maintainers have explicitly stated the priority order:

> **Python quickstart > Rust quickstart > C/C++ guides**

The new Docusaurus site already has the right bones — Python Conversation guide, Webcam Plot, YOLO, LLMs. What's missing is:

1. **Foundations** — define terms before using them
2. **Bug fixes** — commands that actually work on copy-paste
3. **Navigation** — a guided path from zero to working application
4. **Consolidation** — one source of truth, not two

This project provides all four.

---

## 3. Documentation Audit — What I Found

I performed a deep audit of all 8 documentation surfaces listed in the project description. Below is a summary. The full audit report (with exact page references, corrected code, and fix recommendations) is available separately.

### 3.1 Critical Bugs (13 Found)

| Category | Count | Worst Example |
|---|---|---|
| **Broken shell commands** | 5 | `uv venv --seed -p 3.11source .venv/bin/activate` — two commands on one line, shell error at step 1 of the Python guide |
| **Obsolete CLI commands** | 1 | `dora-daemon --run-dataflow dataflow.yml` — correct command is `dora run dataflow.yml` |
| **Code rendering failures** | 2 | Every code block on the old Getting Started page collapses to a single unreadable line |
| **Missing dependencies** | 2 | YOLOv5 example never tells user to install torch, cv2, numpy — instant `ModuleNotFoundError` |
| **Misleading content** | 3 | "early development" disclaimer on a project with 3.1k stars; comment says "20 seconds" but code runs for 10 |

### 3.2 Concept Gaps (9 Found)

Things assumed everywhere, explained nowhere:

| Concept | Used On | Defined On |
|---|---|---|
| Dataflow | Every single page | Zero pages |
| Node vs Operator | Python API, Overview, Getting Started | Overview page — but introduced simultaneously with no recommendation |
| `dora/timer/millis/N` | Every YAML example | Zero pages |
| Apache Arrow / `pa.array()` | Every Python example | Zero pages |
| `dora build` vs `dora run` | New Python guide | Zero pages |
| dora-hub (pre-built nodes) | GitHub README | Not in any tutorial flow |
| Why dora exists (motivation) | — | Zero pages — no "what problem does this solve" section |
| Coordinator / Daemon | Error messages, architecture page | Overview page — but as dense paragraphs with no beginner framing |
| `dora.builder` API | GitHub changelog | No documentation page exists |

### 3.3 Structural Problems (8 Found)

| Problem | Impact |
|---|---|
| Two separate doc sites, no cross-links | Beginners follow outdated instructions |
| No prerequisites callout on any page | Users get stuck installing Rust when they only need Python |
| No "Next Steps" links on any page | Every page is a dead-end |
| No FAQ / troubleshooting page | Common errors have no documented solutions |
| No version stamps on examples | Users don't know if their dora version is compatible |
| Python API buried at sidebar item #7 | Priority audience has to scroll past Rust and C APIs |
| "Brainstorming Ideas" in public sidebar | Internal design docs confuse beginners |
| No language tags on old site code blocks | No syntax highlighting anywhere |

---

## 4. Proposed Solution — What I Will Build

### 4.1 Solution Architecture

I'm building toward this onboarding flow:

```
Step 0: "What is dora?" (30s)
   │    One paragraph + architecture diagram
   ▼
Step 1: Install (2 min)
   │    pip install dora-rs-cli
   ▼
Step 2: Hello World (5 min)
   │    2 Python files + 1 YAML → dora run → see output
   ▼
Step 3: Understand it (5 min)
   │    Annotated walkthrough: dataflow.yml, timers, Arrow, events
   ▼
Step 4: Real-world demo (10 min)
   │    Webcam → YOLO → Rerun (using hub nodes)
   ▼
Step 5: Where to go next
   │    Signpost page → tutorials, hub, architecture, Rust, ROS2
   ▼
Advanced guides (self-directed)
```

### 4.2 Work Breakdown

| Workstream | What It Includes | Hours | Priority |
|---|---|---|---|
| **WS1: Critical Bug Fixes** | Fix all 13 documented bugs. Fix concatenated commands, obsolete CLI references, broken code rendering, add missing dependency installs. | 15 | P0 |
| **WS2: Python Hello World Tutorial** | Complete, runnable talker→listener tutorial with annotations. Includes: prerequisites, install, code, YAML, expected output, what-just-happened explainer. | 20 | P0 |
| **WS3: Core Concepts Page** | Definitions + diagrams for: dataflow, node, operator, timer source, Arrow, coordinator, daemon, dora-hub. Architecture diagram. | 20 | P0 |
| **WS4: Navigation & Structure** | Add "Next Steps" to every page. Add prerequisites callouts. Add version stamps. Remove "Brainstorming Ideas" from sidebar. Add old-site redirect banners. | 15 | P0 |
| **WS5: Improve Existing Guides** | Fix Python Conversation guide (add preamble, fix formatting). Improve Webcam Plot guide. Update YOLO guide. Ensure all code blocks have language tags. | 20 | P1 |
| **WS6: FAQ & Troubleshooting** | Create FAQ page from GitHub issues, Discord questions, and common errors. Cover: install failures, port conflicts, version mismatches, ModuleNotFoundError. | 15 | P1 |
| **WS7: Rust Quickstart** | Write a Rust getting-started tutorial (secondary priority per maintainer guidance). Clean, working Cargo setup + dataflow. | 20 | P1 |
| **WS8: Node Hub Quick Reference** | One-page table of all hub nodes with descriptions, install commands, and example YAML snippets. | 10 | P2 |
| **WS9: dora.builder Guide** | Tutorial for the Python dataflow builder API. Code-defined dataflows instead of YAML. | 15 | P2 |
| **WS10: Example Dataflows** | 3-5 complete, tested example dataflows with README, YAML, and all source files. Covering: basic chat, webcam+YOLO, LLM conversation, multi-node pipeline. | 15 | P2 |
| **WS11: Testing & CI** | Ensure all code examples are tested against dora `main`. Set up or extend CI to catch doc code rot. | 10 | P2 |
| **Total** | | **175** | |

### 4.3 Priority Order

Following maintainer guidance:

1. **P0 (Must-have):** WS1 + WS2 + WS3 + WS4 = 70 hours — Fix critical bugs, create Python hello world, define core concepts, fix navigation
2. **P1 (Should-have):** WS5 + WS6 + WS7 = 55 hours — Improve existing guides, create FAQ, write Rust quickstart
3. **P2 (Nice-to-have):** WS8 + WS9 + WS10 + WS11 = 50 hours — Hub reference, builder guide, examples, CI

If time runs short, P0 alone delivers massive value. P1 adds depth. P2 is stretch.

---

## 5. Deliverables — Concrete Outputs

Every deliverable is a pull request to [dora-rs/dora-rs.github.io](https://github.com/dora-rs/dora-rs.github.io) (the Docusaurus site source) unless otherwise noted.

| # | Deliverable | Format | Location |
|---|---|---|---|
| D1 | **Bug fix PR** — Fix all 13 documented bugs across both sites | Markdown edits | Old site + new site |
| D2 | **"What is dora?" page** — 30-second intro + architecture diagram | Docusaurus MDX | `/docs/getting-started/what-is-dora.md` |
| D3 | **Python Hello World tutorial** — Complete, annotated, runnable | Docusaurus MDX | `/docs/getting-started/hello-world-python.md` |
| D4 | **Core Concepts page** — All term definitions + diagrams | Docusaurus MDX | `/docs/concepts/core-concepts.md` |
| D5 | **Improved Python Conversation guide** — Fixed formatting, added preamble | Docusaurus MDX edit | `/docs/guides/getting-started/conversation_py.md` |
| D6 | **FAQ / Troubleshooting page** | Docusaurus MDX | `/docs/faq.md` |
| D7 | **Rust Quickstart tutorial** | Docusaurus MDX | `/docs/getting-started/hello-world-rust.md` |
| D8 | **Node Hub Quick Reference** | Docusaurus MDX | `/docs/nodes/quick-reference.md` |
| D9 | **dora.builder tutorial** | Docusaurus MDX | `/docs/tutorials/builder-api.md` |
| D10 | **3-5 example dataflows** with README + tested code | Python/YAML/Rust files | `examples/` in main repo |
| D11 | **Navigation improvements** — Next Steps links, prerequisites, version stamps, old-site banners | Markdown edits | Site-wide |
| D12 | **CI test for doc code examples** | GitHub Actions YAML | `.github/workflows/` |
| D13 | **Audit report** — Full documentation review with page references and fix recommendations | Markdown | Submitted with proposal |

---

## 6. Sample Deliverable — Python Hello World Tutorial

> This is a draft of deliverable D3. It demonstrates my writing quality and adherence to the mandatory rules.

---

### Your First dora Dataflow (Python)

Build a two-node dataflow where a Talker sends greetings and a Listener prints them.

:::note Prerequisites
- **Python 3.11+** — [Download](https://www.python.org/downloads/)
- **pip** — Comes with Python
- **OS:** Linux, macOS, or Windows
- **No Rust required** — this tutorial is Python-only
:::

:::tip Tested with
dora v0.5.0, Python 3.11, Ubuntu 22.04 / macOS 14 / Windows 11
:::

#### What You'll Build

Two programs connected by dora:

```
┌──────────┐  greeting   ┌───────────┐
│  Talker  │────────────▶│ Listener  │
│ (Python) │             │ (Python)  │
└──────────┘             └───────────┘
     ▲
     │ tick (every 1 second)
     │
  [dora timer]
```

- The **Talker** wakes up every second (triggered by a built-in dora timer), and sends a greeting.
- The **Listener** receives that greeting and prints it.

---

#### Step 1: Install dora

Install the dora command-line tool and the Python API:

```bash
# Linux / macOS
pip install dora-rs-cli
pip install dora-rs
pip install pyarrow
```

```powershell
# Windows (PowerShell)
pip install dora-rs-cli
pip install dora-rs
pip install pyarrow
```

Verify the installation:

```bash
dora --version
```

**Expected output:**

```
dora-cli 0.5.0
```

:::note What are these packages?
- `dora-rs-cli` — The `dora` command-line tool. You use it to run dataflows.
- `dora-rs` — The Python library your nodes use to send and receive data.
- `pyarrow` — Apache Arrow for Python. dora uses Arrow arrays as its message format — think of it as a fast alternative to JSON for passing data between programs.
:::

---

#### Step 2: Create Your Project

Create a folder with three files:

```bash
mkdir hello-dora
```

```bash
cd hello-dora
```

##### File 1: `talker.py`

This node sends a greeting every time it receives a tick from the timer.

```python
# talker.py — Sends a greeting every second
from dora import Node
import pyarrow as pa

def main():
    # Create a dora node. This connects to the dora coordinator
    # which manages the lifecycle of all nodes.
    node = Node()

    # Listen for events. dora sends events to your node:
    #   - "INPUT" when data arrives (e.g., a timer tick)
    #   - "STOP" when the dataflow is shutting down
    for event in node:
        if event["type"] == "INPUT":
            # Send a greeting to any node listening on our "greeting" output.
            # pa.array() wraps the data in an Apache Arrow array —
            # this is how all dora nodes exchange data.
            node.send_output("greeting", pa.array(["Hello from Talker!"]))

if __name__ == "__main__":
    main()
```

##### File 2: `listener.py`

This node prints every greeting it receives.

```python
# listener.py — Prints greetings from the Talker
from dora import Node

def main():
    node = Node()

    for event in node:
        if event["type"] == "INPUT":
            # event["value"] is an Apache Arrow array.
            # [0] gets the first element.
            # .as_py() converts it to a regular Python string.
            message = event["value"][0].as_py()
            print(f"Listener heard: {message}")

if __name__ == "__main__":
    main()
```

##### File 3: `dataflow.yml`

This is the **dataflow** — a YAML file that tells dora which nodes to run and how to connect them. Think of it as the wiring diagram of your application.

```yaml
# dataflow.yml — Connects Talker to Listener
nodes:
  - id: talker
    path: talker.py               # Path to the Python script
    inputs:
      tick: dora/timer/secs/1     # Built-in timer: sends a "tick" every 1 second
    outputs:
      - greeting                  # This node produces a "greeting" output

  - id: listener
    path: listener.py
    inputs:
      greeting: talker/greeting   # Receives "greeting" from the talker node
```

:::note What is `dora/timer/secs/1`?
It's a **built-in dora timer source**. It sends a tick event to your node at the specified interval. `dora/timer/secs/1` = every 1 second. `dora/timer/millis/100` = every 100 milliseconds. You don't need to write any timer code — dora handles it.
:::

---

#### Step 3: Run It

```bash
dora run dataflow.yml
```

**Expected output:**

```
Listener heard: Hello from Talker!
Listener heard: Hello from Talker!
Listener heard: Hello from Talker!
...
```

The greeting repeats once per second. Press `Ctrl+C` to stop the dataflow.

---

#### Step 4: What Just Happened?

Here's what dora did when you ran `dora run dataflow.yml`:

1. **Read `dataflow.yml`** — The dora coordinator parsed your YAML and built a graph of nodes and connections.
2. **Started the daemon** — A dora daemon process started on your machine to manage node processes and memory.
3. **Launched `talker.py`** — The daemon started your Talker as a separate process.
4. **Launched `listener.py`** — The daemon started your Listener as a separate process.
5. **Connected them** — The daemon set up a shared memory channel so that when Talker calls `send_output("greeting", ...)`, the data appears as an input event in Listener.
6. **Started the timer** — The built-in `dora/timer/secs/1` source began sending tick events to Talker every second.
7. **Data flowed** — Every tick, Talker created an Arrow array with `"Hello from Talker!"` and sent it. Listener received it, extracted the string, and printed it.

All of the coordination, process management, and data routing happened automatically. You just wrote the logic.

---

#### Step 5: Modify It

Try changing the Talker to send a counter:

```python
# talker.py — Sends a numbered greeting
from dora import Node
import pyarrow as pa

def main():
    node = Node()
    counter = 0

    for event in node:
        if event["type"] == "INPUT":
            counter += 1
            node.send_output(
                "greeting",
                pa.array([f"Hello #{counter} from Talker!"])
            )

if __name__ == "__main__":
    main()
```

Run again:

```bash
dora run dataflow.yml
```

**Expected output:**

```
Listener heard: Hello #1 from Talker!
Listener heard: Hello #2 from Talker!
Listener heard: Hello #3 from Talker!
...
```

---

#### Next Steps

You've just built and run your first dora dataflow. Here's where to go next:

- **[Core Concepts](../concepts/core-concepts.md)** — Understand nodes, operators, dataflows, timers, and Arrow data in depth.
- **[Webcam + YOLO Tutorial](./webcam_plot.md)** — Build a real-world 3-node dataflow using pre-built hub nodes: camera → object detection → visualization.
- **[Node Hub](../../nodes/)** — Browse 50+ pre-built nodes you can install with `pip` and compose in YAML — YOLO, Whisper, SAM2, robot arms, and more.
- **[Python API Reference](https://dora-rs.ai/python/index.html)** — Full reference for `Node()`, `send_output()`, event handling, and more.
- **[CLI Reference](../../api/cli.md)** — All `dora` commands: `run`, `build`, `start`, `stop`, `list`, `logs`.

---

## 7. Sample Deliverable — Core Concepts Page

> This is a draft of deliverable D4.

---

### Core Concepts

This page defines every term you'll see in dora documentation. Read it once, then use it as a reference.

:::tip Tested with
dora v0.5.0
:::

#### Dataflow

A **dataflow** is a YAML file (usually named `dataflow.yml`) that describes:

- Which programs (nodes) to run
- What data each node produces (outputs)
- What data each node consumes (inputs)
- How to connect outputs to inputs

It's the wiring diagram of your application. dora reads this file and handles everything else — starting processes, routing data, managing memory.

```yaml
# A minimal dataflow: one node that prints a message every second
nodes:
  - id: printer
    path: printer.py
    inputs:
      tick: dora/timer/secs/1
```

#### Node

A **node** is an independent process — a separate program that dora starts and manages. Each node:

- Runs in its own process (isolated from other nodes)
- Receives **inputs** (data from other nodes or from dora timers)
- Produces **outputs** (data sent to other nodes)
- Can be written in **Python, Rust, C, or C++**

Most dora applications use nodes. If you're starting out, use nodes.

**Python node example (5 lines):**

```python
from dora import Node
import pyarrow as pa

node = Node()
for event in node:
    if event["type"] == "INPUT":
        node.send_output("result", pa.array([42]))
```

#### Operator

An **operator** is a lighter-weight alternative to a node. Instead of running its own process, an operator is a library loaded into a dora **runtime node**. Multiple operators can share one process.

**When to use operators vs nodes:**

| | Node | Operator |
|---|---|---|
| **Process** | Separate process per node | Multiple operators share one process |
| **Isolation** | Full process isolation | Shared address space |
| **Overhead** | Higher (one process each) | Lower (shared process) |
| **Best for** | Python users, beginners, anything that needs full process control | Rust users optimizing for performance, thousands of components |

:::tip
**Start with nodes.** They're simpler, better documented, and work for 90% of use cases. Move to operators only if you're writing Rust and need sub-millisecond communication between components.
:::

#### Timer Source

A **timer source** is a built-in dora input that sends a tick event at a fixed interval. You don't write any timer code — just reference it in your YAML.

**Syntax:**

```yaml
inputs:
  tick: dora/timer/secs/1           # tick every 1 second
  fast_tick: dora/timer/millis/100  # tick every 100 milliseconds
```

The general format is `dora/timer/{unit}/{value}` where unit is `secs` or `millis`.

#### Apache Arrow

**Apache Arrow** is the message format dora uses for all inter-node communication. When one node sends data to another, that data is an Arrow array.

**Why not just send raw bytes or JSON?**

- Arrow supports **zero-copy reads** — the receiving node reads data directly from shared memory without copying it. This is why dora is 10-17x faster than ROS2.
- Arrow is **language-agnostic** — the same data format works in Python, Rust, C, and C++.
- Arrow defines a **C data interface** — no compilation step needed for message types (unlike ROS2 `.msg` files).

**In Python:**

```python
import pyarrow as pa

# Create an Arrow array (this is what you pass to send_output)
text_data = pa.array(["Hello, world!"])
number_data = pa.array([1, 2, 3, 4, 5])
float_data = pa.array([3.14, 2.71])

# Read from an Arrow array (this is what you get from event["value"])
first_element = text_data[0].as_py()  # → "Hello, world!"
```

#### dora-hub (Node Hub)

**dora-hub** is a library of pre-built, reusable nodes. Instead of writing a webcam capture node or a YOLO object detection node from scratch, you install one with pip and reference it in your YAML.

**Example — using a pre-built YOLO node:**

```bash
pip install dora-yolo
```

```yaml
nodes:
  - id: object-detection
    build: pip install dora-yolo
    path: dora-yolo
    inputs:
      image: camera/image
    outputs:
      - bbox
```

**Available hub nodes include:** webcam capture, YOLO, SAM2, Whisper (speech-to-text), Kokoro (text-to-speech), Qwen (LLM), Rerun (visualization), keyboard input, microphone input, and various robot arm controllers (Piper, Aloha, Reachy).

Browse the full catalog: [Node Hub](https://dora-rs.ai/docs/nodes/)

#### Architecture: How dora Works Internally

You don't need to know this to use dora, but it helps when debugging.

```
     You run: dora run dataflow.yml
                    │
                    ▼
            ┌──────────────┐
            │  Coordinator │ ← Reads YAML, validates graph, deploys nodes
            └──────┬───────┘
                   │ spawns (one per machine)
                   ▼
            ┌──────────────┐
            │    Daemon    │ ← Manages local node processes, shared memory,
            └──────┬───────┘   message routing
                   │ starts
           ┌───────┴────────┐
           ▼                ▼
     ┌──────────┐    ┌──────────┐
     │  Node A  │    │  Node B  │ ← Your Python/Rust/C programs
     │ (process)│    │ (process)│
     └──────────┘    └──────────┘
           │                ▲
           │  Arrow data    │
           └────────────────┘
              via shared memory (same machine)
              or TCP/Zenoh (across machines)
```

- **Coordinator** and **Daemon** are internal components. You never interact with them directly — you use the `dora` CLI.
- **Shared memory** is used for same-machine communication (fast, zero-copy).
- **Zenoh** is used for cross-machine communication (distributed dataflows over the network).

#### dora.builder (Python-defined Dataflows)

Instead of writing YAML, you can define your dataflow in Python code using the `dora.builder` API (added August 2025).

```python
from dora import Node
from dora.builder import DataflowBuilder

builder = DataflowBuilder()

# Add nodes programmatically
talker = builder.add_node("talker", path="talker.py")
talker.add_input("tick", "dora/timer/secs/1")
talker.add_output("greeting")

listener = builder.add_node("listener", path="listener.py")
listener.add_input("greeting", "talker/greeting")

# Run it
builder.run()
```

This is useful when your dataflow structure depends on runtime conditions. For most beginners, YAML is simpler.

---

#### Next Steps

- **[Python Hello World Tutorial](../getting-started/hello-world-python.md)** — Build your first dataflow.
- **[Webcam + YOLO Tutorial](../getting-started/webcam_plot.md)** — Build a real-world 3-node application using hub nodes.
- **[Dataflow Configuration Reference](../../api/dataflow-config.md)** — Full YAML schema reference.
- **[Python API Reference](https://dora-rs.ai/python/index.html)** — Complete Python API documentation.

---

## 8. Implementation Timeline — Week by Week

### Community Bonding Period (Weeks -2 to 0)

| Week | Activity | Output |
|---|---|---|
| -2 | Set up local dev environment. Build dora from source. Run all existing examples to verify they work. | Working dev environment, list of examples that fail |
| -1 | Read full codebase docs/ directory. Join Discord. Introduce myself. Discuss priority ordering with mentors Ning and Yu. | Confirmed priority order and scope |
| 0 | Submit initial PR: fix B13 (concatenated commands in Python Conversation guide) as a warm-up contribution. | Merged PR #1 |

### Phase 1: Critical Fixes (Weeks 1-3) — 45 hours

| Week | Workstream | Deliverable | Hours |
|---|---|---|---|
| 1 | WS1: Bug fixes | PR fixing all 13 documented bugs across both sites. Old site redirect banners. | 15 |
| 2 | WS2: Python Hello World | Draft of Python Hello World tutorial (as shown in Section 6). Submit for mentor review. | 12 |
| 3 | WS3: Core Concepts | Draft of Core Concepts page (as shown in Section 7). Submit for mentor review. Incorporate feedback on Hello World, merge. | 18 |

**Milestone 1 checkpoint:** A beginner can follow the Python Hello World from install to running output without hitting a single broken command.

### Phase 2: Foundation Building (Weeks 4-6) — 45 hours

| Week | Workstream | Deliverable | Hours |
|---|---|---|---|
| 4 | WS4: Navigation | Add Next Steps links to all existing pages. Add Prerequisites callouts. Add version stamps. Remove Brainstorming Ideas from sidebar. | 15 |
| 5 | WS5: Improve existing guides | Fix Python Conversation guide (preamble, formatting). Improve Webcam Plot guide. Add language tags to all code blocks. | 15 |
| 6 | WS6: FAQ page | Create FAQ/Troubleshooting page. Source questions from GitHub Discussions, Discord, and Issues. | 15 |

**Milestone 2 checkpoint:** All existing guides are polished. Navigation has no dead-ends. FAQ covers top 10 beginner questions.

### Midterm Evaluation (Week 6-7)

Deliverables for midterm:
- D1 (Bug fixes) ✅
- D2 (What is dora? page) ✅
- D3 (Python Hello World) ✅
- D4 (Core Concepts) ✅
- D5 (Improved existing guides) ✅
- D6 (FAQ) ✅
- D11 (Navigation improvements) ✅

### Phase 3: Expansion (Weeks 7-9) — 45 hours

| Week | Workstream | Deliverable | Hours |
|---|---|---|---|
| 7 | WS7: Rust Quickstart | Write Rust getting-started tutorial. Test against dora main. | 20 |
| 8 | WS8: Node Hub Reference | Create quick-reference table of all hub nodes with descriptions and install commands. | 10 |
| 9 | WS9: dora.builder guide | Write tutorial for Python-defined dataflows. | 15 |

**Milestone 3 checkpoint:** Rust users have a working quickstart. Hub ecosystem is discoverable. Builder API is documented.

### Phase 4: Polish & Stretch (Weeks 10-12) — 40 hours

| Week | Workstream | Deliverable | Hours |
|---|---|---|---|
| 10 | WS10: Example dataflows | Create 3-5 complete, tested example dataflows with READMEs. | 15 |
| 11 | WS11: CI testing | Set up GitHub Actions to test all doc code examples against dora main. | 10 |
| 12 | Final polish | Address all outstanding review comments. Final round of proofreading. Ensure all Next Steps links are correct. Write project summary blog post. | 15 |

**Final evaluation deliverables:** All 13 deliverables complete. All doc code tested in CI. Blog post published.

### Timeline Visualization

```
Week:  1    2    3    4    5    6    7    8    9    10   11   12
       ├────┼────┼────┼────┼────┼────┼────┼────┼────┼────┼────┤
WS1:   ████                              Bug fixes
WS2:        ████                          Python Hello World
WS3:             ████                     Core Concepts
WS4:                  ████               Navigation & structure
WS5:                       ████          Fix existing guides
WS6:                            ████     FAQ page
                                 ▲
                              MIDTERM
WS7:                                 ████████  Rust quickstart
WS8:                                      ████ Hub reference
WS9:                                           ████  Builder guide
WS10:                                               ████  Examples
WS11:                                                    ████ CI
WS12:                                                         ████
                                                                ▲
                                                             FINAL
```

---

## 9. Technical Approach

### 9.1 Tools and Stack

| Tool | Purpose |
|---|---|
| **Docusaurus** | The new docs site framework. All new pages will be Docusaurus MDX. |
| **Markdown + MDX** | Doc format. I'll use Docusaurus admonitions (`:::note`, `:::tip`, `:::warning`), tabs for OS-specific commands, and standard GitHub Flavored Markdown. |
| **dora CLI** | Every code example will be tested by running `dora run` against it. |
| **Python 3.11** | All Python examples tested with Python 3.11. |
| **GitHub Actions** | CI workflow to test doc code examples on every push to ensure they don't rot. |
| **Mermaid** | For architecture diagrams in Docusaurus (supported natively). |

### 9.2 Quality Standards

Every page I write will follow these rules:

| Rule | Why |
|---|---|
| Every concept defined before first use | Eliminates "wait, what does that mean?" moments |
| Every code block has a language tag | Syntax highlighting helps readability |
| Every code block is complete and runnable | No `...` placeholders, no fragments |
| Every tutorial shows expected output | User knows if it worked |
| Every page has Prerequisites and Version Stamp | User knows what they need before starting |
| Every page ends with Next Steps | No dead-ends |
| Shell commands on separate lines | Copy-paste works |
| OS-specific commands shown for Linux/macOS AND Windows | Half of beginners are on Windows |

### 9.3 Review Process

1. **Drafts** — I write in a branch, self-review against the quality checklist above.
2. **Mentor review** — PR to `dora-rs.github.io` for Ning and Yu to review.
3. **Testing** — I run every code example on my machine (Ubuntu + Windows WSL) before marking a PR as ready.
4. **Feedback iteration** — Incorporate mentor feedback. Target ≤ 2 review rounds per deliverable.

### 9.4 How I'll Handle the Two-Site Problem

The old mdBook site (`dora-rs.ai/dora/`) contains some content that doesn't exist on the new site yet (architecture overview, operators vs nodes). My approach:

1. **Week 1:** Add a redirect banner to every old site page pointing to the new site.
2. **Weeks 2-6:** Write replacements for the old site's useful content (Core Concepts replaces Overview; Python Hello World replaces Getting Started).
3. **Weeks 7-12:** Once all old site content has a new-site equivalent, the old site can be fully retired with 301 redirects. I'll prepare the redirect map but leave the final switch to the maintainers.

---

## 10. About Me

### 10.1 Profile

| | |
|---|---|
| **Name** | Mrugaja |
| **Education** | 2nd year undergraduate |
| **Location** | India (IST, UTC+5:30) |
| **GitHub** | [link to profile] |
| **Languages** | Python (proficient), Java (proficient), Rust (learning), C++ (learning) |

### 10.2 Relevant Experience

| Area | Details |
|---|---|
| **Python** | Proficient. Personal projects in ML, data processing, tooling. Comfortable with pyarrow, numpy, pip packaging, virtual environments. |
| **Technical writing** | [Describe any blog posts, README contributions, course documentation, or similar writing experience] |
| **Machine learning** | Coursework and projects. Familiar with YOLO, model inference pipelines, cv2, torch — which are exactly what dora-hub nodes wrap. |
| **Robotics** | Coursework in robotics fundamentals. Understand sensor-actuator loops, control systems, the need for inter-process communication. |
| **Rust** | Currently learning. Reading The Rust Programming Language book. Can read and understand Rust code, not yet writing production Rust. |
| **Open source** | [Describe any prior open source activity — even issues filed, docs read, communities joined] |

### 10.3 Prior Contributions to dora-rs

I am honest: I have not yet contributed to dora-rs. This proposal is my first substantive engagement with the project.

What I have done:
- **Read** the full documentation across both sites (audit is above).
- **Built** dora from source and ran the Python Conversation example.
- **Read** the GitHub Discussions — I can see the same questions repeating that better docs would answer.
- **Studied** the codebase structure, the hub node ecosystem, and the CLI commands.
- **Plan to submit** a warm-up PR fixing B13 (concatenated commands in the Python Conversation guide) during the community bonding period.

### 10.4 Motivation

I'll be direct: I'm applying to this project because I experienced the documentation problem firsthand. When I first tried to use dora-rs:

1. I landed on the old getting-started page. Every code block was one line. I couldn't read it.
2. I found the Python API page. It showed an Operator example labeled as "Try it out!" that was actually Node code. I was confused.
3. I eventually found the new Docusaurus site from the GitHub README. The Python Conversation guide mostly worked, but the first command (`uv venv...source .venv/bin/activate`) was concatenated and failed.
4. I spent time figuring out what a "dataflow" was, what `dora/timer/millis/100` meant, why I needed `pyarrow`. None of this was explained.

I got it working eventually. But I shouldn't have had to reverse-engineer the docs. And neither should the next person.

---

## 11. Why I Am the Right Fit

| Requirement | My Fit |
|---|---|
| **Python proficiency** | Proficient — Python is my primary language. I can write and test all Python examples. |
| **Beginner perspective** | I *am* a dora beginner. I know exactly where the docs fail because I just went through the experience. This perspective disappears once you become an expert. |
| **Writing ability** | This proposal and the sample deliverables demonstrate my writing quality. Every section is structured, every claim has evidence, every recommendation is actionable. |
| **Audit thoroughness** | I found 13 bugs, 9 concept gaps, and 8 structural problems without being asked. The audit report is ready to drive work from day 1. |
| **Rust learning trajectory** | I'm actively learning Rust, which means I can write the Rust quickstart guide (deliverable D7) from a learner's perspective — which is exactly the right perspective for a quickstart. |
| **Availability** | 20 hours/week for the full GSoC period. No other commitments. IST timezone overlaps well with mentors. |

---

## 12. Availability and Communication

| | |
|---|---|
| **Hours per week** | 20 hours, flexible schedule |
| **Timezone** | IST (UTC+5:30) |
| **Availability** | Full GSoC period — no vacations, exams, or other commitments planned |
| **Communication** | Discord (daily check-ins), GitHub PRs (all work), weekly sync call with mentors |
| **Reporting** | Weekly progress update posted to GitHub Discussion or Discord, linked to PRs |
| **Response time** | < 24 hours on all mentor messages |

### Communication Plan

| Frequency | Channel | Content |
|---|---|---|
| **Daily** | Discord | Quick status update: what I worked on, what's next, any blockers |
| **Weekly** | Video call with mentors | Demo deliverables, discuss priorities, plan next sprint |
| **Per deliverable** | GitHub PR | Complete draft with descriptions, screenshots, test instructions |
| **Midterm + Final** | GSoC evaluation page | Formal evaluation with links to all merged PRs |

---

## Appendix A: Full Bug Reference

See the attached [Documentation Audit Report](./dora_docs_review_report.md) for the complete bug inventory with exact page references, corrected code snippets, and fix recommendations for all 13 bugs, 9 concept gaps, and 8 structural problems.

---

## Appendix B: Links Referenced

| Resource | URL |
|---|---|
| dora-rs GitHub | https://github.com/dora-rs/dora |
| New docs site | https://dora-rs.ai/docs/guides/ |
| Python API reference | https://dora-rs.ai/python/index.html |
| Rust API reference | https://docs.rs/dora-node-api/latest/dora_node_api/ |
| Node Hub | https://dora-rs.ai/docs/nodes/ |
| dora-hub repo | https://github.com/dora-rs/dora-hub |
| Discord | https://discord.gg/DXJ6edAtym |
| GitHub Discussions | https://github.com/orgs/dora-rs/discussions |
| Python getting started | https://dora-rs.ai/docs/guides/getting-started/conversation_py |
| Installation | https://dora-rs.ai/docs/guides/Installation/installing |
| Examples | https://dora-rs.ai/docs/examples/ |
| CLI reference | https://dora-rs.ai/docs/api/cli |
| Dataflow config | https://dora-rs.ai/docs/api/dataflow-config |
| dora benchmark | https://github.com/dora-rs/dora-benchmark |
| GSoC 2025 ideas | https://github.com/dora-rs/dora/wiki/GSoC_2025 |
