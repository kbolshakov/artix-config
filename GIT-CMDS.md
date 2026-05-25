# Install Git (if not yet)
```bash
sudo pacman -S git
```

## Set Identity
```bash
git config --global user.name "Username"
git config --global user.email "your@email.com"
```

## Check
```bash
git config --list
```

## Generate SSH key
```bash
ssh-keygen -t ed25519 -C "your@email.com"
```
(Enter for defaults)

## Start agent
bash:
```bash
eval "$(ssh-agent -s)"
ssh-add ~/.ssh/id_ed25519
```
or fish:
```bash
eval (ssh-agent -c)
ssh-add ~/.ssh/id_ed25519
```

## Add the new SSH key
Copy the public key
```bash
cat ~/.ssh/id_ed25519.pub
```
Go to progile -> settings -> SSH and GPG keys -> New SSH key, paste it.
Then test:
```bash
ssh -T git@github.com
```
This should succeed.

## Create or pick existing repo
This one is 'artix-config'.

## Create local repo
e.g. ~/git/artix-config:
```bash
mkdir -p ~/git/artix-config
cd ~/git/artix-config
git init
```

## Link remote
Replace USERNAME:
```bash
git remote add origin git@github.com:USERNAME/artix-config.git
git branch -M main
```
```bash
git pull origin main     # for existing, pull first
git push -u origin main
```
(if a commit already exists)

# Submit changes
```bash
git add .
git commit -m "<change description>"
git push
```
Once the repo is setup, `git push` is enough.

## Useful commands:
```bash
git status
git diff
git log --oneline

git pull --rebase  # for out-of-date repo, when a commit was pushed from another, and a change was started

git fetch
git log HEAD..origin/main --oneline  # view HEAD at GitHub side
git log origin/main..HEAD --oneline  # view HEAD at local side
```
