---
seo:
    description: Update xrpld on Red Hat Enterprise Linux.
labels:
  - Core Server
  - Security
---
# Update on Red Hat Enterprise Linux

This page describes how to keep `xrpld` up to date on Red Hat Enterprise Linux. You can configure scheduled package updates with `systemd`, or run an update immediately when a new release is available.

These instructions assume you have already [installed `xrpld` on a supported version of Red Hat Enterprise Linux using Ripple's `rpm` package distribution](install-xrpld-on-rhel.md). If you are upgrading from `xrpld` 1.6.x or older, remove it and perform a fresh install instead.

{% admonition type="info" name="Note" %}Installing a new package version does not restart the `xrpld` service. The server continues to run the old version until you restart it, so plan a restart after package updates are installed.{% /admonition %}


## Scheduled Updates

On RHEL, you can use a `systemd` service and timer to check for package updates regularly. The service follows the Ripple RPM repository channel you configured when you installed `xrpld`, such as `stable`, `unstable`, or `nightly`.

1. Create a service to update the `xrpld` package:

    ```sh
    cat <<'EOF' | sudo tee /etc/systemd/system/xrpld-upgrade.service
    [Unit]
    Description=Check for and install xrpld package updates
    Wants=network-online.target
    After=network-online.target

    [Service]
    Type=oneshot
    ExecStart=/usr/bin/dnf -y --refresh upgrade xrpld
    TimeoutStartSec=30min
    EOF
    ```

2. Create a timer to run the update service daily:

    ```sh
    cat <<'EOF' | sudo tee /etc/systemd/system/xrpld-upgrade.timer
    [Unit]
    Description=Daily xrpld update check

    [Timer]
    OnCalendar=daily
    RandomizedDelaySec=4h
    Persistent=true

    [Install]
    WantedBy=timers.target
    EOF
    ```

    The randomized delay helps avoid too many servers checking for updates at the same time.

3. Enable the timer:

    ```sh
    sudo systemctl daemon-reload
    sudo systemctl enable --now xrpld-upgrade.timer
    ```

4. Check when the timer runs next:

    ```sh
    systemctl list-timers xrpld-upgrade.timer
    ```

5. After an update is installed, restart `xrpld` during a maintenance window:

    ```sh
    sudo systemctl restart xrpld.service
    ```


## One-Time Scheduled Check

To schedule a one-time update check for 30 minutes from now, run:

```sh
sudo systemd-run \
    --on-active=30m \
    --unit=xrpld-upgrade-once \
    systemctl start xrpld-upgrade.service
```

After the update check runs, restart `xrpld` during a maintenance window if a new package version was installed.


## Immediate Update

To update `xrpld` now, complete the following steps:

1. Download and install the latest `xrpld` package:

    ```sh
    sudo yum update xrpld
    ```

    This update procedure leaves your existing config files in place.

2. Reload the `systemd` unit files:

    ```sh
    sudo systemctl daemon-reload
    ```

3. Restart the `xrpld` service:

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
