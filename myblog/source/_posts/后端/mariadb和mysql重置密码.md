如何在Ubuntu 18.04上重置MySQL或MariaDB Root密码

介绍
忘记密码发生在我们最好的人身上。如果您忘记或丢失了MySQL或MariaDB数据库的root密码，如果您有权访问服务器和具有sudo权限的用户帐户，您仍然可以获得访问权限并重置密码。

注意：在新安装的Ubuntu 18.04上，默认的MySQL或MariaDB配置通常允许您在不提供密码的情况下访问数据库（具有完全管理权限），只要您从系统的root帐户建立连接即可。在这种情况下，可能没有必要重置密码。在继续重置数据库root密码之前，请尝试使用sudo mysql命令访问数据库。如果这导致访问被拒绝错误，请按照本教程中的步骤操作。

本教程演示了如何重置随Ubuntu 18.04上的apt软件包管理器安装的MySQL和MariaDB数据库的root密码。更改root密码的过程取决于您是安装了MySQL还是MariaDB，以及随其他供应商提供的发行版或软件包提供的默认systemd配置。虽然本教程中的说明可能适用于其他系统或数据库服务器版本，但它们已经过专门针对Ubuntu 18.04和分发提供的软件包进行了测试。

准备
要恢复MySQL或MariaDB root密码，您需要：

使用sudo用户或以root权限访问服务器的其他方式访问运行MySQL或MariaDB的Ubuntu 18.04服务器。没有服务器的同学可以在这里购买，不过我个人更推荐您使用免费的腾讯云开发者实验室进行试验，学会安装后再购买服务器。
为了在不影响生产服务器的情况下尝试本教程中的恢复方法，请使用初始服务器创建一个具有sudo权限的常规非root用户的测试服务器。然后按照如何在Ubuntu 18.04上安装MySQL安装MySQL。
步骤1 - 识别数据库版本并停止服务器
Ubuntu 18.04运行MySQL或MariaDB，这是一个流行的替代品，与MySQL完全兼容。您需要使用不同的命令来恢复root密码，具体取决于您安装的密码，因此请按照本节中的步骤确定您正在运行的数据库服务器。

使用以下命令检查您的版本：

mysql --version
如果您正在运行MariaDB，您将在输出中看到“MariaDB”前面带有版本号：

mysql  Ver 15.1 Distrib 10.1.29-MariaDB, for debian-linux-gnu (x86_64) using readline 5.2
如果你正在运行MySQL，你会看到这样的输出：

mysql  Ver 14.14 Distrib 5.7.22, for Linux (x86_64) using  EditLine wrapper
记下哪个数据库，因为这决定了本教程其余部分中要遵循的相应命令。

要更改root密码，您需要关闭数据库服务器。如果您正在运行MariaDB，则可以使用以下命令执行此操作：

sudo systemctl stop mariadb
对于MySQL，通过运行以下命令关闭数据库服务器：

sudo systemctl stop mysql
数据库停止后，您可以在安全模式下重新启动它以重置root密码。

步骤2 - 在没有权限检查的情况下重新启动数据库服务器
在没有权限检查的情况下运行MySQL和MariaDB允许使用root权限访问数据库命令行，而无需提供有效密码。为此，您需要停止数据库加载授权表，该表存储用户权限信息。由于这有一点安全风险，您可能还需要禁用网络以防止其他客户端连接到临时易受攻击的服务器。

根据您安装的数据库服务器，启动服务器而不加载授权表的方式不同。

配置MariaDB以在没有授权表的情况下启动
为了在没有授权表的情况下启动MariaDB服务器，我们将使用systemd单元文件为MariaDB服务器守护程序设置其他参数。

执行以下命令，该命令设置MariaDB在启动时使用的MYSQLD_OPTS环境变量。--skip-grant-tables和--skip-networking选项告诉MariaDB的启动而不加载授权表或网络功能：

sudo systemctl set-environment MYSQLD_OPTS="--skip-grant-tables --skip-networking"
然后启动MariaDB服务器：

sudo systemctl start mariadb
此命令不会产生任何输出，但会重新启动数据库服务器，同时考虑新的环境变量设置。

你可以确保它以sudo systemctl status mariadb开始。

现在，您应该能够以MariaDB root用户身份连接到数据库，而无需提供密码：

sudo mysql -u root
您将立即看到数据库shell提示符：

Type 'help;' or '\h' for help. Type '\c' to clear the current input statement.
​
MariaDB [(none)]>
现在您可以访问数据库服务器，可以更改root密码，如步骤3所示。

配置MySQL以在没有授权表的情况下启动
为了在没有授权表的情况下启动MySQL服务器，您将改变MySQL的systemd配置，以便在启动时将其他命令行参数传递给服务器。

为此，请执行以下命令：

sudo systemctl edit mysql
此命令将在nano编辑器中打开一个新文件，您将使用该文件编辑MySQL的服务覆盖。这些更改了MySQL的默认服务参数。此文件将为空，因此请添加以下内容：

[Service]
ExecStart=
ExecStart=/usr/sbin/mysqld --daemonize --pid-file=/run/mysqld/mysqld.pid --skip-grant-tables --skip-networking
第一个ExecStart语句清除默认值，而第二个语句提供新的启动命令给systemd，包括禁用加载授权表和网络功能的参数。

按CTRL-x退出文件，然后Y保存所做的更改，然后ENTER确认文件名。

重新加载systemd配置以应用这些更改：

sudo systemctl daemon-reload
现在启动MySQL服务器：

sudo systemctl start mysql
该命令将不显示输出，但数据库服务器将启动。授权表和网络将不会启用。

以root用户身份连接到数据库：

sudo mysql -u root
您将立即看到数据库shell提示符：

Type 'help;' or '\h' for help. Type '\c' to clear the current input statement.
​
mysql>
现在您可以访问服务器，您可以更改root密码。

第3步 - 更改Root密码
数据库服务器现在以受限模式运行; 未加载授权表，并且未启用网络支持。这使您可以在不提供密码的情况下访问服务器，但它禁止您执行更改数据的命令。要重置root密码，必须先加载授权表，以便获得对服务器的访问权限。

通过发出FLUSH PRIVILEGES命令告诉数据库服务器重新加载授权表。

FLUSH PRIVILEGES;
您现在可以更改root密码。您使用的方法取决于您使用的是MariaDB还是MySQL。

更改MariaDB密码
如果您使用的是MariaDB，请执行以下语句来设置root帐户的密码，确保替换new_password为您将记住的强大的新密码。

UPDATE mysql.user SET password = PASSWORD('new_password') WHERE user = 'root';
您将看到此输出指示密码已更改：

Query OK, 1 row affected (0.00 sec)
Rows matched: 1  Changed: 1  Warnings: 0
MariaDB允许使用自定义身份验证机制，因此请执行以下两个语句以确保MariaDB将为您分配给root帐户的新密码使用其默认身份验证机制：

UPDATE mysql.user SET authentication_string = '' WHERE user = 'root';
UPDATE mysql.user SET plugin = '' WHERE user = 'root';
您将看到每个语句的以下输出：

Query OK, 0 rows affected (0.01 sec)
Rows matched: 1  Changed: 0  Warnings: 0
密码现在已更改。键入exit以退出MariaDB控制台并继续执行步骤4以在正常模式下重新启动数据库服务器。

更改MySQL密码
对于MySQL，执行以下语句来更改root用户的密码，替换new_password为您将记住的强密码：

UPDATE mysql.user SET authentication_string = PASSWORD('new_password') WHERE user = 'root';
您将看到此输出表明密码已成功更改：

Query OK, 1 row affected (0.00 sec)
Rows matched: 1  Changed: 1  Warnings: 0
MySQL允许使用自定义身份验证机制，因此执行以下语句告诉MySQL使用其默认身份验证机制来使用新密码对root用户进行身份验证：

UPDATE mysql.user SET plugin = 'mysql_native_password' WHERE user = 'root';
您将看到类似于上一个命令的输出：

Query OK, 1 row affected (0.00 sec)
Rows matched: 1  Changed: 1  Warnings: 0
密码现在已更改。键入exit以退出MySQL控制台。

让我们以正常运行模式重启数据库。

第4步 - 将数据库服务器恢复为正常设置
为了以正常模式重新启动数据库服务器，您必须还原所做的更改，以便启用网络并加载授权表。同样，您使用的方法取决于您使用的是MariaDB还是MySQL。

对于MariaDB，取消设置先前设置的MYSQLD_OPTS环境变量：

sudo systemctl unset-environment MYSQLD_OPTS
然后，使用systemctl重启服务：

sudo systemctl restart mariadb
对于MySQL，删除修改后的systemd配置：

sudo systemctl revert mysql
您将看到类似于以下内容的输出：

Removed /etc/systemd/system/mysql.service.d/override.conf.
Removed /etc/systemd/system/mysql.service.d.
然后，重新加载systemd配置以应用更改：

sudo systemctl daemon-reload
最后，重启服务：

sudo systemctl restart mysql
数据库现在重新启动并恢复到正常状态。通过以root用户身份使用密码登录来确认新密码是否有效：

mysql -u root -p
系统将提示您输入密码。输入新密码，您将按预期访问数据库提示。

结论
您已恢复对MySQL或MariaDB服务器的管理访问权限。确保您选择的新密码强大且安全，并将其保存在安全的地方

转载自https://cloud.tencent.com/developer/article/1359782