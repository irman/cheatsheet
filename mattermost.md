# Mattermost

## Upgrade commands

Adapted from the [official upgrade guide](https://docs.mattermost.com/administration/upgrade.html).

`{DATE}`: today's date.

1. Make sure there's no `mattermost` folder at home directory
1. Download the latest `tar.gz` to home folder.
    - Get the link from the [download page](https://about.mattermost.com/download/). Make sure to use the linux team edition.
    - `cd ~`
    - e.g: `wget https://releases.mattermost.com/4.4.2/mattermost-team-4.4.2-linux-amd64.tar.gz`
1. Extract it. This will extract it to `~/mattermost`
    - e.g: `tar -xzf mattermost-team-4.4.2-linux-amd64.tar.gz`
1. Stop service
    - `service mattermost stop`
1. Create backup folder:
    1. `cd ~/mattermost-backup`
    1. `mkdir {DATE}`
1. Backup SQL
    - `mysqldump -u mmuser -p mattermost > ~/mattermost-backup/{DATE}/mattermost.sql`
1. Move existing mattermost installation to backup folder
    - `mv /opt/mattermost ~/mattermost-backup/{DATE}/mattermost`
1. Move new version to the installation path
    - `mv ~/mattermost /opt/mattermost`
1. Restore configs and data from the old version
    - `cp -r ~/mattermost-backup/{DATE}/mattermost/config /opt/mattermost`
    - `cp -r ~/mattermost-backup/{DATE}/mattermost/data /opt/mattermost`
    - `cp -r ~/mattermost-backup/{DATE}/mattermost/logs /opt/mattermost`
1. Give proper ownership
    - `chown -R mattermost:mattermost /opt/mattermost`
1. (HTTPS ONLY) Activate `CAP_NET_BIND_SERVICE` capability
    - `cd /opt/mattermost/bin`
    - `setcap cap_net_bind_service=+ep ./bin/platform`
1. Start the mattermost service
    - `service mattermost start`
1. Upgrade the `config.json` schema (if needed)
    1. Open the **System Console** and change a setting, then revert it. This should enable the **Save** button for that page.
    1. Click **Save**.
    1. Refresh the page.


## Troubleshoot

### Run mattermost on cli manually
```
sudo -u mattermost /opt/mattermost/bin/platform
```

### Can't start mattermost because nginx is running

1. Find `nginx` processes:
    ```
    ps -ef |grep nginx
    ```
1. Kill it!
    ```
    kill -9 [PID]
    ```
