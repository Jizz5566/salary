---
title: '在CentOS7安裝與設定Nagios'
disqus: hackmd
---

在CentOS7安裝與設定Nagios
===

---
## Nagios Server端
### 下載必要程式
1. 創建下載資料夾
> mkdir /software
> cd /software

2. 下載程式
[nagios-4.4.3.tar.gz](https://assets.nagios.com/downloads/nagioscore/releases/nagios-4.4.3.tar.gz)
[nrpe-3.2.1.tar.gz](https://sourceforge.net/projects/nagios/files/nrpe-3.x/nrpe-3.2.1.tar.gz)
[nagios-plugins-2.2.1.tar.gz](https://nagios-plugins.org/download/nagios-plugins-2.2.1.tar.gz)

### 安裝Nagios主程式

1. 安裝依賴包
> yum -y install httpd httpd-devel gcc glibc glibc-common gd gd-devel perl-devel perl-CPAN fcgi perl-FCGI perl-FCGI-ProcManager
> 

2. 解壓縮Nagios Core
>tar zxvf nagios-4.4.3.tar.gz
>tar zxvf nrpe-3.2.1.tar.gz
>tar zxvf nagios-plugins-2.2.1.tar.gz
>

3. 進入目錄
> cd nagios-4.4.3/
> 

4. 創建用戶組
> useradd nagios -s /sbin/nologin
> useradd apache -s /sbin/nologin
> id www
> groupadd nagcmd
> usermod -a -G nagcmd nagios
> usermod -a -G nagcmd www
> usermod -a -G nagcmd apache
> id -n -G nagios
> id -n -G www
> id -n -G apache

6. 編譯Nagios
> ./configure --with-command-group=nagcmd
> 
7. 編譯與安裝
> make all
> make install-init
> make install-commandmode
> make install-config
8. 複製必要模組檔案（包含檔案權限）
> cp -R contrib/eventhandlers/ /usr/local/nagios/libexec/
> chown -R nagios:nagios /usr/local/nagios/libexec/eventhandlers
> /usr/local/nagios/bin/nagios -v /usr/local/nagios/etc/nagios.cfg

9. 安裝完成，跳回上一層目錄
> cd ..

### 安裝Nagios Plug-in
1. nagios-plugin（略）
2. 安裝目錄並且配置
>cd nagios-plugins-2.2.1/
>./configure --with-nagios-user=nagios --with-nagios-group=nagcmd --enable-perl-modules

3. 編譯並安裝
>make && make install
>

### 安裝Nrpe
1. 解壓縮Nrpe（略）
2. 進入安裝目錄並且配置
>./configure 

3. 編譯並安裝
>make all
>make install-plugin
>make install-daemon
>make install-config
>cp sample-config/nrpe.cfg /usr/local/nagios/etc/nrpe.cfg 

4. 確認libexec目錄模組
>ls /usr/local/nagios/libexec/

5. 啟動Nrpe，確認服務正常，並且開機自動執行
>/usr/local/nagios/bin/nrpe -d -c /usr/local/nagios/etc/nrpe.cfg 
>echo "/usr/local/nagios/bin/nrpe -d -c /usr/local/nagios/etc/nrpe.cfg" >> /etc/rc.local
>netstat -lnput | grep 5666

6. 測試Nrpe
>/usr/local/nagios/libexec/check_nrpe -H localhost 
NRPE v3.1.0-rc1
---
## Nagios Client端
### 下載必要程式
1. 創建下載資料夾
> mkdir /software
> cd /software

2. 下載程式
[nrpe-3.2.1.tar.gz](https://sourceforge.net/projects/nagios/files/nrpe-3.x/nrpe-3.2.1.tar.gz)
[nagios-plugins-2.2.1.tar.gz](https://nagios-plugins.org/download/nagios-plugins-2.2.1.tar.gz)

### 安裝Nagios Plug-in

1. 安裝依賴包
> yum install perl-devel perl-CPAN -y

2. 創建用戶
> useradd nagios -M -s /sbin/nologin

3. 解壓縮nagios-plugin（略）
4. 進入目錄後配置
> cd nagios-plugins-2.2.1/
> ./configure --with-nagios-user=nagios --with-nagios-group=nagios --enable-perl-modules
> 
5. 編譯與安裝
> make && make install
> 
### 安裝Nrpe
1. 解壓縮Nrpe（略）
2. 進入安裝目錄並且配置
>./configure 

3. 編譯並安裝
>make all
>make install-plugin
>make install-daemon
>make install-config
>mkdir /usr/local/nagios/etc/
>cp sample-config/nrpe.cfg /usr/local/nagios/etc/nrpe.cfg 

4. 確認libexec目錄模組
>ls /usr/local/nagios/libexec/

5. 啟動Nrpe，確認服務正常，並且開機自動執行
>/usr/local/nagios/bin/nrpe -d -c /usr/local/nagios/etc/nrpe.cfg 
>echo "/usr/local/nagios/bin/nrpe -d -c /usr/local/nagios/etc/nrpe.cfg" >> /etc/rc.local
>netstat -lnput|grep 5666

6. 測試Nrpe
>/usr/local/nagios/libexec/check_nrpe -H localhost 

NRPE v3.1.0-rc1

7. 修改配置文件
> cd /usr/local/nagios/etc/
> vi nrpe.cfg
> allowed_hosts=127.0.0.1,::1    ===>        allowed_hosts=127.0.0.1,::1,"Natios Server IP"(52.163.241.87)

8. 註釋以下內容
> command[check_users]=/usr/local/nagios/libexec/check_users -w 5 -c 10
command[check_load]=/usr/local/nagios/libexec/check_load -r -w .15,.10,.05 -c .30,.25,.20
command[check_hda1]=/usr/local/nagios/libexec/check_disk -w 20% -c 10% -p /dev/hda1
command[check_zombie_procs]=/usr/local/nagios/libexec/check_procs -w 5 -c 10 -s Z
command[check_total_procs]=/usr/local/nagios/libexec/check_procs -w 150 -c 200

9. 在底部增加以下內容

參考[設定檔範例](https://hackmd.io/Ft1ICg3kTzOkXrhyUTxJFw?both#nrpecfg%E8%A8%AD%E5%AE%9AClient%E7%AB%AF)

說明：
* **check_3306&63799** 因PHP Server的Mysql與Redis不對外開放Port，所以透過check_tcp模組本機127.0.0.1服務，經由check_nrpe輸出狀態

* 底下的 **[restart_xxx]** 設定為提供給TelegramBot呼叫的指令。經由Telegram聊天群組發送指令，將指令透過TB機器人丟給Server主機端的Go程式執行，再將結果傳回，實現手機端重啟服務的功能。

***注意！nagios使用者需要開啟在client端開啟root權限，才能執行系統重啟動作***

* 

10. 新增監控記憶體的腳本 (注意！要用vi程式編輯)
> vi /usr/local/nagios/libexec/check_mem.pl

參考[設定檔範例](https://hackmd.io/Ft1ICg3kTzOkXrhyUTxJFw?both#check_mem%E8%85%B3%E6%9C%AC%E4%BD%BF%E7%94%A8vi)


11. 增加腳本的執行權限
> chmod 755 /usr/local/nagios/libexec/check_mem.pl

12. 重啟Nrpe服務
> kill -HUP `ps -ef|grep nrpe|awk 'NR==1{print $2}'`
![](https://i.imgur.com/pice9Jj.png)

13. 本機狀態下執行命令查看結果
> /usr/local/nagios/libexec/check_nrpe -H localhost -c check_mem
> /usr/local/nagios/libexec/check_nrpe -H localhost -c check_disk

---

## Nagios Server端設定
### 測試與Client端連通
> /usr/local/nagios/libexec/check_nrpe -H "Client端IP"
> NRPE v3.1.0-rc1
> /usr/local/nagios/libexec/check_nrpe -H "Client端IP" -c check_disk
> DISK OK - free space: / 4201 MB (24.15% inode=97%);| /=13192MB;13915;15654;0;17394

如果不通或是出現錯誤，請檢查client side的防火牆或是selinux等設定

### 修改cgi.cfg檔案，給予nagios帳號權限（與nagiosadmin相同）
> cd /usr/local/nagios/etc
> grep nagiosadmin cgi.cfg

> authorized_for_system_information=nagiosadmin
authorized_for_configuration_information=nagiosadmin
authorized_for_system_commands=nagiosadmin
authorized_for_all_services=nagiosadmin
authorized_for_all_hosts=nagiosadmin
authorized_for_all_service_commands=nagiosadmin
authorized_for_all_host_commands=nagiosadmin

> sed -i 's/nagiosadmin/nagiosadmin,nagios,apache/g' cgi.cfg
> grep nagiosadmin cgi.cfg


authorized_for_system_information=**nagiosadmin,nagios,apache**
authorized_for_configuration_information=**nagiosadmin,nagios,apache**
authorized_for_system_commands=**nagiosadmin,nagios,apache**
authorized_for_all_services=**nagiosadmin,nagios,apache**
authorized_for_all_hosts=**nagiosadmin,nagios,apache**
authorized_for_all_service_commands=**nagiosadmin,nagios,apache**
authorized_for_all_host_commands=**nagiosadmin,nagios,apache**

> vim cgi.cfg
```gherkin=
physical_html_path=/usr/share/nginx/html/nagios
……
url_html_path=/
```


### 修改nagios.cfg
> vi nagios.cfg +34
 
1. 註解原始設定
> #cfg_file=/usr/local/nagios/etc/objects/localhost.cfg

2. 增加設定

> cfg_file=/usr/local/nagios/etc/objects/services.cfg
cfg_file=/usr/local/nagios/etc/objects/hosts.cfg

### 新增host.cfg與service.cfg並且複製權限
> cd objects/
> touch services.cfg
> head -51 localhost.cfg  > hosts.cfg
> chown -R nagios.nagios *

### 修改nagios啟動腳本
> vim /etc/init.d/nagios +181
#check_config
$NagiosBin -v $NagiosCfgFile;


---
## 修改設定文件（Server端）

接著是按照需求修改cfg文件，主要服務架構如下：
```sequence
Title: Nagios檔案調用流程
Nagios Web->Service.cfg: Core請求服務內容
Note over Service.cfg: 語法：\n[指令]![參數1]![參數2]
Service.cfg->Command.cfg: 讀取command內容
Note over Command.cfg,libexec: command_name\n command_line\n …
participant libexec
Service.cfg->Command.cfg: 傳入設定檔參數(ex:$ARG1$)
Note right of Command.cfg: [指令][參數$xxx$][…]
Command.cfg-->>Nagios Web: 將執行結果Post到Nagios
```

[Nagios異常回報流程(點我開大圖)](https://i.imgur.com/FnXxRXj.png)

```sequence
Title: Nagios異常回報流程
participant Command.cfg
participant Service.cfg
participant Contact.cfg
Note over Command.cfg,Service.cfg: Command.cfg:紀錄可被模組或程式調用的指令集\nContact.cfg:系統發出通知的收件者資訊\nService.cfg:在Nagios執行的服務
Note over Telegram Bot,Telegram Group: Telegram Bot:取得的Telegram API Key獲得Token與room id\nTelegram Group:Telegram聊天群組加入Bot可收到Nagios通知
Contact.cfg->Command.cfg: [1]異常狀況發生\nContact呼叫Command B.cfg\n<notify-service-by-xxx>
Note left of Command.cfg: 執行指定command_name的指令:\n<command_line>
Command.cfg->Telegram Bot: [2]透過指令與參數將監測資料即時回傳給Telegram Bot機器人
Telegram Bot->Telegram Group: [3]聊天室顯示Nagios告警訊息
```
![](https://i.imgur.com/FnXxRXj.png)

### 配置commands.cfg
修改 /usr/local/nagios/etc/objects/command.cfg
（[參考範本](https://hackmd.io/Ft1ICg3kTzOkXrhyUTxJFw?both#commandcfg)）
Shift+G跳到底部增加以下內容：
說明：需依照實際需求修改內容

### 配置contacts.cfg 
修改 /usr/local/nagios/etc/objects/contacts.cfg :
 ex:<notify-host-by-telegram>


> sed -i 's/nagiosadmin/nagiosadmin,nagios/g' cgi.cfg
> grep nagiosadmin cgi.cfg
> 
```gherkin=
define contact {
        contact_name     nagios      ; Short name of user
        use              generic-contact    ; Inherit default values from generic-contact template (defined above)
        alias            Nagios Admin     ; Full name of user
        email            root@localhost ;
        host_notification_commands    notify-host-by-telegram
        host_notification_options     d,u,r
        service_notification_commands notify-service-by-telegram
        service_notification_options  w,u,c,r
}

```
說明：
此設定檔重點在
***(host, service)_notification_commands*** 
需要定義異常呼叫的方式
與 
***(host, service)_notification_options***
需要定義異常呼叫的範圍

若是(主機,服務)異常呼叫方式使用Telegram，則此欄位就要定義為 ***notify-(host,service)-by-telegram***

若要更改(主機,服務)異常呼叫的定義範圍，此欄位需要定義
***w,d,u,r***
參考說明：
>	This directive is used to define the service states for which notifications can be sent out to this contact. Valid options are a combination of one or more of the following: 
>w = notify on WARNING service states, u = notify on UNKNOWN service states, 
>c = notify on CRITICAL service states, 
>r = notify on service recoveries (OK states), and f = notify when the service starts and stops flapping. 
>n = If you specify n (none) as an option, the contact will not receive any type of service notifications.

### 配置host.cfg 
修改 /usr/local/nagios/etc/objects/hosts.cfg:
> vi hosts.cfg +21

修改內容請參考[設定檔](https://hackmd.io/Ft1ICg3kTzOkXrhyUTxJFw?both#hostscfg)

### 配置service.cfg 
修改 /usr/local/nagios/etc/objects/service.cfg:
新增設定，參考[設定檔](https://hackmd.io/Ft1ICg3kTzOkXrhyUTxJFw?both#servicecfg)

### 重啟Nagios服務
> systemctl restart nagios

### 開啟瀏覽器確認Nagios啟動
因Nagios是建立在Web Servicer之上，若開啟網址無法進入Nagios，請檢查Nagios使用者權限，或是Nginx的設定檔，特別是php設定
設定容易在nginx路徑設置與php部份設定出錯，需要特別注意

另外nagios的可視化外掛程式－pnp4nagios則另篇說明



## 參考設定檔
### Nginx設定

:::info
/etc/nginx/conf.d/nagios.conf 
:::

```gherkin=
server{
        listen       80;
        listen  443 ssl;
        server_name  nagios.fitiapp.com;
        access_log /var/log/nginx/nagios_access.log;
        error_log /var/log/nginx/nagios_error.log warn;
        root /usr/share/nginx/html/nagios;
        index index.php index.html;

        location / {
                auth_basic "Private Website!!";
                auth_basic_user_file /usr/local/nagios/etc/htpasswd.users;
                try_files $uri $uri/ /index.php;
        }
        location ~ \.php$ {
                try_files $uri =404;
                fastcgi_pass   127.0.0.1:9000;
                fastcgi_index  index.php;
                fastcgi_split_path_info ^(.+\.php)(/.+)$;
                fastcgi_param  SCRIPT_FILENAME  $document_root$fastcgi_script_name;
                fastcgi_param  SCRIPT_NAME  $fastcgi_script_name;
                include        fastcgi_params;
        }

        location  /nagiosql {
                gzip off;
                alias /usr/share/nginx/html/nagios/nagiosql;
                index index.php index.html;
        }
        location ~ \.cgi$ {
                gzip    off;
                auth_basic "Private Website!!";
                auth_basic_user_file /usr/local/nagios/etc/htpasswd.users;
                root    /usr/share/nginx/html/nagios/cgi-bin/sbin;
                rewrite ^/cgi-bin/(.*)\.cgi /$1.cgi break;
                fastcgi_pass  unix:/var/run/fcgiwrap.socket;
                fastcgi_param SCRIPT_FILENAME $document_root$fastcgi_script_name;
                fastcgi_index index.cgi;
                include fastcgi_params;
        }

        location /pnp4nagios {
                alias /usr/local/pnp4nagios/share;
                index index.php;
                try_files $uri $uri/ @pnp4nagios;
        }

        location @pnp4nagios {
                fastcgi_pass   127.0.0.1:9000;
                fastcgi_index       index.php;
                fastcgi_split_path_info ^(.+\.php)(.*)$;
                fastcgi_param PATH_INFO $fastcgi_path_info;
                include        fastcgi_params;
                fastcgi_param SCRIPT_FILENAME /usr/local/pnp4nagios/share/index.php;
        }

}

```
### cgi.cfg設定
:::info
/usr/local/nagios/etc/cgi.cfg
:::

>grep nagiosadmin cgi.cfg

>sed -i 's/nagiosadmin/nagiosadmin,nagios/g' cgi.cfg

>grep nagiosadmin cgi.cfg
```gherkin=
use_authentication=1

default_user_name=nagios

authorized_for_system_information=nagiosadmin,nagios
authorized_for_configuration_information=nagiosadmin,nagios
authorized_for_system_commands=nagiosadmin,nagios
authorized_for_all_services=nagiosadmin,nagios
authorized_for_all_hosts=nagiosadmin,nagios
authorized_for_all_service_commands=nagiosadmin,nagios
authorized_for_all_host_commands=nagiosadmin,nagios
```

### nrpe.cfg設定(Server端)
:::info
/usr/local/nagios/etc/nrpe.cfg
:::
修改106行
>allowed_hosts=127.0.0.1,::1,[Nagios伺服器IP]

以下註解掉：
```
#command[check_users]=/usr/local/nagios/libexec/check_users -w 5 -c 10
#command[check_load]=/usr/local/nagios/libexec/check_load -r -w .15,.10,.05 -c .30,.25,.20
#command[check_hda1]=/usr/local/nagios/libexec/check_disk -w 20% -c 10% -p /dev/hda1
#command[check_zombie_procs]=/usr/local/nagios/libexec/check_procs -w 5 -c 10 -s Z
#command[check_total_procs]=/usr/local/nagios/libexec/check_procs -w 150 -c 200
```

最下方加入以下內容：

```gherkin=
# my custom monitor items
command[check_users]=/usr/local/nagios/libexec/check_users -w 5 -c 10
command[check_load]=/usr/local/nagios/libexec/check_load -r -w .15,.10,.05 -c .30,.25,.20
command[check_disk]=/usr/local/nagios/libexec/check_disk -w 20% -c 10% -p /
command[check_mem]=/usr/local/nagios/libexec/check_mem.pl -w 90% -c 95%
command[check_swap]=/usr/local/nagios/libexec/check_swap -w 20% -c 10%
```

### nrpe.cfg設定(Client端)
:::info
/usr/local/nagios/etc/nrpe.cfg
:::
修改以下設定：
```gherkin=
server_port=5666
nrpe_user=nagios
nrpe_group=nagios
allowed_hosts=127.0.0.1,::1,[Server端IP]
```
加入以下設定：
```gherkin=
# my custom monitor items
command[check_users]=/usr/local/nagios/libexec/check_users -w 5 -c 10
command[check_load]=/usr/local/nagios/libexec/check_load -r -w .50,.30,.20 -c .80,.60,.40
command[check_disk]=/usr/local/nagios/libexec/check_disk -w 20% -c 10% -p /
command[check_mem]=/usr/local/nagios/libexec/check_mem.pl -w 90% -c 95%
command[check_swap]=/usr/local/nagios/libexec/check_swap -w 20% -c 10%
command[check_3306]=/usr/local/nagios/libexec/check_tcp -H 127.0.0.1 -p 3306 -w 1 -c 5
command[check_63799]=/usr/local/nagios/libexec/check_tcp -H 127.0.0.1 -p 63799 -w 1 -c 5
command[restart_redis]=sudo service redis_63799 restart | tee -a /tmp/restart_redis_63799.txt ; sleep 3s ;tail -n 2 /var/log/redis_63799.log | awk 'BEGIN {FS=" "}NR==1{print $7 " loaded from disk: " $11 " seconds"}NR==2{print $7 " " $8 " " $9 " " $10}'
command[restart_nginx]=sudo service nginx restart ; sudo grep nginx /var/log/messages | tail -n 4 | awk 'BEGIN {FS=" "}NR==1 {print "Starting the " $8 " " $9 " and " $11 " " $12 " " $13 "..."}NR==2{print $10 " syntax is " $13}NR==3{print $9 " test is " $12}NR==4{print "Started The " $8 " " $9 " and " $11 " " $12 " " $13}'
command[restart_mysql]=sudo service mysqld restart ; sleep 3s ; sudo tail -n 4 /var/log/mysqld.log | awk 'BEGIN {FS=" "}NR==1 {print $7 $8 " start!!! process: " $12}NR==2{print $8 " " $9 " "  $10 " " $11}NR==3{print $6 "ready for connections, port: " $15}NR==4{print $6 " "  $7 " ready for connection, port: " $16}'

```
### check_mem腳本(使用vi)
:::info
vi /usr/local/nagios/libexec/check_mem.pl
:::

```gherkin=
#! /usr/bin/perl -w
#
# $Id: check_mem.pl 8 2008-08-23 08:59:52Z rhomann $
#
# check_mem v1.7 plugin for nagios
#
# uses the output of `free` to find the percentage of memory used
#
# Copyright Notice: GPL
#
# History:
# v1.8 Rouven Homann - rouven.homann@cimt.de
# + added findbin patch from Duane Toler
# + added backward compatibility patch from Timour Ezeev
#
# v1.7 Ingo Lantschner - ingo AT boxbe DOT com
# + adapted for systems with no swap (avoiding divison through 0)
#
# v1.6 Cedric Temple - cedric DOT temple AT cedrictemple DOT info
# + add swap monitoring
#       + if warning and critical threshold are 0, exit with OK
#       + add a directive to exclude/include buffers
#
# v1.5 Rouven Homann - rouven.homann@cimt.de
# + perfomance tweak with free -mt (just one sub process started instead of 7)
# + more code cleanup
#
# v1.4 Garrett Honeycutt - gh@3gupload.com
# + Fixed PerfData output to adhere to standards and show crit/warn values
#
# v1.3 Rouven Homann - rouven.homann@cimt.de
#   + Memory installed, used and free displayed in verbose mode
# + Bit Code Cleanup
#
# v1.2 Rouven Homann - rouven.homann@cimt.de
# + Bug fixed where verbose output was required (nrpe2)
#       + Bug fixed where perfomance data was not displayed at verbose output
# + FindBin Module used for the nagios plugin path of the utils.pm
#
# v1.1 Rouven Homann - rouven.homann@cimt.de
#     + Status Support (-c, -w)
# + Syntax Help Informations (-h)
#       + Version Informations Output (-V)
# + Verbose Output (-v)
#       + Better Error Code Output (as described in plugin guideline)
#
# v1.0 Garrett Honeycutt - gh@3gupload.com
#   + Initial Release
#
use strict;
use FindBin;
FindBin::again();
use lib $FindBin::Bin;
use utils qw($TIMEOUT %ERRORS &print_revision &support);
use vars qw($PROGNAME $PROGVER);
use Getopt::Long;
use vars qw($opt_V $opt_h $verbose $opt_w $opt_c);

$PROGNAME = "check_mem";
$PROGVER = "1.8";

# add a directive to exclude buffers:
my $DONT_INCLUDE_BUFFERS = 0;

sub print_help ();
sub print_usage ();

Getopt::Long::Configure('bundling');
GetOptions ("V"   => \$opt_V, "version"    => \$opt_V,
  "h"   => \$opt_h, "help"       => \$opt_h,
        "v" => \$verbose, "verbose"  => \$verbose,
  "w=s" => \$opt_w, "warning=s"  => \$opt_w,
  "c=s" => \$opt_c, "critical=s" => \$opt_c);

if ($opt_V) {
  print_revision($PROGNAME,'$Revision: '.$PROGVER.' $');
  exit $ERRORS{'UNKNOWN'};
}

if ($opt_h) {
  print_help();
  exit $ERRORS{'UNKNOWN'};
}

print_usage() unless (($opt_c) && ($opt_w));

my ($mem_critical, $swap_critical);
my ($mem_warning, $swap_warning);
($mem_critical, $swap_critical) = ($1,$2) if ($opt_c =~ /([0-9]+)[%]?(?:,([0-9]+)[%]?)?/);
($mem_warning, $swap_warning)   = ($1,$2) if ($opt_w =~ /([0-9]+)[%]?(?:,([0-9]+)[%]?)?/);

# Check if swap params were supplied
$swap_critical ||= 100;
$swap_warning  ||= 100;

# print threshold in output message
my $mem_threshold_output = " (";
my $swap_threshold_output = " (";

if ( $mem_warning > 0 && $mem_critical > 0) {
  $mem_threshold_output .= "W> $mem_warning, C> $mem_critical";
}
elsif ( $mem_warning > 0 ) {
  $mem_threshold_output .= "W> $mem_warning";
}
elsif ( $mem_critical > 0 ) {
  $mem_threshold_output .= "C> $mem_critical";
}

if ( $swap_warning > 0 && $swap_critical > 0) {
  $swap_threshold_output .= "W> $swap_warning, C> $swap_critical";
}
elsif ( $swap_warning > 0 ) {
  $swap_threshold_output .= "W> $swap_warning";
}
elsif ( $swap_critical > 0 )  {
  $swap_threshold_output .= "C> $swap_critical";
}

$mem_threshold_output .= ")";
$swap_threshold_output .= ")";

my $verbose = $verbose;

my ($mem_percent, $mem_total, $mem_used, $swap_percent, $swap_total, $swap_used) = &sys_stats();
my $free_mem = $mem_total - $mem_used;
my $free_swap = $swap_total - $swap_used;

# set output message
my $output = "Memory Usage".$mem_threshold_output.": ". $mem_percent.'% <br>';
$output .= "Swap Usage".$swap_threshold_output.": ". $swap_percent.'%';

# set verbose output message
my $verbose_output = "Memory Usage:".$mem_threshold_output.": ". $mem_percent.'% '."- Total: $mem_total MB, used: $mem_used MB, free: $free_mem MB<br>";
$verbose_output .= "Swap Usage:".$swap_threshold_output.": ". $swap_percent.'% '."- Total: $swap_total MB, used: $swap_used MB, free: $free_swap MB<br>";

# set perfdata message
my $perfdata_output = "MemUsed=$mem_percent\%;$mem_warning;$mem_critical";
$perfdata_output .= " SwapUsed=$swap_percent\%;$swap_warning;$swap_critical";


# if threshold are 0, exit with OK
if ( $mem_warning == 0 ) { $mem_warning = 101 };
if ( $swap_warning == 0 ) { $swap_warning = 101 };
if ( $mem_critical == 0 ) { $mem_critical = 101 };
if ( $swap_critical == 0 ) { $swap_critical = 101 };


if ($mem_percent>$mem_critical || $swap_percent>$swap_critical) {
    if ($verbose) { print "<b>CRITICAL: ".$verbose_output."</b>|".$perfdata_output."\n";}
    else { print "<b>CRITICAL: ".$output."</b>|".$perfdata_output."\n";}
    exit $ERRORS{'CRITICAL'};
} elsif ($mem_percent>$mem_warning || $swap_percent>$swap_warning) {
    if ($verbose) { print "<b>WARNING: ".$verbose_output."</b>|".$perfdata_output."\n";}
    else { print "<b>WARNING: ".$output."</b>|".$perfdata_output."\n";}
    exit $ERRORS{'WARNING'};
} else {
    if ($verbose) { print "OK: ".$verbose_output."|".$perfdata_output."\n";}
    else { print "OK: ".$output."|".$perfdata_output."\n";}
    exit $ERRORS{'OK'};
}

sub sys_stats {
    my @memory = split(" ", `free -mt`);
    my $mem_total = $memory[7];
    my $mem_used;
    if ( $DONT_INCLUDE_BUFFERS) { $mem_used = $memory[15]; }
    else { $mem_used = $memory[8];}
    my $swap_total = $memory[18];
    my $swap_used = $memory[19];
    my $mem_percent = ($mem_used / $mem_total) * 100;
    my $swap_percent;
    if ($swap_total == 0) {
  $swap_percent = 0;
    } else {
  $swap_percent = ($swap_used / $swap_total) * 100;
    }
    return (sprintf("%.0f",$mem_percent),$mem_total,$mem_used, sprintf("%.0f",$swap_percent),$swap_total,$swap_used);
}

sub print_usage () {
    print "Usage: $PROGNAME -w <warn> -c <crit> [-v] [-h]\n";
    exit $ERRORS{'UNKNOWN'} unless ($opt_h);
}

sub print_help () {
    print_revision($PROGNAME,'$Revision: '.$PROGVER.' $');
    print "Copyright (c) 2005 Garrett Honeycutt/Rouven Homann/Cedric Temple\n";
    print "\n";
    print_usage();
    print "\n";
    print "-w <MemoryWarn>,<SwapWarn> = Memory and Swap usage to activate a warning message (eg: -w 90,25 ) .\n";
    print "-c <MemoryCrit>,<SwapCrit> = Memory and Swap usage to activate a critical message (eg: -c 95,50 ).\n";
    print "-v = Verbose Output.\n";
    print "-h = This screen.\n\n";
    support();
}
```
### hosts.cfg
:::info
/usr/local/nagios/etc/objects/hosts.cfg
:::

```gherkin=
define hostgroup {

    hostgroup_name          Azure-servers           ; The name of the hostgroup
    alias                   Linux Servers           ; Long name of the group
    members                 Azure-SSR              ; Comma separated list of hosts that belong to this group
}

# Define some hosts

###########127.0.0.1##################

define host {
        host_name                Azure-SSR
        alias                    Azure-SSR
        address                  127.0.0.1
        check_command            check-host-alive
        max_check_attempts        3
        normal_check_interval     2
        retry_check_interval      2
        check_period              24x7
        notification_interval     300
        notification_period       24x7
        notification_options      d,u,r
        contact_groups            admins
        process_perf_data         1
}
###########210.209.15.121##################

define host {
        #use                     linux-server,host-pnp
        host_name                Risk-ES1
        alias                    ES1
        address                  210.209.15.121
        check_command            check-host-alive-es1
        max_check_attempts        3
        check_interval            2
        retry_interval            2
        check_period              24x7
        notification_interval     3
        notification_period       24x7
        notification_options      d,u,r
        contact_groups            admins
        process_perf_data         1
}

```

### service.cfg
:::info
/usr/local/nagios/etc/objects/service.cfg
:::
```gherkin=

###########127.0.0.1##################

define service{
        use                     generic-service,service-pnp
        host_name               Azure-SSR
        service_description     CPU_Load
        check_interval          3
        retry_interval          3
        notification_interval   3
        check_command           check_nrpe!check_load
#这里的check_nrpe不是服务端/usr/local/nagios/libexec/check_nrpe,而是command.cfg里定义的命令
        }

define service{
        use                     generic-service,service-pnp
        host_name               Azure-SSR
        service_description     Disk
        check_interval          3
        retry_interval          3
        notification_interval   3
        check_command           check_nrpe!check_disk
        }

define service{
        use                     generic-service,service-pnp
        host_name               Azure-SSR
        service_description     Memory
        check_interval          3
        retry_interval          3
        notification_interval   3
        check_command           check_nrpe!check_mem
        }
define service{
        use                     generic-service,service-pnp
        host_name               Azure-SSR
        service_description     Ping
        check_interval          3
        retry_interval          3
        notification_interval   3
        check_command           check-ping!10.0.0.7
}

define service{
        use                     generic-service,service-pnp
        host_name               Azure-SSR
        service_description     port_3306
        check_interval          3
        retry_interval          3
        notification_interval   3
        check_command           check-tcp!3306
}

define service{

        use                     generic-service,service-pnp
        host_name               Azure-SSR
        service_description     port_443
        check_interval          3
        retry_interval          3
        notification_interval   3
        check_command           check-tcp!443
}
define service{
        use                     generic-service,service-pnp
        host_name               Azure-SSR
        service_description     port_63799
        check_interval          2
        retry_interval          1
        notification_interval   2
        check_command           check-tcp!63799
}

###########210.209.15.121##################

define service{
        use                     generic-service,service-pnp
        host_name               Risk-ES1
        service_description     CPU_Load
        check_interval          3
        retry_interval          3
        notification_interval   3
        check_command           check_nrpe!check_load
#这里的check_nrpe不是服务端/usr/local/nagios/libexec/check_nrpe,而是command.cfg里定义的命令
        }

define service{
        use                     generic-service,service-pnp
        host_name               Risk-ES1
        service_description     Disk
        check_interval          3
        retry_interval          3
        notification_interval   3
        check_command           check_nrpe!check_disk
        }

define service{
        use                     generic-service,service-pnp
        host_name               Risk-ES1
        service_description     Memory
        check_interval          3
        retry_interval          3
        notification_interval   3
        check_command           check_nrpe!check_mem
        }
define service{
        use                     generic-service,service-pnp
        host_name               Risk-ES1
        service_description     MySql
        check_interval          3
        retry_interval          3
        notification_interval   3
        check_command           check_nrpe!check_3306
        }

define service{
        use                     generic-service,service-pnp
        host_name               Risk-ES1
        service_description     Redis_63799
        check_interval          3
        retry_interval          3
        notification_interval   3
        check_command           check_nrpe!check_63799
        }

define service{
        use                     generic-service,service-pnp
        host_name               Risk-ES1
        display_name            "https://panel.c-rainbow.cn/health.php"
        service_description     MainPanel
        check_interval          3
        retry_interval          3
        notification_interval   3
        check_command           check-https!panel.c-rainbow.cn
}
define service{
        use                     generic-service,service-pnp
        host_name               Risk-ES1
        display_name            "https://risk.c-rainbow.cn/health.php"
        service_description     RiskAPI
        check_interval          3
        retry_interval          3
        notification_interval   3
        check_command           check-https!risk.c-rainbow.cn
}
define service{
        use                     generic-service,service-pnp
        host_name               Risk-ES1
        display_name            "https://api.c-rainbow.cn/test/health"
        service_description     HzAPI
        check_interval          3
        retry_interval          3
        notification_interval   3
        check_command           check-https2!api.c-rainbow.cn!test
}



define service{
        name telegram-service
        _SENDTELEGRAM 1; 
        register 0; 
}

define service{
        name telegram-host
        _SENDTELEGRAM 1; 
        register 0; 
}

```

### command.cfg
:::info
/usr/local/nagios/etc/objects/command.cfg
:::

```gherkin=
#####################Email Notifications####################################

define command {

    command_name    notify-host-by-email
    command_line    /usr/bin/printf "%b" "***** Nagios *****\n\nNotification Type: $NOTIFICATIONTYPE$\nHost: $HOSTNAME$\nState: $HOSTSTATE$\nAddress: $HOSTADDRESS$\nInfo: $HOSTOUTPUT$\n\nDate/Time: $LONGDATETIME$\n" | /bin/mail -s "** $NOTIFICATIONTYPE$ Host Alert: $HOSTNAME$ is $HOSTSTATE$ **" $CONTACTEMAIL$
}



define command {

    command_name    notify-service-by-email
    command_line    /usr/bin/printf "%b" "***** Nagios *****\n\nNotification Type: $NOTIFICATIONTYPE$\n\nService: $SERVICEDESC$\nHost: $HOSTALIAS$\nAddress: $HOSTADDRESS$\nState: $SERVICESTATE$\n\nDate/Time: $LONGDATETIME$\n\nAdditional Info:\n\n$SERVICEOUTPUT$\n" | /bin/mail -s "** $NOTIFICATIONTYPE$ Service Alert: $HOSTALIAS$/$SERVICEDESC$ is $SERVICESTATE$ **" $CONTACTEMAIL$
}


#####################Check Host Alive####################################

define command {

    command_name    check-host-alive-az
    command_line    $USER1$/check_ping -H $HOSTADDRESS$ -w 3000.0,80% -c 5000.0,100% -p 5
}


define command {

    command_name    check-host-alive-es1
    command_line    $USER1$/check_tcp -H $HOSTADDRESS$ -p 51561 -w 0.2 -c 2
}

#####################Telegram Notifications####################################
# commands to send host/service notifications
define command {

	command_name    notify-host-by-telegram
	command_line	curl -X  POST --data chat_id=-1001207542372 --data text="***** Nagios *****%0A%0ANotification Type: $NOTIFICATIONTYPE$%0AHost: $HOSTNAME$%0AState: $HOSTSTATE$%0AAddress: $HOSTADDRESS$%0AInfo: $HOSTOUTPUT$%0ADate/Time: $LONGDATETIME$%0A" "https://api.telegram.org/bot666949207:AAH9Cgq8Bfh42g-aKk08nyNwDBS7R93QoI8/sendMessage"
}


define command {

        command_name    notify-service-by-telegram
        command_line    curl -X POST --data chat_id=-1001207542372 --data "text=***** Nagios ******%0A%0ANotification Type: $NOTIFICATIONTYPE$%0A%0AService: $SERVICEDESC$%0AHost: $HOSTALIAS$%0AURL:  $SERVICEDISPLAYNAME$%0AState: $SERVICESTATE$%0ADate/Time: $LONGDATETIME$%0A%0AAdditional Info:%0A$SERVICEOUTPUT$%0A" "https://api.telegram.org/bot666949207:AAH9Cgq8Bfh42g-aKk08nyNwDBS7R93QoI8/sendMessage"
}


#####################Pnp4nagios Configuration####################################

define command{
      command_name    process-service-perfdata-file
      command_line    /usr/local/pnp4nagios/libexec/process_perfdata.pl --bulk=/usr/local/pnp4nagios/var/service-perfdata

}

define command{
      command_name    process-host-perfdata-file
      command_line    /usr/local/pnp4nagios/libexec/process_perfdata.pl --bulk=/usr/local/pnp4nagios/var/host-perfdata

}


################################################################################
#
# SAMPLE SERVICE CHECK COMMANDS
#
# These are some example service check commands.  They may or may not work on
# your system, as they must be modified for your plugins.  See the HTML
# documentation on the plugins for examples of how to configure command definitions.
#
# NOTE:  The following 'check_local_...' functions are designed to monitor
#        various metrics on the host that Nagios is running on (i.e. this one).
################################################################################

define command {

    command_name    check_local_disk
    command_line    $USER1$/check_disk -w $ARG1$ -c $ARG2$ -p $ARG3$
}



define command {

    command_name    check_local_load
    command_line    $USER1$/check_load -w $ARG1$ -c $ARG2$
}



define command {

    command_name    check_local_procs
    command_line    $USER1$/check_procs -w $ARG1$ -c $ARG2$ -s $ARG3$
}



define command {

    command_name    check_local_users
    command_line    $USER1$/check_users -w $ARG1$ -c $ARG2$
}



define command {

    command_name    check_local_swap
    command_line    $USER1$/check_swap -w $ARG1$ -c $ARG2$
}



define command {

    command_name    check_local_mrtgtraf
    command_line    $USER1$/check_mrtgtraf -F $ARG1$ -a $ARG2$ -w $ARG3$ -c $ARG4$ -e $ARG5$
}



################################################################################
# NOTE:  The following 'check_...' commands are used to monitor services on
#        both local and remote hosts.
################################################################################

define command {

    command_name    check_ftp
    command_line    $USER1$/check_ftp -H $HOSTADDRESS$ $ARG1$
}



define command {

    command_name    check_hpjd
    command_line    $USER1$/check_hpjd -H $HOSTADDRESS$ $ARG1$
}



define command {

    command_name    check_snmp
    command_line    $USER1$/check_snmp -H $HOSTADDRESS$ $ARG1$
}



define command {

    command_name    check_http
    command_line    $USER1$/check_http -I $HOSTADDRESS$ $ARG1$
}



define command {

    command_name    check_ssh
    command_line    $USER1$/check_ssh $ARG1$ $HOSTADDRESS$
}



define command {

    command_name    check_dhcp
    command_line    $USER1$/check_dhcp $ARG1$
}



define command {

    command_name    check_ping
    command_line    $USER1$/check_ping -H $HOSTADDRESS$ -w $ARG1$ -c $ARG2$ -p 5
}



define command {

    command_name    check_pop
    command_line    $USER1$/check_pop -H $HOSTADDRESS$ $ARG1$
}



define command {

    command_name    check_imap
    command_line    $USER1$/check_imap -H $HOSTADDRESS$ $ARG1$
}



define command {

    command_name    check_smtp
    command_line    $USER1$/check_smtp -H $HOSTADDRESS$ $ARG1$
}



define command {

    command_name    check_tcp
    command_line    $USER1$/check_tcp -H $HOSTADDRESS$ -p $ARG1$ 
#$ARG2$
}


define command {

    command_name    check_udp
    command_line    $USER1$/check_udp -H $HOSTADDRESS$ -p $ARG1$ $ARG2$
}



define command {

    command_name    check_nt
    command_line    $USER1$/check_nt -H $HOSTADDRESS$ -p 12489 -v $ARG1$ $ARG2$
}



################################################################################
#
# SAMPLE PERFORMANCE DATA COMMANDS
#
# These are sample performance data commands that can be used to send performance
# data output to two text files (one for hosts, another for services).  If you
# plan on simply writing performance data out to a file, consider using the
# host_perfdata_file and service_perfdata_file options in the main config file.
#
################################################################################

define command {

    command_name    process-host-perfdata
    command_line    /usr/bin/printf "%b" "$LASTHOSTCHECK$\t$HOSTNAME$\t$HOSTSTATE$\t$HOSTATTEMPT$\t$HOSTSTATETYPE$\t$HOSTEXECUTIONTIME$\t$HOSTOUTPUT$\t$HOSTPERFDATA$\n" >> /usr/local/nagios/var/host-perfdata.out
}


define command {

    command_name    process-service-perfdata
    command_line    /usr/bin/printf "%b" "$LASTSERVICECHECK$\t$HOSTNAME$\t$SERVICEDESC$\t$SERVICESTATE$\t$SERVICEATTEMPT$\t$SERVICESTATETYPE$\t$SERVICEEXECUTIONTIME$\t$SERVICELATENCY$\t$SERVICEOUTPUT$\t$SERVICEPERFDATA$\n" >> /usr/local/nagios/var/service-perfdata.out
}

# 'check_nrpe' command definition
define command{
        command_name    check_nrpe
        command_line    $USER1$/check_nrpe -H $HOSTADDRESS$ -c $ARG1$
        }

# 'check_ping' command definition
define command{
        command_name    check-ping
        command_line    $USER1$/check_ping -H $HOSTADDRESS$ -w 100.0,20% -c 200.0,50% -p 3 -t 2
        }

# 'check_http' command definition
define command{
        command_name    check-weburl
        command_line    $USER1$/check_http -H $HOSTADDRESS$ $ARG1$ -w 5 -c 10
        }

# 'check_https' command definition
define command {
        command_name check-https
        command_line	$USER1$/check_http -S -I '$ARG1$' -u https://$ARG1$/health.php -w 5 -c 10
}

# 'check_https' command definition
define command {
        command_name check-https2
        command_line    $USER1$/check_http -S -I '$ARG1$' -u https://$ARG1$\/$ARG2$\/health -w 5 -c 10
}

# 'check_tcp' command definition
define command{
        command_name    check-tcp
        command_line    $USER1$/check_tcp -H $HOSTADDRESS$ -p $ARG1$ -w 0.2 -c 2
        }

```

### 一鍵安裝Nagios參考腳本
:::info
[Click Here](https://segmentfault.com/a/1190000014597362#articleHeader38)
:::

```gherkin=

```

:::info
**command.cfg**
/usr/local/nagios/etc/objects/command.cfg
:::

```gherkin=

```

## 安裝NagioSql

參考連結：
http://www.ttlsa.com/nagios/nagios-interface-management-configuration-tool-nagiosql/


問題排解 Q&A
---
### ./configure nrpe ………
>checking for Kerberos include files... configure: WARNING: could not find include files
checking for pkg-config... pkg-config
checking for SSL headers... configure: error: Cannot find ssl headers

解法：
> yum -y install openssl-devel


### Apache不執行PHP
修改APACHE的配置檔案 httpd.conf
在**<IfModule mime_module>**裡面加入：
>AddType application/x-httpd-php .php
AddType application/x-httpd-php-source .phps

就新增這些內容,然後重啟apache ,執行 apachectl -k graceful


>參考：https://www.itread01.com/p/1411440.html


## 參考文件

:::info
**[Centos-7下Nagios的安装及配置（完整版）](https://segmentfault.com/a/1190000014597362#articleHeader8)** 

**[Nagios 監控你的所有服務狀態 – Nginx 安裝 Nagios 4.1.1](https://shazi.info/nagios-%E7%9B%A3%E6%8E%A7%E4%BD%A0%E7%9A%84%E6%89%80%E6%9C%89%E6%9C%8D%E5%8B%99%E7%8B%80%E6%85%8B-nginx-%E5%AE%89%E8%A3%9D-nagios-4-1-1/)**

:::

###### tags: `CentOS7` `Nagios`
