# Challenge 2: Web Time Traveller

> **Difficulty:** 🟢 EASY | **Success Ratio:** 100%
> **Goal:** Find old/archived versions of a website to discover hidden or removed information.

---

## What They Test

Can you use internet archives, caching services, and historical data sources to find information that was previously available on a website but has since been removed or changed?

---

## Key Concepts

Websites change over time — pages get deleted, secrets get removed, and configurations change. But the internet remembers. Multiple services archive web content, and this data can reveal:

- **Removed pages** with sensitive information
- **Old credentials** or API keys in source code
- **Previous site structures** with hidden directories
- **Deprecated functionality** with known vulnerabilities
- **Old version numbers** revealing vulnerable software

---

## Methodology

### Step 1: Wayback Machine (Primary Tool)

The Internet Archive's Wayback Machine is the #1 source for historical web data.

```bash
# Browser - visit directly
https://web.archive.org/web/*/target.com

# View a specific archived date
https://web.archive.org/web/20200101/target.com

# Wayback Machine API
curl "https://web.archive.org/cdx/search/cdx?url=target.com/*&output=text&fl=original&collapse=urlkey"

# CLI Tools
# waybackurls (Go tool)
go install github.com/tomnomnom/waybackurls@latest
waybackurls target.com | sort -u

# waymore (Python - more comprehensive)
python3 waymore.py -i target.com -mode U
```

#### What to Look For in Archives
- Login pages with different authentication
- Admin panels that were later removed
- API documentation pages
- Configuration files
- JavaScript files with embedded secrets
- Pages that now return 403/404 but were once accessible
- Older versions of the application with known vulns

---

### Step 2: Google Cache & Cached Pages

```bash
# Google Cache
# Search: cache:target.com
# Search: cache:target.com/admin

# Google Dorking for cached/indexed content
site:target.com
site:target.com filetype:pdf
site:target.com filetype:xml
site:target.com filetype:conf
site:target.com filetype:log
site:target.com filetype:sql
site:target.com filetype:bak
site:target.com inurl:admin
site:target.com inurl:login
site:target.com inurl:config
site:target.com intitle:"index of"
site:target.com ext:php intitle:phpinfo
```

---

### Step 3: Check for Backup & Historical Files

```bash
# Common backup file extensions
curl -s -o /dev/null -w "%{http_code}" https://target.com/backup.zip
curl -s -o /dev/null -w "%{http_code}" https://target.com/backup.tar.gz
curl -s -o /dev/null -w "%{http_code}" https://target.com/db_backup.sql
curl -s -o /dev/null -w "%{http_code}" https://target.com/site.tar.gz

# Backup versions of specific files
curl https://target.com/index.html.bak
curl https://target.com/index.html.old
curl https://target.com/index.html~
curl https://target.com/index.html.swp
curl https://target.com/index.html.save
curl https://target.com/index.php.bak
curl https://target.com/config.php.bak
curl https://target.com/web.config.old
curl https://target.com/.htaccess.bak

# Version control exposure
curl https://target.com/.git/config
curl https://target.com/.git/HEAD
curl https://target.com/.svn/entries
curl https://target.com/.hg/store/data

# If .git is exposed, dump the whole repository
# Tool: git-dumper
pip install git-dumper
git-dumper https://target.com/.git/ ./dumped_repo

# Then check git history
cd dumped_repo
git log --oneline
git log --all --diff-filter=D -- .   # Find deleted files
git show HEAD~1:config.php            # View old file versions
```

---

### Step 4: Check robots.txt and sitemap.xml

```bash
# Current robots.txt
curl https://target.com/robots.txt

# What robots.txt usually reveals:
# Disallow: /admin/
# Disallow: /backup/
# Disallow: /private/
# Disallow: /api/internal/
# These are paths the site owner doesn't want crawled = worth investigating!

# Archived robots.txt (may show previously hidden paths)
curl "https://web.archive.org/web/2020/https://target.com/robots.txt"

# Sitemap
curl https://target.com/sitemap.xml
curl https://target.com/sitemap_index.xml
curl https://target.com/sitemaps.xml
```

---

### Step 5: Other Internet Archives & Caching Services

```bash
# Archive.today (manual archiving service)
https://archive.ph/target.com

# Common Crawl (huge web archive)
https://index.commoncrawl.org/CC-MAIN-2023-50-index?url=target.com&output=json

# Bing Cache
# Search on Bing and click "Cached" next to results

# Yandex Cache
# Search on Yandex and look for cached version

# CoralCDN (if still active)
http://target.com.nyud.net
```

---

### Step 6: Check HTTP Headers for Versioning

```bash
# Check response headers
curl -I https://target.com

# Look for:
# Last-Modified: tells you when content was last changed
# ETag: can reveal versioning
# X-Powered-By: technology version
# Server: web server version

# Compare archived headers vs current headers
# Changes in headers can reveal infrastructure changes
```

---

## Common Scenarios in the Lab

| Scenario                   | What to Do                                   |
| -------------------------- | -------------------------------------------- |
| Find a removed page        | Use Wayback Machine to view archived version |
| Find old source code       | Check `.git/` exposure or archived JS files  |
| Find removed admin panel   | Check robots.txt + Wayback for old site map  |
| Find old API keys          | View archived JavaScript files               |
| Find deprecated endpoints  | Compare old vs new sitemap.xml               |
| Find previous version info | Check archived HTTP headers                  |

---

## Useful Tools Summary

| Tool            | Purpose                           | Install                                              |
| --------------- | --------------------------------- | ---------------------------------------------------- |
| **waybackurls** | Extract URLs from Wayback Machine | `go install github.com/tomnomnom/waybackurls@latest` |
| **waymore**     | Comprehensive URL extraction      | `pip install waymore`                                |
| **git-dumper**  | Dump exposed .git repositories    | `pip install git-dumper`                             |
| **gobuster**    | Directory brute forcing           | `apt install gobuster`                               |
| **feroxbuster** | Fast directory brute forcing      | Pre-installed on Kali                                |
| **CyberChef**   | Decode any found encoded data     | Browser: gchq.github.io/CyberChef                    |

---

## Tips for the Interview

1. **Start with Wayback Machine** — most time traveller challenges expect this
2. **Check robots.txt first** — it's a quick win that often points to hidden paths
3. **Look at JavaScript files** — secrets often lived in old JS that was later removed
4. **Try common backup extensions** — `.bak`, `.old`, `~` on every important file
5. **If .git is exposed** — you can recover the entire source code history
6. **Check the page source** — comments like `<!-- removed in v2.0 -->` are hints
