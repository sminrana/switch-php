# switch-php: bash script to change PHP version on your Mac

## Put this code in your .zshrc file, then run source ~/.zshrc

```bash
switch-php() {
    local version=$1
    if [[ -z "$version" ]]; then
        echo "Usage: switch-php <version> (e.g., 7.4, 8.1, 8.2)"
        return 1
    fi

    if ! command -v brew &>/dev/null; then
        echo "❌ Homebrew not found. Please install Homebrew first."
        return 1
    fi

    echo "Switching to PHP $version..."

    # Unlink all installed php@* versions
    for v in $(brew list --formula | grep '^php@'); do
        echo "Unlinking $v..."
        brew unlink "$v" &>/dev/null
    done

    # Link the selected version
    if brew list "php@$version" &>/dev/null; then
        brew link --overwrite --force "php@$version"
        echo "✅ Switched to PHP $version"
    else
        echo "❌ PHP version $version not found. Try: brew install php@$version"
        return 1
    fi

    # Update Apache config if files exist
    local httpd_conf="/opt/homebrew/etc/httpd/httpd.conf"
    local backup_conf="/opt/homebrew/etc/httpd/httpd-${version}.conf"
    if [[ -f "$backup_conf" ]]; then
        cp "$backup_conf" "$httpd_conf"
        echo "Apache config updated."
    else
        echo "⚠️  Backup Apache config $backup_conf not found. Skipping config update."
    fi

    # Update PATH in .zshrc if not already present
    local php_bin="/opt/homebrew/opt/php@$version/bin"
    local zshrc="$HOME/.zshrc"
    if ! grep -q "$php_bin" "$zshrc"; then
        echo "" >> "$zshrc"
        echo "export PATH=\"$php_bin:\$PATH\"" >> "$zshrc"
        echo "" >> "$zshrc"
        echo "Added PHP $version bin path to .zshrc"
    fi


    # Restart Apache service if available
    brew services restart apache2
    echo "✅ Apache restarted."
}
```

## Run this on your terminal

```
switch-php 8.1
```

## Tested on macOS Sequoia

## Installation

### 1. Install PHP

```
brew tap shivammathur/php

brew install shivammathur/php/php@7.0
brew install shivammathur/php/php@7.1
brew install shivammathur/php/php@7.2
brew install shivammathur/php/php@7.3
brew install shivammathur/php/php@7.4
brew install shivammathur/php/php@8.0
brew install shivammathur/php/php@8.1
brew install shivammathur/php/php@8.2
brew install shivammathur/php/php@8.3
brew install shivammathur/php/php@8.4
```

### 2. Keep httpd conf file for each version of PHP, like this

![Example of httpd conf files for each PHP version](https://smindev.s3.amazonaws.com/static/Screenshot2025-06-23at11.46.59%E2%80%AFAM.png)

### Example of my httpd-7.2.conf

![Example of httpd conf file for php 7.2](https://smindev.s3.amazonaws.com/static/Screenshot2025-06-23at11.49.53%E2%80%AFAM.png)

**Important: Just make sure you have httpd.conf file for each PHP version which includes correct PHP path.**

Example of my httpd-8.1.conf

```
LoadModule php_module /opt/homebrew/opt/php@8.1/lib/httpd/modules/libphp.so
```
