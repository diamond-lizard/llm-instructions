# Using the Command Code API in Oh My Pi

Config-only setup that connects [omp](https://github.com/can1357/oh-my-pi) to the
[Command Code](https://commandcode.ai) Provider API — no plugins, no npm, no bun, and no
third-party code executing inside omp. Tested on omp 18.1.11 on Linux.

## What you get

- A `commandcode` provider in omp with the full live model catalog (all models Command Code
  serves — Claude, GPT, Gemini, GLM, DeepSeek, and more; 67 models at the time of writing).
- A statically guaranteed entry for `z-ai/glm-5.3-flash` even when offline.
- Model switching at any time via `/model` in the TUI or `--model` on the command line.

## How it works

Command Code's Provider API exposes standard OpenAI Chat Completions and Anthropic Messages
endpoints (see <https://commandcode.ai/docs/provider>):

```
https://api.commandcode.ai/provider/v1/chat/completions   (OpenAI format)
https://api.commandcode.ai/provider/v1/messages           (Anthropic format)
https://api.commandcode.ai/provider/v1/models             (model list)
```

omp natively speaks both formats. Registering Command Code as a *custom provider* in omp's
`~/.omp/agent/models.yml` is therefore pure configuration — omp's built-in OpenAI adapter
does all the work. Nothing is downloaded or installed beyond the config file itself.

## Requirements

- omp installed and on your PATH (`omp --version`).
- A Command Code account on a plan with API access. Per Command Code's docs every plan
  except the Go plan qualifies (GOAT, Pro, Max, Team, and the pay-as-you-go Provider plan).
- Your Command Code API key (starts with `user_...`).

## Step 1 — Get your API key

Create a key from [Command Code Studio](https://commandcode.ai/studio/). The same key
authenticates both their CLI and the API.

## Step 2 — Create the provider config

Create `~/.omp/agent/models.yml` (the default location; create the directory too, if it
doesn't exist). **If the file already exists, do not overwrite it** — add only the
`commandcode:` block under the existing top-level `providers:` key. The root object may
contain only `providers`; unknown root keys fail validation. For a fresh file:

```yaml
providers:
  commandcode:
    # Command Code Provider API (https://commandcode.ai/docs/provider)
    baseUrl: https://api.commandcode.ai/provider/v1
    # Replace PASTE_YOUR_COMMAND_CODE_KEY_HERE with your user_... key.
    apiKey: PASTE_YOUR_COMMAND_CODE_KEY_HERE
    api: openai-completions
    # Live model discovery from {baseUrl}/models.
    # injectV1: false is correct — the baseUrl already ends in /v1.
    discovery:
      type: openai-models-list
      injectV1: false
    models:
      - id: z-ai/glm-5.3-flash
        name: GLM-5.3 Flash (CC)
        api: openai-completions
        reasoning: true
        input: [text]
        contextWindow: 1048576
        maxTokens: 131072
        cost:
          input: 0
          output: 0
          cacheRead: 0
          cacheWrite: 0
        # Wire compatibility flags, mirroring Command Code's official integration:
        # this gateway expects max_tokens (not omp's max_completion_tokens default)
        # and the legacy system role (not developer).
        compat:
          supportsStore: false
          supportsDeveloperRole: false
          maxTokensField: max_tokens
```

Restart omp (or start a new session) after creating or editing this file.

Then replace the placeholder with your real key and tighten the file permissions, since the
key is stored in it:

```sh
chmod 700 ~/.omp/agent
chmod 600 ~/.omp/agent/models.yml
```

(Linux/macOS. On Windows, skip `chmod` — the profile directory is already private to
your user; the config lives at the equivalent location under your user profile,
`%USERPROFILE%\.omp\agent\`.)

### Key handling details

- omp resolves the `apiKey` value as follows: if an environment variable with that exact
  name exists, its value is used; otherwise the value itself is used as the key.
  So you can either paste the key into the file (simplest) or set an env var instead:
  `export COMMAND_CODE_API_KEY="user_..."` in your shell profile and use
  `apiKey: COMMAND_CODE_API_KEY` in the file.
- A value starting with `!` is executed as a shell command whose stdout becomes the key
  (e.g. `apiKey: "!cat ~/secrets/cc-key"`). Command Code keys start with `user_`, so an
  accidental `!` is unlikely — but don't prefix the key with anything.
- The `cost` fields only affect omp's cost *display*. Command Code's catalog does not
  publish prices, so zeros mean "unknown", not free. Your requests are billed by Command
  Code at its normal rates (see <https://commandcode.ai/docs/resources/pricing-limits>).

## Step 3 — Verify

```sh
omp models
```

Expected output includes a `commandcode` section listing the full catalog, e.g.:

```
commandcode (67)
┌───────────────────────────────────────┬─────────┬─────────┬──────────────┬────────┐
│ model                                 │ context │ max-out │ thinking     │ images │
├───────────────────────────────────────┼─────────┼─────────┼──────────────┼────────┤
│ z-ai/glm-5.3-flash                    │     1M  │  131K   │ low,high,max │ no     │
│ ...                                   │         │         │              │        │
```

Sanity-check the request path (expected result with an incorrect or placeholder key is an
authentication error from Command Code — that proves the wiring works):

```sh
omp -p "say hi" --model commandcode/z-ai/glm-5.3-flash
# with the real key in place this returns an actual completion
```

## Step 4 — Use it

- **TUI:** run `omp`, then `/model` and pick a model under the `commandcode` provider.
  Selecting a model in the model hub and confirming the default persists it across
  sessions (`modelRoles.default` in omp's settings). Switching between Command Code and any
  other provider is free and reversible at any time.
- **Fresh install:** if this is your only configured provider, omp's initial-model
  fallback ("first available model with a valid API key") selects a Command Code model
  automatically — no manual pick needed, and `/model` still switches anytime.
- **Headless:**

  ```sh
  omp -p "Reply with exactly: OK" --model commandcode/z-ai/glm-5.3-flash
  ```

## Choosing other models

The discovery fetch pulls the live catalog automatically, so new models appear without any
config change. To browse ids yourself:

```sh
curl -s https://api.commandcode.ai/provider/v1/models
```

Use the `id` values verbatim as omp model ids (provider-qualified form:
`commandcode/<id>`).

## Troubleshooting

| Symptom | Cause and fix |
| --- | --- |
| Startup warning: `models.yml validation failed — custom providers disabled` | The YAML failed schema validation. Check indentation (two spaces per level), that `id`/`name`/`api` are present on each model, and that `contextWindow`/`maxTokens` are positive integers. omp prints the specific error below the warning. |
| `401 Invalid 'Authorization' header` when chatting | The key is still the placeholder, wrong, or missing. Check the `apiKey` line in `models.yml`. |
| `commandcode` section missing from `omp models` | `models.yml` didn't parse (see first row), or the file is at the wrong path — it must be `~/.omp/agent/models.yml`. |
| Chat errors mentioning `reasoning_effort` | omp sends the selected thinking level as `reasoning_effort` for models marked `reasoning: true`. If a model rejects it, remove `reasoning: true` from that entry, or keep it and add `compat: { supportsReasoningEffort: false }` to omit the wire field. |
| Only your static `models:` entries appear; discovery never fetches | With env-var-style `apiKey: SOME_VAR`, an unset variable resolves to nothing and omp marks the provider unauthenticated, skipping discovery. Export the variable (then restart omp) or paste the key directly into the file. |

## Removing it

Delete the `commandcode:` block (or the whole file, if you keep nothing else in it). No
uninstall step exists because nothing was installed.

## Security notes

- No plugins, no npm/bun, no third-party code: the entire integration is a config file
  interpreted by omp's built-in OpenAI-compatible adapter.
- Network traffic goes only to `api.commandcode.ai` (and `commandcode.ai` for account
  management in your browser). The `/models` list requires no authentication; chat requests
  authenticate with `Authorization: Bearer <key>`.
- The API key is stored wherever you put it — the config file (keep it `chmod 600`) or an
  environment variable. Anyone who can read that file or env var can use your credits.

## What this setup deliberately does not include

This config-only route uses omp's native provider machinery. It does not provide the
plugin-style extras some Command Code integrations add — a `/login` entry (omp's `/login`
is OAuth-only and covers built-in providers), quota/usage slash commands, or automatic
reasoning-effort translation tables. The model catalog itself stays current through the
discovery fetch.
