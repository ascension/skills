# Fresh session on every host

Every relay leg is a **fresh session**, including on Buzz. Standing `@mention` agents keep memory and fail the gate; `draft-create` needs owner review and is not AFK. The Buzz adapter must spawn a one-shot process. If that probe fails, do not run on Buzz — pick another host or stop.

**Status:** accepted

**Considered options:** relax the gate for standing Buzz agents; hybrid (fresh for implement, standing for review). Rejected: the ledger assumes the worker saw only this packet.
