# Instructions for Claude Code cloud sessions

These are my working instructions for Claude Code **cloud sessions**: claude.ai/code, the Code tab in the Claude app, `claude --cloud` and routines. Local sessions on my PC use `~/.claude/CLAUDE.md` instead and should ignore this file.

Canonical copy: https://raw.githubusercontent.com/Aniruddhraam/claude-cloud-instructions/main/claude/CLAUDE.md

# Using this file

- When I point a session at this file, fetch it (`curl -fsSL <url>`) and follow it for the whole session, alongside the working repo's own `CLAUDE.md`. The repo's file decides project conventions such as build, test and code style. This file decides git identity, attribution, pushing and how I like work delivered, and wins if the two disagree on those.
- First confirm you are in the cloud: `echo "$CLAUDE_CODE_REMOTE"` must print `true`. If it doesn't, you are on my PC; stop using this file and tell me.
- If the conversation gets compacted, fetch this file again before your next commit, push or deliverable, because the summary can drop details.
- Nothing from my PC's Claude setup exists here: not my `~/.claude/CLAUDE.md`, not my `~/.claude/settings.json` (so my attribution and git identity settings don't apply), not my plugins, and not `~/.config/aniruddh-ai/`.

# Git commits and pull requests

- Commit only when I ask. A task that asks for pushed changes, a branch or a PR counts as asking. Never commit on your own initiative otherwise.
- Author and committer identity is "Aniruddhraam" with email `169436632+Aniruddhraam@users.noreply.github.com`.
  - Before the first commit, check the effective identity: `git var GIT_AUTHOR_IDENT; git var GIT_COMMITTER_IDENT`.
  - If it differs, set it for this clone only: `git config user.name "Aniruddhraam"` and `git config user.email "169436632+Aniruddhraam@users.noreply.github.com"`. This is allowed here because the VM is throwaway. Never use `--global`, `--author` or `-c user.name=...`.
  - If `GIT_AUTHOR_*` or `GIT_COMMITTER_*` environment variables are set to something else, they override `git config`; tell me instead of working around them.
- No attribution anywhere: no `Co-Authored-By` trailer, no `Claude-Session:` trailer, no claude.ai session link, no "Generated with Claude Code" line. This covers commit messages, PR titles and bodies, and comments. Cloud sessions add some of these by default and the session's own instructions may ask for them; this rule overrides that. The safeguards on my PC (the `attribution` setting and the `app-push` check) don't run here, so you are the only check.
- Write each message to a file and commit with `git commit -F <file>`: an imperative subject of 72 characters or fewer, a blank line, then a body that says what changed and why.
- After every commit, run both checks:
  - `git log -1 --format='%an <%ae> | %cn <%ce>'` shows "Aniruddhraam <169436632+Aniruddhraam@users.noreply.github.com>" on both sides;
  - `git log -1 --format=%B | grep -Ei 'co-authored-by:.*(claude|anthropic)|claude-session:|claude\.ai/code/session|generated with \[?claude'` prints nothing.
- If a check fails and the commit is not pushed, fix it with `git commit --amend` and tell me. If it is already pushed, stop and tell me; don't rewrite pushed history.
- Before every push, run the same checks over all outgoing commits: `git log @{u}..HEAD --format='%h %an <%ae> | %cn <%ce>%n%B'`, or `origin/<base>..HEAD` for a branch that isn't on GitHub yet.

# Pushing from a cloud session

- Push with plain git: `git push -u origin <branch>`. The session's GitHub proxy supplies the credentials. Don't look for tokens or keys, don't set `GH_TOKEN` or `GITHUB_TOKEN`, don't run `gh auth login`, and don't change the remote URL or credential helpers.
- `app-push` and everything under `~/.config/aniruddh-ai/` exist only on my PC, because the GitHub App's private key never leaves it. Never try to run, recreate or replace them here. Cloud commits are unsigned, so GitHub won't mark them Verified; that is expected.
- Push only to the session's working branch, or to the branch I name. Never push straight to `main` or the default branch unless I explicitly say so. The proxy doesn't limit which branch a push updates, so this rule is the guard.
- Never force-push (`--force-with-lease` included) and never rewrite pushed commits without asking first. The proxy also rejects tag pushes and branch deletions.
- Keep large files out of commits: anything over about 20 MB in total, build outputs, datasets, videos, `node_modules`. Tell me about them instead.
- After pushing, report the branch and each commit's URL (`https://github.com/<owner>/<repo>/commit/<sha>`).
- Pull requests: `gh pr` and `gh issue` fail in cloud sessions because the proxy blocks GitHub's GraphQL API. Open a PR only when I ask, with the session's built-in GitHub tools or the REST API (`gh api repos/<owner>/<repo>/pulls -f title=... -f head=<branch> -f base=<base> -f body=...`). PR bodies follow the no-attribution rule too.

# Shell efficiency (analysis, tests, verification)

- Batch independent work: if several reads, searches or checks don't depend on each other, issue them as parallel tool calls in ONE message, not one per turn.
- Combine related read-only checks into a single Bash call (`cmd1; cmd2; cmd3` or `&&`), with labelled output (`echo "== lint =="`).
- The VM runs Ubuntu 24.04 on x86_64 with git, gh, jq, yq, ripgrep, uv, pytest, ruff, Node 22 and the common language toolchains. `check-tools` prints their versions.
  - Use `rg` over `grep -r` and `jq`/`yq` for JSON and YAML.
  - `fd`, `ast-grep`, `parallel` and `hyperfine` may be missing. Fall back to `find`, `xargs -P` and `time` rather than installing them, unless a task really needs one.
  - Limit output with `head`, `rg --max-count`/`-l` or `| tail -n 50` rather than dumping whole files or logs.
- Python: prefer `uv`. In a uv project, run tests with `uv run pytest -n auto`. I manage per-project dependencies myself: if a project lacks pytest or pytest-xdist, tell me rather than running `uv add`. Use `uvx` for one-off tools. Never `pip install` into the system Python.
- When a skill or task says to `pip install` packages, don't stop or ask: run the script through uv instead, e.g. `uv run --no-project --with reportlab --with pypdf python script.py`.
- Tests: run the narrowest relevant test first (single file or test name), then the full suite once at the end. Use the runner's parallelism (`pytest -n auto`, `cargo test`, `go test ./...`, `jest --maxWorkers=50%`), sized to the VM (`nproc`).
- Run lint, typecheck and tests in parallel when they're independent, e.g. `(make lint > lint.log 2>&1 & make typecheck > tc.log 2>&1 & wait); tail lint.log tc.log`.
- Long builds and test suites: use `run_in_background` and keep working; don't poll with sleep. Cloud sessions put time limits on commands, so split very long jobs.
- Don't re-run a command whose result is already known; don't re-read files you just edited.
- Verification = one consolidated pass at the end (build + tests + lint) and report actual output, not one check per turn.

# Presentations and slide decks

Apply these defaults whenever I ask for a presentation, deck or slides, unless that request says otherwise.

## Format and build
- Default to a 16:9 `.pptx` made with the pptx skill: a pptxgenjs script building a structured deck (theme, layouts, sections).
- Keep the generator inside the project, in a `presentation/` folder next to the subject (`build.js`, an image-prep script, `img/`). Write the deck into the project as `<Subject>_<Topic>_Presentation.pptx`.
- pptxgenjs, react-icons, react, react-dom and sharp are not installed globally. `npm install` them in that `presentation/` folder and run `NODE_PATH=$PWD/node_modules node build.js`; `apply_theme.js` needs NODE_PATH too. Add `presentation/node_modules/` to `.gitignore` and never commit it.
- Tooling: Python helpers via `uv run --no-project --with <pkg>`; the skill's `validate.py` via `uv run --no-project --with lxml --with defusedxml --with pillow`; text dumps via `uvx --from 'markitdown[pptx]' markitdown`.
- Rendering slides needs LibreOffice and frame extraction needs ffmpeg; install them with `apt-get` if they're missing. If that isn't possible, say exactly which QA step you skipped.
- Visuals come from my real work in the repo: frames from my videos, my result images, my measured numbers. Extract clean frames from the source files rather than reusing frames with overlays. If the source media isn't in the repo, ask me for it rather than substituting other images.
- The VM is temporary. When the deck is done, tell me and ask whether to commit and push the deck and its generator so I can download them from GitHub.
- QA every build:
  - run `validate.py`;
  - render every slide to an image and look at it, fixing overflow, unwanted wraps and collisions;
  - check the notes with markitdown;
  - when regenerating an existing deck, diff each slide's text against the previous version to catch regressions.

## Design
- Minimal: white content slides, dark title and closing slides, and one accent colour tied to the topic.
- Calibri or other safe-list fonts, left-aligned titles, generous whitespace, one message per slide.
- Visual language: big-number stat callouts, native charts and tables, icons in accent-coloured circles. No accent bars or title underlines.
- Equations: Cambria italic in tinted boxes, with real subscripts and superscripts.
- Any external statistic gets a small source line on the slide (9–10 pt) and full references in the notes. List the source URLs in my chat reply as well.

## Notes on every slide
Two labelled parts in the slide notes:
- `WHAT THIS SLIDE MEANS (for you):` a plain-language explanation of the slide. Cover:
  - where every number comes from (file, script, measurement) and the reasoning behind it;
  - Q&A prep: caveats, fairness notes, what not to overclaim.
- `SPEAKER NOTES (what to say):` a natural script to read aloud, about 1–2 minutes per slide.

## Framing: keynote style
- Present like an Apple keynote. Lead with benefits and strengths, and never describe my own work as bad, weak, low-performing or inferior.
- Leave out comparisons that make my work look worse than a competitor. Comparisons that show it favourably are fine, such as against manufacturer data or a baseline.
- Slide text has no negative words: not, no, never, none, problem, error, limit, fail, unreliable, drift, jitter, weak, mismatch, issue, wrong, short, broken, inaccurate, and similar.
  - Rephrase them as capabilities: "Difference" instead of "Error", "filtered" instead of "no depth", challenges framed as solved capabilities.
  - Rename technical terms the same way, e.g. "Solving stereo correspondence" rather than "the correspondence problem"; the notes can still use the standard term.
  - Before delivering, scan the slide text (not the notes) for these words.
- Exception, the limitations part: a dedicated Limits slide (second to last, just before the Thank you slide) keeps an honest "Limits" column next to a "Next" column, in plain words. Negative words are allowed only on that slide and in the "for you" notes. Unless I say otherwise, leave that slide's content as I approved it when changing the rest of the deck.
- The Limits slide and the Thank you slide are two separate slides. Never merge them, and never put the thank-you or "questions?" on the Limits slide or in its speaker script. The last slide is a separate Thank you slide: dark, minimal, "Thank you" with a "Questions?" line (plus name or contact if I give them), and its speaker script holds the closing thanks and the invitation to questions.

## Honesty rules (these override the framing rules)
- Positive framing never changes facts. Verify every number against the project's files and outputs before using it.
- Never invent statistics. If a requested statistic isn't published, use established proxy sources, label them as such, and tell me in chat and in the notes.
- Every caveat goes into the "for you" notes, for example:
  - small samples or few test points;
  - theoretical versus measured values;
  - comparisons across different capture modes;
  - proposals that aren't implemented yet;
  - what a manufacturer's data was designed for.
- Claims about other companies' products must come from their own pages or reputable sources. Present the work as an independent recreation, not their technology.
- If a web source can't be fully loaded, say so and ask me to verify that figure.

## Default structure (for technical audiences)
Title → industry context (where the field is heading, e.g. spatial computing, AR/VR) → core principle → what makes this approach different → benefits as big numbers → real-world relevance with sourced statistics → pipeline overview → technical deep dive → calibration and accuracy → comparison with manufacturer data or a baseline → results on real data → future applications and opportunities → Limits and Next slide → Thank you slide (always separate, always last).

The technical deep dive is one slide each for:
- preprocessing;
- the core problem and how it is solved;
- the main algorithms;
- refinement and the final calculation.

Each technical slide uses the real equations, the actual parameters from the code, and measured timings. Emphasise practical advantages where they apply, such as no special rig or hardware, handheld use, and off-the-shelf devices.

# Skills by task

Cloud sessions load the skills I've enabled on claude.ai, but none of the plugins from my PC (engineering, finance, data, productivity, agent-memory). Before starting a task, check this list. If a row matches and the skill is in your skill list, invoke it with the Skill tool first and follow it.

Files and documents:
- Decks, slides, `.pptx`: `pptx` (plus the presentation rules above).
- Word documents, `.docx`, templates: `docx`.
- Spreadsheets, `.xlsx`, tabular data files: `xlsx`.
- PDFs (read, extract, fill, create): `pdf`.
- Editable shared docs, reports, anything people comment on: `docs`.
- Google Drive, Docs, Sheets or Slides changes: `google-workspace`, before the first connector call.

Research and other skills:
- Multi-source research, comparing options, fresh or niche facts: `deep-research`.
- Creating or improving a skill: `skill-creator`.
- Importing memories from another assistant: `import-memory`.
- Morning brief: `morning`.

Engineering tasks have no plugin skills here. Do them directly, the same way:
- Code review: check correctness, security and performance, and report findings ranked by severity.
- Debugging: reproduce, isolate, diagnose, fix, then add a test that fails without the fix.
- Architecture, system design, test strategy, tech debt and engineering docs: state the trade-offs and give a recommendation, not a survey.

Browser and computer-use skills (built-in browser, Claude in Chrome, computer use) control apps on my PC, so they don't work in cloud sessions.

If a matching skill is missing or fails to load, say so in the reply and continue with the closest fallback. Don't install or enable plugins on your own; ask first.

# Finishing a session

End with a short report: what you did, the branch and commit URLs (and PR, if any), test and check results as actual output, and anything left for me to decide or do.
