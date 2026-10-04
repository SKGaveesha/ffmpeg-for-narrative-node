# Our own ffmpeg, for Windows and Mac

Navigator's installers carry ffmpeg, ffprobe and ffplay, fetched at build time from a public GitHub repository that we control (and the helper downloads them from it on first run if an install has no copy): `SKGaveesha/narrative-node-ffmpeg-builds`. Both builds are LGPL-only, so a paid product can use them. Owning the hosting means nobody else can rename or retire the file a customer's helper is looking for.

| | Workflow | How | Release the helper reads | File |
|:--|:--|:--|:--|:--|
| Windows x64 | `build-ffmpeg-windows.yml` | BtbN's open-source build scripts, pinned to one commit | `win-ffmpeg-8.1` | `ffmpeg-n8.1-win64-lgpl.zip` |
| macOS arm64, x64 | `build-ffmpeg-mac.yml` | compiled directly on Mac runners | `mac-ffmpeg-8.1` | `ffmpeg-n8.1-macos-<arch>-lgpl.zip` |

(`eugeneware/ffmpeg-static` was checked for Mac and not used: its binaries are built with `--enable-gpl --enable-version3 --enable-nonfree`, which a paid product must not hand to customers, and it has no ffplay, which the audio player needs.)

## One-time setup

1. On GitHub create a **public** repository named `narrative-node-ffmpeg-builds` under the `SKGaveesha` account. It must be public: customers' helpers read its releases without logging in, and public repositories get free Actions minutes.
2. In it add both files under `.github/workflows/`, with the contents of the two `.yml` files in this folder.
3. In that repository, **Actions**: run **Build FFmpeg for Windows** (an hour or more, it compiles everything) and **Build FFmpeg for macOS** (10–15 minutes).
4. When each finishes, **Releases** has `win-ffmpeg-8.1` and `mac-ffmpeg-8.1`, each with its zip(s) and the FFmpeg source tarball.

If you use another repository name, change `FFMPEG_BUILDS_REPO` in `helper/src/core/ffmpeg/install.ts`.

## Updating

Run a workflow again with a newer FFmpeg version. The release tag stays the same, so helpers already in customers' hands keep working. For the Windows build, also read what changed in BtbN's repository before moving `BTBN_COMMIT` in the workflow. When you move to FFmpeg 8.2, change the tags and the `n8.1` in the asset names in `install.ts` too.

## Why LGPL, and what we owe

Neither build uses `--enable-gpl` or `--enable-nonfree`, and both workflows fail if the finished program says otherwise. LGPL asks us to let anyone who has the binary get the source and the build recipe: each release carries the FFmpeg source tarball and a `BUILD-INFO` file naming the exact scripts and configure line. The Navigator installers carry the unmodified binaries with `LICENSE.txt` and a `SOURCE.txt` that points back to these releases, which is how the source offer reaches customers. Anyone can replace the binaries by dropping their own build into the `ffmpeg` folder beside the helper.

## Not tested on real machines

The Mac workflow compiles on Mac runners and the Windows one cross-compiles on Linux; neither has been run yet, so the first run may need a small fix. After it, on a clean Windows PC and on both an Apple Silicon and an Intel Mac, check that `ffmpeg -version` and `ffplay -version` run, that Navigator downloads the tools, plays audio, and that an export with a hardware encoder works.
