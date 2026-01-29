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
```

### Keys & Credentials

If you want to have access to the repository from the devcontainer you sometimes need to do additional tweaks based on how you have access to the repository from the host.

This setup is highly opinionated and thus some scripts will look for SSH keys in specific .devcontainer/.secrets/KEYS/_ directory and will also pass the configuration to the ssh if in .devcontainer/.secrets/SSH/_ folder.
