# Ionos Server

## Start

### User

Create User without home:
```sudo adduser --no-create-home rgb```

At the end I have created a user with the home.

Add User to Sudo:
```sudo usermod -aG sudo rgb```

Change password:
```sudo passwd root```

### Secure `/etc/ssh/sshd_config`

#### Steps

1. Make a backup of OpenSSH server's configuration file `/etc/ssh/sshd_config` and remove comments to make it easier to read:

    ``` bash
    sudo cp --archive /etc/ssh/sshd_config /etc/ssh/sshd_config-COPY-$(date +"%Y%m%d%H%M%S")
    sudo sed -i -r -e '/^#|^$/ d' /etc/ssh/sshd_config
    ```

1. Edit `/etc/ssh/sshd_config` then find and edit or add these settings that should be applied regardless of your configuration/setup:

    **Note**: SSH does not like duplicate contradicting settings. For example, if you have `ChallengeResponseAuthentication no` and then `ChallengeResponseAuthentication yes`, SSH will respect the first one and ignore the second. Your `/etc/ssh/sshd_config` file may already have some of the settings/lines below. To avoid issues you will need to manually go through your `/etc/ssh/sshd_config` file and address any duplicate contradicting settings. (If anyone knows a way to programatically do this I would [love to hear how](#contacting-me).)

    ```
    ########################################################################################################
    # start settings from https://infosec.mozilla.org/guidelines/openssh#modern-openssh-67 as of 2019-01-01
    ########################################################################################################

    # Supported HostKey algorithms by order of preference.
    HostKey /etc/ssh/ssh_host_ed25519_key
    HostKey /etc/ssh/ssh_host_rsa_key
    HostKey /etc/ssh/ssh_host_ecdsa_key

    KexAlgorithms curve25519-sha256@libssh.org,ecdh-sha2-nistp521,ecdh-sha2-nistp384,ecdh-sha2-nistp256,diffie-hellman-group-exchange-sha256

    Ciphers chacha20-poly1305@openssh.com,aes256-gcm@openssh.com,aes128-gcm@openssh.com,aes256-ctr,aes192-ctr,aes128-ctr

    MACs hmac-sha2-512-etm@openssh.com,hmac-sha2-256-etm@openssh.com,umac-128-etm@openssh.com,hmac-sha2-512,hmac-sha2-256,umac-128@openssh.com

    # LogLevel VERBOSE logs user's key fingerprint on login. Needed to have a clear audit track of which key was using to log in.
    LogLevel VERBOSE

    # Use kernel sandbox mechanisms where possible in unprivileged processes
    # Systrace on OpenBSD, Seccomp on Linux, seatbelt on MacOSX/Darwin, rlimit elsewhere.
    # Note: This setting is deprecated in OpenSSH 7.5 (https://www.openssh.com/txt/release-7.5)
    UsePrivilegeSeparation sandbox

    ########################################################################################################
    # end settings from https://infosec.mozilla.org/guidelines/openssh#modern-openssh-67 as of 2019-01-01
    ########################################################################################################

    # don't let users set environment variables
    PermitUserEnvironment no

    # Log sftp level file access (read/write/etc.) that would not be easily logged otherwise.
    Subsystem sftp  internal-sftp -f AUTHPRIV -l INFO

    # only use the newer, more secure protocol
    Protocol 2

    # disable X11 forwarding as X11 is very insecure
    # you really shouldn't be running X on a server anyway
    X11Forwarding no

    # disable port forwarding
    AllowTcpForwarding no
    AllowStreamLocalForwarding no
    GatewayPorts no
    PermitTunnel no

    # don't allow login if the account has an empty password
    PermitEmptyPasswords no

    # ignore .rhosts and .shosts
    IgnoreRhosts yes

    # verify hostname matches IP
    UseDNS no

    Compression no
    TCPKeepAlive no
    AllowAgentForwarding no
    PermitRootLogin no

    # don't allow .rhosts or /etc/hosts.equiv
    HostbasedAuthentication no
    ```

1. Then **find and edit or add** these settings, and set values as per your requirements:

    |Setting|Valid Values|Example|Description|Notes|
    |--|--|--|--|--|
    |<a name="AllowGroups"></a>**AllowGroups**|local UNIX group name|`AllowGroups sshusers`|group to allow SSH access to||
    |**ClientAliveCountMax**|number|`ClientAliveCountMax 0`|maximum number of client alive messages sent without response||
    |**ClientAliveInterval**|number of seconds|`ClientAliveInterval 300`|timeout in seconds before a response request||
    |**ListenAddress**|space separated list of local addresses|<ul><li>`ListenAddress 0.0.0.0`</li><li>`ListenAddress 192.168.1.100`</li></ul>|local addresses `sshd` should listen on|See [Issue #1](https://github.com/imthenachoman/How-To-Secure-A-Linux-Server/issues/1) for important details.|
    |**LoginGraceTime**|number of seconds|`LoginGraceTime 30`|time in seconds before login times-out||
    |**MaxAuthTries**|number|`MaxAuthTries 2`|maximum allowed attempts to login||
    |**MaxSessions**|number|`MaxSessions 2`|maximum number of open sessions||
    |**MaxStartups**|number|`MaxStartups 2`|maximum number of login sessions||
    |<a name="PasswordAuthentication"></a>**PasswordAuthentication**|`yes` or `no`|`PasswordAuthentication no`|if login with a password is allowed||
    |**Port**|any open/available port number|`Port 22`|port that `sshd` should listen on||

    Check `man sshd_config` for more details what these settings mean.

1. Restart ssh:

    ``` bash
    sudo service sshd restart
    ```

1. You can check verify the configurations worked with `sshd -T` and verify the output:

    ``` bash
    sudo sshd -T
    ```

    My file is:


    >```
    >Include /etc/ssh/sshd_config.d/*.conf
    >ChallengeResponseAuthentication yes
    >UsePAM yes
    >#X11Forwarding yes
    >PrintMotd no
    >AcceptEnv LANG LC_*
    >#Subsystem	sftp	/usr/lib/openssh/sftp-server
    >#PermitRootLogin yes
    >PasswordAuthentication no
    >
    >########################################################################################################
    ># start settings from https://infosec.mozilla.org/guidelines/openssh#modern-openssh-67 as of 2019-01-01
    >########################################################################################################
    >
    ># Supported HostKey algorithms by order of preference.
    >HostKey /etc/ssh/ssh_host_ed25519_key
    >HostKey /etc/ssh/ssh_host_rsa_key
    >HostKey /etc/ssh/ssh_host_ecdsa_key
    >
    >KexAlgorithms curve25519-sha256@libssh.org,ecdh-sha2-nistp521,ecdh-sha2-nistp384,ecdh-sha2-nistp256,diffie-hellman-group-exchange-sha256
    >
    >Ciphers chacha20-poly1305@openssh.com,aes256-gcm@openssh.com,aes128-gcm@openssh.com,aes256-ctr,aes192-ctr,aes128-ctr
    >
    >MACs hmac-sha2-512-etm@openssh.com,hmac-sha2-256-etm@openssh.com,hmac-sha2-512,hmac-sha2-256,umac-128@openssh.com
    >
    ># LogLevel VERBOSE logs user's key fingerprint on login. Needed to have a clear audit track of which key was using to log in.
    >LogLevel VERBOSE
    >
    ># Use kernel sandbox mechanisms where possible in unprivileged processes
    ># Systrace on OpenBSD, Seccomp on Linux, seatbelt on MacOSX/Darwin, rlimit elsewhere.
    ># Note: This setting is deprecated in OpenSSH 7.5 (https://www.openssh.com/txt/release-7.5)
    ># UsePrivilegeSeparation sandbox
    >
    >########################################################################################################
    ># end settings from https://infosec.mozilla.org/guidelines/openssh#modern-openssh-67 as of 2019-01-01
    >########################################################################################################
    >
    ># don't let users set environment variables
    >PermitUserEnvironment no
    >
    ># Log sftp level file access (read/write/etc.) that would not be easily logged otherwise.
    >Subsystem sftp  internal-sftp -f AUTHPRIV -l INFO
    >
    ># only use the newer, more secure protocol
    >Protocol 2
    >
    ># disable X11 forwarding as X11 is very insecure
    ># you really shouldn't be running X on a server anyway
    >X11Forwarding no
    >
    ># disable port forwarding
    >AllowTcpForwarding yes # needed for VSC
    >AllowStreamLocalForwarding no
    >GatewayPorts no
    >PermitTunnel no
    >
    ># don't allow login if the account has an empty password
    >PermitEmptyPasswords no
    >
    ># ignore .rhosts and .shosts
    >IgnoreRhosts yes
    >
    ># verify hostname matches IP
    >UseDNS yes
    >
    >Compression no
    >TCPKeepAlive no
    >AllowAgentForwarding no
    >PermitRootLogin no
    >
    ># don't allow .rhosts or /etc/hosts.equiv
    >HostbasedAuthentication no
    >
    ># Custom Options
    >ClientAliveCountMax 0
    >ClientAliveInterval 300
    >LoginGraceTime 30
    >MaxAuthTries 2
    >MaxSessions 2
    >MaxStartups 2
    >
    >Port 1221
    >```


([Table of Contents](#table-of-contents))

## The Network

### Firewall With UFW (Uncomplicated Firewall)

**UFW** works by letting you configure rules that:

- **allow** or **deny**
- **input** or **output** traffic
- **to** or **from** ports

You can create rules by explicitly specifying the ports or with application configurations that specify the ports.

#### Steps

1. Install ufw.

    On Debian based systems:

    ``` bash
    sudo apt install ufw
    ```

1. Allow all outgoing traffic:

    ``` bash
    sudo ufw default allow outgoing comment 'allow all outgoing traffic'
    ```

1. Deny all incoming traffic:

    ``` bash
    sudo ufw default deny incoming comment 'deny all incoming traffic'
    ```

1. Set the rules:

    ``` bash
    sudo ufw limit in 1221 comment 'allow SSH connections in'
    ```

    > ```
    > Rules updated
    > Rules updated (v6)
    > ```

1. Start ufw:

    ``` bash
    sudo ufw enable
    ```

    > ```
    > Command may disrupt existing ssh connections. Proceed with operation (y|n)? y
    > Firewall is active and enabled on system startup
    > ```

1. Fix ufw service not loading after a reboot

    ufw (Uncomplicated Firewall) and Docker. Docker relies on iptables-persistent, which is an interface to a much more powerful and complicated firewall that many people would rather avoid. The problem here is that ufw and iptables-persistent are both ways for creating the same firewall.

    `sudo nano /lib/systemd/system/ufw.service`

    Add `After=netfilter-persistent.service` like this:

    ```
    [Unit]
    Description=Uncomplicated firewall
    Documentation=man:ufw(8)
    DefaultDependencies=no
    Before=network.target
    After=netfilter-persistent.service

    [Service]
    Type=oneshot
    RemainAfterExit=yes
    ExecStart=/lib/ufw/ufw-init start quiet
    ExecStop=/lib/ufw/ufw-init stop

    [Install]
    WantedBy=multi-user.target
    ```

1. If you want to see a status:

    ``` bash
    sudo ufw status
    ```

    > ```
    > Status: active
    > 
    > To                         Action      From
    > --                         ------      ----
    > 22/tcp                     LIMIT       Anywhere                   # allow SSH connections in
    > 22/tcp (v6)                LIMIT       Anywhere (v6)              # allow SSH connections in
    > 
    > 53                         ALLOW OUT   Anywhere                   # allow DNS calls out
    > 123                        ALLOW OUT   Anywhere                   # allow NTP out
    > 80/tcp                     ALLOW OUT   Anywhere                   # allow HTTP traffic out
    > 443/tcp                    ALLOW OUT   Anywhere                   # allow HTTPS traffic out
    > 21/tcp                     ALLOW OUT   Anywhere                   # allow FTP traffic out
    > Mail submission            ALLOW OUT   Anywhere                   # allow mail out
    > 43/tcp                     ALLOW OUT   Anywhere                   # allow whois
    > 53 (v6)                    ALLOW OUT   Anywhere (v6)              # allow DNS calls out
    > 123 (v6)                   ALLOW OUT   Anywhere (v6)              # allow NTP out
    > 80/tcp (v6)                ALLOW OUT   Anywhere (v6)              # allow HTTP traffic out
    > 443/tcp (v6)               ALLOW OUT   Anywhere (v6)              # allow HTTPS traffic out
    > 21/tcp (v6)                ALLOW OUT   Anywhere (v6)              # allow FTP traffic out
    > Mail submission (v6)       ALLOW OUT   Anywhere (v6)              # allow mail out
    > 43/tcp (v6)                ALLOW OUT   Anywhere (v6)              # allow whois
    > ```

    or

    ``` bash
    sudo ufw status verbose
    ```

    > ```
    > Status: active
    > Logging: on (low)
    > Default: deny (incoming), deny (outgoing), disabled (routed)
    > New profiles: skip
    > 
    > To                         Action      From
    > --                         ------      ----
    > 22/tcp                     LIMIT IN    Anywhere                   # allow SSH connections in
    > 22/tcp (v6)                LIMIT IN    Anywhere (v6)              # allow SSH connections in
    > 
    > 53                         ALLOW OUT   Anywhere                   # allow DNS calls out
    > 123                        ALLOW OUT   Anywhere                   # allow NTP out
    > 80/tcp                     ALLOW OUT   Anywhere                   # allow HTTP traffic out
    > 443/tcp                    ALLOW OUT   Anywhere                   # allow HTTPS traffic out
    > 21/tcp                     ALLOW OUT   Anywhere                   # allow FTP traffic out
    > 587/tcp (Mail submission)  ALLOW OUT   Anywhere                   # allow mail out
    > 43/tcp                     ALLOW OUT   Anywhere                   # allow whois
    > 53 (v6)                    ALLOW OUT   Anywhere (v6)              # allow DNS calls out
    > 123 (v6)                   ALLOW OUT   Anywhere (v6)              # allow NTP out
    > 80/tcp (v6)                ALLOW OUT   Anywhere (v6)              # allow HTTP traffic out
    > 443/tcp (v6)               ALLOW OUT   Anywhere (v6)              # allow HTTPS traffic out
    > 21/tcp (v6)                ALLOW OUT   Anywhere (v6)              # allow FTP traffic out
    > 587/tcp (Mail submission (v6)) ALLOW OUT   Anywhere (v6)              # allow mail out
    > 43/tcp (v6)                ALLOW OUT   Anywhere (v6)              # allow whois
    > ```

#### Default Applications

ufw ships with some default applications. You can see them with:

``` bash
sudo ufw app list
```

> ```
> Available applications:
>   AIM
>   Bonjour
>   CIFS
>   DNS
>   Deluge
>   IMAP
>   IMAPS
>   IPP
>   KTorrent
>   Kerberos Admin
>   Kerberos Full
>   Kerberos KDC
>   Kerberos Password
>   LDAP
>   LDAPS
>   LPD
>   MSN
>   MSN SSL
>   Mail submission
>   NFS
>   OpenSSH
>   POP3
>   POP3S
>   PeopleNearby
>   SMTP
>   SSH
>   Socks
>   Telnet
>   Transmission
>   Transparent Proxy
>   VNC
>   WWW
>   WWW Cache
>   WWW Full
>   WWW Secure
>   XMPP
>   Yahoo
>   qBittorrent
>   svnserve
> ```

To get details about the app, like which ports it includes, type:

``` bash
sudo ufw app info [app name]
```

> ``` bash
> sudo ufw app info DNS
> ```
> 
> ```
> Profile: DNS
> Title: Internet Domain Name Server
> Description: Internet Domain Name Server
> 
> Port:
>   53
> ```

#### Custom Application

If you don't want to create rules by explicitly providing the port number(s), you can create your own application configurations. To do this, create a file in `/etc/ufw/applications.d`.

For example, here is what you would use for [Plex](https://support.plex.tv/articles/201543147-what-network-ports-do-i-need-to-allow-through-my-firewall/):

``` bash
cat /etc/ufw/applications.d/plexmediaserver
```

> ```
> [PlexMediaServer]
> title=Plex Media Server
> description=This opens up PlexMediaServer for http (32400), upnp, and autodiscovery.
> ports=32469/tcp|32413/udp|1900/udp|32400/tcp|32412/udp|32410/udp|32414/udp|32400/udp
> ```

Then you can enable it like any other app:

```bash
sudo ufw allow plexmediaserver
```

([Table of Contents](#table-of-contents))
