DISCLAIMER!
This write-up just serves as a proof of concept and is by no means complete or intended for production use. Pls don't hesitate to ask the op in case of questions or findings.

The [remote builder](Use-a-remote-builder-with-qemu) wiki page already describes how to bridge the gap between different architectures when sourcing out the build.

One additional question I had in mind when trying what I describe next was how to have that remote machine hosting the aarch64 build in a vm, thus not neccessarily leaving the local machine at all. Latter which is of advantage if your on a network with restricted routing (i. e. via Cisco AnyConnect). Also the local machine can basically run any OS capable of hosting nix, so doesn't need to run NixOS. I'm using nix in an arch linux via WSL (not WSL2) on Windows 10 for this (`Linux 4.4.0-19041-Microsoft #488-Microsoft Mon Sep 01 13:43:00 PST 2020 x86_64 GNU/Linux`).

I'll show you and recommend the approach using [nixos-shell](https://github.com/Mic92/nixos-shell).

Once you cloned nixos-shell (my patches are based on commit cac57fac668ee3849276531bd1f03604ed568dd0) please patch `Makefile` and `examples/vm.nix`


```
diff --git a/Makefile b/Makefile
index 2a6a321..a8165ee 100644
--- a/Makefile
+++ b/Makefile
@@ -4,6 +4,12 @@ NIXOS_SHELL ?= $(shell nix-build --no-out-link default.nix)/bin/nixos-shell
 all:
 
 test:
+	curl --create-dirs -C - -o examples/qemu.nix https://raw.githubusercontent.com/cleverca22/nixos-configs/048c010eb08e25c5b2a90e993d98654d5b0833b7/qemu.nix
+	curl --create-dirs -C - -o examples/overlays/qemu/default.nix https://raw.githubusercontent.com/cleverca22/nixos-configs/048c010eb08e25c5b2a90e993d98654d5b0833b7/overlays/qemu/default.nix
+	curl --create-dirs -o examples/overlays/qemu/qemu/default.nix https://raw.githubusercontent.com/cleverca22/nixos-configs/048c010eb08e25c5b2a90e993d98654d5b0833b7/overlays/qemu/qemu/default.nix
+	curl --create-dirs -C - -o examples/overlays/qemu/qemu/qemu-stack.patch https://raw.githubusercontent.com/cleverca22/nixos-configs/048c010eb08e25c5b2a90e993d98654d5b0833b7/overlays/qemu/qemu/qemu-stack.patch
+	curl --create-dirs -C - -o examples/overlays/qemu/qemu/qemu-wrap.c https://raw.githubusercontent.com/cleverca22/nixos-configs/048c010eb08e25c5b2a90e993d98654d5b0833b7/overlays/qemu/qemu/qemu-wrap.c
+	sed -i 's!"--disable-bluez"!!g; s!python!python3!g; s!"-lglib-2.0"!"-lglib-2.0" "-lpthread"!g' examples/overlays/qemu/qemu/default.nix
 	$(NIXOS_SHELL) examples/vm.nix
 
 test-resources:
diff --git a/examples/vm.nix b/examples/vm.nix
index 54480b8..3fb1cc8 100644
--- a/examples/vm.nix
+++ b/examples/vm.nix
@@ -1,4 +1,30 @@
 { pkgs, ... }: {
+  imports = [ ./qemu.nix ];
   boot.kernelPackages = pkgs.linuxPackages_latest;
+  boot.kernelModules = [ "kvm-intel" ];
+  qemu-user.aarch64 = true;
+  boot.binfmt.emulatedSystems = [ "aarch64-linux" ];
   services.openssh.enable = true;
+  nixpkgs.config.extra-platforms = [ "aarch64-linux" ];
+  nixpkgs.config.system-features = [ "kvm" ];
+  nix = {
+    trustedUsers = [ "builder" ];
+    extraOptions = ''
+      extra-platforms = aarch64-linux
+      system-features = kvm
+      use-sqlite-wal = false
+    '';
+  };
+
+  users.users.builder = {
+    createHome = true;
+    isNormalUser = true;
+  };
+  virtualisation.qemu.networkingOptions = [
+    # We need to re-define our usermode network driver
+    # since we are overriding the default value.
+    "-net nic,netdev=user.0,model=virtio"
+    # Than we can use qemu's hostfwd option to forward ports.
+    "-netdev user,id=user.0,hostfwd=tcp::2222-:22"
+  ];
 }
```

Having done that you may run `make test` to spawn a nixos via qemu.

I created a host entry in my `.ssh/config`:

```
Host nixos-shell
  Port 2222
  IdentitiesOnly yes
  User root
  HostName 127.0.0.1
  IdentityFile ~/.ssh/id_rsa
```

Also I copied my passwordless private key file as well as the *.pub file part of the key to `~/.ssh/id_rsa[,.pub]`. You may just generate a key pair and use that as long as it is passwordless.

So now in another shell window I can test `ssh nixos-shell nix-store --version` or even after editing ~/.config/nix/nix.conf accordingly

```
builders-use-substitutes = true
builders = ssh://nixos-shell
```

do

```
nix-build --expr '(with import <nixpkgs> { }; runCommand "foo" {} "env > $out")' --option system aarch64-linux --option sandbox false --extra-platforms aarch64-linux
#cat /nix/store/<generated-store-path>-foo
```