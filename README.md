# Server4Agent

<!--
  Generated from the Server4Agent MCP server's tool table and published with
  each release of its registry listing. Changes made here are overwritten.
-->

Servers your AI assistant builds on. It writes the code, runs it and ships each app to a live URL.

Server4Agent is a remote MCP server. Connect it to your AI assistant and ask
for a web page, a site, an app, an API, a bot or a scheduled job. The assistant
gets a server to build on, writes and runs the code there, and publishes each
project at its own live URL. Processes keep running after the conversation
ends, so backends and bots stay up.

There is nothing to install or run on your machine. The server is hosted at
`https://mcp.server4agent.com`, and this repository holds its listing in the MCP
Registry.

## Connect

- **Endpoint:** `https://mcp.server4agent.com`
- **Transport:** streamable HTTP
- **Sign-in:** OAuth 2.1 with dynamic client registration, or an API key sent
  as a bearer token

You need a Server4Agent account: https://www.server4agent.com

### VS Code and GitHub Copilot

Install Server4Agent from the MCP server gallery in the Extensions view
(search for `@mcp Server4Agent`), or add it to `.vscode/mcp.json`:

```json
{
  "servers": {
    "server4agent": {
      "type": "http",
      "url": "https://mcp.server4agent.com"
    }
  }
}
```

Start the server and VS Code opens a browser to sign in. There is no key to
copy, and you can revoke the sign-in from the dashboard. The tools are then
available to Copilot Chat in agent mode.

### GitHub Copilot CLI and other hosts

Add `https://mcp.server4agent.com` as a remote MCP server. A host that supports OAuth opens a
browser to sign in and needs nothing else. A host without OAuth, the Copilot
CLI among them, takes an API key from the dashboard, which you add in that
host's own settings as an `Authorization: Bearer` header. Steps for each
app: https://www.server4agent.com/integrations

### As a plugin or a skill

This repository is also a plugin: the skill plus this server, in one install.
In a host that reads plugin marketplaces:

```
/plugin install server4agent --marketplace Server4Agent/server4agent-mcp
```

For the skill alone, in any agent that reads Agent Skills:

```bash
npx skills add Server4Agent/server4agent-mcp --skill server4agent
```

Setup for more apps: https://www.server4agent.com/integrations

## Use it

Ask in plain language. For example:

- "Make a landing page for my bakery and put it online."
- "Build an API that turns a CSV upload into JSON, and give me the URL."
- "Check our website every hour and keep a log of when it is down."
- "What is running on my servers, and how much have I spent this month?"

How it fits together:

- A **server** is a sized box (small, medium or large) that holds projects. One
  server holds several.
- A **project** is a folder plus an app, with its own URL at
  `<slug>.apps.server4agent.com`. Projects are private until `deploy` makes
  them public.
- Your assistant can write the files itself, or hand a goal to the agent that
  runs on the server (`start_build`, `prompt`), which spends the account's
  AI budget.
- Tools that delete or overwrite are marked destructive, so your host can ask
  you before it runs one.

## Tools

38 tools. `[read-only]` never writes; `[destructive]` may delete or
overwrite.

### Servers

A server is a sized compute box. Projects live inside it and share its resources.

- `create_server`: Provision a sized server box.
- `list_servers` `[read-only]`: List your servers with their tier, kind, status, project sharing, and current project count.
- `get_server` `[read-only]`: Get a server's tier, kind, status, capacity, project count, and project sharing.
- `update_server` `[destructive]`: Update a server's nickname, region, or how much its projects may see of each other.
- `control_server` `[destructive]`: Pause a server (stop), resume it (start), or reboot it (restart).
- `delete_server` `[destructive]`: Tear down a server and every project on it.
- `get_usage` `[read-only]`: Get the account's AI spend and budget: how much has been spent this billing period, the plan's included credits, any budget cap, and whether new tasks are blocked (with the reason).

### Projects

A project is a workspace plus an app inside a server, with its own slug and URL. New projects are private: the URL only serves once you make it public.

- `list_templates` `[read-only]`: List the available project starters.
- `create_project`: Create a project on a server: the home for one web page, site, app, API or bot, with its own URL at &lt;slug&gt;.apps.server4agent.com.
- `list_projects` `[read-only]`: List projects.
- `get_project` `[read-only]`: Get a single project: name, slug, description, public URL, listen port, run plan, timestamps.
- `update_project` `[destructive]`: Rename a project, update its description, change visibility, link access or lifecycle, or correct its run plan.
- `init_project`: Start or resume bring-up for a project seeded from a git/archive source: the on-server agent installs dependencies, starts the app, and confirms it responds.
- `export_project`: Package a project into a portable bundle and return a time-limited download link.
- `import_project`: Recreate a project on a server from a bundle that export_project produced, here or on another Server4Agent deployment.
- `cleanup_project` `[destructive]`: Clean up an ephemeral project once its task is done.
- `delete_project` `[destructive]`: Delete a project.

### Secrets

Env vars for servers and projects. Values are write-only: no tool ever reads one back.

- `set_secret` `[destructive]`: Create or update a secret (an env var, key → value) on the account.
- `list_secrets` `[read-only]`: List the account's secrets: keys, notes, and where each is bound (servers/projects).
- `attach_secret` `[destructive]`: Bind a secret to a scope so runs there see it as an env var.
- `delete_secret` `[destructive]`: Delete a secret and all of its bindings.

### Agent runtime

Start and keep your app running, delegate work to the agent running on the server, or drive the shell yourself.

- `start_app` `[destructive]`: Start a project's app and keep it running: the step that puts a project online after its files are written.
- `prompt`: Send a freeform prompt to the on-server agent.
- `exec` `[destructive]`: Run a shell command on the server and return its output.
- `get_run` `[read-only]`: Get a run's status, its latest progress note, the agent's final response, and what it changed in the workspace (commit, branch, files).
- `get_logs` `[read-only]`: Read the latest log tails from a server: one entry per log file that a project's processes have written (each long-running process appends to .server4agent/&lt;name&gt;.log in its project folder).

### File workspace

Read and write the project's files directly when you do not need the agent.

- `list_files` `[read-only]`: List paths in the server's workspace.
- `read_file` `[read-only]`: Read a file from the server's workspace.
- `write_file` `[destructive]`: Write or overwrite a file in the server's workspace.
- `delete_file` `[destructive]`: Delete a file from the server's workspace.
- `get_download_url`: Mint a time-limited link to download one file from the server's workspace, e.g. after generating a report or converting a file, hand this URL back to the user instead of the raw content.

### Build and deploy

An agent run that builds from a goal, and deploy, which puts a running project online.

- `start_build`: Have the on-server agent build a web page, site, app or API from a plain-language goal.
- `get_build` `[read-only]`: Get a build's status.
- `deploy`: Deploy a project to its live public URL at &lt;slug&gt;.apps.server4agent.com: the last step in putting a web page, site, app or API online.

### Self-tuning

Feed an evaluation report back into the agent's own code so it can fix what the report found.

- `start_tuning`: Fix an agent from an evaluation platform's report.
- `get_tuning_session`: Get a tuning run's status and the agent run behind it.
- `list_tuning_sessions` `[read-only]`: List tuning runs on this account, newest first.
- `report_tuning_result`: Tell the evaluation platform what a tuning run changed, and where the fixed build is reachable.

Every tool's arguments, with worked examples, are in the Agent Skill at
https://www.server4agent.com/skill.md, and in each tool's JSON schema once you
are connected.

## Where your data goes

Your assistant sends data to two Server4Agent endpoints, and to no other
service:

- **`https://mcp.server4agent.com`**, the MCP server this plugin connects. It receives every
  tool call and its arguments: the files, commands, prompts and secret values
  your assistant sends.
- **`https://api.server4agent.com/pages`**, which the skill uses only when no connection is
  set up, to publish a static page without an account. It receives that page's
  files.

The skill also links to pages on https://www.server4agent.com for reference. Nothing is sent there.

How long it is kept:

- Files, apps and secrets on a server stay until you delete them, or the server
  they are on.
- A page published without an account is public for a day and can be claimed
  for three. If nobody claims it, its files are deleted after that.
- Account data is kept while the account is open, and for up to 90 days after
  it closes. Server logs are kept for 30 days.

What happens to it on our side, including the services we use to run it, is in
the privacy policy: https://www.server4agent.com/privacy

## Links

- Website: https://www.server4agent.com
- Connect another app: https://www.server4agent.com/integrations
- Questions: hello@server4agent.com
