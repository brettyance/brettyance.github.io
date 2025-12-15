**1) Title + promise**

- One sentence: what you’ll help the reader accomplish.
    
- Optional: who it’s for (“Proxmox beginners”, “people already running Ceph”, etc.)
    

**2) The problem (why this exists)**

- 3–6 sentences of context.
    
- The “pain” you hit, and what “done” looks like.
    

**3) Prereqs and assumptions**

- Environment assumptions (OS/version, hardware, network, permissions).
    
- What you’re _not_ covering (saves you from scope creep).
    
- A quick checklist readers can confirm before they start.
    

**4) Architecture / mental model**

- A small “how it works” section: components and how data/traffic flows.
    
- If it’s network-y, this is where a diagram belongs.
    
- Define terms you’ll reuse.
    

**5) Plan of attack (high level)**

- 4–8 bullet points describing the phases, not the commands.
    
- This becomes your table of contents.
    

**6) Implementation (the meat)**  
Break into sections that each follow this mini-pattern:

- **Goal** (one sentence)
    
- **Action** (steps/commands/config)
    
- **Verify** (how to confirm it worked; expected output)
    
- **What could go wrong** (1–3 common failure modes + fixes)
    

**7) Testing + validation**

- A dedicated section that proves the result.
    
- Include “failure testing” if relevant (reboot, node loss, service restart, etc.).
    

**8) Security / reliability notes**

- Threat model / safety boundaries (what you isolated, what you exposed).
    
- Backups, rollbacks, least privilege, firewall rules, secrets handling.
    

**9) Results**

- What improved (uptime, latency, maintenance effort).
    
- Any tradeoffs or gotchas you accepted.
    

**10) Appendix**

- Full configs, command dump, troubleshooting table, references/links.