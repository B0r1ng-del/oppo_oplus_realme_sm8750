# 备份与试验说明

## 1. 原文件备份

| 备份文件 | 对应原文件 | 说明 |
|---|---|---|
| `backup/fastbuild_6.6.118_mtk.yml.bak` | `.github/workflows/fastbuild_6.6.118_mtk.yml` | 原版天玑 6.6.118 工作流，逐字节备份 |

**原工作流没有被修改。** 本次 NoMount 试验使用的是独立的新文件：

`.github/workflows/fastbuild_6.6.118_mtk_nomount_test.yml`

需要恢复时：

```bash
cp backup/fastbuild_6.6.118_mtk.yml.bak .github/workflows/fastbuild_6.6.118_mtk.yml
```

## 2. NoMount 试验集成

- NoMount 版本：`v2.2.0`（驱动协议 v22，必须与设备上安装的 NoMount v2.2.0 模块配套）
- 集成脚本：`.github/scripts/setup_nomount.sh`
- 集成方式：built-in
  - `fs/nomount/` 放入 NoMount 内核源码
  - `fs/Makefile` 增加 `obj-$(CONFIG_NOMOUNT) += nomount/`
  - `fs/Kconfig` 增加 `source "fs/nomount/Kconfig"`
  - `arch/arm64/configs/gki_defconfig` 增加 `CONFIG_NOMOUNT=y`

## 3. 编译期备份（脚本自动创建）

`setup_nomount.sh` 在第一次运行时会创建，并且**不会覆盖已有备份**：

- `<kernel>/fs/Makefile.nomount-bak`
- `<kernel>/fs/Kconfig.nomount-bak`
- `<kernel>/arch/arm64/configs/gki_defconfig.nomount-bak`

CI 的 `kernel_workspace` 是临时目录；本地编译时可用这些 `.nomount-bak` 手动还原。

## 4. 以后更新 NoMount 时怎么保持兼容

试验工作流新增了 `nomount_ref` 输入（默认 `v2.2.0`）：

- 升级 NoMount：直接在 Actions 里把 `nomount_ref` 改成新 tag（例如 `v2.3.0`）再运行；
- 脚本本身是幂等的：会替换 `fs/nomount/` 源码，但不会重复追加 Makefile/Kconfig/defconfig；
- 设备上的 NoMount 模块必须与内核驱动协议一致（v2.2.0 对应协议 v22）。

> 如果上游改了 `kernel/src` 目录结构或 Kconfig/Makefile 的接入方式，脚本会在校验阶段明确报错，不会静默产出一个没有 NoMount 的内核。
