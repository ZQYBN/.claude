---
name: user-shorthand
description: User uses 1/0 as shorthand for true/false confirmations
metadata: 
  node_type: memory
  type: user
  originSessionId: 09724639-d775-408f-b811-03291051d9a5
---

The user uses numeric shorthand for binary responses:

- **`1`** = true / yes / proceed / confirm / continue — equivalent to approval
- **`0`** = false / no / stop / redo / reject — equivalent to denial

When the user sends just `1` or `0` in response to a question or proposal, interpret it as the corresponding binary answer.
