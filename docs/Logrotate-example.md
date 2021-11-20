随着时间的推移，Vaultwarden 日志文件会变得很大。使用 logrotate，我们可以定期轮换日志。

```sh
sudo nano /etc/logrotate.d/bitwarden
```

```sh
/var/log/bitwarden/*.log {
    # Perform logrotation as the bitwarden user and group
    su bitwarden bitwarden
    # Rotate daily
    daily
    # Rotate when the size is bigger than 5MB
    size 5M
    # Compress old log files
    compress
    # Keep 4 rotations of log files before removing or mailing to the address specified in a mail directive
    rotate 4
    # Truncate the original log file in place after creating a copy
    copytruncate
    # Don't panic if not found
    missingok
    # Don't rotate log if file is empty
    notifempty
    # Add date instaed of number to rotated log file
    dateext
    # Date format of dateext
    dateformat -%Y-%m-%d-%s
}
```

要在不手动解压缩的情况下查看压缩的日志文件：

```sh
zcat logfile.gz
zless logfile.gz
zgrep -i keyword_search logfile.gz
```