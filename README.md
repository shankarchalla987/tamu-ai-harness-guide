# Using TAMU AI models in coding tools

A step-by-step guide to using the Texas A&M AI Chat API from coding assistants ("harnesses"). Right now it covers GitHub Copilot Chat in VS Code; other harnesses will be added later.

This is an unofficial guide written by a student. It is not maintained by Texas A&M Technology Services.

## The TAMU AI API

Texas A&M runs TAMUS AI Chat ([tamus.ai](https://tamus.ai)), a university-approved chat platform that gives students, faculty, staff and researchers access to models from OpenAI, Anthropic and Google. The same models are available through an API, so you can use them from your own tools instead of the web chat.

The API is OpenAI-compatible: any tool that can talk to an OpenAI-style "chat completions" endpoint can use it.

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

| Model ID (use this in configs) | Suggested display name | Provider |
|---|---|---|
| `protected.Claude-Sonnet-5` | Claude Sonnet 5 (TAMU) | Anthropic |
| `protected.Claude-Opus-5` | Claude Opus 5 (TAMU) | Anthropic |

Model IDs are case-sensitive and include the `protected.` prefix.

These are the two models this guide has been set up with. TAMUS AI offers more. To see every model ID your key can use:

```sh
curl -s https://chat-api.tamu.ai/openai/models \
  -H "Authorization: Bearer $TAMU_API_KEY" | python3 -m json.tool
```

To use another model, copy its `id` from that list into the config for your harness.

## Harnesses

| Harness | Status | Config in this repo |
|---|---|---|
| GitHub Copilot Chat (VS Code) | Working | [github-copilot/chatLanguageModels.json](github-copilot/chatLanguageModels.json) |
| Others | Not written yet | |

## GitHub Copilot Chat in VS Code

Copilot Chat lets you bring your own model through its "Custom Endpoint" provider. Once set up, the TAMU models appear in the chat model picker next to Copilot's built-in ones.

Tested with VS Code 1.139 on macOS.

### What you need

- A recent version of VS Code.
- To be signed in to GitHub Copilot in VS Code. The free Copilot plan is enough. If your Copilot seat comes from an organisation, an admin may need to allow custom models.
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

### Troubleshooting

| What you see | Cause | Fix |
|---|---|---|
| `token expired or invalid: 403` | The URL is the base URL, so VS Code added `/v1`. | Set `url` to the full `/openai/chat/completions` path and reload. |
| `token expired or invalid: 401` | The stored key is missing or wrong. | Re-enter it through **Chat: Manage Language Models** and the gear icon. |
| Models missing from the picker | `vendor` is not `customendpoint`, the JSON is invalid, or the models are switched off. | Check the file against the one in this repo, then check **Chat: Manage Language Models**. |
| Error naming the model | The model ID is misspelled or not available to your key. | Run the `/models` command above and copy the ID exactly. |

Copilot shows "token expired or invalid" for any 401 or 403 from any endpoint, so the message does not mean your GitHub login has a problem. To see the real response from TAMU, open the Output panel (`Cmd+Shift+U`), choose **GitHub Copilot Chat** from the dropdown, and look for the line starting with `Server error`.

## Adding another harness

Each harness gets its own folder with its config file, plus a section in this README with the steps. The facts that carry over to any tool are the chat endpoint, the bearer key, the exact model IDs, and the absence of `/v1` in the path.
