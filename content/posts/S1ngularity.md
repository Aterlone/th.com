+++
title = "S1ngularity Malware"
date = "2025-09-05"
author = "Tane Haines"
cover = ""
tags = ["", ""]
keywords = ["", ""]
description = ""
showFullContent = false
readingTime = false
hideComments = false
+++

I found out about the malware called S1ngularity. This is my writeup on it.

<!--more-->

# Analyzing the *s1ngularity* Malware

Recently, I examined a Node.js–based malware sample called **`s1ngularity`**, which showcases how attackers can abuse JavaScript tooling ecosystems to steal secrets, and dump the secrets anonymously.

---

## Technical Overview

`s1ngularity` is a **Node.js script** that executes on **Linux/macOS systems only**. It deliberately exits on Windows hosts, targeting UNIX-like environments where developer tools and wallets are more commonly installed.

The malware performs several malicious actions:

- **Wallet/Key Harvesting**  
  It recursively searches sensitive directories (`$HOME`, `.config`, `.local/share`, `.ethereum`, `.electrum`, `/etc`, `/tmp`, etc.) for wallet-related files:
  - Keystores (`UTC--`, `keystore.json`)
  - Cryptocurrency wallets (Metamask, Electrum, Ledger, Trezor, Exodus, Phantom, Solflare, Trust Wallet)
  - Private keys (`id_rsa`, `.secret`, `secrets.json`)
  - Environment files (`.env`)

- **System Reconnaissance**  
  It collects system details such as:
  - Hostname, platform, OS type and release
  - Environment variables
  - Installed CLI tools (checks for `claude`, `gemini`, `q`)
  - GitHub CLI authentication tokens
  - NPM account details (`npm whoami` and `.npmrc` content)

- **Persistence & Disruption**  
  The script appends the command:

      sudo shutdown -h 0

  to the user’s shell initialization files (`.bashrc`, `.zshrc`), ensuring the system shuts down immediately on login. The only way the user could fix this would be by logging in as a different user and editing the file.

- **Data Exfiltration**  
  - Harvested files are **Base64-encoded** and staged into `/tmp/inventory.txt`.  
  - If a **GitHub token** is found, the malware automatically creates a new public repository named `s1ngularity-repository`.  
  - It then uploads the stolen data as `results.b64` to the victims GitHub account.
  - This allows for the attacker to scan for public repositories with this name and exfiltrate all of the data into their own environment.
---

## The Functions

- `isOnPathSync(cmd)` -> Detects if CLI tools exist on the system.  
- `runBackgroundSync(cmd, args)` -> Executes background commands and captures output.  
- `forceAppendAgentLine()` -> Establishes persistence by appending shutdown commands.  
- `githubRequest(pathname, method, body, token)` -> Handles authenticated GitHub API requests for repo creation and data upload.  
- `processFile(listPath)` -> Reads `inventory.txt`, encodes file contents to Base64, and stages them for exfiltration.

---

## Why It’s Dangerous

- **Targets Developers & Crypto Users**  
  By focusing on wallet files, SSH keys, and `.env` secrets, this malware zeroes in on high-value assets.  

- **Silent Exfiltration via GitHub**  
  Using the victim’s own GitHub account to host exfiltrated data makes detection harder, since outbound traffic looks legitimate.  

- **Persistence & Sabotage**  
  The forced shutdown on login prevents victims from using their machines normally, effectively locking them out.  

---

## Conclusion

The *s1ngularity* malware demonstrates how attackers can weaponize even small Node.js scripts to compromise user credentials of un-alert developers, as well as more alert developers who happened to update to a non-stable release of the tool. This exploit showed us the importance for developers to keep a watchout for these attacks, alongside the security researchers.  