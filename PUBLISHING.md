# Publishing Checklist

Target GitHub organization: `dreaifeHebi`.

Recommended repositories:

- `mall-admin-template`
- `mall-user-template`
- `mall-backend-template`
- `mall-template-docs`

## Do Not Publish

Remove or keep ignored before creating public repos:

- `.env`, `.env.local`, `.env.development`, `.env.production`, `.env.*.local`
- real Cloudflare tunnel configs such as `cloudflared-config.yml`
- Cloudflare credential JSON files and local credential directories
- generated backend jars: `*.jar`, especially `mall-admin.jar` and `mall-portal.jar`
- build outputs: `dist/`, `target/`, `.m2/`, `node_modules/`
- local agent/runtime folders: `.codex/`, `.agents/`, `.playwright-cli/`
- machine-specific paths, LAN IPs, private hostnames, or private domains

Keep only safe examples:

- `.env.example`
- `cloudflared-config.example.yml`
- Dockerfiles, Nginx config, Compose examples, README files

## Preflight Scan

Run these from each repository before the first public commit:

```bash
find . -name '.env' -o -name '.env.local' -o -name '*.jar' -o -name '*.log' -o -name '.DS_Store'
```

```bash
rg -n "dreaife|192\\.168|100\\.92|/home/dreaife|credentials-file|cloudflared-config\\.yml|BEGIN .*KEY|SECRET|TOKEN|PASSWORD" .
```

The second command will also find placeholder env names. Review each hit and
keep only placeholder/example values.

## Repository Notes

`mall-admin-template` and `mall-user-template` should include their root
`Dockerfile`, `deployment/nginx.conf`, `.env.example`, source, package files,
and README. Do not include `dist/` or local `.env*` files.

`mall-backend-template` should include source, Maven files, PostgreSQL init SQL,
`deployment/Dockerfile.admin`, `deployment/Dockerfile.portal`,
`deployment/docker-compose.yml`, `deployment/.env.example`, and docs. Do not
include generated jars or Maven `target/` directories.

`mall-template-docs` should include public docs and
`deploy/local-docker/`. The base Compose file should stay Cloudflare-free;
Cloudflare Tunnel belongs in the optional Compose override. Keep real `.env`,
tunnel configs, and credentials ignored.
