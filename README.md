# WordPress + Cloudflare Tunnel Stack

This repository provisions WordPress (PHP-FPM), nginx, MySQL, Redis, phpMyAdmin, and helper containers that are ready to sit behind a Cloudflare Zero Trust tunnel.

## Prerequisites

- Docker Compose v2
- A `.env` file providing `DB_ROOT_PASSWORD`, `WP_DB_USER`, `WP_DB_PASSWORD`, and `WP_DB_NAME`
- (Optional) a Cloudflare Tunnel that forwards HTTPS traffic to `http://localhost:7968`

## One-time bootstrap

Use the helper script to create bind mounts, fix permissions, and start the stack:

```bash
./scripts/bootstrap-wordpress.sh
```

The script will:

1. Create the `html`, `db_data`, and `nginx_cache` directories if needed.
2. Run the WordPress container as `root` one time to `chown -R www-data:www-data /var/www/html`, ensuring WordPress can unpack core files into the bind mount.
3. Launch every service defined in `docker-compose.yml`.

After it completes, finish the WordPress installer at `http://localhost:7968/wp-admin/install.php` (or through your Cloudflare hostname).

## Cloudflare Zero Trust / arbitrary hostnames

- `nginx/conf.d/wordpress.conf` now uses `server_name _ ...` so nginx will accept whatever `Host` header Cloudflare Zero Trust injects.
- When creating your tunnel route, point it to `http://localhost:7968` and **do not** enable HTTP host rewrites, since nginx already handles arbitrary hostnames.
- If you want stricter controls later, replace `_` with the explicit domains you intend to serve.

## Re-applying permissions later

If you clone this repo elsewhere or wipe the `html` directory, re-run `./scripts/bootstrap-wordpress.sh` (or just the `docker compose run --rm --entrypoint "" --user root wordpress chown -R www-data:www-data /var/www/html` step) before starting the stack. WordPress requires write access to `/var/www/html` during upgrades and plugin installs.

## Troubleshooting

- **Nginx health check stuck in `starting`:** The health probe now uses `127.0.0.1` explicitly to avoid IPv6/localhost mismatches. If it still fails, run `docker compose logs nginx` and confirm the `wordpress` container is marked healthy.
- **Permission denied writing into `html`:** Ensure the directory on the host is owned by UID/GID 33 (`www-data`). The bootstrap script or `sudo chown -R 33:33 html` fixes it.
- **Cloudflare shows 403/blocked:** Cloudflare Access policies might block unauthenticated users; confirm your policy allows you, and that the tunnel routes to the `7968` port exposed here.
