# Fail2Ban for WordPress

Fail2Ban filter + jail that blocks brute-force attacks against `wp-login.php` and `xmlrpc.php`. Works with both Nginx and Apache access logs. Much lighter than a WordPress security plugin — the ban happens at the firewall level, before PHP even runs.

## Files

| File | Install to | What it does |
|---|---|---|
| `filters/wordpress-login.conf` | `/etc/fail2ban/filter.d/wordpress-login.conf` | Regex that matches brute-force login attempts |
| `jails/wordpress-login.local` | `/etc/fail2ban/jail.d/wordpress-login.local` | Jail: 5 failed attempts in 10 min → 1-hour ban |

## Install

```bash
sudo cp filters/wordpress-login.conf /etc/fail2ban/filter.d/wordpress-login.conf
sudo cp jails/wordpress-login.local  /etc/fail2ban/jail.d/wordpress-login.local
sudo systemctl restart fail2ban
```

Verify the filter matches your log format **before** relying on it:

```bash
sudo fail2ban-regex /var/log/nginx/access.log /etc/fail2ban/filter.d/wordpress-login.conf
```

Check the jail is working and see who's banned:

```bash
sudo fail2ban-client status wordpress-login
sudo fail2ban-client unban --all          # if you lock yourself out
```

## Tuning

- `maxretry` / `findtime`: raise `maxretry` to 10 if your users mistype passwords often.
- `bantime`: `1h` is a good default; repeat offenders can be escalated with `bantime.increment = true` in `jail.local`.
- The filter intentionally ignores GETs to `/wp-admin/` (normal browsing) and only matches login endpoints.

Pairs well with: rate limiting at the Nginx level (`nginx-snippets` repo) and disabling XML-RPC entirely if you don't use the mobile app (see `wordpress-snippets`).

MIT licensed.
