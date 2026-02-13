---
name: cli-design
description: Review and improve CLI program design using principles from clig.dev — the Command Line Interface Guidelines. Covers help text, output, errors, arguments/flags, subcommands, interactivity, configuration, naming, and distribution. Use when designing a new CLI, reviewing CLI UX, or fixing how a command-line tool communicates with users.
---

# CLI Design Guidelines

Condensed from [clig.dev](https://clig.dev) — an open-source guide to writing better command-line programs, updating UNIX principles for the modern day.

When reviewing or building a CLI, work through each section below as a checklist. The examples are as important as the rules — they show what good looks like.

## Core Philosophy

1. **Human-first design** — If a command is used primarily by humans, design for humans first. Shed the baggage of machine-first UNIX conventions where they hurt usability.
2. **Simple parts that work together** — Composability matters. Respect stdin/stdout/stderr, exit codes, signals, and line-based text. Use JSON when structure is needed. Your software will become a part in a larger system — your only choice is whether it will be a well-behaved part.
3. **Consistency across programs** — Follow existing CLI conventions. Users have muscle memory. Break convention only with care and good reason. When convention would compromise usability, it might be time to break with it — but make the decision deliberately.
4. **Say just enough** — A command hanging silently for minutes is as bad as dumping pages of debug output. Both leave the user confused. Clarity over volume.
5. **Ease of discovery** — Comprehensive help, lots of examples, suggest next commands, suggest fixes on errors. Steal ideas from GUIs. Remember-and-type and see-and-point are not mutually exclusive.
6. **Conversation as the norm** — CLI interaction is conversational. Users learn through trial-and-error, multi-step workflows (multiple `git add`s then `git commit`), exploration (`cd` and `ls` to understand a directory), and dry-runs before real runs. Guide users through the conversation — suggest corrections, show intermediate state, confirm before scary actions. At best, it's a pleasant exchange that speeds them on their way. At worst, it's hostile and makes them feel stupid.
7. **Robustness** — Both objective (handle unexpected input gracefully, be idempotent) and subjective (feel solid, not flimsy). Keep users informed, explain common errors, don't print scary stack traces. Simplicity breeds robustness.
8. **Empathy** — Give users the feeling you're on their side, that you want them to succeed, that you've thought carefully about their problems. Delight means exceeding expectations at every turn.
9. **Chaos** — The terminal is a mess. Inconsistencies are everywhere. Yet this chaos has been a source of power — few constraints means all manner of invention. Sometimes you should break the rules. Do so with intention and clarity of purpose. *"Abandon a standard when it is demonstrably harmful to productivity or user satisfaction."* — Jef Raskin

## The Basics

- Use a command-line argument parsing library. Don't hand-roll flag parsing. Good ones: Commander.js (Node), Click/Typer (Python), Cobra (Go), clap (Rust), picocli (Java).
- **Exit code 0** on success, **non-zero on failure**. Map non-zero codes to the most important failure modes.
- **stdout** for primary output and anything machine-readable (this is where piping sends things).
- **stderr** for log messages, errors, and diagnostics. When commands are piped together, stderr is displayed to the user, not fed into the next command.

## Help

- Display help on `-h` and `--help`. For git-like tools, also `myapp help` and `myapp help subcommand`.
- **No-args behavior:** If the command requires arguments and gets none, display concise help — not full help. Include only:
  - A description of what the program does
  - One or two example invocations
  - Key flags (unless there are too many)
  - A pointer to `--help` for more

  `jq` does this well:
  ```
  $ jq
  jq - commandline JSON processor [version 1.6]

  Usage:  jq [options] <jq filter> [file...]

  jq is a tool for processing JSON inputs, applying the given filter
  to its JSON text inputs and producing the filter's results as JSON
  on standard output.

  Example:
      $ echo '{"foo": 0}' | jq .
      {
          "foo": 0
      }

  For a listing of options, use jq --help.
  ```

- Show **full help** on `--help`. Ignore any other flags/args when `-h`/`--help` is passed — you should be able to add `-h` to the end of anything and get help. Don't overload `-h`.
- **Lead with examples.** Users prefer examples over abstract descriptions. Show common uses first, with actual output if it helps. Tell a story with a series of examples, building toward complex uses. Put exhaustive examples in a cheat sheet command or web page.
- Display the **most common flags and commands first** in help text, not alphabetically. Git does this — it groups commands by workflow stage ("start a working area", "work on the current change", "examine the history and state").
- Use **formatting** (bold headings) for scannability, but in a terminal-independent way. When piped through a pager, emit no escape characters.

  Heroku's help is a good model:
  ```
  $ heroku apps --help
  list your apps

  USAGE
    $ heroku apps

  OPTIONS
    -A, --all          include apps in all teams
    -p, --personal     list apps in personal account when a default team is set
    -s, --space=space  filter by space
    -t, --team=team    team to use
    --json             output in json format

  EXAMPLES
    $ heroku apps
    === My Apps
    example
    example2

  COMMANDS
    apps:create     creates a new app
    apps:destroy    permanently destroy an app
    apps:info       show detailed app information
  ```

- **Suggest corrections** when the user mistyped a command. Ask, don't auto-execute — invalid input might be a logical mistake, not a typo. And if you auto-correct silently, you're committing to supporting that syntax forever.
  ```
  $ heroku pss
   ›   Warning: pss is not a heroku command.
  Did you mean ps? [y/n]:
  ```
- Provide a **support path** (website/GitHub link) in top-level help.
- Link to **web docs** from help text. Deep-link to relevant subcommand pages.
- If the command expects piped input and stdin is a TTY, **show help immediately** instead of hanging like `cat` does.

## Documentation

Help text gives a brief, immediate sense of what the tool does. Documentation is the full detail — what it's for, what it isn't for, how everything works.

- **Provide web-based documentation.** People need to search for it online and link others to specific parts.
- **Provide terminal-based documentation.** Fast to access, stays in sync with the installed version, works offline.
- **Consider man pages.** Many users reflexively try `man mycmd`. Use tools like `ronn` to generate them from Markdown. Also make terminal docs accessible via the tool itself (e.g., `npm help ls` is equivalent to `man npm-ls`).

## Output

- **Human-readable output is paramount.** Detect TTY to decide formatting. Humans first, machines second.
- **Have machine-readable output where it doesn't impact usability.** Users should be able to pipe output to `grep` and have it work as expected. *"Expect the output of every program to become the input to another, as yet unknown, program."* — Doug McIlroy
- Support **`--plain`** for plain, tabular text output (one record per line, no wrapping/splitting). Use this when human-readable formatting breaks machine-readable output.
- Support **`--json`** for structured output. JSON integrates with `jq` and web services via `curl`.
- **Display output on success, but keep it brief.** Printing nothing is rarely the best default (it makes commands look broken), but err on the side of less. Offer `-q`/`--quiet` for scripts to suppress non-essential output.
- **If you change state, tell the user.** Explain what just happened so they can model the system in their head. `git push` is the gold standard:
  ```
  $ git push
  Enumerating objects: 18, done.
  Counting objects: 100% (18/18), done.
  Writing objects: 100% (10/10), 2.09 KiB | 2.09 MiB/s, done.
  To github.com:replicate/replicate.git
   + 6c22c90...a2a5217 bfirsh/fix-delete -> bfirsh/fix-delete
  ```
- **Make current state easy to see.** `git status` is the model — it shows state *and* hints at next actions:
  ```
  $ git status
  On branch bfirsh/fix-delete
  Your branch is up to date with 'origin/bfirsh/fix-delete'.

  Changes not staged for commit:
    (use "git add <file>..." to update what will be committed)
    (use "git restore <file>..." to discard changes in working directory)
      modified:   cli/pkg/cli/rm.go

  no changes added to commit (use "git add" and/or "git commit -a")
  ```
- **Suggest next commands** when commands form a workflow. This is how users discover functionality.
- Actions crossing program boundaries (reading/writing files not passed as args, network requests) should be **explicit**.
- **Increase information density with ASCII art.** `ls -l` permissions are a masterclass — scannable at a glance, more patterns emerge as you learn:
  ```
  -rw-r--r-- 1 root root     68 Aug 22 23:20 resolv.conf
  lrwxrwxrwx 1 root root     13 Mar 14 20:24 rmt -> /usr/sbin/rmt
  drwxr-xr-x 4 root root   4.0K Jul 20 14:51 security
  ```
- Use **color with intention.** Highlight important info, red for errors. Don't overuse — if everything is colored, color means nothing.
- **Disable color** when:
  - stdout/stderr is not a TTY (check individually — colors on stderr are still useful when piping stdout)
  - `NO_COLOR` env var is set (non-empty)
  - `TERM=dumb`
  - `--no-color` is passed
  - Consider also supporting a `MYAPP_NO_COLOR` env var
- **No animations** when stdout isn't a TTY. Prevents progress bars becoming Christmas trees in CI logs.
- Use **symbols and emoji** where they add clarity or structure, not as decoration. `yubikey-agent` uses them well to break up a wall of text:
  ```
  $ yubikey-agent -setup
  🔐 The PIN is up to 8 numbers, letters, or symbols. Not just numbers!
  ❌ The key will be lost if the PIN and PUK are locked after 3 incorrect tries.

  Choose a new PIN/PUK:
  Repeat the PIN/PUK:

  🧪 Reticulating splines …

  ✅ Done! This YubiKey is secured and ready to go.
  🤏 When the YubiKey blinks, touch it to authorize the login.
  ```
- Don't show **internal debug info** by default. If output only helps the developer, it shouldn't be shown to users — verbose mode only.
- Don't treat **stderr like a log file** — no `ERR`, `WARN` labels or extraneous context unless in verbose mode.
- Use a **pager** (`less -FIRX`) for long output, only when stdout is a TTY. `-F` doesn't page if content fits one screen, `-I` ignores case in search, `-R` enables color, `-X` leaves content on screen when quitting.

## Errors

Errors are the most common reason users consult documentation. If you make errors *into* documentation, you save them enormous time.

- **Catch errors and rewrite them for humans.** Think of it as a conversation guiding the user in the right direction:
  `"Can't write to file.txt. You might need to make it writable by running 'chmod +w file.txt'."`
- **Signal-to-noise ratio is crucial.** Group similar errors under a single explanatory header. The more irrelevant output, the longer it takes users to find what went wrong.
- **Put the most important information at the end** of output — that's where the eye lands. Use red sparingly and intentionally (it draws the eye).
- For unexpected errors, provide **debug info and bug-report instructions** — but consider writing debug logs to a file instead of printing to the terminal. Don't overwhelm users with info they don't understand.
- **Make bug reports effortless.** Provide a URL that pre-populates as much info as possible.

## Arguments and Flags

**Terminology:** *Arguments* (args) are positional parameters — order matters (`cp foo bar` ≠ `cp bar foo`). *Flags* are named parameters (`-r`, `--recursive`) — order generally doesn't matter.

- **Prefer flags to args.** Flags are self-documenting and easier to evolve. With args, it's sometimes impossible to add new input without breaking existing behavior.
- **Full-length versions of all flags.** Both `-h` and `--help`. Long forms are self-documenting in scripts.
- **Single-letter flags only for commonly used flags.** Don't pollute the short-flag namespace — you'll need letters for flags you add later.
- **Multiple args are fine for the same kind of thing** (e.g., `rm file1 file2`, works with globbing: `rm *.txt`). Two args for *different* things is usually wrong — exception: common primary actions like `cp <src> <dest>`.
- **Use standard flag names** when a standard exists:

  | Flag | Meaning | Examples |
  |------|---------|----------|
  | `-a, --all` | All | `ps`, `fetchmail` |
  | `-d, --debug` | Debug output | |
  | `-f, --force` | Force / skip confirmation | `rm -f` |
  | `--json` | JSON output | |
  | `-h, --help` | Help (only ever help) | |
  | `-n, --dry-run` | Show what would happen | `rsync`, `git add` |
  | `--no-input` | Disable all prompts | |
  | `-o, --output` | Output file | `sort`, `gcc` |
  | `-p, --port` | Port | `psql`, `ssh` |
  | `-q, --quiet` | Less output | |
  | `-u, --user` | User | `ps`, `ssh` |
  | `--version` | Version | |
  | `-v` | Ambiguous (verbose or version) — prefer `-d` for verbose | |

- **Make the default right for most users.** If a good UX is behind a flag, most users will never find it. If `ls` were designed today, it would probably default to `ls -lhF`.
- **Prompt for missing input** when interactive. But **never require a prompt** — always allow flags/args instead. Skip prompts entirely if stdin isn't a TTY.
- **Confirm before dangerous actions** — three levels:
  - *Mild* (deleting a file): Optional prompt. If the command is already named "delete," you probably don't need to ask.
  - *Moderate* (deleting a directory, remote resource, complex bulk modification): Prompt for confirmation. Offer `--dry-run`.
  - *Severe* (deleting an entire app/server): Require typing something non-trivial (the resource name). Support `--confirm="name"` for scriptability.
  - Watch for **non-obvious destruction**: changing a config value from 10 to 1 might implicitly delete 9 things. Treat this as severe.
- **Support `-`** to read from stdin / write to stdout when input/output is a file. Example: `curl https://example.com/file.tar.gz | tar xvf -`
- For optional flag values, use a special word like **`none`** — not blank values (ambiguous).
- **Make args, flags, and subcommands order-independent** where possible. Users constantly hit up-arrow, add a flag at the end, and re-run. Don't surprise them.
- **Never read secrets from flags** — they leak into `ps` output and shell history. Accept via files (`--password-file`), stdin, or secret management. `--password $(< file.txt)` has the same leakage problems.

## Interactivity

- **Only prompt if stdin is a TTY.** In pipes/scripts, throw an error telling the user what flag to pass.
- **`--no-input`** disables all prompts explicitly. If the command requires input, fail and explain which flags to use.
- **Don't echo passwords** as the user types (turn off terminal echo).
- **Let the user escape.** Make it clear how to quit. Ctrl-C should always work. For wrapper programs (SSH, tmux, telnet), document the escape mechanism clearly. SSH uses `~` escape sequences.

## Subcommands

Use subcommands to reduce complexity of a sufficiently complex tool, or to combine closely related tools into one (RCS → Git). They're useful for sharing global flags, help text, configuration, and storage.

- **Be consistent across subcommands.** Same flag names for the same things, similar output formatting.
- **Consistent naming across levels.** For `noun verb` patterns (e.g., `docker container create`), use the same verbs across different nouns. Either `noun verb` or `verb noun` works, but `noun verb` is more common.
- **Don't have ambiguous or similarly-named commands.** "update" vs. "upgrade" is confusing — use different words or disambiguate.

## Robustness

- **Validate user input early.** Check and bail before anything bad happens, with understandable errors.
- **Responsive > fast.** Print something within 100ms. If you're making a network request, print something before you do it so it doesn't look broken.
- **Show progress** for long operations. Spinners, progress bars, estimated time remaining. An animated component reassures the user you haven't crashed. `docker pull` is the model:
  ```
  $ docker image pull ruby
  latest: Pulling from library/ruby
  6c33745f49b4: Pull complete
  ef072fc32a84: Extracting [===============>        ]  7.5MB/7.8MB
  f2ecc74db11a: Downloading [=========>              ]  89MB/192MB
  b0efebc74f25: Downloading [==================>     ]  19MB/22MB
  ```
  When things go well, hide logs behind progress bars. But if there's an error, print the logs — otherwise debugging is impossible.
- **Parallelize** where possible, but keep output clean and non-interleaved. Use libraries — this is code you don't want to write yourself.
- **Set timeouts** on network operations. Make them configurable with sane defaults. Don't hang forever.
- **Make it recoverable.** If it fails transiently (network went down), re-running should pick up where it left off.
- **Make it crash-only.** If you can avoid cleanup after operations (or defer it to next run), the program can exit immediately on failure. This makes it both more robust and more responsive.
- **Expect misuse.** Scripts will wrap it, bad connections will interrupt it, many instances will run concurrently, unexpected environments will surprise you. (macOS filesystems are case-insensitive but case-preserving.)

## Future-proofing

Subcommands, arguments, flags, config files, env vars — these are all interfaces. You're committing to keeping them working.

- **Keep changes additive.** Add new flags rather than changing existing ones incompatibly.
- **Warn before breaking changes.** Deprecate with in-program warnings. Show migration path. Detect when users have already migrated and stop showing the warning.
- **Human output can change.** Encourage `--plain` or `--json` in scripts to keep output stable.
- **No catch-all subcommand.** Don't assume the user means `run` when the first arg isn't a known subcommand. You can never add a subcommand with that name without breaking existing scripts.
- **No arbitrary abbreviations.** Don't let `mycmd i` implicitly mean `install` — you can never add another command starting with `i`. Explicit, stable aliases are fine.
- **No time bombs.** Will your command still work in 20 years? The server most likely to not exist in 20 years is the one you maintain right now.

## Signals

- **Ctrl-C (SIGINT):** Exit ASAP. Print something immediately, before cleanup. Timeout cleanup code so it can't hang forever.
- **Second Ctrl-C:** Skip long cleanup. Tell the user what the next Ctrl-C will do if it's destructive.
  ```
  $ docker-compose up
  …
  ^CGracefully stopping... (press Ctrl+C again to force)
  ```
- **Expect unclean starts.** The program may start without prior cleanup having run.

## Configuration

Configuration tiers — match the mechanism to the tier:

| Tier | Examples | Mechanism |
|------|----------|-----------|
| Changes every invocation | Debug level, dry-run | Flags (+ maybe env vars) |
| Stable per-user, varies per-project | HTTP proxy, color settings, paths | Flags + env vars + `.env` |
| Stable per-project, all users | Build config, docker-compose, Makefile | Version-controlled config file |

- **Precedence** (highest to lowest): flags → shell env vars → project config (`.env`) → user config → system config.
- **Follow XDG Base Directory spec** for config file locations (`~/.config/myapp/`). Supported by yarn, fish, neovim, tmux, and many others.
- **Ask consent** before modifying config you don't own. Prefer creating new config files (`/etc/cron.d/myapp`) over appending to existing ones (`/etc/crontab`). If you must append, use a dated comment to delineate your additions.

## Environment Variables

- Names: **uppercase letters, numbers, underscores only.** No leading numbers.
- Aim for **single-line values.** Multi-line values cause usability issues with `env`.
- **Don't commandeer common names.** Check the POSIX standard env var list.
- **Respect standard env vars:**

  | Var | Purpose |
  |-----|---------|
  | `NO_COLOR` / `FORCE_COLOR` | Disable/force color |
  | `DEBUG` | More verbose output |
  | `EDITOR` | User's preferred editor |
  | `HTTP_PROXY`, `HTTPS_PROXY`, `ALL_PROXY`, `NO_PROXY` | Network proxy |
  | `SHELL` | User's preferred shell (for interactive sessions; use `/bin/sh` for scripts) |
  | `TERM`, `TERMINFO`, `TERMCAP` | Terminal capabilities |
  | `TMPDIR` | Temporary file location |
  | `HOME` | Config file location |
  | `PAGER` | Preferred pager for long output |
  | `LINES`, `COLUMNS` | Screen size |

- **Read `.env` files** where appropriate for project-specific config. Many languages have libraries (Rust, Node, Ruby).
- **Don't use `.env` as a substitute for proper config files.** Limitations: not in source control (no history), string-only, poorly organized, encoding issues, often contains secrets that should be stored more securely.
- **Never read secrets from env vars.** They leak everywhere: child processes, logs, `docker inspect`, `systemctl show`, shell substitutions in `ps`. Accept secrets via credential files, pipes, AF_UNIX sockets, or secret management services.

## Naming

- **Simple, memorable word.** Not too generic (both ImageMagick and Windows used `convert`), not too long.
- **Lowercase only**, dashes if needed. `curl` yes, `DownloadURL` no.
- **Short but not too short.** Very short names (`cd`, `ls`, `ps`) are reserved for ubiquitous tools.
- **Easy to type.** Consider hand ergonomics. Docker Compose was renamed from `plum` to `fig` because `plum` was an awkward one-handed hopscotch on the keyboard.

## Distribution

- **Distribute as a single binary** when possible. Use PyInstaller, pkg, or similar if your language doesn't compile to binaries. If you can't do a single binary, use the platform's native package installer. Tread lightly on the user's computer.
- Language-specific tools (linters, formatters) can assume the interpreter is installed.
- **Make it easy to uninstall.** Put uninstall instructions at the bottom of install instructions — one of the most common times people want to uninstall is right after installing.

## Analytics

- **Never phone home without consent.** Be explicit about what you collect, why, how it's anonymized, and retention period. Users will find out, and they will be angry.
- Prefer **opt-in**. If opt-out, clearly tell users on first run and make disabling easy.
- **Consider alternatives:** instrument web docs, instrument downloads, talk to users directly. Encourage feedback and feature requests in your docs and repos.

---

## Before You Ship — Quick Checklist

**Fundamentals**
- [ ] Exit 0 on success, non-zero on failure
- [ ] Primary output → stdout, messages/errors → stderr
- [ ] `-h` and `--help` work (and on every subcommand)
- [ ] No args → concise help with examples, not an error or a hang

**Help & Discovery**
- [ ] Help leads with examples, not abstract descriptions
- [ ] Most common flags/commands listed first
- [ ] Typos get a "did you mean?" suggestion
- [ ] Support path (URL/GitHub) in top-level help

**Output & Errors**
- [ ] TTY detection drives formatting decisions (color, animations, pager)
- [ ] Color disabled when `NO_COLOR` set, `TERM=dumb`, not a TTY, or `--no-color`
- [ ] State changes explained to the user
- [ ] Errors are human-readable with actionable suggestions
- [ ] `--json` and/or `--plain` available for scripts
- [ ] `-q`/`--quiet` suppresses non-essential output

**Flags & Input**
- [ ] Every flag has a `--long-form`
- [ ] Short flags reserved for common operations only
- [ ] Standard flag names used where applicable (`--force`, `--dry-run`, `--quiet`, etc.)
- [ ] Secrets never accepted via flags or env vars — use files, stdin, or secret managers
- [ ] Prompts only when stdin is a TTY; `--no-input` disables all prompts
- [ ] Dangerous actions require confirmation (severity-appropriate)

**Robustness**
- [ ] Something prints within 100ms — no silent hangs
- [ ] Long operations show progress (spinner, bar, ETA)
- [ ] Network operations have configurable timeouts
- [ ] Re-running after transient failure picks up where it left off
- [ ] Ctrl-C exits immediately; second Ctrl-C skips cleanup

**Future-proofing**
- [ ] No catch-all subcommand
- [ ] No implicit abbreviations of subcommands
- [ ] Changes are additive; breaking changes are warned in advance
- [ ] No dependency on external servers that may disappear
