Several users have reported success with offloading package-building to more powerful hosts:

t184256: [Simple remote building](https://github.com/t184256/nix-on-droid/wiki/Simple-remote-building) (using existing NixOS, binary qemu-aarch64)

bbigras: [Use a remote builder with qemu](https://github.com/t184256/nix-on-droid/wiki/Use-a-remote-builder-with-qemu) (using existing NixOS, building your own qemu-aarch64)

573: [Same machine remote builder how-to](https://github.com/t184256/nix-on-droid/wiki/Same-machine-remote-builder-how-to/95f3a6f1177243920537b3f407b81b8d214086a6) (includes spinning up a VM with nixos-shell, DEPRECATED, see [Same machine remote builder how-to discussion page](https://github.com/t184256/nix-on-droid/discussions/102))

# Warning

If you are rebuilding your whole nix-on-droid system remotely you might get broken uid/gid. This happens because impurity in [this derivation](https://github.com/t184256/nix-on-droid/blob/2301e01d48c90b60751005317de7a84a51a87eb6/modules/user.nix#L10-L17) which when built remotely will get different ids. To fix this you can specify uid/gid explicitly.