# 🎯 Scenario 2: Bug Bounty — Web Application Reconnaissance

> **Real-World Context**: You've signed up for a bug bounty program on HackerOne/Bugcrowd.
> The target is a SaaS company called "CloudApp" with scope: `*.cloudapp.io`.
> Your goal is to find hidden endpoints, forgotten subdomains, and exposed sensitive data.

---

## 📖 The Storyline

CloudApp is a cloud-based project management tool. Their bug bounty scope says:
- ✅ **In Scope**: `*.cloudapp.io`, `api.cloudapp.io`, mobile app APIs
- ❌ **Out of Scope**: `blog.cloudapp.io`, third-party services, social engineering

You need to find as many valid targets as possible before you start testing for vulnerabilities.

---

## 🔄 The Process (Step-by-Step)

### Phase A — Subdomain Discovery (Expand Your Target List)

> The first rule of bug bounty: **more subdomains = more attack surface = more bugs.**

#### Step 1: Multi-Source Subdomain Enumeration

```bash
# 1. Certificate Transparency Logs
curl -s "https://crt.sh/?q=%25.cloudapp.io&output=json" | \
  jq -r '.[].name_value' | sort -u | tee ct_subs.txt

# 2. theHarvester (aggregates multiple sources)
theHarvester -d cloudapp.io -b google,bing,crtsh,dnsdumpster -l 500 -f harvester_results

# 3. Subfinder (passive, uses many sources)
subfinder -d cloudapp.io -all -o subfinder_results.txt

# 4. Amass (most comprehensive)
amass enum -passive -d cloudapp.io -o amass_results.txt

# 5. Combine all results and deduplicate
cat ct_subs.txt subfinder_results.txt amass_results.txt | sort -u > all_subdomains.txt
echo "[+] Total unique subdomains: $(wc -l < all_subdomains.txt)"
```

#### Step 2: Resolve & Filter Live Subdomains

```bash
# Check which subdomains are actually alive
# httpx — probe URLs and check if they respond
cat all_subdomains.txt | httpx -silent -status-code -title -tech-detect -o live_subs.txt

# Alternative: massdns for fast DNS resolution
massdns -r resolvers.txt -t A -o S all_subdomains.txt > resolved.txt

# Filter only the ones that have web servers
cat live_subs.txt | grep "200\|301\|302\|403" | tee interesting_subs.txt
```

**Example Findings**:
```
https://app.cloudapp.io [200] [CloudApp - Dashboard] [React,Nginx]
https://api.cloudapp.io [200] [API Documentation] [Express,Node.js]
https://staging.cloudapp.io [403] [403 Forbidden] [Nginx]
https://admin.cloudapp.io [302] [Redirect to login] [PHP,Apache]
https://old.cloudapp.io [200] [CloudApp v1.0] [Django,Python]
https://graphql.cloudapp.io [200] [GraphQL Playground] [Node.js]
https://jenkins.cloudapp.io [200] [Jenkins Dashboard] [Java]
```

> 🚨 `old.cloudapp.io` (legacy version) and `graphql.cloudapp.io` (GraphQL playground) are GOLDMINE targets!

---

### Phase B — Web Application Fingerprinting

#### Step 3: Technology Stack Identification

```bash
# For each live subdomain, identify the tech stack
whatweb https://app.cloudapp.io -v
whatweb https://api.cloudapp.io -v

# Check HTTP headers (often leak info)
curl -sI https://app.cloudapp.io
# Look for: Server, X-Powered-By, X-AspNet-Version, X-Generator

# Wappalyzer (browser extension) — visit each subdomain
# Records: CMS, frameworks, analytics, CDN, etc.
```

#### Step 4: JavaScript File Analysis

```bash
# Find JS files — they often contain API endpoints, secrets, and comments
# Using gau (get all URLs from known sources)
echo "app.cloudapp.io" | gau | grep "\.js$" | sort -u > js_files.txt

# Download and search for secrets
while read url; do
  curl -s "$url" >> all_js.txt
done < js_files.txt

# Search for juicy patterns
grep -iE "api[_-]?key|secret|password|token|aws|firebase" all_js.txt
grep -iE "https?://[a-zA-Z0-9._-]+\.cloudapp\.io" all_js.txt  # Hidden API endpoints
grep -iE "/api/v[0-9]" all_js.txt  # API versioning
```

**What you might find in JS files**:
```javascript
// Common goldmine findings:
const API_BASE = "https://internal-api.cloudapp.io/v2";  // Hidden API!
const FIREBASE_KEY = "AIzaSy...";                         // Exposed key!
// TODO: Remove debug endpoint before production
const DEBUG_URL = "/api/debug/users";                     // Debug endpoint!
```

#### Step 5: Directory & File Discovery

```bash
# Gobuster with multiple wordlists for each target
gobuster dir -u https://app.cloudapp.io \
  -w /usr/share/wordlists/SecLists/Discovery/Web-Content/raft-medium-directories.txt \
  -x php,asp,aspx,jsp,html,js,json,txt,xml,bak,old,conf,env \
  -t 50 -o gobuster_app.txt

# Check for common sensitive files
TARGETS=(
  "https://app.cloudapp.io/.env"
  "https://app.cloudapp.io/.git/config"
  "https://app.cloudapp.io/wp-config.php.bak"
  "https://app.cloudapp.io/server-status"
  "https://app.cloudapp.io/.DS_Store"
  "https://app.cloudapp.io/debug"
  "https://app.cloudapp.io/phpinfo.php"
  "https://app.cloudapp.io/api/swagger.json"
  "https://app.cloudapp.io/graphql"
  "https://app.cloudapp.io/.well-known/security.txt"
)

for target in "${TARGETS[@]}"; do
  STATUS=$(curl -s -o /dev/null -w "%{http_code}" "$target")
  echo "$STATUS $target"
done
```

---

### Phase C — API Discovery & Mapping

#### Step 6: API Endpoint Enumeration

```bash
# If you found API docs (Swagger/OpenAPI)
curl https://api.cloudapp.io/swagger.json | jq '.paths | keys'
curl https://api.cloudapp.io/api-docs

# GraphQL introspection (if GraphQL endpoint exists)
curl -X POST https://graphql.cloudapp.io/graphql \
  -H "Content-Type: application/json" \
  -d '{"query": "{ __schema { types { name fields { name } } } }"}'

# Fuzz API endpoints
gobuster dir -u https://api.cloudapp.io/api/v1 \
  -w /usr/share/wordlists/SecLists/Discovery/Web-Content/api/api-endpoints.txt \
  -t 30
```

#### Step 7: Parameter Discovery

```bash
# Arjun — find hidden GET/POST parameters
arjun -u https://app.cloudapp.io/search
arjun -u https://api.cloudapp.io/api/v1/users

# Param Miner (Burp Suite extension)
# 1. Right-click request → Extensions → Param Miner → Guess params
# 2. Check for hidden parameters that affect behavior
```

---

### Phase D — Wayback Machine & Historical Data

#### Step 8: Find Forgotten/Removed Content

```bash
# Get all historical URLs from Wayback Machine
echo "cloudapp.io" | gau --subs | sort -u > wayback_urls.txt

# Filter for interesting extensions
cat wayback_urls.txt | grep -iE "\.(php|asp|aspx|jsp|json|xml|conf|env|bak|sql|log)" > interesting_urls.txt

# Check if old endpoints still work
while read url; do
  STATUS=$(curl -s -o /dev/null -w "%{http_code}" "$url")
  if [ "$STATUS" != "404" ]; then
    echo "[ALIVE] $STATUS $url"
  fi
done < interesting_urls.txt
```

**Why Wayback matters**: Companies remove pages from their site but forget to delete the files from the server. Old API endpoints, admin panels, and debug pages often still exist.

---

## 📝 Bug Bounty Recon Checklist

```
☐ Subdomain enumeration (3+ tools)
☐ DNS resolution & live host filtering
☐ Technology fingerprinting
☐ JavaScript file analysis for secrets
☐ Directory bruteforcing on each live subdomain
☐ API endpoint discovery
☐ Wayback Machine historical analysis
☐ Parameter discovery
☐ Screenshots of all live subdomains (for reference)
☐ Document everything in organized notes
```

---

## 🏠 Practice This Scenario At Home

| Real-World Step                | Practice Equivalent                                           |
| ------------------------------ | ------------------------------------------------------------- |
| Subdomain enumeration          | Run against `tesla.com`, `yahoo.com` on crt.sh (passive only) |
| Live host probing              | Probe your lab hosts with httpx / curl                        |
| JS file analysis               | Examine DVWA/Juice Shop JavaScript files                      |
| Directory bruteforcing         | Gobuster on Juice Shop (`http://localhost:3000`)              |
| API discovery                  | Explore Juice Shop API at `/api` and `/rest`                  |
| GraphQL recon                  | Deploy a vulnerable GraphQL app (e.g., DVGA)                  |
| Full bug bounty recon workflow | Treat Metasploitable + Juice Shop as a single "company"       |

### Recommended Practice Targets

1. **OWASP Juice Shop** — Modern SPA with hidden API endpoints
2. **DVWA** — Classic PHP app for directory/file discovery
3. **bWAPP** — Has many hidden pages to discover
4. **Damn Vulnerable GraphQL Application** — `docker run -p 5013:5013 dolevf/dvga`

---

> **Key Takeaway**: Bug bounty recon is about being MORE thorough than other hunters. Everyone scans the main domain — winners find forgotten subdomains, hidden APIs, and leaked secrets in JavaScript files. Automate your recon pipeline so you can focus on testing.
