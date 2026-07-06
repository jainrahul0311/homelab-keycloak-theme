# homelab-keycloak-theme

A Keycloak login theme (React + Tailwind v4 + shadcn/ui, via
[Keycloakify](https://keycloakify.dev)) that builds into a `homelab-theme.jar`
and is published as a GitHub Release.

Bootstrapped from the [`@oussemasahbeni/keycloakify-login-shadcn`](https://www.npmjs.com/package/@oussemasahbeni/keycloakify-login-shadcn)
theme package — **not a fork.** The theme's UI source is a pinned npm
dependency pulled in at install time (`keycloakify sync-extensions`), so this
repo stays tiny and there is **no upstream to merge from** — updating is just a
version bump (see [Updating](#updating)).

> **Dark mode is built in** — light / dark / system toggle with a persistent
> preference. Every accent preset and base palette below is defined for both.

---

## 1. Get the JAR

Every push to `main` builds the theme and updates a rolling **`latest`** GitHub
Release. Stable download URL:

```bash
curl -L -o homelab-theme.jar \
  https://github.com/jainrahul0311/homelab-keycloak-theme/releases/latest/download/homelab-theme.jar
```

Or build locally: `npm ci && npm run build-keycloak-theme` → `dist_keycloak/homelab-theme.jar`.

---

## 2. Theming — runtime env vars (no rebuild)

All options are **runtime environment variables** read by the Keycloak server
(`kcContext.properties`). Set them on the Keycloak process and restart to
change the look — the same JAR covers every combination, no rebuild.

| Variable | Default | Allowed values | Description |
| --- | --- | --- | --- |
| `SHADCN_THEME_PRESET` | `neutral` | [accent presets](#accent-presets) | Primary/brand accent color. |
| `SHADCN_THEME_BASE` | `neutral` | [base palettes](#base-palettes) | Neutral surface palette (bg, cards, borders, muted, ring). |
| `SHADCN_THEME_RADIUS` | `default` | `default`, `none`, `small`, `medium`, `large` | Global border radius. |
| `SHADCN_THEME_FONT` | `geist` | [font presets](#font-presets) | Main font family. |
| `SHADCN_THEME_LAYOUT` | `two-column` | `two-column`, `centered-card`, `image-aside` | Outer page layout. |
| `SHADCN_THEME_LOGO_WHITE_URL` | `""` | URL or `%BASE_URL%/file` | Light-mode logo. |
| `SHADCN_THEME_LOGO_DARK_URL` | `""` | URL or `%BASE_URL%/file` | Dark-mode logo. |
| `SHADCN_THEME_SIDE_IMAGE_URL` | `""` | URL or `%BASE_URL%/file` | Side image for `image-aside` layout. |
| `SHADCN_THEME_PLACEHOLDER` | `true` | `true`, `false` | Show/hide input placeholders on fixed auth forms. |

Defaults live in `vite.config.ts` (`environmentVariables`). They are only
fallbacks — the Keycloak server env always wins at runtime.

### Accent presets
`neutral` · `amber` · `blue` · `cyan` · `emerald` · `fuchsia` · `green` · `indigo` · `lime` · `orange` · `pink` · `purple` · `red` · `rose` · `sky` · `teal` · `violet` · `yellow`

### Base palettes
`neutral` · `stone` · `zinc` · `mauve` · `olive` · `mist` · `taupe`

### Font presets
`inter` · `geist` · `manrope` · `figtree` · `source-sans-3` · `ibm-plex-sans` · `lora` · `playfair-display` · `jetbrains-mono`

---

## 3. Install on Keycloak

The theme registers as **`homelab-theme`** — select it under **Admin Console →
Realm settings → Themes → Login theme**.

### Docker / Docker Compose

```yaml
services:
  keycloak:
    image: quay.io/keycloak/keycloak:26.0
    command: ["start"] # or start-dev for local testing
    environment:
      SHADCN_THEME_PRESET: indigo
      SHADCN_THEME_BASE: zinc
      SHADCN_THEME_FONT: inter
      KC_HOSTNAME: auth.example.com
    volumes:
      - ./homelab-theme.jar:/opt/keycloak/providers/homelab-theme.jar:ro
```

```bash
docker compose up -d
# after changing a SHADCN_THEME_* value:
docker compose up -d --force-recreate keycloak
```

### systemd / bare-metal

```bash
sudo cp homelab-theme.jar /opt/keycloak/providers/
sudo systemctl edit keycloak   # add Environment=SHADCN_THEME_PRESET=indigo etc.
sudo -u keycloak /opt/keycloak/bin/kc.sh build
sudo systemctl restart keycloak
```

> Adding/replacing the JAR needs `kc.sh build` (or a fresh container). Changing
> only a `SHADCN_THEME_*` value needs just a restart.

---

## 4. Local development

```bash
npm ci
npm run dev                  # Vite dev server
npm run storybook            # preview every page with live theme controls
npm run build-keycloak-theme # build the JAR -> dist_keycloak/homelab-theme.jar
```

---

## 5. Updating

The theme UI comes from the `@oussemasahbeni/keycloakify-login-shadcn`
dependency (and `keycloakify` / `@keycloakify/login-ui`). To update:

```bash
npm install @oussemasahbeni/keycloakify-login-shadcn@latest
npm run build-keycloak-theme   # postinstall re-syncs the theme source
```

There is no fork to merge — bumping the version is the whole update. To
customize a specific page and version-control it, run
`npx keycloakify own src/login/pages/<page>/Page.tsx` to take ownership of it.

---

## 6. CI / Release

`.github/workflows/release.yml` on every push/PR to `main`:

- **build** — `npm run build-keycloak-theme` → uploads the JAR artifact.
- **security_scan** — Trivy filesystem scan (`CRITICAL`/`HIGH`).
- **release** — on push to `main`: updates the rolling `latest` Release with the JAR.

No deployment secrets — the JAR is distributed via GitHub Releases and pulled
onto the Keycloak host manually.
