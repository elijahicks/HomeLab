# Media Server: Jellyfin

Jellyfin was selected over Plex specifically for its fully open-source
model — no third-party account requirement and no data leaving the
local network for core functionality.

## Deployment

```bash
mkdir -p ~/jellyfin/config ~/jellyfin/cache ~/movies

docker run -d --name jellyfin \
  -p 8096:8096 \
  -v ~/jellyfin/config:/config \
  -v ~/jellyfin/cache:/cache \
  -v ~/movies:/media \
  --restart unless-stopped \
  jellyfin/jellyfin
```

## Library configuration

Configured a Movies library pointed at `/media` (the in-container path
mapped to the host's `~/movies` bind mount). Note for anyone following
this pattern: Jellyfin's folder picker browses the container's own
filesystem, not the host's, so host folder names won't appear — only
the mapped mount points will.

## Extending to TV content

TV libraries require a specific on-disk layout for episode detection:

```
~/tvshows/<Show Name>/Season 01/<episode file>
```

Adding a new bind mount to an already-running container requires
recreating it (configuration and library metadata persist independently
in `~/jellyfin/config`, so nothing is lost):

```bash
docker stop jellyfin && docker rm jellyfin

docker run -d --name jellyfin \
  -p 8096:8096 \
  -v ~/jellyfin/config:/config \
  -v ~/jellyfin/cache:/cache \
  -v ~/movies:/media \
  -v ~/tvshows:/tvshows \
  --restart unless-stopped \
  jellyfin/jellyfin
```

## Scaling storage

The initial deployment uses only the host's internal drive. The
documented growth path is either (a) mounting additional storage and
adding it as a second folder within the same Jellyfin library, or (b)
relocating the media directory to larger external storage and replacing
the original path with a symlink, keeping every existing container
configuration and bind mount unchanged.

## Content policy

This project only stores media the operator has legal rights to store
locally — ripped personal media, personally produced video, and public
domain works. It is not configured or intended as a piracy tool, and no
BitTorrent or indexer integration is part of this stack.
