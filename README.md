# HiveLogic Monitor

Download: https://github.com/csk5369/hivelogic-monitor/releases/latest/download/HiveLogic-Monitor-Setup.exe

This repo holds the **built installer**, not the source. The source lives in
[`csk5369/hivelogic-live`](https://github.com/csk5369/hivelogic-live) under
`hivelogic-monitor-agent/`.

## Shipping a new version

1. Bump `version` in `hivelogic-monitor-agent/package.json` (in `hivelogic-live`)
   and merge that. Skipping this means no machine ever downloads the build --
   electron-updater compares versions and a repeat is not an update.
2. On a **Windows** machine, from `hivelogic-live/hivelogic-monitor-agent`:

   ```
   npm install
   npm run build:win
   ```

3. Copy the three files electron-builder puts in `dist/` into **this** repo and
   push to `main`:

   - `HiveLogic-Monitor-Setup.exe`
   - `HiveLogic-Monitor-Setup.exe.blockmap`
   - `latest.yml`

   Commit all three together. `latest.yml` records the installer's size and
   hash, so a mismatched pair breaks the update on every user's machine.

That's it. `.github/workflows/release.yml` reads the version out of
`latest.yml`, creates the release, and then checks that it is published (not a
draft) with both required assets attached. Nothing in the workflow needs
editing per release.

Agents pick it up within 30 minutes -- they check at startup and on a 30-minute
timer, download in the background, then prompt to install.

## Checking who actually updated

From HiveLogic's Supabase (the agent reports its version on every heartbeat as
of 1.2.4):

```sql
select p.email, a.device_name, a.agent_version, a.last_seen_at
  from monitor_agents a
  join profiles p on p.id = a.employee_id
 where a.status = 'active'
 order by a.last_seen_at desc;
```

A version older than the current release means that machine has not updated.
`null` means an agent from before 1.2.4, which never reported one.

## History

Until 2026-08-17 the workflow had `v1.2.3` hardcoded in both its existence
check and its create command, with `&& exit 0` in between. Once v1.2.3 existed
it could only ever succeed at doing nothing: a newer installer would be checked
out, the release found present, and the run reported green with no release
created and no error logged. That silently blocked the 1.2.4 rollout while the
server had already begun enforcing its half of the same change.
