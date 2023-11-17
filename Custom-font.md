# Recommended way

The font needs to be a single TTF file. The recommended way to set it is via the configuration option: https://nix-community.github.io/nix-on-droid/nix-on-droid-options.html#opt-terminal.font

# Historical notes preserved for the reference

(taken from https://github.com/t184256/nix-on-droid/issues/120#issuecomment-867219785)

TL;DR: you can set one font, drop it into `~/.termux/font.ttf`

Setting fonts from Nix is hard to do right. I've started twice or thrice, never finished.

Font facts:

* in normal nix-on-droid usage (just running the terminal), none of the linux/home-manager font configuration facilities matter, fontconfig is ignored
* the only thing that renders fonts is my fork of the Termux app
* the fork has the styling addon baked in, so it looks for a `~/.termux/font.ttf`
* so, good news are, you can change the font
* bad news are, it has to be a regular file and not a symlink to `/nix/store` because, as far as the terminal app is concerned, there's no `/nix/store`. Auto-copying it on activation sounds hacky. Patching the app to do the sort of broken symlink resolution we already do for login script is definitely an option, but I'm basically allergic to Android development. PRs are welcome though.
* another bad thing is, Termux terminal emulator isn't exactly breathtaking with its font-rendering capabilities, even doing obliques instead of italics (my poor font)
* finally, there's no family detection, fallback chain or anything, none of this stuff. one ttf to rule them all

So, if you want a font with extra symbols, you'd have to find a TTF that has all the symbols you want in one file and then put it at `~/.termux/font.ttf`.