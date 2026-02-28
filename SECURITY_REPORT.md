# Security Assessment Report — Rosavo PR Website
**Date:** February 28, 2026  
**Files Reviewed:** Ros_2.html, contacts.html, case_studies.html, detailed_mar.html, Team.html  
**Assessment Type:** Static Frontend Security Review

---

## Summary

This is a static HTML/CSS/JS site with no backend, which significantly limits the attack surface. There are no SQL injection or server-side vulnerabilities. However, several important client-side and configuration issues were found and corrected.

---

## Findings & Fixes

### 🔴 HIGH — Missing Security Headers (All 5 files)

**Issue:** None of the HTML pages included any security-related HTTP meta tags. Without these, browsers apply no restrictions on resource loading, framing, or MIME-type sniffing, making the site vulnerable to:
- **Clickjacking**: The site could be embedded in a malicious `<iframe>` to trick users.
- **MIME Sniffing**: Browsers might misinterpret file types, enabling script injection.
- **Data leakage**: Referrer headers would leak the full URL to third-party resources.

**Fix Applied:** Added to every page:
```html
<meta http-equiv="Content-Security-Policy" content="default-src 'self' [approved domains]; ...">
<meta http-equiv="X-Content-Type-Options" content="nosniff">
<meta http-equiv="X-Frame-Options" content="DENY">
<meta http-equiv="Referrer-Policy" content="strict-origin-when-cross-origin">
<meta http-equiv="Permissions-Policy" content="geolocation=(), camera=(), microphone=()">
```

The CSP explicitly whitelists only the domains actually used (Tailwind CDN, Google Fonts, Google image CDN). Any injected external resource would be blocked.

---

### 🔴 HIGH — Incorrect Phone Number in Clickable Link (contacts.html)

**Issue:** The "Call Us" button had `href="tel:+15550000000"` (a US placeholder/test number) but displayed the real Kenyan number `+254 740 100 238`. A user tapping the button would dial the wrong number.

**Fix Applied:** Changed to `href="tel:+254740100238"` to match the displayed number.

---

### 🟠 MEDIUM — Malformed HTML: `<a>` Wrapping `<button>` with Swapped Closing Tags (Ros_2.html)

**Issue:** The "Learn about our culture" element had its closing tags in the wrong order:
```html
<a href="Team.html"> <button ...> ... </a>  ← closes <a> first
</button>  ← then closes <button>
```
This is invalid HTML. Browsers attempt to auto-correct it, but the behaviour is inconsistent across browsers and can cause the link to break or the button to be unclickable in some environments.

**Fix Applied:** Corrected to proper nesting: `<a href="Team.html"><button>...</button></a>`

---

### 🟠 MEDIUM — Duplicate Stylesheet Requests (contacts.html, Team.html)

**Issue:** Both files loaded the Material Symbols font stylesheet **twice** in the `<head>`. This wastes a network request, can cause rendering jank, and is a sign of unreviewed code that could introduce other duplicates.

**Fix Applied:** Removed the duplicate `<link>` tag from both files.

---

### 🟡 LOW — WhatsApp Button with Dead `href="#"` (contacts.html)

**Issue:** The WhatsApp "Chat" quick-action card linked to `href="#"` (no destination). Users expecting to open WhatsApp would be confused, and a security-aware user might suspect phishing.

**Fix Applied:** Updated to `href="https://wa.me/254740100238"` with `rel="noopener noreferrer" target="_blank"` so it opens the correct WhatsApp chat safely.

---

### 🟡 LOW — Social Media Links Missing `rel="noopener noreferrer"` (contacts.html, Team.html)

**Issue:** External links (social icons, share buttons) used `href="#"` with no `rel` attribute. When real URLs are added, without `rel="noopener noreferrer"`, the opened tab could access the opener page via `window.opener`, a known tabnapping attack vector.

**Fix Applied:** Added `rel="noopener noreferrer"` and descriptive `aria-label` attributes to all social/share links.

---

### 🟡 LOW — "Join Our Team" Button with No Action (Team.html)

**Issue:** The "Join Our Team" button was a plain `<button>` with no `href`, `onclick`, or `form` action. Clicking it does nothing, which is confusing UX and could mislead users into thinking they've submitted an application.

**Fix Applied:** Converted to an `<a>` styled as a button pointing to `contacts.html`.

---

### 🟡 LOW — Invalid Tailwind Class `w-25` (detailed_mar.html)

**Issue:** Service card images used `class="rounded-lg w-25 h-auto"`. `w-25` is not a valid Tailwind CSS class (Tailwind uses a specific spacing scale: w-24, w-28, etc.). The image would render at 0 width or browser default, depending on the Tailwind build.

**Fix Applied:** Changed to `w-full h-auto` so images fill their container correctly.

---

### 🟡 LOW — "Contact" Nav Button Not Linked (case_studies.html)

**Issue:** The Contact item in the bottom navigation bar was a `<button>` element with no `href` or event handler — clicking it did nothing.

**Fix Applied:** Converted to an `<a href="contacts.html">` for consistent navigation.

---

### ℹ️ INFO — Dead Links to `project-detail.html` (case_studies.html)

**Issue:** All three case study cards and the featured project link to `project-detail.html`, which does not exist in the uploaded file set. These are broken links.

**Action:** Added `aria-label="Case study detail (coming soon)"` as a marker. You should either create the page or remove the links until it's ready.

---

### ℹ️ INFO — Hidden Bottom Navigation (detailed_mar.html)

**Issue:** The bottom `<nav>` element has `class="hidden"`, making it invisible. Navigation relies solely on the header back button. This is not a security issue but is a UX gap — users cannot navigate forward from this page without going back.

**Recommendation:** Remove the `hidden` class or add explicit forward navigation links.

---

### ℹ️ INFO — Tailwind CDN in Production

**Issue:** All pages use `<script src="https://cdn.tailwindcss.com">`. This is a development/prototyping tool — it applies all Tailwind styles at runtime via JavaScript, which is significantly slower and downloads all ~3MB of Tailwind CSS.

**Recommendation:** Before launching, switch to a proper Tailwind CLI/PostCSS build that generates a small, purged CSS file. This also removes the dependency on an external CDN script, improving both performance and CSP control.

---

## Corrected Files

All 5 HTML files have been corrected and are provided alongside this report:
- `Ros_2.html` ✅
- `contacts.html` ✅
- `case_studies.html` ✅
- `detailed_mar.html` ✅
- `Team.html` ✅
