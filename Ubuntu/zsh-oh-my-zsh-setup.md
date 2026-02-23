# Zsh + Oh My Zsh Installation Guide (Server-Safe)

This guide walks you through **installing zsh**, **Oh My Zsh**, **useful autocomplete plugins**, and **making zsh the default shell** on a Linux server (e.g. Ubuntu/Debian). It also includes a **safe fallback configuration** to ensure SSH sessions always land in zsh.

The steps below are clean, minimal, and production-safe.

---

## 1. Install required packages

```bash
sudo apt update
sudo apt install -y zsh git curl
```

---

## 2. Make zsh the default login shell

```bash
sudo chsh -s /usr/bin/zsh <YOUR_USERNAME>
```

Replace `<YOUR_USERNAME>` with your actual username (e.g. `jlanghein`).

> Note: This affects **new login sessions** only.

---

## 3. Start zsh for the first time

Either log out and log back in via SSH, or run:

```bash
exec zsh
```

When prompted by the *zsh new user configuration*, choose:

```text
0  (create empty .zshrc and exit)
```

This keeps the setup clean for Oh My Zsh.

---

## 4. Install Oh My Zsh

```bash
sh -c "$(curl -fsSL https://raw.githubusercontent.com/ohmyzsh/ohmyzsh/master/tools/install.sh)"
```

When asked whether to overwrite `.zshrc`, choose **Yes**.

---

## 5. Install autocomplete & highlighting plugins

### zsh-autosuggestions

```bash
git clone https://github.com/zsh-users/zsh-autosuggestions \
  ${ZSH_CUSTOM:-~/.oh-my-zsh/custom}/plugins/zsh-autosuggestions
```

### zsh-syntax-highlighting

```bash
git clone https://github.com/zsh-users/zsh-syntax-highlighting.git \
  ${ZSH_CUSTOM:-~/.oh-my-zsh/custom}/plugins/zsh-syntax-highlighting
```

---

## 6. Enable plugins and set theme

Edit your zsh configuration:

```bash
nano ~/.zshrc
```

### Enable plugins

Replace the plugins line with:

```bash
plugins=(git zsh-autosuggestions zsh-syntax-highlighting)
```

> `zsh-syntax-highlighting` must be **last**.

### Set a server-safe theme

Change:

```bash
ZSH_THEME="robbyrussell"
```

To one of the following (no special fonts required):

```bash
ZSH_THEME="af-magic"
```

or

```bash
ZSH_THEME="jreese"
```

### Optional: subtle autosuggestion color

Add near the bottom:

```bash
ZSH_AUTOSUGGEST_HIGHLIGHT_STYLE='fg=8'
```

Reload:

```bash
source ~/.zshrc
```

---

## 7. Ensure SSH sessions always switch to zsh

Some environments start `bash` even when the login shell is set to zsh. To fix this safely, add the following to the **BOTTOM** of `~/.bashrc`:

```bash
# Automatically switch to zsh for interactive SSH sessions
case $- in
  *i*)
    if [ -n "$SSH_CONNECTION" ] && command -v zsh >/dev/null 2>&1; then
      exec zsh
    fi
    ;;
esac
```

This:
- only affects **interactive shells**
- only triggers over **SSH**
- does not affect scripts, cron jobs, or scp/rsync

---

## 8. Final result

You now have:

- zsh as your default shell
- Oh My Zsh managing configuration
- inline command autosuggestions
- syntax highlighting
- a clean, production-safe prompt
- reliable behavior over SSH

---

## Notes

- Do **not** apply the `.bashrc` switch for `root` unless explicitly desired
- Avoid heavy themes or fonts on production servers
- This setup is compatible with tmux, sudo, and non-interactive shells

---

*Last updated: February 2026*
