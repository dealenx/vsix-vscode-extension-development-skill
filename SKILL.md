---
name: vsix-vscode-extension-dev
description: Use when developing VS Code extensions (.vsix), VS Code API, package.json contributes, activation events, webview, language providers, tree views, MCP/Language Model tools, or any vscode extension API work. Covers scaffolding, bundling, testing, debugging, and publishing.
---

# VS Code Extension (VSIX) Development Skill

Comprehensive guide for developing VS Code extensions based on official Microsoft samples and community best practices.

## Table of Contents

1. [Project Structure & Scaffolding](#1-project-structure--scaffolding)
2. [package.json Reference](#2-packagejson-reference)
3. [Activation & Lifecycle](#3-activation--lifecycle)
4. [Commands](#4-commands)
5. [UI Components](#5-ui-components)
6. [Language Features](#6-language-features)
7. [Tree Views](#7-tree-views)
8. [Webview](#8-webview)
9. [Language Server Protocol (LSP)](#9-language-server-protocol-lsp)
10. [MCP & Language Model Tools](#10-mcp--language-model-tools)
11. [Custom Editors & Notebooks](#11-custom-editors--notebooks)
12. [File System Providers](#12-file-system-providers)
13. [Authentication](#13-authentication)
14. [Configuration & Settings](#14-configuration--settings)
15. [Snippets & Language Config](#15-snippets--language-config)
16. [Themes](#16-themes)
17. [Telemetry & Localization](#17-telemetry--localization)
18. [Testing](#18-testing)
19. [Build & Bundling](#19-build--bundling)
20. [Debugging](#20-debugging)
21. [Publishing](#21-publishing)
22. [Best Practices](#22-best-practices)
23. [Code Templates](#23-code-templates)
24. [Common Patterns Reference](#24-common-patterns-reference)

---

## 1. Project Structure & Scaffolding

### Minimal Extension Structure

```
my-extension/
  .vscode/
    launch.json          # F5 debug configuration
    tasks.json            # Build task (npm: watch)
  src/
    extension.ts          # Entry point: activate() / deactivate()
  package.json            # Extension manifest
  tsconfig.json           # TypeScript config
  .gitignore              # out, node_modules, .vscode-test/, *.vsix
```

### Extension with Multiple Modules

```
my-extension/
  src/
    extension.ts          # activate(), delegates to modules
    commands/
      myCommand.ts
    providers/
      myProvider.ts
    tools/
      tool.ts             # Abstract MCP tool base
      myTool.ts           # Concrete tool
  package.json
  tsconfig.json
```

### LSP Extension Structure

```
my-lsp-extension/
  client/
    src/
      extension.ts        # LanguageClient setup
    package.json
    tsconfig.json
  server/
    src/
      server.ts            # Language server implementation
    package.json
    tsconfig.json
  package.json             # Root: postinstall runs both client & server installs
```

### Scaffolding with yo code

```bash
npm install -g yo generator-code
yo code
# Choose: New Extension (TypeScript), New Extension (JavaScript),
#         New Color Theme, New Language Support, New Code Snippets, etc.
```

---

## 2. package.json Reference

### Essential Fields

```json
{
  "name": "my-extension",
  "displayName": "My Extension",
  "description": "What the extension does",
  "version": "0.0.1",
  "publisher": "your-publisher-id",
  "engines": { "vscode": "^1.100.0" },
  "categories": ["Programming Languages", "Snippets", "Other"],
  "activationEvents": [],
  "main": "./out/extension.js",
  "contributes": { ... },
  "scripts": {
    "vscode:prepublish": "npm run compile",
    "compile": "tsc -p ./",
    "watch": "tsc -watch -p ./",
    "lint": "eslint",
    "pretest": "npm run compile",
    "test": "node ./out/test/runTest.js"
  },
  "devDependencies": {
    "@types/vscode": "^1.100.0",
    "@types/node": "^22",
    "typescript": "^5.8.2",
    "eslint": "^9.13.0",
    "@eslint/js": "^9.13.0",
    "typescript-eslint": "^8.26.0",
    "@stylistic/eslint-plugin": "^2.9.0"
  }
}
```

### All Contribution Points

| Contribution Point | Description | Key Fields |
|---|---|---|
| `commands` | Command palette commands | `command`, `title`, `category` |
| `menus` | Menu item placements | `command`, `when`, `group`, `alt` |
| `keybindings` | Keyboard shortcuts | `command`, `key`, `mac`, `when` |
| `configuration` | Settings | `title`, `properties` (with type, default, scope) |
| `configurationDefaults` | Default editor/workspace settings | Language-specific overrides |
| `colors` | Themable colors | `id`, `defaults: {light, dark, highContrast}` |
| `snippets` | Code snippets | `language`, `path` |
| `languages` | Language definitions | `id`, `extensions`, `aliases`, `configuration` |
| `grammars` | TextMate grammars | `language`, `scopeName`, `path` |
| `themes` | Color themes | `label`, `uiTheme`, `path` |
| `iconThemes` | File icon themes | `id`, `label`, `path` |
| `productIconThemes` | Product icon themes | `id`, `label`, `path` |
| `views` | Sidebar panels | `id`, `name`, `type`, `when` |
| `viewsContainers` | Activity bar / sidebar panels | `id`, `title`, `icon` |
| `viewsWelcome` | Welcome content for views | `view`, `contents`, `when` |
| `customEditors` | Custom editor providers | `viewType`, `selector`, `priority` |
| `notebooks` | Notebook types | `type`, `displayName`, `selector` |
| `terminal` | Terminal profiles | `id`, `title`, `icon` |
| `debuggers` | Debug adapters | `type`, `label`, `runtime`, `configurationAttributes` |
| `problemMatchers` | Task problem matchers | `name`, `owner`, `pattern` |
| `taskDefinitions` | Task types | `type`, `required`, `properties` |
| `authentication` | Auth providers | `id`, `label` |
| `chatParticipants` | Copilot Chat participants | `id`, `name`, `commands` |
| `languageModelTools` | AI/LM tools for Copilot | `name`, `displayName`, `modelDescription`, `inputSchema` |
| `mcpServerDefinitionProviders` | MCP server providers | `id`, `label` |
| `l10n` | Localization bundle path | `"./l10n"` |

### Extension Kind (Remote Support)

```json
{
  "extensionKind": ["ui", "workspace"]
}
```

- `"ui"`: Extension runs on the local machine (UI extensions: themes, webviews, keybindings)
- `"workspace"`: Extension runs on the remote machine (language features, debuggers, tools)
- Order matters: `["ui", "workspace"]` = prefer local, fallback to remote

---

## 3. Activation & Lifecycle

### Activation Events (Modern Approach)

In modern VS Code (1.74+), `activationEvents` can be omitted or set to `[]`. VS Code auto-detects activation from `contributes`. Only use explicit events when you need non-standard activation.

### Available Activation Events

```json
{
  "activationEvents": [
    "*",                                    // Always activate (AVOID unless necessary)
    "onLanguage:javascript",                 // Language file opened
    "onCommand:myExt.myCmd",                 // Command invoked
    "onDebug",                                // Debug session starts
    "onDebugResolve:type",                   // Debug resolve
    "onDebugInitialConfigurations",          // Debug configs requested
    "onFileSystem:scheme",                   // FS scheme accessed
    "onView:viewId",                         // View opened
    "onWebviewPanel:viewType",              // Webview panel opened
    "onUri:myExt",                           // URI handler invoked
    "onNotebook:type",                       // Notebook opened
    "workspaceContains:.eslintrc",           // File exists in workspace
    "onStartupFinished"                      // After VS Code fully loaded
  ]
}
```

### Activation Best Practices

| Pattern | When to Use | Example |
|---|---|---|
| `[]` (empty, auto) | Default for most extensions | Commands, views, themes, configs |
| `onLanguage:` | Language features | LSP, completions, diagnostics |
| `onCommand:` | Only when command is not in `contributes` | Dynamic commands |
| `onStartupFinished` | Background tasks that should not slow startup | Indexing, polling |
| `*` | ONLY when absolutely necessary | Status bar, global event listeners |

### activate() / deactivate()

```typescript
export function activate(context: vscode.ExtensionContext) {
  // All disposables must go into context.subscriptions
  context.subscriptions.push(
    vscode.commands.registerCommand('myExt.hello', () => {
      vscode.window.showInformationMessage('Hello!');
    })
  );
}

export function deactivate() {
  // Only needed for async cleanup.
  // context.subscriptions handles synchronous disposal automatically.
  // Return a Thenable<string> | void for async cleanup.
}
```

---

## 4. Commands

### Basic Command

```typescript
const disposable = vscode.commands.registerCommand(
  'myExtension.helloWorld',
  () => {
    vscode.window.showInformationMessage('Hello World!');
  }
);
context.subscriptions.push(disposable);
```

### Command with Arguments

```typescript
// In package.json: "command": "myExt.openFile"
// Invoked via: vscode.commands.executeCommand('myExt.openFile', uri)
vscode.commands.registerCommand('myExt.openFile', (uri: vscode.Uri) => {
  vscode.window.showTextDocument(uri);
});
```

### Text Editor Command

```typescript
// Gets active textEditor automatically, disabled if no editor
vscode.commands.registerTextEditorCommand('myExt.reverseWord', (editor, edit) => {
  const selection = editor.selection;
  const text = editor.document.getText(selection);
  editor.edit(editBuilder => {
    editBuilder.replace(selection, text.split('').reverse().join(''));
  });
});
```

### Command with Progress

```typescript
vscode.commands.registerCommand('myExt.process', async () => {
  await vscode.window.withProgress(
    {
      location: vscode.ProgressLocation.Notification,
      title: 'Processing...',
      cancellable: true,
    },
    async (progress, token) => {
      token.onCancellationRequested(() => {
        vscode.window.showInformationMessage('Cancelled');
      });
      for (let i = 0; i < 100; i++) {
        if (token.isCancellationRequested) { break; }
        progress.report({ message: `Step ${i + 1}`, increment: 1 });
        await new Promise(r => setTimeout(r, 50));
      }
    }
  );
});
```

### package.json Command Contribution

```json
{
  "contributes": {
    "commands": [
      {
        "command": "myExt.hello",
        "title": "Say Hello",
        "category": "My Extension",
        "icon": {
          "light": "resources/light/icon.svg",
          "dark": "resources/dark/icon.svg"
        }
      }
    ]
  }
}
```

### Menu Context (when clause)

```json
{
  "contributes": {
    "menus": {
      "editor/context": [
        {
          "command": "myExt.hello",
          "when": "editorHasSelection && resourceExtname == .ts",
          "group": "myGroup@1"
        }
      ],
      "commandPalette": [
        {
          "command": "myExt.internalCmd",
          "when": "false"
        }
      ]
    }
  }
}
```

Common `when` clause keys:
- `editorHasSelection`, `editorFocus`, `editorTextFocus`
- `resourceExtname`, `resourceLangId`, `resourceFilename`
- `view == myViewId`, `viewItem =~ /itemType/`
- `isInDiffEditor`, `isInEmbeddedEditor`

---

## 5. UI Components

### Status Bar Item

```typescript
const statusItem = vscode.window.createStatusBarItem(
  vscode.StatusBarAlignment.Right,
  100 // priority: higher = further left/right
);
statusItem.text = '$(megaphone) Hello';
statusItem.command = 'myExt.statusClick';
statusItem.tooltip = 'Click for details';
statusItem.show();

context.subscriptions.push(statusItem);
context.subscriptions.push(
  vscode.window.onDidChangeActiveTextEditor(() => updateStatusBar(statusItem))
);
```

### Notifications

```typescript
// Information
vscode.window.showInformationMessage('Info message');

// With action buttons
const answer = await vscode.window.showInformationMessage(
  'Continue?',
  'Yes', 'No'
);

// Modal (blocks UI)
vscode.window.showInformationMessage('Important!', { modal: true }, 'OK');

// Warning
vscode.window.showWarningMessage('Warning!', 'Fix', 'Ignore');

// Error
vscode.window.showErrorMessage('Error occurred', 'Show Logs');

// Progress notification
vscode.window.withProgress({
  location: vscode.ProgressLocation.Notification,
  title: 'Working...',
  cancellable: true,
}, async (progress, token) => {
  progress.report({ increment: 10, message: 'Step 1' });
  // ...
});

// Window progress (in status bar area)
vscode.window.withProgress({
  location: vscode.ProgressLocation.Window,
  title: 'Indexing',
}, async () => { /* ... */ });
```

### Quick Pick

```typescript
// Simple pick
const result = await vscode.window.showQuickPick(
  ['Option A', 'Option B', 'Option C'],
  { placeHolder: 'Choose an option', canPickMany: false }
);

// Advanced QuickPick with items
const pick = vscode.window.createQuickPick();
pick.items = [
  { label: '$(file) File', description: 'Open a file', detail: 'Open file from disk' },
  { label: '$(folder) Folder', description: 'Open a folder', kind: vscode.QuickPickItemKind.Separator },
  { label: '$(globe) Remote', description: 'Open remote' },
];
pick.onDidChangeSelection(selection => {
  if (selection[0]) {
    vscode.window.showInformationMessage(`Selected: ${selection[0].label}`);
    pick.hide();
  }
});
pick.onDidHide(() => pick.dispose());
pick.show();
```

### Input Box

```typescript
const name = await vscode.window.showInputBox({
  prompt: 'Enter your name',
  placeHolder: 'John Doe',
  value: 'default',
  validateInput: (text) => {
    if (!text || text.length < 3) {
      return 'Name must be at least 3 characters';
    }
    return undefined; // valid
  }
});
```

### Multi-step Input Pattern

```typescript
async function multiStepInput() {
  const title = 'New Project';
  const state: Record<string, string> = {};

  // Step 1: Name
  const name = await vscode.window.showInputBox({
    title, step: 1, totalSteps: 3,
    prompt: 'Project name',
  });
  if (!name) { return; }
  state.name = name;

  // Step 2: Template
  const template = await vscode.window.showQuickPick(
    ['basic', 'react', 'vue'],
    { title, step: 2, totalSteps: 3 }
  );
  if (!template) { return; }
  state.template = template;

  // Step 3: Location
  const folder = await vscode.window.showOpenDialog({
    title, step: 3, totalSteps: 3,
    canSelectFolders: true,
  });
  // ...
}
```

---

## 6. Language Features

### Completion Item Provider

```typescript
vscode.languages.registerCompletionItemProvider(
  { scheme: 'file', language: 'mylang' },
  {
    provideCompletionItems(document, position, token, context) {
      const item1 = new vscode.CompletionItem('Hello');
      item1.kind = vscode.CompletionItemKind.Function;
      item1.detail = 'A greeting function';
      item1.documentation = new vscode.MarkdownString('says **Hello**');
      item1.insertText = new vscode.SnippetString('hello(${1:name})');
      item1.commitCharacters = ['.', ','];
      item1.command = { command: 'editor.action.triggerSuggest', title: 'Re-trigger' };

      const item2 = new vscode.CompletionItem('world');
      item2.kind = vscode.CompletionItemKind.Variable;

      return [item1, item2];
    },
  },
  '.', ':' // trigger characters
);
```

### Inline Completion Provider

```typescript
// NOTE: requires enabledApiProposals: ["inlineCompletionsAdditions"]
vscode.languages.registerInlineCompletionItemProvider(
  { pattern: '**' },
  {
    provideInlineCompletionItems(document, position, context, token) {
      const line = document.lineAt(position.line).text;
      const prefix = line.substring(0, position.character);
      if (prefix.endsWith('// ')) {
        return {
          items: [
            new vscode.InlineCompletionItem(
              'TODO: implement',
              new vscode.Range(position, position)
            ),
          ],
        };
      }
      return { items: [] };
    },
  }
);
```

### CodeLens Provider

```typescript
class CodelensProvider implements vscode.CodeLensProvider {
  private _onDidChangeCodeLenses = new vscode.EventEmitter<void>();
  onDidChangeCodeLenses = this._onDidChangeCodeLenses.event;

  provideCodeLenses(document: vscode.TextDocument): vscode.CodeLens[] {
    const config = vscode.workspace.getConfiguration('myExt');
    if (!config.get('enableCodeLens')) { return []; }

    const lenses: vscode.CodeLens[] = [];
    const regex = /TODO/g;
    const text = document.getText();
    let match;
    while ((match = regex.exec(text))) {
      const pos = document.positionAt(match.index);
      const range = document.lineAt(pos.line).range;
      lenses.push(new vscode.CodeLens(range, {
        title: 'Resolve TODO',
        command: 'myExt.resolveTodo',
        arguments: [range],
      }));
    }
    return lenses;
  }

  resolveCodeLens(lens: vscode.CodeLens): vscode.CodeLens {
    lens.command = {
      title: 'Click to resolve',
      command: 'myExt.resolveTodo',
    };
    return lens;
  }
}

context.subscriptions.push(
  vscode.languages.registerCodeLensProvider('*', new CodelensProvider())
);
```

### Code Actions (Quick Fix)

```typescript
class Emojizer implements vscode.CodeActionProvider {
  provideCodeActions(
    document: vscode.TextDocument,
    range: vscode.Range,
    context: vscode.CodeActionContext
  ): vscode.CodeAction[] {
    const text = document.getText(range);
    if (!text.includes(':)')) { return []; }

    const fix = new vscode.CodeAction(
      'Convert :) to emoji',
      vscode.CodeActionKind.QuickFix
    );
    fix.isPreferred = true;
    fix.edit = new vscode.WorkspaceEdit();
    fix.edit.replace(
      document.uri,
      range,
      text.replace(/:\)/g, '\u{1F642}')
    );
    return [fix];
  }
}

context.subscriptions.push(
  vscode.languages.registerCodeActionProvider(
    { scheme: 'file' },
    new Emojizer(),
    { providedCodeActionKinds: [vscode.CodeActionKind.QuickFix] }
  )
);
```

### Diagnostics

```typescript
const diagnosticCollection = vscode.languages.createDiagnosticCollection('myExt');
context.subscriptions.push(diagnosticCollection);

function updateDiagnostics(document: vscode.TextDocument) {
  const diagnostics: vscode.Diagnostic[] = [];
  const text = document.getText();
  const regex = /TODO/g;
  let match;
  while ((match = regex.exec(text))) {
    const pos = document.positionAt(match.index);
    const range = new vscode.Range(pos, pos.translate(0, 4));
    diagnostics.push(new vscode.Diagnostic(
      range,
      'TODO found',
      vscode.DiagnosticSeverity.Warning
    ));
  }
  diagnosticCollection.set(document.uri, diagnostics);
}

context.subscriptions.push(
  vscode.workspace.onDidChangeTextDocument(e => updateDiagnostics(e.document))
);
context.subscriptions.push(
  vscode.window.onDidChangeActiveTextEditor(e => {
    if (e) { updateDiagnostics(e.document); }
  })
);
```

### Hover Provider

```typescript
vscode.languages.registerHoverProvider('javascript', {
  provideHover(document, position, token) {
    const range = document.getWordRangeAtPosition(position);
    const word = document.getText(range);
    return new vscode.Hover([
      `**${word}**`,
      `Definition of ${word}`,
    ]);
  }
});
```

### Definition Provider

```typescript
vscode.languages.registerDefinitionProvider('javascript', {
  provideDefinition(document, position, token) {
    // Return location(s) where symbol at position is defined
    return new vscode.Location(document.uri, new vscode.Position(0, 0));
  }
});
```

### Semantic Tokens Provider

```typescript
const legend = new vscode.SemanticTokensLegend(
  ['keyword', 'function', 'variable', 'string', 'comment'],
  ['declaration', 'documentation', 'readonly']
);

vscode.languages.registerDocumentSemanticTokensProvider(
  { language: 'mylang' },
  {
    provideDocumentSemanticTokens(document) {
      const builder = new vscode.SemanticTokensBuilder(legend);
      const text = document.getText();
      // Parse and push tokens: builder.push(line, charOffset, length, tokenType, tokenModifiers)
      return builder.build();
    }
  },
  legend
);
```

### Signature Help Provider

```typescript
vscode.languages.registerSignatureHelpProvider('javascript', {
  provideSignatureHelp(document, position, token, context) {
    const help = new vscode.SignatureHelp();
    help.signatures = [
      new vscode.SignatureInformation(
        'func(param1: string, param2: number)',
        'Function description'
      )
    ];
    help.signatures[0].parameters = [
      new vscode.ParameterInformation('param1', 'First parameter'),
      new vscode.ParameterInformation('param2', 'Second parameter'),
    ];
    help.activeSignature = 0;
    help.activeParameter = 0;
    return help;
  }
}, ['(', ',']);
```

### Document Symbol Provider (Outline)

```typescript
vscode.languages.registerDocumentSymbolProvider('javascript', {
  provideDocumentSymbols(document, token) {
    return [
      new vscode.DocumentSymbol(
        'MyClass',
        'Class description',
        vscode.SymbolKind.Class,
        new vscode.Range(0, 0, 10, 0),
        new vscode.Range(0, 0, 0, 7)
      ),
    ];
  }
});
```

### Call Hierarchy Provider

```typescript
vscode.languages.registerCallHierarchyProvider('javascript', {
  prepareCallHierarchy(document, position) {
    const range = document.getWordRangeAtPosition(position);
    const name = document.getText(range);
    return new vscode.CallHierarchyItem(
      vscode.SymbolKind.Function,
      name, '', document.uri, range, range
    );
  },
  provideIncomingCalls(item, token) { return []; },
  provideOutgoingCalls(item, token) { return []; },
});
```

---

## 7. Tree Views

### Basic Tree Data Provider

```typescript
class MyTreeProvider implements vscode.TreeDataProvider<TreeItem> {
  private _onDidChangeTreeData = new vscode.EventEmitter<TreeItem | undefined>();
  onDidChangeTreeData = this._onDidChangeTreeData.event;

  refresh() { this._onDidChangeTreeData.fire(undefined); }

  getTreeItem(element: TreeItem): vscode.TreeItem {
    return element;
  }

  getChildren(element?: TreeItem): TreeItem[] {
    if (!element) {
      return [
        new TreeItem('Item 1', vscode.TreeItemCollapsibleState.Collapsed),
        new TreeItem('Item 2', vscode.TreeItemCollapsibleState.None),
      ];
    }
    if (element.label === 'Item 1') {
      return [
        new TreeItem('Child 1', vscode.TreeItemCollapsibleState.None),
        new TreeItem('Child 2', vscode.TreeItemCollapsibleState.None),
      ];
    }
    return [];
  }
}

const provider = new MyTreeProvider();
vscode.window.registerTreeDataProvider('myViewId', provider);
```

### Tree View with Commands & Icons

```typescript
class MyItem extends vscode.TreeItem {
  constructor(label: string, collapsible: vscode.TreeItemCollapsibleState) {
    super(label, collapsible);
    this.iconPath = new vscode.ThemeIcon('folder');
    this.command = {
      title: 'Open',
      command: 'myExt.openItem',
      arguments: [this],
    };
    this.contextValue = 'myItem'; // for context menu when clause
  }
}
```

### package.json for Tree Views

```json
{
  "contributes": {
    "viewsContainers": {
      "activitybar": [
        { "id": "myExplorer", "title": "My Explorer", "icon": "media/icon.svg" }
      ]
    },
    "views": {
      "myExplorer": [{ "id": "myViewId", "name": "My View", "icon": "media/view.svg" }],
      "explorer": [{ "id": "myOutline", "name": "My Outline" }]
    },
    "commands": [
      { "command": "myExt.refresh", "title": "Refresh", "icon": "$(refresh)" }
    ],
    "menus": {
      "view/title": [
        { "command": "myExt.refresh", "when": "view == myViewId", "group": "navigation" }
      ],
      "view/item/context": [
        { "command": "myExt.deleteItem", "when": "view == myViewId && viewItem == myItem" }
      ]
    }
  }
}
```

### Tree View with Drag and Drop

```typescript
class DragAndDropProvider implements vscode.TreeDataProvider<MyItem> {
  // ... standard TreeDataProvider methods ...

  dragMimeTypes = ['application/vnd.code.tree.myViewId'];
  dropMimeTypes = ['application/vnd.code.tree.myViewId'];

  handleDrag?(
    source: MyItem[],
    dataTransfer: vscode.DataTransfer,
    token: vscode.CancellationToken
  ): void {
    dataTransfer.set(
      'application/vnd.code.tree.myViewId',
      new vscode.DataTransferItem(source.map(s => s.id))
    );
  }

  handleDrop?(
    target: MyItem | undefined,
    dataTransfer: vscode.DataTransfer,
    token: vscode.CancellationToken
  ): void {
    const data = dataTransfer.get('application/vnd.code.tree.myViewId');
    if (data) {
      // Move items to target
      this._onDidChangeTreeData.fire(undefined);
    }
  }
}

vscode.window.createTreeView('myViewId', {
  treeDataProvider: new DragAndDropProvider(),
  dragAndDropController: { /* ... */ },
});
```

---

## 8. Webview

### Webview Panel (Singleton Pattern)

```typescript
class MyWebviewPanel {
  public static currentPanel: MyWebviewPanel | undefined;
  private readonly _panel: vscode.WebviewPanel;
  private _disposables: vscode.Disposable[] = [];

  public static createOrShow(context: vscode.ExtensionContext) {
    if (MyWebviewPanel.currentPanel) {
      MyWebviewPanel.currentPanel._panel.reveal(vscode.ViewColumn.One);
      return;
    }
    MyWebviewPanel.currentPanel = new MyWebviewPanel(context);
  }

  private constructor(context: vscode.ExtensionContext) {
    this._panel = vscode.window.createWebviewPanel(
      'myWebview',
      'My Panel',
      vscode.ViewColumn.One,
      {
        enableScripts: true,
        localResourceRoots: [
          vscode.Uri.joinPath(context.extensionUri, 'media'),
        ],
      }
    );

    this._panel.webview.html = this._getHtmlForWebview(this._panel.webview, context);

    this._panel.onDidDispose(() => this.dispose(), null, this._disposables);

    this._panel.webview.onDidReceiveMessage(
      (message) => {
        switch (message.command) {
          case 'alert':
            vscode.window.showInformationMessage(message.text);
            return;
        }
      },
      null,
      this._disposables
    );
  }

  public dispose() {
    MyWebviewPanel.currentPanel = undefined;
    this._panel.dispose();
    while (this._disposables.length) {
      this._disposables.pop()?.dispose();
    }
  }

  private _getHtmlForWebview(webview: vscode.Webview, context: vscode.ExtensionContext): string {
    const nonce = getNonce();
    const scriptUri = webview.asWebviewUri(
      vscode.Uri.joinPath(context.extensionUri, 'media', 'main.js')
    );
    const styleUri = webview.asWebviewUri(
      vscode.Uri.joinPath(context.extensionUri, 'media', 'style.css')
    );

    return `<!DOCTYPE html>
<html>
<head>
  <meta http-equiv="Content-Security-Policy"
        content="default-src 'none';
                style-src ${webview.cspSource} 'nonce-${nonce}';
                script-src 'nonce-${nonce}';">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <link href="${styleUri}" rel="stylesheet">
  <title>My Panel</title>
</head>
<body>
  <h1>Hello Webview</h1>
  <button id="btn">Click Me</button>
  <script nonce="${nonce}" src="${scriptUri}"></script>
</body>
</html>`;
  }
}

function getNonce(): string {
  let text = '';
  const possible = 'ABCDEFGHIJKLMNOPQRSTUVWXYZabcdefghijklmnopqrstuvwxyz0123456789';
  for (let i = 0; i < 32; i++) {
    text += possible.charAt(Math.floor(Math.random() * possible.length));
  }
  return text;
}
```

### Webview Sidebar View

```typescript
class MyViewProvider implements vscode.WebviewViewProvider {
  public static readonly viewType = 'myExt.myView';
  private _view?: vscode.WebviewView;

  constructor(private readonly _extensionUri: vscode.Uri) {}

  resolveWebviewView(
    webviewView: vscode.WebviewView,
    _context: vscode.WebviewViewResolveContext,
    _token: vscode.CancellationToken
  ) {
    this._view = webviewView;
    webviewView.webview.options = {
      enableScripts: true,
      localResourceRoots: [this._extensionUri],
    };
    webviewView.webview.html = this._getHtml();

    webviewView.webview.onDidReceiveMessage((message) => {
      switch (message.command) {
        case 'add':
          // Handle add
          break;
      }
    });
  }

  public addColor() {
    if (this._view) {
      this._view.webview.postMessage({ type: 'addColor' });
    }
  }
}
```

### package.json for Webview View

```json
{
  "contributes": {
    "views": {
      "explorer": [{ "type": "webview", "id": "myExt.myView", "name": "My View" }]
    },
    "commands": [
      { "command": "myExt.addColor", "title": "Add Color" }
    ],
    "menus": {
      "view/title": [
        { "command": "myExt.addColor", "when": "view == myExt.myView", "group": "navigation" }
      ]
    }
  }
}
```

### Webview Security Best Practices

1. **Always use CSP**: `default-src 'none'` + specific allowed sources
2. **Always use nonce**: Random nonce per webview creation for inline scripts
3. **Use `asWebviewUri()`**: Never use direct file paths for local resources
4. **Validate messages**: Always validate data from `onDidReceiveMessage`
5. **Never trust webview input**: Sanitize all data before using in extension

---

## 9. Language Server Protocol (LSP)

### Client Setup

```typescript
import { LanguageClient, LanguageClientOptions, ServerOptions, TransportKind } from 'vscode-languageclient/node';

let client: LanguageClient;

export function activate(context: vscode.ExtensionContext) {
  const serverModule = vscode.Uri.joinPath(context.extensionUri, 'server', 'out', 'server.js');

  const serverOptions: ServerOptions = {
    run: { module: serverModule.fsPath, transport: TransportKind.ipc },
    debug: {
      module: serverModule.fsPath,
      transport: TransportKind.ipc,
      options: { execArgv: ['--nolazy', '--inspect=6009'] }
    }
  };

  const clientOptions: LanguageClientOptions = {
    documentSelector: [{ scheme: 'file', language: 'plaintext' }],
    synchronize: {
      fileEvents: vscode.workspace.createFileSystemWatcher('**/.clientrc')
    }
  };

  client = new LanguageClient(
    'myLSP',
    'My Language Server',
    serverOptions,
    clientOptions
  );

  client.start();
}

export function deactivate(): Thenable<void> | undefined {
  return client?.stop();
}
```

### Multi-root LSP (Per-folder servers)

```typescript
const clients: Map<string, LanguageClient> = new Map();

export async function activate(context: vscode.ExtensionContext) {
  async function didOpenTextDocument(document: vscode.TextDocument) {
    if (document.languageId !== 'plaintext') { return; }
    const folder = vscode.workspace.getWorkspaceFolder(document.uri);
    if (!folder) { return; }
    if (clients.has(folder.uri.toString())) { return; }

    const client = new LanguageClient(/* ... */);
    client.start();
    clients.set(folder.uri.toString(), client);
  }

  vscode.workspace.onDidOpenTextDocument(didOpenTextDocument);
  vscode.workspace.onDidChangeWorkspaceFolders((event) => {
    for (const removed of event.removed) {
      const client = clients.get(removed.uri.toString());
      if (client) { client.stop(); clients.delete(removed.uri.toString()); }
    }
  });
}
```

---

## 10. MCP & Language Model Tools

This is the key integration pattern from the `vscode-extension-and-mcp-together` sample.

### Architecture: VS Code native LM Tools vs External MCP

| Approach | Transport | When to Use |
|---|---|---|
| **Native LM Tools** (`vscode.lm`) | In-process, no external server | Simple tools, extension-managed tools |
| **MCP Server Definition Provider** | stdio/SSE external process | External MCP servers, remote services |
| **Chat Participant** | In-process, conversational AI | Interactive AI conversations |

### Native LM Tool: package.json Declaration

```json
{
  "contributes": {
    "languageModelTools": [
      {
        "name": "my_tool",
        "displayName": "My Tool",
        "toolReferenceName": "myTool",
        "userDescription": "Does X for the user",
        "modelDescription": "Use this tool to do X when the user asks about Y. It takes parameter Z which means W.",
        "icon": "$(rocket)",
        "tags": ["utility", "automation"],
        "canBeReferencedInPrompt": true,
        "inputSchema": {
          "type": "object",
          "properties": {
            "filePath": {
              "type": "string",
              "description": "Path to the file to process"
            },
            "recursive": {
              "type": "boolean",
              "description": "Process recursively",
              "default": false
            }
          },
          "required": ["filePath"]
        }
      }
    ]
  }
}
```

### Native LM Tool: Abstract Base Class (Template Method Pattern)

```typescript
import * as vscode from 'vscode';

export abstract class Tool implements vscode.LanguageModelTool<object> {
  abstract toolName: string;

  async invoke(
    options: vscode.LanguageModelToolInvocationOptions<object>,
    token: vscode.CancellationToken
  ): Promise<vscode.LanguageModelToolResult> {
    try {
      const response = await this.call(options, token);
      return new vscode.LanguageModelToolResult([
        new vscode.LanguageModelTextPart(response),
      ]);
    } catch (error) {
      const errorPayload = {
        isError: true,
        message: error instanceof Error ? error.message : String(error),
      };
      return new vscode.LanguageModelToolResult([
        new vscode.LanguageModelTextPart(JSON.stringify(errorPayload)),
      ]);
    }
  }

  abstract call(
    options: vscode.LanguageModelToolInvocationOptions<object>,
    token: vscode.CancellationToken
  ): Promise<string>;
}
```

### Native LM Tool: Concrete Implementation

```typescript
import * as vscode from 'vscode';
import { Tool } from './tool';

export class FileSearchTool extends Tool {
  public readonly toolName = 'file_search_tool';

  async call(
    options: vscode.LanguageModelToolInvocationOptions<{
      pattern: string;
      maxResults?: number;
    }>,
    _token: vscode.CancellationToken
  ): Promise<string> {
    const { pattern, maxResults = 10 } = options.input as any;
    const files = await vscode.workspace.findFiles(pattern, undefined, maxResults);
    vscode.window.showInformationMessage(`Found ${files.length} files`);
    return JSON.stringify({
      success: true,
      files: files.map(f => f.fsPath),
    });
  }
}
```

### Native LM Tool: Registration in activate()

```typescript
export function activate(context: vscode.ExtensionContext) {
  const fileSearchTool = new FileSearchTool();
  context.subscriptions.push(
    vscode.lm.registerTool(fileSearchTool.toolName, fileSearchTool)
  );
}
```

### Chat Participant

```typescript
const PARTICIPANT_ID = 'myExt.myAssistant';

vscode.chat.createChatParticipant(PARTICIPANT_ID, {
  async handler(request, context, stream, token) {
    const messages = [
      vscode.LanguageModelChatMessage.Assistant('You are a helpful assistant.'),
    ];

    for (const turn of context.history) {
      if (turn instanceof vscode.ChatResponseTurn) {
        let text = '';
        for (const part of turn.parts) {
          if (part instanceof vscode.ChatResponseMarkdownPart) {
            text += part.value.value;
          }
        }
        messages.push(vscode.LanguageModelChatMessage.Assistant(text));
      } else {
        messages.push(vscode.LanguageModelChatMessage.User(request.prompt));
      }
    }

    messages.push(vscode.LanguageModelChatMessage.User(request.prompt));

    const response = await request.model.sendRequest(messages, {}, token);
    for await (const chunk of response.text) {
      stream.markdown(chunk);
    }
  },
});
```

### package.json for Chat Participant

```json
{
  "contributes": {
    "chatParticipants": [
      {
        "id": "myExt.myAssistant",
        "name": "myAssistant",
        "commands": [
          { "name": "explain", "description": "Explain code" },
          { "name": "fix", "description": "Fix issues" }
        ]
      }
    ]
  }
}
```

### MCP Server Definition Provider (External MCP Servers)

```typescript
vscode.lm.registerMcpServerDefinitionProvider('myMcpProvider', {
  onDidChangeMcpServerDefinitions: new vscode.EventEmitter<void>().event,

  async provideMcpServerDefinitions(token) {
    const servers: vscode.McpServerDefinition[] = [];

    const config = vscode.workspace.getConfiguration('myExt');
    const serverUrl = config.get<string>('mcpServerUrl');
    if (serverUrl) {
      servers.push(
        new vscode.McpStdioServerDefinition(
          'my-server',
          'My MCP Server',
          new vscode.RelativePattern(
            vscode.workspace.workspaceFolders?.[0]!,
            'node_modules/.bin/my-server'
          ),
          ['start']
        )
      );
    }
    return servers;
  }
});
```

### Using LM API Directly (not as chat participant)

```typescript
const models = await vscode.lm.selectChatModels({
  vendor: 'copilot',
  family: 'gpt-4o',
});

if (models.length > 0) {
  const response = await models[0].sendRequest(
    [vscode.LanguageModelChatMessage.User('Explain this code')],
    {},
    token
  );
  for await (const chunk of response.text) {
    // process text chunk
  }
}
```

---

## 11. Custom Editors & Notebooks

### Custom Text Editor

```typescript
vscode.window.registerCustomEditorProvider(
  'myExt.customEditor',
  {
    resolveCustomTextEditor(document, webviewPanel, token) {
      webviewPanel.webview.options = { enableScripts: true };
      webviewPanel.webview.html = getHtml(document, webviewPanel.webview);

      const changeSubscription = vscode.workspace.onDidChangeTextDocument(
        (e) => {
          if (e.document.uri.toString() === document.uri.toString()) {
            webviewPanel.webview.postMessage({ type: 'update' });
          }
        }
      );

      webviewPanel.onDidDispose(() => changeSubscription.dispose());
    },
  },
  { supportsMultipleEditorsPerDocument: true }
);
```

### package.json for Custom Editor

```json
{
  "contributes": {
    "customEditors": [
      {
        "viewType": "myExt.customEditor",
        "displayName": "My Custom Editor",
        "selector": [
          { "filenamePattern": "*.myext" }
        ],
        "priority": "default"
      }
    ]
  }
}
```

### Notebook Serializer

```typescript
vscode.workspace.registerNotebookSerializer(
  'myNotebook',
  {
    deserializeNotebook(data: Uint8Array) {
      const contents = new TextDecoder().decode(data);
      const cellData: vscode.NotebookCellData[] = [];
      // Parse contents into cells
      return new vscode.NotebookData(cellData);
    },
    serializeNotebook(data: vscode.NotebookData) {
      // Serialize back to file format
      return new Uint8Array(Buffer.from(JSON.stringify(data)));
    }
  },
  { transientOutputs: true }
);
```

---

## 12. File System Providers

### Virtual File System

```typescript
class MemFS implements vscode.FileSystemProvider {
  private _emitter = new vscode.EventEmitter<vscode.FileChangeEvent[]>();
  onDidChangeFile = this._emitter.event;

  private root = new Entry('', vscode.FileType.Directory);

  stat(uri: vscode.Uri): vscode.FileStat { /* return stat */ }
  readDirectory(uri: vscode.Uri): [string, vscode.FileType][] { /* list dir */ }
  readFile(uri: vscode.Uri): Uint8Array { /* read file */ }
  writeFile(uri: vscode.Uri, content: Uint8Array, opts: vscode.FileWriteOptions) { /* write */ }
  createDirectory(uri: vscode.Uri) { /* mkdir */ }
  delete(uri: vscode.Uri, opts: vscode.FileDeleteOptions) { /* delete */ }
  rename(oldUri: vscode.Uri, newUri: vscode.Uri, opts: vscode.FileRenameOptions) { /* rename */ }
  watch(uri: vscode.Uri, opts: vscode.FileWatchOptions): vscode.Disposable {
    return new vscode.Disposable(() => {});
  }
}

context.subscriptions.push(
  vscode.workspace.registerFileSystemProvider('memfs', new MemFS(), { isCaseSensitive: true })
);
```

---

## 13. Authentication

### Register Auth Provider

```typescript
vscode.authentication.registerAuthenticationProvider(
  'myAuthProvider',
  'My Auth',
  {
    async createSession(scopes: string[]) {
      const token = await getAccessToken(scopes);
      const session: vscode.AuthenticationSession = {
        id: '1',
        accessToken: token,
        account: { id: 'user-id', label: 'User' },
        scopes,
      };
      return session;
    },
    async getSessions(scopes?: string[]) {
      // Return existing sessions
      return [];
    },
    async removeSession(sessionId: string) {
      // Remove session
    },
  },
  { supportsMultipleAccounts: false }
);
```

### Use Built-in GitHub Auth

```typescript
const session = await vscode.authentication.getSession('github', ['repo', 'user'], {
  createIfNone: true,
});
if (session) {
  const octokit = new Octokit({ auth: session.accessToken });
  const user = await octokit.users.getAuthenticated();
}
```

---

## 14. Configuration & Settings

### package.json Configuration

```json
{
  "contributes": {
    "configuration": [
      {
        "id": "myExt",
        "title": "My Extension",
        "order": 1,
        "properties": {
          "myExt.enableFeature": {
            "type": "boolean",
            "default": true,
            "description": "Enable the feature",
            "scope": "window",
            "order": 0
          },
          "myExt.maxResults": {
            "type": "number",
            "default": 100,
            "description": "Maximum number of results",
            "scope": "resource",
            "minimum": 1,
            "maximum": 1000
          },
          "myExt.serverUrl": {
            "type": "string",
            "default": "",
            "description": "Server URL",
            "scope": "machine",
            "format": "uri"
          },
          "myExt.filePatterns": {
            "type": "array",
            "default": ["**/*.ts"],
            "description": "File patterns to search",
            "scope": "window",
            "items": { "type": "string" }
          }
        }
      }
    ]
  }
}
```

### Configuration Scopes

| Scope | Meaning | Shared across machines? |
|---|---|---|
| `window` | Global, applies to all workspaces in a window | Yes |
| `resource` | Per-folder/workspace (can differ per workspace) | No |
| `machine` | Machine-specific (paths, etc.) | No |
| `machine-overridable` | Machine-specific but user can override | No |
| `language-overridable` | Can be overridden per language | Depends |

### Reading & Writing Configuration

```typescript
// Read
const config = vscode.workspace.getConfiguration('myExt');
const enabled = config.get<boolean>('enableFeature', true);

// Read resource-scoped config for a specific file
const resourceConfig = vscode.workspace.getConfiguration('myExt', document.uri);
const maxResults = resourceConfig.get<number>('maxResults', 100);

// Write
await config.update('enableFeature', false, vscode.ConfigurationTarget.Global);
await config.update('maxResults', 50, vscode.ConfigurationTarget.Workspace);
await resourceConfig.update('maxResults', 25, vscode.ConfigurationTarget.WorkspaceFolder);

// Listen for changes
context.subscriptions.push(
  vscode.workspace.onDidChangeConfiguration(e => {
    if (e.affectsConfiguration('myExt')) {
      // Reload settings
    }
  })
);
```

---

## 15. Snippets & Language Config

### Snippet Contribution (package.json)

```json
{
  "contributes": {
    "snippets": [
      {
        "language": "javascript",
        "path": "./snippets/javascript.json"
      }
    ]
  }
}
```

### Snippet File Format

```json
{
  "Console Log": {
    "prefix": "log",
    "body": [
      "console.log('$1', $2);",
      "$0"
    ],
    "description": "Log output to console"
  },
  "If Statement": {
    "prefix": "if",
    "body": [
      "if (${1:condition}) {",
      "\t$0",
      "}"
    ],
    "description": "If statement"
  }
}
```

### Language Configuration

```json
{
  "contributes": {
    "languages": [
      {
        "id": "mylang",
        "extensions": [".mylang"],
        "aliases": ["My Language", "mylang"],
        "configuration": "./language-configuration.json"
      }
    ]
  }
}
```

### language-configuration.json

```json
{
  "comments": {
    "lineComment": "//",
    "blockComment": ["/*", "*/"]
  },
  "brackets": [
    ["{", "}"],
    ["[", "]"],
    ["(", ")"]
  ],
  "autoClosingPairs": [
    { "open": "{", "close": "}" },
    { "open": "[", "close": "]" },
    { "open": "(", "close": ")" },
    { "open": "'", "close": "'", "notIn": ["string", "comment"] },
    { "open": "\"", "close": "\"", "notIn": ["string"] },
    { "open": "`", "close": "`", "notIn": ["string", "comment"] }
  ],
  "autoCloseBefore": ";:.,=}])> \n\t",
  "surroundingPairs": [
    ["{", "}"],
    ["[", "]"],
    ["(", ")"],
    ["'", "'"],
    ["\"", "\""],
    ["`", "`"]
  ],
  "folding": {
    "markers": {
      "start": "^\\s*//\\s*#region\\b",
      "end": "^\\s*//\\s*#endregion\\b"
    }
  },
  "wordPattern": "(-?\\d*\\.\\d\\w*)|([^\\`~!@#%^&*()\\-=+[{]}\\\\|;:'\",.<>/?\\s]+)",
  "indentationRules": {
    "increaseIndentPattern": "\\{\\s*$",
    "decreaseIndentPattern": "^\\s*\\}"
  }
}
```

---

## 16. Themes

### Color Theme

```json
{
  "contributes": {
    "themes": [
      {
        "label": "My Light Theme",
        "uiTheme": "vs",
        "path": "./themes/light.tmTheme"
      },
      {
        "label": "My Dark Theme",
        "uiTheme": "vs-dark",
        "path": "./themes/dark.tmTheme"
      }
    ]
  }
}
```

`uiTheme` values: `"vs"` (light), `"vs-dark"` (dark), `"hc-black"` (high contrast)

### File Icon Theme

```json
{
  "contributes": {
    "iconThemes": [
      { "id": "myIcons", "label": "My Icons", "path": "./icons/icon-theme.json" }
    ]
  }
}
```

### Product Icon Theme

```json
{
  "contributes": {
    "productIconThemes": [
      { "id": "my-product-icons", "label": "My Product Icons", "path": "./theme/product-icon-theme.json" }
    ]
  }
}
```

### Themable Colors from Extension

```json
{
  "contributes": {
    "colors": [
      {
        "id": "myExt.highlightBackground",
        "description": "Background color for highlights",
        "defaults": {
          "light": "#FF000055",
          "dark": "#FF000055",
          "highContrast": "#FF0000"
        }
      }
    ]
  }
}
```

Use in code:
```typescript
const color = new vscode.ThemeColor('myExt.highlightBackground');
const decorationType = vscode.window.createTextEditorDecorationType({
  backgroundColor: color,
});
```

---

## 17. Telemetry & Localization

### Telemetry

```typescript
const logger = vscode.env.createTelemetryLogger({
  sendEventData(eventName, properties) {
    // Send to your telemetry backend
  },
  sendErrorData(eventName, data, properties) {
    // Send error data
  },
});

// Log usage event
logger.logUsage('commandInvoked', { command: 'myExt.hello' });

// Log error
logger.logError('processingFailed', { error: 'timeout' }, { stack: '...' });
```

### Localization (l10n)

```json
// package.json
{
  "l10n": "./l10n"
}
```

```typescript
// In code
const msg = vscode.l10n.t('Hello World');
const msg2 = vscode.l10n.t('Hello {name}', { name: 'User' });
```

```json
// l10n/bundle.l10n.ja.json (Japanese bundle)
{
  "Hello World": "こんにちは世界",
  "Hello {name}": "こんにちは{name}"
}
```

In package.json, use `%key%` syntax for localizable strings:
```json
{
  "commands": [{
    "command": "myExt.hello",
    "title": "%myExt.hello.title%"
  }]
}
```

---

## 18. Testing

### Method 1: @vscode/test-electron (Classic)

```
src/test/
  runTest.ts          # Downloads VS Code, runs tests
  suite/
    index.ts          # Mocha TDD setup, glob for test files
    extension.test.ts  # Actual tests
```

**runTest.ts:**
```typescript
import * as path from 'path';
import { runTests } from '@vscode/test-electron';

async function main() {
  try {
    const extensionDevelopmentPath = path.resolve(__dirname, '../../');
    const extensionTestsPath = path.resolve(__dirname, './suite/index');
    await runTests({ extensionDevelopmentPath, extensionTestsPath });
  } catch (err) {
    console.error('Failed to run tests');
    process.exit(1);
  }
}

main();
```

**suite/index.ts:**
```typescript
import * as path from 'path';
import * as Mocha from 'mocha';
import * as glob from 'glob';

export async function run(): Promise<void> {
  const mocha = new Mocha({ ui: 'tdd' });
  mocha.useColors(true);
  const testsRoot = path.resolve(__dirname, '..');
  return new Promise((resolve, reject) => {
    glob('**/**.test.js', { cwd: testsRoot }, (err, files) => {
      if (err) { return reject(err); }
      files.forEach(f => mocha.addFile(path.resolve(testsRoot, f)));
      try {
        mocha.run((failures) => {
          if (failures > 0) { reject(new Error(`${failures} tests failed`)); }
          else { resolve(); }
        });
      } catch (err) { reject(err); }
    });
  });
}
```

**extension.test.ts:**
```typescript
import * as assert from 'assert';
import * as vscode from 'vscode';

suite('Extension Test Suite', () => {
  test('Extension should be present', () => {
    assert.ok(vscode.extensions.getExtension('publisher.myExtension'));
  });

  test('Command should be registered', async () => {
    const commands = await vscode.commands.getCommands(true);
    assert.ok(commands.includes('myExt.hello'));
  });
});
```

**package.json scripts:**
```json
{
  "scripts": {
    "pretest": "npm run compile",
    "test": "node ./out/test/runTest.js"
  },
  "devDependencies": {
    "@vscode/test-electron": "^2.3.9",
    "mocha": "^10.2.0",
    "@types/mocha": "^10.0.0",
    "glob": "^7.1.4",
    "source-map-support": "^0.5.21"
  }
}
```

### Method 2: @vscode/test-cli (Newer)

```json
{
  "scripts": {
    "test": "vscode-test"
  },
  "devDependencies": {
    "@vscode/test-cli": "^0.0.8",
    "@vscode/test-electron": "^2.3.9",
    "mocha": "^10.2.0",
    "@types/mocha": "^10.0.0"
  }
}
```

Create `.vscode-test.mjs`:
```javascript
import { defineConfig } from '@vscode/test-cli';

export default defineConfig({
  files: 'out/test/**/*.test.js',
  mocha: {
    ui: 'tdd',
    timeout: 20000,
  },
});
```

---

## 19. Build & Bundling

### tsc Only (Simplest)

```json
{
  "scripts": {
    "vscode:prepublish": "npm run compile",
    "compile": "tsc -p ./",
    "watch": "tsc -watch -p ./"
  },
  "main": "./out/extension.js"
}
```

### esbuild (Recommended for performance)

**esbuild.js:**
```javascript
const esbuild = require('esbuild');

const production = process.argv.includes('--production');
const watch = process.argv.includes('--watch');

async function main() {
  const ctx = await esbuild.context({
    entryPoints: ['src/extension.ts'],
    bundle: true,
    format: 'cjs',
    minify: production,
    sourcemap: !production,
    plugins: [/* custom plugins */],
    logLevel: production ? undefined : 'info',
    platform: 'node',
    target: 'ES2024',
    external: ['vscode'],
    outfile: 'dist/extension.js',
  });

  if (watch) {
    await ctx.watch();
  } else {
    await ctx.rebuild();
    await ctx.dispose();
  }
}

main().catch(console.error);
```

**package.json:**
```json
{
  "scripts": {
    "vscode:prepublish": "npm run package",
    "package": "check-types && lint && node esbuild.js --production",
    "watch": "npm-run-all -p watch:*",
    "watch:esbuild": "node esbuild.js --watch",
    "watch:tsc": "tsc -p tsconfig.json --noEmit --watch --preserveWatchOutput",
    "check-types": "tsc -p tsconfig.json --noEmit",
    "lint": "eslint"
  },
  "main": "./dist/extension.js"
}
```

**Key esbuild options:**
- `bundle: true`: Bundle all dependencies into single file
- `format: 'cjs'`: CommonJS required by VS Code
- `platform: 'node'`: Node.js runtime
- `external: ['vscode']`: Never bundle the vscode module
- `minify: production`: Minify only for production builds

### webpack

**webpack.config.js:**
```javascript
module.exports = {
  target: 'node',
  mode: 'none',
  entry: './src/extension.ts',
  output: {
    filename: 'extension.js',
    path: path.join(__dirname, 'dist'),
    libraryTarget: 'commonjs2',
  },
  devtool: 'source-map',
  externals: { vscode: 'commonjs vscode' },
  resolve: { mainFields: ['browser', 'module', 'main'], extensions: ['.ts', '.js'] },
  module: {
    rules: [{ test: /\.ts$/, exclude: /node_modules/, use: [{ loader: 'ts-loader', options: { compilerOptions: { module: 'es6' } } }] }],
  },
};
```

**package.json:**
```json
{
  "scripts": {
    "vscode:prepublish": "webpack --mode production",
    "webpack-dev": "webpack --mode development --watch",
    "compile": "webpack --mode development"
  },
  "main": "./dist/extension"
}
```

### Build Comparison

| Aspect | tsc | esbuild | webpack |
|---|---|---|---|
| Speed | Slow | Very fast | Moderate |
| Bundling | No (one file per .ts) | Yes (single bundle) | Yes (single bundle) |
| Size | Large (many files) | Small (minified) | Small (minified) |
| Setup | Zero config | Minimal config | Complex config |
| Source maps | Yes | Yes | Yes |
| Recommended for | Simple extensions | Most extensions | Legacy extensions |

---

## 20. Debugging

### .vscode/launch.json

```json
{
  "version": "0.2.0",
  "configurations": [
    {
      "name": "Run Extension",
      "type": "extensionHost",
      "request": "launch",
      "runtimeExecutable": "${execPath}",
      "args": ["--extensionDevelopmentPath=${workspaceFolder}"],
      "outFiles": ["${workspaceFolder}/out/**/*.js"],
      "preLaunchTask": "npm: watch"
    },
    {
      "name": "Extension Tests",
      "type": "extensionHost",
      "request": "launch",
      "runtimeExecutable": "${execPath}",
      "args": [
        "--extensionDevelopmentPath=${workspaceFolder}",
        "--extensionTestsPath=${workspaceFolder}/out/test/suite/index"
      ],
      "outFiles": ["${workspaceFolder}/out/test/**/*.js"],
      "preLaunchTask": "npm: compile"
    }
  ]
}
```

### .vscode/tasks.json

```json
{
  "version": "2.0.0",
  "tasks": [
    {
      "type": "npm",
      "script": "watch",
      "problemMatcher": "$tsc-watch",
      "isBackground": true,
      "presentation": { "reveal": "never" },
      "group": { "kind": "build", "isDefault": true }
    }
  ]
}
```

---

## 21. Publishing

### Install vsce

```bash
npm install -g @vscode/vsce
```

### Create Publisher

```bash
vsce create-publisher your-publisher-name
vsce login your-publisher-name
```

### Package as VSIX

```bash
vsce package
# Creates: my-extension-0.0.1.vsix
```

### Publish to Marketplace

```bash
vsce publish
vsce publish minor  # Bump minor version and publish
vsce publish patch  # Bump patch version and publish
```

### Install VSIX Locally

```bash
code --install-extension my-extension-0.0.1.vsix
```

### package.json Requirements for Publishing

```json
{
  "publisher": "your-publisher-name",
  "repository": { "type": "git", "url": "https://github.com/you/repo" },
  "license": "MIT",
  "icon": "images/icon.png",
  "galleryBanner": { "color": "#1e1e1e", "theme": "dark" },
  "keywords": ["vscode", "extension", "utility"],
  "categories": ["Programming Languages", "Snippets"]
}
```

---

## 22. Best Practices

### Activation & Performance

1. **Use lazy activation**: Prefer `onLanguage:`, `onCommand:`, `onView:` over `*`
2. **Empty `activationEvents` array**: Modern VS Code auto-detects from contributes
3. **Minimize `activate()` work**: Only register providers/commands, defer heavy init
4. **Use `onStartupFinished`** for background tasks that need to run but don't block startup

### Disposables & Memory

1. **Always push to `context.subscriptions`**: Every register* call returns a Disposable
2. **Use DisposableStore for grouped cleanup**: Create multiple subscriptions stores per feature
3. **Singleton pattern for webviews**: Avoid creating duplicate panels
4. **Detach event listeners**: All event listeners pushed to subscriptions auto-dispose

### Commands

1. **Prefix command IDs**: Use extension name as prefix (`myExt.commandName`)
2. **Set `category` in commands**: Groups commands in the Command Palette
3. **Hide internal commands**: Use `"when": "false"` in `commandPalette` menu
4. **Use `when` clauses**: Show commands only in relevant contexts

### Webview

1. **Always use Content Security Policy (CSP)**
2. **Always use nonce for scripts**
3. **Use `asWebviewUri()`** for local resource references
4. **Validate all messages** from webview before acting on them
5. **Post messages** for communication: `panel.webview.postMessage()` / `panel.webview.onDidReceiveMessage()`
6. **Implement `WebviewPanelSerializer`** for panel state persistence across reloads

### MCP & Language Model Tools

1. **Dual declaration**: Declare in `package.json` (metadata) AND register in code (implementation)
2. **`name` must match**: The `name` in `languageModelTools` must match `toolName` in code
3. **Write good `modelDescription`**: This is the prompt that tells the AI when to use your tool
4. **Always handle errors gracefully**: Return `LanguageModelToolResult` with error payload, never throw
5. **Use `Tool` base class pattern**: Template Method simplifies tool development
6. **Use `canBeReferencedInPrompt: true`**: Lets users reference with `#toolName` in chat
7. **Define `inputSchema`**: JSON Schema for structured tool parameters

### Build

1. **Prefer esbuild** over webpack: faster builds, simpler config
2. **Always externalize `vscode`**: Never bundle the vscode module
3. **Use `check-types` + esbuild**: tsc for type checking, esbuild for bundling
4. **Minify for production**: Set `minify: true` in `vscode:prepublish` script

### Testing

1. **Use `@vscode/test-cli`** (newer, simpler) over raw `@vscode/test-electron`
2. **Test activation**: Verify extension activates and commands are registered
3. **Test in Extension Development Host**: Tests run in real VS Code instance
4. **Increase Mocha timeout**: VS Code startup is slow, set `timeout: 20000+`

### Package & Publish

1. **Never commit `node_modules/` or `out/`**: Add to `.gitignore`
2. **Add `.vscode-test/` to `.gitignore`**: Test artifacts
3. **Add `*.vsix` to `.gitignore`**: Built packages
4. **Use `.vscodeignore`**: Exclude test files, source maps, etc. from VSIX package
5. **Set `repository`**: Required for marketplace publishing

---

## 23. Code Templates

### Complete Minimal Extension (TypeScript + tsc)

**package.json:**
```json
{
  "name": "my-extension",
  "displayName": "My Extension",
  "description": "My VS Code extension",
  "version": "0.0.1",
  "publisher": "my-publisher",
  "engines": { "vscode": "^1.100.0" },
  "categories": ["Other"],
  "activationEvents": [],
  "main": "./out/extension.js",
  "contributes": {
    "commands": [
      { "command": "myExtension.helloWorld", "title": "Hello World" }
    ]
  },
  "scripts": {
    "vscode:prepublish": "npm run compile",
    "compile": "tsc -p ./",
    "watch": "tsc -watch -p ./",
    "lint": "eslint"
  },
  "devDependencies": {
    "@types/vscode": "^1.100.0",
    "@types/node": "^22",
    "typescript": "^5.8.2",
    "eslint": "^9.13.0",
    "@eslint/js": "^9.13.0",
    "typescript-eslint": "^8.26.0",
    "@stylistic/eslint-plugin": "^2.9.0"
  }
}
```

**tsconfig.json:**
```json
{
  "compilerOptions": {
    "module": "commonjs",
    "target": "ES2024",
    "lib": ["ES2024"],
    "outDir": "out",
    "sourceMap": true,
    "strict": true,
    "rootDir": "src",
    "esModuleInterop": true
  },
  "exclude": ["node_modules", ".vscode-test"]
}
```

**src/extension.ts:**
```typescript
import * as vscode from 'vscode';

export function activate(context: vscode.ExtensionContext) {
  context.subscriptions.push(
    vscode.commands.registerCommand('myExtension.helloWorld', () => {
      vscode.window.showInformationMessage('Hello World!');
    })
  );
}

export function deactivate() {}
```

**.gitignore:**
```
out
node_modules
.vscode-test/
*.vsix
```

### Complete Extension with MCP Tools

**src/tool.ts:**
```typescript
import * as vscode from 'vscode';

export abstract class Tool implements vscode.LanguageModelTool<object> {
  abstract toolName: string;

  async invoke(
    options: vscode.LanguageModelToolInvocationOptions<object>,
    token: vscode.CancellationToken
  ): Promise<vscode.LanguageModelToolResult> {
    try {
      const response = await this.call(options, token);
      return new vscode.LanguageModelToolResult([
        new vscode.LanguageModelTextPart(response),
      ]);
    } catch (error) {
      return new vscode.LanguageModelToolResult([
        new vscode.LanguageModelTextPart(JSON.stringify({
          isError: true,
          message: error instanceof Error ? error.message : String(error),
        })),
      ]);
    }
  }

  abstract call(
    options: vscode.LanguageModelToolInvocationOptions<object>,
    token: vscode.CancellationToken
  ): Promise<string>;
}
```

**src/myTool.ts:**
```typescript
import * as vscode from 'vscode';
import { Tool } from './tool';

export class MyTool extends Tool {
  public readonly toolName = 'my_tool';

  async call(
    options: vscode.LanguageModelToolInvocationOptions<{
      input: string;
    }>,
    _token: vscode.CancellationToken
  ): Promise<string> {
    const { input } = options.input as any;
    vscode.window.showInformationMessage(`Processing: ${input}`);
    return JSON.stringify({ success: true, result: `Processed: ${input}` });
  }
}
```

**src/extension.ts:**
```typescript
import * as vscode from 'vscode';
import { MyTool } from './myTool';

export function activate(context: vscode.ExtensionContext) {
  context.subscriptions.push(
    vscode.commands.registerCommand('myExtension.helloWorld', () => {
      vscode.window.showInformationMessage('Hello World!');
    })
  );

  const myTool = new MyTool();
  context.subscriptions.push(
    vscode.lm.registerTool(myTool.toolName, myTool)
  );
}
```

### Complete Extension with esbuild

**esbuild.js:**
```javascript
const esbuild = require('esbuild');

const production = process.argv.includes('--production');

async function main() {
  await esbuild.build({
    entryPoints: ['src/extension.ts'],
    bundle: true,
    format: 'cjs',
    minify: production,
    sourcemap: !production,
    platform: 'node',
    target: 'ES2024',
    external: ['vscode'],
    outfile: 'dist/extension.js',
  });
}

main().catch(console.error);
```

**package.json scripts:**
```json
{
  "scripts": {
    "vscode:prepublish": "npm run package",
    "package": "npm run check-types && npm run lint && node esbuild.js --production",
    "compile": "node esbuild.js",
    "watch": "node esbuild.js --watch",
    "check-types": "tsc --noEmit",
    "lint": "eslint"
  },
  "main": "./dist/extension.js",
  "devDependencies": {
    "@types/vscode": "^1.100.0",
    "@types/node": "^22",
    "typescript": "^5.8.2",
    "esbuild": "^0.25.0",
    "eslint": "^9.13.0",
    "@eslint/js": "^9.13.0",
    "typescript-eslint": "^8.26.0",
    "@stylistic/eslint-plugin": "^2.9.0"
  }
}
```

### Complete Extension with Tests

**src/test/runTest.ts:**
```typescript
import * as path from 'path';
import { runTests } from '@vscode/test-electron';

async function main() {
  try {
    const extensionDevelopmentPath = path.resolve(__dirname, '../../');
    const extensionTestsPath = path.resolve(__dirname, './suite/index');
    await runTests({ extensionDevelopmentPath, extensionTestsPath });
  } catch (err) {
    console.error('Failed to run tests');
    process.exit(1);
  }
}

main();
```

**src/test/suite/index.ts:**
```typescript
import * as path from 'path';
import * as Mocha from 'mocha';
import * as glob from 'glob';

export async function run(): Promise<void> {
  const mocha = new Mocha({ ui: 'tdd', timeout: 20000 });
  mocha.useColors(true);
  const testsRoot = path.resolve(__dirname, '..');
  return new Promise((resolve, reject) => {
    glob('**/**.test.js', { cwd: testsRoot }, (err, files) => {
      if (err) { return reject(err); }
      files.forEach(f => mocha.addFile(path.resolve(testsRoot, f)));
      try {
        mocha.run(failures => {
          if (failures > 0) { reject(new Error(`${failures} tests failed`)); }
          else { resolve(); }
        });
      } catch (err) { reject(err); }
    });
  });
}
```

**src/test/suite/extension.test.ts:**
```typescript
import * as assert from 'assert';
import * as vscode from 'vscode';

suite('Extension Tests', () => {
  test('Extension activates', async () => {
    const ext = vscode.extensions.getExtension('myPublisher.myExtension');
    assert.ok(ext);
    await ext!.activate();
    assert.strictEqual(ext!.isActive, true);
  });

  test('Command is registered', async () => {
    const commands = await vscode.commands.getCommands(true);
    assert.ok(commands.includes('myExtension.helloWorld'));
  });

  test('Command executes', async () => {
    await vscode.commands.executeCommand('myExtension.helloWorld');
  });
});
```

---

## 24. Common Patterns Reference

### Disposable Registration Pattern

```typescript
// Always push to context.subscriptions
context.subscriptions.push(
  vscode.commands.registerCommand('cmd', () => {}),
  vscode.languages.registerCompletionItemProvider(sel, provider),
  vscode.window.createStatusBarItem(vscode.StatusBarAlignment.Right),
  vscode.workspace.onDidChangeConfiguration(handler),
);
```

### Singleton Webview Panel Pattern

```typescript
class MyPanel {
  private static currentPanel: MyPanel | undefined;
  public static readonly viewType = 'myPanel';

  public static createOrShow(context: vscode.ExtensionContext) {
    if (MyPanel.currentPanel) {
      MyPanel.currentPanel._panel.reveal();
      return;
    }
    MyPanel.currentPanel = new MyPanel(context);
  }

  private constructor(context: vscode.ExtensionContext) { /* ... */ }
  public dispose() { MyPanel.currentPanel = undefined; /* cleanup */ }
}
```

### Event-Driven Update Pattern

```typescript
class MyProvider implements vscode.TreeDataProvider<MyItem> {
  private _onDidChange = new vscode.EventEmitter<MyItem | undefined>();
  readonly onDidChangeTreeData = this._onDidChange.event;

  refresh() { this._onDidChange.fire(undefined); }

  // ... getTreeItem, getChildren
}

// External trigger to refresh
vscode.commands.registerCommand('myExt.refresh', () => provider.refresh());
```

### Provider Registration Pattern

```typescript
export function register(context: vscode.ExtensionContext) {
  const provider = new MyProvider(context);
  context.subscriptions.push(
    vscode.window.registerWebviewViewProvider(MyProvider.viewType, provider)
  );
  return provider;
}
```

### Configuration Watcher Pattern

```typescript
let configuredValue: string;
function updateConfig() {
  configuredValue = vscode.workspace.getConfiguration('myExt').get('setting', 'default');
}
updateConfig();

context.subscriptions.push(
  vscode.workspace.onDidChangeConfiguration(e => {
    if (e.affectsConfiguration('myExt')) {
      updateConfig();
    }
  })
);
```

### Document Change Watcher Pattern

```typescript
function updateOnDocumentChange(document: vscode.TextDocument) {
  if (document.languageId !== 'typescript') { return; }
  // Update diagnostics, decorations, etc.
}

context.subscriptions.push(
  vscode.window.onDidChangeActiveTextEditor(e => {
    if (e) { updateOnDocumentChange(e.document); }
  }),
  vscode.workspace.onDidChangeTextDocument(e => {
    updateOnDocumentChange(e.document);
  }),
);
```

### Secret Storage Pattern

```typescript
export function activate(context: vscode.ExtensionContext) {
  // Secret storage is per-extension, persisted securely
  const { secrets } = context;

  await secrets.store('apiToken', 'my-secret-token');
  const token = await secrets.get('apiToken');
  await secrets.delete('apiToken');

  context.subscriptions.push(
    secrets.onDidChange(e => {
      if (e.key === 'apiToken') {
        // React to token changes
      }
    })
  );
}
```

### Global State Pattern

```typescript
export function activate(context: vscode.ExtensionContext) {
  const { globalState } = context;

  await globalState.update('lastOpenTime', Date.now());
  const lastOpen = globalState.get<number>('lastOpenTime', 0);
}
```

### Codicon Reference

Common codicons used in extensions:
- `$(file)`, `$(folder)`, `$(folder-opened)`
- `$(search)`, `$(refresh)`, `$(add)`, `$(remove)`
- `$(play)`, `$(stop)`, `$(debug-start)`
- `$(check)`, `$(error)`, `$(warning)`, `$(info)`
- `$(settings-gear)`, `$(terminal)`, `$(git-branch)`
- `$(rocket)`, `$(star)`, `$(heart)`, `$(megaphone)`

Full list: https://code.visualstudio.com/api/references/icons-in-labels

---

## Decision Helper: Which APIs Do I Need?

| I want to... | Use these APIs |
|---|---|
| Run code on command | `commands.registerCommand` + `contributes.commands` |
| Show UI messages/picks | `window.showInformationMessage`, `showQuickPick`, `showInputBox` |
| Add status bar item | `window.createStatusBarItem` |
| Add sidebar tree view | `TreeDataProvider` + `contributes.views` |
| Show custom HTML UI | `window.createWebviewPanel` or `WebviewViewProvider` |
| Provide completions | `languages.registerCompletionItemProvider` |
| Show inline code lenses | `languages.registerCodeLensProvider` |
| Add QuickFix/code actions | `languages.registerCodeActionProvider` |
| Show errors/warnings | `languages.createDiagnosticCollection` |
| Provide hover info | `languages.registerHoverProvider` |
| Go to definition | `languages.registerDefinitionProvider` |
| Syntax highlighting | `contributes.grammars` + TextMate grammar |
| Semantic highlighting | `languages.registerDocumentSemanticTokensProvider` |
| Code folding | `contributes.languages` + `language-configuration.json` |
| Snippets | `contributes.snippets` + snippet JSON |
| Settings | `contributes.configuration` + `workspace.getConfiguration` |
| Custom file editor | `window.registerCustomEditorProvider` + `contributes.customEditors` |
| Notebook support | `workspace.registerNotebookSerializer` + `contributes.notebooks` |
| Virtual filesystem | `workspace.registerFileSystemProvider` |
| Source control integration | `scm.createSourceControl` |
| Handle URIs from browser | `window.registerUriHandler` |
| Auth/OAuth | `authentication.registerAuthenticationProvider` |
| Make AI tools (MCP) | `vscode.lm.registerTool` + `contributes.languageModelTools` |
| AI chat participant | `chat.createChatParticipant` + `contributes.chatParticipants` |
| AI model provider | `lm.registerLanguageModelChatProvider` + `contributes.languageModelChatProviders` |
| Language server (LSP) | `vscode-languageclient` + separate server process |
| Color theme | `contributes.themes` + `.tmTheme` file |
| File icons | `contributes.iconThemes` |
| Telemetry | `env.createTelemetryLogger` |
| Localization | `l10n.t()` + `contributes.l10n` |
| Proposed API | `enabledApiProposals` array + `@vscode/dts` |