# Use a remote builder with qemu

It seems possible to use a remote builder to avoid building packages on Android.

qemu.nix: https://github.com/bqv/nixos/blob/live/profiles/misc/qemu.nix
overlay: https://github.com/bqv/nixos/tree/live/overlays/qemu

## Builder side

```nix
let
  qemuOverlay = (import ./overlays/qemu);
in
{
  imports = [ ./qemu.nix ];

  boot.kernelModules = [ "kvm-intel" ];
  qemu-user.arm = true;
  boot.binfmt.emulatedSystems = [ "aarch64-linux" ];

  nix = {
    trustedUsers = [ "builder" ];
  };

  users.users.builder = {
    createHome = true;
    isNormalUser = true;
  };
}
```

## Android side

~/.config/nix/nix.conf
```conf
builders-use-substitutes = true
builders = ssh://builder
```

~/.ssh/config
```conf
Host builder
    HostName <the ip>
    User builder
    IdentitiesOnly yes
    IdentityFile ~/.ssh/nix_remote
```

~/.ssh/nix_remote is a ssh key without a password. Add the pub key on the builder machine.

You need to ssh first to the host to make ssh accept the host key.

You can test with `nix build '(with import <nixpkgs> { }; runCommand "foo" {} "uname -a > $out")' --builders 'ssh://builder' -j0`

or `nix-on-droid switch --max-job 0`