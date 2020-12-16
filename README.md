##Работа с загрузчиком
Создаем Vagrantfile с системой на lvm
```sh
Vagrant.configure("2") do |config|
  config.vm.box = "R2DXT/centos-7-5"
  config.vm.box_version = "1.0"
end
```
запускаем виртуалку
```sh
vagrant up
``` 
##1. Попасть в систему без пароля несколькими способами
1.1. Простейший вариант загрузится с диска или флешки в режиме Rescue Mode; 
1.2. Доходим до GRUB. Жмем "е" - находим строку "linux16 /vmlinuz-...." дописываем systemd.debug-shell жмем Ctrl+X переключаемся на TTY9: Ctrl + Alt + 9;
1.3. Доходим до GRUB. Жмем "е" - находим строку "linux16 /vmlinuz-...." заменяем "ro" на "rw" убираем quiet и дописываем rd.break жмем Ctrl+X;
##2. Установить систему с LVM, после чего переименовать VG
Смотрим исходное состояние и меняем имя VG
```sh
sudo -s 
vgs
  VG         #PV #LV #SN Attr   VSize   VFree
  VolGroup00   1   2   0 wz--n- <38.97g    0
vgrename VolGroup00 VChangeName
  Volume group "VolGroup00" successfully renamed to "VChangeName"
```
Вносим с помощью изменения в /boot/grub2/grub.cfg, /etc/default/grub, /etc/fstab где заменяем "VolGroup00" на "VChangeName", проверяем и пересоздаем initrd
```sh
cat /boot/grub2/grub.cfg /etc/default/grub /etc/fstab | grep V
ChangeName
        linux16 /vmlinuz-3.10.0-862.2.3.el7.x86_64 root=/dev/mapper/VolGroup00-LogVol00 ro n
o_timer_check console=tty0 console=ttyS0,115200n8 net.ifnames=0 biosdevname=0 elevator=noop 
crashkernel=auto rd.lvm.lv=VChangeName/LogVol00 rd.lvm.lv=VChangeName/LogVol01 rhgb quiet   
GRUB_CMDLINE_LINUX="no_timer_check console=tty0 console=ttyS0,115200n8 net.ifnames=0 biosdev
name=0 elevator=noop crashkernel=auto rd.lvm.lv=VChangeName/LogVol00 rd.lvm.lv=VChangeName/L
ogVol01 rhgb quiet"
/dev/mapper/VChangeName-LogVol00 /                       xfs     defaults        0 0 
mkinitrd -f -v /boot/initramfs-$(uname -r).img $(uname -r)
*** Creating initramfs image file '/boot/initramfs-3.10.0-862.2.3.el7.x86_64.img' done ***
```
и перезагружаемся
```sh
reboot 
```
Проверяем
```sh
sudo -s
vgs
  VG         #PV #LV #SN Attr   VSize   VFree
  VChangeName   1   2   0 wz--n- <38.97g    0
```
##3. Добавить модуль в initrd
Создаем дир для модуля
```sh
mkdir /usr/lib/dracut/modules.d/01test
```
качаем файлы примера и кладем в дир
```sh
yum install git -y 
git clone https://github.com/R2DXT/boot
cd boot 
cp mod* te* /usr/lib/dracut/modules.d/01test
```
пересобираем initrd и перезагружаемся
```sh
dracut -f -v
*** Creating initramfs image file '/boot/initramfs-3.10.0-862.2.3.el7.x86_64.img' done ***
reboot
```
Доходим до GRUB. Жмем "е" - находим строку "linux16 /vmlinuz-...." убираем rghb quiet
и видим в выводе терминала пингвина!
