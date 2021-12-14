# Setup and start sshd

## Manual setup

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

## nix-on-droid config

You can also use this snippet to ease the manual setup process:

```nix
let
  sshdTmpDirectory = "${config.user.home}/sshd-tmp";
  sshdDirectory = "${config.user.home}/sshd";
  pathToPubKey = "...";
  port = 8022;
in
{
  build.activation.sshd = ''
    $DRY_RUN_CMD mkdir $VERBOSE_ARG --parents "${config.user.home}/.ssh"
    $DRY_RUN_CMD cat ${pathToPubKey} > "${config.user.home}/.ssh/authorized_keys"

    if [[ ! -d "${sshdDirectory}" ]]; then
      $DRY_RUN_CMD rm $VERBOSE_ARG --recursive --force "${sshdTmpDirectory}"
      $DRY_RUN_CMD mkdir $VERBOSE_ARG --parents "${sshdTmpDirectory}"

      $VERBOSE_ECHO "Generating host keys..."
      $DRY_RUN_CMD ${pkgs.openssh}/bin/ssh-keygen -t rsa -b 4096 -f "${sshdTmpDirectory}/ssh_host_rsa_key" -N ""

      $VERBOSE_ECHO "Writing sshd_config..."
      $DRY_RUN_CMD echo -e "HostKey ${sshdDirectory}/ssh_host_rsa_key\nPort ${toString port}\n" > "${sshdTmpDirectory}/sshd_config"

      $DRY_RUN_CMD mv $VERBOSE_ARG "${sshdTmpDirectory}" "${sshdDirectory}"
    fi
  '';

  environment.packages = [
    (pkgs.writeScriptBin "sshd-start" ''
      #!${pkgs.runtimeShell}

      echo "Starting sshd in non-daemonized way on port ${toString port}"
      ${pkgs.openssh}/bin/sshd -f "${sshdDirectory}/sshd_config" -D
    '')
  ];
}
```