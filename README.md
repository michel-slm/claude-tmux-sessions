# claude-tmux-sessions

[![AI-DECLARATION: copilot](https://img.shields.io/badge/䷼%20AI--DECLARATION-copilot-fee2e2?labelColor=fee2e2)](https://ai-declaration.md)

Keep long-running [Claude Code](https://code.claude.com/) sessions
across restarts and machines. Each named session runs in its own
tmux window; `claude-tmux-sessions` records them, starts them again
after a reboot or crash, backs up their transcripts and memory to
move them to another machine, and follows a session's directory when
it moves.

Only sessions with a name count (`/rename` in a session, or
`claude --name`); unnamed ones are left alone.

## Requirements

- Linux (it reads `/proc`)
- bash 4.4 or later, jq, tmux
- for `backup` and `restore`: gpg, tar and zstd

It reads the files Claude Code keeps for each running session in
`~/.claude/sessions/`, which are not a documented interface, so a
Claude Code update can break it. It was written against Claude Code
in November 2026.

## Install

Put `claude-tmux-sessions` somewhere on your `PATH`, and have Claude
Code run it after every turn with a Stop hook in
`~/.claude/settings.json`, so the snapshot stays current and each
tmux window is named after its session:

```json
{
  "hooks": {
    "Stop": [
      {
        "hooks": [
          {
            "type": "command",
            "command": "claude-tmux-sessions sync >/dev/null 2>&1 || true",
            "timeout": 10
          }
        ]
      }
    ]
  }
}
```

## Use

| Command | What it does |
|---|---|
| `save` | record the live named sessions and their window order |
| `sync` | `save`, then `rename` (what the Stop hook runs) |
| `list` | show the saved snapshot |
| `rename` | name the tmux windows after the sessions in them |
| `resume [-n]` | start each saved session that isn't running, with `claude --resume`, in a new window, in the saved order |
| `backup <dir>` | write the snapshots, transcripts and memory to one file in `<dir>`, encrypted with a passphrase |
| `restore <file>` | put a backup's files in place and list the directories missing on this machine |
| `move [-n] <old dir> <new dir>` | after moving a directory, move its sessions' transcripts and memory along |

`-n` (or `--dry-run`) shows what would happen.

### After a reboot or crash

Start tmux and run `claude-tmux-sessions resume`. A save that finds no
live sessions keeps the old snapshot, so the Stop hook firing early
doesn't wipe it. `resume` refuses to start anything while a saved
session's directory is missing.

### Moving to another machine

```sh
claude-tmux-sessions backup ~/Sync/backups     # gpg asks for a passphrase
# on the other machine:
claude-tmux-sessions restore ~/Sync/backups/claude-backup-<host>-<time>.tar.zst.gpg
claude-tmux-sessions resume -n
claude-tmux-sessions resume
```

The backup holds `~/.claude/projects` (every session's transcripts
and memory) and the snapshots. `restore` keeps a local file that is
newer than the backup's, and exits 3 if a saved session's directory
doesn't exist yet: clone or create it, then resume. Paths are kept
as they are, so both machines need the same home directory.

To keep some sessions out of backups (work under embargo, a client's
code), list their directories in `CLAUDE_BACKUP_EXCLUDE`, separated
by `:`. Sessions started in those directories or under them are left
out, and so is any directory whose name begins with one of them
followed by `-` (`~/src/private-old` for `~/src/private`), since
Claude Code's folder names don't tell `/` and `-` apart.

To test a backup without touching your own files, restore it into a
throwaway home and compare:

```sh
t=$(mktemp -d)
HOME=$t GNUPGHOME=~/.gnupg XDG_STATE_HOME= claude-tmux-sessions restore <file>
diff -rq ~/.claude/projects "$t/.claude/projects"
```

### Moving a directory

Claude Code files a session's transcripts and memory under
`~/.claude/projects/` by the directory it started in, so after a
checkout moves, `claude --resume` no longer finds them. Exit the
sessions, move the directory, then run
`claude-tmux-sessions move <old dir> <new dir>`. It moves the
project folders of the directory and of the directories under it,
merges into folders that already exist, rewrites the working
directory recorded in the transcripts, updates the snapshots, and
runs `git worktree repair` for a checkout with linked worktrees.
It refuses while a session of either directory is running.

## Containers

Sessions running in different [distrobox](https://distrobox.it/)
containers are saved to separate snapshots, by `$CONTAINER_ID`
(`host` outside a container), because each needs its own `claude`
and tmux to start again. Run `resume` and `restore` inside each
container. Without containers there is a single snapshot.

## Configuration

| Variable | Meaning |
|---|---|
| `CLAUDE_BACKUP_EXCLUDE` | directories, separated by `:`, whose sessions `backup` leaves out |
| `CLAUDE_TMUX` | the tmux program to use, overriding the one recorded for each session |
| `CLAUDE_TMUX_SOCKET` | the tmux server name (`tmux -L`) for sessions saved outside tmux; default `claude` |

Without `CLAUDE_TMUX`, a session goes back to the tmux program that
ran its server when it was saved; for older snapshots, `opentmux` is
used if installed, otherwise `tmux`.

Snapshots are kept in `${XDG_STATE_HOME:-~/.local/state}/claude-tmux-sessions/`.

## License

MIT; see [LICENSE](LICENSE).
