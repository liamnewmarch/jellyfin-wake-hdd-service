# Jellyfin wake HDD service

This package installs a systemd timer which polls Jellyfin. When a client connects, it wakes hard disks so they’re ready for playback.

## Install

Download the `.deb` from the [latest release](https://github.com/liamnewmarch/jellyfin-wake-hdd-service/releases) and install it:

```sh
sudo apt install ./jellyfin-wake-hdd_1.0.0_all.deb
```

Or build it yourself (see [Building](#building) below).

## Configure

First, create the API key under Jellyfin’s **Dashboard → API Keys**.

Then edit `/etc/jellyfin-wake-hdd.conf`:

```sh
JELLYFIN_URL="http://localhost:8096"
JELLYFIN_API_KEY="your-api-key-here"
WAKE_PATHS=(
    "/mnt/my-first-hdd/TV Shows"
    "/mnt/my-other-hdd/Movies"
)
```

You can see the full option reference with `man jellyfin-wake-hdd.conf`.

Finally, start the service:

```sh
sudo systemctl start jellyfin-wake-hdd.timer
```

The service is enabled by default, so once configured it will also start automatically on boot.

## Usage

```sh
# Check the timer’s schedule and last/next run
systemctl list-timers jellyfin-wake-hdd.timer

# Follow what each run finds and does
journalctl -u jellyfin-wake-hdd.service -f

# Change how often it runs (default: every 30s) without losing the change on upgrade
sudo systemctl edit jellyfin-wake-hdd.timer
```

## How it works

Each run queries Jellyfin’s `/Sessions` API for the number of connected clients. If at least `MIN_SESSIONS` (default 1) are connected — whether actively playing or just idle on the home screen — it runs `stat` on each path in `WAKE_PATHS`. A `stat` is enough to force the underlying drive (and, on enclosures that wake members together, the whole enclosure) to spin up, without reading meaningful data off disk (and without depending on another system package like `hdparm`).

This deliberately wakes drives as soon as a client *connects*, so the drives have time to finish spinning up while the user is browsing.

## Building

Requires `debhelper` and `devscripts`:

```sh
sudo apt install debhelper devscripts
dpkg-buildpackage -us -uc -b
```

## License

MIT — see [LICENSE](LICENSE).
