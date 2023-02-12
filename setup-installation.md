# 安装

- [最低系统要求](#system-requirements)
- [基于Web的安装](#web-based-installation)
    - [安装故障排除](#troubleshoot-installation)
- [命令行安装](#command-line-installation)
- [安装后步骤](#post-install-steps)
    - [删除安装文件](#delete-install-files)
    - [审阅配置](#config-review)
    - [设置调度程序](#crontab-setup)
    - [设置队列工作者](#queue-setup)

<div class="og-description">
    Winter CMS的安装方式的文档。
</div>

您可以使用[基于Web的安装程序](#web-based-installation)或[Composer安装说明](../help/using-composer)中的两种方式安装Winter。在继续之前，您应该检查服务器是否满足最低系统要求。

<a name="system-requirements"></a>
## 最低系统要求1

Winter CMS对于Web托管有一些服务器要求：

- PHP版本7.2或更高
- PDO PHP扩展（以及您要连接到的数据库的相关驱动程序）
- cURL PHP扩展
- OpenSSL PHP扩展
- Mbstring PHP扩展
- ZipArchive PHP扩展
- GD PHP扩展
- SimpleXML PHP扩展

某些OS分发可能需要您手动安装一些所需的PHP扩展。

在使用Ubuntu时，可以运行以下命令来安装所有必需的扩展：

```bash
sudo apt-get update &&
sudo apt-get install php php-ctype php-curl php-xml php-fileinfo php-gd php-json php-mbstring php-mysql php-sqlite3 php-zip
```

在使用 SQL Server 数据库引擎时，您需要安装 [group concatenation](https://groupconcat.codeplex.com/) 用户定义的聚合。

<a name="web-based-installation"></a>
## 基于Web的安装

[Web Installer](https://github.com/wintercms/web-installer)是推荐给**非技术用户**安装 Winter 的方法。它比命令行安装更简单，不需要任何特殊技能。

> **注意：**如果您是开发者，我们建议您 [通过 Composer 安装](../help/using-composer)

1. 在将要托管 Winter CMS 安装的网络服务器上准备一个空目录。它可以是主域、子域或子文件夹。
2. 在该文件夹中从 Winter CMS Web Installer 的最新版本中 [下载 "install.zip" 文件](https://github.com/wintercms/web-installer/releases/latest)。
3. 解压下载的 ZIP 文件。
4. 将提取的所有文件和文件夹授予写权限。
5. 在网页浏览器中导航到指向该文件夹的 URL，并在 URL 结尾处包括 `/install.html`。
6. 按照安装程序给出的说明进行操作。

![Winter CMS Installer](https://github.com/wintercms/docs/blob/main/images/web-installer.jpg?raw=true) {.img-responsive .frame}

<a name="troubleshoot-installation"></a>
### 解决网页安装问题

1. **无法连接到 Winter Marketplace API**：如果您的服务器有一个防火墙阻止到端口 443 的请求，您需要允许此端口的请求和响应。请联系您的系统管理员以允许访问此端口。

2. **安装程序在“确定依赖关系”或“安装依赖关系”步骤失败**：在底层，网页安装程序使用 Composer 处理和安装运行 Winter CMS 所必需的依赖项 - 请注意，您 *不* 需要安装作为 CLI 工具的 Composer。此过程可能需要更多的内存完成 - 如果您的环境限制应用程序的内存使用，请考虑临时允许安装程序使用多达 1.5GB 的内存，然后在安装完成后减少它。安装程序将尝试自动执行此操作。

3. **安装程序未正确显示或功能**：网页安装程序基于现代前端框架建立，可能需要使用更新版本的浏览器。请考虑安装 Mozilla Firefox，Microsoft Edge 或 Google Chrome 并保持其最新。

4. **无法解析 JSON 响应**：根据您的互联网连接情况，下载所有源文件可能需要的时间超过了 [`max_execution_time` PHP 配置值

5. **无法确定 Winter CMS 的依赖关系，您的 composer.json 文件无效。**：使用共享主机的人报告了此错误，通常是由于 `disable_functions` PHP INI 设置中列出的 `proc_*` 方法或未启用的 PHP 的 `pcntl` 扩展引起的。不幸的是，目前没有任何可行的解决方法，但您可以在没有这些限制的本地开发环境上使用安装程序，然后在完成后即可将完整目录复制到共享主机上。我们正在积极寻找更长期的修复方法。

<a name="command-line-installation"></a>
## 命令行安装

如果您更喜欢命令行或想要使用Composer，可以在[使用Composer页面](../help/using-composer)上使用CLI安装过程。

<a name="post-install-steps"></a>
## 安装后的步骤

安装完成后，您可能需要进行一些设置。

<a name="delete-install-files"></a>
### 删除安装文件

如果您使用了[向导安装程序](#wizard-installation)，出于安全原因，您应该验证安装文件已被删除。 Winter安装程序尝试清理自己，但您应该始终验证它们已被成功删除：

```css
 ┣ 📂 install       <== 安装目录
 ┣ 📜 install.html  <== 安装脚本
```

<a name="config-review"></a>
### 审核配置

配置文件存储在应用程序的“config”目录中。虽然每个文件都包含每个设置的说明，但重要的是要审核[适用于您情况的常见配置选项](../setup/configuration)。

例如，在生产环境中，您可能希望启用[CSRF保护](../setup/configuration#csrf-protection)。而在开发环境中，您可能希望启用[最新的更新](../setup/configuration#edge-updates)。

虽然大多数配置都是可选的，但我们强烈建议在生产环境中禁用[调试模式](../setup/configuration#debug-mode)。您还可能希望使用[公共文件夹](../setup/configuration#public-folder)以增加安全性。

<a name="crontab-setup"></a>
### 设置计划任务调度程序

为了使定时任务正常运行，您应该在服务器上添加以下 Cron 条目。通常使用命令 `crontab -e` 编辑 crontab。

    * * * * * php /path/to/artisan schedule:run >> /dev/null 2>&1

请务必将 `/path/to/artisan` 替换为 Winter 根目录中 `artisan` 文件的绝对路径。此 cron 将每分钟调用命令调度程序，Winter 将评估任何计划任务并运行到期的任务。

> **注意**：如果您正在将其添加到 `/etc/cron.d`，则需要在 `* * * * *` 之后立即指定用户。



<a name="queue-setup"></a>
### 设置队列工作者

您可以选择设置一个外部队列来处理排队的作业。默认情况下，这些将由平台异步处理。这种行为可以通过在`config/queue.php`中设置`default`参数来更改。

如果您决定使用`database`队列驱动程序，最好在Crontab中为命令`php artisan queue:work --once`添加一个条目，以处理队列中的第一个可用作业。

