## Services

> @TODO: move service management to Ansible control and separate playbook.

### PHP

See `.bash_profile`.

`PATH="/usr/local/opt/php@8.2/bin:$PATH"`
`PATH="/usr/local/opt/php@8.2/sbin:$PATH"`

```bash
#setup daemon
mkdir -p ~/Library/LaunchAgents
cp /usr/local/opt/php@8.2/homebrew.mxcl.php@8.2.plist ~/Library/LaunchAgents/
launchctl load -w ~/Library/LaunchAgents/homebrew.mxcl.php@8.2.plist
```

```bash
# /usr/local/etc/php/8.2/php.ini

memory_limit = 1G
date.timezone = America/Los_Angeles
upload_max_filesize = 32M
post_max_size = 64M
```

```bash
brew services start php@8.2
```

```bash
pecl uninstall -r apcu && pecl install apcu && pecl uninstall -r yaml && pecl install yaml && pecl uninstall -r xdebug && pecl install xdebug
```

#### Composer for PHP
Composer is installed by Ansible with Homebrew when the main playbook is run.

See `.exports`.

```bash
export COMPOSER_PROCESS_TIMEOUT=2000
export COMPOSER_MEMORY_LIMIT=-1
```

### Ruby
#### rbenv

*Compatibility note:* rbenv is incompatible with RVM. Please make sure to fully uninstall RVM and remove any references to it from your shell initialization files before installing rbenv.

### MySQL
```bash
# create mysql config
cp -v $(brew --prefix mysql)/support-files/my-default.cnf $(brew --prefix)/etc/my.cnf
```

```bash
cat >> $(brew --prefix)/etc/my.cnf <<'EOF'

# Settings overrides
innodb_buffer_pool_size = 128M
max_allowed_packet = 1073741824
innodb_file_per_table = 1
EOF
```

```bash
# start mysql
brew services start mysql
```

```bash
# secure mysql
$(brew --prefix mysql)/bin/mysql_secure_installation
```

### Apache
```bash
# Stop the built-in Apache
sudo apachectl stop
sudo launchctl unload -w /System/Library/LaunchDaemons/org.apache.httpd.plist 2>/dev/null
```

```bash
# Install Apache 2.4
brew install httpd
```

```bash
# Ensure Apache server is auto-started
sudo brew services start httpd
```

#### Apache Configuration
```bash
# /usr/local/etc/httpd/httpd.conf
```

### NGINX
```bash
sudo cp -v /usr/local/opt/nginx/*.plist /Library/LaunchDaemons/ &&
sudo chown root:wheel /Library/LaunchDaemons/homebrew.mxcl.nginx.plist &&
sudo launchctl load /Library/LaunchDaemons/homebrew.mxcl.nginx.plist
```

### Local Web Server

#### Add DNS Domains, Enable dnsmasq daemon
This will route requests to any url ending in **.build** back to your own computer. The goal is to use urls like http://example.com.build for development while you work on http://example.com

```bash
mkdir -pv $(brew --prefix)/etc/ && \
echo 'address=/.build/127.0.0.1' > $(brew --prefix)/etc/dnsmasq.conf && \
sudo cp -v $(brew --prefix dnsmasq)/homebrew.mxcl.dnsmasq.plist /Library/LaunchDaemons && \
sudo launchctl load -w /Library/LaunchDaemons/homebrew.mxcl.dnsmasq.plist && \
sudo mkdir -v /etc/resolver && \
sudo zsh -c 'echo "nameserver 127.0.0.1" > /etc/resolver/build'

#flush cache
sudo discoveryutil mdnsflushcache && scutil --dns
```

#### Enable virtual hosts
This will allow you to serve folders under ~/Sites/ as websites.

* ~/Sites
  * example.com
    * htdocs
      * index.html

to access this site, visit http://example.com.build


#### Match production server paths
```bash
sudo mkdir -p /var/ && sudo ln -s ~/Sites /var/www
```

### Drupal

#### Drush Launcher
Follow the setup instructions on the [Drush Launcher](https://github.com/drush-ops/drush-launcher) project page.
