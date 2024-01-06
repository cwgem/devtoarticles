{%- # TOC start (generated with https://github.com/derlin/bitdowntoc) -%}

- [What Is A CVE](#what-is-a-cve)
- [CVE Format](#cve-format)
- [CVE Analysis](#cve-analysis)
   * [Is The Affected Software Present](#is-the-affected-software-present)
   * [Severity](#severity)
   * [Exploit Vector](#exploit-vector)
   * [Fix Version](#fix-version)
   * [Embargo](#embargo)
   * [Validation](#validation)
- [Having A Reporting Program](#having-a-reporting-program)
- [Conclusion](#conclusion)

{%- # TOC end -%}

CVEs (Common Vulnerabilities and Exposures) are an official collection of recognized security issues. Knowing these is valuable not just for security, but for other teams such as DevOps and development. A major problem is that some people don't know how to handle a CVE properly. This can lead to unnecessary panic and a loss of valuable company time. In this article I'll be going over what a CVE is and how to analyze one.

## What Is A CVE

The concept of CVE was [created back in 1999](https://www.cve.org/About/History) as a way to centralize vulnerability details. While there is a central governing organization, [MITRE](https://www.mitre.org/), actual vulnerability reporting is often handled by other parties. These are known as [CVE Number Authorities](https://www.cve.org/ProgramOrganization/CNAs) or CNAs. CNAs span a number of scopes which indicate what CVEs they can report against. For example, Adobe is authorized to publish a CVE for one of it's products, but would not be able to publish a CVE for the Python programming language. Some notable CNA examples:

- Adobe
- Apple
- Python Foundation
- Oracle

There are also root CNAs which handle organization of specific scopes of CNAs. Google is one example of a root CNA which covers organization of CVEs related to non Android and Chrome Google products.

## CVE Format

CVEs are currently published based off a [JSON format specification](https://www.cve.org/AllResources/CveServices#cve-json-5). Here's an example of an entry for CVE-2023-0038 hosted on the [CVE Database in GitHub](https://github.com/CVEProject/cvelist/blob/master/2023/0xxx/CVE-2023-0038.json):

```json
{
    "data_version": "4.0",
    "data_type": "CVE",
    "data_format": "MITRE",
    "CVE_data_meta": {
        "ID": "CVE-2023-0038",
        "ASSIGNER": "",
        "STATE": "PUBLIC"
    },
    "description": {
        "description_data": [
            {
                "lang": "eng",
                "value": "The \"Survey Maker \u2013 Best WordPress Survey Plugin\" plugin for WordPress is vulnerable to Stored Cross-Site Scripting via survey answers in versions up to, and including, 3.1.3 due to insufficient input sanitization and output escaping. This makes it possible for unauthenticated attackers to inject arbitrary web scripts when submitting quizzes that will execute whenever a user accesses the submissions page."
            }
        ]
    },
    "problemtype": {
        "problemtype_data": [
            {
                "description": [
                    {
                        "lang": "eng",
                        "value": "CWE-79 Improper Neutralization of Input During Web Page Generation ('Cross-site Scripting')"
                    }
                ]
            }
        ]
    },
    "affects": {
        "vendor": {
            "vendor_data": [
                {
                    "vendor_name": "ays-pro",
                    "product": {
                        "product_data": [
                            {
                                "product_name": "Survey Maker \u2013 Best WordPress Survey Plugin",
                                "version": {
                                    "version_data": [
                                        {
                                            "version_value": "*",
                                            "version_affected": "="
                                        }
                                    ]
                                }
                            }
                        ]
                    }
                }
            ]
        }
    },
    "references": {
        "reference_data": [
            {
                "url": "https://www.wordfence.com/threat-intel/vulnerabilities/id/a2a58fab-d4a3-4333-8495-e094ed85bb61",
                "refsource": "MISC",
                "name": "https://www.wordfence.com/threat-intel/vulnerabilities/id/a2a58fab-d4a3-4333-8495-e094ed85bb61"
            },
            {
                "url": "https://plugins.trac.wordpress.org/browser/survey-maker/tags/3.1.4/public/partials/class-survey-maker-submissions-summary-shortcode.php?rev=2839688#L311",
                "refsource": "MISC",
                "name": "https://plugins.trac.wordpress.org/browser/survey-maker/tags/3.1.4/public/partials/class-survey-maker-submissions-summary-shortcode.php?rev=2839688#L311"
            }
        ]
    },
    "credits": [
        {
            "lang": "en",
            "value": "Chloe Chamberland"
        }
    ],
    "impact": {
        "cvss": [
            {
                "version": "3.1",
                "vectorString": "CVSS:3.1/AV:N/AC:L/PR:N/UI:N/S:C/C:L/I:L/A:N",
                "baseScore": 7.2,
                "baseSeverity": "HIGH"
            }
        ]
    }
}
```

The basic breakdown is:

- A description of the CVE
- Type of vulnerability (cross site scripting)
- What is affected
- References (such as vendor postings and bug reports)
- Credit for report
- Overall impact

These are sufficient data points to have a basic idea of if one is affected by a vulnerability.

## CVE Analysis

Since we have the information at hand on what a vulnerability is about it's time to analyze it for potential impact. One thing to keep in mind is that each CVE has a set of circumstances to be considered vulnerable. That means specific setups may not be impacted by it, and that upgrades should be handled in an informed manner to reduce unnecessary upgrades which bring unnecessary risk with it.

### Is The Affected Software Present

First off is seeing if the software in question is part of your infrastructure. Server software such as nginx are generally easier to tell. In corporate environments there may be a vulnerability scanning solution in place (or something such as [dependabot](https://github.blog/2023-01-12-a-smarter-quieter-dependabot/)). There could also be some kind of software inventory system in place as well. If you're in a corporate environment and don't have a system in place, I highly recommend looking into it.

### Severity

Next to consider is the severity, which is an accumulated score system known as CVSS (Common Vulnerability Scoring System). Calculation of this is based on a number of properties of the exploit, some including:

- Is a proof of concept exploit available?
- What level of permissions are needed for the exploit?
- Is an official fix available?
- Is network access required?

and so forth. NIST (National Institute of Standards and Technology) has a [calculator with each of the properties explained](https://nvd.nist.gov/vuln-metrics/cvss/v3-calculator). The resulting score will indicate how severe the exploit is. For example on the previous wordpress plugin CVE:

```json
    "impact": {
        "cvss": [
            {
                "version": "3.1",
                "vectorString": "CVSS:3.1/AV:N/AC:L/PR:N/UI:N/S:C/C:L/I:L/A:N",
                "baseScore": 7.2,
                "baseSeverity": "HIGH"
            }
        ]
    }
```

The attack vector is network based, complexity is low, no permissions are required, and there is no user interaction required. This puts it at a higher severity due to this. Even so the resulting impact is fairly low to critical components such as system availability and integrity. Most security teams will want high and above vulnerabilities handled in a very short time frame. Lower severity indicates the ability to pull off the exploit might not be feasible or apply to your particular configuration.

### Exploit Vector

Now that severity has been analyzed the next step is to look into how this particular vulnerability is exploited. This is important in cases of highly configurable software such as application servers where features can be enabled/disabled at will. Lets take for example [CVE-2022-41741](https://github.com/CVEProject/cvelist/blob/master/2022/41xxx/CVE-2022-41741.json)'s description text:

> NGINX Open Source before versions 1.23.2 and 1.22.1, NGINX Open Source Subscription before versions R2 P1 and R1 P1, and NGINX Plus before versions R27 P1 and R26 P1 have a vulnerability in the module ngx_http_mp4_module that might allow a local attacker to corrupt NGINX worker memory, resulting in its termination or potential other impact using a specially crafted audio or video file. The issue affects only NGINX products that are built with the ngx_http_mp4_module, when the mp4 directive is used in the configuration file. Further, the attack is possible only if an attacker can trigger processing of a specially crafted audio or video file with the module ngx_http_mp4_module.

In this case you'll only be affected if you're using the mp4 module in nginx and the system is allowed to process user input media files. That means if you aren't working with the plugin there is no rush to implement a fix. There also may be environmental aspects to a vulnerability. [CVE-2023-5528](https://github.com/CVEProject/cvelist/blob/master/2023/5xxx/CVE-2023-5528.json) for example only affects kubernetes nodes running on Windows systems. Ensuring you understand exploit vectors can help avoid unnecessary prioritization that could put business value add tasks on hold.

### Fix Version

This is one of the areas where CVE scanning may show its true colors. Taking a CVE at face value, you might think you simply update to the latest version and then it's done. It's not that simple however. The version you're currently on and the version to upgrade two may contain changes that are not related to the vulnerability at all. This means introducing additional risk into systems in case another non-security bug is present. Due to this it's not uncommon for vendors to implement backports. This is where they take the patch and make it work for a previous version of the package. This helps reduce the introduction of risk into systems. For example, if we look at [CVE-2023-46724](https://github.com/CVEProject/cvelist/blob/master/2023/46xxx/CVE-2023-46724.json):

```json
{
    "product_name": "squid",
    "version": {
        "version_data": [
            {
                "version_affected": "=",
                "version_value": ">= 3.3.0.1, < 6.4"
            }
        ]
    }
}
```

The affected versions are greater than `3.3.0.1` and less than `6.4`. However, if we look at the [RedHat security advisory](https://access.redhat.com/security/cve/CVE-2023-46724) you'll notice that the fixed package is "squid:4" or more specifically `squid-4.15-7.module+el8.9.0+20975+25f17541.5.x86_64.rpm`. Due to RedHat Linux being targeted to slower moving enterprise customers, it's within their business interest to backport instead of updating to the mentioned version 6.4 in the original report. This is why when analyzing a CVE you need to understand where your source of the package is, for example:

- Your distribution
- Part of a container or virtual machine image
- Part of a language's package manager
- Manual installation via automation such as AWS Systems Manager or Ansible

It is important to note that you may not always have the luxury of a backport available. In such cases you can attempt to do it yourself, or just deal with the added risk of unrelated changes (going over the changelog of a package is a good idea). 

### Embargo

This is a special case for CVEs which have an extremely large number of affected systems and require coordinated effort to push a release out. OpenSSL is where you'd be likely to see an embargo occur (this happened somewhat recently with [CVE-2022-3786](https://github.com/CVEProject/cvelist/blob/master/2022/3xxx/CVE-2022-3786.json)). This gives time for vendors to upgrade systems / packages (such as Microsoft for example) and allows companies to ensure proper staffing to deal with the issue. The primary purpose of all of this is to prevent the possibility of exploits out in the wild causing a war room situation. That's not to say an exploit won't show up, but the less the chance the better.

### Validation

Now assuming you are affected it's time to take action. I will note that some extremely serious vulnerabilities (such as embargo'ed ones) you may not have the luxury of full validation due to the time sensitive nature of it. How you achieve a package update will depend on your particular setup and the source of your packages. Depending on the impact of updating packages it may require a scheduled maintenance window. Before deployment though it's recommended to do some validation. If the report has a proof of concept it's possible to use this to validate that your upgrade will work. You should only use this to validate if the PoC has no destructive operations such as deleting files or DDoS-ing a server. I recommend doing an upgrade and run of the PoC **ON A NON PRODUCTION DEV/TEST SYSTEM**. If you're hosting on a cloud provider such as AWS you might need permission ahead of time depending on how the PoC operates.

You'll also want to do system integrity validation with the new version. This is where you do the upgrade, then run your test suites (or cry because you don't have any) to ensure nothing will break. A broken server can sometimes be seen as worse as a vulnerable one. In some cases you may need to report back to the original bug report where the issue was found. Once everything looks good you can look towards moving it to productions. Blue/Green deployment is another method of helping ensure everything is good by doing a test deployment with validation to ensure critical applications are available.

## Having A Reporting Program

While I've covered how to analyze CVEs there are cases where software you work on may be the target of a vulnerability. The first step is having a dedicated channel for reporting potential security vulnerabilities. It should be highly monitored and if possible initial response made within the same business day. Always keep in communication with the reporter. Failure to do so mean they could potentially release their vulnerability findings (with a potential exploit) before you have a time to fix it. If you expect that security reports may be somewhat more common it might be a good idea to become a CNA and report CVEs on your own.

Some organizations even go so far as to provide financial incentive to report security issues through bug bounty programs. The Department Of Homeland Security has a [bug bounty program](https://bugcrowd.com/dhs-vdp) for example. In some cases you may see amounts anywhere from $10,000 - $20,000 depending on the impact of the security vulnerability. Whether or not an organization needs a bug bounty program very much will depend on how high value of a target it is (a mom and pop shop's online store is probably not a good case for one). 

## Conclusion

I hope this article has provided insight on how CVEs are formatted, and what information you can get out of them. Being able to understand the potential impact of a CVE can go a long way in ensuring the relevant fixes are properly prioritized, and it's not putting crucial project work at risk unnecessarily. I highly recommend understanding how CVEs work even if you're not in a security specific role. It can really help if you need to implement the fixes yourself.