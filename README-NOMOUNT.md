# NoMount v1.1.1 – Flash Guide for Mojito (Redmi Note 10)

## What is NoMount?

NoMount replaces OverlayFS/MagicMount with direct VFS path redirection built into the kernel.
This kernel already has `CONFIG_NOMOUNT=y` compiled in. To activate the hooks you must also
flash the NoMount metamodule via KernelSU-Next manager.

---

## Step 1 – Get the kernel zip

Download the latest **mojito-KSUNext-SUSFS-\*.zip** artifact from the
[Actions tab](https://github.com/skumbays/mojito-kernel-build/actions) of this repo.

## Step 2 – Get the NoMount metamodule zip

The metamodule zip is produced by the **Build NoMount Metamodule Zip** step in the same CI run.
Download the **nomount-metamodule-v1.1.1-\<run\>.zip** artifact from the same Actions run.

If you prefer to build it yourself, see [BUILD-NOMOUNT-MODULE.md](BUILD-NOMOUNT-MODULE.md).

## Step 3 – Flash order (do NOT skip order)

Flash via TWRP or KernelSU-Next manager:

```
1. Flash mojito-KSUNext-SUSFS-<date>.zip          (kernel + KSU-Next + SUSFS + NoMount kernel)
2. Reboot to system
3. Open KernelSU-Next manager → Modules → ⊕
4. Select NoMount-v1.1.1.zip and flash
5. Reboot
```

## Step 4 – Verify NoMount is active

After reboot, open KernelSU-Next manager → Modules.
You should see **NoMount** listed with status **Enabled**.

From an adb shell (or a terminal emulator with root):

```sh
# Run the nm CLI (inside the module's bin dir)
/data/adb/modules/nomount/bin/nm ver
# Expected output: NoMount kernel version (e.g. "1.1.1")

# List active rules (empty on fresh install)
/data/adb/modules/nomount/bin/nm ls

# Test: redirect a file
/data/adb/modules/nomount/bin/nm add /vendor/etc/test.conf /data/local/tmp/my.conf
/data/adb/modules/nomount/bin/nm ls          # rule should appear
/data/adb/modules/nomount/bin/nm del /vendor/etc/test.conf    # clean up
```

If `nm ver` fails with **exit code 3**, the kernel Netlink socket is not responding.
This means the kernel was not compiled with `CONFIG_NOMOUNT=y`. Re-flash the kernel.

## Troubleshooting

| Symptom | Cause | Fix |
|---|---|---|
| Module disabled immediately after flash | Netlink missing → kernel not NoMount-patched | Flash correct kernel first |
| Bootloop → module auto-disabled | metamount.sh detected a crash on previous boot | Safe; reflash or remove module |
| `nm ver` → exit 3 | CONFIG_NOMOUNT=n in running kernel | Ensure you flashed THIS kernel |
| Module not visible in manager | Flash did not complete cleanly | Re-flash zip |

---

_This guide covers Xiaomi Redmi Note 10 (mojito/sunny), Android 12–13._
