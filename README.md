# greetd-config

Standalone greetd login manager configuration with regreet (GTK4 greeter), multi-distro support (Fedora/Arch), configurable authentication, and optional game mode.

## Quick Start

```bash
# Clone to /opt
sudo git clone https://github.com/MasonRhodesDev/greetd-config.git /opt/greetd-config

# First install (interactive)
sudo /opt/greetd-config/install.sh

# Update after changing options
sudo /opt/greetd-config/install.sh --update
```

## How It Works

The installer symlinks static config files from this repo into `/etc/greetd/`, so edits to files in `/opt/greetd-config/` take effect immediately without re-running the installer.

**Symlinked (edit in repo, changes are instant):**
- `config/mode-desktop.toml` → `/etc/greetd/mode-desktop.toml`
- `config/regreet.toml` → `/etc/greetd/regreet.toml`
- `config/regreet.css` → `/etc/greetd/regreet.css`
- `config/regreet-launcher.sh` → `/etc/greetd/regreet-launcher.sh`
- `config/environments` → `/etc/greetd/environments`
- `assets/wallpaper.png` → `/etc/greetd/bg.png`
- `scripts/*` → `/etc/greetd/scripts/*`

**Generated at install time (re-run `--update` to refresh):**
- `hypr.conf` — rendered from `hypr.conf.tmpl` (work monitor conditional on hostname)
- `mode-game.toml` — interpolates auto-login username
- `/etc/pam.d/greetd` — assembled from `config/pam-snippets/` based on selected features

End to end:

```mermaid
flowchart TD
    subgraph install ["Install time — /opt/greetd-config/install.sh"]
        repo["/opt/greetd-config"]
        repo -->|"symlink: mode-desktop.toml, regreet.toml, regreet.css,<br/>regreet-launcher.sh, environments, scripts/*,<br/>wallpaper as /etc/greetd/bg.png"| etcgreetd["/etc/greetd/"]
        repo -->|"render config/hypr.conf.tmpl<br/>(hostname-conditional monitor block)"| hyprconf["/etc/greetd/hypr.conf"]
        repo -->|"interpolate autologin user"| modegame["/etc/greetd/mode-game.toml"]
        snippets["config/pam-snippets/<br/>auth-unix / fingerprint / google-authenticator / u2f / jumpcloud<br/>+ arch/ and fedora/ overrides"] -->|"assemble per selected features<br/>(authselect features on Fedora)"| pamfile["/etc/pam.d/greetd"]
    end

    subgraph runtime ["Runtime"]
        greetd["greetd"] -->|"/etc/greetd/config.toml points at<br/>mode-desktop.toml or mode-game.toml"| hyprland["Hyprland greeter compositor<br/>(start-hyprland -- --config /etc/greetd/hypr.conf)"]
        hyprland -->|exec-once| regreet["regreet (GTK4)"]
        regreet -->|"credentials via greetd IPC"| pam["PAM stack /etc/pam.d/greetd"]
        pam -->|"auth OK"| session["user session"]
        gamemode["systemd/game-mode.service<br/>(gamepad detection)"] -.->|"swap config.toml to mode-game.toml,<br/>restart greetd"| greetd
        greetd -.->|"mode-game.toml: autologin,<br/>gamescope + Steam"| gamesession["game session<br/>(scripts/game-mode-wrapper.sh optional)"]
    end

    etcgreetd --> greetd
    hyprconf --> hyprland
    pamfile --> pam
    modegame --> greetd
```

## Features

- **Greeter**: regreet (GTK4) running under Hyprland compositor
- **Authentication**: Standard UNIX, JumpCloud, fingerprint, U2F, Google Authenticator 2FA
- **Desktop integration**: GNOME Keyring, KDE Wallet auto-unlock
- **Game mode**: Optional gamepad-activated auto-login to Steam/gamescope
- **Multi-distro**: Fedora and Arch Linux

## Repo Structure

```
greetd-config/
├── install.sh                    # Interactive installer
├── config/
│   ├── mode-desktop.toml         # Desktop mode greetd config
│   ├── mode-game.toml            # Game mode template (user interpolated)
│   ├── hypr.conf.tmpl            # Hyprland config template
│   ├── regreet.toml              # ReGreet greeter config
│   ├── regreet.css               # ReGreet styling
│   ├── regreet-launcher.sh       # Launcher with focus-follow
│   ├── focus-listener.sh         # Monitor focus listener
│   ├── environments              # Wayland env vars
│   └── pam-snippets/             # Modular PAM config snippets
├── scripts/
│   ├── sync-keyring-password     # Keyring password sync
│   ├── game-mode-wrapper.sh      # Alternative Steam launcher
│   ├── setup-fingerprint         # Fingerprint enrollment wizard
│   ├── setup-u2f                 # U2F token registration
│   └── setup-google-auth         # Google Authenticator setup
├── systemd/
│   └── game-mode.service         # Gamepad detection service
├── themes/
│   └── regreet.css               # Legacy theme CSS
└── assets/
    └── wallpaper.png             # Login background
```

## License

Freely redistributable for personal and commercial use.
