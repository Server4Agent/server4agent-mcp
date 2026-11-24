---
name: server4agent
description: Build and host web pages, sites, apps, APIs, bots and scheduled jobs at a live URL on a persistent server that keeps them running after the conversation ends. Use when the user asks to put something online, publish or share a page, deploy to a live URL, host an app or API, keep a bot running, run a script on a schedule, or delegate building to an on-server agent. Works through the Server4Agent MCP tools, or over HTTP with an API key.
when_to_use: Triggers on requests like "make a page and put it online", "deploy this to a live URL", "host this app", "spin up a server", "create a project", "run this script remotely", "provision some compute", or "delegate this task to a server agent".
---

Use Server4Agent's MCP tools (the `server4agent` MCP server) to build and host
web pages, sites and apps at a live URL: provision servers, manage projects, and
write and run the code on the box yourself. An agent that runs there can also
build for you, when the user asks for it.

## One-time setup

Check whether the connection already exists by looking for Server4Agent's
tools, such as `create_server` and `deploy`. Hosts prefix their names (for
example `mcp__server4agent__create_server`), and a plugin that bundles the
server has them from the start. If they are there, skip this section.

If they are not, connect the MCP server at `https://mcp.server4agent.com`.
There are two ways in, and the first is better:

**Sign in through the browser (preferred).** Server4Agent runs a standard OAuth
2.1 authorization server with dynamic client registration, so an OAuth-capable
MCP host needs nothing but the URL. Add it as a remote MCP server, and the host
opens a browser to sign in. Nothing to copy, and the token refreshes and can be
revoked from the dashboard.

```bash
your-mcp-host mcp add --transport http server4agent https://mcp.server4agent.com
```

**API key (for hosts without OAuth, and for CI).** The person you are
working for creates a key in the dashboard at https://www.server4agent.com and
adds it to their MCP host's own settings as an `Authorization: Bearer`
header. The steps for each host are at https://www.server4agent.com/docs/mcp.
Leave the key to them: never ask for it in the conversation, and never write it
into a file.

## Tools

37 tools. `[read-only]` never writes; `[destructive]` may delete or
overwrite, so confirm with the user before calling one.
Each line is the tool's first sentence. Its full description and arguments
come with the tool from the MCP server.

### Servers

A server is a sized compute box. Projects live inside it and share its resources.

- `create_server(task?, tier?, kind?, region?, project_sharing?, secrets?)`
  Provision a sized server box.
- `list_servers()` `[read-only]`
  List your servers with their tier, kind, status, project sharing, and current project count.
- `get_server(server_id)` `[read-only]`
  Get a server's tier, kind, status, capacity, project count, and project sharing.
- `update_server(server_id, task?, region?, project_sharing?)` `[destructive]`
  Update a server's nickname, region, or how much its projects may see of each other.
- `control_server(server_id, action)` `[destructive]`
  Pause a server (stop), resume it (start), or reboot it (restart).
- `delete_server(server_id)` `[destructive]`
  Tear down a server and every project on it.
- `get_usage()` `[read-only]`
  Get the account's AI spend and budget: how much has been spent this billing period, the plan's included credits, any budget cap, and whether new tasks are blocked (with the reason).

### Projects

A project is a workspace plus an app inside a server, with its own slug and URL. New projects are private: the URL only serves once you make it public.

- `list_templates()` `[read-only]`
  List the available project starters.
- `create_project(server_id, name?, description?, visibility?, access?, access_password?, lifecycle?, template?, source?, secrets?)`
  Create a project on a server: the home for one web page, site, app, API or bot, with its own URL at <slug>.apps.server4agent.com.
- `list_projects(server_id?)` `[read-only]`
  List projects.
- `get_project(project_id)` `[read-only]`
  Get a single project: name, slug, description, public URL, listen port, run plan, timestamps.
- `update_project(project_id, name?, description?, visibility?, access?, access_password?, lifecycle?, run_plan?)` `[destructive]`
  Rename a project, update its description, change visibility, link access or lifecycle, or correct its run plan.
- `init_project(project_id, proceed?)` `[destructive]`
  Start or resume bring-up for a project seeded from a git/archive source: the on-server agent installs dependencies, starts the app, and confirms it responds.
- `export_project(project_id, ttl_seconds?)`
  Package a project into a portable bundle and return a time-limited download link.
- `import_project(server_id, bundle, name?)`
  Recreate a project on a server from a bundle that export_project produced, here or on another Server4Agent deployment.
- `cleanup_project(project_id)` `[destructive]`
  Clean up an ephemeral project once its task is done.
- `delete_project(project_id)` `[destructive]`
  Delete a project.

### Secrets

Env vars for servers and projects. Values are write-only: no tool ever reads one back.

- `set_secret(key, value, description?)` `[destructive]`
  Create or update a secret (an env var, key → value) on the account.
- `list_secrets()` `[read-only]`
  List the account's secrets: keys, notes, and where each is bound (servers/projects).
- `attach_secret(key, value?, server_id, project_id?)` `[destructive]`
  Bind a secret to a scope so runs there see it as an env var.
- `delete_secret(secret_id)` `[destructive]`
  Delete a secret and all of its bindings.

### Agent runtime

Start and keep your app running, drive the shell yourself, or, when the user asks for it, hand work to the agent running on the server.

- `start_app(project_id, command, name?)` `[destructive]`
  Start a project's app and keep it running: the step that puts a project online after its files are written.
- `prompt(server_id, prompt, project_id?, follow_up_of?)` `[destructive]`
  Send a freeform prompt to the on-server agent.
- `exec(server_id, command, project_id?)` `[destructive]`
  Run a shell command on the server and return its output.
- `get_run(server_id, run_id)` `[read-only]`
  Get a run's status, its latest progress note, the agent's final response, and what it changed in the workspace (commit, branch, files).
- `get_logs(server_id, project_id?, lines?)` `[read-only]`
  Read the latest log tails from a server: one entry per log file that a project's processes have written (each long-running process appends to .server4agent/<name>.log in its project folder).

### File workspace

Read and write the project's files directly when you do not need the agent.

- `list_files(server_id, project_id?)` `[read-only]`
  List paths in the server's workspace.
- `read_file(server_id, path, project_id?, encoding?)` `[read-only]`
  Read a file from the server's workspace.
- `write_file(server_id, path, content, project_id?, encoding?)` `[destructive]`
  Write or overwrite a file in the server's workspace.
- `delete_file(server_id, path, project_id?)` `[destructive]`
  Delete a file from the server's workspace.
- `get_download_url(server_id, path, project_id?, ttl_seconds?)`
  Mint a time-limited link to download one file from the server's workspace, e.g. after generating a report or converting a file, hand this URL back to the user instead of the raw content.

### Build and deploy

Put a running project online with deploy, or, when the user asks for one, start an agent run that builds from a goal.

- `start_build(server_id, goal, project_id?)` `[destructive]`
  Have the on-server agent build a web page, site, app or API from a plain-language goal.
- `get_build(server_id, build_id)` `[read-only]`
  Get a build's status.
- `deploy(server_id, project_id?)`
  Deploy a project to its live public URL at <slug>.apps.server4agent.com: the last step in putting a web page, site, app or API online.

### Custom domains

Serve a public project at a domain the user owns, with HTTPS issued automatically. On paid plans. The user adds two DNS records; check_domain switches the domain on once both are found.

- `add_domain(project_id, domain)`
  Connect a custom domain the user owns to a project, so the app also answers at that address over HTTPS.
- `check_domain(project_id, domain)`
  Look up a custom domain's DNS records now and, when both are right, switch it on: the certificate is issued automatically and the domain starts serving within a few minutes.
- `remove_domain(project_id, domain)` `[destructive]`
  Disconnect a custom domain from a project.

## Common workflows

**Build and deploy a project: you write the code**
This is the way to build anything, from one page to an API: you write the
code, and these tools put it on the server and online. Nothing here serves a
directory. A project's URL is routed to one port inside the box, so a page or a
server is only live once a process is answering on that port.

1. `list_servers` and reuse a running one with room for another project, or
   `create_server` when there is none. Either gives you a `server_id`. A new
   server takes about a minute to start: poll `get_server` until
   `provisioning` reads `ready` before writing files.
2. `create_project(server_id, name, visibility="public")` returns `project_id`
   and `listen_port`.
3. `write_file(server_id, "public/index.html", "<html>...", project_id)`, and
   the same for each other file. Static pages go under `public/`: the
   project folder itself holds
   `.server4agent/env`, the secrets bound to this project, and a file server
   rooted at the project would publish them.
4. `exec` installs dependencies (`npm install`,
   `pip install -r requirements.txt`) and waits for them.
5. `start_app(project_id, command)`. It runs the command in the background
   with `PORT` set to the project's port, records it so the app comes back
   after a restart, and waits for the port to answer.

```
start_app  project_id, command: "node server.js"
start_app  project_id, command: "python3 -m http.server $PORT --bind 0.0.0.0 --directory public"
```

6. The result says whether the app is answering, with its log tail and the
   URL. If it exited, the log says why: fix that and call `start_app` again.
   Calling it again with the same `name` is also how a changed server picks
   up new code.
7. `deploy(server_id, project_id)`. It checks the app is listening, makes the
   project public if it is not, and returns the URL once it answers.

Your app has to listen on `$PORT`, not a port of its own.

**Hand a build to the on-server agent, when the user asks for it**
An agent on the server can write and start the code instead. It spends the
account's AI budget, so use it when the user asks for that agent, not as the
way to build.
1. `create_project` as above.
2. `prompt(server_id, "Build and serve ...", project_id)`, or `start_build`
   with the goal. Poll `get_run` or `get_build` until it finishes.
3. `deploy(server_id, project_id)` once the run finishes.

**Follow up on a run**
Every `prompt` call starts a new conversation. To change what a run built, or
to answer a run whose status is `awaiting_input`, pass its id:
`prompt(server_id, "Add a login page", follow_up_of=run_id)`. The agent picks
up that conversation in the same folder and project, so it knows what it did.

**Ephemeral one-shot task**
1. `create_project(server_id, name, lifecycle="ephemeral")`.
2. `exec` to do the work, or `prompt` when the user asks for the on-server agent.
3. `cleanup_project(project_id)` frees the slot when done.

**Check capacity before creating**
`get_server(server_id)` reports projects used against the tier cap. At the cap,
free a slot or move the server up a tier before calling `create_project`,
which otherwise fails.

**Watch spend before a long run**
`get_usage()` reports spend for the period, the budget cap, and whether new
tasks are blocked. Worth a call before a series of `prompt` invocations.

## Notes that save a round trip

- **Some questions are for people, not tools.** The Business or Enterprise
  plan, running in your own cloud, a security questionnaire, or an account
  problem no tool can fix: write to hello@server4agent.com for the person you
  are helping (the form at https://www.server4agent.com/contact ends with a
  human check). A person answers within one business day.
- Projects are **private by default**. A newly created project's `url` is
  `null` until you set `visibility="public"`. The slug and URL do not change
  when you toggle visibility.
- A project URL is answered by **a process, not a folder**. Bind the project's
  `listen_port` (from `create_project` or `get_project`) rather than a port of
  your own: an in-pod proxy owns the one published port and routes each public
  project's hostname to its own `listen_port`. On a box old enough to predate
  that proxy there is no demux and 8080 is published directly, so if the URL
  will not answer on `listen_port`, that is the fallback worth trying.
- **What the URL answers tells you which thing is wrong.** A redirect to a
  "Nothing found" page means nothing is routed to that slug: the project is
  private, or it does not exist (the two are deliberately indistinguishable).
  A 200 reading "server4agent: unknown project" or "server4agent: no project
  routed to this server yet" means the request reached the box but the in-pod
  proxy has no route for it yet, so give reconciliation a moment. A 503
  "Temporarily unavailable" means the routing is right and nothing is
  listening on `listen_port`, which is the one your own app causes. A sign-in
  page means the project's `access` is `team`, so it is serving fine and only
  the account's members get in (an allowlist can narrow it further still). A
  password prompt means `access` is `password`: also serving fine, for
  anyone who has the password. Set or change it with `update_project`
  `access_password`; it is never returned, so ask the owner for it rather
  than looking for it.
- A process started through `exec` **dies with the call**. Start anything
  long-running with `start_app`, which also records it in the project's
  `.server4agent/start.sh` so a restart, a pause and a resume bring it back.
  Other lines in that file are left alone. If you edit it by hand, keep it to
  plain, self-contained commands, one per line: boot and resume do not replay
  it the same way, so a line that leans on a `cd` or an `export` from an
  earlier line works in one and not the other.
- `deploy` **publishes, it does not build**. It refuses when nothing is
  listening on the project's port, so start the app first (`start_app`, or
  `prompt`). On success it returns the project's URL with status `live`,
  or `publishing` when the URL has not answered yet: call it again shortly.
- `create_project` **fails at the tier cap**: small holds 3 projects, medium 10,
  large is unlimited. Moving a server down a tier is refused when the smaller
  cap would not fit the projects already there.
- `prompt` and `start_build` are **asynchronous**. They return an id, and you
  poll `get_run` or `get_build` for progress.
- Secret **values are write-only**. `list_secrets` returns keys, notes, and
  bindings, never a value.
- **Email needs no provider key.** An app that should email the people on the
  account (an alert, a report) can POST to the URL in its `S4A_EMAIL_URL` env
  var, sending `S4A_EMAIL_TOKEN` as a bearer token, with a JSON body of
  `subject`, `text` and, optionally, `to`. A project's `exec` commands and its
  `start_app` apps get both variables, and `start_app` keeps them in
  `.server4agent/env.sh` for a cron job to source. It reaches only the
  account's own people at confirmed addresses (the owner when `to` is left
  out), so there is nothing to ask the user for. Mail to anyone else needs
  their own provider, set with `set_secret`.
- Argument details beyond the signatures above are in each tool's own JSON
  schema, which your MCP host already has once connected.
