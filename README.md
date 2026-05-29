# vsix-vscode-extension-development-skill

[![skills.sh](https://img.shields.io/endpoint?url=https://www.skills.sh/api/badge/dealenx/vsix-vscode-extension-development-skill)](https://www.skills.sh/dealenx/vsix-vscode-extension-development-skill)

Comprehensive guide for developing VS Code extensions (.vsix). Based on official [Microsoft vscode-extension-samples](https://github.com/microsoft/vscode-extension-samples) and [vscode-extension-and-mcp-together](https://github.com/dealenx/vscode-extension-and-mcp-together).

## Установка

Установить глобально:

```bash
npx skills add dealenx/vsix-vscode-extension-development-skill -g
```

Установить в проект:

```bash
npx skills add dealenx/vsix-vscode-extension-development-skill
```

## What This Skill Covers

- **Project scaffolding** — structure, tsconfig, package.json templates
- **package.json reference** — all contribution points, activation events, extension kind
- **Commands** — registration, menus, keybindings, when clauses
- **UI components** — status bar, notifications, quick pick, input box, progress
- **Language features** — completions, CodeLens, code actions, diagnostics, hover, definition, semantic tokens, signature help, call hierarchy
- **Tree views** — TreeDataProvider, drag-and-drop, view containers
- **Webview** — panel, sidebar view, CSP/nonce security, message passing, singleton pattern
- **LSP** — LanguageClient setup, multi-root servers
- **MCP & Language Model Tools** — native LM tools, Tool base class pattern, chat participants, MCP server definition providers
- **Custom editors & notebooks** — text/binary editors, notebook serializer
- **File system providers** — virtual filesystem
- **Authentication** — custom auth providers, GitHub auth
- **Configuration** — settings, scopes, reading/writing, change listeners
- **Snippets & language config** — snippet files, language-configuration.json
- **Themes** — color themes, icon themes, product icon themes, themable colors
- **Telemetry & localization** — telemetry logger, l10n API
- **Testing** — @vscode/test-electron, @vscode/test-cli, Mocha setup
- **Build & bundling** — tsc, esbuild (recommended), webpack comparison
- **Debugging** — launch.json, tasks.json
- **Publishing** — vsce packaging and marketplace publishing
- **Best practices** — activation, disposables, webview security, MCP tools, build optimization
- **Code templates** — minimal extension, extension with MCP tools, extension with esbuild, extension with tests
- **Pattern reference** — disposable registration, singleton webview, event-driven, configuration watcher, document change watcher, secret storage, global state

## Key Design Decisions from Research

### From Microsoft Official Samples (70+ examples)

1. **Activation events**: Modern VS Code uses `activationEvents: []` (auto-detect from contributes). Only use explicit events for non-standard activation.
2. **Build tool preference**: esbuild is the recommended bundler — fastest builds, simplest config.
3. **Disposable pattern**: Every `register*()` call returns a Disposable; always push to `context.subscriptions`.
4. **LSP architecture**: Separate `client/` and `server/` directories with independent package.json files.
5. **Testing**: `@vscode/test-cli` (newer, simpler) over raw `@vscode/test-electron` orchestration.

### From vscode-extension-and-mcp-together

1. **Template Method pattern for MCP tools**: Abstract `Tool` base class with `invoke()` (template) and `call()` (abstract hook) — subclasses only implement `call()` returning `Promise<string>`.
2. **Dual declaration**: `contributes.languageModelTools` in package.json (metadata) + `vscode.lm.registerTool()` in code (implementation). The `name` field must match `toolName`.
3. **Error resilience**: `Tool.invoke()` always returns `LanguageModelToolResult`, never throws — errors are structured JSON payloads.
4. **Zero runtime dependencies**: The extension uses only the `vscode` API module, no npm runtime deps.
5. **`canBeReferencedInPrompt: true`**: Enables `#toolName` syntax in Copilot Chat for explicit user invocation.
6. **`modelDescription`**: The prompt fragment sent to the AI model — write it carefully as it determines when/how the AI uses your tool.

## License

MIT