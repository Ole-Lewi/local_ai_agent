# 🤖 Local AI Agent — ReAct vs LangGraph
Two agent architectures built from scratch, running entirely offline on a local LLM via Ollama.
No cloud APIs. No internet required. Just a local model, a tool, and two different ways to make an agent think.

# The Experiment
This repo explores a fundamental question in agentic AI:

How does a ReAct agent (LangChain) compare to a graph-based agent (LangGraph) when solving the same task?

Both agents are given the same tool (get_word_length) and the same class of problem. The architecture underneath is completely different.

# Agent 1 — ReAct Agent (agent1.py)
Uses LangChain's classic ReAct (Reason + Act) pattern with a hand-crafted prompt template.
Question → Thought → Action → Observation → Thought → Final Answer
The agent follows a structured reasoning loop defined entirely in the prompt. LangChain's AgentExecutor manages the loop, parsing the LLM's output at each step and deciding when to stop.
Model: llama3.2:1b via Ollama
Pattern: ReAct (prompt-driven reasoning loop)
Key components: create_react_agent, AgentExecutor, PromptTemplate

# Agent 2 — LangGraph Agent (langgraph_agent.py)
Uses LangGraph to model the agent as an explicit state machine with typed state, nodes, and conditional edges.
                  ┌─────────┐
      START ─────▶│   LLM   │
                  └────┬────┘
                       │
              ┌────────▼────────┐
              │ should_continue? │
              └────────┬────────┘
                  ┌────┴────┐
            tool_call?    no tool call
                  │              │
           ┌──────▼──────┐      END
           │  Tool Node  │
           └──────┬──────┘
                  │
            back to LLM ──▶ ...
The graph explicitly encodes when to call tools and when to stop — no prompt engineering needed for the control flow. The LLM decides via tool calls; the graph decides what to do with that decision.
Model: llama3.2:3b via Ollama
Pattern: Graph-based state machine
Key components: StateGraph, AgentState, conditional_edges, tool_node, END

# Why This Comparison Matters
ReAct (agent1.py)LangGraph (langgraph_agent.py)Control flowEncoded in the promptEncoded in the graphState managementImplicit (prompt history)Explicit (AgentState TypedDict)DebuggabilityHard — depends on LLM parsingEasy — each node is inspectableFlexibilityLimited to prompt structureFully customizable graphTracingVerbose outputNode-by-node executionBest forSimple, quick agentsComplex, production agents

# Tech Stack
ComponentToolLLMllama3.2:3b / llama3.2:1b (Ollama)Agent frameworkLangChain + LangGraphTool binding@tool decorator + bind_toolsStateTypedDict + Annotated with operator.addTracingDisabled (LANGCHAIN_TRACING_V2=false) — offline setup

Running Locally
Prerequisites

Ollama installed and running
Python 3.10+

1. Pull the models
bashollama pull llama3.2:3b
ollama pull llama3.2:1b
2. Clone and install
bashgit clone https://github.com/Ole-Lewi/local_ai_agent.git
cd local_ai_agent
pip install langchain langchain-ollama langgraph
3. Run either agent
bash# ReAct agent
python agent1.py

# LangGraph agent
python langgraph_agent.py
No .env file needed — no external API keys required.

# Why Offline?
This project was built under real-world constraints — limited internet access and a RAM-constrained machine (Lenovo ThinkPad T470s). Every design decision reflects that:

Ollama for local model serving (no API calls)
Small models (1b, 3b) that fit within available RAM
LANGCHAIN_TRACING_V2=false to disable LangSmith (requires internet)
No vector databases or external services

Building under constraints forces you to understand what each component actually does — rather than just connecting cloud services together.

# What's Next

 Add SqliteSaver for persistent conversation memory
 Add more tools (calculator, file reader, web search toggle)
 Build a voice interface (Vosk STT + Piper TTS)
 Evolve into a full Jarvis-like personal assistant


# Author
Lewis Miano (Lincoln)
ALX Backend Web Dev · ML/NLP · Agentic AI Systems
GitHub · Portfolio Bot