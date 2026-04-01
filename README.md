# How to Secure Your Git-Ass

A comprehensive guide to identifying and dismantling targeted phishing campaigns and social engineering attacks on GitHub.

---

## Table of Contents

1. [The Threat Landscape](#1-the-threat-landscape)
2. [Case Study: The Uniswap Job Scam](#2-case-study-the-uniswap-job-scam)
3. [Common Red Flags](#3-common-red-flags)
4. [Technical Tactics: Living off the Land](#4-technical-tactics-living-off-the-land)
5. [Forensic Workflow: Analyze Without Risk](#5-forensic-workflow-analyze-without-risk)
   - 5.1 [URL Analysis via Sandbox](#51-url-analysis-via-sandbox)
   - 5.2 [Command Line Inspection](#52-command-line-inspection)
   - 5.3 [Infrastructure Inspection](#53-infrastructure-inspection)
6. [Active Countermeasures: Reporting and Takedown](#6-active-countermeasures-reporting-and-takedown)
   - 6.1 [Report the Redirect to Google](#61-report-the-redirect-to-google)
   - 6.2 [Contact the Hosting Provider](#62-contact-the-hosting-provider)
   - 6.3 [Inform the Domain Registrar](#63-inform-the-domain-registrar)
   - 6.4 [Report GitHub Abuse](#64-report-github-abuse)
7. [Harden Your GitHub Notification Settings](#7-harden-your-github-notification-settings)
8. [Conclusion](#8-conclusion)
9. [License](#9-license)

---

## 1. The Threat Landscape

Developers on GitHub are increasingly targeted by highly professional social engineering campaigns. These attacks exploit two things attackers know about most developers: they are active on GitHub, and they are open to new opportunities.

The playbook is simple and effective. Attackers create a Discussion or Issue in a public repository, tag dozens of developers simultaneously, and impersonate well-known companies. The goal is to get you to click a link that either harvests your credentials or delivers malware.

What makes these campaigns dangerous is not technical sophistication on the victim's end — it is the deliberate abuse of trusted infrastructure. The attacker does not need to compromise GitHub or Google. They just use them as a delivery vehicle.

---

## 2. Case Study: The Uniswap Job Scam

The following is a real example of a phishing message distributed via GitHub Discussions. It is reproduced here for educational purposes.

```
Hi,
Your latest projects on GitHub got our attention. We are growing Uniswap
and searching for devs with skills match ours.

All positions are fully remote. Annual salary is paid in USD.

Available roles & pay:
  - Engineering: Staff BE, FE, SC, Infrastructure — up to $450k
  - Product & Design: Product Manager, Sr. Design, Design Engineer — up to $350k
  - Business & Ops: BizDev, Partners, Community Ops, Hiring, Solutions — up to $300k
  - Marketing: Dev Relations, Tech Writer, Content Eng — up to $300k

Next Instructions:
Fill out this form here: https://share.google/4fxOM8PJAd0y9tQ5R

[...]

This outreach is not random. If you find your GitHub handle here, we are
reaching out because your profile fits our roles:

@lihuang-hacker @VolkanSah @litprome @Gizmo-Verindipencil @stonding
@kushfintech @rasmustaarnby @XozXa @yigex @cakeca @QingChuanAIX
@Schrodingerp @BramVanPevenage @carpeJay @chrisspx @Nandaxy @pwoit
[...]
```

This message was delivered via a GitHub notification because the account was mentioned in a Discussion. The link points to `share.google`, a legitimate Google domain used for Drive file sharing, which causes most automated phishing filters to pass it without warning. The actual destination is an attacker-controlled server.

---

## 3. Common Red Flags

**Unrealistic compensation without process**

Legitimate companies, including Uniswap, do not offer up to $450,000 annually to unknown candidates without a prior interview, technical assessment, or any formal application. If the offer sounds too good to require due diligence, that is by design.

**Mass tagging**

Real recruiters contact candidates individually. A single notification that tags 30 to 40 GitHub accounts simultaneously is a bulk spam operation. The "selected profiles" framing is a social engineering technique to make mass outreach feel personal.

**Trusted domain as a redirect shield**

The link uses `share.google`, which is a genuine Google service for sharing Drive content. Because the domain is `google.com`, security filters and even experienced developers may not question it. This is the core technical trick: the attacker owns none of the infrastructure in the message itself. Google, GitHub, and the notification system are all legitimate. Only the final destination is malicious.

**Urgency and flattery combined**

"This outreach is not random" and "your profile fits our roles" are classic social engineering phrases designed to bypass skepticism by appealing to professional ego.

**No verifiable sender identity**

The message contains no named recruiter, no company email address, and no link to a verifiable job posting on the official Uniswap website or careers page.

---

## 4. Technical Tactics: Living off the Land

The term "Living off the Land" (LotL) originates in offensive security and describes the practice of using legitimate tools and infrastructure already present in an environment rather than deploying custom malware. Attackers apply the same concept to phishing infrastructure.

In this campaign, the chain looks like this:

```
GitHub Discussion (legitimate)
    --> share.google link (legitimate Google domain)
        --> Redirect to attacker VPS (e.g., DigitalOcean, Hetzner, Vultr)
            --> Phishing form or malware delivery
```

Each hop uses a service with a strong reputation. Only the final server is attacker-controlled. This structure is designed to defeat:

- Email and link scanners that check only the top-level domain
- Automated reputation systems that trust major cloud providers
- Users who have been trained to check whether a link goes to a "real" company domain

The attacker typically registers either a fresh throwaway domain or an aged domain — one that was registered years ago and has built up a clean reputation score — specifically to bypass domain-age filters in security tools.

**Why `share.google` works so well**

`share.google` is the domain Google uses for its file sharing functionality. A link like `https://share.google/xxxxxx` is indistinguishable at the domain level from a legitimate shared document. No browser will warn you. No corporate firewall will block it by default. The redirect to the malicious destination happens server-side, often with a short delay or only under certain user-agent conditions to further complicate automated analysis.

---

## 5. Forensic Workflow: Analyze Without Risk

Never click a suspicious link directly. The following workflow lets you analyze the full redirect chain, inspect the hosting infrastructure, and gather evidence for reporting — all without exposing your machine or credentials.

### 5.1 URL Analysis via Sandbox

Use a remote analysis service. These tools visit the URL on their own isolated infrastructure and return a full report including screenshots, network requests, and the final destination.

Recommended services:

- [urlscan.io](https://urlscan.io) — shows the effective URL after all redirects, a screenshot of the final page, and all network requests made during the visit
- [VirusTotal](https://www.virustotal.com) — aggregates results from dozens of security vendors

What to look for in the report:

- **Effective URL**: The actual destination after all redirects. If a `share.google` link ends up at `erm.mmuszynski.com` or a bare IP address, that is your confirmation.
- **Screenshot**: urlscan.io renders the final page visually. You can see the phishing form without ever loading it yourself.
- **Certificate issue date**: An SSL certificate issued within the last few days is a strong indicator of a throwaway campaign domain.
- **Hosting ASN**: Check whether the server is hosted on a consumer VPS provider (DigitalOcean, Hetzner, Vultr, Linode). Legitimate company infrastructure is typically hosted on enterprise providers or corporate data centers.

### 5.2 Command Line Inspection

For a quick first check without opening a browser, use `curl` to inspect the redirect without following it:

```bash
curl -sI --max-redirs 0 "https://share.google/4fxOM8PJAd0y9tQ5R"
```

The `-I` flag fetches headers only (no body). `--max-redirs 0` tells curl to stop at the first redirect. The `Location:` header in the response will show you where the link actually points before you follow it.

Example output:

```
HTTP/2 302
location: https://erm.mmuszynski.com/apply
...
```

If the Location header points anywhere other than an official company domain, stop and treat the link as malicious.

To inspect the DNS records of the destination domain:

```bash
dig +short erm.mmuszynski.com
whois $(dig +short erm.mmuszynski.com | head -1)
```

This gives you the registrar, registration date, and often the abuse contact directly.

### 5.3 Infrastructure Inspection

Once you have the destination IP address from the urlscan report or from `dig`:

- Look up the ASN and hosting provider at [ipinfo.io](https://ipinfo.io) or [bgp.he.net](https://bgp.he.net)
- Cross-reference the SSL certificate at [crt.sh](https://crt.sh) — search by domain to see when the certificate was issued and whether other domains share the same certificate (this can reveal the full scope of a campaign)
- Check domain registration date at [whois.domaintools.com](https://whois.domaintools.com)

Document everything: IP address, hosting provider, registrar, certificate fingerprint, and the full redirect chain. You will need this for the reports in the next section.

---

## 6. Active Countermeasures: Reporting and Takedown

Understanding the attack is only half the work. To actually stop the campaign, each link in the infrastructure chain must be reported to the responsible party. The goal is to get the redirect taken down at Google, the server taken offline by the hosting provider, and the domain suspended by the registrar.

### 6.1 Report the Redirect to Google

Google operates the Safe Browsing program and accepts abuse reports for malicious content hosted on or redirecting through Google infrastructure.

Report URL: https://safebrowsing.google.com/safebrowsing/report_phish/

In your report, include:
- The full `share.google` link
- The final destination URL from your sandbox analysis
- A short description: "This Google Share link redirects to a phishing page impersonating [company]. The link is being distributed via GitHub to target developers."

### 6.2 Contact the Hosting Provider

This is typically the fastest and most effective takedown vector. Hosting providers take abuse seriously because their IP ranges can be blocklisted globally if they tolerate malicious tenants.

Steps:

1. Identify the hosting provider from the IP lookup (e.g., DigitalOcean, Hetzner, Vultr)
2. Find their abuse contact — most providers list it at `abuse@provider.com` or in their WHOIS record
3. Send an email with the following information:
   - Subject: `Phishing Campaign Hosted on Your Infrastructure`
   - The malicious IP address
   - The malicious URL
   - The original attack vector (GitHub Discussion link)
   - A brief description of the campaign and its targets
   - Your sandbox analysis report URL (urlscan.io link)

Most major providers respond within 24 hours and will null-route or suspend the offending server.

Common abuse contacts:

| Provider | Abuse Contact |
|---|---|
| DigitalOcean | abuse@digitalocean.com |
| Hetzner | abuse@hetzner.com |
| Vultr | abuse@vultr.com |
| OVH | abuse@ovh.net |
| Linode / Akamai | abuse@linode.com |
| AWS | abuse@amazonaws.com |

### 6.3 Inform the Domain Registrar

Even if the hosting provider takes the server offline, the domain itself can be re-pointed to a new server within minutes. Reporting to the registrar can result in the domain being suspended entirely.

Steps:

1. Find the registrar via WHOIS (`whois domain.com | grep Registrar`)
2. Use the registrar's abuse form or email their abuse department
3. Common registrars and their abuse contacts:

| Registrar | Abuse Contact |
|---|---|
| Namecheap | abuse@namecheap.com |
| GoDaddy | abuse@godaddy.com |
| Tucows / OpenSRS | abuse@tucows.com |
| Porkbun | abuse@porkbun.com |
| Cloudflare Registrar | abuse@cloudflare.com |

Report the domain as "Fraud / Phishing" and attach the same evidence package as above. Domain suspensions typically take 24 to 72 hours but effectively kill the entire campaign infrastructure.

### 6.4 Report GitHub Abuse

Report the Discussion or Issue directly on GitHub to remove the notification for all tagged users and flag the attacker's account.

Steps:

1. Open the Discussion or Issue containing the phishing message
2. Click the three-dot menu on the post
3. Select "Report content"
4. Choose "Spam or phishing"

Additionally, report the account that created the post:

1. Navigate to the attacker's GitHub profile
2. Click the block/report button
3. Select "Report abuse" and choose "Spam"

GitHub's Trust & Safety team reviews these reports and can remove the content, block the account, and retroactively clean up notifications for all mentioned users.

---

## 7. Harden Your GitHub Notification Settings

Prevention is more efficient than forensics. These settings reduce your exposure to this class of attack.

**Disable automatic repository watching**

By default, GitHub watches repositories you interact with and notifies you of all activity. This makes you visible to any attacker who posts in a repo you have touched.

Go to: `github.com/settings/notifications`

Under "Automatic watching", disable "Automatically watch repositories".

**Restrict who can mention you**

GitHub does not currently allow you to block mentions from unknown users entirely, but you can reduce noise by setting notification routing to only your verified email and reviewing which organizations you are publicly listed in.

**Use a notification filter**

In your email client or GitHub notification inbox, create a filter that flags any notification where you are mentioned alongside more than two or three other users in the same message. Legitimate mention-based notifications are almost always singular.

**Check your public profile exposure**

Attackers scrape GitHub to build target lists. Consider whether your email address, organization memberships, and contribution activity need to be fully public. Reducing your public footprint does not hide your work — it just raises the cost of targeting you specifically.

---

## 8. Conclusion

Security starts with skepticism. If a link appears safe because it uses a trusted domain, and the offer attached to it is too good to require any real vetting, the attacker is relying on exactly that combination to bypass your defenses.

The `share.google` technique is effective precisely because it is technically correct. The domain is owned by Google. The link does resolve. The SSL certificate is valid. None of that means the destination is safe.

By analyzing the redirect chain before clicking, inspecting the hosting infrastructure, and reporting each layer — platform, provider, and registrar — you protect not only your own system but reduce the attack surface for every other developer on the target list.

If you received one of these messages: you are not special, you were just on a list. Report it and move on.

---

## 9. License

This guide is released under the [MIT License](LICENSE). You are free to use, adapt, and redistribute it. If you improve it, consider opening a pull request.

---

*Contributions welcome. If you encounter a new campaign variant or a better analysis technique, open an issue or submit a PR.*
