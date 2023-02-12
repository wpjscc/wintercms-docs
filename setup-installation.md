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
## 最低系统要求

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

When using the SQL Server database engine, you will need to install the [group concatenation](https://groupconcat.codeplex.com/) user-defined aggregate.

<a name="web-based-installation"></a>
## Web-based installation

The [Web Installer](https://github.com/wintercms/web-installer) is the recommended way to install Winter for **non-technical users**. It is simpler than the command-line installation and doesn't require any special skills.

> **NOTE:** If you are a developer, we recommend that you [install via Composer instead](../help/using-composer)

1. Prepare an empty directory on the web server that will host your Winter CMS installation. It can be a main domain, sub-domain or subfolder.
2. [Download the "install.zip" file](https://github.com/wintercms/web-installer/releases/latest) from the latest release of the Winter CMS Web Installer into this folder.
3. Unzip the downloaded ZIP file.
4. Grant write permissions to all files and folders that were extracted.
5. In your web browser, navigate to the URL pointing to that folder, and include `/install.html` at the end of the URL.
6. Follow the instructions given in the installer.

![Winter CMS Installer](https://github.com/wintercms/docs/blob/main/images/web-installer.jpg?raw=true) {.img-responsive .frame}

<a name="troubleshoot-installation"></a>
### Troubleshooting a web-based installation

1. **Unable to connect to the Winter Marketplace API**: If your server has a firewall blocking requests to port 443, you will need to allow requests and responses for this port. Contact your system administrator to allow access to this port.

2. **Installer fails on the "Determining dependencies" or "Installing dependencies" step**: Under the hood, the web installer uses Composer to process and install the dependencies necessary to run Winter CMS - note, you *do not* need Composer installed as a CLI tool for this to work. This process may require a larger amount of memory to complete - if your environment restricts memory usage for your applications, consider allowing up to 1.5GB of memory temporarily for the installer, then reduce it after the installation is complete. The installer will try to do this automatically.

3. **Installer does not display or function correctly**: The web installer has been built on modern frontend frameworks, and may require the use of a more recent browser version. Consider installing Mozilla Firefox, Microsoft Edge or Google Chrome and keeping it up-to-date.

4. **Unable to parse JSON response**: Depending on your internet connection, downloading all the source files may take more time than the [`max_execution_time` PHP configuration value](https://www.php.net/manual/en/info.configuration.php#ini.max-execution-time) allows; causing the download to end with an incomplete file base. Modify the PHP configuration to increase this value and try again.

5. **Unable to determine dependencies for Winter CMS. Your composer.json file is invalid.**: This error has been reported by people using shared hosting, and is usually the result of either the `proc_*` methods being listed in the `disable_functions` PHP INI setting, or the `pcntl` extension for PHP not being enabled. Unfortunately, at this stage, there is no workaround, but you may be able to use the Installer on a local development environment that does not have these limitations, and then simply copy the full directory to your shared hosting once completed. We are actively looking into more long-term fix for this behaviour.

<a name="command-line-installation"></a>
## Command-line installation

If you feel more comfortable with a command-line or want to use Composer, there is a CLI install process on the [Using Composer page](../help/using-composer)

<a name="post-install-steps"></a>
## Post-installation steps

There are some things you may need to set up after the installation is complete.

<a name="delete-install-files"></a>
### Delete installation files

If you have used the [Wizard installer](#wizard-installation), for security reasons you should verify the installation files have been deleted. The Winter installer attempts to cleanup after itself, but you should always verify that they have been successfullly removed:

```css
 ┣ 📂 install       <== Installation directory
 ┣ 📜 install.html  <== Installation script
```

<a name="config-review"></a>
### Review configuration

Configuration files are stored in the `config` directory of the application. While each file contains descriptions for each setting, it is important to review the [common configuration options](../setup/configuration) available for your circumstances.

For example, in production environments you may want to enable [CSRF protection](../setup/configuration#csrf-protection). While in development environments, you may want to enable [bleeding edge updates](../setup/configuration#edge-updates).

While most configuration is optional, we strongly recommend disabling [debug mode](../setup/configuration#debug-mode) for production environments. You may also want to use a [public folder](../setup/configuration#public-folder) for additional security.

<a name="crontab-setup"></a>
### Setting up the scheduler

For scheduled tasks to operate correctly, you should add the following Cron entry to your server. Editing the crontab is commonly performed with the command `crontab -e`.

    * * * * * php /path/to/artisan schedule:run >> /dev/null 2>&1

Be sure to replace `/path/to/artisan` with the absolute path to the `artisan` file in the root directory of Winter. This cron will call the command scheduler every minute, in which Winter will evaluate any scheduled tasks and run the tasks that are due.

> **NOTE**: If you are adding this to `/etc/cron.d`, you'll need to specify a user immediately after `* * * * *`.

<a name="queue-setup"></a>
### Setting up queue workers

You may optionally set up an external queue for processing queued jobs. By default, these will be handled asynchronously by the platform. This behavior can be changed by setting the `default` parameter in the `config/queue.php`.

If you decide to use the `database` queue driver, it is a good idea to add a Crontab entry for the command `php artisan queue:work --once` to process the first available job in the queue.
