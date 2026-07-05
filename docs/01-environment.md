# Phase 1: Environment Setup

> ⏱️ **Time to complete**: 10 minutes

In this phase, we'll create our project folder, set up a Python virtual environment, and install the core dependencies. This foundation ensures everyone has a consistent, isolated development environment.

---

## 🎯 Learning Objectives

By the end of this phase, you will:
- Understand why virtual environments matter
- Create an isolated Python environment for the project
- Install the core packages we'll use throughout the workshop

---

## 📁 Step 1: Create the Project Folder

Open your terminal and run:

```bash
# Create and navigate to the project folder
mkdir langchain-chainlit-workshop
cd langchain-chainlit-workshop
```

---

## 🐍 Step 2: Create a Virtual Environment

A **virtual environment** is an isolated Python installation. This means:
- Packages we install won't affect your system Python
- We can have specific versions without conflicts
- Easy to delete and recreate if something goes wrong

{% tabs %}
{% tab title="Using uv (Recommended)" %}
```bash
# Create virtual environment
uv venv

# Activate it
# macOS/Linux:
source .venv/bin/activate

# Windows:
.venv\Scripts\activate
```
{% endtab %}

{% tab title="Using pip/venv" %}
```bash
# Create virtual environment
python3 -m venv .venv

# Activate it
# macOS/Linux:
source .venv/bin/activate

# Windows:
.venv\Scripts\activate
```
{% endtab %}
{% endtabs %}

### 🔍 What Just Happened?

When you created the virtual environment:
1. Python created a `.venv` folder in your project
2. This folder contains a copy of the Python interpreter
3. It has its own `site-packages` directory for libraries
4. When activated, your shell's `PATH` is modified to use this Python first

**You'll know it's active when you see `(.venv)` at the start of your terminal prompt.**

> � **Analogy:** Think of a virtual environment as a separate toolbox for each project. Without one, every Python project on your machine shares the exact same tools (packages) and versions — so one project needing `langchain 1.x` while another needs `0.3.x` would break at least one of them. A virtual environment gives each project its own isolated toolbox, so nothing conflicts.

> �💡 **Tip:** To exit the virtual environment later, simply run `deactivate`.

---

## 📦 Step 3: Create Requirements File

Create a file called `requirements.txt` with our core dependencies:

```bash
# Create the requirements file
cat > requirements.txt << 'EOF'
# Chat UI Framework
chainlit==2.11.1

# LLM Orchestration
langchain==1.3.11
langchain-core==1.4.8
langchain-openai==1.3.3

# HTTP requests for tools
httpx==0.28.1

# Environment variable management
python-dotenv==1.2.2

# MCP Integration
mcp==1.28.1

# MCP Adapters
langchain-mcp-adapters==0.3.0
EOF
```

Or manually create `requirements.txt` with this content:

```txt
# Chat UI Framework
chainlit==2.11.1

# LLM Orchestration
langchain==1.3.11
langchain-core==1.4.8
langchain-openai==1.3.3

# HTTP requests for tools
httpx==0.28.1

# Environment variable management
python-dotenv==1.2.2

# MCP Integration
mcp==1.28.1

# MCP Adapters
langchain-mcp-adapters==0.3.0
```

### 🔍 What Are These Packages?

| Package | Purpose |
|---------|---------|
| `chainlit` | Provides the chat web interface - handles UI, message history, and real-time streaming |
| `langchain` | Framework for building LLM applications - chains, agents, tools |
| `langchain-openai` | OpenAI-compatible integration (works with GitHub Models) |
| `httpx` | Modern async HTTP client for calling external APIs |
| `python-dotenv` | Loads environment variables from `.env` files |
| `mcp` | Model Context Protocol for standardized agent communication |

> 📘 **Why exact versions (`==`) instead of a range (`>=`)?** Pinning to an exact version means everyone installs the *identical* release. Libraries move fast — a newer version could rename a function or change behavior — so exact pins avoid the classic "it works on my machine but not yours" problem during a live workshop. In your own projects later, you'll often relax this once you've tested compatibility.

---

## 📥 Step 4: Install Dependencies

{% tabs %}
{% tab title="Using uv (Recommended)" %}
```bash
uv pip install -r requirements.txt
```
This typically completes in 5-10 seconds! ⚡
{% endtab %}

{% tab title="Using pip" %}
```bash
pip install -r requirements.txt
```
This may take 1-2 minutes.
{% endtab %}
{% endtabs %}

---

## 📄 Step 5: Create Environment File

We'll store sensitive data (API keys) in a `.env` file. Create it now as a placeholder:

> 📘 **What's an "environment variable"?** It's a named value stored outside your code, at the level of your operating system or terminal session, that any program can read. In Python we read them with `os.getenv("NAME")`. We use them for secrets like API keys so those secrets never get hard-coded directly into files that might be shared, screen-shared, or committed to git.

```bash
cat > .env << 'EOF'
# GitHub Models (we'll fill this in Phase 2)
GITHUB_TOKEN=your_github_token_here

# WeatherAPI (we'll fill this in Phase 5)
WEATHER_API_KEY=your_weather_api_key_here
EOF
```

### 🔒 Security Note

The `.env` file should **never** be committed to git. Let's create a `.gitignore`:

> 📘 **Why does this matter?** Git tracks every change to every committed file — forever, in the project's history. If a secret like `GITHUB_TOKEN` ends up in a commit, deleting it later isn't enough; it still exists in that history and anyone with repo access (or a public GitHub search) could find it. `.gitignore` tells Git "never track this file at all," which is the safest way to keep secrets out of version control entirely.

```bash
cat > .gitignore << 'EOF'
# Virtual environment
.venv/
venv/

# Environment variables (secrets!)
.env

# Python cache
__pycache__/
*.pyc

# Chainlit files
.chainlit/
EOF
```

---

## 🗂️ Current Project Structure

Your project should now look like this:

```
langchain-chainlit-workshop/
├── .venv/              # Virtual environment (hidden folder)
├── .env                # Environment variables (we'll fill later)
├── .gitignore          # Git ignore rules
└── requirements.txt    # Python dependencies
```

---

## ✅ Checkpoint: Environment Ready

Let's verify everything is set up correctly:

### Test 1: Virtual Environment Active
```bash
which python
# Should show: /path/to/langchain-chainlit-workshop/.venv/bin/python
```

On Windows:
```powershell
where python
# Should show path containing .venv
```

### Test 2: Packages Installed
```bash
python -c "import chainlit; print(f'Chainlit version: {chainlit.__version__}')"
python -c "import langchain; print(f'LangChain version: {langchain.__version__}')"
```

**Expected output**:
```
Chainlit version: 2.11.1
LangChain version: 1.3.11
```

### Test 3: Environment File Exists
```bash
cat .env
# Should show the template with placeholder values
```

### ✨ All Tests Pass?

**Great job!** Your development environment is ready.

👉 **Next up: [Phase 2: GitHub Models Configuration](02-github-models.md)**

---

## ❓ Common Issues

### "uv: command not found"
Use pip instead:
```bash
python3 -m venv .venv
source .venv/bin/activate  # or .venv\Scripts\activate on Windows
pip install -r requirements.txt
```

### "No module named chainlit"
Make sure your virtual environment is activated. You should see `(.venv)` in your prompt.

### Installation is very slow
- Check your internet connection
- Try using uv for faster installs
- If behind a corporate proxy, you may need to configure pip

### "Permission denied" errors
- Don't use `sudo` with pip in a virtual environment
- Make sure you activated the virtual environment first
