Building remotely with qemu-user-aarch64,
using a prebuilt qemu user emulation binary.

# Prerequisites:
1. a powerful NixOS 20.09 x86_64 machine
2. that you can ssh to using `ssh HOST` without passwords (e.g., with a passwordless key).

# Builder

Add the following to the system configuration:

``` nix
let
  qemu-aarch64-static = pkgs.stdenv.mkDerivation {
    name = "qemu-aarch64-static";

    src = builtins.fetchurl {
      url = "https://github.com/multiarch/qemu-user-static/releases/download/v5.
1.0-7/qemu-aarch64-static";
      sha256 = "0yzlrlknslvas58msrbbq3hazphyydrbaqrd840bd1c7vc9lcrh6";
    };

    dontUnpack = true;
    installPhase = "install -D -m 0755 $src $out/bin/qemu-aarch64-static";
  };
in
{
  # ...

  boot.binfmt.registrations.aarch64 = {
    interpreter = "${qemu-aarch64-static}/bin/qemu-aarch64-static";
    magicOrExtension = ''\x7fELF\x02\x01\x01\x00\x00\x00\x00\x00\x00\x00\x00\x00\x02\x00\xb7\x00'';
    mask = ''\xff\xff\xff\xff\xff\xff\xff\x00\xff\xff\xff\xff\xff\xff\x00\xff\xfe\xff\xff\xff'';
  };
  nix.extraOptions = ''
    extra-platforms = aarch64-linux
    trusted-users = USER
  '';
  nix.sandboxPaths = [ "/run/binfmt/aarch64=${qemu-aarch64-static}/bin/qemu-aarch64-static" ];
}
```

CAUTION: if you're using current unstable/flake-powered Nix without https://github.com/NixOS/nixpkgs/pull/103137,
don't set `nix.sandboxPaths`, but set
`sandbox-paths = /bin/sh=${pkgs.busybox-sandbox-shell}/bin/busybox /run/binfmt/aarch64=${qemu-aarch64-static}/bin/qemu-aarch64-static` instead.

# Android side

```
mkdir -p ~/.config/nix
echo -e "builders-use-substitutes = true\nbuilders = ssh-ng://USER@HOST" >> ~/.config/nix/nix.conf
```