---
seo:
    description: Update xrpld on Ubuntu or Debian Linux.
labels:
  - Core Server
  - Security
---
# Update on Ubuntu or Debian

This page describes how to keep `xrpld` up to date on Ubuntu or Debian Linux. You can configure scheduled package updates with `unattended-upgrades`, or run an update immediately when a new release is available.

These instructions assume you have already [installed `xrpld` on a supported version of Ubuntu or Debian using Ripple's `deb` package](install-xrpld-on-ubuntu.md). If you are upgrading from `xrpld` 1.6.x or older, remove it and perform a fresh install instead.

{% admonition type="info" name="Note" %}Installing a new package version does not restart the `xrpld` service. The server continues to run the old version until you restart it, so plan a restart after package updates are installed.{% /admonition %}


## Scheduled Updates

On Ubuntu and Debian, use `unattended-upgrades` to install package updates from the Ripple `deb` repository. This configuration follows the repository channel you select, such as `stable`, `unstable`, or `nightly`.

1. Install `unattended-upgrades`:

    ```sh
    sudo apt -y update
    sudo apt -y install unattended-upgrades
    ```

2. Choose what `unattended-upgrades` is allowed to update.

    Create `/etc/apt/apt.conf.d/51xrpld-auto-upgrade` using one of the following configurations. You can control the update scope from this file; you don't need to edit the default unattended-upgrades config.

    To update only `xrpld`, use this configuration. The `#clear` directives remove the default unattended-upgrades origins from other APT config files, so this automatic update configuration does not also install operating system updates.

    ```sh
    cat <<'EOF' | sudo tee /etc/apt/apt.conf.d/51xrpld-auto-upgrade
    #clear Unattended-Upgrade::Allowed-Origins;
    #clear Unattended-Upgrade::Origins-Pattern;
    Unattended-Upgrade::Origins-Pattern:: "site=repos.ripple.com,component=stable";
    Unattended-Upgrade::Package-Whitelist:: "xrpld";
    EOF
    ```

    To let `unattended-upgrades` update `xrpld` and any other packages from origins that are already allowed by the system, omit the `#clear` directives and the package whitelist:

    ```sh
    cat <<'EOF' | sudo tee /etc/apt/apt.conf.d/51xrpld-auto-upgrade
    Unattended-Upgrade::Origins-Pattern:: "site=repos.ripple.com,component=stable";
    EOF
    ```

    In either configuration, if you installed `xrpld` from the `unstable` or `nightly` channel, replace `stable` with that channel name.

    {% admonition type="warning" name="Caution" %}APT combines unattended-upgrades settings from all files in `/etc/apt/apt.conf.d/`. Without the `#clear` directives, unattended-upgrades can install updates for packages from any allowed origin configured on the system, not only `xrpld`. In APT configuration, `#clear` is a directive, not a commented-out line, so copy those lines exactly if you want to update only `xrpld`. On many systems, the default configuration also allows operating system security updates.{% /admonition %}

    To check the merged configuration, run:

    ```sh
    apt-config dump | grep -E 'Unattended-Upgrade::(Allowed-Origins|Origins-Pattern|Package-Whitelist)'
    ```

3. Make sure periodic updates are enabled in `/etc/apt/apt.conf.d/20auto-upgrades`:

    ```txt
    APT::Periodic::Update-Package-Lists "1";
    APT::Periodic::Unattended-Upgrade "1";
    ```

4. Check the timers that run package updates:

    ```sh
    systemctl list-timers apt-daily.timer apt-daily-upgrade.timer
    ```

5. Optional: run the update check immediately.

    To check what would happen without installing anything, run:

    ```sh
    sudo unattended-upgrade --dry-run --debug
    ```

    In the debug output, `Allowed origins` shows which repositories `unattended-upgrades` can use, and the `applying set` lines show the packages it would actually install. If you used the `xrpld`-only configuration above, `Allowed origins` should include only the Ripple repository and the `applying set` lines should include only `xrpld`.

    If you filter the debug output, include `applying set` in your filter so you don't hide packages that do not include `xrpld` in their names:

    ```sh
    sudo unattended-upgrade --dry-run --debug 2>&1 | \
        grep -iE 'allowed origins|initial whitelist|packages that will be upgraded|applying set|left to upgrade'
    ```

    To install matching updates now, run the same command without `--dry-run`:

    ```sh
    sudo unattended-upgrade --debug
    ```

6. After an update is installed, restart `xrpld` during a maintenance window:

    ```sh
    sudo systemctl restart xrpld.service
    ```


## One-Time Scheduled Check

To schedule a one-time update check for 30 minutes from now, run:

```sh
sudo systemd-run \
    --on-active=30m \
    --unit=xrpld-upgrade-once \
    /usr/bin/unattended-upgrade --debug
```

This uses the `unattended-upgrades` configuration from the scheduled update steps, including the `xrpld` package whitelist.

To check when the one-time timer runs, use:

```sh
systemctl list-timers xrpld-upgrade-once.timer
```

To check the output after it runs, use:

```sh
journalctl -u xrpld-upgrade-once.service
```

After the update check runs, restart `xrpld` during a maintenance window if a new package version was installed.


## Immediate Update

To update `xrpld` now, complete the following steps:

1. Update repositories:

    ```sh
    sudo apt -y update
    ```

2. Upgrade the `xrpld` package:

    ```sh
    sudo apt -y install --only-upgrade xrpld
    ```

3. Reload the `systemd` unit files:

    ```sh
    sudo systemctl daemon-reload
    ```

4. Restart the `xrpld` service:

    ```sh
    sudo systemctl restart xrpld.service
    ```


## See Also

- **Concepts:**
    - [The `xrpld` Server](../../concepts/networks-and-servers/index.md)
    - [Consensus](../../concepts/consensus-protocol/index.md)
- **Tutorials:**
    - [Capacity Planning](capacity-planning.md)
    - [Troubleshoot xrpld](../troubleshooting/index.md)
- **References:**
    - [xrpld API Reference](../../references/http-websocket-apis/index.md)
        - [`xrpld` Commandline Usage](../commandline-usage.md)
        - [server_info method][]

{% raw-partial file="/docs/_snippets/common-links.md" /%}
