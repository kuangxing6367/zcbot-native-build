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

## 32 位（i386 / i686 / armv7）适配说明

原生扩展对 32 位架构做了显式加固，避免「抠门架构」上因 `usize`/`u32` 仅 4 字节而触发的静默回绕：

- **像素缓冲分配**：先用 `u64` 计算 `宽 × 高 × 4` 总字节数，超出当前平台地址空间直接报错，
  不再依赖 `(u32) 乘法 as usize` 的回绕（否则会分配出尺寸错误的缓冲、造成越界写入）。
- **像素线性索引**：统一改用 `usize` 运算（`y*width + x`）再乘 4，32/64 位行为一致。
- **圆 / 椭圆 / 圆角 / 线条的距离判定**：平方距离改用 `i64`，修正 `r*r` 在 `r>46340` 时的
  `i32` 回绕（该问题在 32 位与 64 位下都存在，32 位内存受限环境更常见大半径场景）。

win32 / linux-i686 由 CI 交叉编译产出，与主仓库 `bin/win32`、`bin/linux-i686` 对应；
运行期由 `image_renderer` 插件按 `platform.machine()` / `sys.maxsize` 自动选路。
