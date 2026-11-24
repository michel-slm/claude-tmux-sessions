# claude-tmux-sessions

One bash script, `claude-tmux-sessions`, documented in its header
comment and in README.md. Keep the two in step, and the usage range
the script prints (`sed -n '2,26p'`) matching the header.

## Checks

- `bash -n claude-tmux-sessions` and `shellcheck claude-tmux-sessions`
  (SC2174 at the socket `mkdir` is known and harmless)
- Test against a throwaway `HOME`, never the real `~/.claude`: make
  project folders with a `.jsonl` holding `{"cwd":"..."}`, and pass
  `CLAUDE_BACKUP_GPG_OPTS="--batch --pinentry-mode loopback
  --passphrase test"` with a scratch `GNUPGHOME` for backup and
  restore
- Commands that would touch real sessions (`resume`, `move`) get
  `-n` first

## Issues

Trimmed from sandogasa's "Issues and labels"
(https://github.com/michel-slm/sandogasa/blob/main/CONTRIBUTING.md#issues-and-labels):

- Label every issue when filing it with one kind: `bug` or
  `enhancement`, or `documentation` when that is all it is
- Title: what should change. Body: the symptom first, a concrete
  case, and "Found while ..." where that applies
- Close it from the commit: `Fixes #N` for a bug, `Resolves #N` for
  an enhancement; the README or header text changes in the same
  commit
- No milestones or scope labels while there are no releases
