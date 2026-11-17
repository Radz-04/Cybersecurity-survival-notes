# Information Gathering

Passive and active reconnaissance, DNS, web technologies, and network services.

**Author:** Radzivon Ferrigni Dioguardi

## Contents

1. [Method, scope, and reconnaissance](#1-method-scope-and-reconnaissance)
2. [Domain registration with RDAP and WHOIS](#2-domain-registration-with-rdap-and-whois)
3. [Infrastructure, hosting, and CDNs](#3-infrastructure-hosting-and-cdns)
4. [DNS structure and records](#4-dns-structure-and-records)
5. [Querying DNS and reading responses](#5-querying-dns-and-reading-responses)
6. [Email authentication with SPF, DKIM, and DMARC](#6-email-authentication-with-spf-dkim-and-dmarc)
7. [DNS enumeration and zone transfers](#7-dns-enumeration-and-zone-transfers)
8. [Subdomains and Certificate Transparency](#8-subdomains-and-certificate-transparency)
9. [DNS resolution and HTTP verification](#9-dns-resolution-and-http-verification)
10. [Web reconnaissance and HTTP semantics](#10-web-reconnaissance-and-http-semantics)
11. [Web metadata files, TLS, and fingerprinting](#11-web-metadata-files-tls-and-fingerprinting)
12. [Search engines and web archives](#12-search-engines-and-web-archives)
13. [theHarvester and account exposure](#13-theharvester-and-account-exposure)
14. [Nmap host discovery](#14-nmap-host-discovery)
15. [Nmap scan techniques, ports, and states](#15-nmap-scan-techniques-ports-and-states)
16. [Service and OS detection, NSE, and output](#16-service-and-os-detection-nse-and-output)

## Command conventions

The examples use Linux shell syntax. Replace these documentation placeholders with assets within the authorized scope. The commands illustrate usage and were provided without results from an actual scan.

```bash
DOMAIN='example.com'
HOST='www.example.com'
URL="https://$HOST"
TARGET='192.0.2.10'
NETWORK='192.0.2.0/24'
DNS_SERVER='192.0.2.53'
```

For tools that use external providers, available sources, quotas, and flags can change. Check the examples against the help output for the installed version.

## 1. Method, scope, and reconnaissance

Information gathering collects details about the systems under assessment, including domains, hostnames, addresses, services, and application components. Network discovery and service identification help characterize what is exposed within the scope. Web reconnaissance also examines public content, metadata, technologies, and application entry points.

*Footprinting* commonly refers to collecting information and using it to build a picture of the target. Describe each reconnaissance activity by the systems it contacts and the data it collects.

### Test scope

The scope defines the boundaries of the assessment. The rules of engagement specify permitted activities, operating limits, and conditions before testing starts.

| Item | What it covers |
| --- | --- |
| In-scope systems | Explicitly included domains, IP addresses, ranges, URLs, and applications. |
| Out-of-scope systems | Shared infrastructure, third-party services, and unauthorized assets. |
| Activities | Permitted techniques and restrictions on scanning, enumeration, and requests. |
| Operating conditions | Testing hours, traffic limits, source addresses, and points of contact. |

A link, DNS record, or redirect to an external service leaves those boundaries in place. Check newly discovered names against the agreed scope before contacting them directly.

### Passive and active reconnaissance

| Approach | Examples | Interaction |
| --- | --- | --- |
| Passive with respect to the target | Search indexes, web archives, and certificate logs. | The request goes to a third-party service. |
| Active | HTTP requests, authoritative DNS queries, host discovery, and scans. | The request reaches the infrastructure under assessment. |

Reading a search result uses the search engine. Opening the listed URL can contact the site itself. An OSINT tool may also include DNS resolution, HTTP checks, or other active modules.

## 2. Domain registration with RDAP and WHOIS

Registration data describes a domain as recorded by the registry and registrar. Published DNS records and website responses provide different kinds of information.

The Registration Data Access Protocol (RDAP) provides structured data over HTTP and mechanisms for locating the authoritative service. WHOIS is the older protocol and returns text in formats that can vary.

Since January 28, 2025, ICANN has identified RDAP as the definitive source for generic top-level domain (gTLD) registration data. Remaining WHOIS services and obligations depend on the registry and domain type. For country-code top-level domains (ccTLDs), check the relevant registry's guidance. [ICANN RDAP transition announcement](https://www.icann.org/en/announcements/details/icann-update-launching-rdap-sunsetting-whois-27-01-2025-en)

### Fields to look for

| Field | Meaning |
| --- | --- |
| Domain name | The registered domain name. |
| Registrar / IANA ID | The responsible registrar and its identifier, when published. |
| Events / dates | Available dates such as registration, update, and expiration. |
| Status | Administrative status codes for the domain. |
| Nameservers | DNS servers declared in the registration data. |
| Entities / contacts | Published entities and contacts, some of which may be redacted. |

ICANN Lookup provides access to RDAP data for supported domains. The traditional command remains useful where the queried WHOIS service supplies registration data.

```bash
whois "$DOMAIN"
```

### Administrative status codes

`clientTransferProhibited` indicates that the registrar has requested that domain transfers be rejected. It describes a transfer restriction. `clientHold` and `serverHold` affect publication in DNS and can prevent normal resolution.

A registrant's name, address, or email may be withheld under publication and data protection rules. Available technical information can still be examined. Identifying the web server operator requires evidence beyond the absence of registrant data.

## 3. Infrastructure, hosting, and CDNs

A website may rely on several providers for domain registration, DNS, hosting, CDN delivery, and application services. One organization can fill several of these roles.

Netcraft Site Report collects observations about technologies and infrastructure. Search DNS looks up hostnames in the service's dataset. The available fields depend on what the service has collected for that site.

### Reading infrastructure data

| Data | What it describes |
| --- | --- |
| IPv4 / IPv6 addresses | Published or observed endpoints for the name under review. |
| Netblock / provider | The organization associated with an address block or infrastructure. |
| Reverse DNS | The name returned by the address's PTR record. |
| Nameserver | A component of the DNS service, distinct from the HTTP service. |
| Hosting history | Historical observations, including the dates on which they were recorded. |
| Detected technologies | Matches inferred from observed evidence. |

### Proxies and origin addresses

When Cloudflare proxying is enabled for a supported record, DNS returns Cloudflare anycast addresses and HTTP or HTTPS traffic passes through Cloudflare's network. The returned address belongs to the intermediary and can differ from the origin address.

Cloudflare proxy status is configured per record. Its nameservers can serve both proxied and unproxied records. A provider name associated with an IP describes part of the infrastructure.

An address from a historical record may have been reassigned or may have stopped serving the site. Confirm current attribution and scope before contacting it directly.

### Certificates and web identifiers

Certificates can expose the issuer, validity period, and listed names. Web content may contain analytics identifiers or third-party resources. These details can help identify components and relationships. A shared provider or resource is insufficient evidence of common ownership.

## 4. DNS structure and records

The Domain Name System organizes a distributed, hierarchical namespace. A zone is an administered portion of that namespace and can delegate parts of the tree to other zones.

The system's stub resolver sends requests to a recursive resolver. The recursive resolver can answer from cache or query other servers. An authoritative server answers for the zones it serves. Public recursive resolvers use queries and caching to obtain records across the namespace.

A record's TTL limits how long it can be cached. Use it to interpret cache behavior.

| Type | Data or purpose |
| --- | --- |
| `A` | IPv4 address associated with the name. |
| `AAAA` | IPv6 address associated with the name. |
| `NS` | Authoritative nameserver for the zone or delegation. |
| `CNAME` | Canonical name for which the queried name is an alias. |
| `MX` | Mail exchanger and preference value. Individual mailboxes are outside the scope of the record. |
| `TXT` | One or more text strings used by protocols and verification mechanisms. |
| `SOA` | Zone authority data, including the server, contact, serial number, and timing parameters. |
| `PTR` | A referenced domain name, also used in reverse lookup zones. |
| `SRV` | Priority, weight, port, and target for a specific service. |
| `HINFO` | Declared CPU and operating system information. |

A name can have multiple addresses, and an alias can point to external infrastructure. Published DNS data and connection results are separate observations.

### HTTPS and SVCB records

RFC 9460 defines SVCB and HTTPS records, which publish information for locating services and selecting connection parameters. These records can supplement A and AAAA lookups during web reconnaissance. A site can support HTTPS even when its DNS has no HTTPS record.

```bash
dig "$DOMAIN" HTTPS
```

## 5. Querying DNS and reading responses

`dig` sends a query to the selected server and shows the response. When `@server` is omitted, it uses the DNS servers configured in the local environment.

```bash
dig "$DOMAIN" A
dig "$DOMAIN" AAAA
dig "$DOMAIN" NS
dig "$DOMAIN" MX
dig "$DOMAIN" TXT
dig "$DOMAIN" SOA
dig +short "$DOMAIN" A
dig -x "$TARGET"
dig @"$DNS_SERVER" "$DOMAIN" A
```

`+short` produces compact output. Keep the full output when examining the status, flags, and response sections. `-x` builds a reverse lookup query for the specified address.

### Response structure

| Item | What to read |
| --- | --- |
| `QUESTION` | The requested name, class, and type. |
| `ANSWER` | Records returned in response to the question. |
| `AUTHORITY` | Relevant authority or delegation information. |
| `ADDITIONAL` | Extra information that may help with resolution. |
| `SERVER` | The server that returned the response. |

### Response codes

| Code | Meaning |
| --- | --- |
| `NOERROR` | The server reported no error. The requested record type may still be absent. |
| `NXDOMAIN` | The response indicates that the requested name does not exist. |
| `SERVFAIL` | The server could not complete the query. |
| `REFUSED` | The server refused the operation. |

A timeout means the client received no response within the allotted time. A name that does not exist and an existing name that lacks a record of a particular type are different conditions.

Servers can return limited responses to `ANY` queries. Query the required record types explicitly for routine collection.

```bash
host "$DOMAIN"
host -t MX "$DOMAIN"
nslookup -type=MX "$DOMAIN"
```

## 6. Email authentication with SPF, DKIM, and DMARC

Mail-related DNS records describe routing, keys, and policies. Determining the authentication result for a specific message also requires data from that message and its delivery.

### SPF

Sender Policy Framework lets a domain authorize hosts to use its SMTP identity, such as the domain in `MAIL FROM`. SPF checks the SMTP sender or HELO identity. The visible `From` header is a separate field.

An SPF record is published as a TXT record. The receiver evaluates mechanisms according to the protocol. The `all` mechanism matches any source that reaches that point in the evaluation.

| Qualifier | SPF result when the mechanism matches |
| --- | --- |
| `+` / `+all` | Pass. |
| `-` / `-all` | Fail. |
| `~` / `~all` | Softfail. |
| `?` / `?all` | Neutral. |

The recipient applies its own rules to these results. `-all` specifies an SPF failure for matching sources. How a recipient handles a softfail, including whether it routes the message to spam, depends on its policy.

### DKIM

DomainKeys Identified Mail associates a signing domain with a message through a signature. The public key is located using the signing domain and selector from the signature's `d=` and `s=` tags. DKIM verifies the parts covered by the signature. Message encryption and proof of the human author's identity are separate functions.

```bash
dig "$DOMAIN" TXT
dig "_dmarc.$DOMAIN" TXT
# Replace selector with the selector used in the signature.
dig "selector._domainkey.$DOMAIN" TXT
```

### DMARC

DMARC validates use of the domain in the visible `From` header through a successful SPF or DKIM check aligned with that domain. A valid DKIM signature from an unrelated domain does not meet the alignment requirement on its own.

| Policy | Published preference |
| --- | --- |
| `p=none` | Expresses no message handling preference. |
| `p=quarantine` | Indicates that messages that fail validation should be treated as suspicious. |
| `p=reject` | Indicates that the use of the domain should be considered invalid for these failures. |

The receiver incorporates this information into its own handling decisions. The published policy, validation result, and final delivery decision are separate.

RFC 9989, published in May 2026, obsoletes RFCs 7489 and 9091. It removes the old `pct` tag and introduces `t` for testing mode. [RFC 9989](https://www.rfc-editor.org/info/rfc9989/)

## 7. DNS enumeration and zone transfers

Enumeration collects records and names using queries, external datasets, or dictionary-based guesses. DNSDumpster is a service for finding hosts and DNS relationships. Keep its dataset results attributable to that service when you verify them later.

### DNSRecon

DNSRecon supports general queries, SRV lookups, wildcard detection, brute-force enumeration, reverse lookups, and AXFR tests. The mode and flags determine which requests are made.

```bash
dnsrecon -h
dnsrecon -d "$DOMAIN" -t std
dnsrecon -d "$DOMAIN" -t axfr
dnsrecon -d "$DOMAIN" -D wordlist.txt -t brt
```

In the package help used for these notes, `std` collects SOA, NS, A, AAAA, MX, and SRV records. `axfr` attempts a transfer against the nameservers. `brt` uses the specified dictionary. `-a` adds AXFR to standard enumeration. Check the installed version before using additional options, since different modes generate different requests.

### DNSenum

DNSenum automates A, NS, and MX lookups, AXFR attempts, dictionary-based enumeration, and other checks depending on its options. It issues multiple queries and may carry out additional checks.

```bash
dnsenum --help
dnsenum --noreverse "$DOMAIN"
```

`--noreverse` disables the reverse lookup phase. Other checks may still run. The program can use a default dictionary even when `-f` is omitted, so consult the help for your distribution's package.

### AXFR and IXFR

AXFR transfers the full DNS zone over TCP. IXFR supports incremental transfers. Both distribute DNS data, and server rules determine access.

```bash
dig @"$DNS_SERVER" "$DOMAIN" AXFR
```

A successful AXFR returns the data in the transferred zone. An organization may have other zones or hosts outside it. A refusal or failure describes the outcome of that attempt. A timeout alone leaves the cause undetermined.

DNSSEC provides data origin authentication and integrity. Its protections concern the authenticity of DNS data. An unsigned zone lacks that protection. Establishing that spoofing occurred requires separate evidence.

## 8. Subdomains and Certificate Transparency

A subdomain is hierarchically below another name. It can be discovered through search indexes, DNS datasets, certificate logs, or queries built from a dictionary.

Certificate Transparency uses publicly verifiable logs to make TLS certificate issuance observable. Names in certificates can contribute to reconnaissance. A log entry records a certificate, and current DNS or HTTP behavior requires separate verification.

A wildcard certificate such as `*.example.com` covers a pattern without listing individual hostnames. Historical names may have fallen out of use. Check discovered names against the scope and current data.

### Sublist3r

Sublist3r collects subdomains from multiple sources and has options for brute-force enumeration and port checks. Its original README refers to older dependencies and Python versions. Check compatibility and provider availability on the actual installation.

```bash
python3 sublist3r.py -h
python3 sublist3r.py -d "$DOMAIN" -o subdomains.txt
```

`-b` enables brute-force enumeration. `-p` adds checks for the specified ports. These options go beyond querying external datasets.

### Subfinder

Subfinder is a passive subdomain enumeration tool. It queries configurable sources, some of which require API credentials and enforce quotas.

```bash
subfinder -version
subfinder -ls
subfinder -d "$DOMAIN" -silent -o candidates.txt
```

`-ls` lists available sources, and `-o` saves the output. To include source information in structured output, use the following command.

```bash
subfinder -d "$DOMAIN" -oJ -cs -o candidates.jsonl
```

`-all` uses all available sources. `-recursive` selects sources that can handle deeper subdomain levels. It limits source selection, so avoid assuming every discovered name will be recursively enumerated. `-active` and `-nW` add resolution checks to the passive collection workflow.

## 9. DNS resolution and HTTP verification

ProjectDiscovery's documentation shows how to pass collection results to other tools. Each stage answers a different question about a name, the DNS data returned for it, or the HTTP endpoint it may reach.

| Stage | Tool | Information collected |
| --- | --- | --- |
| Candidate names | Subfinder | Subdomains found in the queried sources. |
| Resolution | dnsx | DNS responses for the requested record types. |
| Web endpoints | httpx | HTTP or HTTPS responses and metadata. |

### dnsx

`dnsx` reads names from a file or standard input and queries DNS. It requests A records by default. Request AAAA records as well to include IPv6 addresses.

```bash
# in-scope.txt: names already checked against the scope.
dnsx -l in-scope.txt -a -aaaa -resp -json -o dns.jsonl
```

`-resp` includes the response, and `-json` produces JSONL. Other record types can be requested with flags such as `-cname` or `-mx`. The documentation distinguishes resolving a list with `-l` from brute-force enumeration with a domain and dictionary using `-d` and `-w`.

DNS wildcards can return responses for names that were never explicitly configured. `dnsx` provides wildcard filters. The `-wd` mode requires the domain, and its compatibility with ordinary record-type flags should be checked before combining them.

### ProjectDiscovery httpx

This section uses ProjectDiscovery's reconnaissance CLI named `httpx`. A Python library shares its name. The CLI sends HTTP or HTTPS requests to the supplied hosts or URLs.

```bash
httpx -l in-scope.txt -sc -title -td -ip -location \
  -json -o http.jsonl
```

`-sc` shows the status code, `-title` shows the page title, `-td` reports detected technologies, `-ip` reports the IP address, and `-location` shows the redirect destination declared in the response. `-json` saves one JSON object per result.

`-location` displays the `Location` value. Use `-fr` to follow HTTP redirects or `-fhr` to restrict redirect following to the same host. Separate probes can display details such as TLS or CDN information. Set `-rl` and `-t` for the rate and concurrency allowed by the engagement.

## 10. Web reconnaissance and HTTP semantics

Web reconnaissance examines URLs, content, responses, metadata, and components. Framework or server identification relies on observable evidence such as headers, cookies, page structure, and published resources.

### Requests with curl

```bash
curl -I "$URL/"
curl -i "$URL/"
curl -v "$URL/"
curl -sS -D headers.txt -o page.html "$URL/"
```

`-I` sends a HEAD request and returns headers. A GET request can also return a response body. `-i` displays headers and body, and `-v` adds diagnostic details. `-D` saves the headers, and `-o` saves the body. `-sS` hides the progress meter while retaining error messages.

`curl` follows HTTP redirects when `-L` is enabled. Check the `Location` value against the scope before following a redirect, since the destination may be different.

### Status codes and headers

| Item | Meaning |
| --- | --- |
| `200 OK` | The request succeeded. The meaning of the content depends on the method. |
| `301` / `302` | A redirect response. Examine `Location` and the destination. |
| `401 Unauthorized` | The request lacks valid credentials for the resource. |
| `403 Forbidden` | The server understood the request and refuses to fulfill it. |
| `404 Not Found` | The server found no representation or declines to disclose that it exists. |
| `Server` | Information declared about the software handling the response. |

A `403` can have several causes. Identifying a WAF requires more evidence. A `200` at a path named `wp-login.php` calls for examining the response before identifying WordPress. Headers and content can be customized, so corroborate technology matches using several observations.

### Page content

HTML comments, generator tags, scripts, and framework references can provide clues. The findings describe the content returned by the specific URL and request.

```bash
curl -sS "$URL/" | grep -i 'generator'
```

## 11. Web metadata files, TLS, and fingerprinting

### robots.txt and sitemaps

`robots.txt` publishes instructions for crawlers that follow the Robots Exclusion Protocol. A path listed under `Disallow` is visible in the file. Authentication and authorization are enforced separately.

A sitemap lists URLs and metadata for search engines. An index can group multiple sitemap files. Sitemap coverage and search engine indexing depend on how the site and search engine handle each URL.

```bash
curl -i "$URL/robots.txt"
curl -i "$URL/sitemap.xml"
```

The sitemap path shown is a common convention. A site can use a different location and may list its sitemaps in `robots.txt`.

### WhatWeb

WhatWeb identifies web technologies using plugins that analyze clues in responses. Its documented default aggression level is 1. Higher levels may send additional requests to refine matches.

```bash
whatweb --help
whatweb "$URL/"
```

A match can refer to an intermediary or the origin application. Precise version identification and vulnerability applicability require supporting evidence beyond a header or content match.

### wafw00f

`wafw00f` looks for WAF signatures in HTTP responses. The project describes sending an ordinary request first, then potentially malicious requests if needed, and analyzing the responses. This is an active check and may generate events in security systems.

```bash
wafw00f --help
wafw00f --list
wafw00f "$URL/"
```

`-v` increases verbosity, and `-a` looks for all matching signatures. A result with no match can still occur when a product or configuration is outside the tool's recognition coverage.

### Examining a TLS connection

```bash
openssl s_client -connect "$HOST:443" \
  -servername "$HOST" -verify_hostname "$HOST" \
  -verify_return_error </dev/null
```

SNI, set with `-servername`, supplies the requested hostname to the server. `-verify_hostname` checks that the certificate matches the hostname. `-verify_return_error` stops the connection on verification errors. The result also depends on the certificate authorities trusted by the local environment.

## 12. Search engines and web archives

Advanced queries narrow indexed results. Use them to find pages and documents the search engine knows about. The results reflect the search index and the query.

### Search operators documented by Google

| Operator | Use | Example |
| --- | --- | --- |
| `site:` | Limit results to a supported domain, URL, or prefix. | `site:example.com` |
| `filetype:` | Search for a file type. | `site:example.com filetype:pdf` |
| Quotation marks | Search for an exact phrase. | `site:example.com "security policy"` |
| `-` | Exclude a term. | `site:example.com -careers` |

The `site:` operator can omit URLs that are already in the index. Keep the operator and its value together, without a space.

Operators such as `inurl:`, `intitle:`, and `intext:` often appear in Google dork lists. Check their availability and behavior against current documentation. The examples here use operators found in Google's documentation.

Searching queries the search engine. Opening a link or downloading the document can contact the site itself. A missing result leaves the existence of the file or service unconfirmed.

### Wayback Machine

The Wayback Machine lets you search archived URLs and select dates or captures. The copies can show content, paths, and references from an earlier version of a website.

Archive coverage varies, and some pages, images, resources, or dates may be missing. A capture reflects the site at the time it was recorded. Verifying current content requires a separate observation.

While browsing an archive, pay attention to links that lead back to the live site. Capture dates document the provenance of archived content.

### Finding and collecting web content

OWASP includes examining comments, metadata, and JavaScript files in web information gathering. Inspecting available content may reveal components and additional paths. Treat these findings as clues to attribute and verify before drawing conclusions about vulnerabilities.

## 13. theHarvester and account exposure

theHarvester collects information from selected providers, including search engines, certificate logs, and public datasets. The data it returns depend on the sources and tool version. The project README documents running it with `uv` from the project directory.

```bash
uv run theHarvester -h
uv run theHarvester -d "$DOMAIN" -b crtsh,certspotter
uv run theHarvester -d "$DOMAIN" -b crtsh,certspotter \
  -f report
```

If the installed package provides an executable, use the direct command and check that package's help output.

```bash
theHarvester -h
```

### Sources, options, and output formats

`-d` selects the domain, `-b` selects the sources, `-l` sets the requested limit, and `-f` sets the output prefix. The project README describes JSONL output and compatibility reports in JSON and XML. Check the documentation for your version before relying on a particular format.

Sources may require API keys or impose quotas. Distinguish a provider that returns no results from one that reports an error. DNS resolution and direct verification add interactions beyond third-party lookups. Check the installed help before interpreting `-v` as a virtual host option.

### Accounts found in data breaches

Have I Been Pwned lets you check for exposure in known breaches. Domain searches require proof of control through the service's verification process. API access and available data can depend on the plan.

Breach records distinguish the incident name, breach date, date added, and affected data classes. Finding an email address in a breach establishes its association with that incident. Current password exposure or account control requires separate evidence.

Pwned Passwords checks passwords against a separate dataset. Its range query sends the first five characters of the hash so the returned suffixes can be checked locally. The plaintext password stays local. A match concerns the password dataset and cannot establish exposure of a specific email and password pair.

## 14. Nmap host discovery

Host discovery looks for hosts that respond to the selected probes. Port scanning and service identification are separate steps. Nmap can use ICMP Echo and other probe types, depending on privileges, the network, and the options selected.

```bash
nmap -sL -n "$NETWORK"
sudo nmap -sn "$NETWORK"
sudo nmap -sn -PE "$TARGET"
sudo nmap -sn -PS80,443 "$TARGET"
```

### Discovery options

| Option | Effect |
| --- | --- |
| `-sL` | Lists targets without checking whether they are up. |
| `-sn` | Performs host discovery without the ordinary port scan. |
| `-n` | Disables Nmap's reverse DNS lookups. A target supplied as a hostname still needs to be resolved to an address. |
| `-PE` | Uses ICMP Echo for discovery. |
| `-PS80,443` | Selects TCP SYN probes on the specified ports. |
| `-PA` | Selects TCP ACK probes, with behavior that depends on privileges. |
| `-Pn` | Skips ordinary host discovery and attempts the requested scan on every specified target. |

On local Ethernet, Nmap can use ARP for IPv4 and Neighbor Discovery for IPv6. Even with `-Pn`, it may still need to resolve local MAC addresses.

### Interpreting a lack of response

An unanswered probe can mean the host is down or that it was unreachable using that technique. With `-Pn`, Nmap proceeds with the requested scan without requiring a discovery response. Treat reachability as unconfirmed until supported by a response.

```bash
nmap -Pn -sT -p 80,443 "$TARGET"
```

Using `-Pn` on a large range can increase scan time substantially because the requested probes also go to addresses with no host.

### IPv6

Nmap's `-6` flag enables supported IPv6 features. Record which address family was tested, since services exposed over IPv6 can differ from those seen over IPv4.

```bash
nmap -6 -sT -p 80,443 2001:db8::10
```

`2001:db8::10` is a documentation address. Replace it with an authorized IPv6 target that is reachable from the testing environment.

## 15. Nmap scan techniques, ports, and states

### TCP and UDP scans

`-sT` uses the operating system's TCP connect call. `-sS` sends SYN probes and interprets the replies, typically without completing the TCP connection. It requires sufficient privileges to send raw packets. Both techniques generate observable traffic.

`-sU` scans UDP ports. A UDP response can indicate an open port. Some ICMP messages indicate that a port is closed or filtered. An unanswered probe can leave the state ambiguous.

```bash
nmap -sT -p 22,80,443 --reason "$TARGET"
sudo nmap -sS -p 22,80,443 "$TARGET"
sudo nmap -sU -p 53,123,161 "$TARGET"
```

### Port selection

By default, Nmap scans the 1,000 most common ports for the selected protocol. `-F` usually reduces that set to 100 ports. `--top-ports` uses frequency data from Nmap's database.

```bash
nmap -sT --top-ports 100 "$TARGET"
nmap -sT -p- "$TARGET"
sudo nmap -sU -p- "$TARGET"
```

`-p-` selects ports 1 through 65535 for the protocol being scanned. It applies to TCP with `-sT` and to UDP with `-sU`. Port 0 can be requested explicitly.

### The six Nmap port states

| State | Meaning |
| --- | --- |
| `open` | An application accepts connections or datagrams for the protocol. |
| `closed` | The port responds, and no application appears to be listening. |
| `filtered` | Filtering interferes with probes or replies, so Nmap cannot determine whether the port is open. |
| `unfiltered` | The port is reachable, and the scan method cannot distinguish open from closed. |
| `open\|filtered` | The scan method cannot distinguish open from filtered. |
| `closed\|filtered` | The scan method cannot distinguish closed from filtered. |

These states describe what Nmap observed from a particular network location using a particular technique. The same port can have a different state when checked from another network. Record `open|filtered` as an ambiguous result. An open port indicates an available service. Establishing a vulnerability requires separate evidence.

## 16. Service and OS detection, NSE, and output

### Service and version detection

The name in the `SERVICE` column can come from the port database. `-sV` sends application-layer probes to identify the protocol, product, and version when possible. Some services return too little information for a precise match.

```bash
nmap -sT -sV -p 22,80,443 "$TARGET"
```

Registered port assignments are conventions. IANA notes that traffic on an assigned port can use a different service. Identify the service from the observed communication.

| Port / protocol | Commonly associated service |
| --- | --- |
| 21/TCP | FTP control channel. |
| 22/TCP | SSH. |
| 25/TCP and 587/TCP | SMTP and message submission, respectively. |
| 53/UDP and 53/TCP | DNS. |
| 80/TCP and 443/TCP | HTTP and HTTPS, respectively. |
| 445/TCP | Microsoft-DS, commonly SMB. |
| 161/UDP | SNMP. |

### Operating system fingerprinting

`-O` compares features of TCP/IP responses with Nmap's fingerprint database. Conditions are better when Nmap finds at least one open and one closed TCP port. A match is an inference from responses, and an unmatched result leaves the operating system undetermined.

```bash
sudo nmap -O "$TARGET"
```

### Nmap Scripting Engine

The Nmap Scripting Engine (NSE) runs scripts for different purposes. `-sC` selects the `default` category. Review each script's documentation to understand its behavior and potential impact.

```bash
nmap --script-help http-title
nmap -sT -sV -p 80,443 --script http-title "$TARGET"
```

`http-title` retrieves the title of the default page and can follow redirects according to the HTTP library's rules. `-A` enables OS detection, version detection, script scanning, and traceroute. Port selection is handled separately. `-T4` selects a timing profile. Its suitability depends on the network.

### Saving scan output

```bash
mkdir -p scans
nmap -sT -sV -p 22,80,443 -oA scans/tcp "$TARGET"
```

`-oA` produces `.nmap`, `.xml`, and `.gnmap` files. `-oN file.txt` saves normal output, and `-oX file.xml` saves XML. Nmap has deprecated the grepable format. Use XML for structured processing.
