# Devcontainers

Here we document decisions and explain the logic behind some of the opinionated dev setup.

## Security

### Preferred option

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

### Other commands that you can use

This a list of snippets...

```bash
# Generate key, encrypt it with password... (you can leave empty for random password)
age-keygen | age -p > ./.devcontainer/.secrets/KEYS/sops_age_key_dev_env.txt.age

# First time you try to use sops decrypt it can fail because we need to create encrypted sops file
sops encrypt --age <age_pub_key> .env > .env

# Once the encrypted file is created we need
sops encrypt --age 'ssh-ed25519 <ssh_pub_key> developer@company.domain' .env.example > .env
```

```bash
# Generate age key and encrypt it with age password (Save )
age-keygen | age -p > ./.devcontainer/.secrets/KEYS/sops_age_key_dev_env.txt.age

# Decrypt age password file
SOPS_AGE_KEY_FILE="./.devcontainer/.secrets/KEYS/sops_age_key.txt" EDITOR="code --wait" sops .env
```

```bash
age-keygen -o sops_age_key.txt
sops encrypt --age <pub_key> .env > .enc.env
SOPS_AGE_KEY_FILE="./sops_age_key.txt" sops decrypt .enc.env > .env

# How to open vscode 
SOPS_AGE_KEY_FILE="./sops_age_key.txt" EDITOR="code --wait" sops .env
```