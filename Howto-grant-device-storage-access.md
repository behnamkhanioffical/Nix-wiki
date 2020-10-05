In the app settings from Android grant storage access permission.
Then you should see:
`/sdcard` and
`/data/data/com.termux.nix/files/home`.

On some devices / android setups [the location of](https://github.com/t184256/nix-on-droid/issues/89) the writeable storage location might differ, i. e. I had

```
/mnt/sdcard
```

for my sdcard and

```
<internal storage>
```

when accessing the files via Android.