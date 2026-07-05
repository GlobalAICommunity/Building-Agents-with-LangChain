# Phase 2: GitHub Models Configuration

> ⏱️ **Time to complete**: 10 minutes

In this phase, we'll set up access to GitHub Models - a **completely free** way to use powerful LLMs like GPT-4.1, Llama, and more. No credit card required!

---

## 🎯 Learning Objectives

By the end of this phase, you will:
- Understand what GitHub Models offers and why we use it
- Generate a Personal Access Token (PAT)
- Configure your environment for API access
- Test the connection to verify it works
- Understand the LangChain ChatOpenAI integration

---

## 🧠 First, What Exactly *Is* an LLM?

A **Large Language Model (LLM)** is a program trained on enormous amounts of text (books, websites, code, conversations) so that it can predict, one small piece at a time, what text should come next. At its core, it's a very sophisticated "autocomplete." What makes it feel intelligent is scale — with billions of parameters and huge training data, that autocomplete becomes capable of answering questions, writing code, and holding conversations.

A few terms you'll see throughout the workshop:

| Term | Meaning |
|------|---------|
| **Prompt** | The text/instructions you send to the model |
| **Completion / Response** | The text the model generates back |
| **Token** | A chunk of text (roughly ¾ of a word). Models read and write text token-by-token, and API usage/limits are measured in tokens — not characters or words |
| **Temperature** | A setting (usually 0–1) controlling randomness: lower = more predictable/focused, higher = more varied/creative |
| **Context window** | The maximum amount of text (in tokens) the model can "see" at once, including the entire conversation history |

We never talk to the model directly — we go through an **API** (Application Programming Interface): a defined way for our Python code to send a prompt over the internet and receive a completion back, without needing to know anything about how the model runs internally.

---

## 🤖 What is GitHub Models?

GitHub Models is a free service that provides:
- Access to top LLMs (GPT-4.1, GPT-4o, Llama 3.3, Mistral, and more)
- OpenAI-compatible API endpoints
- No credit card or payment required
- Rate limits suitable for development and learning

### Why GitHub Models for This Workshop?

| Benefit | Description |
|---------|-------------|
| **Free** | No cost, no credit card required |
| **Easy Setup** | Just need a GitHub account you already have |
| **OpenAI Compatible** | Works with existing LangChain tools |
| **Multiple Models** | Access to various AI models |

### Available Models (as of 2026)

| Model | Provider | Best For |
|-------|----------|----------|
| `openai/gpt-4.1-nano` | OpenAI | Fast, efficient for learning |
| `openai/gpt-4.1` | OpenAI | Higher quality responses |
| `openai/gpt-4o` | OpenAI | Complex reasoning, best quality |
| `Meta-Llama-3.3-70B-Instruct` | Meta | Open-source alternative |

We'll use **`openai/gpt-4.1-nano`** in this workshop for fast responses.

---

## 🔑 Step 1: Generate a GitHub Personal Access Token

A Personal Access Token (PAT) is like a password that allows applications to authenticate with GitHub on your behalf.

### Navigate to Token Settings

1. Go to [github.com](https://github.com) and log in
2. Click your **profile picture** (top right)
3. Click **Settings**
4. Scroll down in the left sidebar to **Developer settings** (at the bottom)
5. Click **Personal access tokens** → **Tokens (classic)**

Or go directly to: [github.com/settings/tokens](https://github.com/settings/tokens)

### Create a New Token

1. Click **"Generate new token"** → **"Generate new token (classic)"**

2. Fill in the form:
   - **Note**: `Workshop - LangChain Chainlit` (or any description)
   - **Expiration**: `7 days` (enough for the workshop)
   - **Scopes**: No scopes needed! Leave all boxes **unchecked**

{% hint style="warning" %}
**Important**: For GitHub Models, you don't need any scopes selected. A token with no scopes can still access the Models API.
{% endhint %}

3. Scroll down and click **"Generate token"**

4. **COPY THE TOKEN IMMEDIATELY!** 
   
   You'll see something like: `ghp_xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx`
   
   ⚠️ This is shown only once. If you lose it, you'll need to generate a new one.

---

## ⚙️ Step 2: Configure Your Environment

Now let's add the token to your `.env` file:

```bash
# Open the .env file in your editor and replace the placeholder
# It should look like this:

GITHUB_TOKEN=ghp_your_actual_token_here
WEATHER_API_KEY=your_weather_api_key_here
```

### Using the Terminal

```bash
# On macOS/Linux - replace with your actual token
sed -i '' 's/your_github_token_here/ghp_your_actual_token/' .env

# Or just edit with any text editor:
# VS Code: code .env
# Nano: nano .env
# Vim: vim .env
```

---

## 🔍 Step 3: Understand the Architecture

Here's what happens when we call GitHub Models:

```
┌─────────────────┐     ┌──────────────────────┐     ┌─────────────┐
│   Your App      │────▶│   GitHub Models API  │────▶│  LLM Model  │
│   (LangChain)   │     │   (OpenAI Compatible)│     │  (GPT-4.1)  │
└─────────────────┘     └──────────────────────┘     └─────────────┘
         │                        │
         │                        │
    Uses GITHUB_TOKEN        Routes to model
    for authentication       you specified
```

### Key Details

| Setting | Value |
|---------|-------|
| **API Base URL** | `https://models.github.ai/inference` |
| **API Format** | OpenAI-compatible |
| **Authentication** | Bearer token (your GitHub PAT) |
| **Model Name** | e.g., `openai/gpt-4.1-nano` |

Because the API is OpenAI-compatible, we can use `langchain-openai` package with a custom base URL.

---

## 🧪 Step 4: Test the Connection

Let's verify everything works with a simple test script.

Create a file called `test_github_models.py` in a new `phase-02` folder:

```bash
mkdir -p phase-02
cd phase-02
touch test_github_models.py
```

Now open `test_github_models.py` and add this code:

```python
"""
Phase 2: Test GitHub Models Connection
Run with: python test_github_models.py

This script verifies that your GitHub Models connection is working correctly.
It's the foundation for all subsequent phases.

Key Concepts Introduced:
- Loading environment variables with dotenv
- Creating a ChatOpenAI client configured for GitHub Models
- Making a simple LLM request
"""

import os
from dotenv import load_dotenv
from langchain_openai import ChatOpenAI

# Load environment variables from .env file
# This reads the GITHUB_TOKEN from your .env file
load_dotenv()


def main():
    # Get the token from environment
    github_token = os.getenv("GITHUB_TOKEN")
    
    if not github_token or github_token == "your_github_token_here":
        print("❌ Error: GITHUB_TOKEN not set in .env file")
        print("   Please add your GitHub token to the .env file")
        return
    
    print("🔄 Testing connection to GitHub Models...")
    
    # Create the LLM client using LangChain's ChatOpenAI
    # GitHub Models uses an OpenAI-compatible API, so we can use ChatOpenAI
    # but point it to GitHub's endpoint instead of OpenAI's
    llm = ChatOpenAI(
        model="openai/gpt-4.1-nano",  # Model format: provider/model-name
        api_key=github_token,          # Your GitHub PAT for authentication
        base_url="https://models.github.ai/inference",  # GitHub's inference endpoint
        temperature=0.7,               # Controls randomness (0=deterministic, 1=creative)
    )
    
    try:
        # Send a simple test message
        response = llm.invoke("Say 'Hello, Workshop!' and nothing else.")
        print(f"✅ Success! Model responded: {response.content}")
        
    except Exception as e:
        print(f"❌ Error: {e}")
        print("\nTroubleshooting:")
        print("- Make sure your GitHub token is valid")
        print("- Check your internet connection")
        print("- Verify the token hasn't expired")


if __name__ == "__main__":
    main()
```

### 🔍 Understanding the Code

| Code | What It Does |
|------|--------------|
| `load_dotenv()` | Reads `.env` file and loads variables into environment |
| `os.getenv("GITHUB_TOKEN")` | Gets the token from environment |
| `ChatOpenAI(...)` | Creates a LangChain chat model client |
| `base_url=...` | Points to GitHub instead of OpenAI |
| `llm.invoke(...)` | Sends a message and gets a response |

Run the test:

```bash
python test_github_models.py
```

**Expected output**:
```
🔄 Testing connection to GitHub Models...
✅ Success! Model responded: Hello, Workshop!
```

---

## 🔍 Behind the Scenes: What Just Happened?

When you ran the test script:

1. **`load_dotenv()`** - Read the `.env` file and loaded `GITHUB_TOKEN` into the environment

2. **`ChatOpenAI(...)`** - Created an LLM client with:
   - Custom `base_url` pointing to GitHub's API
   - Your GitHub token as the `api_key`
   - Model set to `openai/gpt-4.1-nano`

3. **`llm.invoke(...)`** - Sent an HTTP POST request to:
   ```
   https://models.github.ai/inference/chat/completions
   ```
   With headers:
   ```
   Authorization: Bearer ghp_xxxxx
   Content-Type: application/json
   ```

   > 📘 **What does "Bearer" mean?** It's a standard authentication scheme — it just means "whoever presents (bears) this token is authorized to make the request." Almost every modern API (not just GitHub's) uses this same pattern, so this concept will come up again outside this workshop.

   And body:
   ```json
   {
     "model": "openai/gpt-4.1-nano",
     "messages": [{"role": "user", "content": "Say 'Hello, Workshop!'..."}],
     "temperature": 0.7
   }
   ```

   > 📘 **What is JSON?** JSON (JavaScript Object Notation) is a lightweight, text-based way to represent data — essentially Python dictionaries and lists written out as text so they can travel over the network. Every request to, and response from, the LLM API is JSON under the hood, even though LangChain's `ChatOpenAI` hides that detail behind normal Python objects and method calls.

4. **Response** - GitHub routed the request to the GPT-4.1-nano model and returned the completion

---

## 📚 Key Concepts to Remember

These concepts will be used in **every subsequent phase**:

| Concept | Code | Used For |
|---------|------|----------|
| Load env vars | `load_dotenv()` | Reading secrets from `.env` |
| Get env var | `os.getenv("GITHUB_TOKEN")` | Accessing the token |
| Create LLM | `ChatOpenAI(...)` | Making AI requests |
| Send message | `llm.invoke(message)` | Getting AI responses |

---

## 🗂️ Current Project Structure

```
langchain-chainlit-workshop/
├── .venv/
├── .env                    # Now contains your GITHUB_TOKEN
├── .gitignore
├── requirements.txt
└── phase-02/
    └── test_github_models.py   # Test script
```

---

## ✅ Checkpoint: GitHub Models Connected

| Check | Result |
|-------|--------|
| Token generated | ✓ Have a `ghp_xxx` token |
| Token saved | ✓ In `.env` file |
| Connection works | ✓ `test_github_models.py` shows success |

### 🎉 Connection Working?

**Excellent!** You now have free access to powerful LLMs.

👉 **Next up: [Phase 3: Your First Chainlit Chat App](03-chainlit-basics.md)**

---

## ❓ Common Issues

### "401 Unauthorized" or "Authentication failed"
- Double-check your token is copied correctly (starts with `ghp_`)
- Make sure there are no extra spaces in the `.env` file
- Verify the token hasn't expired

### "Model not found" or "404"
- Check the model name is exactly `openai/gpt-4.1-nano`
- The format is `provider/model-name`

### "Rate limit exceeded"
- GitHub Models has rate limits for free tier
- Wait a minute and try again
- For workshops, the facilitator may have backup tokens

### Token appears in git history
- If you accidentally committed your token:
  1. Revoke it immediately on GitHub
  2. Generate a new one
  3. Update your `.env` file
  4. Add `.env` to `.gitignore` (already done if you followed Phase 1)
