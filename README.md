<div align="center">
  <img src="assets/logo.svg" alt="Tech for Humans" width="280" />
</div>

# cursor-framework-plugins

A **Cursor** plugin marketplace focused on the **Tech for Humans** ecosystem and **`@t4h.framework/*`** packages.

## Repository layout

| Path                                  | Purpose                                                                                       |
| ------------------------------------- | --------------------------------------------------------------------------------------------- |
| **`.cursor-plugin/marketplace.json`** | Marketplace manifest: name, version, owner, and plugin list (`pluginRoot`: `plugins`).        |
| **`plugins/framework/`**              | **Framework** plugin: agent rules, library usage skills, post-edit hooks, and **MCP** config. |

## Clone the repository and test locally

### Clone

```bash
git clone https://github.com/tech4humans-brasil/cursor-framework-plugins.git
cd cursor-framework-plugins
```

### Test plugins locally

Load your plugin from `~/.cursor/plugins/local`:

1. Create a folder for your plugin, for example `~/.cursor/plugins/local/my-plugin`.
2. Copy your plugin files into that folder. Make sure `.cursor-plugin/plugin.json` sits at the **root** of the plugin (not at the root of the whole monorepo).
3. Restart Cursor or run **Developer: Reload Window**.
4. Confirm the plugin’s components load in Cursor (rules, skills, MCP servers, and so on).
5. Copy the plugin directory recursively (`cp` does not copy folders without **`-R`**):

```bash
mkdir -p ~/.cursor/plugins/local
cp -R /path/to/my-plugin ~/.cursor/plugins/local/my-plugin
```

After you change files in the repo, run `cp -R` again (or remove the destination folder first for a clean copy).

**In this repository**, the publishable **Framework** plugin lives under `plugins/framework/`. Example after cloning:

```bash
mkdir -p ~/.cursor/plugins
cp -R "$PWD/plugins/framework" ~/.cursor/plugins/framework
```

## **Framework** plugin

- **`plugins/framework/.cursor-plugin/plugin.json`** — plugin metadata (name, version, keywords).
- **`plugins/framework/mcp.json`** — **[Model Context Protocol](https://modelcontextprotocol.io/)** server definitions for Cursor (which MCP servers the plugin wires up). Currently declares **`miro-mcp`** (`url`: `https://mcp.miro.com/`, not disabled). Adjust **`autoApprove`** or add servers here as needed; see Cursor’s MCP documentation for how plugin-level MCP is loaded.
- **`plugins/framework/rules/`** — rules (`.mdc`): coding standards, review checklist, security.
- **`plugins/framework/skills/`** — one skill per `@t4h.framework/*` package, plus meta skills and the index:
  - **`framework-skills-index`** — package → skill map, recommended app layout (`main` + `workflows/` + `constants/`), and when to confirm details with the user.
  - **`framework-building-workflows`** — how to assemble **`App`** + **`Workflow`** files and which **`framework-*`** skill to open for each concern (HTTP, auth, ORM, Amber, etc.).
  - **`framework-amber`**, **`framework-cache`**, **`framework-core`** (including **`App`**, **`Workflow`**, **`History`**, **`startWorkflow`** / **`StartWorkflowActivity`** / **`WorkflowRef`**), **`framework-fs`**, **`framework-http`**, **`framework-jwks`**, **`framework-oidc`**, **`framework-orm`**, **`framework-sftp`**, **`framework-smtp`** — direct API usage for each package.
- **`plugins/framework/hooks/hooks.json`** — e.g. **`afterFileEdit`** to run formatting via **`scripts/format-code.sh`**.

## Usage

1. In Cursor, install or reference this repository using your team’s **marketplace / plugins** workflow (root manifest: **`.cursor-plugin/marketplace.json`**).
2. Enable the **Framework** plugin to inherit rules, skills, hooks, and **MCP** settings from **`plugins/framework/mcp.json`** in the workspace.
3. For **`@t4h.framework/*`** work, start with **`framework-skills-index`**, then open the package-specific skill from its internal table.

## Requirements

- Projects using **`@t4h.framework/*`** typically require **Node.js ≥ 22** (see each `package.json` in the framework monorepo).

## License

**MIT** (see `plugins/framework/.cursor-plugin/plugin.json` and any plugin license files).

## Author

**Tech for Humans** — support: `suporte@tech4h.com.br`.
