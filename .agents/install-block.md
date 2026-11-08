# The canonical install block

One install story, one wording. `README.md`, `.changeset/*`, and every page under `docs/` must say **this** and nothing else. Change it here first, then propagate.

## Clone this repo, run scripts/link-skills.sh

This repo's only user is its owner, and every machine that runs the skills keeps a clone of it. Install is a matter of: clone the repo somewhere stable and leave it there, then run the repo's own script, which symlinks every skill outside `misc/` and `deprecated/` into the harness skill directories: `~/.claude/skills` (Claude Code) and `~/.agents/skills` (Codex and other cross-agent readers). `in-progress/` rides along on purpose: this daily-driver install is exactly where its feedback loop runs.

<canonical-block name="clone-and-link">

```bash
git clone https://github.com/agendaga/pocock-skills
cd pocock-skills
bash scripts/link-skills.sh
```

</canonical-block>

The links point at the clone's absolute path, so the clone has to stay where it was cloned: move or delete it and every skill dangles. Put it wherever your working checkouts live.

## Updating

`git pull --ff-only` is the whole update story: the links are live, so anything pulled is installed the moment the clone updates. Re-run the script only after adding, removing, or renaming a skill, so the directory entries match.

## Machines without the clone: copy instead

Where a symlink does not fit (a harness that does not resolve one, Windows, a machine the clone will not live on), copy the promoted set instead. A copy is standalone: it carries no connection to the clone, and its update story is re-running the same command after a pull, with deletion done by hand.

<canonical-block name="copy-fallback">

```bash
mkdir -p ~/.claude/skills
for s in skills/engineering/*/ skills/productivity/*/; do cp -R "$s" ~/.claude/skills/; done   # or ~/.agents/skills/
```

</canonical-block>

The loop, not a plain `cp -R skills/*/*`, is deliberate: a bare glob collides on both buckets' root `README.md` and drags those repo-organisation pages in, where they are not skills. Where only one skill is wanted, copy only that one:

<canonical-block name="one-skill-copy">

```bash
cp -R skills/engineering/<name> ~/.claude/skills/
```

</canonical-block>

## One source only

Installed skills are either the clone's symlinks or owned copies. Never mix either with a subscription to the same skills from upstream, npm (`npx skills`), or a plugin marketplace: every skill arrives twice, and the subscription holds the upstream set, which is not what this fork ships. Remove any such subscription before installing.
