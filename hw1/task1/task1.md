## Задание 1. Kernel and Module Inspection.

### Демонстрация версии ядра ОС

```shell
uname -r
```

Вывод команды:
```
6.8.0-84-generic
```

### Демонстрация всех загруженных модулей ядра

```shell
lsmod
```

Вывод команды:

```
Module                  Size  Used by
tls                   155648  0
veth                   45056  0
nf_conntrack_netlink    57344  0
xt_nat                 12288  16
xt_tcpudp              16384  40
xt_conntrack           12288  14
xt_MASQUERADE          16384  14
xt_set                 20480  0
ip_set                 61440  1 xt_set
nft_chain_nat          12288  7
nf_nat                 61440  3 xt_nat,nft_chain_nat,xt_MASQUERADE
nf_conntrack          196608  5 xt_conntrack,nf_nat,xt_nat,nf_conntrack_netlink,xt_MASQUERADE
nf_defrag_ipv6         24576  1 nf_conntrack
nf_defrag_ipv4         12288  1 nf_conntrack
xt_addrtype            12288  4
nft_compat             20480  88
nf_tables             376832  861 nft_compat,nft_chain_nat
libcrc32c              12288  3 nf_conntrack,nf_nat,nf_tables
nfnetlink              20480  5 nft_compat,nf_conntrack_netlink,nf_tables,ip_set
xfrm_user              61440  1
xfrm_algo              16384  1 xfrm_user
exfat                 110592  1
vboxnetadp             28672  0
vboxnetflt             36864  0
vboxdrv               663552  2 vboxnetadp,vboxnetflt
binfmt_misc            24576  1
nls_iso8859_1          12288  1
nvidia_uvm           1413120  0
snd_hda_codec_realtek   200704  1
snd_hda_codec_hdmi     94208  1
snd_hda_codec_generic   122880  1 snd_hda_codec_realtek
snd_hda_intel          61440  2
nvidia_drm             77824  7
nvidia_modeset       1212416  11 nvidia_drm
snd_intel_dspcfg       36864  1 snd_hda_intel
intel_rapl_msr         20480  0
snd_intel_sdw_acpi     16384  1 snd_intel_dspcfg
intel_rapl_common      40960  1 intel_rapl_msr
snd_hda_codec         204800  4 snd_hda_codec_generic,snd_hda_codec_hdmi,snd_hda_intel,snd_hda_codec_realtek
snd_hda_core          139264  5 snd_hda_codec_generic,snd_hda_codec_hdmi,snd_hda_intel,snd_hda_codec,snd_hda_codec_realtek
nvidia              35643392  436 nvidia_uvm,nvidia_modeset
snd_hwdep              20480  1 snd_hda_codec
snd_pcm               192512  4 snd_hda_codec_hdmi,snd_hda_intel,snd_hda_codec,snd_hda_core
joydev                 32768  0
edac_mce_amd           28672  0
snd_seq_midi           24576  0
input_leds             12288  0
snd_seq_midi_event     16384  1 snd_seq_midi
kvm_amd               208896  0
snd_rawmidi            57344  1 snd_seq_midi
kvm                  1413120  1 kvm_amd
snd_seq               118784  2 snd_seq_midi,snd_seq_midi_event
irqbypass              12288  1 kvm
snd_seq_device         16384  3 snd_seq,snd_seq_midi,snd_rawmidi
snd_timer              49152  2 snd_seq,snd_pcm
crct10dif_pclmul       12288  1
polyval_clmulni        12288  0
polyval_generic        12288  1 polyval_clmulni
snd                   143360  15 snd_hda_codec_generic,snd_seq,snd_seq_device,snd_hda_codec_hdmi,snd_hwdep,snd_hda_intel,snd_hda_codec,snd_hda_codec_realtek,snd_timer,snd_pcm,snd_rawmidi
ghash_clmulni_intel    16384  0
sha256_ssse3           32768  0
video                  77824  1 nvidia_modeset
soundcore              16384  1 snd
sha1_ssse3             32768  0
k10temp                16384  0
ccp                   143360  1 kvm_amd
aesni_intel           356352  0
crypto_simd            16384  1 aesni_intel
cryptd                 24576  2 crypto_simd,ghash_clmulni_intel
wmi_bmof               12288  0
rapl                   20480  0
mac_hid                12288  0
sch_fq_codel           24576  2
overlay               212992  1
iptable_filter         12288  0
ip6table_filter        12288  0
ip6_tables             36864  1 ip6table_filter
br_netfilter           32768  0
bridge                421888  1 br_netfilter
stp                    12288  1 bridge
llc                    16384  2 bridge,stp
arp_tables             28672  0
msr                    12288  0
parport_pc             53248  0
ppdev                  24576  0
lp                     28672  0
parport                73728  3 parport_pc,lp,ppdev
efi_pstore             12288  0
ip_tables              32768  1 iptable_filter
x_tables               65536  12 ip6table_filter,xt_conntrack,iptable_filter,nft_compat,xt_tcpudp,xt_addrtype,xt_nat,xt_set,ip6_tables,ip_tables,xt_MASQUERADE,arp_tables
autofs4                57344  2
hid_generic            12288  0
usbhid                 77824  0
uas                    28672  1
hid                   180224  2 usbhid,hid_generic
usb_storage            86016  1 uas
crc32_pclmul           12288  0
r8169                 118784  0
i2c_piix4              32768  0
ahci                   49152  2
xhci_pci               24576  0
realtek                36864  1
libahci                53248  1 ahci
xhci_pci_renesas       20480  1 xhci_pci
wmi                    28672  2 video,wmi_bmof
gpio_amdpt             16384  0
```

### Отключение автозагрузки модуля cdrom

* Добавляем в черный список на автозагрузку модуль cdrom:

```shell
echo "blacklist cdrom" | sudo tee /etc/modprobe.d/blacklist-cdrom.conf
```

* Обновляем конфиги инициализации:

```shell
sudo update-initramfs -u
```

* Перезагружаем ОС

```shell
sudo reboot
```

* Проверяем наличие cdrom в списке загруженных модулей:

```shell
lsmod | grep cdrom
```

Вывод пустой - значит модуль не был загружен.

### Описание конфигурации ядра (параметр CONFIG_XFS_FS)

```shell
grep CONFIG_XFS_FS /boot/config-$(uname -r)
```

Вывод команды:

```
CONFIG_XFS_FS=m
```

Это означает, что поддержка файловой системы xfs осуществляется с помощью модуля. Если бы было значение `y`
это бы значило, что поддержка вкомпиллирована в ядро. Убедимся в наличии модуля xfs:

```shell
modinfo xfs
```

Вывод:

```
filename:       /lib/modules/6.8.0-84-generic/kernel/fs/xfs/xfs.ko
license:        GPL
description:    SGI XFS with ACLs, security attributes, realtime, quota, no debug enabled
author:         Silicon Graphics, Inc.
alias:          fs-xfs
srcversion:     793648A9C7674C6A3069017
depends:        libcrc32c
retpoline:      Y
intree:         Y
name:           xfs
vermagic:       6.8.0-84-generic SMP preempt mod_unload modversions 
sig_id:         PKCS#7
signer:         Build time autogenerated kernel key
sig_key:        17:B4:B2:E0:C0:D9:BA:38:3E:BC:EA:6E:BD:06:7B:B4:38:60:C5:FF
sig_hashalgo:   sha512
signature:      6D:AD:AD:16:DC:78:79:96:6F:2E:26:CE:44:D6:60:1C:55:EB:9D:D3:
                0E:0F:D3:3F:7F:8C:81:F2:62:01:94:6E:73:01:80:C5:99:E9:F5:46:
                A8:52:B5:8E:40:E3:AF:D5:7F:63:5F:BA:71:12:6A:84:3E:89:B6:8F:
                7F:88:D5:10:FE:19:E5:02:34:FE:9E:83:73:AF:1F:84:AC:81:62:BE:
                92:DC:3D:21:07:D7:0E:37:90:E8:10:45:B9:2E:1D:D9:C7:EB:C7:E3:
                04:53:D9:CB:46:69:31:59:BA:48:56:29:C0:DD:49:80:55:14:CD:80:
                D3:4B:BC:05:BA:7E:B9:DA:A4:B2:A3:F9:B2:3E:A2:19:BF:AB:01:53:
                02:18:ED:8C:0B:BF:B4:E7:63:DE:FA:FD:85:D5:84:F5:E1:DC:C7:B6:
                E9:98:93:0C:9D:72:0B:4E:5C:F8:BE:DD:D2:3C:A2:A1:E0:B2:D8:8C:
                0D:09:19:5B:D8:86:FD:19:C8:4E:AC:91:BF:6E:50:3E:B1:88:CC:2E:
                A4:1C:45:B3:EB:96:40:33:4B:5B:90:1D:89:39:90:B4:D7:FD:F2:11:
                34:9A:3C:4A:7C:3B:10:0B:BE:C8:7F:BE:AB:EB:40:92:CF:CC:4B:13:
                5E:DE:E2:D9:EB:48:48:80:76:9F:96:D5:D6:5E:6B:39:EF:5E:2E:77:
                77:C4:BE:71:80:A9:51:F1:84:55:80:3C:D2:51:B6:3C:D2:0E:1B:8D:
                3D:22:98:1A:62:88:D5:3F:43:B5:17:D0:C9:CE:08:7F:66:A0:4C:B6:
                CB:30:A4:50:0E:6A:FC:17:B1:C3:8A:99:48:5B:61:1D:58:2D:70:12:
                43:B6:19:39:C6:AE:8A:72:75:58:C2:54:82:2E:CD:D3:AA:FF:4B:83:
                B4:F2:DF:85:F5:1C:A6:87:F6:62:73:ED:BA:C6:E3:3F:26:B0:25:15:
                DB:30:98:E7:02:B9:0E:BA:BA:A8:0F:2B:53:41:DD:92:E7:CB:74:78:
                DE:13:35:81:6C:4E:4E:51:1D:7D:57:9E:5C:B6:0D:40:34:25:9E:D2:
                7C:6B:5F:84:EB:6E:6B:C1:87:D3:15:FE:51:CF:76:04:7C:FF:4B:5B:
                98:96:77:23:79:0D:63:76:65:47:B5:57:85:BA:1D:BF:23:67:45:03:
                58:5B:93:A6:02:FA:A1:92:35:2E:3D:61:2C:DA:F6:3C:F3:34:99:5E:
                C0:2E:04:36:EC:88:28:28:1B:13:0E:94:D9:E4:BA:A3:23:37:A6:25:
                FB:7F:14:15:E3:11:38:DD:D6:3C:31:DD:8B:08:FF:BA:D2:52:34:BC:
                C6:BA:27:4E:D6:02:16:21:9D:55:CC:67
```