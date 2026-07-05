# Phase 4: Introducing LangChain Agents

> ⏱️ **Time to complete**: 15 minutes

In this phase, we'll convert our chat app to use a **LangChain Agent**. This prepares us for adding tools in Phase 5.

---

## 🎯 Learning Objectives

By the end of this phase, you will:
- Understand what an AI agent is
- Convert from direct LLM calls to an agent
- Use the new `create_agent()` function
- Prepare for tool integration

---

## � LangChain, LangGraph, and Agents — How They Fit Together

- **LangChain** is a framework of building blocks for LLM apps: prompts, models, tools, memory, and — most relevant here — **agents**.
- **LangGraph** is a lower-level library (by the same team) that represents an application as a graph of steps that can loop, branch, and hold state. You won't write LangGraph code directly in this workshop, but it's worth knowing: `create_agent()` builds a LangGraph graph behind the scenes. That's *why* an agent can "loop" (think → act → observe → repeat) instead of only answering once like a plain LLM call.
- A **chain** (an older, simpler LangChain concept) is a fixed, one-directional sequence of steps — always A then B then C, decided in advance by you. An **agent** is more flexible: at each step, the *model itself* decides what happens next (answer directly? call a tool? call another tool?). That dynamic decision-making is the whole point of Phases 5 and 6.

---

## �🤖 What is an Agent?

An **agent** is an AI that can:
1. **Think** about what to do
2. **Act** using tools (APIs, functions, etc.)
3. **Observe** the results
4. **Repeat** until task is complete

```
┌─────────────────────────────────────────┐
│            Agent Loop                    │
│                                          │
│   Think ──▶ Act ──▶ Observe             │
│     ▲                   │                │
│     └───────────────────┘                │
│       (repeat until done)                │
└─────────────────────────────────────────┘
```

**In this phase**, we create an agent WITHOUT tools - just to learn the concept. **In Phase 5**, we'll add tools.

---

## 📁 Step 1: Create Your Project Folder

```bash
mkdir -p phase-04
cd phase-04
touch app.py
```

---

## 📝 Step 2: Start with Phase 3 Code

Create `app.py` by copying the essentials from Phase 3:

```python
import os
import chainlit as cl
from dotenv import load_dotenv
from langchain_openai import ChatOpenAI

load_dotenv()

def get_llm():
    return ChatOpenAI(
        model="openai/gpt-4.1-nano",
        api_key=os.getenv("GITHUB_TOKEN"),
        base_url="https://models.github.ai/inference",
    )

@cl.on_chat_start
async def start():
    cl.user_session.set("chat_history", [])
    await cl.Message(content="👋 Hi! I'm Aria. How can I help?").send()

@cl.on_message
async def main(message: cl.Message):
    # We'll replace this with agent code
    pass
```

---

## 🔧 Step 3: Add the Agent Import

Add this import at the top:

```python
from langchain.agents import create_agent
```

---

## 📋 Step 4: Create the System Prompt

Add a system prompt that defines the agent's behavior. Put this after `load_dotenv()`:

```python
from datetime import date

SYSTEM_PROMPT = f"""You are a helpful AI assistant named Aria.
- Be friendly and conversational
- Give concise but thorough answers
- Admit when you don't know something

Current date: {date.today().strftime("%B %d, %Y")}
"""
```

---

## 🤖 Step 5: Create the Agent

Add a function to create the agent. Put this after `get_llm()`:

```python
def create_assistant_agent():
    llm = get_llm()
    
    agent = create_agent(
        model=llm,
        tools=[],  # No tools yet - we'll add them in Phase 5!
        system_prompt=SYSTEM_PROMPT,
    )
    
    return agent
```

**Key points:**
- `tools=[]` - Empty list means no tools (for now)
- `system_prompt` - Defines the agent's personality
- The agent wraps the LLM with reasoning capabilities

---

## 🚀 Step 6: Update on_chat_start

Replace the `start()` function to create the agent:

```python
@cl.on_chat_start
async def start():
    agent = create_assistant_agent()
    
    cl.user_session.set("agent", agent)
    cl.user_session.set("chat_history", [])
    
    await cl.Message(content="👋 Hi! I'm Aria. How can I help?").send()
```

---

## 💬 Step 7: Handle Messages with the Agent

Replace the `main()` function with agent-based message handling:

```python
@cl.on_message
async def main(message: cl.Message):
    agent = cl.user_session.get("agent")
    chat_history = cl.user_session.get("chat_history")
    
    # Add user message to history (simple dict format)
    chat_history.append({"role": "user", "content": message.content})
    
    # Stream the response
    msg = cl.Message(content="")
    full_response = ""

    async for data, _ in agent.astream(
        {"messages": chat_history}, 
        stream_mode="messages"
    ):
        chunks = data.content_blocks
        if len(chunks) > 0:
            chunk = chunks[-1]["text"]
            full_response += chunk
            await msg.stream_token(chunk)

    await msg.send()

    # Save assistant response
    chat_history.append({"role": "assistant", "content": full_response})
    cl.user_session.set("chat_history", chat_history)
```

> 📘 **Why dicts instead of `HumanMessage`/`AIMessage` now?** `create_agent()` accepts simple `{"role": ..., "content": ...}` dictionaries — the same format used by the raw OpenAI-style API and the most common convention across the industry. You can still use LangChain message objects if you prefer; dicts are just less typing. Both represent the exact same three roles from Phase 3 (`system`/`user`/`assistant`).

> 📘 **What's `content_blocks`?** Modern LLM responses aren't always plain text — they can include text, tool calls, images, etc. as separate labeled "blocks." `content_blocks` is a list of these blocks; `chunks[-1]["text"]` grabs the text out of the most recent one. Right now (no tools yet) it's always plain text, but this structure is exactly what lets the *same* streaming code also handle tool calls, as you'll see in Phase 5.

**What's different from Phase 3:**
| Phase 3 | Phase 4 |
|---------|---------|
| `llm.astream(messages)` | `agent.astream({"messages": ...})` |
| `HumanMessage()` objects | Simple dicts `{"role": "user", ...}` |
| Direct LLM call | Agent loop (even without tools) |

---

## 📋 Your Complete Code

Here's what `app.py` should look like now:

```python
import os
from datetime import date
import chainlit as cl
from dotenv import load_dotenv
from langchain_openai import ChatOpenAI
from langchain.agents import create_agent

load_dotenv()

SYSTEM_PROMPT = f"""You are a helpful AI assistant named Aria.
- Be friendly and conversational
- Give concise but thorough answers
- Admit when you don't know something

Current date: {date.today().strftime("%B %d, %Y")}
"""

def get_llm():
    return ChatOpenAI(
        model="openai/gpt-4.1-nano",
        api_key=os.getenv("GITHUB_TOKEN"),
        base_url="https://models.github.ai/inference",
        temperature=0.7,
    )

def create_assistant_agent():
    llm = get_llm()
    
    agent = create_agent(
        model=llm,
        tools=[],
        system_prompt=SYSTEM_PROMPT,
    )
    
    return agent

@cl.on_chat_start
async def start():
    agent = create_assistant_agent()
    
    cl.user_session.set("agent", agent)
    cl.user_session.set("chat_history", [])
    
    await cl.Message(content="👋 Hi! I'm Aria. How can I help?").send()

@cl.on_message
async def main(message: cl.Message):
    agent = cl.user_session.get("agent")
    chat_history = cl.user_session.get("chat_history")
    
    chat_history.append({"role": "user", "content": message.content})
    
    msg = cl.Message(content="")
    full_response = ""

    async for data, _ in agent.astream(
        {"messages": chat_history}, 
        stream_mode="messages"
    ):
        chunks = data.content_blocks
        if len(chunks) > 0:
            chunk = chunks[-1]["text"]
            full_response += chunk
            await msg.stream_token(chunk)

    await msg.send()

    chat_history.append({"role": "assistant", "content": full_response})
    cl.user_session.set("chat_history", chat_history)
```

---

## ▶️ Step 8: Run and Test

```bash
chainlit run app.py -w
```

### Test Scenarios

| Test | Expected |
|------|----------|
| "Hello!" | Friendly greeting |
| "What's your name?" | "Aria" |
| "What's today's date?" | Correct date |
| Follow-up questions | Context remembered |

---

## 💡 What Did We Gain?

Right now, the agent behaves similarly to Phase 3. **So why bother?**

The agent architecture gives us:
1. **Easy tool integration** - Just add tools to the list (Phase 5!)
2. **Built-in reasoning** - Agent can decide what to do
3. **Cleaner code** - Agent handles message flow
4. **Future-proof** - Ready for advanced features

---

## 🗂️ Project Structure

```
phase-04/
└── app.py          # Agent-based chat
```

---

## ✅ Checkpoint

| Check | Status |
|-------|--------|
| Using `create_agent()` | ☐ |
| `tools=[]` (empty) | ☐ |
| Streaming works | ☐ |
| Memory works | ☐ |
| Date is correct | ☐ |

### 🎉 Agent Working?

You now have a LangChain agent ready for tools!

👉 **Next: [Phase 5: Adding Tools](05-tool-calling.md)**

---

## ❓ Common Issues

### "No module named 'langchain.agents'"
```bash
pip install --upgrade langchain langchain-openai
```

### Streaming not working
Make sure you have `stream_mode="messages"` in the `astream()` call.

### "content_blocks" error
Check you're unpacking correctly: `async for data, _ in agent.astream(...)`
