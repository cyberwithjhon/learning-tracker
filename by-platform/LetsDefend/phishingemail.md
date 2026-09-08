# Phishing Email – Case Study

> Platform: LetsDefend
> 
> 
> Difficulty: Beginner
> 
> Category: Email Analysis / Phishing / SOC
> 

---

# Objective

This lab was designed to teach the fundamentals of phishing-email investigation from a SOC analyst perspective.

The main skills involved were:

- Inspecting an email beyond its visible content.
- Reading email headers and identifying the `Return-Path`.
- Extracting and analyzing URLs contained in an email.
- Distinguishing a legitimate hosting provider from malicious content hosted on that provider.
- Using VirusTotal to enrich indicators.
- Identifying a Body SHA-256 associated with an HTTP response.
- Combining multiple indicators to reach a phishing verdict.

The broader objective was to practice **evidence-based email triage rather than relying only on visual intuition**.

---

# Initial Thoughts

My first approach was to inspect the email and understand what it was trying to make the recipient do.

The message appeared to be related to PayPal and a reward, which immediately suggested a possible phishing scenario. However, I did not want to classify it only because the message looked suspicious.

The initial assumption was that the important evidence would be hidden in the email source and in the URL embedded in the message.

The investigation therefore moved from:

```
Visible email
    ↓
Headers
    ↓
Embedded link
    ↓
External reputation / infrastructure analysis
```

---

# My Methodology

I approached the investigation as an indicator-enrichment process.

### Step 1 — Understand the lure

The email used PayPal branding and a reward-related message to encourage interaction.

### Step 2 — Inspect the underlying headers

Instead of trusting the visible sender, I inspected the raw email data and looked for the `Return-Path`.

This revealed:

```
bounce@rjttznyzjjzydnillquh.designclub.uk.com
```

That created an immediate mismatch between the claimed brand and the actual mail infrastructure.

### Step 3 — Identify the embedded URL

The next indicator was the URL contained in the message.

The relevant domain was:

```
storage.googleapis.com
```

### Step 4 — Investigate the infrastructure

VirusTotal was used to investigate the URL/domain and inspect the associated HTTP response details.

This was an important reasoning step because `storage.googleapis.com` is a legitimate Google service.

The correct question was therefore not:

> “Is Google legitimate?”
> 

but:

> “Is this specific resource and its context legitimate?”
> 

### Step 5 — Extract the Body SHA-256

The HTTP response information exposed the Body SHA-256:

```
13945ecc33afee74ac7f72e1d5bb73050894356c4bf63d021a1a53e76830567f5
```

### Step 6 — Correlate the evidence

The final classification was based on the combination of:

```
Brand impersonation
+
Suspicious Return-Path
+
Unrelated/random infrastructure
+
Suspicious embedded resource
+
External reputation/context
=
Phishing
```

---

# Challenges

## Understanding the difference between a legitimate domain and malicious use of that domain

One of the most useful parts of the investigation was `storage.googleapis.com`.

The domain itself is legitimate Google infrastructure.

That creates an important analytical challenge: a legitimate domain does not automatically mean that every resource hosted on it is legitimate.

This changed the investigation from simple domain reputation checking to **resource-level analysis**.

---

# Mistakes

### Searching too broadly when a specific indicator was already available

Some of the investigation/research became broader than necessary.

**What I would avoid next time:** formulate the next question from the evidence already collected.

For example:

```
I have the URL
      ↓
What exactly does the challenge ask for?
      ↓
Domain
      ↓
Domain analysis
      ↓
HTTP response
      ↓
Body SHA-256
```

This is more efficient than searching for generic phishing information.

---

### Confusing the URL/domain with the content hash

A URL, domain, and HTTP response body are three different things.

**What I would avoid next time:** explicitly identify what is being hashed before copying or calculating any hash.

---

# Key Decisions

## 1. Inspecting the raw email headers

This was the most important early decision.

**Why it was correct:** visible sender information can be misleading, while headers provide technical delivery information.

The `Return-Path` exposed infrastructure that did not match the PayPal theme.

---

## 2. Investigating the embedded URL

The email’s URL provided a second independent indicator.

**Why it was correct:** phishing emails commonly rely on malicious or abused infrastructure behind links.

---

## 3. Looking beyond the reputation of the hosting provider

The use of Google infrastructure could easily create a false sense of legitimacy.

**Why it was correct:** security analysis should distinguish between a legitimate platform and the legitimacy of a specific resource hosted on that platform.

---

## 4. Correlating multiple indicators

The final verdict was not based on one artifact.

It was based on the combined evidence:

```
Email theme
+
Return-Path
+
URL
+
Infrastructure
+
HTTP response
+
External analysis
```

This is much closer to real SOC triage than simply labeling an email suspicious because it “looks fake.”

---

# New Concepts Learned

- `Return-Path` is an important email-header indicator during phishing investigations.
- The visible sender and the underlying mail infrastructure can tell different stories.
- Legitimate cloud providers can be abused to host malicious content.
- Phishing classification is stronger when multiple independent indicators support the same conclusion.
- Indicator enrichment is a core SOC investigation workflow.

---

# New Tools

| Tool | Purpose | When would I use it again? |
| --- | --- | --- |
| VirusTotal | Enrich URLs/domains and inspect HTTP response information | During phishing, malware, and IOC investigations |
| Browser developer/search tools | Navigate external analysis results and locate specific fields | When working through web-based security-analysis platforms |

---

# Commands Worth Remembering

```bash
# Search an exported .eml file for the Return-Path
grep -i "Return-Path:" email.eml

# Search for URLs in an email source
grep -Eoi 'https?://[^" <]+' email.eml
```

If the email is available as a local `.eml` file, these commands provide a quick way to extract useful indicators without relying entirely on the graphical mail client.

---

# Blue Team Perspective

## How could this attack be detected?

A SOC could detect similar phishing attempts through:

- Email security gateway alerts.
- Mismatch between the claimed brand and sender/return-path infrastructure.
- URLs pointing to cloud-storage or other third-party hosting platforms.
- Newly observed or unusual sender domains.
- User reports of unexpected reward/account messages.
- URL reputation detections.
- Browser or proxy telemetry showing users accessing suspicious resources.
- Repeated delivery of the same message to multiple users.

## How could it be mitigated?

- Use secure email gateways with URL reputation and sandboxing.
- Apply SPF, DKIM, and DMARC correctly and monitor failures.
- Rewrite or detonate suspicious URLs in a safe analysis environment.
- Block known malicious URLs and indicators.
- Educate users about unexpected rewards, urgency, and account-verification requests.
- Restrict access to suspicious newly observed infrastructure where appropriate.
- Monitor cloud-hosting abuse patterns rather than blocking entire legitimate providers.

## What logs or alerts would be useful?

Useful telemetry includes:

- Email gateway logs.
- Message IDs and delivery metadata.
- Sender/recipient relationships.
- `Return-Path`, `From`, and `Reply-To` values.
- URL click/proxy logs.
- DNS queries.
- Endpoint browser telemetry.
- VirusTotal or other threat-intelligence enrichment.
- SIEM correlation between email delivery and subsequent URL access.

---

# Key Takeaways

- **Lesson 1:** Never trust the visible sender alone; inspect the email headers.
- **Lesson 2:** A legitimate hosting provider can still host malicious content.
- **Lesson 3:** Investigate the exact resource, not only the parent domain.
- **Lesson 4:** Understand what an indicator represents before using it as evidence.
- **Lesson 5:** Correlating several moderate indicators can produce a strong phishing verdict.

---

# What I Want to Practice Next

I would reinforce:

- Email-header analysis.
- SPF, DKIM, DMARC, and ARC interpretation.
- MIME structure and `.eml` analysis.
- URL extraction and normalization.
- VirusTotal URL/domain investigations.
- Identifying phishing hosted on legitimate cloud infrastructure.
- IOC enrichment and pivoting.
- Writing concise SOC investigation timelines.
- Converting phishing analysis into SIEM detection logic.
- Building phishing detections around sender/return-path mismatches and suspicious URL infrastructure.

---

# Personal Reflection

If I repeated this lab today, I would make the investigation more structured from the beginning.

I would follow:

```
1. Read the email
2. Identify the social-engineering lure
3. Inspect headers
4. Extract Return-Path
5. Extract every relevant URL
6. Separate domain from full URL
7. Investigate the exact resource
8. Extract the requested indicators
9. Correlate evidence
10. Make the final verdict
```

This lab taught me that phishing analysis is fundamentally an exercise in **evidence correlation**.

The technical solution is relatively simple, but the transferable SOC skill is learning to move from a suspicious-looking message to a defensible incident classification using headers, URLs, infrastructure, and external intelligence.
