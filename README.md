# UIGen

![UIGen Banner](https://raw.githubusercontent.com/yourusername/yourrepo/main/.github/assets/banner.png)

[![License](https://img.shields.io/badge/license-MIT-blue.svg)](LICENSE)
[![Node.js](https://img.shields.io/badge/node-%3E%3D18-brightgreen.svg)](https://nodejs.org/)
[![Build Status](https://github.com/yourusername/yourrepo/actions/workflows/claude.yml/badge.svg)](https://github.com/yourusername/yourrepo/actions)
[![npm version](https://img.shields.io/npm/v/your-package.svg)](https://www.npmjs.com/package/your-package)

---

## Project Overview

**UIGen** is an AI‑powered React component generator that lets developers describe UI components in natural language and instantly receive production‑ready React code with a live preview. The tool runs entirely in the browser with a virtual file system, ensuring a fast, secure, and sandboxed experience.

---

## Features

- **AI‑driven component generation** using Anthropic Claude.
- **Live preview** with hot‑reloading for immediate visual feedback.
- **Virtual file system** – no files are written to disk unless you explicitly export them.
- **Rich code editor** with syntax highlighting, auto‑completion, and error detection.
- **User persistence** – signed‑in users can save and manage their generated components.
- **Export functionality** to download the generated project as a zip.
- **Responsive design** powered by Tailwind CSS v4.
- **Extensible architecture** – easy to add new AI providers or storage back‑ends.

---

## Installation

### Prerequisites

- **Node.js** version 18 or newer
- **npm** (comes with Node) or **yarn**

### Setup Steps

1. **Clone the repository**
   ```bash
   git clone https://github.com/yourusername/yourrepo.git
   cd yourrepo
   ```

2. **Create an environment file** (optional – required for AI generation)
   ```bash
   cp .env.example .env
   # Edit .env and add your Anthropic API key if you want AI generation
   # ANTHROPIC_API_KEY=your-key-here
   ```

3. **Install dependencies and set up the database**
   ```bash
   npm run setup
   ```
   This script will:
   - Install all npm packages
   - Generate the Prisma client
   - Apply SQLite migrations

---

## Usage

### Development Server

Start the development server with hot‑reloading:
```bash
npm run dev
```
Open your browser at **http://localhost:3000**.

### Generating Components

1. **Sign in** or continue as an anonymous user.
2. In the chat interface, describe the component you need (e.g., "Create a responsive navigation bar with a dark theme").
3. The AI will generate the component files and display a live preview.
4. Switch to the **Code** tab to view/edit the generated files.
5. Iterate by providing additional instructions to refine the UI.

### Exporting

Click the **Export** button to download a zip archive containing the generated project files.

---

## Contributing

We welcome contributions! Please follow these steps:

1. **Fork the repository**.
2. **Create a feature branch**:
   ```bash
   git checkout -b feature/your-feature-name
   ```
3. **Make your changes** and ensure the test suite passes:
   ```bash
   npm run test
   ```
4. **Commit your changes** with clear commit messages.
5. **Push to your fork** and open a Pull Request.

Please read our [CODE_OF_CONDUCT.md](CODE_OF_CONDUCT.md) and adhere to the project's coding standards.

---

## License

This project is licensed under the **MIT License** – see the [LICENSE](LICENSE) file for details.

---

## Contact

- **Author**: Your Name (<your.email@example.com>)
- **GitHub**: https://github.com/yourusername/yourrepo
- **Twitter**: @yourhandle (optional)

Feel free to open an issue for bugs, feature requests, or general questions.

---

*Happy coding with UIGen!*
