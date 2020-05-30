For now you can follow these steps to create a working sshd server:

1. Install `openssh` via `nix-env` or `nix-on-droid`
2. Create host key: `ssh-keygen -t rsa -b 4096 -f ssh_host_rsa_key -N ""`
3. Create `sshd_config` config file with following content:
    ```
    HostKey /absolute/path/to/ssh_host_rsa_key
    Port 8022
    ```
4. Create `~/.ssh/authorized_keys` and add a public key
5. Run `sshd` with absolute path like: `$(which sshd) -f /path/to/sshd_config` (`-D` for disabling daemonizing, `-d` for debug output)
6. Connect to your phone via `ssh -p 8022 -l nix-on-droid -i /path/to/id_rsa <ip of phone>`

We will provide a module for the sshd config and service in the future to simplify the initial setup.

It is recommended to source `~/.nix-profile/etc/profile.d/nix-on-droid-session-init.sh` from `~/.profile`.

Furthermore the login shell is not recognized yet, so you may start your desired shell manually once logged via ssh.