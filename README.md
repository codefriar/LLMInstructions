# LLM-Agentic-Code-Instructions

A community-driven repository for best-practice, multi-contributor "instruction" files tailored for Large Language Model (LLM) coding agents such as Claude-code, Codex, and others. These files help guide code generation, enforce standards, and improve project consistency when using agentic code tools.

---

## 🚀 What is This?

This repository is a central location for high-quality, community-curated instruction files that teach LLM-based coding agents how to write code for specific languages, frameworks, or projects.

***Instruction files*** help LLMs like Claude, Codex, and others generate code that matches your team's standards and practices.
Contributions are welcome for any language, framework, or use case!

---

## 🤖 Supported LLM Coding Agents

- [Claude-code by Anthropic](https://www.anthropic.com/)
- [OpenAI Codex](https://platform.openai.com/docs/guides/code)
- ...and others! PRs for more agent instructions encouraged.

---

## 📁 Directory Structure

```
ruby/
  CLAUDE.md
  AGENT.md

swift/
  CLAUDE.md
  AGENT.md

salesforce/
  CLAUDE.md
  AGENT.md
...
CONTRIBUTING.md
CODE_OF_CONDUCT.md
README.md
```

---

## 📚 Usage Instructions

### For Claude-code

1. **Where to Place Instruction Files**
   Place relevant instruction files in your project repository under:
   ```
   CLAUDE.md
   ```
   Example: `CLAUDE.md`

2. **How to Use**
   - When initializing your Claude agent for a project, load or reference the appropriate file(s) from `/Language-Name/Vendor.md`.
   - Optionally, summarize or paste file contents into Claude's system prompt or context window.

### For Codex (OpenAI)

1. **Where to Place Instruction Files**
   Place instruction files in your project repository root:
   ```
   AGENTS.md
   ```
   Example: `AGENT.md`

2. **How to Use(Advanced)**
   - place your LLM specific instruction file in the appropriate place within your project directory. For specific instructions per LLM vendor, keep reading.

3. **Vendor Specific Instructions**
    - **OpenAI Codex**
        - copy this repo's Language-Name/AGENTS.md file to:
            - ~/.codex/AGENTS.md - personal global guidance
            - AGENTS.md at your projects checkout root - shared project notes
            - AGENTS.md in the current working directory - sub-folder/feature specifics
    - **Anthropic Claude**
        - copy this repo's Language-Name/CLAUDE.md file to:
            - ~/.claude/CLAUDE.md - for personal global inclusion in all projects.
            - CLAUDE.md in your current projects checkout root - for project specific instruction

---

## 🌱 Contributing

We welcome all contributions! See [CONTRIBUTING.md](CONTRIBUTING.md) for guidelines.

- Add instruction files for new languages, frameworks, or LLM agents.
- Improve or update existing files.
- Discuss or propose new formats and best practices.

---

## 🤝 Code of Conduct

Please review our [CODE_OF_CONDUCT.md](CODE_OF_CONDUCT.md) before contributing.  
We are committed to a welcoming, inclusive, and respectful environment for all contributors.

---

## 📢 Join the Community!

- Open issues for suggestions or requests.
- Submit PRs to add or improve instruction files.
- Share this project with your team or community!

---

**Let’s make LLM coding agents more effective, ethical, and community-driven, one instruction file at a time.**