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

This repository provides the following GitHub Actions located in the `.github/actions/` directory:

1.  **`Install SOPS` (`./.github/actions/install-sops`)**

    - **Purpose:** Installs a specific version of SOPS. This action is primarily used as a dependency by other actions in this repository.
    - **Note:** The actual `action.yml` for `install-sops` is not directly visible here but is referenced by other actions. Ensure it's correctly set up if you're modifying action paths.

2.  **`Decrypt SOPS Secrets` (`./.github/actions/load-sops-secrets`)**

    - **Purpose:** Decrypts a SOPS-encrypted file (e.g., an environment file) and loads the secrets as environment variables in your GitHub Actions workflow. Secrets are automatically masked in logs.
    - **Inputs:**
      - `sops-age-key`: (Required) The AGE private key for SOPS decryption.
      - `sops-file-path`: (Required) Path to the SOPS encrypted file.
    - **Example Usage:**
      ```yaml
      - name: Decrypt and Load Secrets
        uses: your-org/your-repo/.github/actions/load-sops-secrets@main # Adjust path accordingly
        with:
          sops-age-key: ${{ secrets.SOPS_AGE_KEY }}
          sops-file-path: ".env.sops"
      ```

3.  **`Execute SOPS Command` (`./.github/actions/execute-sops-command`)**

    - **Purpose:** Sets up SOPS, provides the AGE key, and executes a specified SOPS command (e.g., `sops --encrypt --in-place secrets.yaml`, `sops updatekeys secrets.yaml`).
    - **Inputs:**
      - `sops-age-key`: (Required) The AGE private key for SOPS operations.
      - `sops-args`: (Required) Arguments to pass to the SOPS command.
      - `sops-version`: (Optional) The version of SOPS to install (defaults to the version specified in the `install-sops` action).
      - `working-directory`: (Optional) The working directory to run the sops command in (defaults to `.`).
    - **Example Usage:**

      ```yaml
      - name: Encrypt a file in-place
        uses: your-org/your-repo/.github/actions/execute-sops-command@main # Adjust path accordingly
        with:
          sops-age-key: ${{ secrets.SOPS_AGE_KEY }}
          sops-args: "-e -i my-secrets.yaml"
          working-directory: "./config"

      - name: Update keys for a file
        uses: your-org/your-repo/.github/actions/execute-sops-command@main # Adjust path accordingly
        with:
          sops-age-key: ${{ secrets.SOPS_AGE_KEY_FOR_UPDATE }} # May need a different key or ensure the current one has rights
          sops-args: "updatekeys --yes my-infra-secrets.json"
      ```

## Repository Structure

- `.github/actions/`: Contains the custom GitHub Actions.
  - `install-sops/`: Action to install SOPS (dependency).
  - `load-sops-secrets/`: Action to decrypt files and load secrets into the environment.
  - `execute-sops-command/`: Action to execute arbitrary SOPS commands.
  - `sops.md`: Detailed guide for local SOPS setup and usage with 1Password.
- `.sops.yaml`: (Typically located in repositories using SOPS, not necessarily this one unless managing its own config). Configuration file for SOPS, defining encryption rules, key groups, etc.

## Contributing

Contributions are welcome! Please feel free to submit pull requests or open issues for bugs, feature requests, or improvements.

## License

MIT
