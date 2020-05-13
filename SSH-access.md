Lifted from https://github.com/t184256/nix-on-droid/issues/32#issuecomment-575930345:

> For now you can follow these steps to create a working sshd server:

>     Install openssh via nix-env or nix-on-droid or what ever.
>     Create host key: ssh-keygen -t rsa -b 4096 -f ssh_host_rsa_key -N ""
>     Create sshd_config config file with following content:

>     HostKey /absolute/path/to/ssh_host_rsa_key
>     Port 8022

>     Create ~/.ssh/authorized_keys and add a public key
>     Run sshd with absolute path like: /data/data/com.termux.nix/files/home/.nix-profile/bin/sshd -f /path/to/sshd_config
>     Connect to your phone via ssh -p 8022 -l nix-on-droid -i /path/to/id_rsa <ip of phone>

> We will provide a module for the sshd config and service in the future to simplify the initial setup.


Later on, https://github.com/t184256/nix-on-droid/issues/32 also suggests sourcing `~/.nix-profile/etc/profile.d/nix-on-droid-session-init.sh` from `~/.profile`.