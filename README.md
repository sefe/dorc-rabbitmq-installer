# DOrc RabbitMQ Installer

Windows MSI installer that packages [RabbitMQ Server](https://www.rabbitmq.com/) with [Erlang/OTP](https://www.erlang.org/) for automated deployment via [DOrc](https://github.com/sefe/dorc).

## What It Builds

The CI workflow produces a `drop` artifact containing:

| File | Purpose |
|------|---------|
| `Server/RabbitMQInstaller.msi` | WiX MSI that installs RabbitMQ Server and registers the Windows service |
| `Server/RabbitMQInstaller.msi.json` | DOrc deployment descriptor (maps MSI parameters to DOrc properties) |
| `Server/otp_win64_<version>.exe` | Erlang/OTP installer (prerequisite — deployed alongside the MSI) |

## Current Versions

| Component | Version |
|-----------|---------|
| RabbitMQ  | 4.2.5   |
| Erlang/OTP | 27.3.4.1 |
| WiX Toolset SDK | 6.0.2 |

## Updating Versions

Versions are parameterised — update the env vars in [`.github/workflows/build.yml`](.github/workflows/build.yml):

```yaml
env:
  RABBITMQ_VERSION: '4.2.5'
  ERLANG_VERSION: '27.3.4.1'
```

Or trigger a one-off build with different versions via **Actions → RabbitMQ Installer Build → Run workflow** and fill in the version inputs.

The workflow downloads both binaries from GitHub Releases during build, so no large files are stored in this repo.

## DOrc Integration

Configure your DOrc project with:

| Setting | Value |
|---------|-------|
| **ArtefactsUrl** | `https://api.github.com/repos/sefe/dorc-rabbitmq-installer` |
| **ArtefactsSubPaths** | `build.yml` |
| **SourceControlType** | `GitHub` |

When triggering a deployment via the [DOrc GitHub Action](https://github.com/sefe/dorc-github-action), use:

- **build-text**: `RabbitMQ Installer Build`
- **build-num**: `latest` (or a specific run number)

## Local Development

Prerequisites: [.NET SDK](https://dotnet.microsoft.com/download) (for WiX Toolset SDK restore).

```powershell
# Download binaries (or place them manually in RabbitMQInstaller/)
$rmq = "4.2.5"; $erl = "27.3.4.1"
Invoke-WebRequest "https://github.com/rabbitmq/rabbitmq-server/releases/download/v$rmq/rabbitmq-server-windows-$rmq.zip" -OutFile "RabbitMQInstaller/rabbitmq-server-windows-$rmq.zip"
Invoke-WebRequest "https://github.com/erlang/otp/releases/download/OTP-$erl/otp_win64_$erl.exe" -OutFile "RabbitMQInstaller/otp_win64_$erl.exe"

# Build
dotnet build RabbitMQInstaller.sln -c Release -p:Platform=x64
```

The MSI output is in `RabbitMQInstaller/bin/x64/Release/`.

## Automatic Update Checks

A scheduled workflow ([`update-check.yml`](.github/workflows/update-check.yml)) runs every Monday and:

1. Checks for new stable RabbitMQ releases on GitHub
2. Checks for new Erlang/OTP patch releases in the current major series (27.x)
3. Verifies Windows binary assets exist for both
4. Opens a PR with the version bumps if updates are available

You can also trigger it manually via **Actions → Check for RabbitMQ Updates → Run workflow**.
