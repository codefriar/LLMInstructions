# LLM-Agentic-Code-Instructions

A community-driven repository for best-practice, multi-contributor "instruction" files tailored for Large Language Model (LLM) coding agents like Claude-code, Codex, and others. These files help guide code generation, enforce standards, and improve project consistency when using agentic code tools.

---

## 🚀 What is This?

This repository is a central place for high-quality, community-curated instruction files that teach LLM-based coding agents how to write code for specific languages, frameworks, or projects.

- **Instruction files** help LLMs like Claude, Codex, etc., generate code that matches your team's standards and practices.
- Contributions welcome for any language, framework, or use case!

---

## 🤖 Supported LLM Coding Agents

- [Claude-code by Anthropic](https://www.anthropic.com/)
- [OpenAI Codex](https://platform.openai.com/docs/guides/code)
- ...and others! PRs for more agent instructions encouraged.

---

## 📁 Directory Structure

```
instructions/
  claude/
    ruby-best-practices.md
    python-flask-best-practices.md
  codex/
    js-react-best-practices.md
    go-microservices-best-practices.md
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
   /instructions/claude/
   ```
   Example: `/instructions/claude/ruby-best-practices.md`

2. **How to Use**
   - When initializing your Claude agent for a project, load or reference the appropriate file(s) from `/instructions/claude/`.
   - Optionally, summarize or paste file contents into Claude's system prompt or context window.

3. **Project-Specific Instructions**
   - For project-specific conventions, add a file in `/instructions/claude/project-name.md`.

### For Codex (OpenAI)

1. **Where to Place Instruction Files**
   Place instruction files in your project repository under:
   ```
   /instructions/codex/
   ```
   Example: `/instructions/codex/python-flask-best-practices.md`

2. **How to Use**
   - When setting up your prompt, include the most relevant instruction file content at the start of your prompt, or reference it in your application logic.
   - For best results, keep the instruction files concise and focused.

3. **Project-Specific Instructions**
   - Add custom files in `/instructions/codex/project-name.md` for unique project standards.

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