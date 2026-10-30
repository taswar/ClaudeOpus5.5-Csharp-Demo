# Contributing

Thanks for helping improve this sample. It's a small .NET 10 console app with five samples that call Claude Opus 5.5 through Microsoft Foundry, using the official Anthropic .NET SDK. Contributions that make it clearer, more correct, or more useful to other .NET developers are welcome.

## Ways to contribute

- **Report a bug.** A sample fails, or the README doesn't match what the code does.
- **Fix the docs.** Typos, unclear steps, missing troubleshooting cases.
- **Build a roadmap item.** The [README roadmap](README.md#roadmap) lists what's planned, such as streaming responses, structured JSON output into C# records, prompt caching, and running with a plain Anthropic API key.
- **Share results.** If you ran a sample against your own code or screenshots, an issue describing what you saw helps others.

For anything larger than a small fix, open an issue first so we can agree on the approach before you write code.

## Set up

1. Install the [.NET 10 SDK](https://dotnet.microsoft.com/download/dotnet/10.0) (not just the runtime).
2. Clone the repo and run the offline sample to check your setup:

   ```bash
   git clone https://github.com/taswar/ClaudeOpus5.5-Csharp-Demo.git
   cd ClaudeOpus5.5-Csharp-Demo
   dotnet run -- 3
   ```

   Sample 3 needs no API key, so it's the quickest way to confirm the project builds and runs.

3. To run the live samples, deploy Claude Opus 5.5 on a Microsoft Foundry resource, then set `AZURE_AI_FOUNDRY_RESOURCE`, `AZURE_AI_FOUNDRY_API_KEY` and, if your deployment isn't called `claude-opus-5-5`, `AZURE_AI_FOUNDRY_DEPLOYMENT` with user secrets, as described in the [README](README.md#run-against-claude).

## Guidelines

- **Never commit secrets.** No API keys, resource names, tenant IDs or tokens in code, config files or screenshots. Settings belong in user secrets or environment variables.
- **Keep the agent loop bounded.** Sample 5 has a turn limit, a token cap per call and a shared timeout. Keep those limits, and don't let a tool write files, run commands or reach the network.
- **Keep sample 3 offline.** It has to keep working with no API key, so people can check their setup first.
- **Keep it a sample.** The code should stay short and readable. Prefer clarity over abstraction, and add a dependency only when it clearly helps.
- **Match the existing style.** Target `net10.0`, keep nullable reference types on, and follow the naming and layout already in `Program.cs`.
- **Keep the README in sync.** If you add a sample, change settings or change output, update the README in the same pull request, including the sample table and the roadmap.
- **One change per pull request.** Small, focused pull requests are easier to review and merge.

## Pull request checklist

- [ ] `dotnet build` succeeds with no new warnings.
- [ ] `dotnet run -- 3` still passes offline.
- [ ] Any sample you changed runs against your own Foundry deployment, or you explain why you couldn't run it.
- [ ] No secrets, resource names or personal data are in the diff, including in screenshots.
- [ ] The README reflects any change in samples, settings or output.
- [ ] The pull request description says what changed and why.

## Reporting a bug

Open an issue and include:

- Which sample you ran (`dotnet run -- 1` to `5`, or `all`) and what you expected.
- The full error message, with any keys or resource names removed.
- Your .NET SDK version (`dotnet --version`) and operating system, and whether you used WSL.
- Your deployment name, if it isn't `claude-opus-5-5`.

## Security issues

Please don't report security problems in a public issue. Use **Security → Report a vulnerability** on the repo to send a private report.

## License

By contributing, you agree that your contributions are licensed under the [MIT License](LICENSE) that covers this project.
