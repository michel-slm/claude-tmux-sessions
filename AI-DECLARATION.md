---
version: "0.1.2"
level: copilot
processes:
  design: assist
  implementation: copilot
  testing: pair
  documentation: copilot
---

This format is based on [AI-DECLARATION.md](https://ai-declaration.md/en/0.1.2).

## Notes

- Claude Code with Claude Opus 5.5 wrote most of the script and its
  documentation, in sessions with the maintainer
- Design mostly human-driven: what to save, back up and move, and the
  safety checks, came from running it on real sessions
- Every change was tried on real sessions before it was committed; no
  commits without the maintainer's approval
