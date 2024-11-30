---
title: 'Install Multiple PHP version di MacOS'
date: '2023-08-24 16:10:00'
---

Kali ini kita akan install beberapa PHP version di MacOS

install brew command
```bash
/bin/bash -c "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/HEAD/install.sh)"
```

install PHP

```bash
brew install php@8.0
brew install php@8.1
brew install php@8.2

# set php8.2 as default
brew link --force --overwrite php@8.2

echo 'export PATH="/usr/local/opt/php@8.0/bin:$PATH"' >> /.zshenv
echo 'export PATH="/usr/local/opt/php@8.1/sbin:$PATH"' >> /.zshenv
echo 'export PATH="/usr/local/opt/php@8.2/sbin:$PATH"' >> /.zshenv
```

