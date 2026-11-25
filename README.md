# ponwrt-CI

参考 [VIKINGYFY/OpenWRT-CI](https://github.com/VIKINGYFY/OpenWRT-CI) 的云编译流程，
为 [pbs05/ponwrt](https://github.com/pbs05/ponwrt) 定制的 GitHub Actions 编译项目。

- 编译源码：`https://github.com/pbs05/ponwrt.git`（分支 `master`）
- 编译配置：`Config/5585f-cu.txt`
- 目标设备：`airoha / an7581`，`fiberhome_hg5585f-cu`

## 目录结构

```text
ponwrt-CI/
├── Config/
│   └── 5585f-cu.txt          # 完整 .config（不叠加 GENERAL.txt）
├── Scripts/
│   └── Settings.sh           # 固件默认值调整（主机名、IP、WIFI、语言等）
├── .github/
│   ├── scripts/              # 构建环境准备与报告收集脚本
│   └── workflows/
│       ├── PONWRT-5585F.yml  # 本项目的编译入口
│       ├── WRT-CORE.yml      # 公用编译核心
│       ├── Auto-Clean.yml    # 手动清理旧 Release / Workflow
│       └── Cache-Clean.yml   # 手动清理 Actions 缓存
├── .gitattributes
└── LICENSE
```

## 编译流程

`PONWRT-5585F` 调用公用核心 `WRT-CORE`，按以下顺序执行：

1. 安装编译依赖、准备磁盘空间
2. 克隆 `pbs05/ponwrt`
3. 恢复 `staging_dir`、`toolchain`、`ccache` 缓存
4. 更新并安装 feeds
5. 写入 `Config/5585f-cu.txt`，执行 `Settings.sh` 调整默认值
6. 准备 Go/LLVM/Rust 主机工具链
7. 下载源码、并行编译（失败自动串行重试）
8. 打包固件并发布 GitHub Release

## 触发方式

仅支持手动触发：

- **编译固件**：在 Actions 页面选择 `PONWRT-5585F` → `Run workflow`。
- **清理旧 Release / Workflow**：手动运行 `Auto-Clean`。
- **清理 Actions 缓存**：手动运行 `Cache-Clean`。

手动编译参数：

| 参数 | 说明 |
| --- | --- |
| `PACKAGE` | 临时追加插件配置，多个配置用换行分隔 |
| `TEST` | 仅生成配置文件并发布，不编译固件 |

## 使用说明

1. 将本目录推送到 GitHub 仓库的默认分支。
2. 仓库 `Settings → Actions → General` 中允许 Actions 读写仓库内容
   （Release 与缓存清理需要 `write-all` 权限）。
3. 在 Actions 页面手动运行 `PONWRT-5585F` 开始编译。

编译产物会以 `5585f-cu-pbs05-master-YY.MM.DD-HH.MM.SS` 为 Tag
发布在 Release 中，固件文件名形如
`...-pbs05-master-none-YY.MM.DD-HH.MM.SS.bin`，
同时附带本次实际使用的 `.config`。

## 更新配置

在本地 ponwrt 源码目录中执行 `make menuconfig`，保存后把生成的 `.config`
覆盖到 `Config/5585f-cu.txt` 即可：

```bash
cp /path/to/ponwrt/.config Config/5585f-cu.txt
```

本项目使用完整配置，不再叠加 `Config/GENERAL.txt`；
如需追加公共配置，可自行创建 `Config/GENERAL.txt`，核心会自动识别。
