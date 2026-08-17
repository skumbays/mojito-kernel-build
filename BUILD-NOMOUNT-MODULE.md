# Building the NoMount Metamodule Zip Locally

If the CI artifact is unavailable, you can build the zip yourself.

## Requirements

- [zig](https://ziglang.org/download/) 0.14.0+ (for cross-compilation)
- zip
- git

## Steps

```sh
# 1. Clone NoMount at v1.1.1
git clone --depth=1 -b v1.1.1 https://github.com/maxsteeel/nomount nomount-src
cd nomount-src

# 2. Compile nm binary for arm64 (mojito)
cd userspace/src
zig cc -target aarch64-linux -Oz -static -nostdlib -ffreestanding \
  -fno-unwind-tables -fno-ident -Wno-invalid-noreturn \
  -Wl,--entry=_start nm.c -o nm-arm64

# 3. Compile nm binary for arm (32-bit compat)
zig cc -target arm-linux -Oz -static -nostdlib -ffreestanding \
  -fno-unwind-tables -fno-ident -Wno-invalid-noreturn \
  -Wl,--entry=_start nm.c -o nm-arm

# 4. Place binaries
cd ../..
mkdir -p module/bin
install -m 0755 userspace/src/nm-arm64 module/bin/nm-arm64
install -m 0755 userspace/src/nm-arm   module/bin/nm-arm

# 5. Package zip
cd module
zip -r9 ../NoMount-v1.1.1.zip . --exclude '*.git*' --exclude '*.DS_Store'
ls -lh ../NoMount-v1.1.1.zip
```

The resulting **NoMount-v1.1.1.zip** is a valid Magisk/KernelSU module zip.
Flash it after the kernel via KernelSU-Next manager or TWRP.
```

## CI reference

The [mojito-kernel-build main.yml](.github/workflows/main.yml) also runs these steps automatically
as the **"Build NoMount Metamodule Zip"** step (zig is downloaded by CI). The artifact
**nomount-metamodule-v1.1.1-\<run\>** is uploaded for 90 days per run.
```
