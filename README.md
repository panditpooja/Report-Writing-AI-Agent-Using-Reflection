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

1. **Generation Node**: Creates reports using `gpt-oss-20b:free` model
2. **Reflection Node**: Critiques reports using `gpt-oss-120b:free` model  
3. **Tools Node**: Executes tool calls (e.g., Tavily search) for both agents
4. **Routing Logic**: Intelligently routes between nodes based on tool calls and iteration limits

## Prerequisites

- Python 3.8 or higher
- OpenRouter API key (for accessing LLM models)
- Tavily API key (for web search functionality)
- Jupyter Notebook or JupyterLab (for running the notebook)

## Installation

1. **Clone the repository** (or download the files)

2. **Install dependencies**:
   ```bash
   pip install -r requirements.txt
   ```

3. **Set up environment variables**:
   ```bash
   cp .env.example .env
   ```
   Then edit `.env` and add your API keys:
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
- **Reflection Model**: `openai/gpt-oss-120b:free` (more capable, good for critique)

You can modify these in the notebook cells where models are defined.

### Tools

Currently configured with:
- **Tavily Search**: Web search tool for gathering information (max 5 results)

## Project Structure

```
.
├── reflection_report_writing_system.ipynb  # Main notebook
├── README.md                                # This file
├── requirements.txt                         # Python dependencies
├── .env.example                             # Environment variables template
└── .gitignore                               # Git ignore rules
```

## How It Works

### State Management

The system uses a `State` TypedDict that tracks:
- `messages`: Conversation history
- `last_node`: Which node last called tools (for proper routing)

### Routing Logic

1. **After Generate**: 
   - If tool calls exist → route to tools
   - If iteration limit reached → end
   - Otherwise → route to reflect

2. **After Reflect**:
   - If tool calls exist → route to tools
   - Otherwise → route back to generate

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
- Iteration limit is set to 10 non-tool messages (can be adjusted in `route_after_generate`)
- System continues iterating until the limit is reached (no automatic quality-based stopping)

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

