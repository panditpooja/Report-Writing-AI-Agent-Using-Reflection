# Report Writing AI Agent Using Reflection

A sophisticated AI agent system that generates high-quality reports through an iterative reflection process. The system uses LangGraph to orchestrate a multi-agent workflow where a generator creates reports and a reflector critiques and improves them iteratively.

## Overview

This system implements a **reflection-based report writing agent** that:

1. **Generates** an initial report based on a user-provided topic
2. **Reflects** on the generated report by critiquing content, structure, depth, and style
3. **Iterates** by incorporating reflections to improve the report
4. **Repeats** the reflection and revision steps for a set number of iterations

The system uses different LLM models for generation (faster) and reflection (more capable), and both agents can use tools (like web search) to gather information and provide feedback.

## Features

- ✅ **Dual-Model Architecture**: Uses different models optimized for generation vs. reflection
- ✅ **Tool Integration**: Both generator and reflector can use tools (e.g., Tavily search) to gather information
- ✅ **Iterative Improvement**: Automatically refines reports through multiple reflection cycles
- ✅ **State Management**: Tracks node execution and tool usage for proper routing
- ✅ **Error Handling**: Properly handles ToolMessage types and routing edge cases

## Architecture

The system is built using **LangGraph** and follows this workflow:

```
User Topic → Generate → [Tools?] → Reflect → [Tools?] → Generate → ... → Final Report
```

### Components

1. **Generation Node**: Creates reports using `openai/gpt-oss-20b:free` model
2. **Reflection Node**: Critiques reports using `nvidia/nemotron-nano-9b-v2:free` model  
3. **Tools Node**: Executes tool calls (e.g., Tavily search) for both agents
4. **Routing Logic**: Intelligently routes between nodes based on tool calls and iteration limits

## Prerequisites

- Python 3.8 or higher
- OpenRouter API key (for accessing LLM models)
- Tavily API key (for web search functionality)
- Jupyter Notebook or JupyterLab (for running the notebook)

## Installation

1. **Clone the repository** 
   ```bash
   git clone https://github.com/panditpooja/Report-Writing-AI-Agent-Using-Reflection.git

2. **Install dependencies**:
   ```bash
   pip install -r requirements.txt
   ```

3. **Set up environment variables**:
   Create a `.env` file in the project root and add your API keys:
   ```
   OPENROUTER_API_KEY=your_openrouter_api_key_here
   TAVILY_API_KEY=your_tavily_api_key_here
   ```

## Usage

1. **Open the notebook**:
   ```bash
   jupyter notebook reflection_report_writing_system.ipynb
   ```
   Or use JupyterLab:
   ```bash
   jupyter lab reflection_report_writing_system.ipynb
   ```

2. **Run all cells** in order, or execute them step by step

3. **Modify the topic** in the last cell:
   ```python
   topic = "Your report topic here"
   ```

4. **Run the agent**:
   ```python
   await run_agent()
   ```

## Configuration

### Models

The system uses OpenRouter to access different models:

- **Generation Model**: `openai/gpt-oss-20b:free` (faster, good for generation)
- **Reflection Model**: `nvidia/nemotron-nano-9b-v2:free` (more capable, good for critique)

You can modify these in the notebook cells where models are defined.

### Tools

Currently configured with:
- **Tavily Search**: Web search tool for gathering information (max 5 results)

## Project Structure

```
.
├── reflection_report_writing_system.ipynb  # Main notebook
├── README.md                               # This file (user documentation)
├── requirements.txt                        # Python dependencies
├── final_report.txt                        # Output file (generated after running)
├── .env.example                            # Example environment variables
└── .gitignore                              # Git ignore rules
```

**Note**: You'll need to create a `.env` file with your API keys (see Installation section).

## How It Works

### State Management

The system uses a `State` TypedDict that tracks:
- `messages`: Conversation history (accumulates across iterations)
- `last_node`: Which node last called tools (for proper routing)
- `finished`: Boolean flag indicating if the report is complete (set by reflection node)

### Routing Logic

1. **After Generate**: 
   - If tool calls exist → route to tools
   - If iteration limit reached → end
   - Otherwise → route to reflect

2. **After Reflect**:
   - If tool calls exist → route to tools
   - If `finished` flag is `True` → end (report is complete)
   - Otherwise → route back to generate (continue iteration)

3. **After Tools**:
   - Route back to the node that called the tools (generate or reflect)

### Reflection Process

The reflector:
1. Receives the generated report
2. Swaps message roles (AI becomes Human, Human becomes AI) for critique perspective
3. Preserves ToolMessages to maintain context
4. Provides detailed feedback on:
   - Content accuracy
   - Completeness/depth
   - Structure & presentation
   - Style & tone
   - Citations & sources

## Feedback Mechanism Explained

### How Feedback Flows Through the System

The feedback mechanism uses a clever **role-swapping technique** to make the reflection model treat the generated report as user-provided content. Here's how it works:

#### Feedback Flow Diagram

```
┌─────────────────────────────────────────────────────────────────┐
│                    ITERATION 1                                   │
└─────────────────────────────────────────────────────────────────┘

1. GENERATE NODE
   ┌─────────────────┐
   │ User: "Topic X" │
   └────────┬────────┘
            │
            ▼
   ┌──────────────────────────┐
   │ AI: "Report v1..."       │ ← Generated report
   └────────┬─────────────────┘
            │
            ▼
2. REFLECT NODE (Role Swap Happens Here)
   ┌─────────────────────────────────────────────┐
   │ BEFORE SWAP:                                │
   │   Human: "Topic X"                          │
   │   AI: "Report v1..."                        │
   │                                             │
   │ AFTER SWAP:                                 │
   │   AI: "Topic X" (swapped)                   │
   │   Human: "Report v1..." (swapped)          │
   │                                             │
   │ Reflection Model sees:                      │
   │   "A human gave me this report to review"   │
   └────────┬────────────────────────────────────┘
            │
            ▼
   ┌─────────────────────────────────────────────┐
   │ Reflection Model generates feedback:        │
   │ "Good start but needs more detail.          │
   │  REPORT STATUS: NEEDS REVISION"             │
   └────────┬────────────────────────────────────┘
            │
            ▼
   ┌─────────────────────────────────────────────┐
   │ Convert feedback to HumanMessage            │
   │ (So generator sees it as user feedback)     │
   └────────┬────────────────────────────────────┘
            │
            ▼
3. ROUTING
   ┌─────────────────────────────────────────────┐
   │ Check finish flag: False                    │
   │ → Route back to GENERATE                    │
   └────────┬────────────────────────────────────┘
            │
            ▼
┌─────────────────────────────────────────────────────────────────┐
│                    ITERATION 2                                   │
└─────────────────────────────────────────────────────────────────┘

4. GENERATE NODE (Sees Full History)
   ┌─────────────────────────────────────────────┐
   │ Message History:                            │
   │   Human: "Topic X"                          │
   │   AI: "Report v1..."                        │
   │   Human: "Good start but needs more..."     │ ← Feedback
   └────────┬────────────────────────────────────┘
            │
            ▼
   ┌─────────────────────────────────────────────┐
   │ AI: "Improved Report v2..."                 │ ← Incorporates feedback
   └─────────────────────────────────────────────┘
            │
            ▼
   (Cycle continues until finish flag = True)
```

### Key Components

#### 1. Role Swapping in Reflection Node

```python
# In reflection_node():
cls_map = {"ai": HumanMessage, "human": AIMessage}
translated = [
    cls_map[msg.type](content=msg.content) 
    for msg in state["messages"]
]
```

**Why?** The reflection model needs to see the report as something to critique, not as its own output. By swapping roles:
- The generated report (originally `AIMessage`) becomes a `HumanMessage`
- The original request (originally `HumanMessage`) becomes an `AIMessage`
- The reflection model now thinks: "A human gave me this report to review"

#### 2. Feedback as HumanMessage

After reflection, the feedback is converted back to a `HumanMessage`:

```python
return {
    "messages": [HumanMessage(content=res.content)],  # Feedback appears as user input
    "finished": is_complete
}
```

This makes the generation model treat the feedback as if a human reviewer provided it, naturally incorporating it into the next iteration.

#### 3. Finish Flag Logic

The system uses a sophisticated finish flag that checks:
- ✅ Explicit completion status ("REPORT STATUS: COMPLETE")
- ✅ No negative indicators after completion
- ✅ No conditional completions ("complete, but...")
- ✅ No explicit "needs revision" status

Only when all conditions are met does the system mark the report as complete and stop iterating.

### Concrete Example

Here's a real example of how the feedback flows through the system:

#### Iteration 1

**User Input:**
```
Human: "Who is the innovation officer of Biosphere 2 and his work?"
```

**Generation Node Output:**
```
AI: "**Innovation Officer of Biosphere 2: Jeff Larsen**

Jeff Larsen serves as the Innovation Officer at Biosphere 2...
[Report continues with basic information]"
```

**Reflection Node (with role swap):**
```
[Internally, roles are swapped]
Reflection Model sees:
  AI: "Who is the innovation officer of Biosphere 2 and his work?"
  Human: "**Innovation Officer of Biosphere 2: Jeff Larsen**..."

Reflection Model outputs:
"The report provides a good introduction to Jeff Larsen's role, but it lacks 
specific details about his key initiatives and contributions. The structure 
is clear, but more depth on his work with AI and autonomous systems would 
strengthen the report.

REPORT STATUS: NEEDS REVISION"
```

**Feedback Converted to HumanMessage:**
```
Human: "The report provides a good introduction to Jeff Larsen's role, but 
it lacks specific details about his key initiatives and contributions. The 
structure is clear, but more depth on his work with AI and autonomous systems 
would strengthen the report.

REPORT STATUS: NEEDS REVISION"
```

#### Iteration 2

**Generation Node (sees full history):**
```
Message History:
  Human: "Who is the innovation officer of Biosphere 2 and his work?"
  AI: "**Innovation Officer of Biosphere 2: Jeff Larsen**..." [v1]
  Human: "The report provides a good introduction... REPORT STATUS: NEEDS REVISION"

Generation Model now generates:
AI: "**Innovation Officer of Biosphere 2: Jeff Larsen – A Comprehensive Report**

[Enhanced report with more detail on AI initiatives, autonomous systems, 
and specific contributions - incorporates the feedback]"
```

**Reflection Node:**
```
Reflection Model: "The report now includes comprehensive details about Larsen's 
work with AI and autonomous systems. The structure is excellent, and all 
sections are well-developed. The report is thorough and complete.

REPORT STATUS: COMPLETE - No further revisions needed"
```

**Finish Flag:**
```
✅ Finish flag set to True
✅ System ends iteration
✅ Final report saved
```

### Message History Accumulation

The key insight is that **all messages accumulate** in the conversation history:

```
Iteration 1: [User] → [Report v1] → [Feedback 1]
Iteration 2: [User] → [Report v1] → [Feedback 1] → [Report v2] → [Feedback 2]
Iteration 3: [User] → [Report v1] → [Feedback 1] → [Report v2] → [Feedback 2] → [Report v3] → [Feedback 3]
```

Each generation sees the full history, allowing it to:
- Understand what was tried before
- See what feedback was given
- Build upon previous iterations
- Avoid repeating mistakes

## Example Output

```
=== Generated Report ===
[Initial report content...]

[DEBUG] Tools executed: 12 messages

=== Generated Report ===
[Improved report after tool usage...]

=== Reflection ===
[Detailed feedback with recommendations...]

=== Generated Report ===
[Revised report incorporating feedback...]

=== Reflection ===
[Additional feedback and recommendations...]
```

## Troubleshooting

### Common Issues

1. **API Key Errors**: Ensure your `.env` file has correct API keys
2. **Import Errors**: Run `pip install -r requirements.txt` to install dependencies
3. **Tool Errors**: Check that Tavily API key is valid and has credits
4. **Routing Issues**: The system should handle routing automatically, but check state management if issues occur

### Debug Mode

Uncomment the debug line in `run_agent()` to see all event keys:
```python
# print(f"Event keys: {list(event.keys())}")
```

## Limitations

- Requires API keys for OpenRouter and Tavily
- Model availability depends on OpenRouter service
- Free tier models may have rate limits
- Iteration limit is set to 15 non-tool messages (can be adjusted in `route_after_generate`)
- System automatically stops when reflection indicates report is complete (via finish flag)
- Fallback iteration limit prevents infinite loops if finish flag logic fails

## Acknowledgments

- Built with [LangGraph](https://github.com/langchain-ai/langgraph)
- Uses [LangChain](https://github.com/langchain-ai/langchain) for LLM integration
- Powered by [OpenRouter](https://openrouter.ai/) for model access
- Uses [Tavily](https://tavily.com/) for web search

## Contributing

Contributions are welcome! Please feel free to submit issues or pull requests.

## ✍️ Author

**Pooja Pandit**  
Master's in Information Science (Machine Learning)  
The University of Arizona

[![GitHub](https://img.shields.io/badge/GitHub-panditpooja-black?logo=github)](https://github.com/panditpooja)
[![LinkedIn](https://img.shields.io/badge/LinkedIn-pooja--pandit-blue?logo=linkedin)](https://www.linkedin.com/in/pooja-pandit-177978135/)

