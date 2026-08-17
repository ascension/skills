# Native adapter

Load only when the host is a harness isolated-agent primitive.

## Spawn

Use that primitive with zero inherited turns (for example `fork_turns: "none"`) and one packet. Pass the spawn intent's `model` and `effort` explicitly. A new agent that copies conversation history fails the gate.

## Wait and read

Wait until the primitive reports the session settled. The receipt is the agent's return value. Confirm the primitive can create nested subagents before the first mutating leg (review requires parallel Standards and Spec).
