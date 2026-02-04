# Deployment Verification Checklist

Use this checklist before and after each deployment to prevent blank-screen regressions.

## Pre-Deploy Checks

### 1. Clean Build Validation
- [ ] Delete existing `dist/` folder: `rm -rf frontend/dist`
- [ ] Run clean build: `cd frontend && npm run build`
- [ ] Verify build completes without errors or warnings
- [ ] **CRITICAL:** Confirm `frontend/dist/index.html` exists at the root of `dist/`
- [ ] **CRITICAL:** Confirm `frontend/dist/assets/` folder exists with hashed JS/CSS files
- [ ] Open `dist/index.html` in a text editor and verify:
  - Script tags reference `./assets/[name]-[hash].js` (NOT `/src/main.tsx`)
  - Link tags reference `./assets/[name]-[hash].css`
  - No references to source files (`/src/`)

### 2. Publish Directory Configuration
- [ ] **CRITICAL:** Verify hosting/deployment configuration publishes ONLY `frontend/dist/`
- [ ] Confirm `index.html` will be served from the root of the published output (not nested)
- [ ] Verify no source folders (`frontend/src/`, `frontend/public/`, etc.) are deployed
- [ ] Check that `frontend/public/404.html` is copied into `dist/` during build

### 3. Local Production Preview
- [ ] Serve the built `dist/` folder locally: `npx serve frontend/dist`
- [ ] Open in browser (typically http://localhost:3000)
- [ ] **CRITICAL:** Verify homepage renders completely (not blank white screen)
- [ ] Check browser DevTools Console:
  - No uncaught exceptions
  - No "Failed to load module" errors
  - Runtime diagnostics logs appear
- [ ] Check Network tab:
  - All JS bundles load successfully (200 status, NOT 404)
  - All CSS bundles load successfully (200 status, NOT 404)
  - No MIME type errors
  - Assets load from `./assets/` paths

### 4. SPA Fallback Test (Local)
- [ ] While serving `dist/`, navigate directly to a non-root path (e.g., `http://localhost:3000/about`)
- [ ] Verify the app loads (not a 404 or blank page)
- [ ] Check that `404.html` redirects to root and the app handles the route

### 5. Code Quality
- [ ] No `console.error` or uncaught exceptions in Console
- [ ] Error boundary is in place (check `App.tsx` wraps content with `AppErrorBoundary`)
- [ ] Runtime diagnostics are enabled (check `runtimeDiagnostics.ts` logs appear)

## Post-Deploy Checks

### 6. Production Site Verification (Anonymous User)
- [ ] Navigate to production URL (canister or custom domain)
- [ ] **CRITICAL:** Homepage renders (NOT blank white screen)
- [ ] Verify all sections visible: Header, Hero, About, Faculty, Courses, Contact, Footer
- [ ] WhatsApp floating button appears
- [ ] Browser Console shows no uncaught errors
- [ ] **CRITICAL:** Network tab audit:
  - Main JS entry bundle loads (200 status, NOT 404)
  - CSS bundle loads (200 status, NOT 404)
  - All assets load successfully
  - No "Failed to fetch" or CORS errors

### 7. Hard Refresh Test
- [ ] Clear browser cache (Ctrl+Shift+R or Cmd+Shift+R)
- [ ] Reload page
- [ ] Verify homepage still renders correctly (not blank)
- [ ] Check Console and Network tabs again (should be clean)

### 8. Deep Link / SPA Fallback Test (Production)
- [ ] Navigate directly to a non-root path (e.g., `https://yourdomain.com/about`)
- [ ] Verify the app loads (not a 404 or blank page)
- [ ] Check Console for any routing errors
- [ ] Verify `404.html` fallback is working correctly

### 9. Custom Domain Test (if applicable)
- [ ] Navigate to custom domain (e.g., sangharshclasses.in)
- [ ] Verify homepage renders identically to canister URL
- [ ] Check Console and Network tabs (should match canister behavior)
- [ ] Test deep link on custom domain

### 10. Authentication Flow (Logged-In User)
- [ ] Click Login/Connect button
- [ ] Complete Internet Identity authentication
- [ ] Verify app still renders after login (no crash or blank screen)
- [ ] Check Console for any auth-related errors

### 11. Missing Token Test (Logged-In Without Admin Token)
- [ ] Log in without `caffeineAdminToken` in URL hash
- [ ] Verify homepage still renders (public UI not blocked)
- [ ] Check Console: should see "[SafeActor] Skipping access control initialization (no token)"
- [ ] No crash or blank screen

### 12. Mobile Verification
- [ ] Open production site on mobile device (or DevTools mobile emulation)
- [ ] Verify responsive layout works
- [ ] Check Console for mobile-specific errors
- [ ] Test navigation and interactions

### 13. Sitemap & SEO Verification
- [ ] **CRITICAL:** Verify `frontend/dist/sitemap.xml` exists after build
- [ ] **CRITICAL:** Verify `frontend/dist/robots.txt` exists after build
- [ ] Navigate to `https://yourdomain.com/sitemap.xml` in production
- [ ] Verify sitemap returns HTTP 200 (NOT 404 or 3xx redirect)
- [ ] Verify response body is raw XML (starts with `<?xml version="1.0"`)
- [ ] Verify response does NOT contain HTML markers (`<!DOCTYPE html>`, `<html>`)
- [ ] Check HTTP headers using browser DevTools Network tab or `curl -I`:
  - `Content-Type: application/xml` (or `application/xml; charset=utf-8`)
  - NOT `Content-Type: text/html`
- [ ] Navigate to `https://yourdomain.com/robots.txt` in production
- [ ] Verify robots.txt returns HTTP 200 with plain text content
- [ ] Verify robots.txt contains `Sitemap:` directive pointing to sitemap.xml
- [ ] Test in Google Search Console:
  - Submit sitemap URL
  - Verify no "Incorrect HTTP header content-type" errors
  - Confirm sitemap is successfully fetched and parsed

### 14. SEO Title & Meta Description Verification
- [ ] **CRITICAL:** View live homepage HTML source (right-click → View Page Source)
- [ ] Verify `<title>` tag exactly equals: `Sangharsh Classes | Sangharsh Classes Muzaffarpur`
- [ ] Verify NO occurrences of "Premium Coaching Institute" in the HTML source
- [ ] Verify `<meta name="description">` tag exists with non-empty content
- [ ] Verify `<meta name="title">` tag contains: `Sangharsh Classes | Sangharsh Classes Muzaffarpur`
- [ ] Verify `<meta property="og:title">` tag contains: `Sangharsh Classes | Sangharsh Classes Muzaffarpur`
- [ ] **Google Search Console Re-crawl:**
  - Navigate to Google Search Console (https://search.google.com/search-console)
  - Use URL Inspection tool for `https://sangharshclasses.in/`
  - Click "Request Indexing" to trigger immediate re-crawl
  - Note: Verification file already exists at `frontend/public/google53082ab74af04c28.html`
  - Wait 24-48 hours for Google to update search results with new title

## Rollback Criteria

If any of the following occur, **IMMEDIATELY ROLL BACK**:
- Blank white screen on production
- Uncaught exceptions in Console on initial load
- Main JS/CSS assets fail to load (404 errors)
- Homepage sections fail to render
- Hard refresh breaks the site
- Deep links result in 404 or blank pages
- Sitemap.xml returns HTML instead of XML
- Sitemap.xml returns 404 or wrong Content-Type

## Common Issues & Fixes

### Blank White Screen
- **Cause:** Wrong publish directory or `base` config in `vite.config.ts`
- **Fix:** Ensure `base: './'` is at top level of Vite config (NOT inside `build` object)
- **Fix:** Verify publish directory is `frontend/dist` (not `frontend/` or repo root)

### 404 for JS/CSS Assets
- **Cause:** Incorrect asset paths in built `index.html`
- **Fix:** Check `base` config in `vite.config.ts` is at top level
- **Fix:** Verify `dist/index.html` references `./assets/` paths (not absolute `/assets/`)

### Deep Links Show 404
- **Cause:** Missing SPA fallback configuration
- **Fix:** Ensure `frontend/public/404.html` exists and is deployed
- **Fix:** Configure hosting to serve `404.html` for missing routes

### Source Files Deployed Instead of Build
- **Cause:** Wrong publish directory
- **Fix:** Set publish directory to `frontend/dist` (not `frontend/`)
- **Fix:** Verify `dist/index.html` contains hashed asset references (not `/src/main.tsx`)

### Sitemap Returns HTML Instead of XML
- **Cause:** SPA fallback redirecting sitemap.xml to index.html
- **Fix:** Ensure `404.html` excludes `/sitemap.xml` from redirect logic
- **Fix:** Add `_redirects` file to explicitly serve sitemap.xml as static file
- **Fix:** Add `_headers` file to set `Content-Type: application/xml` for sitemap.xml

### Sitemap Shows "Incorrect HTTP header content-type" in Google Search Console
- **Cause:** Server serving sitemap.xml with `text/html` Content-Type
- **Fix:** Add `_headers` file in `frontend/public/` with XML Content-Type directive
- **Fix:** Verify hosting platform supports `_headers` file (Netlify, Cloudflare Pages, etc.)
- **Fix:** If hosting doesn't support `_headers`, configure Content-Type in hosting settings

## Notes
- Keep this checklist updated as new critical features are added
- Document any deployment-specific configuration (environment variables, build flags, etc.)
- If a regression occurs, add a new checklist item to prevent it in the future
- Always test locally with `npx serve dist` before deploying to production
- For Internet Computer deployments, ensure `.ic-assets.json` or equivalent configuration properly serves static files with correct MIME types
