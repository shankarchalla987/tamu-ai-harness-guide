# Using TAMU AI Models in Coding Tools

**Texas A&M students can code with Claude, GPT and Gemini for free.** Your NetID already gives you access through TAMUS AI. This guide shows how to plug those models into the coding tools ("harnesses") you actually use.

This is an unofficial guide. It is not maintained by Texas A&M Technology Services.

| Harness | Get it | Status | Setup |
|---|---|---|---|
| GitHub Copilot Chat (VS Code) | [Free for students](https://github.com/education/students) | Working | [Jump to steps](#github-copilot-chat-in-vs-code) |
| Cline (VS Code) | [Free extension](https://marketplace.visualstudio.com/items?itemName=saoudrizwan.claude-dev) | Working | [Jump to steps](#cline-in-vs-code) |

## Before you start

You need a TAMU AI API key. The official docs at [docs.it.tamu.edu/ai](https://docs.it.tamu.edu/ai) explain how to get one and what data you may use with it.

| | |
|---|---|
| Base URL | `https://chat-api.tamu.ai/openai` |
| Chat endpoint | `https://chat-api.tamu.ai/openai/chat/completions` |
| Model to start with | `protected.Claude-Sonnet-5` |

Three things to know:

- **There is no `/v1` in the URL.** Many tools add it automatically, and TAMU then rejects the request with a 403. The steps below avoid this.
- **Usage is limited per day.** The allowance is tied to your NetID and resets between 6 and 7 p.m. Central Time.
- **Keep your key private.** Do not paste it into a file in a project, a notebook or a repository.

### Check that your key works

Run this in a terminal, with your key in place of `paste-your-key-here`:

```sh
export TAMU_API_KEY="paste-your-key-here"

curl https://chat-api.tamu.ai/openai/chat/completions \
  -H "Authorization: Bearer $TAMU_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{"model":"protected.Claude-Sonnet-5","messages":[{"role":"user","content":"hi"}]}'
```

A JSON reply with the model's answer means the key works. A 401 means the key is wrong.

## Models

Start with one of these two. They are the ones tested for this guide.

| Model ID | Best for |
|---|---|
| `protected.Claude-Sonnet-5` | Everyday coding |
| `protected.Claude-Opus-5` | Harder problems; uses your allowance faster |

Copy model IDs exactly. They are case-sensitive and always start with `protected.`.

<details>
<summary>Full model catalog</summary>

Listed as the API reported them on 2 October 2026. Untested in a harness. Note that some IDs contain spaces and others use hyphens.

**Anthropic** — `protected.Claude Opus 4.8`, `protected.Claude Opus 4.7`, `protected.Claude Opus 4.6`, `protected.Claude Opus 4.5`, `protected.Claude Opus 4.1`, `protected.Claude Sonnet 4.6`, `protected.Claude Sonnet 4.5`, `protected.Claude Sonnet 4`, `protected.Claude-Haiku-4.5`, `protected.Claude 3.5 Haiku`

**OpenAI** — `protected.gpt-5.6-sol`, `protected.gpt-5.6-terra`, `protected.gpt-5.6-luna`, `protected.gpt-5.5`, `protected.gpt-5.4`, `protected.gpt-5.4-nano`, `protected.gpt-5.2`, `protected.gpt-5.1`, `protected.gpt-5`, `protected.gpt-5-mini`, `protected.gpt-5-nano`, `protected.gpt-4.1`, `protected.gpt-4.1-mini`, `protected.gpt-4.1-nano`, `protected.gpt-4o`, `protected.o3`, `protected.o3-mini`, `protected.o4-mini`

**Google** — `protected.gemini-3.5-flash`, `protected.gemini-3.1-flash-lite`, `protected.gemini-2.5-pro`, `protected.gemini-2.5-flash`, `protected.gemini-2.5-flash-lite`

**Meta** — `protected.llama3.2`

**Not chat models** (image generation and embeddings — do not add these to a coding assistant) — `protected.gpt-image-2`, `protected.gpt-image-1.5`, `protected.gpt-image-1-mini`, `protected.gemini-3.1-flash-image`, `protected.gemini-3.1-flash-lite-image`, `protected.text-embedding-3-small`

The catalog changes. To see the current list:

```sh
curl -s https://chat-api.tamu.ai/openai/models \
  -H "Authorization: Bearer $TAMU_API_KEY" \
  | python3 -c "import json,sys; print('\n'.join(sorted(m['id'] for m in json.load(sys.stdin)['data'])))"
```

</details>

## GitHub Copilot Chat in VS Code

The TAMU models appear in Copilot's model picker next to the built-in ones. You need to be signed in to GitHub Copilot in VS Code; the free plan is enough. Tested with VS Code 1.139 on macOS.

1. **Open (or create) `chatLanguageModels.json`** in your VS Code user folder:

   | OS | Path |
   |---|---|
   | macOS | `~/Library/Application Support/Code/User/chatLanguageModels.json` |
   | Windows | `%APPDATA%\Code\User\chatLanguageModels.json` |
   | Linux | `~/.config/Code/User/chatLanguageModels.json` |

2. **Paste in** the contents of [github-copilot/chatLanguageModels.json](github-copilot/chatLanguageModels.json) and save. If your file already has other providers, add the `TAMU AI` block to the existing list instead of replacing it.

3. **Reload VS Code.** Press `Cmd+Shift+P` (`Ctrl+Shift+P` on Windows and Linux) and run **Developer: Reload Window**.

4. **Enter your key.** Run **Chat: Manage Language Models** from the same menu, click the gear icon next to **TAMU AI**, and paste your key. VS Code keeps it in your system's secure storage, not in the JSON file.

5. **Test it.** Open Copilot Chat, pick **Claude Sonnet 5 (TAMU)** in the model picker, and send a message.

<details>
<summary>If something goes wrong</summary>

Copilot shows any 401 or 403 as "token expired or invalid". It does not mean your GitHub login is broken.

- **`token expired or invalid: 403`** — a `url` in the file is the base URL, so VS Code added `/v1`. Each `url` must end in `/openai/chat/completions`.
- **`token expired or invalid: 401`** — enter your key again (step 4).
- **Models missing from the picker** — open **Chat: Manage Language Models** and switch them on. Also check that the JSON is valid and `vendor` is `customendpoint`.

For the real error, open the Output panel (`Cmd+Shift+U`), choose **GitHub Copilot Chat**, and look for `Server error`.

</details>

<details>
<summary>What the config file contains</summary>

The file sets up seven models: Claude Sonnet 5, Claude Opus 5, Claude Haiku 4.5, GPT-5.5, GPT-5 mini, Gemini 2.5 Pro and Gemini 3.5 Flash. Only the first two have been tested. Delete any entry you do not want, or copy an entry and change its `id` and `name` to add another.

| Field | Value | Why |
|---|---|---|
| `vendor` | `customendpoint` | Selects Copilot's Custom Endpoint provider. Any other value is ignored. |
| `apiType` | `chat-completions` | The TAMU API speaks the OpenAI chat completions format. |
| `apiKey` | `${input:...}` | A reference to the stored key. VS Code may rewrite it; never put the key itself here. |
| `id` | `protected.Claude-Sonnet-5` | The exact model ID sent to the API. |
| `name` | `Claude Sonnet 5 (TAMU)` | The label in the model picker. You can change it. |
| `url` | `https://chat-api.tamu.ai/openai/chat/completions` | The full endpoint, so VS Code does not add `/v1`. |
| `toolCalling`, `vision` | `true` | Allows agent mode tools and image input. |
| `maxInputTokens`, `maxOutputTokens` | `128000`, `16000` | Working values, not official limits. |

If your Copilot seat comes from an organisation, an admin may need to allow custom models.

</details>

## Cline in VS Code

Cline is a free, open-source coding agent. It can read your project, edit files and run commands. There is no config file to copy; you type the settings into Cline. You do not need a Cline account or ClinePass. Tested with Cline 4.1.22 on macOS.

1. **Open the provider screen.** Click the Cline icon in the VS Code sidebar and choose to use your own API key. If you have used Cline before, click the gear icon at the top of its panel instead.

2. **Set API Provider to `OpenAI Compatible`.** It starts on OpenRouter.

3. **Fill in the fields:**

   | Field | Value |
   |---|---|
   | Base URL | `https://chat-api.tamu.ai/openai` |
   | API Key | your TAMU AI key |
   | Model ID | `protected.Claude-Sonnet-5` |

   Use the base URL here, not the full endpoint. Cline adds `/chat/completions` itself. This is the opposite of Copilot.

4. **Set the model details**, if Cline shows a **Model Configuration** or **Advanced** section: supports images on, context window `128000`, max output tokens `16000`. Leave the prices at 0.

5. **Click Continue and test it.** Send `hi` or `explain this project`.

Cline has two modes. **Plan** discusses a change without touching anything. **Act** edits files and runs commands, and asks for your approval before each action.

<details>
<summary>Make your daily allowance last</summary>

An agent uses far more tokens than a chat window. Every step resends the instructions, the files it has read and the whole conversation.

- Start a new task when the token counter at the top of a task gets large.
- Use Sonnet for everyday work. To use Opus only for planning, tick **Use different models for Plan and Act modes** and set Plan mode to `protected.Claude-Opus-5`.
- Do not add whole folders or very large files to the context unless you need them.

</details>

<details>
<summary>If something goes wrong</summary>

- **403** — the Base URL contains `/v1`. Set it to exactly `https://chat-api.tamu.ai/openai`.
- **401** — the key is wrong. Paste it again in Cline's settings (gear icon).
- **Model not found** — the Model ID must match the catalog exactly, including `protected.` and the capital letters.
- **Rate limit or usage limit errors** — your daily allowance is used up. A new key or a different tool will not help. Wait for the evening reset.

</details>

## Adding another harness

Each harness gets a section in this README with the steps, plus its own folder if it has a config file to copy. The facts that carry over to any tool are the chat endpoint, the bearer key, the exact model IDs, and the absence of `/v1` in the path.
