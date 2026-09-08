# Redeemer – Case Study

> Platform: Hack The Box
> 
> 
> Difficulty: Easy
> 

---

# Objective

This lab was designed to reinforce practical skills in:

- Network reconnaissance and full TCP port enumeration.
- Service and version identification with Nmap.
- Redis service enumeration.
- CLI-based interaction with exposed infrastructure services.
- Identifying sensitive data without relying on complex exploitation.
- Adapting methodology when the initial approach does not produce useful results.

The main lesson was that effective enumeration can reveal a direct path to the objective without requiring a vulnerability exploit.

---

# Initial Thoughts

My initial approach was to perform basic connectivity and network reconnaissance against the target.

I initially assumed that a standard Nmap scan would be enough to identify the relevant attack surface. When the first scan did not reveal anything useful, I changed direction and performed a full TCP port scan.

At the beginning, I also had a tendency to think in terms of finding an exploit once a service was identified. The lab ultimately showed that this assumption was unnecessary: the exposed service itself provided enough access to reach the objective.

---

# My Methodology

The investigation evolved through several stages:

```
Target verification
        ↓
Initial reconnaissance
        ↓
Full TCP port scan
        ↓
Service/version identification
        ↓
Redis-specific investigation
        ↓
Direct connection with redis-cli
        ↓
Key enumeration
        ↓
Interesting data identified
        ↓
Flag retrieved
```

The decisive indicator was the discovery of:

```
6379/tcp open redis Redis key-value store 5.0.7
```

Once Redis was identified, I moved from generic reconnaissance to service-specific enumeration.

I briefly investigated possible exploits with `searchsploit`, but the absence of relevant results reinforced the need to inspect the service directly.

The most important reasoning change was:

> **Do not assume exploitation is required just because a service is exposed. First determine what the service allows you to access.**
> 

---

# Challenges

The challenge was understanding how to interact effectively with Redis. I performed some broad web research around Redis commands and authentication before establishing a focused enumeration workflow.

Once connected with `redis-cli`, the path became much clearer:

```
KEYS *
```

returned:

```
1) "flag"
2) "temp"
3) "stor"
4) "numb"
```

The presence of a key named `flag` immediately provided a strong indicator.

---

# Mistakes

### Searching for exploits too early

I ran:

```bash
searchsploit Redis key-value store 5.0.7
```

and received no useful results.

**What I would avoid next time:** investigate service behavior and access controls before looking for an exploit.

### Invalid commands inside Redis

I attempted commands such as:

```
SHOWDATABASE;
ls
```

which were not valid Redis commands.

**What I would avoid next time:** explicitly identify which environment I am currently interacting with before using a command.

---

# Key Decisions

## 1. Performing a full port scan

This was the most important reconnaissance decision.

It revealed Redis on port `6379.`

**Why it was correct:** it expanded the attack surface instead of assuming that common ports represented the complete target.

---

## 2. Switching from exploit hunting to service enumeration

After identifying Redis, I did not need to force an exploit path.

**Why it was correct:** direct access to the service was already possible.

An alternative would have been to continue searching for a Redis vulnerability, but that would have added complexity without evidence that exploitation was necessary.

---

## 3. Enumerating Redis keys

I used:

```
KEYS *
```

The `flag` key immediately stood out.

**Why it was correct:** Redis is a key-value data store, so enumerating keys is a natural way to understand what data is exposed.

---

## 4. Validating the interesting key

I used:

```
GET flag
```

which returned the flag:

```
03e1d2b376c37ab3f5319922053953eb
```

**Why it was correct:** it directly validated the hypothesis that the key contained the objective.

---

# New Concepts Learned

- Full TCP enumeration is important when standard scans reveal little or nothing.
- Redis exposes a different interaction model from a Linux shell or SQL database.
- Service enumeration should precede exploitation whenever possible.
- An exposed data store can itself represent a serious security weakness.
- Meaningful key names can dramatically reduce enumeration time.
- A lack of exploit results does not mean the service is uninteresting.
- The ability to change hypotheses is an important penetration-testing skill.

---

# New Tools

| Tool | Purpose | When would I use it again? |
| --- | --- | --- |
| `nmap` | Port, service, and version enumeration | During initial network reconnaissance and whenever the attack surface is unclear |
| `redis-cli` | Direct interaction with a Redis server | When Redis is discovered and direct access needs to be tested or enumerated |
| `searchsploit` | Local exploit database search | After identifying a service/version, but preferably after basic service enumeration |
| Web search | Documentation and command research | When I need to understand an unfamiliar service or validate a specific hypothesis |

---

# Commands Worth Remembering

```bash
# Full TCP port enumeration
nmap -p- --min-rate=1500 10.129.18.70 -Pn

# Service and version detection
nmap -sVC -p6379 10.129.18.70 -Pn

# Connect directly to Redis
redis-cli -h 10.129.18.70
```

Once inside Redis:

```
# Enumerate keys
KEYS *

# Retrieve the value of a specific key
GET flag
```

### Why they are useful

`nmap -p-` helps avoid missing services running on non-standard ports.

`nmap -sVC` provides service and version information that helps guide further investigation.

`redis-cli` allows direct interaction with Redis once the service is exposed.

`KEYS *` provides a quick view of the keys available in the current Redis database, while `GET` retrieves the value associated with a specific key.

---

# Blue Team Perspective

An exposed Redis instance should be treated as a significant security concern, especially if authentication is not enforced and sensitive information is stored in it.

### Detection

A blue team could look for:

- Unexpected external connections to TCP `6379`.
- Redis authentication failures or unusual connection patterns.
- Commands such as `KEYS`, `GET`, or other enumeration activity from unexpected hosts.
- Network connections to Redis from systems that do not normally interact with it.
- Sensitive data being accessed from Redis outside normal application behavior.

### Mitigation

- Do not expose Redis directly to untrusted networks.
- Restrict access using network segmentation and firewall rules.
- Enable appropriate Redis authentication/access controls.
- Avoid storing sensitive information unnecessarily.
- Monitor Redis connections and administrative activity.
- Keep the Redis deployment properly configured and maintained.

### Useful logs and alerts

Useful telemetry would include:

- Source and destination IPs for connections to TCP `6379`.
- Redis authentication and command activity where logging is available.
- Firewall/network-flow logs.
- IDS/IPS alerts for unexpected Redis exposure or access.
- Host-level process and network telemetry on the Redis server.

The key blue-team lesson is that the attack did not require a sophisticated exploit. **Misconfigured exposure and excessive data access can be enough.**

---

# Key Takeaways

- **Lesson 1:** Always consider the full attack surface; important services may run on unexpected ports.
- **Lesson 2:** Identify and understand a service before assuming exploitation is necessary.
- **Lesson 3:** Direct access to an exposed data store can be more important than finding a CVE.
- **Lesson 4:** Errors such as invalid commands are useful feedback about gaps in mental models and tooling knowledge.
- **Lesson 5:** A disciplined, evidence-driven workflow is more valuable than knowing a large number of commands.

---

# Personal Reflection

If I repeated this lab today, I would make the process much more systematic.

I would:

1. Perform a full TCP port scan earlier if the initial scan produced little information.
2. Identify the service and version.
3. Test access controls before searching extensively for exploits.
4. Move immediately into Redis-specific enumeration once direct access was confirmed.

The technical solution itself was simple. The more valuable lesson was methodological.

**This lab taught me that good penetration testing is not about finding the most sophisticated exploit; it is about continuously testing hypotheses against evidence and choosing the simplest valid path to the objective.**
