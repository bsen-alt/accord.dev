# Accord Developer Content

Official documentation, tutorials, and API reference for the **Accord** policy engine.

This repository contains the source content for the documentation and learning materials hosted at [navirondynamics.com/accord](https://navirondynamics.com/accord).

---

## 📚 What is this repository?

`accord.dev` is the home for all narrative documentation, API references, and guides for the Accord project.

While the source code lives in the [Accord](https://github.com/bsen-alt/Accord) repository, this repository is dedicated to **content**. It allows the community to improve documentation, fix typos, and add tutorials without needing to modify the core library codebase.

---

## 📂 Structure

The repository is divided into two main sections:

- **`learn/`**: Narrative content, tutorials, and guides.
  - `introduction/`: "What is Accord?", "System of Agreement".
  - `getting-started/`: Installation, first policy, server mode.
  - `advanced/`: Simulation engine, rollback, graph visualization.
  - `blog/`: Release notes, announcements, and ecosystem updates.
- **`docs/`**: Technical reference material.
  - `api/`: Detailed API documentation (e.g., `Accord.create`, `accord.check`).
  - `cli/`: Command-line interface reference (e.g., `serve`, `policy rollback`).

---

## 🤝 Contributing

We welcome contributions to improve the documentation! You can help by:

1. **Fixing Issues:** Correcting typos, broken links, or grammatical errors.
2. **Improving Clarity:** Rewriting confusing sections to make them easier to understand.
3. **Adding Examples:** Providing real-world code snippets and use cases.
4. **Translation:** (Future) Localizing content for different regions.

**How to Contribute:**

1. Fork this repository.
2. Create a new branch (`git checkout -b feature/my-doc-update`).
3. Make your changes to the `.mdx` or `.md` files.
4. Submit a Pull Request.

---

## 🛠 Tech Stack

This content is written using **MDX** (Markdown + JSX). This allows us to embed interactive React components—such as live code editors or the Accord simulation interface—directly into the documentation.

- **Format:** MDX
- **Engine:** React
- **Deployment:** The content is pulled and rendered by the Naviron Dynamics website platform.

---

## 🔗 Links

- **Main Library:** [navirondynamics/accord](https://github.com/bsen-alt/Accord)
- **Official Website:** [navirondynamics.com/accord](https://navirondynamics.com/accord)
- **NPM Package:** [@navirondynamics/accord](https://www.npmjs.com/package/@navirondynamics/accord)

---

## 📜 License

This documentation is part of the Accord project and is licensed under the **ISC License**, the same as the core library.

---

**Accord** by Naviron Dynamics. All systems nominal.
