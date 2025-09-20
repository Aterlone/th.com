+++
title = "Shai-Hulud Detector"
date = "2025-09-18"
author = "Tane Haines"
cover = ""
tags = ["malware", "nodejs", "security", "npm", "threat-intelligence"]
keywords = ["shai-hulud", "Node.js malware", "GitHub token detection", "bundle scanner", "supply-chain security"]
description = "A simple and practical Python tool to scan Node.js bundles for suspicious activity and potential GitHub token exfiltration."
showFullContent = false
readingTime = true
hideComments = false
+++

I made a small Python script to check JavaScript bundles for suspicious stuff, especially things that malware like **Shai-Hulud** might try to do. Thought I’d share a bit about it.

Here's the github in case you are interested: [Cick Here](https://github.com/TaneHaines/shai-hulud-detector)

<!--more-->

# Shai-Hulud Bundle Scanner

Basically, the idea came from seeing how Node.js supply-chain attacks can sneak in through packages. I wanted something really simple that could:

- Spot sudden bundle size increases  
- Check for GitHub token prefixes like `ghp_` or `gho_`  
- Look for suspicious network calls (`fetch`, `XMLHttpRequest`)  
- Peek at base64 or hex-encoded strings that might hide tokens  

---

## Why I Made It

I haven't really seen any proper tools for Shai-Hulud prevention despite it being around for a few days, so I decided to make one.

---

## How It Works

1. **Bundle Size Monitoring**  
   The script checks the current size of your bundle and compares it to a baseline. If it’s grown too much since the last check, it flags it.  

2. **GitHub Token Check**  
   It looks for `ghp_` or `gho_` anywhere in the file. It also tries to decode base64 and hex strings in case tokens are hidden.  

3. **Suspicious Network Calls**  
   It scans for `fetch()` or `XMLHttpRequest` calls to external URLs, which could mean someone is trying to send your data somewhere.  

---

## How to Use It

Run:
python3 check_bundle_size.py


- First time, it asks for your bundle path.  
- Then it checks for size changes, token prefixes, and network calls.  
- Warnings and info are printed in color-coded text so you can see what’s happening at a glance.  

---

### Example Output

If everything looks fine:

```
Bundle file: dist/bundle.js
Current bundle size: 3.45 MB
Growth within limit (+0.12 MB)
No GitHub token prefixes or suspicious network calls detected
```

If something looks off:

```
Warning: Bundle grew by 4.2 MB (limit 3 MB)
Potential GitHub token prefixes detected: ghp_
Suspicious network calls found: fetch
```

---

## Takeaway

We really need small and simple scripts in our gh directories which scan for malicious packages. It's quick to make and can prevent a lot of chaos... this is to you 
**CEO's: LOW COST + HIGH SECURITY** :0.

Anyway, please use my tool or create your own threat detection tools. I'd love some PR's which include changes/suggestions to improve my tool. 
https://github.com/TaneHaines/shai-hulud-detector

Make sure to star it so more people know about it!
