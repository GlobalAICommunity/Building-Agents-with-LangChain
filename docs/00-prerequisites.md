# Prerequisites

> ⏱️ **Time to complete**: 5 minutes (or done before the workshop)

Before we dive into building our AI chat agent, let's make sure you have everything ready. This phase is a quick check to ensure a smooth workshop experience.

{% hint style="info" %}
**New to programming or AI?** Welcome — this workshop assumes **zero prior experience** with LangChain, Chainlit, or building AI agents. Here are a few terms you'll see constantly, as a quick preview (we'll re-explain each one again, in context, the first time it actually matters):

- **Terminal (command line)**: A text-based way to control your computer by typing commands instead of clicking icons. Every step in this workshop happens here.
- **LLM (Large Language Model)**: The "brain" behind AI chat tools like ChatGPT — a model trained on huge amounts of text that predicts and generates human-like language.
- **API (Application Programming Interface)**: A way for two pieces of software to talk to each other, usually over the internet. We'll use an API to send text to an LLM and get a reply back.
- **API key / token**: A secret string that proves "this request is allowed" — like a password, but for software instead of humans.
- **Package**: A bundle of reusable code someone else already wrote (e.g., `chainlit`, `langchain`) that we install instead of writing everything from scratch.

Don't worry about memorizing these — just skim and move on.
{% endhint %}

---

## 🎒 What You Need

### 1. **A Computer with Terminal Access**

| OS | Terminal |
|----|----------|
| macOS | Terminal.app or iTerm2 |
| Windows | PowerShell, CMD, or Windows Terminal |
| Linux | Any terminal emulator |

> 💡 **Never used a terminal before?** No problem — this workshop only needs a handful of commands (mostly `cd`, `python`, and `pip`/`uv`). Every step tells you exactly what to type, so you can copy-paste if you're unsure.

### 2. **Python 3.10 or Higher**

Check your Python version:

```bash
python3 --version
# or on Windows:
python --version
```

**Expected output**: `Python 3.10.x` or higher (3.11, 3.12, 3.13 are all fine)

{% hint style="info" %}
**Don't have Python?** 
- macOS: `brew install python@3.12`
- Windows: Download from [python.org](https://www.python.org/downloads/)
- Linux: `sudo apt install python3.12` (Ubuntu/Debian)
{% endhint %}

> 📘 **Why does the version matter?** Newer libraries (like `langchain`) use Python language features that don't exist in older releases. If your Python is too old, `pip install` will fail with a confusing error — checking this now saves time later.

### 3. **Git Installed**

```bash
git --version
```

**Expected output**: `git version 2.x.x`

### 4. **A Code Editor**

We recommend **Visual Studio Code** with the Python extension, but any editor works.

### 5. **A GitHub Account**

You'll need this to access GitHub Models (free LLM API). 

👉 No account? Create one at [github.com](https://github.com/signup)

> 💡 **Note:** In Phase 2, you'll generate a "Personal Access Token" (PAT) from this account — think of it as a temporary password that lets our Python code talk to GitHub's AI models on your behalf. There's nothing to set up yet; we'll walk through it step by step.

### 6. **Internet Connection**

Required for:
- Downloading packages
- Calling GitHub Models API
- Calling WeatherAPI

---

## 📋 Pre-Workshop Checklist

Run through this checklist. You should be able to check all boxes:

- [ ] Python 3.10+ installed and accessible from terminal
- [ ] Git installed
- [ ] GitHub account created and you can log in
- [ ] Code editor ready (VS Code recommended)
- [ ] Stable internet connection

---

## 🔑 Accounts to Create (If You Haven't Already)

### GitHub Account
If you don't have one:
1. Go to [github.com/signup](https://github.com/signup)
2. Follow the registration process
3. Verify your email address

### WeatherAPI Account (Free)
We'll use this for the tool-calling demo:
1. Go to [weatherapi.com](https://www.weatherapi.com/)
2. Click "Sign Up" (top right)
3. Choose the **Free** plan
4. You'll get an API key after email verification

{% hint style="warning" %}
**Save your WeatherAPI key!** You'll need it in Phase 5.
{% endhint %}

---

## 💻 Optional but Recommended

### uv (Fast Python Package Manager)

We'll use `uv` for faster dependency installation. It's optional but makes things much faster:

```bash
# macOS/Linux
curl -LsSf https://astral.sh/uv/install.sh | sh

# Windows (PowerShell)
powershell -ExecutionPolicy ByPass -c "irm https://astral.sh/uv/install.ps1 | iex"
```

Verify installation:
```bash
uv --version
```

If you don't want to install `uv`, that's fine - we'll provide `pip` alternatives for every command.

---

## ✅ Checkpoint: Prerequisites Complete

Before moving on, verify:

| Check | Command | Expected |
|-------|---------|----------|
| Python | `python3 --version` | 3.10+ |
| Git | `git --version` | 2.x.x |
| GitHub | Log into github.com | Success |
| WeatherAPI | Check email for API key | Have the key |

### 🎉 All Green?

**Excellent!** You're ready for the workshop. 

👉 **Next up: [Phase 1: Environment Setup](01-environment.md)**

---

## ❓ Common Issues

### "python3: command not found"
- Windows: Try `python` instead of `python3`
- macOS: Install via `brew install python@3.12`
- Make sure Python is in your PATH

### "I forgot my WeatherAPI key"
- Log into [weatherapi.com](https://www.weatherapi.com/)
- Go to Dashboard → Your API Key
- You can regenerate it if needed

### "I can't create a GitHub account"
- Ask the workshop facilitator for a shared demo token
- You can pair with someone who has an account
