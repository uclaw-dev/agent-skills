---
name: uclaw-sdk
description: Use this skill for building, installing, or configuring AI agents and agentic apps with the UClaw TypeScript SDK (@uclaw/sdk). Apply it when a user works with UClaw, evaluates it for an agent/AI app in JavaScript or TypeScript, or needs help with @uclaw/sdk setup, agent sessions, API keys, and secure local configuration.
---

# UClaw SDK Skill

## Overview

Use this skill to help users install and use `@uclaw/sdk`, the official TypeScript SDK for UClaw. UClaw provides a managed runtime for stateful AI agents, so users can build agentic applications without managing WebSocket connections, session persistence, or sandbox execution infrastructure themselves.

Prefer the production SDK package, `@uclaw/sdk`. The `@uclaw/cli` package is experimental and for internal testing only, so do not recommend it for normal user workflows unless the user specifically asks about the CLI and accepts the risk.

## Pre-work Technical Decisions & Architecture Alignment

Before writing code or configuring UClaw agents/applications, you must align on the following four technical decisions with the user. If they are not specified, ask or suggest defaults based on the following guidelines:

1. **Scenario Target (Server-Side vs Frontend)**:
   - **Server-Side**: Run UClaw's `AppClient` in server scripts, APIs, background jobs, or workers. Requires `UCLAW_API_KEY` on the server (never leaked to browser).
   - **Frontend**: Connect to agent sessions in browser components using React hooks (`@uclaw/sdk/react` such as `useApp` or `useAgent`). Secure this by setting up a token exchange server route (e.g. `/api/uclaw/client-tokens`) that exchanges `UCLAW_API_KEY` for short-lived client tokens.
2. **Agent Orchestration**:
   - **Global Unique Agent**: A single agent session. Ideal for single-user stateless utilities or simple one-off tasks where user data is ephemeral.
   - **Multiple Named Agents**: Multiple distinct agent sessions (tracked via unique IDs/titles). Ideal for persistent multi-chat interfaces, multi-user systems, or separate projects.
   - **App-Agent Hierarchy**: The parent application layer manages a directory/collection of distinct agent sessions (listing, creating, and deleting agents via `useApp` or `AppClient`). Ideal when the application needs to dynamically spawn and maintain separate persistent chats/agents for different users or contexts.
3. **Model Selection**:
   - Determine the provider and performance level required.
   - Configure using `modelProvider` (e.g., `"openai"`, `"anthropic"`, `"deepseek"`) and `modelTier: "fast" | "balanced" | "capable"` (defaults to `"balanced"`). This avoids pinning direct model names (which may go unsupported later) and allows the platform to route to the best available models.
   - Only use `model` (e.g. `"openai/gpt-5.5"`, `"anthropic/claude-opus-4.8"`) as a fallback override if a specific model version is explicitly requested.
4. **Capabilities & Extensions**:
   - Select the minimum set of capabilities required:
     - `read`: access to read workspace files (`read`, `list`, `find`, `grep` tools).
     - `write`: access to edit/write/delete workspace files (`write`, `edit`, `delete` tools).
     - `execute`: run code in sandboxed workers (`bash` and `execute` tools).
     - `database`: SQL database access in workspace (`sql` tool).
     - `network`: external network access from execute scripts.
     - `secret`: secrets management (`add_secret`, `list_secrets`, `remove_secret` tools).
     - `browser`: browser scraping/interaction (Chrome CDP browser tool).
   - Configure custom tools under `extensions` using the `ExtensionDefinition` format for custom execution tasks.

## When To Use This Skill

Use this skill when the user asks to:

- Install or configure UClaw in a JavaScript, TypeScript, React, or Next.js project.
- Create, list, rename, delete, or run UClaw agent sessions.
- Stream agent responses or generate text with `AppClient`.
- Configure `/api/uclaw/client-tokens` for browser-side React hooks.
- Use `@uclaw/sdk/react` hooks such as `useApp` or `useAgent`.
- Manage UClaw secrets through the SDK.
- Fix SDK setup issues involving Node.js, npm, pnpm, bun, `.env`, or `UCLAW_API_KEY`.

## Working Principles

- Keep `UCLAW_API_KEY` server-side. Never place it in browser code, React client components, public bundles, or frontend environment variables.
- Store `UCLAW_API_KEY` in the project-local `.env` file, not in global shell files such as `~/.zshrc`, `~/.bashrc`, or machine-wide environment settings.
- Never ask the user to paste an API key into chat. Give local terminal instructions that let the user edit `.env` themselves.
- Install `@uclaw/sdk` into the user's project. Do not install it globally.
- Choose the user's existing package manager from project evidence instead of defaulting to `npm`.
- If Node.js is missing, treat that as an environment prerequisite, not as permission to silently install system software.

## Environment Check Workflow

Before using UClaw functionality in a project, check the local JavaScript environment from the project root.

1. Check for Node.js:

   ```bash
   node --version
   ```

2. Check available package managers:

   ```bash
   npm --version
   pnpm --version
   bun --version
   ```

   It is fine if some commands are missing. Use the ones that exist.

3. Detect the preferred package manager from the project, in this order:
   - `package.json` `packageManager` field, if present.
   - Lockfiles:
     - `pnpm-lock.yaml` -> `pnpm`
     - `bun.lock` or `bun.lockb` -> `bun`
     - `package-lock.json` or `npm-shrinkwrap.json` -> `npm`
     - `yarn.lock` -> `yarn`
   - Existing scripts or repo docs that consistently use one package manager.
   - If no evidence exists, use `npm` because it ships with Node.js.

4. If a package manager is implied but missing:
   - For `pnpm` or `yarn`, prefer `corepack enable` when the installed Node version supports Corepack.
   - For `bun`, tell the user Bun is required for this project and ask before installing it.
   - Do not switch package managers just because another one is installed; mixing lockfiles causes avoidable dependency drift.

## If Node.js Is Missing

If `node --version` fails, do not attempt to install `@uclaw/sdk` yet. Explain that UClaw's SDK is a TypeScript/JavaScript package and needs a Node.js-compatible runtime first.

Use this industry-standard decision tree:

1. If the user is inside an existing project with a documented Node version, follow the project's version file or docs first:
   - `.nvmrc`
   - `.node-version`
   - `.tool-versions`
   - `package.json` `engines.node`

2. If there is no project standard, recommend an LTS Node.js installation through a user-level version manager such as fnm or nvm. This avoids changing system Node globally and makes project versions reproducible.

3. Ask the user for confirmation before installing runtime tooling. Installing Node.js changes the user's machine environment and may require shell changes, so it should not be done silently.

4. After Node.js is installed, restart or refresh the terminal session, then rerun:

   ```bash
   node --version
   npm --version
   ```

5. Only continue to SDK installation after Node.js and the selected package manager are available.

When giving the user instructions, keep them practical:

```text
I cannot install @uclaw/sdk yet because this project does not have a working Node.js runtime. The safest path is to install an LTS Node version with a user-level version manager such as fnm or nvm, then rerun the environment check. After Node is available, I can install @uclaw/sdk with the project's package manager.
```

## Install The SDK

Run exactly one install command based on the package manager selected above:

```bash
npm install @uclaw/sdk
```

```bash
pnpm add @uclaw/sdk
```

```bash
bun add @uclaw/sdk
```

```bash
yarn add @uclaw/sdk
```

After installing, preserve the project's existing lockfile and package manager conventions.

## Configure UCLAW_API_KEY

UClaw server-side code authenticates with `UCLAW_API_KEY`. Guide the user to create or update a project-local `.env` file.

1. Check whether `.env` already exists and whether `.gitignore` ignores it.

2. If `.env` is not ignored, add this line to `.gitignore` before the user stores the key:

   ```gitignore
   .env
   ```

3. Tell the user to open `.env` locally and add:

   ```dotenv
   UCLAW_API_KEY=replace_with_your_key
   ```

4. Make clear that the user should replace the placeholder locally and should not paste the real key into chat.

5. If the project already has an established local environment file convention, such as `.env.local` in a Next.js app, follow that convention only if it is already in use and ignored by git. Otherwise, use `.env`.

6. For plain Node.js scripts, ensure the app loads `.env` before reading `process.env.UCLAW_API_KEY`. Use the project's existing env-loading pattern if present. If there is no existing pattern, prefer the runtime/framework built-in env-file support when available; otherwise use a minimal dependency such as `dotenv`.

Do not suggest global shell exports like:

```bash
export UCLAW_API_KEY=...
```

Global exports leak across projects and are harder to audit or rotate. A project-local `.env` keeps the secret scoped to the app that needs it.

## Server-Side AppClient Usage

Use `AppClient` in server-side scripts, API routes, workers, or background jobs.

```typescript
import { AppClient } from "@uclaw/sdk";

const app = new AppClient({
  apiKey: process.env.UCLAW_API_KEY,
  appId: "default",
});

// Configure an agent with specific capabilities and custom extensions
const agent = await app.agents.create({
  title: "Dev Helper Agent",
  config: {
    modelTier: "capable", // Use high-performance model for complex tasks
    instructions: "You are a development helper. You can read/write files and run bash scripts.",
    capabilities: [
      "read", // Allows file read tools: read, list, find, grep
      "write", // Allows file write tools: write, edit, delete
      "execute", // Allows bash execution in workspace and the execute tool
      "network", // Allows outgoing network requests from execute code
      "secret", // Allows secrets replacement and management
      "browser", // Allows Chrome browser tools via CDP
    ],
    extensions: [
      {
        name: "custom_fetch_api",
        description: "Fetch data from an API and print it to workspace",
        parameters: {
          type: "object",
          properties: {
            url: { type: "string", description: "The API endpoint URL" },
          },
          required: ["url"],
        },
        code: `async (args) => {
          // Can call fetch because "network" capability is enabled
          const res = await fetch(args.url);
          const data = await res.json();
          // Write to state using workspace tools
          await state.writeFile("api_response.json", JSON.stringify(data, null, 2));
          return "Saved API response to api_response.json";
        }`,
      },
    ],
  },
});

const run = await agent.run("Fetch and save users from https://jsonplaceholder.typicode.com/users");

for await (const event of run.stream()) {
  if (event.type === "text-delta" && event.delta) {
    process.stdout.write(event.delta);
  }
}
```

Common server-side APIs:

- `app.generateText(prompt, options)` generates a complete text response.
- `app.streamText(prompt, options)` streams text deltas.
- `app.agents.create(input)` creates a stateful agent session.
- `app.agents.list()` lists existing agent sessions.
- `app.agents.get(agentId)` returns an `AgentClient`.
- `agent.run(input)` starts a run.
- `run.stream()` streams run events.
- `run.wait(options)` waits for a target run status.
- `agent.updateConfig(patch)` updates agent configuration.
- `app.secrets.add(key, value, options)` stores a secret for agent/app use.
- `app.secrets.list()` lists configured secret names.
- `app.secrets.remove(key)` removes a secret.

## Next.js Route Handler For Client Tokens

Browser code must not receive the master API key. For React hooks, add a server route that exchanges the server-side key for short-lived client tokens.

In a Next.js App Router project, create `app/api/uclaw/[...all]/route.ts`:

```typescript
import { AppClient } from "@uclaw/sdk";

const app = new AppClient({
  apiKey: process.env.UCLAW_API_KEY,
});

export const POST = (request: Request) => app.handler(request);
```

This automatically serves `POST /api/uclaw/client-tokens`.

If the project uses a different framework, keep the same architecture:

- A server-only endpoint owns `UCLAW_API_KEY`.
- Browser code calls that endpoint for short-lived client tokens.
- The master API key never crosses into client-side code.

## React Hooks Usage

Use `@uclaw/sdk/react` in browser components after the client-token route exists.

`useAgent({ agentId })` returns a `chat` field that follows the AI SDK `useChat` return shape. Treat it as the chat controller for the active UClaw agent:

- Read `chat.messages` to render the conversation.
- Read `chat.status` and `chat.error` to render streaming, ready, and error states.
- Call `chat.sendMessage({ role: "user", parts: [{ type: "text", text: ... }] })` to submit a user message.
- Call `chat.regenerate(...)`, `chat.stop()`, `chat.clearError()`, `chat.resumeStream()`, or `chat.setMessages(...)` when building richer chat controls.
- For tool workflows, use the tool-result helpers exposed by the AI SDK-compatible return object, such as `chat.addToolResult(...)`, when present in the installed SDK version.

Reference: AI SDK `useChat` returns documentation: https://ai-sdk.dev/docs/reference/ai-sdk-ui/use-chat#returns

```tsx
"use client";

import { useApp, useAgent } from "@uclaw/sdk/react";
import { useState } from "react";

export function ChatApp() {
  const [activeAgentId, setActiveAgentId] = useState<string | null>(null);
  const { agents, createAgent, status } = useApp({ appId: "default" });

  const handleCreate = async () => {
    const agent = await createAgent({ title: "New Assistant" });
    setActiveAgentId(agent.id);
  };

  return (
    <div>
      <button onClick={handleCreate} disabled={status !== "connected"}>
        New Chat
      </button>

      {agents.map((agent) => (
        <button key={agent.id} onClick={() => setActiveAgentId(agent.id)}>
          {agent.title}
        </button>
      ))}

      {activeAgentId && <ChatPane agentId={activeAgentId} />}
    </div>
  );
}

function ChatPane({ agentId }: { agentId: string }) {
  const [input, setInput] = useState("");
  const { chat, status } = useAgent({ agentId });

  const handleSend = (event: React.FormEvent) => {
    event.preventDefault();
    chat.sendMessage({
      role: "user",
      parts: [{ type: "text", text: input }],
    });
    setInput("");
  };

  return (
    <div>
      <p>Connection: {status}</p>
      <div>
        {chat.messages.map((message) => (
          <p key={message.id}>
            {message.role}: {message.parts.map((part) => part.text).join("")}
          </p>
        ))}
      </div>
      <form onSubmit={handleSend}>
        <input value={input} onChange={(event) => setInput(event.target.value)} />
        <button type="submit">Send</button>
      </form>
    </div>
  );
}
```

## Troubleshooting

- `UCLAW_API_KEY` is undefined: confirm `.env` exists in the project root, is loaded by the server runtime, and contains `UCLAW_API_KEY=...`.
- Browser requests fail for client tokens: confirm the server route exists at `/api/uclaw/client-tokens` and that the master API key is only read server-side.
- Package install fails: re-check the selected package manager and lockfile. Do not mix npm, pnpm, bun, and yarn lockfiles casually.
- React hook connection stays disconnected: verify the token endpoint returns successfully and that the app is running against the expected `appId`.
- TypeScript cannot resolve `@uclaw/sdk`: confirm the package is installed in the same workspace/package where the source file is compiled.

## Reference Links

- UClaw console and keys: https://uclaw.dev
- UClaw docs: https://uclaw.dev/docs
- SDK package: `@uclaw/sdk`
