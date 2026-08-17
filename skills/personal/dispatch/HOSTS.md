# Hosts

A named **adapter** on the spawn intent or in the user's message is the **pin**. Otherwise detect the live **host**, then load only that host's adapter. PATH presence is not a live session.

## Detect

Check what this session actually is, **first match wins**. Cursor-in-Herdr also has a native isolated-agent primitive; that is still Herdr.

1. **Herdr** — `herdr pane current` returns this session's pane (`HERDR_PANE_ID` may be unset). Adapter: [adapters/herdr.md](adapters/herdr.md)
2. **Buzz** — the session is a Buzz agent turn (channel/relay context in the prompt, or `BUZZ_RELAY_URL` plus a Buzz identity). Adapter: [adapters/buzz.md](adapters/buzz.md)
3. **Pi** — this process is `pi` and will dispatch further `pi -p --no-session` legs. Adapter: [adapters/pi.md](adapters/pi.md)
4. **Native** — this harness exposes an isolated-agent primitive and no host above matched. Adapter: [adapters/native.md](adapters/native.md)

Zero matches and no named adapter: stop and ask for a **pin**. A pin names exactly one host above.

With no named adapter, stay on the host this session is already on.

## Adapter contract

Each adapter implements **spawn**, **wait**, and **read** for a spawn intent. It resolves `model` and `effort` against the host catalog (`--help`, `--list-models`). A name it cannot resolve, or a runner that cannot start, is a capability failure — stop as `blocked` or ask.

Spawn produces a **fresh session**.
