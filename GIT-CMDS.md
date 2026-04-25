# Install Git (if not yet)
```
sudo pacman -S git
```

## Set Identity
```
git config --global user.name "Your Name"
git config --global user.email "your@email.com"
```

## Check
```
git config --list
```

## Generate SSH key
```
ssh-keygen -t ed25519 -C "your@email.com"
```
(Enter for defaults)

## Start agent
bash:
```
eval "$(ssh-agent -s)"
ssh-add ~/.ssh/id_ed25519
```
or fish:
```
eval (ssh-agent -c)
ssh-add ~/.ssh/id_ed25519
```

## Add the new SSH key
Copy the public key
```
cat ~/.ssh/id_ed25519.pub
```
Go to progile -> settings -> SSH and GPG keys -> New SSH key, paste it.
Then test:
```
ssh -T git@github.com
```
This should succeed.

## Create or pick existing repo
This one is 'artix-config'.

## Create local repo
e.g. ~/git/artix-config:
```
mkdir -p ~/git/artix-config
cd ~/git/artix-config
git init
```

## Link remote
Replace USERNAME:
```
git remote add origin git@github.com:USERNAME/artix-config.git
git branch -M main
```
```
git push -u origin main
```
(if a commit already exists)

# Submit changes
```
git add .
git commit -m "<change description>"
git push
```
Once the repo is setup, `git push` is enough.

## Useful commands:
```
git status
git diff
git log --oneline
```
