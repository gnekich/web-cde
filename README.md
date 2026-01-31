# WEB CDE

This repository is intended for developers that are looking for a containerised development environment setup for developing web applications.

Focus of this repository is Node.js setup. With specific set of tools...

- Google Cloud SDK
- Google Cloud SQL Proxy
- Hasura CLI
- Node.js v24

## VsCode Devcontainers

We are using vscode devcontainers to do the setup.

The vscode should be set such that you can do the commit signing out of the box, thus we also provide .vscode configuration.

## Scripts

## Setup

to use this boilerplate for new project you can use following commands:

```bash
mkdir newProject
cd newProject
git clone https://github.com/gnekich/web-cde
mv web-cde/{.,}* .
rm -rf web-cde
# rm -rf .git # (optional)
```

or you can install [degit](https://github.com/Rich-Harris/degit)


### Git config

```bash
git config user.name "John Doe"
git config user.email "redacted@redacted"
git config gpg.format ssh
git config user.signingkey ./.devcontainer/.secrets/KEYS/your-user-signing-priv-keys
git branch -m main
```

### Keys & Credentials

If you want to have access to the repository from the devcontainer you sometimes need to do additional tweaks based on how you have access to the repository from the host.

This setup is highly opinionated and thus some scripts will look for SSH keys in specific .devcontainer/.secrets/KEYS/_ directory and will also pass the configuration to the ssh if in .devcontainer/.secrets/SSH/_ folder.

### SOPS

We should use SOPS with age whenever possible...

First let's generate few keys... one for developer (we recommend using a password) and one key for deployments (local-deployment, without password, because it is easier setup in CI/CD and not necessary more secure and should only be used for deployment)

```bash
# Generate ED25519 key for the developer it can be used for SSH, signing commits, encrypting/sharing local or env specific .env, sending sensitive files or diagrams, etc. 
# Please change "developer@company.domain" and name of the key "developer-key" (USE PASSWORD)
ssh-keygen -t ed25519 -C "developer@company.domain"  -f ./.devcontainer/.secrets/KEYS/developer-key

# Generate deployment key (SSH, well why not, no password, same situation if you are using age key it should have no password for deployment, as SOPS_AGE_PASSPHRASE= does not exist as an option outside testing) I like this more as I get both pub and priv key while in age key I need to make pub key file...
# Ref: https://github.com/getsops/sops/issues/933
ssh-keygen -t ed25519 -C "local-env-developer@company.domain"  -f ./.devcontainer/.secrets/KEYS/local-env-key
```

Encrypting the .env files to .enc.env
```bash
# First time to create .enc.env as empty sops/age file
sops encrypt --age "$(cat ./.devcontainer/.secrets/KEYS/developer-key.pub)" --input-type dotenv --output-type dotenv .enc.env > .enc.env

# You can also do this for multiple dev team members
sops encrypt --age "$(cat ./.devcontainer/.secrets/KEYS/developer-key.pub),$(cat ./.devcontainer/.secrets/KEYS/local-env-key.pub)" --input-type dotenv --output-type dotenv .enc.env > .enc.env

# This way we can encrypt the data to correct type; dotenv, because it ends on .example it will default to json, also do this for input type !!! or else you will get bugged encrypted file... If your file is called example.env you don't need to do this.
sops encrypt --age "$(cat ./.devcontainer/.secrets/KEYS/developer-key.pub)" --input-type dotenv --output-type dotenv .env.example > .enc.env

# To live edit in the vscode .enc.env
SOPS_AGE_SSH_PRIVATE_KEY_FILE="./.devcontainer/.secrets/KEYS/developer-key" EDITOR="code --wait" sops .enc.env

# To decrypt .enc.env (this will ask for password)
SOPS_AGE_SSH_PRIVATE_KEY_FILE="./.devcontainer/.secrets/KEYS/developer-key" sops decrypt .enc.env

# In deployment use recipient without password like local-env-key
SOPS_AGE_SSH_PRIVATE_KEY_FILE="./.devcontainer/.secrets/KEYS/local-env-key" sops decrypt .enc.env
```

### Age

Because we have age as our main encrypt/decrypt tool we can do this for files.

```bash
# Encrypt
age \
    -R './.devcontainer/.secrets/KEYS/developer-key.pub' \
    -r "$(curl https://github.com/benjojo.keys)" sensitive.file > payload.age

# Decrypt
age -d -i './.devcontainer/.secrets/KEYS/developer-key' payload.age > result
```