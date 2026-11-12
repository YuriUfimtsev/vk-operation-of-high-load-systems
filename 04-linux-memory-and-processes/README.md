# Домашнее задание 3. Linux. Архитектура и файловые системы

Дополнительные пакеты:

```bash
sudo apt install -y strace lvm2 xfsprogs parted

(после успешно пройденной установки и повторного выполнения команды)

yuri@ubuntu-hp:~$ sudo apt install -y strace lvm2 xfsprogs parted
Reading package lists... Done
Building dependency tree... Done
Reading state information... Done
parted is already the newest version (3.4-2build1).
strace is already the newest version (5.16-0ubuntu3).
lvm2 is already the newest version (2.03.11-2.1ubuntu5).
xfsprogs is already the newest version (5.13.0-1ubuntu2.1).
The following packages were automatically installed and are no longer required:
  libbrotli-dev libfontconfig-dev libfontconfig1-dev libfreetype-dev
  libfreetype6-dev libpng-dev libpng-tools libsdl-ttf2.0-0 libxkbcommon-dev
  python3-docker python3-dockerpty python3-docopt python3-dotenv
  python3-jsonschema python3-pyrsistent python3-texttable python3-websocket
Use 'sudo apt autoremove' to remove them.
0 upgraded, 0 newly installed, 0 to remove and 176 not upgraded.
```

## Задание 1. Kernel and Module Inspection

### Шаг 1.1. Версия ядра

```bash
yuri@ubuntu-hp:~$ uname -r
6.8.0-138-generic

yuri@ubuntu-hp:~$ uname -a
Linux ubuntu-hp 6.8.0-138-generic #138~22.04.1-Ubuntu SMP PREEMPT_DYNAMIC Fri Aug  7 13:43:15 UTC  x86_64 x86_64 x86_64 GNU/Linux

yuri@ubuntu-hp:~$ cat /proc/version
Linux version 6.8.0-138-generic (buildd@lcy02-amd64-008) (x86_64-linux-gnu-gcc-12 (Ubuntu 12.3.0-1ubuntu1~22.04.3) 12.3.0, GNU ld (GNU Binutils for Ubuntu) 2.38) #138~22.04.1-Ubuntu SMP PREEMPT_DYNAMIC Fri Aug  7 13:43:15 UTC 
```

`uname -r` показывает релиз ядра, `uname -a` дополнительно архитектуру и дату сборки, `/proc/version` также версию компилятора, которым собрано ядро.

### Шаг 1.2. Загруженные модули

```bash
yuri@ubuntu-hp:~$ lsmod
Module                  Size  Used by
xt_conntrack           12288  3
nft_chain_nat          12288  3
xt_MASQUERADE          16384  3
nf_nat                 61440  2 nft_chain_nat,xt_MASQUERADE
nf_conntrack_netlink    57344  0
nf_conntrack          200704  4 xt_conntrack,nf_nat,nf_conntrack_netlink,xt_MASQUERADE
nf_defrag_ipv6         24576  1 nf_conntrack
nf_defrag_ipv4         12288  1 nf_conntrack
xfrm_user              61440  1
xfrm_algo              16384  1 xfrm_user
xt_addrtype            12288  2
nft_compat             20480  8
nf_tables             380928  136 nft_compat,nft_chain_nat
libcrc32c              12288  3 nf_conntrack,nf_nat,nf_tables
nfnetlink              20480  4 nft_compat,nf_conntrack_netlink,nf_tables
br_netfilter           32768  0
bridge                425984  1 br_netfilter
stp                    12288  1 bridge
llc                    16384  2 bridge,stp
ccm                    20480  6
rfcomm                102400  4
snd_ctl_led            24576  0
ledtrig_audio          12288  1 snd_ctl_led
snd_soc_skl_hda_dsp    24576  6
snd_soc_hdac_hdmi      49152  1 snd_soc_skl_hda_dsp
snd_soc_intel_hda_dsp_common    16384  1 snd_soc_skl_hda_dsp
snd_sof_probes         20480  0
snd_hda_codec_realtek   212992  1
snd_hda_codec_generic   122880  1 snd_hda_codec_realtek
snd_soc_dmic           12288  1
snd_sof_pci_intel_tgl    12288  0
snd_sof_intel_hda_common   217088  1 snd_sof_pci_intel_tgl
soundwire_intel        73728  1 snd_sof_intel_hda_common
snd_sof_intel_hda_mlink    45056  2 soundwire_intel,snd_sof_intel_hda_common
soundwire_cadence      45056  1 soundwire_intel
snd_sof_intel_hda      24576  1 snd_sof_intel_hda_common
snd_hda_codec_hdmi     94208  1
snd_sof_pci            24576  2 snd_sof_intel_hda_common,snd_sof_pci_intel_tgl
snd_sof_xtensa_dsp     12288  1 snd_sof_intel_hda_common
snd_sof               380928  4 snd_sof_pci,snd_sof_intel_hda_common,snd_sof_probes,snd_sof_intel_hda
xe                   2736128  0
snd_sof_utils          16384  1 snd_sof
snd_soc_hdac_hda       24576  1 snd_sof_intel_hda_common
snd_hda_ext_core       32768  5 snd_sof_intel_hda_common,snd_soc_hdac_hdmi,snd_soc_hdac_hda,snd_sof_intel_hda_mlink,snd_sof_intel_hda
snd_soc_acpi_intel_match    98304  2 snd_sof_intel_hda_common,snd_sof_pci_intel_tgl
cmac                   12288  2
snd_soc_acpi           16384  2 snd_soc_acpi_intel_match,snd_sof_intel_hda_common
algif_hash             12288  1
soundwire_generic_allocation    12288  1 soundwire_intel
soundwire_bus         110592  3 soundwire_intel,soundwire_generic_allocation,soundwire_cadence
algif_skcipher         12288  1
overlay               212992  0
af_alg                 32768  6 algif_hash,algif_skcipher
snd_soc_core          442368  8 soundwire_intel,snd_sof,snd_sof_intel_hda_common,snd_soc_hdac_hdmi,snd_soc_hdac_hda,snd_sof_probes,snd_soc_dmic,snd_soc_skl_hda_dsp
bnep                   32768  2
drm_gpuvm              45056  1 xe
drm_exec               12288  2 drm_gpuvm,xe
gpu_sched              61440  1 xe
snd_compress           28672  2 snd_soc_core,snd_sof_probes
drm_suballoc_helper    20480  1 xe
ac97_bus               12288  1 snd_soc_core
intel_uncore_frequency    16384  0
drm_ttm_helper         12288  1 xe
intel_uncore_frequency_common    16384  1 intel_uncore_frequency
snd_pcm_dmaengine      16384  1 snd_soc_core
x86_pkg_temp_thermal    20480  0
snd_hda_intel          61440  0
intel_powerclamp       24576  0
snd_intel_dspcfg       36864  3 snd_hda_intel,snd_sof,snd_sof_intel_hda_common
coretemp               24576  0
snd_intel_sdw_acpi     16384  2 snd_sof_intel_hda_common,snd_intel_dspcfg
snd_hda_codec         204800  8 snd_hda_codec_generic,snd_hda_codec_hdmi,snd_hda_intel,snd_hda_codec_realtek,snd_soc_intel_hda_dsp_common,snd_soc_hdac_hda,snd_sof_intel_hda,snd_soc_skl_hda_dsp
kvm_intel             491520  0
snd_hda_core          139264  11 snd_hda_codec_generic,snd_hda_codec_hdmi,snd_hda_intel,snd_hda_ext_core,snd_hda_codec,snd_hda_codec_realtek,snd_soc_intel_hda_dsp_common,snd_sof_intel_hda_common,snd_soc_hdac_hdmi,snd_soc_hdac_hda,snd_sof_intel_hda
joydev                 32768  0
kvm                  1409024  1 kvm_intel
snd_hwdep              20480  1 snd_hda_codec
rtw89_8852ae           12288  0
i915                 4300800  28
irqbypass              12288  1 kvm
rtw89_8852a           724992  1 rtw89_8852ae
snd_pcm               192512  12 snd_hda_codec_hdmi,snd_hda_intel,snd_hda_codec,soundwire_intel,snd_sof,snd_sof_intel_hda_common,snd_soc_hdac_hdmi,snd_compress,snd_soc_core,snd_sof_utils,snd_hda_core,snd_pcm_dmaengine
crct10dif_pclmul       12288  1
polyval_clmulni        12288  0
uvcvideo              147456  0
polyval_generic        12288  1 polyval_clmulni
rtw89_pci              81920  1 rtw89_8852ae
videobuf2_vmalloc      20480  1 uvcvideo
btusb                  81920  0
ghash_clmulni_intel    16384  0
btrtl                  32768  1 btusb
snd_seq_midi           24576  0
sha256_ssse3           32768  0
uvc                    12288  1 uvcvideo
snd_seq_midi_event     16384  1 snd_seq_midi
btintel                57344  1 btusb
rtw89_core            737280  2 rtw89_pci,rtw89_8852a
sha1_ssse3             32768  0
videobuf2_memops       16384  1 videobuf2_vmalloc
processor_thermal_device_pci_legacy    12288  0
btbcm                  24576  1 btusb
snd_rawmidi            57344  1 snd_seq_midi
aesni_intel           356352  7
videobuf2_v4l2         36864  1 uvcvideo
processor_thermal_device    20480  1 processor_thermal_device_pci_legacy
btmtk                  12288  1 btusb
intel_rapl_msr         20480  0
mei_pxp                16384  0
mei_hdcp               28672  0
videodev              356352  2 videobuf2_v4l2,uvcvideo
binfmt_misc            24576  1
crypto_simd            16384  1 aesni_intel
processor_thermal_wt_hint    16384  1 processor_thermal_device
hid_sensor_accel_3d    16384  1
hid_sensor_gyro_3d     16384  0
drm_buddy              20480  2 xe,i915
hid_sensor_rotation    16384  0
hid_sensor_custom_intel_hinge    12288  0
bluetooth            1044480  34 btrtl,btmtk,btintel,btbcm,bnep,btusb,rfcomm
snd_seq               122880  2 snd_seq_midi,snd_seq_midi_event
mac80211             1753088  2 rtw89_core,rtw89_pci
cryptd                 24576  3 crypto_simd,ghash_clmulni_intel
processor_thermal_rfim    32768  1 processor_thermal_device
ttm                   110592  3 drm_ttm_helper,xe,i915
videobuf2_common       81920  4 videobuf2_vmalloc,videobuf2_v4l2,uvcvideo,videobuf2_memops
hid_sensor_trigger     20480  9 hid_sensor_gyro_3d,hid_sensor_custom_intel_hinge,hid_sensor_accel_3d,hid_sensor_rotation
snd_seq_device         16384  3 snd_seq,snd_seq_midi,snd_rawmidi
ecdh_generic           16384  1 bluetooth
nls_iso8859_1          12288  1
cmdlinepart            12288  0
rapl                   20480  0
hp_wmi                 36864  0
processor_thermal_rapl    16384  1 processor_thermal_device
hp_bioscfg             81920  0
intel_cstate           24576  0
industrialio_triggered_buffer    12288  1 hid_sensor_trigger
drm_display_helper    237568  2 xe,i915
snd_timer              49152  2 snd_seq,snd_pcm
mc                     81920  4 videodev,videobuf2_v4l2,uvcvideo,videobuf2_common
ecc                    45056  1 ecdh_generic
input_leds             12288  0
spi_nor               167936  0
cfg80211             1368064  3 rtw89_core,mac80211,rtw89_8852a
serio_raw              20480  0
platform_profile       12288  1 hp_wmi
intel_rapl_common      40960  2 intel_rapl_msr,processor_thermal_rapl
firmware_attributes_class    12288  1 hp_bioscfg
ee1004                 16384  0
hid_multitouch         36864  0
kfifo_buf              12288  1 industrialio_triggered_buffer
wmi_bmof               12288  0
cec                    94208  3 drm_display_helper,xe,i915
snd                   143360  28 snd_ctl_led,snd_hda_codec_generic,snd_seq,snd_seq_device,snd_hda_codec_hdmi,snd_hwdep,snd_hda_intel,snd_hda_codec,snd_hda_codec_realtek,snd_sof,snd_timer,snd_soc_hdac_hdmi,snd_compress,snd_soc_core,snd_pcm,snd_rawmidi
mtd                    98304  3 spi_nor,cmdlinepart
mei_me                 53248  2
hid_sensor_iio_common    24576  5 hid_sensor_gyro_3d,hid_sensor_trigger,hid_sensor_custom_intel_hinge,hid_sensor_accel_3d,hid_sensor_rotation
processor_thermal_wt_req    12288  1 processor_thermal_device
rc_core                73728  1 cec
industrialio          126976  8 industrialio_triggered_buffer,hid_sensor_gyro_3d,hid_sensor_trigger,hid_sensor_custom_intel_hinge,kfifo_buf,hid_sensor_accel_3d,hid_sensor_rotation
processor_thermal_power_floor    12288  1 processor_thermal_device
soundcore              16384  2 snd_ctl_led,snd
libarc4                12288  1 mac80211
mei                   172032  5 mei_hdcp,mei_pxp,mei_me
i2c_algo_bit           16384  2 xe,i915
processor_thermal_mbox    12288  4 processor_thermal_power_floor,processor_thermal_wt_req,processor_thermal_rfim,processor_thermal_wt_hint
igen6_edac             24576  0
intel_soc_dts_iosf     20480  1 processor_thermal_device_pci_legacy
intel_pmc_core        118784  0
int3403_thermal        16384  0
soc_button_array       20480  0
intel_vsec             20480  1 intel_pmc_core
int340x_thermal_zone    16384  2 int3403_thermal,processor_thermal_device
pmt_telemetry          16384  1 intel_pmc_core
intel_hid              32768  0
int3400_thermal        24576  0
pmt_class              12288  1 pmt_telemetry
acpi_thermal_rel       20480  1 int3400_thermal
sparse_keymap          12288  2 hp_wmi,intel_hid
acpi_pad              184320  0
mac_hid                12288  0
sch_fq_codel           24576  2
msr                    12288  0
parport_pc             53248  0
ppdev                  24576  0
lp                     28672  0
parport                73728  3 parport_pc,lp,ppdev
efi_pstore             12288  0
ip_tables              32768  0
x_tables               65536  5 xt_conntrack,nft_compat,xt_addrtype,ip_tables,xt_MASQUERADE
autofs4                57344  2
hid_sensor_custom      28672  0
r8153_ecm              12288  0
cdc_ether              24576  1 r8153_ecm
hid_sensor_hub         28672  7 hid_sensor_gyro_3d,hid_sensor_trigger,hid_sensor_iio_common,hid_sensor_custom_intel_hinge,hid_sensor_accel_3d,hid_sensor_rotation,hid_sensor_custom
usbnet                 61440  2 r8153_ecm,cdc_ether
intel_ishtp_hid        32768  0
r8152                 143360  1 r8153_ecm
mii                    20480  2 usbnet,r8152
nvme                   61440  2
nvme_core             212992  3 nvme
nvme_auth              28672  1 nvme_core
hid_generic            12288  0
rtsx_pci_sdmmc         36864  0
intel_lpss_pci         28672  0
spi_intel_pci          12288  0
i2c_i801               36864  0
intel_ish_ipc          32768  0
intel_lpss             12288  1 intel_lpss_pci
i2c_hid_acpi           12288  0
crc32_pclmul           12288  0
ucsi_acpi              12288  0
xhci_pci               24576  0
typec_ucsi             61440  1 ucsi_acpi
spi_intel              32768  1 spi_intel_pci
i2c_smbus              16384  1 i2c_i801
rtsx_pci              143360  1 rtsx_pci_sdmmc
intel_ishtp            69632  2 intel_ishtp_hid,intel_ish_ipc
idma64                 20480  0
vmd                    24576  0
video                  77824  2 xe,i915
i2c_hid                40960  1 i2c_hid_acpi
xhci_pci_renesas       20480  1 xhci_pci
typec                 106496  1 typec_ucsi
hid                   180224  5 i2c_hid,hid_multitouch,hid_sensor_hub,intel_ishtp_hid,hid_generic
wmi                    28672  4 hp_wmi,video,hp_bioscfg,wmi_bmof
pinctrl_tigerlake      28672  2
```

```bash
yuri@ubuntu-hp:~$ lsmod | tail -n +2 | wc -l
226
```

Всего загружено 226 модулей. Столбцы: имя модуля, размер в памяти, счётчик использования и модули, которые от него зависят.

### Шаг 1.3. Отключение автозагрузки cdrom

Проверим, собран ли cdrom модулем и загружен ли он сейчас:

```bash
yuri@ubuntu-hp:~$ grep -E 'CONFIG_CDROM=|CONFIG_BLK_DEV_SR=' /boot/config-$(uname -r)
CONFIG_CDROM=y
CONFIG_BLK_DEV_SR=y

yuri@ubuntu-hp:~$ lsmod | grep -E 'cdrom|sr_mod'

yuri@ubuntu-hp:~$ modinfo cdrom | head -n 5
name:           cdrom
filename:       (builtin)
license:        GPL
file:           drivers/cdrom/cdrom
parm:           debug:bool

yuri@ubuntu-hp:~$ modinfo -n sr_mod
(builtin)
```

`CONFIG_CDROM=y`, `CONFIG_BLK_DEV_SR=y` и `filename: (builtin)`, то есть на этом ядре (6.8.0-138-generic) cdrom и драйвер CD-приводов sr_mod встроены в ядро, не собраны загружаемыми модулями. Поэтому их нет в `lsmod`, и выгрузить их нельзя:
```bash
yuri@ubuntu-hp:~$ sudo modprobe -r cdrom
modprobe: FATAL: Module cdrom is builtin.
```

Запрещаем автозагрузку через modprobe.d:

```bash
yuri@ubuntu-hp:~$ sudo tee /etc/modprobe.d/blacklist-cdrom.conf > /dev/null <<'EOF'
blacklist cdrom
blacklist sr_mod
install cdrom /bin/false
EOF

yuri@ubuntu-hp:~$ cat /etc/modprobe.d/blacklist-cdrom.conf
blacklist cdrom
blacklist sr_mod
install cdrom /bin/false

Если бы модули были не встроенными, мы бы ещё выполнили update-initramfs -u -- то есть пересобрали образ с модулями, который ядро загружает на самом раннем этапе. Он нужен, чтобы blacklist подействовал и на модули, которые грузятся оттуда, ещё до основной системы. Но cdrom и sr_mod встроены в само ядро, в initramfs их нет как отдельных модулей, поэтому пересборка ничего не изменит
```

* `blacklist` запрещает автоматическую загрузку модуля по alias (при обнаружении устройства).
* `blacklist` не мешает загрузить модуль как зависимость другого, поэтому добавлен `install cdrom /bin/false` -- вместо загрузки модуля modprobe выполнит `/bin/false`.

`blacklist`  действует только на загружаемые модули (файлы `.ko` -- kernel object), а встроенный код стартует вместе с ядром, modprobe в этом не участвует. Поэтому на этом ядре файл ни на что не влияет -- он сработает на ядре, где cdrom собран модулем (`CONFIG_CDROM=m`).

Полностью отключить встроенный cdrom можно только пересборкой ядра с `CONFIG_CDROM=m` или `CONFIG_CDROM=n`.

### Шаг 1.4. Конфигурация ядра

```bash
yuri@ubuntu-hp:~$ ls -l /boot/config-$(uname -r)
-rw-r--r-- 1 root root 287160 авг  7 15:07 /boot/config-6.8.0-138-generic

yuri@ubuntu-hp:~$ grep 'CONFIG_XFS_FS' /boot/config-$(uname -r)
CONFIG_XFS_FS=m
```

Конфигурация, с которой собрано ядро, лежит в `/boot/config-<версия>`. Каждый параметр принимает значение:
* `y`: функция встроена в ядро;
* `m`: собрана загружаемым модулем;
* `is not set`: не собрана.

`CONFIG_XFS_FS=m` означает, что поддержка XFS собрана модулем `xfs`, он подгружается при монтировании XFS.

## Задание 2. Наблюдение за VFS

### Шаг 2.1. strace

```bash
yuri@ubuntu-hp:~$ strace -e trace=openat,read,write,close cat /etc/os-release > /dev/null
openat(AT_FDCWD, "/etc/ld.so.cache", O_RDONLY|O_CLOEXEC) = 3
close(3)                                = 0
openat(AT_FDCWD, "/lib/x86_64-linux-gnu/libc.so.6", O_RDONLY|O_CLOEXEC) = 3
read(3, "\177ELF\2\1\1\3\0\0\0\0\0\0\0\0\3\0>\0\1\0\0\0P\237\2\0\0\0\0\0"..., 832) = 832
close(3)                                = 0
openat(AT_FDCWD, "/usr/lib/locale/locale-archive", O_RDONLY|O_CLOEXEC) = 3
close(3)                                = 0
openat(AT_FDCWD, "/etc/os-release", O_RDONLY) = 3
read(3, "PRETTY_NAME=\"Ubuntu 22.04.4 LTS\""..., 131072) = 386
write(1, "PRETTY_NAME=\"Ubuntu 22.04.4 LTS\""..., 386) = 386
read(3, "", 131072)                     = 0
close(3)                                = 0
close(1)                                = 0
close(2)                                = 0
+++ exited with 0 +++
```

Вывод strace идёт в stderr, поэтому он виден в терминале, а stdout `cat` перенаправлен в `/dev/null`.

* `openat(... "/etc/ld.so.cache" ...)`, `openat(... "libc.so.6" ...)`, `read(3, "\177ELF...", 832)` -- динамический загрузчик открывает кеш библиотек и читает ELF-заголовок libc. Это происходит при запуске любой динамически слинкованной программы.
* `openat(... "/usr/lib/locale/locale-archive" ...)` -- загрузка настроек локали.
* `openat(AT_FDCWD, "/etc/os-release", O_RDONLY) = 3` -- открытие нужного файла только на чтение, ядро выдало файловый дескриптор 3. `AT_FDCWD` означает, что путь считается от текущего каталога (здесь путь абсолютный). `/etc/os-release` -- это символическая ссылка на `/usr/lib/os-release`, VFS разрешает её сама.
* `read(3, "PRETTY_NAME=\"Ubuntu 22.04.4 LTS\", 131072) = 386` -- чтение содержимого файла (386 байт) буфером 128 KiB.
* `read(3, "", 131072) = 0` -- 0 байт означает конец файла.
* `close(3)`, `close(1)`, `close(2)` --закрытие файла и стандартных потоков.

`write(1, "PRETTY_NAME=\"Ubuntu 22.04.4 LTS\""..., 386) = 386`: `cat` пишет в stdout (fd 1), который перенаправлен в `/dev/null`. Вызов записи выполняется, но драйвер `/dev/null` сразу сообщает об успешной записи и отбрасывает данные. Поэтому записи в какой-либо файл на диске не происходит.


## Задание 3. LVM Management

### Шаг 3.1. Диск на 2 GiB

Работу выполняю на Ubuntu, установленной второй ОС с Windows, без виртуальной машины. Добавить физический диск `/dev/sdb` нельзя, поэтому эмулирую его loop-устройством -- файлом на 2 GiB, который ядро представляет как блочное устройство, для LVM отличий не будет.

```bash
yuri@ubuntu-hp:~$ truncate -s 2G ~/sdb.img

yuri@ubuntu-hp:~$ DISK=$(sudo losetup -fP --show ~/sdb.img)

yuri@ubuntu-hp:~$ echo $DISK
/dev/loop20

yuri@ubuntu-hp:~$ lsblk $DISK
NAME   MAJ:MIN RM SIZE RO TYPE MOUNTPOINTS
loop20   7:20   0   2G  0 loop 
```

* `truncate` создаёт разреженный файл на 2 GiB.
* `losetup -f` берёт первое свободное loop-устройство, `-P` включает поддержку разделов на нём, `--show` печатает имя устройства.

### Шаг 3.2. Раздел

```bash
yuri@ubuntu-hp:~$ sudo parted -s $DISK mklabel gpt mkpart primary 1MiB 100% set 1 lvm on

yuri@ubuntu-hp:~$ sudo parted $DISK print
Model: Loopback device (loopback)
Disk /dev/loop20: 2147MB
Sector size (logical/physical): 512B/512B
Partition Table: gpt
Disk Flags: 

Number  Start   End     Size    File system  Name     Flags
 1      1049kB  2146MB  2145MB               primary  lvm

yuri@ubuntu-hp:~$ lsblk $DISK
NAME       MAJ:MIN RM SIZE RO TYPE MOUNTPOINTS
loop20       7:20   0   2G  0 loop 
└─loop20p1 259:6    0   2G  0 part 
```

Создали таблицу разделов GPT и один раздел `loop20p1` на весь диск с флагом `lvm`. Отступ 1 MiB нужен для выравнивания раздела.

### Шаг 3.3. Physical Volume

```bash
yuri@ubuntu-hp:~$ sudo pvcreate ${DISK}p1
  Physical volume "/dev/loop20p1" successfully created.

yuri@ubuntu-hp:~$ sudo pvs
  PV            VG Fmt  Attr PSize  PFree 
  /dev/loop20p1    lvm2 ---  <2,00g <2,00g
```

### Шаг 3.4. Volume Group

```bash
yuri@ubuntu-hp:~$ sudo vgcreate vg_highload ${DISK}p1
  Volume group "vg_highload" successfully created

yuri@ubuntu-hp:~$ sudo vgs
  VG          #PV #LV #SN Attr   VSize  VFree 
  vg_highload   1   0   0 wz--n- <2,00g <2,00g

yuri@ubuntu-hp:~$ sudo vgdisplay vg_highload | grep -E 'PE Size|Total PE|Free'
  PE Size               4,00 MiB
  Total PE              511
  Free  PE / Size       511 / <2,00 GiB
```

Пространство VG делится на физические экстенты (PE) по 4 MiB, из них выделяются логические тома.

### Шаг 3.5. Logical Volumes

```bash
yuri@ubuntu-hp:~$ sudo lvcreate -n data_lv -L 1200M vg_highload
  Logical volume "data_lv" created.

yuri@ubuntu-hp:~$ sudo lvcreate -n logs_lv -l 100%FREE vg_highload
  Logical volume "logs_lv" created.

yuri@ubuntu-hp:~$ sudo lvs
  LV      VG          Attr       LSize   Pool Origin Data%  Meta%  Move Log Cpy%Sync Convert
  data_lv vg_highload -wi-a-----   1,17g                                                    
  logs_lv vg_highload -wi-a----- 844,00m                                           
```

* `-L 1200M` -- фиксированный размер 1200 MiB (300 экстентов).
* `-l 100%FREE` -- всё оставшееся свободное место VG.

### Шаг 3.6. Файловые системы

```bash
yuri@ubuntu-hp:~$ sudo mkfs.ext4 /dev/vg_highload/data_lv
mke2fs 1.46.5 (30-Dec-2021)
Discarding device blocks: done                            
Creating filesystem with 307200 4k blocks and 76800 inodes
Filesystem UUID: 0e5dd594-8b58-442f-871b-64d620cb37a3
Superblock backups stored on blocks: 
	32768, 98304, 163840, 229376, 294912

Allocating group tables: done                            
Writing inode tables: done                            
Creating journal (8192 blocks): done
Writing superblocks and filesystem accounting information: done 


yuri@ubuntu-hp:~$ sudo mkfs.xfs /dev/vg_highload/logs_lv
meta-data=/dev/vg_highload/logs_lv isize=512    agcount=4, agsize=54016 blks
         =                       sectsz=512   attr=2, projid32bit=1
         =                       crc=1        finobt=1, sparse=1, rmapbt=0
         =                       reflink=1    bigtime=0 inobtcount=0
data     =                       bsize=4096   blocks=216064, imaxpct=25
         =                       sunit=0      swidth=0 blks
naming   =version 2              bsize=4096   ascii-ci=0, ftype=1
log      =internal log           bsize=4096   blocks=1566, version=2
         =                       sectsz=512   sunit=0 blks, lazy-count=1
realtime =none                   extsz=4096   blocks=0, rtextents=0
Discarding blocks...Done.
```

### Шаг 3.7. Монтирование

```bash
yuri@ubuntu-hp:~$ sudo mkdir -p /mnt/app_data /mnt/app_logs

yuri@ubuntu-hp:~$ sudo mount /dev/vg_highload/data_lv /mnt/app_data
yuri@ubuntu-hp:~$ sudo mount /dev/vg_highload/logs_lv /mnt/app_logs

yuri@ubuntu-hp:~$ df -hT /mnt/app_data /mnt/app_logs
Filesystem                      Type  Size  Used Avail Use% Mounted on
/dev/mapper/vg_highload-data_lv ext4  1,2G   24K  1,1G   1% /mnt/app_data
/dev/mapper/vg_highload-logs_lv xfs   838M   39M  800M   5% /mnt/app_logs

yuri@ubuntu-hp:~$ lsblk $DISK
NAME                    MAJ:MIN RM  SIZE RO TYPE MOUNTPOINTS
loop20                    7:20   0    2G  0 loop 
└─loop20p1              259:6    0    2G  0 part 
  ├─vg_highload-data_lv 252:0    0  1,2G  0 lvm  /mnt/app_data
  └─vg_highload-logs_lv 252:1    0  844M  0 lvm  /mnt/app_logs
```

`data_lv` отформатирован как ext4 и примонтирован в `/mnt/app_data`, `logs_lv` отформатировар как xfs и примонтирован в `/mnt/app_logs`. В `lsblk` (list block devices) видим вся цепочку: диск, раздел, логические тома и точки монтирования.

## Задание 4. Использование pseudo filesystem

### Шаг 4.1. Модель CPU и объём памяти

```bash
yuri@ubuntu-hp:~$ grep -m1 'model name' /proc/cpuinfo
model name	: 11th Gen Intel(R) Core(TM) i5-1135G7 @ 2.40GHz

yuri@ubuntu-hp:~$ grep MemTotal /proc/meminfo
MemTotal:       16140508 kB
```

`/proc` это псевдофайловая система -- файлов на диске нет, ядро формирует содержимое в момент чтения. `MemTotal` указан в KiB (в выводе пишется `kB`, но фактически 1 KiB = 1024 байта). Это объём RAM, доступный ядру, он немного меньше физического, так как часть памяти резервируется при загрузке.

### Шаг 4.2. PPid текущего shell

```bash
yuri@ubuntu-hp:~$ echo $$
6363

yuri@ubuntu-hp:~$ grep -E '^(Name|Pid|PPid)' /proc/$$/status
Name:	bash
Pid:	6363
PPid:	6328

yuri@ubuntu-hp:~$ ps -o pid,comm -p $(awk '/^PPid/ {print $2}' /proc/$$/status)
    PID COMMAND
   6328 gnome-terminal-
   
Сам текущий процесс -- bash
yuri@ubuntu-hp:~$ ps -o pid,comm -p $(awk '/^Pid/ {print $2}' /proc/$$/status)
    PID COMMAND
   6363 bash
```

`$$` -- это специальная переменная shell, в которой хранится PID текущего процесса оболочки. Поэтому `/proc/$$/status` указывает на каталог процесса текущего bash. `PPid` -- это PID родительского процесса (здесь это gnome-terminal-, который запустил bash).

### Шаг 4.3. I/O scheduler

```bash
yuri@ubuntu-hp:~$ lsblk -d -o NAME,SIZE,TYPE
NAME      SIZE TYPE
loop0       4K loop
loop1    59,6M loop
loop2     5,4M loop
loop3   189,4M loop
loop4    59,6M loop
loop5   313,1M loop
loop6     105M loop
loop7   105,2M loop
loop8    55,5M loop
loop9    55,5M loop
loop10  313,7M loop
loop11   63,8M loop
loop12   73,9M loop
loop13     74M loop
loop14   63,8M loop
loop15   66,8M loop
loop16   66,8M loop
loop17  273,5M loop
loop18    272M loop
loop19  252,6M loop
loop20      2G loop
loop21   80,8M loop
loop22  164,8M loop
loop23  349,7M loop
loop24  505,1M loop
loop25    516M loop
loop26  589,6M loop
loop27  606,1M loop
loop28   91,7M loop
loop29  290,8M loop
loop30  112,2M loop
loop31  112,6M loop
loop32  274,2M loop
loop33  191,4M loop
loop34    173M loop
loop35    173M loop
loop36  119,6M loop
loop37  119,6M loop
loop38   12,9M loop
loop39   12,2M loop
loop40   38,8M loop
loop41   49,3M loop
loop42    576K loop
loop43    576K loop
loop45  106,6M loop
loop46   98,5M loop
loop47  114,1M loop
loop48   87,7M loop
loop49    6,5M loop
loop50  106,6M loop
nvme0n1 476,9G disk

yuri@ubuntu-hp:~$ cat /sys/block/nvme0n1/queue/scheduler
[none] mq-deadline 
```

Активный планировщик указан в квадратных скобках, остальные доступны для переключения.
У меня активный планировщик none, то есть ядро не переупорядочивает запросы ввода-вывода и сразу отправляет их устройству.
mq-deadline (планировщик ввода-вывода ядра Linux) доступен для переключения

Настройка берётся из `/sys`, это тоже псевдофайловая система (sysfs), через которую ядро показывает параметры устройств.

### Шаг 4.4. MTU

```bash
yuri@ubuntu-hp:~$ ip route show default
default via 192.168.1.1 dev enx00e04c2c4dc0 proto dhcp metric 100 
default via 192.168.1.1 dev wlo1 proto dhcp metric 600 

yuri@ubuntu-hp:~$ cat /sys/class/net/wlo1/mtu
1500

yuri@ubuntu-hp:~$ ip link show wlo1
3: wlo1: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc noqueue state UP mode DORMANT group default qlen 1000
    link/ether 74:12:b3:e0:e4:5d brd ff:ff:ff:ff:ff:ff
    altname wlp2s0

```

Основной интерфейс определил по маршруту по умолчанию (`dev wlo1`). MTU 1500 байт -- максимальный размер IP-пакета, который передаётся без фрагментации (Maximum Transmission Unit).
# Домашнее задание 3. Linux. Архитектура и файловые системы

Дополнительные пакеты:

```bash
sudo apt install -y strace lvm2 xfsprogs parted

(после успешно пройденной установки и повторного выполнения команды)

yuri@ubuntu-hp:~$ sudo apt install -y strace lvm2 xfsprogs parted
Reading package lists... Done
Building dependency tree... Done
Reading state information... Done
parted is already the newest version (3.4-2build1).
strace is already the newest version (5.16-0ubuntu3).
lvm2 is already the newest version (2.03.11-2.1ubuntu5).
xfsprogs is already the newest version (5.13.0-1ubuntu2.1).
The following packages were automatically installed and are no longer required:
  libbrotli-dev libfontconfig-dev libfontconfig1-dev libfreetype-dev
  libfreetype6-dev libpng-dev libpng-tools libsdl-ttf2.0-0 libxkbcommon-dev
  python3-docker python3-dockerpty python3-docopt python3-dotenv
  python3-jsonschema python3-pyrsistent python3-texttable python3-websocket
Use 'sudo apt autoremove' to remove them.
0 upgraded, 0 newly installed, 0 to remove and 176 not upgraded.
```

## Задание 1. Kernel and Module Inspection

### Шаг 1.1. Версия ядра

```bash
yuri@ubuntu-hp:~$ uname -r
6.8.0-138-generic

yuri@ubuntu-hp:~$ uname -a
Linux ubuntu-hp 6.8.0-138-generic #138~22.04.1-Ubuntu SMP PREEMPT_DYNAMIC Fri Aug  7 13:43:15 UTC  x86_64 x86_64 x86_64 GNU/Linux

yuri@ubuntu-hp:~$ cat /proc/version
Linux version 6.8.0-138-generic (buildd@lcy02-amd64-008) (x86_64-linux-gnu-gcc-12 (Ubuntu 12.3.0-1ubuntu1~22.04.3) 12.3.0, GNU ld (GNU Binutils for Ubuntu) 2.38) #138~22.04.1-Ubuntu SMP PREEMPT_DYNAMIC Fri Aug  7 13:43:15 UTC 
```

`uname -r` показывает релиз ядра, `uname -a` дополнительно архитектуру и дату сборки, `/proc/version` также версию компилятора, которым собрано ядро.

### Шаг 1.2. Загруженные модули

```bash
yuri@ubuntu-hp:~$ lsmod
Module                  Size  Used by
xt_conntrack           12288  3
nft_chain_nat          12288  3
xt_MASQUERADE          16384  3
nf_nat                 61440  2 nft_chain_nat,xt_MASQUERADE
nf_conntrack_netlink    57344  0
nf_conntrack          200704  4 xt_conntrack,nf_nat,nf_conntrack_netlink,xt_MASQUERADE
nf_defrag_ipv6         24576  1 nf_conntrack
nf_defrag_ipv4         12288  1 nf_conntrack
xfrm_user              61440  1
xfrm_algo              16384  1 xfrm_user
xt_addrtype            12288  2
nft_compat             20480  8
nf_tables             380928  136 nft_compat,nft_chain_nat
libcrc32c              12288  3 nf_conntrack,nf_nat,nf_tables
nfnetlink              20480  4 nft_compat,nf_conntrack_netlink,nf_tables
br_netfilter           32768  0
bridge                425984  1 br_netfilter
stp                    12288  1 bridge
llc                    16384  2 bridge,stp
ccm                    20480  6
rfcomm                102400  4
snd_ctl_led            24576  0
ledtrig_audio          12288  1 snd_ctl_led
snd_soc_skl_hda_dsp    24576  6
snd_soc_hdac_hdmi      49152  1 snd_soc_skl_hda_dsp
snd_soc_intel_hda_dsp_common    16384  1 snd_soc_skl_hda_dsp
snd_sof_probes         20480  0
snd_hda_codec_realtek   212992  1
snd_hda_codec_generic   122880  1 snd_hda_codec_realtek
snd_soc_dmic           12288  1
snd_sof_pci_intel_tgl    12288  0
snd_sof_intel_hda_common   217088  1 snd_sof_pci_intel_tgl
soundwire_intel        73728  1 snd_sof_intel_hda_common
snd_sof_intel_hda_mlink    45056  2 soundwire_intel,snd_sof_intel_hda_common
soundwire_cadence      45056  1 soundwire_intel
snd_sof_intel_hda      24576  1 snd_sof_intel_hda_common
snd_hda_codec_hdmi     94208  1
snd_sof_pci            24576  2 snd_sof_intel_hda_common,snd_sof_pci_intel_tgl
snd_sof_xtensa_dsp     12288  1 snd_sof_intel_hda_common
snd_sof               380928  4 snd_sof_pci,snd_sof_intel_hda_common,snd_sof_probes,snd_sof_intel_hda
xe                   2736128  0
snd_sof_utils          16384  1 snd_sof
snd_soc_hdac_hda       24576  1 snd_sof_intel_hda_common
snd_hda_ext_core       32768  5 snd_sof_intel_hda_common,snd_soc_hdac_hdmi,snd_soc_hdac_hda,snd_sof_intel_hda_mlink,snd_sof_intel_hda
snd_soc_acpi_intel_match    98304  2 snd_sof_intel_hda_common,snd_sof_pci_intel_tgl
cmac                   12288  2
snd_soc_acpi           16384  2 snd_soc_acpi_intel_match,snd_sof_intel_hda_common
algif_hash             12288  1
soundwire_generic_allocation    12288  1 soundwire_intel
soundwire_bus         110592  3 soundwire_intel,soundwire_generic_allocation,soundwire_cadence
algif_skcipher         12288  1
overlay               212992  0
af_alg                 32768  6 algif_hash,algif_skcipher
snd_soc_core          442368  8 soundwire_intel,snd_sof,snd_sof_intel_hda_common,snd_soc_hdac_hdmi,snd_soc_hdac_hda,snd_sof_probes,snd_soc_dmic,snd_soc_skl_hda_dsp
bnep                   32768  2
drm_gpuvm              45056  1 xe
drm_exec               12288  2 drm_gpuvm,xe
gpu_sched              61440  1 xe
snd_compress           28672  2 snd_soc_core,snd_sof_probes
drm_suballoc_helper    20480  1 xe
ac97_bus               12288  1 snd_soc_core
intel_uncore_frequency    16384  0
drm_ttm_helper         12288  1 xe
intel_uncore_frequency_common    16384  1 intel_uncore_frequency
snd_pcm_dmaengine      16384  1 snd_soc_core
x86_pkg_temp_thermal    20480  0
snd_hda_intel          61440  0
intel_powerclamp       24576  0
snd_intel_dspcfg       36864  3 snd_hda_intel,snd_sof,snd_sof_intel_hda_common
coretemp               24576  0
snd_intel_sdw_acpi     16384  2 snd_sof_intel_hda_common,snd_intel_dspcfg
snd_hda_codec         204800  8 snd_hda_codec_generic,snd_hda_codec_hdmi,snd_hda_intel,snd_hda_codec_realtek,snd_soc_intel_hda_dsp_common,snd_soc_hdac_hda,snd_sof_intel_hda,snd_soc_skl_hda_dsp
kvm_intel             491520  0
snd_hda_core          139264  11 snd_hda_codec_generic,snd_hda_codec_hdmi,snd_hda_intel,snd_hda_ext_core,snd_hda_codec,snd_hda_codec_realtek,snd_soc_intel_hda_dsp_common,snd_sof_intel_hda_common,snd_soc_hdac_hdmi,snd_soc_hdac_hda,snd_sof_intel_hda
joydev                 32768  0
kvm                  1409024  1 kvm_intel
snd_hwdep              20480  1 snd_hda_codec
rtw89_8852ae           12288  0
i915                 4300800  28
irqbypass              12288  1 kvm
rtw89_8852a           724992  1 rtw89_8852ae
snd_pcm               192512  12 snd_hda_codec_hdmi,snd_hda_intel,snd_hda_codec,soundwire_intel,snd_sof,snd_sof_intel_hda_common,snd_soc_hdac_hdmi,snd_compress,snd_soc_core,snd_sof_utils,snd_hda_core,snd_pcm_dmaengine
crct10dif_pclmul       12288  1
polyval_clmulni        12288  0
uvcvideo              147456  0
polyval_generic        12288  1 polyval_clmulni
rtw89_pci              81920  1 rtw89_8852ae
videobuf2_vmalloc      20480  1 uvcvideo
btusb                  81920  0
ghash_clmulni_intel    16384  0
btrtl                  32768  1 btusb
snd_seq_midi           24576  0
sha256_ssse3           32768  0
uvc                    12288  1 uvcvideo
snd_seq_midi_event     16384  1 snd_seq_midi
btintel                57344  1 btusb
rtw89_core            737280  2 rtw89_pci,rtw89_8852a
sha1_ssse3             32768  0
videobuf2_memops       16384  1 videobuf2_vmalloc
processor_thermal_device_pci_legacy    12288  0
btbcm                  24576  1 btusb
snd_rawmidi            57344  1 snd_seq_midi
aesni_intel           356352  7
videobuf2_v4l2         36864  1 uvcvideo
processor_thermal_device    20480  1 processor_thermal_device_pci_legacy
btmtk                  12288  1 btusb
intel_rapl_msr         20480  0
mei_pxp                16384  0
mei_hdcp               28672  0
videodev              356352  2 videobuf2_v4l2,uvcvideo
binfmt_misc            24576  1
crypto_simd            16384  1 aesni_intel
processor_thermal_wt_hint    16384  1 processor_thermal_device
hid_sensor_accel_3d    16384  1
hid_sensor_gyro_3d     16384  0
drm_buddy              20480  2 xe,i915
hid_sensor_rotation    16384  0
hid_sensor_custom_intel_hinge    12288  0
bluetooth            1044480  34 btrtl,btmtk,btintel,btbcm,bnep,btusb,rfcomm
snd_seq               122880  2 snd_seq_midi,snd_seq_midi_event
mac80211             1753088  2 rtw89_core,rtw89_pci
cryptd                 24576  3 crypto_simd,ghash_clmulni_intel
processor_thermal_rfim    32768  1 processor_thermal_device
ttm                   110592  3 drm_ttm_helper,xe,i915
videobuf2_common       81920  4 videobuf2_vmalloc,videobuf2_v4l2,uvcvideo,videobuf2_memops
hid_sensor_trigger     20480  9 hid_sensor_gyro_3d,hid_sensor_custom_intel_hinge,hid_sensor_accel_3d,hid_sensor_rotation
snd_seq_device         16384  3 snd_seq,snd_seq_midi,snd_rawmidi
ecdh_generic           16384  1 bluetooth
nls_iso8859_1          12288  1
cmdlinepart            12288  0
rapl                   20480  0
hp_wmi                 36864  0
processor_thermal_rapl    16384  1 processor_thermal_device
hp_bioscfg             81920  0
intel_cstate           24576  0
industrialio_triggered_buffer    12288  1 hid_sensor_trigger
drm_display_helper    237568  2 xe,i915
snd_timer              49152  2 snd_seq,snd_pcm
mc                     81920  4 videodev,videobuf2_v4l2,uvcvideo,videobuf2_common
ecc                    45056  1 ecdh_generic
input_leds             12288  0
spi_nor               167936  0
cfg80211             1368064  3 rtw89_core,mac80211,rtw89_8852a
serio_raw              20480  0
platform_profile       12288  1 hp_wmi
intel_rapl_common      40960  2 intel_rapl_msr,processor_thermal_rapl
firmware_attributes_class    12288  1 hp_bioscfg
ee1004                 16384  0
hid_multitouch         36864  0
kfifo_buf              12288  1 industrialio_triggered_buffer
wmi_bmof               12288  0
cec                    94208  3 drm_display_helper,xe,i915
snd                   143360  28 snd_ctl_led,snd_hda_codec_generic,snd_seq,snd_seq_device,snd_hda_codec_hdmi,snd_hwdep,snd_hda_intel,snd_hda_codec,snd_hda_codec_realtek,snd_sof,snd_timer,snd_soc_hdac_hdmi,snd_compress,snd_soc_core,snd_pcm,snd_rawmidi
mtd                    98304  3 spi_nor,cmdlinepart
mei_me                 53248  2
hid_sensor_iio_common    24576  5 hid_sensor_gyro_3d,hid_sensor_trigger,hid_sensor_custom_intel_hinge,hid_sensor_accel_3d,hid_sensor_rotation
processor_thermal_wt_req    12288  1 processor_thermal_device
rc_core                73728  1 cec
industrialio          126976  8 industrialio_triggered_buffer,hid_sensor_gyro_3d,hid_sensor_trigger,hid_sensor_custom_intel_hinge,kfifo_buf,hid_sensor_accel_3d,hid_sensor_rotation
processor_thermal_power_floor    12288  1 processor_thermal_device
soundcore              16384  2 snd_ctl_led,snd
libarc4                12288  1 mac80211
mei                   172032  5 mei_hdcp,mei_pxp,mei_me
i2c_algo_bit           16384  2 xe,i915
processor_thermal_mbox    12288  4 processor_thermal_power_floor,processor_thermal_wt_req,processor_thermal_rfim,processor_thermal_wt_hint
igen6_edac             24576  0
intel_soc_dts_iosf     20480  1 processor_thermal_device_pci_legacy
intel_pmc_core        118784  0
int3403_thermal        16384  0
soc_button_array       20480  0
intel_vsec             20480  1 intel_pmc_core
int340x_thermal_zone    16384  2 int3403_thermal,processor_thermal_device
pmt_telemetry          16384  1 intel_pmc_core
intel_hid              32768  0
int3400_thermal        24576  0
pmt_class              12288  1 pmt_telemetry
acpi_thermal_rel       20480  1 int3400_thermal
sparse_keymap          12288  2 hp_wmi,intel_hid
acpi_pad              184320  0
mac_hid                12288  0
sch_fq_codel           24576  2
msr                    12288  0
parport_pc             53248  0
ppdev                  24576  0
lp                     28672  0
parport                73728  3 parport_pc,lp,ppdev
efi_pstore             12288  0
ip_tables              32768  0
x_tables               65536  5 xt_conntrack,nft_compat,xt_addrtype,ip_tables,xt_MASQUERADE
autofs4                57344  2
hid_sensor_custom      28672  0
r8153_ecm              12288  0
cdc_ether              24576  1 r8153_ecm
hid_sensor_hub         28672  7 hid_sensor_gyro_3d,hid_sensor_trigger,hid_sensor_iio_common,hid_sensor_custom_intel_hinge,hid_sensor_accel_3d,hid_sensor_rotation,hid_sensor_custom
usbnet                 61440  2 r8153_ecm,cdc_ether
intel_ishtp_hid        32768  0
r8152                 143360  1 r8153_ecm
mii                    20480  2 usbnet,r8152
nvme                   61440  2
nvme_core             212992  3 nvme
nvme_auth              28672  1 nvme_core
hid_generic            12288  0
rtsx_pci_sdmmc         36864  0
intel_lpss_pci         28672  0
spi_intel_pci          12288  0
i2c_i801               36864  0
intel_ish_ipc          32768  0
intel_lpss             12288  1 intel_lpss_pci
i2c_hid_acpi           12288  0
crc32_pclmul           12288  0
ucsi_acpi              12288  0
xhci_pci               24576  0
typec_ucsi             61440  1 ucsi_acpi
spi_intel              32768  1 spi_intel_pci
i2c_smbus              16384  1 i2c_i801
rtsx_pci              143360  1 rtsx_pci_sdmmc
intel_ishtp            69632  2 intel_ishtp_hid,intel_ish_ipc
idma64                 20480  0
vmd                    24576  0
video                  77824  2 xe,i915
i2c_hid                40960  1 i2c_hid_acpi
xhci_pci_renesas       20480  1 xhci_pci
typec                 106496  1 typec_ucsi
hid                   180224  5 i2c_hid,hid_multitouch,hid_sensor_hub,intel_ishtp_hid,hid_generic
wmi                    28672  4 hp_wmi,video,hp_bioscfg,wmi_bmof
pinctrl_tigerlake      28672  2
```

```bash
yuri@ubuntu-hp:~$ lsmod | tail -n +2 | wc -l
226
```

Всего загружено 226 модулей. Столбцы: имя модуля, размер в памяти, счётчик использования и модули, которые от него зависят.

### Шаг 1.3. Отключение автозагрузки cdrom

Проверим, собран ли cdrom модулем и загружен ли он сейчас:

```bash
yuri@ubuntu-hp:~$ grep -E 'CONFIG_CDROM=|CONFIG_BLK_DEV_SR=' /boot/config-$(uname -r)
CONFIG_CDROM=y
CONFIG_BLK_DEV_SR=y

yuri@ubuntu-hp:~$ lsmod | grep -E 'cdrom|sr_mod'

yuri@ubuntu-hp:~$ modinfo cdrom | head -n 5
name:           cdrom
filename:       (builtin)
license:        GPL
file:           drivers/cdrom/cdrom
parm:           debug:bool

yuri@ubuntu-hp:~$ modinfo -n sr_mod
(builtin)
```

`CONFIG_CDROM=y`, `CONFIG_BLK_DEV_SR=y` и `filename: (builtin)`, то есть на этом ядре (6.8.0-138-generic) cdrom и драйвер CD-приводов sr_mod встроены в ядро, не собраны загружаемыми модулями. Поэтому их нет в `lsmod`, и выгрузить их нельзя:
```bash
yuri@ubuntu-hp:~$ sudo modprobe -r cdrom
modprobe: FATAL: Module cdrom is builtin.
```

Запрещаем автозагрузку через modprobe.d:

```bash
yuri@ubuntu-hp:~$ sudo tee /etc/modprobe.d/blacklist-cdrom.conf > /dev/null <<'EOF'
blacklist cdrom
blacklist sr_mod
install cdrom /bin/false
EOF

yuri@ubuntu-hp:~$ cat /etc/modprobe.d/blacklist-cdrom.conf
blacklist cdrom
blacklist sr_mod
install cdrom /bin/false

Если бы модули были не встроенными, мы бы ещё выполнили update-initramfs -u -- то есть пересобрали образ с модулями, который ядро загружает на самом раннем этапе. Он нужен, чтобы blacklist подействовал и на модули, которые грузятся оттуда, ещё до основной системы. Но cdrom и sr_mod встроены в само ядро, в initramfs их нет как отдельных модулей, поэтому пересборка ничего не изменит
```

* `blacklist` запрещает автоматическую загрузку модуля по alias (при обнаружении устройства).
* `blacklist` не мешает загрузить модуль как зависимость другого, поэтому добавлен `install cdrom /bin/false` -- вместо загрузки модуля modprobe выполнит `/bin/false`.

`blacklist`  действует только на загружаемые модули (файлы `.ko` -- kernel object), а встроенный код стартует вместе с ядром, modprobe в этом не участвует. Поэтому на этом ядре файл ни на что не влияет -- он сработает на ядре, где cdrom собран модулем (`CONFIG_CDROM=m`).

Полностью отключить встроенный cdrom можно только пересборкой ядра с `CONFIG_CDROM=m` или `CONFIG_CDROM=n`.

### Шаг 1.4. Конфигурация ядра

```bash
yuri@ubuntu-hp:~$ ls -l /boot/config-$(uname -r)
-rw-r--r-- 1 root root 287160 авг  7 15:07 /boot/config-6.8.0-138-generic

yuri@ubuntu-hp:~$ grep 'CONFIG_XFS_FS' /boot/config-$(uname -r)
CONFIG_XFS_FS=m
```

Конфигурация, с которой собрано ядро, лежит в `/boot/config-<версия>`. Каждый параметр принимает значение:
* `y`: функция встроена в ядро;
* `m`: собрана загружаемым модулем;
* `is not set`: не собрана.

`CONFIG_XFS_FS=m` означает, что поддержка XFS собрана модулем `xfs`, он подгружается при монтировании XFS.

## Задание 2. Наблюдение за VFS

### Шаг 2.1. strace

```bash
yuri@ubuntu-hp:~$ strace -e trace=openat,read,write,close cat /etc/os-release > /dev/null
openat(AT_FDCWD, "/etc/ld.so.cache", O_RDONLY|O_CLOEXEC) = 3
close(3)                                = 0
openat(AT_FDCWD, "/lib/x86_64-linux-gnu/libc.so.6", O_RDONLY|O_CLOEXEC) = 3
read(3, "\177ELF\2\1\1\3\0\0\0\0\0\0\0\0\3\0>\0\1\0\0\0P\237\2\0\0\0\0\0"..., 832) = 832
close(3)                                = 0
openat(AT_FDCWD, "/usr/lib/locale/locale-archive", O_RDONLY|O_CLOEXEC) = 3
close(3)                                = 0
openat(AT_FDCWD, "/etc/os-release", O_RDONLY) = 3
read(3, "PRETTY_NAME=\"Ubuntu 22.04.4 LTS\""..., 131072) = 386
write(1, "PRETTY_NAME=\"Ubuntu 22.04.4 LTS\""..., 386) = 386
read(3, "", 131072)                     = 0
close(3)                                = 0
close(1)                                = 0
close(2)                                = 0
+++ exited with 0 +++
```

Вывод strace идёт в stderr, поэтому он виден в терминале, а stdout `cat` перенаправлен в `/dev/null`.

* `openat(... "/etc/ld.so.cache" ...)`, `openat(... "libc.so.6" ...)`, `read(3, "\177ELF...", 832)` -- динамический загрузчик открывает кеш библиотек и читает ELF-заголовок libc. Это происходит при запуске любой динамически слинкованной программы.
* `openat(... "/usr/lib/locale/locale-archive" ...)` -- загрузка настроек локали.
* `openat(AT_FDCWD, "/etc/os-release", O_RDONLY) = 3` -- открытие нужного файла только на чтение, ядро выдало файловый дескриптор 3. `AT_FDCWD` означает, что путь считается от текущего каталога (здесь путь абсолютный). `/etc/os-release` -- это символическая ссылка на `/usr/lib/os-release`, VFS разрешает её сама.
* `read(3, "PRETTY_NAME=\"Ubuntu 22.04.4 LTS\", 131072) = 386` -- чтение содержимого файла (386 байт) буфером 128 KiB.
* `read(3, "", 131072) = 0` -- 0 байт означает конец файла.
* `close(3)`, `close(1)`, `close(2)` --закрытие файла и стандартных потоков.

`write(1, "PRETTY_NAME=\"Ubuntu 22.04.4 LTS\""..., 386) = 386`: `cat` пишет в stdout (fd 1), который перенаправлен в `/dev/null`. Вызов записи выполняется, но драйвер `/dev/null` сразу сообщает об успешной записи и отбрасывает данные. Поэтому записи в какой-либо файл на диске не происходит.


## Задание 3. LVM Management

### Шаг 3.1. Диск на 2 GiB

Работу выполняю на Ubuntu, установленной второй ОС с Windows, без виртуальной машины. Добавить физический диск `/dev/sdb` нельзя, поэтому эмулирую его loop-устройством -- файлом на 2 GiB, который ядро представляет как блочное устройство, для LVM отличий не будет.

```bash
yuri@ubuntu-hp:~$ truncate -s 2G ~/sdb.img

yuri@ubuntu-hp:~$ DISK=$(sudo losetup -fP --show ~/sdb.img)

yuri@ubuntu-hp:~$ echo $DISK
/dev/loop20

yuri@ubuntu-hp:~$ lsblk $DISK
NAME   MAJ:MIN RM SIZE RO TYPE MOUNTPOINTS
loop20   7:20   0   2G  0 loop 
```

* `truncate` создаёт разреженный файл на 2 GiB.
* `losetup -f` берёт первое свободное loop-устройство, `-P` включает поддержку разделов на нём, `--show` печатает имя устройства.

### Шаг 3.2. Раздел

```bash
yuri@ubuntu-hp:~$ sudo parted -s $DISK mklabel gpt mkpart primary 1MiB 100% set 1 lvm on

yuri@ubuntu-hp:~$ sudo parted $DISK print
Model: Loopback device (loopback)
Disk /dev/loop20: 2147MB
Sector size (logical/physical): 512B/512B
Partition Table: gpt
Disk Flags: 

Number  Start   End     Size    File system  Name     Flags
 1      1049kB  2146MB  2145MB               primary  lvm

yuri@ubuntu-hp:~$ lsblk $DISK
NAME       MAJ:MIN RM SIZE RO TYPE MOUNTPOINTS
loop20       7:20   0   2G  0 loop 
└─loop20p1 259:6    0   2G  0 part 
```

Создали таблицу разделов GPT и один раздел `loop20p1` на весь диск с флагом `lvm`. Отступ 1 MiB нужен для выравнивания раздела.

### Шаг 3.3. Physical Volume

```bash
yuri@ubuntu-hp:~$ sudo pvcreate ${DISK}p1
  Physical volume "/dev/loop20p1" successfully created.

yuri@ubuntu-hp:~$ sudo pvs
  PV            VG Fmt  Attr PSize  PFree 
  /dev/loop20p1    lvm2 ---  <2,00g <2,00g
```

### Шаг 3.4. Volume Group

```bash
yuri@ubuntu-hp:~$ sudo vgcreate vg_highload ${DISK}p1
  Volume group "vg_highload" successfully created

yuri@ubuntu-hp:~$ sudo vgs
  VG          #PV #LV #SN Attr   VSize  VFree 
  vg_highload   1   0   0 wz--n- <2,00g <2,00g

yuri@ubuntu-hp:~$ sudo vgdisplay vg_highload | grep -E 'PE Size|Total PE|Free'
  PE Size               4,00 MiB
  Total PE              511
  Free  PE / Size       511 / <2,00 GiB
```

Пространство VG делится на физические экстенты (PE) по 4 MiB, из них выделяются логические тома.

### Шаг 3.5. Logical Volumes

```bash
yuri@ubuntu-hp:~$ sudo lvcreate -n data_lv -L 1200M vg_highload
  Logical volume "data_lv" created.

yuri@ubuntu-hp:~$ sudo lvcreate -n logs_lv -l 100%FREE vg_highload
  Logical volume "logs_lv" created.

yuri@ubuntu-hp:~$ sudo lvs
  LV      VG          Attr       LSize   Pool Origin Data%  Meta%  Move Log Cpy%Sync Convert
  data_lv vg_highload -wi-a-----   1,17g                                                    
  logs_lv vg_highload -wi-a----- 844,00m                                           
```

* `-L 1200M` -- фиксированный размер 1200 MiB (300 экстентов).
* `-l 100%FREE` -- всё оставшееся свободное место VG.

### Шаг 3.6. Файловые системы

```bash
yuri@ubuntu-hp:~$ sudo mkfs.ext4 /dev/vg_highload/data_lv
mke2fs 1.46.5 (30-Dec-2021)
Discarding device blocks: done                            
Creating filesystem with 307200 4k blocks and 76800 inodes
Filesystem UUID: 0e5dd594-8b58-442f-871b-64d620cb37a3
Superblock backups stored on blocks: 
	32768, 98304, 163840, 229376, 294912

Allocating group tables: done                            
Writing inode tables: done                            
Creating journal (8192 blocks): done
Writing superblocks and filesystem accounting information: done 


yuri@ubuntu-hp:~$ sudo mkfs.xfs /dev/vg_highload/logs_lv
meta-data=/dev/vg_highload/logs_lv isize=512    agcount=4, agsize=54016 blks
         =                       sectsz=512   attr=2, projid32bit=1
         =                       crc=1        finobt=1, sparse=1, rmapbt=0
         =                       reflink=1    bigtime=0 inobtcount=0
data     =                       bsize=4096   blocks=216064, imaxpct=25
         =                       sunit=0      swidth=0 blks
naming   =version 2              bsize=4096   ascii-ci=0, ftype=1
log      =internal log           bsize=4096   blocks=1566, version=2
         =                       sectsz=512   sunit=0 blks, lazy-count=1
realtime =none                   extsz=4096   blocks=0, rtextents=0
Discarding blocks...Done.
```

### Шаг 3.7. Монтирование

```bash
yuri@ubuntu-hp:~$ sudo mkdir -p /mnt/app_data /mnt/app_logs

yuri@ubuntu-hp:~$ sudo mount /dev/vg_highload/data_lv /mnt/app_data
yuri@ubuntu-hp:~$ sudo mount /dev/vg_highload/logs_lv /mnt/app_logs

yuri@ubuntu-hp:~$ df -hT /mnt/app_data /mnt/app_logs
Filesystem                      Type  Size  Used Avail Use% Mounted on
/dev/mapper/vg_highload-data_lv ext4  1,2G   24K  1,1G   1% /mnt/app_data
/dev/mapper/vg_highload-logs_lv xfs   838M   39M  800M   5% /mnt/app_logs

yuri@ubuntu-hp:~$ lsblk $DISK
NAME                    MAJ:MIN RM  SIZE RO TYPE MOUNTPOINTS
loop20                    7:20   0    2G  0 loop 
└─loop20p1              259:6    0    2G  0 part 
  ├─vg_highload-data_lv 252:0    0  1,2G  0 lvm  /mnt/app_data
  └─vg_highload-logs_lv 252:1    0  844M  0 lvm  /mnt/app_logs
```

`data_lv` отформатирован как ext4 и примонтирован в `/mnt/app_data`, `logs_lv` отформатировар как xfs и примонтирован в `/mnt/app_logs`. В `lsblk` (list block devices) видим вся цепочку: диск, раздел, логические тома и точки монтирования.

## Задание 4. Использование pseudo filesystem

### Шаг 4.1. Модель CPU и объём памяти

```bash
yuri@ubuntu-hp:~$ grep -m1 'model name' /proc/cpuinfo
model name	: 11th Gen Intel(R) Core(TM) i5-1135G7 @ 2.40GHz

yuri@ubuntu-hp:~$ grep MemTotal /proc/meminfo
MemTotal:       16140508 kB
```

`/proc` это псевдофайловая система -- файлов на диске нет, ядро формирует содержимое в момент чтения. `MemTotal` указан в KiB (в выводе пишется `kB`, но фактически 1 KiB = 1024 байта). Это объём RAM, доступный ядру, он немного меньше физического, так как часть памяти резервируется при загрузке.

### Шаг 4.2. PPid текущего shell

```bash
yuri@ubuntu-hp:~$ echo $$
6363

yuri@ubuntu-hp:~$ grep -E '^(Name|Pid|PPid)' /proc/$$/status
Name:	bash
Pid:	6363
PPid:	6328

yuri@ubuntu-hp:~$ ps -o pid,comm -p $(awk '/^PPid/ {print $2}' /proc/$$/status)
    PID COMMAND
   6328 gnome-terminal-
   
Сам текущий процесс -- bash
yuri@ubuntu-hp:~$ ps -o pid,comm -p $(awk '/^Pid/ {print $2}' /proc/$$/status)
    PID COMMAND
   6363 bash
```

`$$` -- это специальная переменная shell, в которой хранится PID текущего процесса оболочки. Поэтому `/proc/$$/status` указывает на каталог процесса текущего bash. `PPid` -- это PID родительского процесса (здесь это gnome-terminal-, который запустил bash).

### Шаг 4.3. I/O scheduler

```bash
yuri@ubuntu-hp:~$ lsblk -d -o NAME,SIZE,TYPE
NAME      SIZE TYPE
loop0       4K loop
loop1    59,6M loop
loop2     5,4M loop
loop3   189,4M loop
loop4    59,6M loop
loop5   313,1M loop
loop6     105M loop
loop7   105,2M loop
loop8    55,5M loop
loop9    55,5M loop
loop10  313,7M loop
loop11   63,8M loop
loop12   73,9M loop
loop13     74M loop
loop14   63,8M loop
loop15   66,8M loop
loop16   66,8M loop
loop17  273,5M loop
loop18    272M loop
loop19  252,6M loop
loop20      2G loop
loop21   80,8M loop
loop22  164,8M loop
loop23  349,7M loop
loop24  505,1M loop
loop25    516M loop
loop26  589,6M loop
loop27  606,1M loop
loop28   91,7M loop
loop29  290,8M loop
loop30  112,2M loop
loop31  112,6M loop
loop32  274,2M loop
loop33  191,4M loop
loop34    173M loop
loop35    173M loop
loop36  119,6M loop
loop37  119,6M loop
loop38   12,9M loop
loop39   12,2M loop
loop40   38,8M loop
loop41   49,3M loop
loop42    576K loop
loop43    576K loop
loop45  106,6M loop
loop46   98,5M loop
loop47  114,1M loop
loop48   87,7M loop
loop49    6,5M loop
loop50  106,6M loop
nvme0n1 476,9G disk

yuri@ubuntu-hp:~$ cat /sys/block/nvme0n1/queue/scheduler
[none] mq-deadline 
```

Активный планировщик указан в квадратных скобках, остальные доступны для переключения.
У меня активный планировщик none, то есть ядро не переупорядочивает запросы ввода-вывода и сразу отправляет их устройству.
mq-deadline (планировщик ввода-вывода ядра Linux) доступен для переключения

Настройка берётся из `/sys`, это тоже псевдофайловая система (sysfs), через которую ядро показывает параметры устройств.

### Шаг 4.4. MTU

```bash
yuri@ubuntu-hp:~$ ip route show default
default via 192.168.1.1 dev enx00e04c2c4dc0 proto dhcp metric 100 
default via 192.168.1.1 dev wlo1 proto dhcp metric 600 

yuri@ubuntu-hp:~$ cat /sys/class/net/wlo1/mtu
1500

yuri@ubuntu-hp:~$ ip link show wlo1
3: wlo1: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc noqueue state UP mode DORMANT group default qlen 1000
    link/ether 74:12:b3:e0:e4:5d brd ff:ff:ff:ff:ff:ff
    altname wlp2s0

```

Основной интерфейс определил по маршруту по умолчанию (`dev wlo1`). MTU 1500 байт -- максимальный размер IP-пакета, который передаётся без фрагментации (Maximum Transmission Unit).
