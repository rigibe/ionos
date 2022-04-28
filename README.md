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
