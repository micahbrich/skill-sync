# skill-sync

Keep every skills repo cloned under `~/skills` installed into every coding agent on this machine, and rescue skills that a tool created outside any repo.

One clone per repo, on every machine that should have it. Which repos a machine clones is the only configuration.

## Setup

```sh
git clone https://github.com/micahbrich/skill-sync.git ~/skills/skill-sync
~/skills/skill-sync/bin/skill-sync clone micahbrich/internal-skills micahbrich/skills
~/skills/skill-sync/bin/skill-sync install   # Claude Code SessionStart hook
~/skills/skill-sync/bin/skill-sync
```

Add a company repo on the machines that should have it, and nowhere else:

```sh
~/skills/skill-sync/bin/skill-sync clone micahbrich/every-skills
```

## What a run does

1. `git pull --ff-only` in each repo under `~/skills`.
2. Adopt strays. A real directory with a `SKILL.md` in an agent's skills folder that the skills CLI lock file does not know about was made by hand. It moves into the inbox repo's `skills/inbox/`, is committed as "Adopt skills from <host>", and pushed.
3. `npx skills add <repo> --skill '*' -g` for each repo, which links its skills into every detected agent. Lines in a repo's `third-party.txt` install the same way. `npx skills update -g` refreshes third-party skills.
4. Link the machine profile: the first `profile.md` from a repo that is not the inbox, else the inbox's, to `~/.agents/profile.md`.

Every step converges. Running twice changes nothing the second time.

## Repo conventions

- Skills live in `skills/<name>/SKILL.md`.
- An empty `.skill-sync-inbox` file marks the one repo that receives adopted skills.
- `profile.md` holds machine-specific context that shared skills read from `~/.agents/profile.md`.
- `third-party.txt` lists `owner/repo` lines to install alongside.

## Filing adopted skills

`skill-sync hook claude` prints a note when `skills/inbox/` is not empty. Claude Code feeds SessionStart hook output to the session, so the agent you are already talking to reads each repo's README and moves the skill with `git mv`. `skill-sync hook codex` prints the same note in Codex's JSON shape.

## Commands

| Command | Does |
| --- | --- |
| `skill-sync` | pull, adopt, install, link profile |
| `skill-sync status` | print what a run would do, change nothing |
| `skill-sync hook claude\|codex` | print the inbox note, then run a sync in the background |
| `skill-sync clone owner/repo…` | clone into `~/skills` |
| `skill-sync install` | add the SessionStart hook to `~/.claude/settings.json` |

`SKILLS_HOME` overrides `~/skills`. Logs from background runs go to `~/.agents/skill-sync.log`.
