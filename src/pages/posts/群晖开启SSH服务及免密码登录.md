---
layout: '../../layouts/MarkdownPost.astro'
title: '群晖开启SSH及免密码登录配置'
pubDate: 2023-05-27
description: '群晖使用'
author: 'ZhJy'
cover:
    url: 'https://cdn.jsdelivr.net/gh/conscloud/picgotemp/imgplus/202305270954319.png'
    square: 'https://cdn.jsdelivr.net/gh/conscloud/picgotemp/imgplus/202305270954319.png'
    alt: 'cover'
tags: ["群晖"] 
theme: 'light'
featured: true

meta:
 - name: author
   content: ZhJy
 - name: keywords
   content: key3, key4

keywords: 群晖, DSM, SSH!
---
### 前言

![](https://cdn.jsdelivr.net/gh/conscloud/picgotemp/imgplus/202305270954319.png)

群晖开启root账户免密登录与linux服务器的操作大致相同。 

我的群晖DSM版本是7.1.1

<img src="https://cdn.jsdelivr.net/gh/conscloud/picgotemp/imgplus/202305270925295.png" alt="image-20230527092511230" style="zoom:50%;" />

### 1.开启SSH服务

群晖从7开始默认关闭了“admin"账户，并禁用最大权限的系统账户“root”登录网页控制台。

先使用群晖安装时建立的普通管理员账户（加入了administrators用户组的用户）登录web控制台后，依次点击“控制面板”-“终端机和SNMP”，勾选“启用SSH功能”，再点击右下角的“应用”按钮即完成开启SSH服务。**建议修改默认端口号。**

<img src="https://cdn.jsdelivr.net/gh/conscloud/picgotemp/imgplus/202305270928199.png" alt="image-20230527092800109" style="zoom:67%;" />

### 2.允许ROOT账号登录

通过普通管理员账户进行ssh登录，输入sudo - i 回车后再次输入管理员密码，就能切换到root账户。

<img src="https://cdn.jsdelivr.net/gh/conscloud/picgotemp/imgplus/202305270932547.png" alt="image-20230527093234465" style="zoom:50%;" />

给root账户设置密码,其中xxx为你想要设置的密码。

```shell
synouser --setpw root xxx
```

修改sshd_config文件的内容：

```bash
PermitRootLogin yes					#允许root登录
PasswordAuthentication yes          #开启密码认证
ChallengeResponseAuthentication no
UsePAM yes						    #开户密码认证
```

修改完后，别忘记保存。

### 3.开启密钥登录

确认在root用户下，输入“ssh-keygen”命令创建密钥，id\_rsa是新生成的私钥，id\_rsa.pub是对应的公钥

```shell
ssh-keygen -t rsa -b 2048 -C "root_rsa_key"
```

将id_rsa.pub文件内容追加到“/root/.ssh/authorized_keys”文件中

```shell
cat /etc/ssh/id_rsa.pub >> /root/.ssh/authorized_keys
```

*注意authorized_keys权限至少是root账户有rw（否则执行以下2条命令“chmod 700 ~/.ssh”、“ chmod 600 ~/.ssh/authorized_keys”）*

将id_rsa文件复制到本地，重启ssh服务

```shell
synosystemctl reload sshd
synosystemctl restart sshd
```

在本地就可以通过工具（如xshell等）免密码连接到群晖了。

最后可以修改sshd_config文件，禁止使用密码认证登录，提高安全性

```bash
PasswordAuthentication no          #关闭密码认证
```

**最后，如果修改sshd_config文件导致ssh功能无法使用而其他功能正常，群晖可以正常登录网页控制台，可以开启telnet,把错误的sshd_config改回去！**

### 附录

**群晖353条syno命令:**

```bash
syno-abuser-blocklist-checker
syno-dump-core.sh
syno-init-disk-health-db
syno-letsencrypt
syno8021Xtool
synoRTCTime
syno_adv_test
syno_bios_bootperf_record
syno_disk_config_check
syno_disk_ctl
syno_disk_data_collector
syno_disk_db_update
syno_disk_dsl
syno_disk_dump
syno_disk_firm_status_update
syno_disk_health_predict
syno_disk_health_record
syno_disk_latency_collector
syno_disk_log_convert
syno_disk_log_import_from_xml
syno_disk_performance_cache_update
syno_disk_performance_delete_record
syno_disk_performance_monitor
syno_disk_schedule_test
syno_disk_smart_mail_send
syno_disk_sysfs_get
syno_disk_sysfs_set
syno_disk_test_log_import_from_xml
syno_disk_test_scheduler_set
syno_disk_testlog_convert
syno_disk_wcache_config_init
syno_disk_wdda
syno_drive_bundle
syno_ew_check.sh
syno_expansion
syno_fan_debug
syno_hdd_util
syno_hibernation_debug
syno_hook_trgr
syno_hw_video_transcoding.sh
syno_ip_conflict_detect
syno_iptables_common
syno_led_brightness
syno_led_mask_on
syno_mem_check
syno_mem_single_channel_action
syno_mib_disk_mapping
syno_mib_disk_tool
syno_microp_control
syno_predict_disk_health
syno_pstore_collect
syno_scemd_connector
syno_sched_poweroff
syno_smart_result_collect
syno_smart_test
syno_spectre_meltdown_tool
syno_ssd_trim
syno_ssd_trim_schedule
syno_storage_bkgrd_task
syno_swap_ctl
syno_syslog_check_ctl
syno_system_dump
syno_update_disk_log_information
syno_user_info
syno_volume_analyze
synoabnormalloginmail
synoabnormalloginnotify
synoacltool
synoafp
synoagentregisterd
synoagentregistertool
synoappbkp
synoappconfigcache
synoappnotify
synoapppriv_updater
synoarchive
synoarchivetool
synoauth
synoautoblock
synoautonano
synobackgroundtask
synobackup
synobackupd
synobandwidth
synobios_uninit
synobootseq
synobootupcheck
synobtrfssnap
synobtrfssnapusage
synocacheadvisor
synocacheadvisord
synocacheclient
synocachepinfiled
synocachepinfiletool
synocachepinfiletool-status
synocachepinfiletool.sh
synocfgen
synocgid
synocgitool
synocheckgroup
synocheckhotspare
synocheckinfo
synocheckiscsitrg
synochecknetworkcfg
synocheckshare
synocheckuser
synocleaner
synocloudserviceauth
synocmsclient
synocodectool
synoconfbkp
synoconfd
synocontentextract
synocontentextractd
synocontentsearchutils
synocopy
synocredential
synocrond
synocrtchecksum
synocrtregister
synocrtunregister
synocsp
synodatacollect
synodataverifier
synodate
synodbudconfig
synodbudd
synodbudgetinfo
synodbudinfo
synodbudisrunning
synodbudupdate
synodbudvcdiff
synodbudvolume
synodctest
synodd
synoddnsinfo
synodisk
synodiskdatacollect
synodiskfind
synodisklatencyd
synodiskmanagertool
synodiskpathparse
synodiskport
synodiskstat
synodiskwddad
synodsdefault
synodsinfo
synodsmloginhealthcheck
synodsmnotify
synoethinfo
synoexternal
synoextractjep
synofanconfig
synofilehandle
synofilehandlecleancache
synofileutil
synofirewall
synofirewallUpdater
synoflashcache
synoflashcachechecknotifymissing
synoflashcacheshareapplytool
synoflvconv
synofsbdctl
synofstool
synoftpchecker
synogear
synogetkeyvalue
synogetstate.sh
synogpoclientd
synogrinst
synogroup
synohacore
synohtmlhandler
synohwctl
synoindex
synoindex-bin-scheduler
synoindex-bin-sdk-hook-db-tool
synoindex_mgr
synoindex_package.sh
synoindexd
synoindexnotifyd
synoindexplugind
synoindexscand
synoindexworkerd
synoinsid
synoiscsiep
synoiscsitop
synoiscsiwebapi
synokerneltz
synolanstatus
synoldapclient
synoldapclientd
synolegalnotifier
synolog-linker
synologaccd
synologand
synologanutil
synologconfgen
synologconvert
synologrotated
synologset
synologset1
synomediaparser
synomediaparserd
synomibclient-event
synomibtool
synomigratewallpaper
synomkflv
synomkflvd
synomkthumb
synomkthumbd
synomoduletool
synomount
synomustache
synomyds
synonclient_send
synonet
synonetd
synonetdtool
synoneteventd
synonetseqadj
synonfs
synonfstest
synonfstop
synonode
synonotify
synonotifyconvert
synonotifydbtransfer
synonvme
synootp
synoovstool
synopartition
synopasswordmail
synopayment
synoperfeventd
synoperformancediagnose
synopersonalupdater
synopftest
synopingpong
synopkg
synopkgctl
synopkghelper
synopkicompatsync
synoplatform
synoportforward
synopostgres
synopoweroff
synopppoe
synopreferencedir
synoprint
synopsql
synopyntlmd
synoquota
synoraid5stat
synoraidtool
synorbdctl
synorecycle
synorelayd
synorenewdefaultcert
synoretainer
synoretentionconf
synoretentiontest
synoretentiontestutil.sh
synorollinggroupid
synorouterportfwd
synoroutertool
synorsyncdtool
synosavetime
synoscgi
synoscgi-socket-get-memory.js
synoscgi________________________________________________________
synoscgi_socket.js
synoschedmulti
synoschedmultirun
synoschedtask
synoschedtool
synoscheduled
synosdutils
synosearch
synosearchagent
synoselfcheck
synoselfcheck-min
synoservicemigrate
synosetkeyvalue
synosetnoatime
synoshare
synosharequota
synosharesnapshot
synosharesnaptool
synosharesnaptree
synosharesnaptree_reconstruct.sh
synosharingbackup
synosharingchecker
synosharingcron
synosharingurl
synoshortcutmigrate.min.js
synoshutdown
synosmartblock
synosnapschedtask.sh
synosnmp_communicator
synosnmpcd
synosnmpcd_db_updater
synosocket
synospace
synospace.sh
synossdbundlehotplug
synossdbundlemonitor
synosshdutils
synostgdisk
synostgpool
synostgreclaim
synostgsysraid
synostgtask
synostgvolume
synostorage
synostoragecore
synostoraged
synosubvoltype
synosupportchannelchecker
synosyncdctime
synosyslogcheck
synosyslogmail
synosystemctl
synotaskmgr
synotc
synotc_common
synothumb
synotifyd
synotifydutil
synotimecontrol
synotlstool
synotokenmgr
synotune
synoupgrade
synoupgradepreserve
synoupnp
synoups
synoups_battery_notify.sh
synoupscommon
synousb
synousbdisk
synouser
synouserdir
synouserhome
synovolumesnapshot
synovpnc
synovspace
synovspace_wrapper
synow3
synow3tool
synowebapi
synowedjat-exec
synowin
```