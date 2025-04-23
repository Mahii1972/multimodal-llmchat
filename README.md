# Vue 3 Chatbot

A Vue 3 chatbot application similar to a Rust CLI tool, with features like code block formatting, syntax highlighting, and model selection.

## Features

- Select from multiple AI models
- Session-based chat history
- Code block formatting with show/hide and copy options
- Modern UI with Tailwind CSS

## Setup

1. Clone the repository
2. Install dependencies with `npm install`
3. Copy `.env.example` to `.env` and add your webhook URL
4. Run the development server with `npm run dev`

## Usage

1. Select a model from the dropdown
2. Start a chat session
3. Type your messages and press Send
4. View the responses with formatted code blocks
5. End the session when done

## Environment Variables

- `VITE_WEBHOOK_URL`: Your API webhook URL

## Recommended IDE Setup

[VSCode](https://code.visualstudio.com/) + [Volar](https://marketplace.visualstudio.com/items?itemName=Vue.volar) (and disable Vetur).

## Type Support for `.vue` Imports in TS

TypeScript cannot handle type information for `.vue` imports by default, so we replace the `tsc` CLI with `vue-tsc` for type checking. In editors, we need [Volar](https://marketplace.visualstudio.com/items?itemName=Vue.volar) to make the TypeScript language service aware of `.vue` types.

## Customize configuration

See [Vite Configuration Reference](https://vite.dev/config/).

## Project Setup

```sh
npm install
```

### Compile and Hot-Reload for Development

```sh
npm run dev
```

### Type-Check, Compile and Minify for Production

```sh
npm run build
```

### Lint with [ESLint](https://eslint.org/)

```sh
npm run lint
```
