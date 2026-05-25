---
layout: home

hero:
  name: Enterprise Docs
  text: Enterprise Documentation Solution
  tagline: Modern documentation framework based on VitePress with multi-language support and theme switching, designed for team collaboration
  image:
    src: /hero-image.svg
    alt: Enterprise Docs
  actions:
    - theme: brand
      text: Get Started
      link: /en/guide/
    - theme: alt
      text: View on GitHub
      link: https://github.com/your-org/your-repo
    - theme: alt
      text: v1.0.0
      link: /en/release-notes

features:
  - icon: 🚀
    title: Ready to Use
    details: Built on VitePress, enjoy an ultimate development experience and fast build speed.
  - icon: 🌍
    title: Internationalization
    details: Built-in Chinese and English support, easily extend to more languages.
  - icon: 🎨
    title: Theme Customization
    details: Support light/dark theme switching, inspired by Vue.js official website design.
  - icon: 📱
    title: Responsive Design
    details: Perfect adaptation for desktop and mobile, providing consistent user experience.
  - icon: 🔍
    title: Powerful Search
    details: Local search support, quickly find the content you need.
  - icon: 📝
    details: Support Markdown extended syntax, making document writing more efficient.
---

## Quick Start

### Installation

::: code-group

```bash [npm]
npm create vitepress@latest my-docs -- --template enterprise
```

```bash [yarn]
yarn create vitepress my-docs --template enterprise
```

```bash [pnpm]
pnpm create vitepress my-docs --template enterprise
```

:::

### Basic Usage

```bash
# Install dependencies
npm install

# Start development server
npm run dev

# Build for production
npm run build
```

## Project Structure

```
.
├── docs
│   ├── .vitepress
│   │   ├── config.ts          # Configuration file
│   │   └── theme              # Theme directory
│   │       ├── index.ts       # Theme entry
│   │       ├── components     # Custom components
│   │       └── styles         # Style files
│   ├── guide                  # Guide documentation
│   ├── components             # Component documentation
│   ├── api                    # API documentation
│   └── examples               # Example code
├── public                     # Static assets
└── package.json
```

## Core Features

### 🎯 Enterprise Features

- **Permission Management**: Built-in role-based access control system
- **Version Control**: Support for document versioning and history tracking
- **Collaborative Editing**: Multi-person collaborative editing and commenting
- **SEO Optimization**: Search engine optimization to improve document visibility

### 💻 Developer Friendly

- **TypeScript Support**: Complete TypeScript type definitions
- **Hot Reload**: Real-time preview of changes during development
- **Component-based**: Reusable Vue component system
- **Plugin System**: Flexible plugin extension mechanism

### 🎨 Design System

- **Design Tokens**: Unified design variable management
- **Responsive Layout**: Adaptive to different screen sizes
- **Dark Mode**: System-level dark mode support
- **Accessibility**: Following WCAG accessibility guidelines

## Contributing

We welcome all forms of contributions! Please read our [Contributing Guide](/en/contributing) to learn how to participate in project development.

## License

[MIT](https://choosealicense.com/licenses/mit/) Licensed © 2024-Present [Your Company](https://yourcompany.com)