# chenyongdong

## Cursor Cloud specific instructions

### Unity environment

This VM has the Unity toolchain installed for game development:

- **Unity Hub** `3.18.3` — installed via the official Unity apt repo; binary at `/usr/bin/unityhub`.
- **Unity Editor** `6000.0.77f1` (Unity 6 LTS) — installed at `~/Unity/Hub/Editor/6000.0.77f1/Editor/Unity`.

Non-obvious notes for running Unity headlessly in this VM:

- There is no display server, so wrap GUI/Editor invocations with `xvfb-run -a` (e.g. `xvfb-run -a unityhub --headless editors --installed`).
- `Failed to connect to the bus` / dbus errors printed by Unity Hub are harmless in this headless VM and can be ignored.
- Print the editor version (no license needed): `~/Unity/Hub/Editor/6000.0.77f1/Editor/Unity -version`.
- List installed editors: `xvfb-run -a unityhub --headless editors --installed`.
- List available releases to install: `xvfb-run -a unityhub --headless editors --releases`.
- Install another editor version: `xvfb-run -a unityhub --headless install --version <version>`.

#### Licensing (required before creating/building projects)

The Editor launches and initializes fine, but **creating, opening, or building a project requires an activated Unity license** (even the free Personal license needs a Unity ID login). Without one, batch commands fail with `No valid Unity Editor license found. Please activate your license.` (exit code 198).

To activate the free Personal license non-interactively, provide Unity ID credentials as secrets (`UNITY_EMAIL`, `UNITY_PASSWORD`; `UNITY_SERIAL` only for Pro/Plus) and run:

```
xvfb-run -a ~/Unity/Hub/Editor/6000.0.77f1/Editor/Unity \
  -batchmode -nographics -quit -logFile /dev/stdout \
  -username "$UNITY_EMAIL" -password "$UNITY_PASSWORD"
```

After activation, a typical create-and-build smoke test:

```
xvfb-run -a ~/Unity/Hub/Editor/6000.0.77f1/Editor/Unity \
  -batchmode -nographics -quit -logFile /dev/stdout \
  -createProject /tmp/HelloUnity
```

### Feishu (飞书) desktop client

The ByteDance Feishu Linux desktop client is installed:

- **Package** `bytedance-feishu-stable` `7.66.11-0` (installed from the official `.deb`).
- **Binaries**: `/usr/bin/bytedance-feishu` (and `/opt/bytedance/feishu/bytedance-feishu`).

Non-obvious notes:

- Feishu is an Electron GUI app. There is no display server in this VM, so launch it under a virtual display: `xvfb-run -a /opt/bytedance/feishu/bytedance-feishu --no-sandbox --disable-gpu &`. The `--no-sandbox` flag is required inside the container; `--disable-gpu` avoids GPU init noise.
- For actual interactive use (screenshots), launch it on the computer-use desktop with the same flags.
- The app opens to a **QR-code login screen**; completing login (and any messaging "hello world") requires a real Feishu account (scan QR with the mobile app, or phone/email login). No credentials are provisioned in this VM.
- Get the latest Linux `.deb` download link dynamically (links are signed/expiring): `curl -s -A 'Mozilla/5.0' "https://www.feishu.cn/api/package_info?platform=10"` and read `.data.download_link` (platform `10` = x86_64, `12` = arm64). Feishu does not provide an apt repo, so it is not auto-upgraded.
