# Phase 3: Building a Chat Interface with Chainlit

> ⏱️ **Time to complete**: 15 minutes

In this phase, we'll build a chat interface step by step. You'll start with a minimal app and progressively add features.

---

## 🎯 Learning Objectives

By the end of this phase, you will:
- Create a Chainlit chat app
- Connect it to the LLM from Phase 2
- Add streaming responses
- Implement conversation memory

---

## 💬 What is Chainlit (and why not build a website from scratch)?

Normally, building a chat web app means writing HTML/CSS/JavaScript for the UI, a backend server, WebSocket code for real-time updates, and wiring it all together. **Chainlit** is a Python library that does all of that for you — you write plain Python functions, and it renders a full chat UI in the browser automatically.

Two special functions are all you need to get started:
- `@cl.on_chat_start` — runs once when a user opens the chat
- `@cl.on_message` — runs every time the user sends a message

### 📘 Wait, what does the `@` symbol mean ("decorators")?

A **decorator** is Python syntax that attaches extra behavior to a function without you writing that behavior yourself. `@cl.on_message` tells Chainlit: "call this function automatically whenever a new chat message arrives." You never call `main()` yourself — Chainlit calls it for you at the right moment. This is called an **event-driven** pattern: you write handlers for events (chat started, message received), and the framework decides when to run them.

### 📘 What does `async def` / `await` mean?

You'll see `async def main(...)` and `await cl.Message(...).send()` throughout this workshop. In short:
- `async def` marks a function as one that can *pause* while waiting on something slow (like a network call to the LLM) without freezing the whole app for other users.
- `await` is used inside an `async def` function to say "pause here until this finishes, then continue."

You don't need to master async programming to follow along — just remember: **whenever code talks to the network (an LLM or an API), it will be `async` and you `await` it.**

---

## 📁 Step 1: Create Your Project Folder

```bash
mkdir -p phase-03
cd phase-03
touch app.py
```

---

## 🚀 Step 2: Start with a Minimal App

Create `app.py` with just the basics - an app that echoes back what you type:

```python
import chainlit as cl

@cl.on_message
async def main(message: cl.Message):
    await cl.Message(content=f"You said: {message.content}").send()
```

**Run it:**
```bash
chainlit run app.py -w
```

Open http://localhost:8000 and type something. You should see it echoed back!

> 💡 The `-w` flag enables auto-reload when you save changes.

---

## 🤖 Step 3: Connect to the LLM

Now let's make it use AI. **Replace** your `app.py` with:

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

@cl.on_message
async def main(message: cl.Message):
    llm = get_llm()
    response = await llm.ainvoke(message.content)
    await cl.Message(content=response.content).send()
```

**What we added:**
- `get_llm()` - Same LLM setup from Phase 2
- `llm.ainvoke()` - Calls the AI asynchronously

> 📘 **`invoke` vs `ainvoke`:** Phase 2's test script used `llm.invoke(...)` (synchronous — it blocks until the response is ready). Chainlit apps handle many users concurrently, so we use the `a`-prefixed async version, `ainvoke`, paired with `await`, so the server can keep serving other users while waiting on the LLM.

**Test it:** Ask "What is Python?" - you get a real AI response!

**But there's a problem...** Try asking "Tell me more about it." The bot doesn't remember what "it" refers to!

---

## 🧠 Step 4: Add Conversation Memory

The LLM needs to see previous messages to have context. **Replace** your `app.py`:

```python
import os
import chainlit as cl
from dotenv import load_dotenv
from langchain_openai import ChatOpenAI
from langchain_core.messages import HumanMessage, AIMessage, SystemMessage

load_dotenv()

SYSTEM_PROMPT = """You are a helpful AI assistant named Aria. 
Be friendly and concise."""

def get_llm():
    return ChatOpenAI(
        model="openai/gpt-4.1-nano",
        api_key=os.getenv("GITHUB_TOKEN"),
        base_url="https://models.github.ai/inference",
    )

@cl.on_chat_start
async def start():
    # Initialize chat history with system prompt
    cl.user_session.set("chat_history", [SystemMessage(content=SYSTEM_PROMPT)])
    await cl.Message(content="👋 Hi! I'm Aria. How can I help?").send()

@cl.on_message
async def main(message: cl.Message):
    llm = get_llm()
    chat_history = cl.user_session.get("chat_history")
    
    # Add user message to history
    chat_history.append(HumanMessage(content=message.content))
    
    # Call LLM with FULL history
    response = await llm.ainvoke(chat_history)
    
    # Add AI response to history
    chat_history.append(AIMessage(content=response.content))
    cl.user_session.set("chat_history", chat_history)
    
    await cl.Message(content=response.content).send()
```

**What's new:**
| Code | Purpose |
|------|---------|
| `@cl.on_chat_start` | Runs once when chat opens |
| `cl.user_session` | Stores data for each user |
| `SystemMessage` | Gives the AI its personality |
| `HumanMessage` / `AIMessage` | Tracks the conversation |

> 📘 **Concept: chat "roles"** Every LLM conversation is really just a list of messages, each tagged with a **role**:
> - `system` — instructions that set the AI's behavior (the user never sees this)
> - `user` (aka Human) — what the person typed
> - `assistant` (aka AI) — what the model replied
>
> LangChain's `SystemMessage`, `HumanMessage`, and `AIMessage` classes are exactly these three roles. Sending the **entire list** back to the model on every turn is how it "remembers" the conversation — the model itself has no memory between API calls; we're just replaying the transcript each time.

> 📘 **What is `cl.user_session`?** Chainlit gives every connected browser tab its own private, in-memory storage — like a Python dictionary only that user can read/write. It's how each workshop attendee chats with "their own" Aria without seeing anyone else's conversation. It is **not saved to disk** — if the server restarts, all sessions are lost (fine for a workshop, but not how you'd persist memory in production).

**Test it:** Say "My name is Alex", then ask "What's my name?" - it remembers!

---

## ⚡ Step 5: Add Streaming

Waiting for the full response is slow. Let's stream it word-by-word.

**Replace** the `@cl.on_message` function:

```python
@cl.on_message
async def main(message: cl.Message):
    llm = get_llm()
    chat_history = cl.user_session.get("chat_history")
    
    chat_history.append(HumanMessage(content=message.content))
    
    # Create empty message for streaming
    msg = cl.Message(content="")
    full_response = ""
    
    # Stream token by token
    async for chunk in llm.astream(chat_history):
        if chunk.content:
            full_response += chunk.content
            await msg.stream_token(chunk.content)
    
    await msg.send()
    
    chat_history.append(AIMessage(content=full_response))
    cl.user_session.set("chat_history", chat_history)
```

Also update `get_llm()` to enable streaming:

```python
def get_llm():
    return ChatOpenAI(
        model="openai/gpt-4.1-nano",
        api_key=os.getenv("GITHUB_TOKEN"),
        base_url="https://models.github.ai/inference",
        streaming=True,  # ADD THIS!
    )
```

**What changed:**
- `streaming=True` - Enables token-by-token output
- `llm.astream()` - Returns chunks as they're generated
- `msg.stream_token()` - Displays each chunk immediately

> 📘 **How does streaming actually work?** The LLM API doesn't wait until the whole answer is ready — it sends back small pieces of text ("chunks", roughly one token at a time) as soon as they're generated. `llm.astream(...)` gives you these chunks as an **async generator** you loop over with `async for`. Each chunk is appended to the UI immediately via `stream_token()`, which is why the response appears to "type itself out" instead of showing up all at once.

**Test it:** Ask a longer question and watch the response appear progressively!

---

## 📋 Your Final Code

Here's what your complete `app.py` should look like:

```python
import os
import chainlit as cl
from dotenv import load_dotenv
from langchain_openai import ChatOpenAI
from langchain_core.messages import HumanMessage, AIMessage, SystemMessage

load_dotenv()

SYSTEM_PROMPT = """You are a helpful AI assistant named Aria. 
Be friendly and concise."""

def get_llm():
    return ChatOpenAI(
        model="openai/gpt-4.1-nano",
        api_key=os.getenv("GITHUB_TOKEN"),
        base_url="https://models.github.ai/inference",
        temperature=0.7,
        streaming=True,
    )

@cl.on_chat_start
async def start():
    cl.user_session.set("chat_history", [SystemMessage(content=SYSTEM_PROMPT)])
    await cl.Message(content="👋 Hi! I'm Aria. How can I help?").send()

@cl.on_message
async def main(message: cl.Message):
    llm = get_llm()
    chat_history = cl.user_session.get("chat_history")
    
    chat_history.append(HumanMessage(content=message.content))
    
    msg = cl.Message(content="")
    full_response = ""
    
    async for chunk in llm.astream(chat_history):
        if chunk.content:
            full_response += chunk.content
            await msg.stream_token(chunk.content)
    
    await msg.send()
    
    chat_history.append(AIMessage(content=full_response))
    cl.user_session.set("chat_history", chat_history)
```

---

## 🗂️ Project Structure

```
phase-03/
└── app.py          # Chat application
```

---

## ✅ Checkpoint

| Test | Try | Expected |
|------|-----|----------|
| Basic chat | "Hello!" | Friendly greeting |
| Memory | "I'm Alex" → "What's my name?" | "Alex" |
| Streaming | Long question | Words appear progressively |
| Identity | "What's your name?" | "Aria" |

### 🎉 All Working?

You've built a chat interface with memory and streaming!

👉 **Next: [Phase 4: LangChain Agents](04-langchain.md)**

---

## ❓ Common Issues

### Memory not working
Make sure you have both `@cl.on_chat_start` AND `cl.user_session.set()` after each message.

### Streaming not working  
Check `streaming=True` in `get_llm()` and use `astream()` not `ainvoke()`.

### Port 8000 in use
```bash
chainlit run app.py -w --port 8001
```
