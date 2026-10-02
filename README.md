# zcbot 原生渲染扩展编译仓库

专门负责 `zcbot_render`（pyo3 + fontdue）多平台原生库的交叉编译，
产物（`zcbot_render.pyd` / `zcbot_render.so`）供主仓库
`core_plugins/image_renderer/native/bin/<平台>/` 使用。

## 目录

- `src/` `Cargo.toml` `Cargo.lock` — 与主仓库 `core_plugins/image_renderer/native/` 保持同步的源码
- `.github/workflows/build.yml` — 编译矩阵

## 平台矩阵

| 产物目录 | target | 构建方式 |
|---|---|---|
| win64 | x86_64-pc-windows-msvc | 原生 cargo |
| win32 | i686-pc-windows-msvc | 原生 cargo |
| linux-x86_64 | x86_64-unknown-linux-gnu | cargo-zigbuild |
| linux-aarch64 | aarch64-unknown-linux-gnu | cargo-zigbuild |
| linux-armv7 | armv7-unknown-linux-gnueabihf | cargo-zigbuild |
| linux-i686 | i686-unknown-linux-gnu | cargo-zigbuild |
| linux-loongarch64 | loongarch64-unknown-linux-gnu | cargo-zigbuild |

每次 push 到 main 会构建并上传 artifacts；推 `v*` tag 时额外把各平台二进制
发布到 GitHub Release（打包为 `zcbot_render-native-libs.zip`）。

## 同步流程

源码改动先在主仓库修改，然后复制本仓库：

```bash
cp -r <主仓库>/core_plugins/image_renderer/native/{src,Cargo.toml,Cargo.lock} .
git commit -am "sync from main" && git push
```

构建成功后从 Release/artifacts 取回二进制放回主仓库 `bin/`。
