# Silicon for VS Code

Syntax highlighting, the **Sigil Dark** color theme, and a built-in
**language server** for the
[Silicon programming language](https://github.com/natescode/sigil) — a
bootstrappable language that targets WebAssembly Text Format (WAT)
and runs under WASI.

The language server (LSP) is bundled directly in the VSIX. No
separate clone, no extra `bun` install — `node` is enough.

## What's in here

- **Silicon language grammar** (`syntaxes/silicon.tmLanguage.json`) —
  TextMate scopes for `.si` files: definition keywords (`@let`, `@fn`,
  `@var`, `@type`, `@extern`, `@stratum_*`), control flow (`@if`,
  `@loop`, `@match`, `@return`, `@break`, `@continue`), function calls
  (`&name`, `&mod::fn`), primitive types (`Int`, `Float`, `Bool`,
  `String`, `Void`), string literals with `\n` / `\t` / `\\` escapes,
  numeric literals (decimal, hex, binary, octal, float), and `#` line
  comments / `##` doc comments.
- **Sigil Dark color theme** (`themes/sigil-dark-color-theme.json`) —
  the same palette the Sigil playground uses, ported to VS Code's
  full UI surface (editor, sidebar, tabs, status bar, terminal).
- **Language configuration** (`language-configuration.json`) —
  bracket pairs, auto-closing single quotes, line-comment toggle.
- **Language server** (`lsp/`) — diagnostics, document symbols
  (outline / breadcrumb), go-to-definition, and hover for any `.si`
  file in the workspace.  The extension spawns this server as a
  `node` child process on `.si` activation; the bundled
  `lsp/dist/index.js` is included in the VSIX.
- **VS Code client** (`client/`) — thin wrapper that wires the
  language server into the editor.

## Language server settings

| Setting | Default | What it does |
| --- | --- | --- |
| `silicon.lsp.enabled` | `true` | Turn the language server on / off. |

Command: **`Silicon: Restart LSP`** in the command palette
(`Ctrl/Cmd-Shift-P`) restarts the server without reloading the window.

## Install (Marketplace)

Coming soon — for now, install locally per the section below.

## Install (local development)

```bash
git clone https://github.com/natescode/silicon-vscode.git
cd silicon-vscode
bun install
bun run build              # builds client + server bundles
# Run the extension in a fresh VS Code window:
code --extensionDevelopmentPath="$(pwd)"
```

Or symlink into `~/.vscode/extensions/`:

```bash
ln -s "$(pwd)" ~/.vscode/extensions/silicon-vscode
```

Then in VS Code:

1. Open any `.si` file — syntax highlighting kicks in automatically.
2. **Cmd/Ctrl + K, Cmd/Ctrl + T** → pick **Sigil Dark**.

## Build + test

```bash
bun run build               # bundles client/dist/extension.js + lsp/dist/index.js
bun run test:server         # LSP smoke test (dev mode, runs under bun)
bun run scripts/smoke-bundled.ts   # LSP smoke test against the bundled artifact under node
```

## Repository layout

```
client/                 VS Code extension client
  src/extension.ts      Spawns lsp/dist/index.js on .si activation
  dist/extension.js     Bundled output (.vsix)
lsp/                    Language Server Protocol implementation
  src/
    index.ts            stdio entry + handler registry
    workspace.ts        per-doc parse/elaborate/typecheck cache
    handlers/           diagnostics, document-symbol, definition, hover
    smoke.test.ts       bun:test wrapper around _smoke-run.ts
  dist/                 Bundled output (.vsix)
    index.js
    std.wat             runtime asset (read by the bundled compiler)
    silicon-official.ohm
src/                    Vendored snapshot of the Sigil compiler frontend
                        (parser, AST, elaborator, typechecker, IR,
                        codegen, modules, platforms).  Frozen at
                        natescode/sigil commit b08f6c4c — the last
                        point before sigil went 100% self-hosted.
                        Imported by lsp/src/* via `../../src/...`
                        relative paths.
boot/strata/builtin/    Silicon strata source loaded at request time
                        by the bundled server.
syntaxes/, themes/      TextMate grammar + color theme
scripts/                Dev scripts (bundled-artifact smoke runner)
```

## Why the LSP is bundled here

The language server source used to live in its own repo
(`natescode/silicon-lsp`) on the theory that vim / emacs / helix
users might want to use it standalone.  In practice, every real
user was going through VS Code, and the separate repo just added
ceremony to releases without buying anything.  Bundled distribution
gives end users a single Marketplace install, no `bun` requirement,
no extra clones.

If you actually want to drive the LSP from another editor: the
bundled `lsp/dist/index.js` is a standalone `node`-runnable LSP
binary.  Point your editor's LSP client at
`node /path/to/lsp/dist/index.js --stdio`.

## Palette

The theme reuses the exact CSS variables from the playground's
`sigil-dark`:

| Token / surface | Color   |
| --- | --- |
| Background | `#0d1117` |
| Foreground text | `#e6edf3` |
| Comments | `#484f58` italic |
| Strings | `#3fb950` |
| Numbers / booleans | `#79c0ff` |
| Keywords (`@let`, `@fn`, …) | `#a371f7` |
| Function names (`&add`) | `#d2a8ff` |
| Primitive types | `#58a6ff` |
| Operators | `#ff7b72` |
| Accent (cursor, badges) | `#a371f7` |

## License

MIT, matching the upstream Sigil project.
