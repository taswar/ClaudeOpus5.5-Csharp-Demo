# Claude Opus 5.5 C# Demo (.NET 10)

A C# console application demonstrating the Anthropic Messages API through Microsoft Foundry: code review, effort settings, refusal handling, vision, and a bounded tool-calling loop.

The default deployment name in the code is `claude-opus-5-5`. Set `AZURE_AI_FOUNDRY_DEPLOYMENT` to the **actual deployment name in your resource** if it differs. A default name in this sample does not create a deployment or guarantee model availability for your subscription.

## Prerequisites

- **.NET 10 SDK**, not just the runtime: [Download .NET 10](https://dotnet.microsoft.com/download/dotnet/10.0).
- Windows, Linux, macOS, or WSL with the SDK installed in the environment where you run the application.
- Access to NuGet to restore dependencies.
- For live cases (1, 2, 4, and 5):
  - An Azure subscription and a **Microsoft Foundry resource** with access to an appropriate deployed Claude model.
  - The resource name and a valid **API key for that Foundry resource**. This is not an API key from the public Anthropic Console. API-key authentication must be enabled for the resource.
  - A deployment supporting the requested features: effort settings for cases 2 and 5, images for case 4, and tool use for case 5.
  - Network access to `https://<resource-name>.services.ai.azure.com` over HTTPS, including any required private-network/VPN access.
- For case 4: a real PNG image no larger than **4 MiB**. Additional service image limits may apply.

**Case 3 runs offline** without an Azure resource, API key, or model deployment, once packages have been restored. Live calls can incur inference charges; case 5 can make multiple calls.

This sample uses API-key authentication. Azure CLI login, Azure Developer CLI, a database, and a hosted-agent deployment are not required.

## Required .NET libraries

These package references are already included in `ClaudeOpus55FoundryDemo.csproj`:

| NuGet package | Referenced version | Purpose |
| --- | --- | --- |
| `Anthropic` | `12.50.0` | Messages API, message types, images, effort settings, and tool definitions |
| `Anthropic.Foundry` | `0.7.1` | Foundry client and resource API-key authentication |
| `Microsoft.Extensions.Configuration.EnvironmentVariables` | `10.0.0` | Read environment-variable configuration |
| `Microsoft.Extensions.Configuration.UserSecrets` | `10.0.0` | Read development secrets outside the project directory |

`System.Text.Json` is supplied by .NET; no separate package is needed. Restore installs the referenced packages and their transitive dependencies automatically.

## 1. Restore and build

Open a terminal in the directory containing `ClaudeOpus55FoundryDemo.csproj`:

```sh
dotnet --version
dotnet restore
dotnet build
```

Confirm that the selected SDK can target `net10.0`.

For an initial offline smoke check:

```sh
dotnet run -- 3
```

Expected output:

```text
Request declined. Review the request or seek human review.
Offline refusal-handler check passed.
```

## 2. Configure Foundry access

The application loads .NET user secrets first, then environment variables. **Environment variables override user secrets.** It does not load `.env` or `appsettings.json` files.

| Setting | Required | Value |
| --- | --- | --- |
| `AZURE_AI_FOUNDRY_RESOURCE` | All cases except standalone case 3 | Resource subdomain, for example `my-foundry-resource` |
| `AZURE_AI_FOUNDRY_API_KEY` | All cases except standalone case 3 | API key belonging to that resource |
| `AZURE_AI_FOUNDRY_DEPLOYMENT` | Optional | Actual model deployment name; defaults to `claude-opus-5-5` when unset |

### Which resource name should I use?

Use the resource subdomain from your Foundry endpoint. For example:

```text
Resource endpoint: https://my-foundry-resource.services.ai.azure.com
Resource setting:  my-foundry-resource
SDK API endpoint:  https://my-foundry-resource.services.ai.azure.com/anthropic
```

Find the parent resource in your Foundry project's details, or open the resource in the Azure portal and inspect its endpoint and keys. Use the endpoint's subdomain, **not** a project name, deployment name, resource-group name, subscription ID, or portal URL.

The application also accepts recognized HTTPS resource URLs and extracts the subdomain:

```text
https://my-foundry-resource.services.ai.azure.com/anthropic
https://my-foundry-resource.cognitiveservices.azure.com/
```

The SDK still connects to the `services.ai.azure.com/anthropic` endpoint. A recognized URL is not proof that the resource has a compatible Claude deployment. Prefer the bare resource name to avoid ambiguity. Bare full hostnames such as `my-foundry-resource.services.ai.azure.com` are not accepted; use the subdomain or a supported HTTPS URL.

### Option A: .NET user secrets (local development)

The project already has a `UserSecretsId`; do not run `dotnet user-secrets init` again. Run these commands from the project directory, replacing the placeholders locally:

```sh
dotnet user-secrets set "AZURE_AI_FOUNDRY_RESOURCE" "my-foundry-resource"
dotnet user-secrets set "AZURE_AI_FOUNDRY_API_KEY" "<your-foundry-resource-api-key>"
dotnet user-secrets set "AZURE_AI_FOUNDRY_DEPLOYMENT" "<your-deployment-name>"
```

User secrets are stored outside the repository but are **not encrypted**. Commands containing keys may also remain in shell history. Do not commit keys, paste them into shared logs, or send them in chat. Use an appropriate secret-management mechanism for non-development environments.

### Option B: environment variables

**Bash / Linux / macOS / WSL:**

```bash
export AZURE_AI_FOUNDRY_RESOURCE="my-foundry-resource"
export AZURE_AI_FOUNDRY_API_KEY="<your-foundry-resource-api-key>"
export AZURE_AI_FOUNDRY_DEPLOYMENT="<your-deployment-name>"
dotnet run -- 1
```

**PowerShell:**

```powershell
$env:AZURE_AI_FOUNDRY_RESOURCE = "my-foundry-resource"
$env:AZURE_AI_FOUNDRY_API_KEY = "<your-foundry-resource-api-key>"
$env:AZURE_AI_FOUNDRY_DEPLOYMENT = "<your-deployment-name>"
dotnet run -- 1
```

These environment assignments apply to the current shell session and its child processes.

### WSL notes

Windows and WSL have separate SDK installations and user-secret stores. If running with Linux `dotnet`, configure user secrets inside WSL as the same Linux user. Do not assume Windows PowerShell environment variables are automatically forwarded to WSL.

From a WSL terminal, navigate to your checkout and run the normal setup commands above. From Windows PowerShell, you can invoke the app explicitly in WSL (adjust the distribution and path):

```powershell
wsl -d Ubuntu --cd /home/taswar/projects/code/ClaudeOpus55FoundryDemo -- dotnet run -- 1
```

For WSL vision runs, pass a Linux path such as `/home/taswar/images/error-dialog.png` or `/mnt/c/Users/<user>/Pictures/error-dialog.png`, not a Windows drive path.

## 3. Run the samples

```text
dotnet run -- [1|2|3|4|5|all] [optional-png-path]
```

The first `--` separates `dotnet run` options from application arguments. With no arguments, the app runs **case 1**. The PNG path is the second application argument and is used only by case 4, including when running `all`. Quote paths containing spaces.

| Argument | Sample | Behavior |
| --- | --- | --- |
| `1` | Basic code review | Sends a bundled C# counter to the model for a thread-safety review; up to 4,096 output tokens |
| `2` | Low-effort response | Summarizes three illustrative failing test names with `Effort.Low`; up to 4,096 output tokens |
| `3` | Offline refusal handling | Deserializes a synthetic refusal response and verifies the local handler; no model call |
| `4` | Vision | Sends a PNG and asks about the exception shown and its likely cause; up to 2,048 output tokens |
| `5` | Tool-calling migration plan | Uses `Effort.High` and `read_sample_file` to read two bundled snippets and propose a .NET Framework 4.8 to .NET 10 migration plan; up to 8 turns and 8,192 output tokens per call |
| `all` | All samples | Runs cases 1–5 in order; requires live configuration even though case 3 itself is offline |

### Examples

```sh
# Default: case 1
dotnet run

# Run each case explicitly
dotnet run -- 1
dotnet run -- 2
dotnet run -- 3
dotnet run -- 4
dotnet run -- 5

# Vision using a custom PNG
dotnet run -- 4 "images/error-dialog.png"

# Run all cases with the default image path
dotnet run -- all

# Run all cases with a custom image
dotnet run -- all "images/error-dialog.png"

# Build and run a Release configuration
dotnet run --configuration Release -- 1
```

### Case-specific notes

- **Case 2** does not execute tests. It asks the model to reason from example test names.
- **Case 3** tests refusal-handling logic using a fixture; it does not deliberately trigger a live model refusal.
- **Case 4** defaults to `error-dialog.png` relative to the process working directory. No image is bundled. A missing file prints a skip message, not a failure. An oversized file or invalid PNG signature fails the case. The selected image is sent to the service; remove sensitive information before using it.
- **Case 5** reads only the in-memory `LegacyApp.csproj` and `OrderRepository.cs` examples embedded in `Program.cs`. It does not read arbitrary files, connect to a database, edit the project, execute commands, or perform an actual migration. No .NET Framework installation or SQL Server package is required for those text snippets.

## Output, cancellation, and errors

- Live responses print text and a stop reason. Output is model-generated and can vary between runs.
- A `max_tokens` stop reason prints a truncation warning; do not treat the answer as complete.
- Press **Ctrl+C** to request cancellation.
- Live execution has a shared **ten-minute limit** across the run, not ten minutes per case.
- On a caught case failure, the app prints exception details, endpoint, and deployment to standard error and sets exit code `1`. The configured API key is redacted from the exception text; review logs for other sensitive details before sharing.
- With `all`, ordinary case failures do not prevent later cases from running. Cancellation stops the sequence. Missing or invalid startup configuration prevents the sequence from starting.
- Unsupported case arguments print usage and return; currently this does not set a nonzero exit code.

## Troubleshooting

| Symptom | What to check |
| --- | --- |
| `AZURE_AI_FOUNDRY_RESOURCE is not set` | Configure the resource in the same OS/user environment that runs `dotnet`. Check for environment-variable overrides. |
| Invalid resource configuration | Use the bare resource subdomain or a recognized HTTPS resource URL, not a portal URL or project/deployment name. |
| `AZURE_AI_FOUNDRY_API_KEY is not set` | Configure the Foundry resource API key through user secrets or environment variables. |
| `I/O exception` | Read the inner exception. Check resource configuration, DNS, HTTPS connectivity, proxy/VPN settings, and TLS trust in the runtime environment. Do not disable certificate validation. |
| Authentication or authorization error | Confirm the key belongs to the selected resource, is current, and key authentication is permitted. Check applicable access/network restrictions. |
| Deployment/model not found | Set `AZURE_AI_FOUNDRY_DEPLOYMENT` to the exact deployment name in this resource. |
| Unsupported effort, vision, or tools | Verify that the selected model deployment supports the feature used by that case. |
| Rate limiting or quota errors | Check deployment quota/capacity and retry later; case 5 and `all` make more requests. |
| Vision skipped | Supply a PNG path or put `error-dialog.png` in the working directory. |
| Agent turn limit reached | Review the partial output. The tool loop is deliberately bounded to eight turns. |
| SDK cannot target `net10.0` | Install/select the .NET 10 SDK in the environment executing the build. |

An earlier version passed a full Cognitive Services URL directly into `AnthropicFoundryApiKeyCredentials`, which expects a resource name and produced an I/O error. The current code normalizes supported URLs before creating the client and includes inner exceptions in failure output.

## Project layout

```text
ClaudeOpus55FoundryDemo.csproj  Target framework and NuGet references
Program.cs                    Configuration, sample cases, and error handling
README.md                     Setup and usage
```

## References

- [Use Anthropic Claude models in Microsoft Foundry](https://learn.microsoft.com/azure/foundry/foundry-models/how-to/use-foundry-models-claude)
- [Microsoft Foundry SDKs and endpoints](https://learn.microsoft.com/azure/foundry/how-to/develop/sdk-overview)
- [.NET Secret Manager](https://learn.microsoft.com/aspnet/core/security/app-secrets)