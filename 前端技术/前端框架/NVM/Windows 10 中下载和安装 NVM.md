# Windows 10 中下载和安装 NVM

[toc]

NVM 是 Node Version Manager（Node 版本管理工具）的缩写，是一个命令行工具，用于管理和切换到不同版本的 Node.js。

> **如果你已经安装了 Node.js，你需要卸载它，这样在使用不同版本的 Node 和从 NPM 注册表安装包时不会导致错误。**
>
> **如果你安装了 yarn，卸载它，安装 NVM 后重新安装它。你不想在安装和使用 NPM 注册表中的包时遇到奇怪的错误**。

> Windows中删除Node.js：
>
> 1.从卸载程序卸载程序和功能。
>
> 2.重新启动（或者您可能会从任务管理器中杀死所有与节点相关的进程）。
>
> 3.寻找这些文件夹并删除它们（及其内容）（如果还有）。根据您安装的版本，UAC设置和CPU架构，这些可能或可能不存在：
>
> - C:\Program Files (x86)\Nodejs
> - C:\Program Files\Nodejs
> - C:\Users\{User}\AppData\Roaming\npm（或%appdata%\npm）
> - C:\Users\{User}\AppData\Roaming\npm-cache（或%appdata%\npm-cache）
>
> 4.检查您的%PATH%环境变量以确保没有引用Nodejs或npm存在。
>
> 5.如果仍然没有卸载，请where node在命令提示符下键入，您将看到它所在的位置 - 删除（也可能是父目录）。
>
> 6.重新启动，很好的措施。

之后重新启动你的电脑，打开命令提示符或 PowerShell，然后运行 `node -v` 以确认 Node 已被卸载。

![ss1-2](Windows%2010%20%E4%B8%AD%E4%B8%8B%E8%BD%BD%E5%92%8C%E5%AE%89%E8%A3%85%20NVM.assets/ss1-2.png)

## 下载nvm

前往 [nvm-windows 仓库](https://github.com/coreybutler/nvm-windows#installation--upgrades)，然后单击立即下载

![ss2-2](Windows%2010%20%E4%B8%AD%E4%B8%8B%E8%BD%BD%E5%92%8C%E5%AE%89%E8%A3%85%20NVM.assets/202305051642872.png)

![image-5](Windows%2010%20%E4%B8%AD%E4%B8%8B%E8%BD%BD%E5%92%8C%E5%AE%89%E8%A3%85%20NVM.assets/202305051643322.png)

下载完成后安装,一路next(可自有调整nvm安装路径)

打开 PowerShell 或命令提示符并运行 `nvm -v` 以确认安装。

```bash
nvm -v
```

![ss6-2](Windows%2010%20%E4%B8%AD%E4%B8%8B%E8%BD%BD%E5%92%8C%E5%AE%89%E8%A3%85%20NVM.assets/202305051644065.png)

## NVM 安装不同版本的 Node.js 和 NPM

打开 PowerShell

要安装最新版本的 Node，请运行 `nvm install latest`。

```bash
nvm install latest
```

要安装特定版本的 Node，你需要先运行 `nvm list available`，以便查看可用的 Node 版本。

```bash
nvm list available
....
....
....
# 安装特定版本 nvm install node-version-number
nvm install 14.20.0
```

要使用特定版本的 Node，请运行：

```bash
# 使用最新版本
nvm use latest
# 使用长期支持版本
nvm use lts
# 使用你已安装的任何其他版本
nvm use 14.20.0
```

