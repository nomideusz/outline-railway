# Deploy and Host Outline on Railway

[![Deploy on Railway](https://railway.com/button.svg)](https://railway.com/new/template/outline-production?referralCode=sLjDXb&utm_medium=integration&utm_source=button&utm_campaign=outline-production)

[Outline](https://www.getoutline.com/) is the fast, open-source team wiki and knowledge base: nested documents in collections, real-time collaborative editing, Markdown shortcuts, full-text search, comments, version history, public sharing, templates, an API and imports from Notion, Confluence and Markdown. This template runs the official Outline image with Postgres, Redis and a volume for uploads, and it works the moment the deploy is green: no SSO provider, OAuth app or SMTP server to set up first.

## About Hosting Outline

The stack is three services: Outline, Postgres and Redis.

- **Usable on first boot.** Outline normally refuses to let anyone in until you connect Slack, Google, Microsoft or an OIDC provider, or an SMTP server for magic links. This template uses Outline's built-in first-run setup instead: open the URL, create the workspace and admin account, then add a passkey (Face ID, Touch ID, Windows Hello, a password manager or a security key) and sign in with it from then on. SSO or email, which teammates need for their first sign-in, are optional variables you can fill in later.
- **Upstream's own image, pinned.** Outline runs from the official `outlinewiki/outline` image at a fixed release tag (1.10.1), not `latest`, so a redeploy never surprises you with an untested version.
- **Upgrades that migrate themselves.** Outline runs its database migrations on every boot (guarded by a lock), and Railway's health check on `/_health`, which checks both Postgres and Redis, only switches traffic once they finish. Upgrading is bumping the image tag.
- **Uploads on a volume.** Images and attachments are stored on a persistent volume at `/var/lib/outline/data` and served through Outline itself, so there is no bucket or CORS to configure.
- **Generated secrets.** `SECRET_KEY` and `UTILS_SECRET` are generated as 64-character hex strings at deploy time. Postgres and Redis get their own generated passwords and are reachable only on the private network.

## Common Use Cases

- A team wiki and internal knowledge base that replaces Confluence or Notion
- Engineering docs, runbooks, onboarding guides and meeting notes in one searchable place
- Public documentation or help pages published from the same workspace with share links
- A self-hosted home for documents migrated from Notion, Confluence or Markdown exports

## Dependencies for Outline Hosting

- Postgres 17 (included, private network only)
- Redis 8 (included, private network only)

### Deployment Dependencies

- [Outline self-hosting documentation](https://docs.getoutline.com/s/hosting)
- [Outline on GitHub](https://github.com/outline/outline)
- [Template source on GitHub](https://github.com/nomideusz/outline-railway)

### Implementation Details

**Create your workspace first.** Open the Outline service's Railway domain as soon as the deploy is green. A new server shows a "Create workspace" form: enter a workspace name, your name and your email, and you are signed in as the admin. Until you do, anyone who finds the URL could claim the instance, so do it right away.

**Then add a passkey.** Go to Settings → Passkeys (under Account) and add one. Your browser session lasts three months; after that, or on a new device, the passkey is how you sign back in. Without a passkey, SSO or SMTP, a signed-out admin has no way back in.

**Inviting teammates needs SSO or email.** Outline has no passwords, and a new member's first sign-in comes from SSO or an emailed link, so out of the box the instance is ready for its admin, and teammates can join once you fill in the SSO or `SMTP_*` variables below. After their first sign-in they can add passkeys too.

**Single sign-on (optional).** Fill in `OIDC_CLIENT_ID`, `OIDC_CLIENT_SECRET` and `OIDC_ISSUER_URL` for any OpenID Connect provider (Authentik, Keycloak, Authelia, Zitadel, Okta, Auth0...); endpoints are discovered automatically. Or fill in `GOOGLE_CLIENT_ID` and `GOOGLE_CLIENT_SECRET` for Google Workspace. Redirect URIs are `https://<your-domain>/auth/oidc.callback` and `https://<your-domain>/auth/google.callback`. Accounts are matched by email, so the admin you created keeps working. Slack, Microsoft Entra, Discord and GitHub sign-in are also supported; see Outline's hosting docs for their variables.

**Email (optional).** Fill in the `SMTP_*` variables to turn on email sign-in links, invitations and notifications. This is also the recovery path if everyone loses their passkeys. Railway only allows outbound SMTP on the Pro plan; on other plans use SSO instead.

**Custom domain.** After adding one, set `URL` to `https://your-domain`. Passkeys are tied to the domain they were created on, so add a new passkey after switching domains (sign in once with the old domain's session, SSO or email).

**Back up `SECRET_KEY` and the volume.** `SECRET_KEY` signs sessions and encrypts stored integration tokens; changing it logs everyone out. The volume holds every uploaded file; documents themselves live in Postgres.

**Resources.** The stack runs one Outline process (`WEB_CONCURRENCY=1`) that serves the web app, real-time collaboration, websockets and background jobs. Outline otherwise starts one process per CPU, which would multiply its memory use on Railway. On Railway, Outline idled at about 420 MB of RAM and the whole stack (with Postgres and Redis) at about 525 MB.

## Why Deploy Outline on Railway?

Railway is a singular platform to deploy your infrastructure stack. Railway will host your infrastructure so you don't have to deal with configuration, while allowing you to vertically and horizontally scale it.

By deploying Outline on Railway, you are one step closer to supporting a complete full-stack application with minimal burden. Host your servers, databases, AI agents, and more on Railway.
