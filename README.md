# SOPS Utilities and GitHub Actions

This repository provides a collection of tools and GitHub Actions to streamline secrets management using [Mozilla SOPS](https://github.com/getsops/sops). It aims to simplify encrypting, decrypting, and managing sops-encrypted files both in CI/CD pipelines and local development environments.

## Overview

The primary goal is to offer robust and easy-to-use mechanisms for handling secrets, particularly leveraging `age` encryption.

Key features include:

- GitHub Actions for decrypting secrets and executing SOPS commands within workflows.
- A comprehensive guide for setting up SOPS locally, integrating with 1Password for secure `age` key handling.

## Getting Started

### Prerequisites

Before using the tools in this repository, ensure you have the following (as applicable to your use case):

- **For local development (refer to `sops.md`):**
  - [SOPS](https://github.com/getsops/sops#installation)
  - [age](https://github.com/FiloSottile/age#installation)
  - [1Password CLI](https://developer.1password.com/docs/cli/get-started/) (if using the `osops` helper)
- **For GitHub Actions:**
  - An `age` private key.

### Local Development and `osops`

For detailed instructions on setting up your local environment, generating `age` keys, configuring your editor for SOPS, and using the `osops` shell function for seamless secret handling with 1Password, please refer to:

➡️ **[SOPS Secrets Management Setup Guide](./.github/actions/sops.md)**

This guide covers:

- Generating and storing `age` keys.
- The `osops` function to automatically fetch your `SOPS_AGE_KEY` from 1Password.
- Common commands like `osops -e -i <file>` (encrypt) and `osops edit <file>`.

### GitHub Actions

This repository provides a unified GitHub Action, `action.yml` at the root, which can perform various SOPS operations based on its inputs. You can use it by referencing `your-org/your-repo@main` (replace `your-org/your-repo` with the actual path to this repository, e.g., `byont-ventures/sops@main`).

**Unified SOPS Utility Action (`uses: byont-ventures/sops@main`)**

- **Purpose:** Installs SOPS, decrypts SOPS-encrypted files to environment variables, or executes arbitrary SOPS commands.
- **Key Input:**

  - `operation`: (Required) Specifies the action to perform. Can be:
    - `'install'`: Only installs SOPS.
    - `'load-secrets'`: Decrypts a SOPS file and loads secrets into the environment.
    - `'execute-command'`: Executes a given SOPS command.

- **Common Inputs:**

  - `sops-version`: (Optional) The version of SOPS to install. Defaults to `v3.10.2`.
  - `sops-age-key`: (Required for `load-secrets` and `execute-command`) The AGE private key for SOPS operations.
  - `sops-file-path`: (Required for `load-secrets`) Path to the SOPS encrypted file.
  - `sops-args`: (Required for `execute-command`) Arguments to pass to the SOPS command.
  - `working-directory`: (Optional, for `execute-command`) The working directory to run the sops command in. Defaults to `.`.

- **Example Usages:**

  1.  **Install SOPS only:**

      ```yaml
      - name: Install SOPS
        uses: byont-ventures/sops@main
        with:
          operation: "install"
          sops-version: "v3.10.2" # Optional, uses default if not specified
      ```

  2.  **Decrypt and Load Secrets:**

      ```yaml
      - name: Decrypt and Load Secrets
        uses: byont-ventures/sops@main
        with:
          operation: "load-secrets"
          sops-age-key: ${{ secrets.SOPS_AGE_KEY }}
          sops-file-path: ".env.sops"
      ```

      Secrets are automatically masked in logs.

  3.  **Execute a SOPS Command (e.g., encrypt a file):**

      ```yaml
      - name: Encrypt a file in-place
        uses: byont-ventures/sops@main
        with:
          operation: "execute-command"
          sops-age-key: ${{ secrets.SOPS_AGE_KEY }}
          sops-args: "-e -i my-secrets.yaml"
          working-directory: "./config" # Optional
      ```

  4.  **Execute a SOPS Command (e.g., update keys):**
      ```yaml
      - name: Update keys for a file
        uses: byont-ventures/sops@main
        with:
          operation: "execute-command"
          sops-age-key: ${{ secrets.SOPS_AGE_KEY_FOR_UPDATE }} # May need a different key
          sops-args: "updatekeys --yes my-infra-secrets.json"
      ```

## Repository Structure

- `action.yml`: The main GitHub Action file for all SOPS utilities.
- `.github/actions/`:
  - `sops.md`: Detailed guide for local SOPS setup and usage with 1Password.
- `.sops.yaml`: (Typically located in repositories using SOPS, not necessarily this one unless managing its own config). Configuration file for SOPS, defining encryption rules, key groups, etc.

## Contributing

Contributions are welcome! Please feel free to submit pull requests or open issues for bugs, feature requests, or improvements.

## License

MIT
