# Async Research: Fix external WhatsApp links missing `noopener`

## Project summary
EONET landing is a single-page Astro marketing site in Indonesian for promoting internet packages and routing visitors to WhatsApp consultation. It uses Astro components with scoped CSS, a shared global stylesheet, and the `@astrojs/cloudflare` adapter for Cloudflare deployment.

## Relevant files examined and why
- `README.md` — Documents the Astro project structure and primary commands (`npm run dev`, `npm run build`, preview, Wrangler dry-run). Confirms build output is the main validation path.
- `CLAUDE.md` — Provides project-specific architecture and conventions: Astro single-page site, section components, global CSS design system, no client JS, no configured test suite, and `npm run build` as primary validation. It also noted WhatsApp CTAs used `target="_blank"` and `rel="noreferrer"` before this fix.
- `package.json` — Shows ESM package configuration and scripts. There are no lint/test scripts; only Astro dev/build/preview commands are configured.
- `package-lock.json` — Present as npm lockfile, indicating npm is the package manager. No dependency changes are needed for this task.
- `astro.config.mjs` — Minimal Astro config using `defineConfig` with the Cloudflare adapter. No security/link handling abstractions are configured here.
- `tsconfig.json` — Extends `astro/tsconfigs/strict`; relevant to build validation but no type changes are expected.
- `wrangler.jsonc` — Cloudflare deployment metadata. Unrelated to link attributes but relevant to deployment validation documented in README.
- `src/pages/index.astro` — Main route composition. Imports `Layout`, `Header`, `Hero`, `Packages`, `Features`, and `Footer`; confirms affected components are rendered on the home page.
- `src/layouts/Layout.astro` — HTML shell with metadata, favicon, global CSS import, and fixed background. No external links here.
- `src/styles/global.css` — Shared design tokens, utilities, buttons, containers, section styling, responsive rules, and reduced-motion handling. No link-security logic; link styles are CSS-only.
- `src/components/Hero.astro` — Contains the hero CTA WhatsApp link with `target="_blank"`; updated to `rel="noopener noreferrer"`.
- `src/components/Packages.astro` — Defines package data and renders package-card WhatsApp consultation links dynamically; updated rendered external links to `rel="noopener noreferrer"`.
- `src/components/Footer.astro` — Contains footer/contact WhatsApp CTA with `target="_blank"`; updated to `rel="noopener noreferrer"`.
- `src/components/Header.astro` — Contains only same-page anchor navigation and no external `_blank` links.
- `src/components/Features.astro` — Informational section with no external `_blank` links.
- `.vscode/extensions.json`, `.vscode/launch.json`, `.gitignore`, and `public/` assets — Reviewed as part of repository mapping. They do not affect this link-security fix.

## Patterns and conventions identified
- Source is organized in standard Astro form: `src/pages/` for routes, `src/layouts/` for page shell, `src/components/` for landing-page sections, `src/styles/global.css` for shared tokens/utilities, and `public/` for static root assets.
- `src/pages/index.astro` is composition-only and imports components by relative paths using double quotes.
- Components are `.astro` files with frontmatter when needed, static HTML markup, and colocated scoped `<style>` blocks. There is no client-side JavaScript in the current app.
- Content/copy is Indonesian and branded around EONET Connection.
- Styling relies on shared CSS variables (`--space-*`, `--text-*`, `--accent-*`, etc.) and utility classes (`container`, `btn`, `surface-panel`, section classes).
- External WhatsApp links use the same phone number/base URL pattern (`https://wa.me/6289624424649?...`), open in a new tab via `target="_blank"`, and now set `rel="noopener noreferrer"`.
- There is no shared Link/CTA component or helper for external links; the `rel` attribute is written inline in each component.
- Repository-wide grep for `target="_blank"` found exactly the three source locations specified by the task (`Hero.astro`, `Packages.astro`, `Footer.astro`). Each has explicit `noopener`.
- Testing/validation: no dedicated tests or linter are configured. README/CLAUDE recommend `npm run build` as the primary validation command. Wrangler dry-run is optional after a successful build.

## Implementation strategy
1. Modify only the three affected source components:
   - `src/components/Hero.astro`
   - `src/components/Packages.astro`
   - `src/components/Footer.astro`
2. Replace each inline `rel="noreferrer"` on external WhatsApp links that also have `target="_blank"` with `rel="noopener noreferrer"`.
3. Preserve all existing URLs, copy, classes, formatting, and link behavior. This is an attribute-only security hardening change.
4. Re-run grep for `target="_blank"` / `rel="noopener noreferrer"` to confirm source `_blank` links include explicit `noopener`.
5. Run `npm run build` for validation when dependencies are installed.

## Validation notes
- `git grep 'target="_blank"'` shows the three application WhatsApp links in `Hero.astro`, `Packages.astro`, and `Footer.astro`.
- `git grep 'rel="noopener noreferrer"'` shows all three application WhatsApp links updated.
- `npm run build` could not complete in the current environment because the Astro binary is not installed: `sh: 1: astro: not found`.

## Potential risks or considerations
- The change is low-risk because `noopener` only prevents the opened third-party WhatsApp page from accessing `window.opener`; it should not alter visible UI or navigation behavior.
- Keeping `noreferrer` preserves existing referrer-suppression behavior. The final token order is `noopener noreferrer`.
- Because there is no shared link abstraction, future external links could reintroduce the issue if authors copy an old pattern. A future improvement could be a shared external CTA component or documented convention, but that is outside this focused fix.
