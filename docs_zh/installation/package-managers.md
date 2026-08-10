---
title: 通过包管理器安装 (社区维护)
description: "通过 Homebrew、Nix 或 Arch AUR 安装社区维护的 Ferron 软件包，并介绍已安装命令与服务的说明。"
---

Ferron 有几个社区维护的软件包，可以通过各种包管理器进行安装。以下是通过包管理器安装 Ferron 的说明。

## Homebrew (macOS 和 GNU/Linux)

要通过 Homebrew 安装 Ferron，您可以运行以下命令：

```bash
brew install ferron
```

此命令安装 `ferron` 命令，该命令运行一个 Web 服务器。

您可以在 [Homebrew Formulae](https://formulae.brew.sh/formula/ferron) 上查看此 Ferron 软件包。

## Nix 不稳定版 (GNU/Linux)

要通过 Nix (不稳定频道) 安装 Ferron，您可以运行以下命令：

```bash
nix-shell -p ferron
```

此命令安装 `ferron` 命令（运行 Web 服务器）和 `ferron-passwd` 命令（Ferron Web 服务器的密码生成实用程序）。

您可以在 [Nixpkgs](https://search.nixos.org/packages?channel=unstable&show=ferron&from=0&size=50&sort=relevance&type=packages&query=ferron) 上查看此 Ferron 软件包。

## `yay` (来自 AUR；Arch Linux)

要从 AUR (Arch User Repository) 安装 Ferron，您可以运行以下命令：

```bash
yay -S ferronweb
```

此命令安装 `ferron` 命令（运行 Web 服务器）和 `ferron-passwd` 命令（Ferron Web 服务器的密码生成实用程序）。此命令还安装一个 `systemd` 服务，可以使用 `sudo systemctl start ferron` 启动。

对于所有 `yay` 提示，按“Enter”键使用默认值。

如果您尚未安装 `yay` 命令，可以使用以下命令安装它：

```bash
sudo pacman -Sy # 更新软件包数据库
sudo pacman -S pacman # 更新 `pacman` 以修复潜在的依赖项错误
sudo pacman -S git base-devel # 安装 `yay` 所需的软件包
git clone https://aur.archlinux.org/yay-bin.git
cd yay-bin
makepkg -Cf && makepkg -si # 清理构建 `yay` 软件包，然后安装该软件包及其依赖项
```

您可以在 [AUR](https://aur.archlinux.org/packages/ferronweb) 上查看此 Ferron 软件包。
