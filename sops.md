# SOPS Secrets Management Setup Guide

Note that this is an opinionated way of working with sops, based on our preferences. There's many good ways. Probably even better ones out there. [Mozilla SOPS](https://github.com/getsops/sops) offers extensive documentaion.

## Prerequisites

Ensure you have Homebrew installed, then set up the required tools:

```bash
# Install SOPS and Age encryption tool
brew install sops
brew install age
```

## Generate Age Encryption Key

Generate an Age key pair for encrypting and decrypting secrets:

```bash
age-keygen
```

> **Important:**
>
> - Securely store the generated key in 1Password
> - Keep the private key confidential
> - The public key will be used for encryption

## Configure Editor for SOPS

Set up your preferred editor with the necessary wait flag:

```bash
# Export editor with --wait flag to ensure proper file handling
export EDITOR="cursor --wait"
# The --wait flag ensures sops receives the exit code only after the file is closed
```

## 1Password CLI Integration for Secret Management

### Create a Shell Function for Seamless Secret Handling

Add the following function to your `~/.zshrc` or `~/.bashrc`:

```bash
osops() {
    # 1Password configuration
    local vault_name="Private"
    local item_name="AGE_KEY"
    local field_name="SECRET_KEY"

    # Retrieve Age secret key from 1Password
    local age_secret
    age_secret=$(op item get "$item_name" --vault "$vault_name" --field "$field_name" 2>/dev/null)

    if [ -z "$age_secret" ]; then
        echo "Error: Unable to retrieve the Age secret key from 1Password CLI." >&2
        return 1
    fi

    # Set the SOPS Age key for encryption/decryption
    export SOPS_AGE_KEY="$age_secret"

    # Run SOPS with all provided arguments
    sops "$@"
}
```

### Usage Instructions

#### Encrypt a File in Place

```bash
# Encrypt an environment file
osops -e -i .prod.env
```

#### Edit an Encrypted File

```bash
# Open and edit an encrypted file
osops edit .prod.env
```

### Updating keys

Edit the `.sops.yaml` file as desired

```bash
sops updatekeys ./apps/crawler/.prod.env
```

## Best Practices

- Always use the `osops` function to manage encrypted files
- Ensure you're signed in to 1Password CLI before running commands
- Rotate encryption keys periodically
- Never commit unencrypted sensitive files to version control

## Troubleshooting

- Verify 1Password CLI is installed and configured
- Check that you're signed in to 1Password (`op signin`)
- Ensure the correct vault and item names are used in the `osops` function

## Security Notes

- The Age encryption key is fetched dynamically from 1Password
- Only authorized personnel with 1Password access can decrypt the files
- The encryption key is never stored permanently in the environment
