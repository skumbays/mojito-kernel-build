# NoMount Metamodule CI Step

Paste this step into .github/workflows/main.yml **after** the
"Integrate NoMount v1.1.1 (kernel-side)" step and **before** the Clang download step.

```yaml
      - name: Build NoMount Metamodule Zip
        # Reuses the nomount-src/ clone from the kernel-integration step above.
        # Produces NoMount-v1.1.1.zip — a flashable KernelSU-Next module that
        # activates the VFS path-redirection hooks already compiled into the kernel.
        run: |
          set -e

          echo "=== Install zig (arm64 cross-compilation, same flags as upstream CI) ==="
          ZIG_VER=0.14.0
          wget -q "https://ziglang.org/download/${ZIG_VER}/zig-linux-x86_64-${ZIG_VER}.tar.xz" \
            -O /tmp/zig.tar.xz
          tar -C /tmp -xf /tmp/zig.tar.xz
          ZIG="/tmp/zig-linux-x86_64-${ZIG_VER}/zig"
          "$ZIG" version

          echo "=== Compile nm binary for arm64 (mojito) ==="
          cd nomount-src/userspace/src
          "$ZIG" cc -target aarch64-linux -Oz -static -nostdlib -ffreestanding \
            -fno-unwind-tables -fno-ident -Wno-invalid-noreturn \
            -Wl,--entry=_start nm.c -o nm-arm64

          echo "=== Compile nm binary for arm (32-bit compat) ==="
          "$ZIG" cc -target arm-linux -Oz -static -nostdlib -ffreestanding \
            -fno-unwind-tables -fno-ident -Wno-invalid-noreturn \
            -Wl,--entry=_start nm.c -o nm-arm

          wc -c nm-arm64 nm-arm
          file nm-arm64 nm-arm

          echo "=== Place binaries in module/bin/ ==="
          cd "$GITHUB_WORKSPACE"
          mkdir -p nomount-src/module/bin
          install -m 0755 nomount-src/userspace/src/nm-arm64 nomount-src/module/bin/nm-arm64
          install -m 0755 nomount-src/userspace/src/nm-arm   nomount-src/module/bin/nm-arm
          ls -lh nomount-src/module/bin/

          echo "=== Update versionCode in module.prop ==="
          sed -i "s/^versionCode=.*/versionCode=${{ github.run_number }}/" \
            nomount-src/module/module.prop
          cat nomount-src/module/module.prop

          echo "=== Package flashable module zip ==="
          cd nomount-src/module
          zip -r9 "$GITHUB_WORKSPACE/NoMount-v1.1.1.zip" . \
            --exclude '*.git*' --exclude '*.DS_Store'
          ls -lh "$GITHUB_WORKSPACE/NoMount-v1.1.1.zip"
          echo "Metamodule zip ready."
```

And add this **after** the existing "Upload Artifact" step:

```yaml
      - name: Upload NoMount Metamodule
        # Flash this zip via KernelSU-Next manager AFTER flashing the kernel.
        # Activates the VFS path-redirection hooks (CONFIG_NOMOUNT=y) in the kernel.
        uses: actions/upload-artifact@v4
        with:
          name: nomount-metamodule-v1.1.1-${{ github.run_number }}
          path: NoMount-v1.1.1.zip
          retention-days: 90
          if-no-files-found: error
```
