---
layout: false
title: SSH tips
author: Auxcoder
categories: Tools
date: 2015-06-15 17:48:32
tags:
- SSH
- RSA
comments: false
---

## Checking for existing SSH keys

```sh
ls -al ~/.ssh
```

If an existing pair of public/private keys exist then we can add it to the shh client.

## Generating new public and private keys
This is very nice command, you will get a 1024 bit key pair (public and private keys) with no password prompts and many extra things.

```sh
ssh-keygen -b 1024 -t dsa -f id_dsa -P ''
```


## Adding your SSH key to the ssh-agent
When adding your SSH key to the agent, use the default macOS `ssh-add` command, and not one installed by _macports_, _homebrew_, or some other external source.

*Start the ssh-agent in the background.*
```sh
eval "$(ssh-agent -s)"
```

If you're using macOS Sierra 10.12.2 or later, you will need to modify your `~/ssh/config` file to automatically load keys into the `ssh-agent` and store passphrases in your keychain`

*automatically load keys into the ssh-agent and store passphrases in your keychain.*
```
Host *
 AddKeysToAgent yes
 UseKeychain yes
 IdentityFile ~/.ssh/id_rsa
```

Add your SSH private key to the ssh-agent and store your passphrase in the keychain. If you created your key with a different name, or if you are adding an existing key that has a different name, replace id_rsa in the command with the name of your private key file.

*Add key to ssh-agent*
```sh
ssh-add -K ~/.ssh/id_rsa
```

## Testing your SSH connection

```ssh
ssh -T git@github.com
```

## Compress and copy file to local from remote
```sh
ssh kiubmenc@kiubmen.com tar czpf - /home1/kiubmenc/www/kiubmen/wp-content/themes/kiubgular/ | tar xzpf - -C /Users/pavelduran/Desktop
```
```sh
ssh ensipes@ensip.es tar czpf - /home5/cardisig/ | tar xzpf - -C /Users/kiub/Desktop/
```

## Connecting Vía SSH
```sh
ssh kiubmen@kiubmen.com
```
