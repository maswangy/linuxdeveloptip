# Tips for Linux kernel & driver development

strip modules when linux kernel installs modules
```
# make INSTALL_MOD_STRIP=1 modules_install
```

checkpatch.pl usage
* show type of issue: --show-types
* show only specified types: --types
```
$ ./scripts/checkpatch.pl --file --show-types drivers/staging/greybus/*.c
----------------------------------------
drivers/staging/greybus/arche-apb-ctrl.c
----------------------------------------
CHECK:LINE_SPACING: Please don't use multiple blank lines
#24: FILE: drivers/staging/greybus/arche-apb-ctrl.c:24:
+
+

CHECK:PARENTHESIS_ALIGNMENT: Alignment should match open parenthesis
#74: FILE: drivers/staging/greybus/arche-apb-ctrl.c:74:
+	if (apb->init_disabled ||
+			apb->state == ARCHE_PLATFORM_STATE_ACTIVE)

$./scripts/checkpatch.pl --file --types="LONG_LINE" drivers/staging/greybus/*.c | less
```

change keyboard layout for console
```
try...
# sudo dpkg-reconfigure console-setup
# dpkg-reconfigure keyboard-configuration
# reboot
```

checkout a remote branch
```
git fetch
git checkout test
```
But there are multiple remote, it doesn't download the branch, then:
```
git checkout -b test <name of remote>/test
```

check the coding style of external driver source file with checkpatch.pl
* --file: check regular file, not patch
* --no-tree: no kernel source tree
* --help prints other options
```
gohkim@ws00837:~/study/little_challenge/task05$ ~/work/linux-torvalds/scripts/checkpatch.pl --file --no-tree drv.c
WARNING: please, no spaces at the start of a line
#25: FILE: drv.c:25:
+    printk(KERN_EMERG "usb_task05_probe\n");$
...
```

build a debian package from source
```
$ dpkb-buildpackage -d
-d: 의존성/충돌 체크 안함
-b: 소스 체크 안함 
```

change console keyboard layout: ``sudo apt-get install console-common``

git command for prerry: ``git log --pretty=format:"%h %ad%x09%s"``

get guid of dm-13: ``pbkvm dm2guid dm-13``

fio: generate IOs with various types, size, interval and so on
```
[global]
description=Emulation of Storage Server Access Pattern
bssplit=512/20:1k/16:2k/9:4k/12:8k/19:16k/10:32k/8:64k/4
fadvise_hint=0
#rw=randrw:2
rw=write
direct=1

ioengine=libaio
iodepth=64
iodepth_batch_submit=64
iodepth_batch_complete=64
numjobs=4
gtod_reduce=1
group_reporting=1

time_based=1
runtime=30

[job]
filename=./test1
```

network stress test
```
netperf -H 10.66.83.169 -l 600
iperf with multithreads
iperf -c <IP> -P 16 -t 3600
```

slub debugging detail: https://www.kernel.org/doc/Documentation/vm/slub.txt

SCST: http://scst.sourceforge.net/

iostat
* https://www.kernel.org/doc/Documentation/block/stat.txt
* https://www.kernel.org/doc/Documentation/iostats.txt
* ``dstat -df``: check amount of read/write for block device at each second

pkill: kill process with name
* ``sleep $((60*10)) & pkill mprime`` -> kill mprime after 60-min

pgrep: list processes of specific user
* ``pgrep -u gohkim -l`` -> print pid of gohkim as weel as command

flush buffer of disks: ``blockdev --flushbufs``

perf report cannot print the kernel symbols even-if the kernel is built with CONFIG_KALLSYMS and CONFIG_KALLSYMS_ALL
```
gohkim@ws00837:~/hdd/intel-iommu-perf$ ./perf report -k /proc/kallsyms 
Warning:
Kernel address maps (/proc/{kallsyms,modules}) were restricted.

Check /proc/sys/kernel/kptr_restrict before running 'perf record'.

gohkim@ws00837:~/hdd/intel-iommu-perf$ cat /proc/sys/kernel/kptr_restrict 
1

root@ws00837:/usr/src/linux-source-4.2.0/linux-source-4.2.0# echo 0 > /proc/sys/kernel/kptr_restrict 

gohkim@ws00837:~/hdd/intel-iommu-perf$ cat /proc/sys/kernel/kptr_restrict 
0
```

How to develop against linux-next: https://lwn.net/Articles/289245/

check the event-log of Megaraid device
* ``$ sudo storcli /c0 show events``

run shell command via ssh
* ``$ ssh systemsboy@rhost.systemsboy.edu 'ls -l; ps -aux; whoami'``

gvncviewer
* port: ``ps aux | grep kvm | grep <job-name>``  ===> -vnc 0.0.0.0:50  ===> 50 is port number
* ``$ gvncviewer <pserver-ip>:<port>``

linux-next 업데이트
```
$git remote update
$git reset --hard origin/master
```

패치 파일 생성
* ``git format-patch -3 -s --cover-letter --subject-prefix=PATCHv2``
* -3: 가장 최신 커밋부터 3개까지
* --cover-letter옵션: 패치 파일 통계 등 자동 생성됨
* --subject-prefix: 제목에 PATCH대신 들어갈 말머리
* -s: signed off by를 자동으로 추가)

패치파일 전송
* -–thread --no-chain-reply-to 0000-cover-letter를 가장 먼저 전송하고 reply로 0001~0002로 전송함
```
git send-email --to=gioh.kim@lge.com --thread --no-chain-reply-to 0000-cover-letter.patch 0001-patch-name.patch 0002-2nd.patch
```
여러 patch 파일들의 통계 : diffstat 패치파일이름

```
sudo echo 3 > /proc/sys/vm/drop_caches는 permission denied가 발생한다.
이렇게 할것
echo 1 | sudo tee /proc/sys/vm/drop_caches
```

defconfig 파일 만드는 방법: make savedefconfig

Howto cherry-pick patch from mainline kernel:
```
$ git remote add mainline git://git.kernel.org/pub/scm/linux/kernel/git/torvalds/linux.git
$ git fetch mainline
```
Then cherry pick the commits you need, which for this Haswell HDMI audio bug meant these three commits (as best I can tell):
```
$ git cherry-pick 9419ab6b72325e20789a61004cf68dc9e909a009
$ git cherry-pick c88d4e84e639df9a9640ecff71de2501a84d1f48
$ git cherry-pick 17df3f55652f7ea8fb1197b5c32e227b3da9f215
```

브랜치를 만들었는데 remote에 있는 브랜치랑 연결이 안될때
```
git branch --set-upstream-to=origin/feature/jessie-builds-3.3.2-5pb6-storage  feature/jessie-builds-3.3.2-5pb6-storage
```

awk pattern maching example
* ``'/<pattern>/{<action>}'``
```
gohkim@ws00837:~$ cat /proc/meminfo | awk '/MemFree/{print $0}'
MemFree:         1458512 kB
gohkim@ws00837:~$ cat /proc/meminfo | awk '/MemFree/{print $1}'
MemFree:
gohkim@ws00837:~$ cat /proc/meminfo | awk '/MemFree/{print $2}'
1458528
gohkim@ws00837:~$ cat /proc/meminfo | awk '/MemFree/{printf "%d\n", $2 * 0.9 / 64 ;}'
27592
```

gitconfig: pretty log printing
```
gohkim@ws00837:~/work/linux-stable$ cat ~/.gitconfig
...

[alias]
    lg1 = log --graph --abbrev-commit --decorate --format=format:'%C(bold blue)%h%C(reset) - %C(bold green)(%ar)%C(reset) %C(white)%s%C(reset) %C(dim white)- %an%C(reset)%C(bold yellow)%d%C(reset)'
    lg2 = log --graph --abbrev-commit --decorate --format=format:'%C(bold blue)%h%C(reset) - %C(bold cyan)%aD%C(reset) %C(bold green)(%ar)%C(reset)%C(bold yellow)%d%C(reset)%n''          %C(white)%s%C(reset) %C(dim white)- %an%C(reset)' --all
    lg = !"git lg1"
```

debug udevd: show what rules are applied
```
sudo pkill udevd
sudo udevd --debug-trace --verbose --suppress-syslog
```
