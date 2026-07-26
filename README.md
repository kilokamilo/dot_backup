# dot_backup

Backs up top-level home directory dotfiles (`.config`, `.local`, shell rc files, etc.) into a timestamped tar archive on the Desktop.

Heavy or disposable trees (caches, browser data, Trash, and similar) are excluded at create time.

The purpose of this tool is to make a backup of just the user configuration files. 

## Requirements

- POSIX `sh`
- GNU `find`, `sed` (`-z`), `tar` (`--null`, `--exclude-from`), `du`, `mktemp`

## Usage

```sh
./dot_backup
```

No arguments. On success, prints the exclude list and the archive path/size.

### Output

| Item | Value |
|------|--------|
| Directory | `$XDG_DESKTOP_DIR` if set, else `$HOME/Desktop` |
| Filename | `user_${USER}_dotfiles_YYYYMMDDTHHMMSS.tar` |

Example: `user_kam_dotfiles_20260725T183045.tar`

### What is included

Everything matching `.*` directly under `$HOME` (`find -maxdepth 1`), as recursive tree contents when the entry is a directory.

### What is currently excluded

Paths are built from `$HOME` (stored in the archive without a leading `/`). Current list:

- `.cache`
- `.var/app`
- `.mozilla`
- `.local/share/Trash`
- `.local/share/akonadi`
- `.local/share/okular/docdata`
- `.config/google-chrome`
- `.local/share/klipper/data`
- `.local/share/waydroid`

Edit the heredoc in `dot_backup` to add or remove excludes. Do not put a trailing `/` on exclude paths (GNU tar matching is picky).

## Behavior notes

- Exclusions are applied while writing the archive (not packed then deleted).
- Archive members look like `home/$USER/.config/...` (leading `/` stripped via `tar -C /`).
- Same-second re-runs can overwrite; the timestamp includes time down to the second.
- The archive is uncompressed `.tar` for simple restore and speed. You can
  compress later with your choice of compression tool and settings
- Secrets under `$HOME` (e.g. `.ssh`, `.gnupg`) **are** included unless you exclude them.

## Restore

List contents:

```sh
tar -tf ~/Desktop/user_${USER}_dotfiles_*.tar | less
```

Extract into `/` (paths are rooted at `home/...`):

```sh
tar -C / -xvf ~/Desktop/user_${USER}_dotfiles_YYYYMMDDTHHMMSS.tar
```

Or extract under a staging directory:

```sh
mkdir -p /tmp/dot-restore
tar -C /tmp/dot-restore -xvf ~/Desktop/user_${USER}_dotfiles_YYYYMMDDTHHMMSS.tar
# files land in /tmp/dot-restore/home/$USER/...
```

## Install (optional)

```sh
install -m 755 dot_backup /usr/local/bin/dot_backup
```
