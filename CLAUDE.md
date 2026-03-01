# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

A single-file PCOS symptom-checker quiz (`pcos-checker.html`) written in Bosnian, designed to be embedded as a Shopify custom page. No build system, no dependencies, no framework — everything ships inline.

## Development

**Preview locally:**
```bash
start pcos-checker.html       # Windows — opens in default browser
```

**Deploy to Shopify:**
1. Shopify Admin → Online Store → Pages → Add Page
2. Click `<>` (HTML editor) in the content area
3. Paste only the `<body>` content (4 `<section>` divs + `<script>` tag)
4. The `<head>` tags (Google Fonts + Phosphor Icons CDN) belong in `theme.liquid`; when pasting the full file as a standalone page, use a blank Liquid template to suppress theme padding

**Git / GitHub:**
- Repo: https://github.com/by-hana/pcos-checker
- Branch: `main`
- Commit with conventional-commit prefixes: `feat:`, `fix:`, `style:`, `refactor:`, `docs:`

**Commit and push after every meaningful change.** This is non-negotiable — no work should exist only on disk. After completing any task (new feature, bug fix, copy change, style tweak), stage the relevant files, write a descriptive commit message, and push to `origin main` before considering the task done.

Commit message format:
```
<type>: <short imperative summary>

<optional body — what changed and why, if not obvious>

Co-Authored-By: Claude Sonnet 4.6 <noreply@anthropic.com>
```

Examples of good commit messages:
- `fix: correct score boundary — score 9 now maps to Srednji not Nizak`
- `style: increase answer button tap target to 48px on mobile`
- `feat: add conditional CTA copy for high-risk result screen`
- `refactor: extract RISK_CONTENT into separate const block`

Never batch unrelated changes into one commit. One logical change = one commit.

## Architecture

Everything lives in `pcos-checker.html` in three sections:

### `<head>` — external resources only
- Google Fonts CDN: **Playfair Display** (headings) + **Inter** (body)
- Phosphor Icons CDN (`@phosphor-icons/web@2.1.1`)

### `<style>` — CSS custom properties + screen system
Brand tokens are defined as CSS variables on `:root`:
```
--lavender: #cabad7  |  --plum: #4f364b  |  --orange: #f15722  |  --bg: #faf8fc
```
Screens are toggled via two classes:
- `.screen` → `opacity:0; pointer-events:none; position:absolute`
- `.screen--active` → `opacity:1; pointer-events:auto; position:relative`

Question card transitions use `@keyframes slideInCard` (forward) and `@keyframes slideInCardBack` (back navigation).

### `<script>` — vanilla JS IIFE, no frameworks
Key objects and their roles:

| Symbol | Role |
|---|---|
| `CONFIG` | Klaviyo keys + Shopify URLs — **replace before going live** |
| `QUESTIONS[]` | 12 question objects `{ text, answers: [{label, value}] }` |
| `RISK_CONTENT` | `{low, mid, high}` → badge, headline, body copy, CTAs |
| `state` | Runtime state: `currentQuestion`, `answers[]`, `score`, `riskLevel`, `email`, `submitting` |

Core functions:
- `activateScreen(name)` — switches the visible screen
- `renderQuestion(idx, direction)` — injects question HTML and wires answer buttons
- `handleAnswerClick()` — stores answer, disables buttons, auto-advances after 400 ms
- `calculateScore()` — sums `state.answers[]`; thresholds: ≤8 low, 9–16 mid, 17–24 high
- `subscribeToKlaviyo(email)` — async POST to Klaviyo client subscriptions API; failure is silent, results always render
- `renderResults()` — injects risk badge, copy, and CTAs from `RISK_CONTENT`

## Klaviyo Integration

```
POST https://a.klaviyo.com/client/subscriptions/?company_id={COMPANY_ID}
revision: 2023-12-15
```
Custom profile properties sent: `pcos_result`, `pcos_score`, `pcos_score_max: 24`, `quiz_completed: true`, `quiz_date` (ISO date).
HTTP 202 = success. The public company ID is safe to expose in browser JS.

**Required Klaviyo setup:**
- Flow triggered by "Joined List" on the PCOS list → immediate email with PDF download link
- Optional conditional split on `pcos_result` for per-risk-level copy

## Brand Palette

| Token | Hex |
|---|---|
| Lavender | `#cabad7` |
| Deep plum | `#4f364b` |
| Accent orange | `#f15722` |
| Page background | `#faf8fc` |
| White | `#ffffff` |

## Before Going Live

Replace these four values in `CONFIG` (~line 294 of `pcos-checker.html`):
```js
KLAVIYO_COMPANY_ID: 'YOUR_PUBLIC_API_KEY'  // 6-char Klaviyo public key
KLAVIYO_LIST_ID:    'YOUR_LIST_ID'          // from Klaviyo list URL
SHOP_URL:           '/collections/all'
BOOKING_URL:        '/pages/booking'
```
