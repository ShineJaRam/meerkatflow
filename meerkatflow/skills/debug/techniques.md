# Debug techniques

## 1. Backward value tracing (deep call chains)

Where the symptom shows is usually not the cause. Follow the bad value back to its source.

1. Find the first line where the value is wrong.
2. Find who passed it (Grep for callers; a debugger or the framework's devtools for runtime values).
3. Was it correct at the caller? Yes → cause is in between. No → back to 2.
4. If you cannot follow by hand, instrument just before the suspect:
   ```ts
   console.log('[dbg] <spot>', JSON.stringify(value), new Error().stack)
   ```
   `new Error().stack` shows the caller. Remove every `[dbg]` when done.
5. Fix at the source. Blocking at the symptom lets it leak through another path.

## 2. Intermittent repro (breaks only sometimes)

Mostly timing. Never add a guessed `setTimeout`.

- **Suspect in order:** response arrival order (fast request returns late) → reactive/effect scheduling → before/after initialization (mount, connect, warm-up) → leftover cache/global state → two writers on the same rows (isolation level, lock order) → retries and duplicate delivery.
- **Force the repro:**
  - Network race: throttle the network, or delay only the suspect request (an interceptor, a proxy, a sleep in a test double).
  - Order dependence: repeat the action fast (tab switching, filter spamming, acting right after reload/restart).
  - Leftover state: fresh process/tab vs a long-running one, right after login vs later.
- **When fixing, wait for the condition, not for time:**
  - Work after a render/update → the framework's post-update hook (e.g. React `useEffect`)
  - Stale late response overwrites newer result → compare a request id, or cancel the older request (`AbortController`)
  - Two writers race → a constraint, lock or idempotency key at the storage layer, not check-then-write in code
  - Run once a value is ready → subscribe to it with a readiness check, not a delay
  - An arbitrary sleep (`setTimeout(fn, 300)`) is not a fix. It breaks again on a slower machine.

## 3. Defense in depth (after fixing the source)

Source fix first. Then guard only the boundaries where the same kind of value could leak again.

- Where external data becomes internal models (mapper/adapter/parser): empty, null, type mismatch
- Where user input goes to storage/requests
- Do not sprinkle null checks over internal functions. It hides causes.

## 4. When did it break (find the culprit change)

For "it used to work", bisect commits. Use a separate worktree so the working one is untouched.

```bash
M=$(git worktree list --porcelain | sed -n '1s/^worktree //p'); T="$(dirname "$M")/bisect-tmp"
git -C "$M" worktree add --detach "$T" <end of suspect range>
cd "$T" && git bisect start <bad commit> <good commit>
# each step: run the repro → git bisect good | bad
git bisect reset; git -C "$M" worktree remove "$T"
```

If dependencies must be installed there, follow step 4 of "Creating the worktree" in the spec skill.

## 5. Polluting test

Passes alone, fails together → an earlier test left global state behind. Run the failing file alone, then together with a suspect, serially. Example with vitest (use your runner's serial and ordering flags):

```bash
node_modules/.bin/vitest run <failing file>                     # confirm it passes alone
node_modules/.bin/vitest run <suspect file> <failing file> --no-file-parallelism
```

Halve the suspect set to find the culprit. Tests that fail alone at HEAD are out of scope for this procedure.
