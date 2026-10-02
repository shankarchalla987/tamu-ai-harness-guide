# Using TAMU AI models in coding tools

**Texas A&M students can use any coding harness for their projects and courses, and code efficiently for free.** Your NetID already gives you access to Claude, GPT and Gemini models through TAMUS AI — this guide shows you how to plug them into the tools you actually code in.

A step-by-step guide to using the Texas A&M AI Chat API from coding assistants ("harnesses"). Right now it covers GitHub Copilot Chat in VS Code; other harnesses will be added later.

This is an unofficial guide. It is not maintained by Texas A&M Technology Services.

## The TAMU AI API

More about the TAMU AI API: [docs.it.tamu.edu/ai](https://docs.it.tamu.edu/ai)

| | |
|---|---|
| Base URL | `https://chat-api.tamu.ai/openai` |
| Chat endpoint | `https://chat-api.tamu.ai/openai/chat/completions` |
| Authentication | `Authorization: Bearer <your API key>` |
| Streaming | Supported |

Two things to know before you start:

- **There is no `/v1` in the path.** Many tools add `/v1` automatically. A request to `/openai/v1/chat/completions` is rejected with a 403. This is the most common setup problem, and the steps below avoid it.
- **Usage is metered.** TAMUS AI has a daily token allowance per user. See the official documentation at [docs.it.tamu.edu/ai](https://docs.it.tamu.edu/ai) for the allowance, the data-classification rules, and how to get an API key.

### Keep your key out of files

Your API key is tied to your NetID. Do not paste it into a config file, a notebook, or a repository. Keep it in an environment variable for terminal use:

```sh
export TAMU_API_KEY="paste-your-key-here"
```

Add that line to `~/.zshrc` (macOS) or `~/.bashrc` (Linux) if you want it in every terminal.

### Check that your key works

```sh
curl https://chat-api.tamu.ai/openai/chat/completions \
  -H "Authorization: Bearer $TAMU_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{"model":"protected.Claude-Sonnet-5","messages":[{"role":"user","content":"hi"}]}'
```

You should get a JSON reply containing the model's answer. If you get a 401, the key is wrong or the variable is not set in that terminal.

## Models

Copy the model ID exactly as written. IDs are case-sensitive, always start with `protected.`, and some contain spaces while others use hyphens (`protected.Claude Opus 4.8` but `protected.Claude-Opus-5`).

**Start with these two** — they are the ones tested in a harness for this guide:

| Model ID | Name |
|---|---|
| `protected.Claude-Opus-5` | Claude Opus 5 |
| `protected.Claude-Sonnet-5` | Claude Sonnet 5 |

<details>
<summary><strong>Full model catalog</strong> (click to expand)</summary>

Listed as the API reported them on 2 October 2026. Untested in a harness.

**Anthropic** — `protected.Claude Opus 4.8`, `protected.Claude Opus 4.7`, `protected.Claude Opus 4.6`, `protected.Claude Opus 4.5`, `protected.Claude Opus 4.1`, `protected.Claude Sonnet 4.6`, `protected.Claude Sonnet 4.5`, `protected.Claude Sonnet 4`, `protected.Claude-Haiku-4.5`, `protected.Claude 3.5 Haiku`

**OpenAI** — `protected.gpt-5.6-sol`, `protected.gpt-5.6-terra`, `protected.gpt-5.6-luna`, `protected.gpt-5.5`, `protected.gpt-5.4`, `protected.gpt-5.4-nano`, `protected.gpt-5.2`, `protected.gpt-5.1`, `protected.gpt-5`, `protected.gpt-5-mini`, `protected.gpt-5-nano`, `protected.gpt-4.1`, `protected.gpt-4.1-mini`, `protected.gpt-4.1-nano`, `protected.gpt-4o`, `protected.o3`, `protected.o3-mini`, `protected.o4-mini`

**Google** — `protected.gemini-3.5-flash`, `protected.gemini-3.1-flash-lite`, `protected.gemini-2.5-pro`, `protected.gemini-2.5-flash`, `protected.gemini-2.5-flash-lite`

**Meta** — `protected.llama3.2`

**Not chat models** (image generation and embeddings — do not add these to a coding assistant) — `protected.gpt-image-2`, `protected.gpt-image-1.5`, `protected.gpt-image-1-mini`, `protected.gemini-3.1-flash-image`, `protected.gemini-3.1-flash-lite-image`, `protected.text-embedding-3-small`

</details>

The catalog changes, so check it yourself:

```sh
curl -s https://chat-api.tamu.ai/openai/models \
  -H "Authorization: Bearer $TAMU_API_KEY" \
  | python3 -c "import json,sys; print('\n'.join(sorted(m['id'] for m in json.load(sys.stdin)['data'])))"
```

To use another model, copy its `id` from that output into the config for your harness.

## Harnesses

| Harness | Get the harness | Status | Config in this repo |
|---|---|---|---|
| GitHub Copilot Chat (VS Code) | [Free for students](https://github.com/education/students) · [VS Code](https://code.visualstudio.com/) | Working | [github-copilot/chatLanguageModels.json](github-copilot/chatLanguageModels.json) |
| Others | | Not written yet | |

## GitHub Copilot Chat in VS Code

Copilot Chat lets you bring your own model through its "Custom Endpoint" provider. Once set up, the TAMU models appear in the chat model picker next to Copilot's built-in ones.

Tested with VS Code 1.139 on macOS.

### What you need

- A recent version of VS Code.
- To be signed in to GitHub Copilot in VS Code. The free plan is enough — students can also get Copilot Pro free through the [GitHub Student Developer Pack](https://github.com/education/students). If your Copilot seat comes from an organisation, an admin may need to allow custom models.
- A TAMU AI API key that passes the curl check above.

### Steps

1. **Open the Copilot model config file.** It is called `chatLanguageModels.json` and lives in your VS Code user folder:

   | OS | Path |
   |---|---|
   | macOS | `~/Library/Application Support/Code/User/chatLanguageModels.json` |
   | Windows | `%APPDATA%\Code\User\chatLanguageModels.json` |
   | Linux | `~/.config/Code/User/chatLanguageModels.json` |

   If the file does not exist, create it. If it exists and already has content, make a copy of it first.

2. **Paste in the config from this repo.** Copy the contents of [github-copilot/chatLanguageModels.json](github-copilot/chatLanguageModels.json) into that file and save.

   If your file already has other providers, add the `TAMU AI` block to the existing list instead of replacing the whole file.

3. **Reload VS Code.** Press `Cmd+Shift+P` (`Ctrl+Shift+P` on Windows and Linux) and run **Developer: Reload Window**.

4. **Enter your API key.** Press `Cmd+Shift+P` and run **Chat: Manage Language Models**. Find the **TAMU AI** provider, click its gear icon, and paste your key into the API key field.

   VS Code stores the key in your operating system's secure storage (the Keychain on macOS). It is not written to the JSON file. The file only holds a reference such as `${input:tamuApiKey}`, and VS Code may replace that with its own reference like `${input:chat.lm.secret.1234abcd}`. Both are fine.

5. **Pick a model and test it.** Open Copilot Chat, click the model picker at the bottom of the chat box, and choose **Claude Sonnet 5 (TAMU)** or **Claude Opus 5 (TAMU)**. Send a message.

   If the models are not in the picker, open **Chat: Manage Language Models** again and make sure they are switched on.

### What the config fields mean

| Field | Value | Why |
|---|---|---|
| `vendor` | `customendpoint` | Selects Copilot's Custom Endpoint provider. Any other value is ignored. |
| `apiType` | `chat-completions` | The TAMU API speaks the OpenAI chat completions format. |
| `apiKey` | `${input:...}` | A reference to the stored key. Never put the key itself here. |
| `id` | `protected.Claude-Sonnet-5` | The exact model ID sent to the API. |
| `name` | `Claude Sonnet 5 (TAMU)` | The label shown in the model picker. You can change it. |
| `url` | `https://chat-api.tamu.ai/openai/chat/completions` | The full endpoint path. See the warning below. |
| `toolCalling`, `vision` | `true` | Lets Copilot use agent mode tools and image input with the model. |
| `maxInputTokens`, `maxOutputTokens` | `128000`, `16000` | Working values, not official limits. |

**Use the full URL, not the base URL.** If `url` is `https://chat-api.tamu.ai/openai`, VS Code turns it into `https://chat-api.tamu.ai/openai/v1/chat/completions`, which TAMU rejects. Ending the URL in `/chat/completions` stops VS Code from changing it.

### If something goes wrong

- **`token expired or invalid: 403`** — the `url` is the base URL, so VS Code added `/v1`. Use the full `/openai/chat/completions` path and reload.
- **`token expired or invalid: 401`** — re-enter your key via **Chat: Manage Language Models** → gear icon.
- **Models missing from the picker** — check `vendor` is `customendpoint`, the JSON is valid, and the models are switched on.

Copilot reports any 401 or 403 as "token expired or invalid", so it does not mean your GitHub login is broken. For the real error, open the Output panel (`Cmd+Shift+U`) → **GitHub Copilot Chat** and look for `Server error`.

## Adding another harness

Each harness gets its own folder with its config file, plus a section in this README with the steps. The facts that carry over to any tool are the chat endpoint, the bearer key, the exact model IDs, and the absence of `/v1` in the path.
