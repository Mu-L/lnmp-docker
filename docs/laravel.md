# Laravel 最佳实践

## 安装 Laravel

```bash
$ cd app

$ lnmp-laravel new laravel
```

具体请查看 [这里](command.md)

### Laravel 版本

上面的命令会安装 Laravel 最新的主线版本（8.x），如果你要安装特定版本可以加上 **版本号**

```bash
$ cd app

# $ lnmp-laravel new FOLDER VERSION

$ lnmp-laravel new laravel5.5 5.5
```

或者直接使用 `composer` 安装

```bash
$ cd app

$ lnmp-composer create-project laravel/laravel laravel5.5 "5.5.*"
```

## 设置 Laravel .env 文件

正确配置服务的 `HOST`，填写 `127.0.0.1` 将连接不到服务，具体原因不再赘述。

```bash
DB_HOST=mysql

REDIS_HOST=redis

MEMCACHED_HOST=memcached
```

## 安装 [laravel-ide-helper](https://github.com/barryvdh/laravel-ide-helper)

```bash
$ lnmp-composer require --dev barryvdh/laravel-ide-helper
```

```bash
$ lnmp-php artisan ide-helper:eloquent
$ lnmp-php artisan ide-helper:generate
$ lnmp-php artisan ide-helper:meta
$ lnmp-php artisan ide-helper:models
```

**.gitignore**

```bash
# .gitignore

.phpstorm.meta.php
_ide_helper.php

.php_cs.cache
```

## Windows 运行 Laravel 响应缓慢的问题

* Docker Desktop 上 Docker 运行在虚拟机，项目文件位于 Windows
* Docker Desktop(WSL2) 上 Docker 运行在 WSL2(仍然是虚拟机)，项目文件位于 Windows

以上两种情况项目文件位于 Windows 均为跨主机, 故存在文件性能问题。

这些情况下 `vendor` 目录可以使用数据卷（数据卷存在于虚拟机中）。[vsCode](https://code.visualstudio.com/docs/remote/containers-advanced#_use-a-targeted-named-volume) 的说明和笔者提出的方案原理大致相同。

由于文件性能问题，不推荐将项目文件放置于 Windows。

另一个方案是将项目文件夹放置于 WSL2，有以下两种方案：

第一种方案是使用 **vsCode remote WSL** WSL 远程开发

> 项目放置于 WSL2 也可以使用 PHPStorm，请参考 https://github.com/khs1994-docker/php-demo

**在 Docker 设置中启用 WSL2 集成**

`Resources` -> `WSL INTEGRATION`-> `Enable integration with additional distros:` -> `开启你所使用的 WSL2 （例如：Ubuntu）`

**安装 vsCode 扩展**

```bash
$ code --install-extension ms-vscode-remote.remote-wsl
```

**在 `.env` `.env.ps1` 中修改变量**

```bash
# .env
APP_ROOT=/app
```

```powershell
# .env.ps1
# 请将 Ubuntu 换成你使用的 WSL2 名称
$WSL2_DIST="Ubuntu"
```

**以上步骤仅需执行一次，后续开发从以下步骤开始**

**启动 LNMP**

```bash
$ ./lnmp-docker up
```

**打开 vsCode**

```powershell
# 打开 /app
# 适用于首次使用，暂无子目录，需要到 /app 中新建项目目录
$ lnmp-docker code

# 打开 /app 子目录（e.g. laravel）
# $ lnmp-docker code laravel
```

在 vsCode 中点击菜单栏 `查看` -> `终端`

在出现的终端中执行命令，本例以添加 `laravel/jetstream` 组件为例：(请提前将本项目的 `bin` 目录加入到 PATH)

```powershell
# 你也可以在 Windows 终端执行命令

$ cd \\wsl.localhost\ubuntu\app
```

```bash
# 安装 laravel 到 laravel 文件夹
# $ lnmp-laravel new laravel
# 文件可能因为权限问题无法编辑，自行更改权限
# $ wsl -d Ubuntu -u root -- chown -R 1000:1000 /app
# $ wsl -d Ubuntu -u root -- chmod -R 777 /app/laravel/storage/app
# $ wsl -d Ubuntu -u root -- chmod -R 777 /app/laravel/storage/logs
# $ wsl -d Ubuntu -u root -- chmod -R 777 /app/laravel/storage/framework
$ cd laravel

$ lnmp-composer require laravel/jetstream

$ lnmp-php artisan jetstream:install inertia

$ lnmp-npm install

$ lnmp-npm run dev

# 打开 http://127.0.0.1/register 查看页面
```

附录：查看本项目的 `bin` 目录在 WSL2 中的路径

在 Windows 终端中执行

```powershell
$ cd ~/lnmp/bin

$ wsl -d <WSL名称> -- wslpath "'$PWD'"
# 将结果追加到 WSL2 中的 PATH 环境变量中
```

第二种方案是使用 **vsCode remote container** 容器远程开发

**在 Docker 设置中启用 WSL2 集成**

`Resources` -> `WSL INTEGRATION`-> `Enable integration with additional distros:` -> `开启你所使用的 WSL2 （例如：Ubuntu）`

**安装 vsCode 扩展**

```bash
$ code --install-extension ms-vscode-remote.remote-containers
```

**在 `.env` `.env.ps1` 中修改变量**

```bash
# .env
APP_ROOT=/app

# 增加 vscode-remote-container-workspace 服务
LNMP_SERVICES="nginx mysql php7 redis vscode-remote-container-workspace"
```

```powershell
# .env.ps1
# 请将 Ubuntu 换成你使用的 WSL2 名称
$WSL2_DIST="ubuntu"
```

**以上步骤仅需执行一次，后续开发从以下步骤开始**

**启动 LNMP**

```bash
$ ./lnmp-docker up
```

**打开 vsCode**

左下角 `打开远程窗口` -> `Remote-Containers: Attach to Running Container...` -> 选择 `lnmp_vscode-remote-container-workspace_1` 容器 -> 出现新窗口 -> 左面选择打开文件夹 -> 输入 `/app`

在 vsCode 中点击菜单栏 `查看` -> `终端`

在出现的终端中执行命令，本例以添加 `laravel/jetstream` 组件为例：

```bash
# 安装 laravel 到 laravel 文件夹
# $ composer create-project --prefer-dist laravel/laravel laravel
# 文件可能因为权限问题无法编辑，自行更改权限
# $ wsl -d Ubuntu -u root -- chown -R 1000:1000 /app
# $ wsl -d Ubuntu -u root -- chmod -R 777 /app/laravel/storage/app
# $ wsl -d Ubuntu -u root -- chmod -R 777 /app/laravel/storage/logs
# $ wsl -d Ubuntu -u root -- chmod -R 777 /app/laravel/storage/framework
$ cd laravel

$ composer require laravel/jetstream

$ php artisan jetstream:install inertia
Vue scaffolding installed successfully.
Please run "npm install && npm run dev" to compile your fresh scaffolding.
Authentication scaffolding generated successfully.
```

在 Windows 终端中执行以下命令：

```powershell
$ lnmp-docker code-run -w /app/laravel npm install

$ lnmp-docker code-run -w /app/laravel npm run dev

# 打开 http://127.0.0.1/register 查看页面
```

## PHP 相关的 vsCode 扩展

* `xdebug.php-debug`
* 更多扩展请查看 https://github.com/khs1994-docker/lnmp/blob/master/.devcontainer/devcontainer.json **extensions** 项

## 运行 Laravel 队列(Queue)

* **方案1：** 使用 **宿主机** 的系统级的守护程序（systemd 等）来运行以下命令。具体请查看 [systemd](systemd.md)

```bash
$ lnmp-docker php7-cli php /app/laravel/artisan queue:work --tries=3
```

* **方案2：** 参考 `config/s6` 或 `config/supervisord` 在一个容器中同时运行多个服务 (两钟方案中均包含了 Laravel 队列等示例)。

## 运行 Laravel 调度器(Schedule)

参考上一节队列的说明。

```bash
$ lnmp-docker php7-cli php /app/laravel/artisan schedule:run
```

## 运行 Laravel horizon

* https://laravel.com/docs/8.x/horizon

```bash
$ lnmp-composer require laravel/horizon

$ lnmp-php artisan horizon:install
```

**配置**

`config/horizon.php` `environments` 数组必须包含当前 Laravel 运行的环境。

**跳过验证**

`app/Providers/HorizonServiceProvider.php`

```php
protected function gate()
{
    Gate::define('viewHorizon', function ($user = null) {
        return true;
        return in_array($user->email, [
            'taylor@laravel.com',
        ]);
    });
}
```

参考上一节队列的说明。

```bash
$ lnmp-docker php7-cli php /app/laravel/artisan horizon
```
