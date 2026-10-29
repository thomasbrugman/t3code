# Architecture context

Cursor context for the `thomasbrugman/t3code` fork. The repository docs are the source of
truth. Keep this file out of pull requests to `pingdotgg/t3code`.

Read these before changing behavior:

- [Architecture](../docs/internals/overview.md) — ownership, orchestration, checkpoints
- [Glossary](../docs/internals/glossary.md) — environment, client, project, thread, turn
- [AGENTS.md](../AGENTS.md) — how a turn is committed, where code lives, how to test
- [Effect services](../docs/internals/effect-services.md), [testing](../docs/internals/testing.md),
  and [web UI](../docs/internals/web-ui.md) when the change touches those areas

## Shape of the system

Web, desktop, and mobile clients send typed WebSocket requests to the server that owns the
workspace. The server turns each request into a command. The orchestrator
(`apps/server/src/orchestration-v2/Orchestrator.ts`) serializes commands and decides events
without doing I/O. The event sink commits those events, the projections the UI reads, the command
receipt, and outbox effects in one transaction. The effect worker runs the effects, such as a
provider turn or a checkpoint, and feeds the results back as commands.

Provider CLIs run as subprocesses. Each provider has an adapter that translates its protocol into
orchestration events. A turn ends with a checkpoint, which is a hidden git ref used for diffs and
restore.

## Where code lives

- `apps/server` — WebSocket server, orchestration, providers, checkpointing
- `apps/web` — React/Vite UI. `apps/desktop` wraps it with Electron. `apps/mobile` is React
  Native. `apps/marketing` is the site.
- `packages/contracts` — schemas that cross the wire
- `packages/shared` — shared runtime utilities
- `packages/client-runtime` — client logic shared by web and mobile
- `.repos/` — vendored read-only references. Do not edit or import them.

A wire change starts in `packages/contracts`. A behavior a person can trigger belongs in a service
method, not only in a WebSocket handler.
