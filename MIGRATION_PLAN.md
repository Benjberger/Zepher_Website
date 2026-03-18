# Zepher Website Migration Plan
## WordPress (GoDaddy) → Static Site (Netlify)

---

## Overview

Migrate from the current WordPress site hosted on GoDaddy to the new static HTML site hosted on Netlify. Email is unaffected (hosted separately). Domain DNS stays on GoDaddy, pointed at Netlify.

---

## Pre-Migration Checklist

- [ ] Verify the current live site's important URLs (use Google Search Console or `site:yourdomain.com` in Google)
- [ ] Take screenshots of the current site for reference
- [ ] Confirm your GoDaddy account login and DNS management access
- [ ] Ensure the Formspree contact form is working (test at the current dev URL)

---

## Step-by-Step Migration

### Phase 1: Prepare the Repository (30 min)

1. **Rename the HTML file** to `index.html` — Netlify (and all web servers) serve `index.html` as the default page
   ```
   zepher-website-final (10).html  →  index.html
   ```

2. **Add a `_redirects` file** for SEO — redirect any old WordPress URLs to the homepage so you don't lose search rankings:
   ```
   # Netlify _redirects file
   /wp-admin       /        301
   /wp-login.php   /        301
   /wp-content/*   /        301
   /feed           /        301
   /feed/          /        301
   /*              /index.html  200
   ```
   The last rule is a fallback — any URL that doesn't match a file serves your site (good for single-page apps).

3. **Add a `robots.txt`** (optional but recommended):
   ```
   User-agent: *
   Allow: /
   Sitemap: https://yourdomain.com/sitemap.xml
   ```

4. **Add a basic `sitemap.xml`** for search engines:
   ```xml
   <?xml version="1.0" encoding="UTF-8"?>
   <urlset xmlns="http://www.sitemaps.org/schemas/sitemap/0.9">
     <url>
       <loc>https://yourdomain.com/</loc>
       <lastmod>2026-03-18</lastmod>
       <priority>1.0</priority>
     </url>
   </urlset>
   ```

5. **Push changes** to the `main` branch on GitHub

### Phase 2: Deploy on Netlify (15 min)

1. **Create a Netlify account** at [netlify.com](https://netlify.com) (free tier is fine)
2. **Connect your GitHub repo** — click "Add new site" → "Import an existing project" → select `Zepher_Website`
3. **Configure build settings:**
   - Build command: *(leave blank — no build step needed)*
   - Publish directory: `/` or `.` (the root of the repo)
4. **Deploy** — Netlify will give you a temporary URL like `https://zepher-xyz.netlify.app`
5. **Test everything** at the Netlify preview URL:
   - [ ] All sections load and scroll correctly
   - [ ] Contact form submits successfully
   - [ ] Mobile responsive layout works
   - [ ] Images and fonts load
   - [ ] No console errors

### Phase 3: Connect Your Domain (15 min)

1. **In Netlify:** Go to Site settings → Domain management → Add custom domain → enter your domain (e.g., `zepher.com`)
2. **In GoDaddy DNS:** Update your DNS records:

   | Type  | Name | Value                        | TTL    |
   |-------|------|------------------------------|--------|
   | A     | @    | `75.2.60.5`                  | 600    |
   | CNAME | www  | `your-site-name.netlify.app` | 600    |

   > Netlify will show you the exact values to use. The A record IP may differ — always use what Netlify provides.

3. **Remove old GoDaddy hosting DNS records** — delete any A records or CNAMEs that point to the old WordPress hosting
4. **Enable HTTPS** in Netlify → Domain settings → HTTPS → Verify DNS and provision certificate (automatic, free via Let's Encrypt)

   > **Important:** Do NOT delete your MX records or any email-related DNS records. Since your email is hosted separately, those records must stay intact.

### Phase 4: Verify & Monitor (1-2 days)

1. **DNS propagation** — changes typically take 15 min to 48 hours. Use [dnschecker.org](https://dnschecker.org) to monitor
2. **Test the live site** once DNS propagates:
   - [ ] `https://yourdomain.com` loads the new site
   - [ ] `https://www.yourdomain.com` loads the new site
   - [ ] HTTPS padlock shows (certificate is valid)
   - [ ] Contact form works on the live domain
   - [ ] Old WordPress URLs redirect to homepage (not 404)
3. **Submit updated sitemap** to Google Search Console
4. **Monitor search rankings** over the following weeks

### Phase 5: Cleanup (after 1-2 weeks of stable operation)

1. **Cancel WordPress hosting** on GoDaddy (keep your domain registration active!)
   - ⚠️ **Do NOT cancel domain registration** — only cancel the hosting/WordPress plan
2. **Keep a backup** of your WordPress site export (just in case) — you can export via WP Admin → Tools → Export before canceling
3. **Update Google Search Console** if your site's URL structure changed

---

## Cost Comparison

| Item | GoDaddy (current) | Netlify (new) |
|------|-------------------|---------------|
| Hosting | ~$10-25/mo | **Free** (free tier) |
| SSL Certificate | Included or extra | **Free** (Let's Encrypt) |
| Domain Registration | ~$15-20/yr | Keep on GoDaddy (~$15-20/yr) |
| Form Handling | WordPress plugins | **Free** (Formspree free tier) |

**Estimated savings: ~$120-300/year** on hosting alone.

---

## Rollback Plan

If something goes wrong:
1. In GoDaddy DNS, point the A record back to GoDaddy's hosting IP (note it down before changing!)
2. The WordPress site will be live again within minutes to hours (DNS propagation)
3. This is why we keep WordPress running for 1-2 weeks after the switch

---

## Key Risks & Mitigations

| Risk | Mitigation |
|------|-----------|
| DNS misconfiguration breaks email | Only change A and www CNAME records; leave MX records untouched |
| SEO ranking drop | `_redirects` file catches old WordPress URLs; submit sitemap to Google |
| Form stops working | Formspree is independent of hosting; test before and after migration |
| Downtime during DNS switch | Set low TTL (600s) before switching; keep WordPress live as fallback |

---

## Timeline

| Phase | Time | When |
|-------|------|------|
| Prepare repo | 30 min | Day 1 |
| Deploy on Netlify | 15 min | Day 1 |
| Connect domain | 15 min | Day 1 |
| Verify & monitor | 1-2 days | Days 1-3 |
| Cancel old hosting | 5 min | Week 2-3 |

**Total hands-on time: ~1 hour**
