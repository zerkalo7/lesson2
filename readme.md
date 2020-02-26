## OTUS-Linux-Lesson 2
# Задание основное и *

Добавляем диск в Vagrantfile 
```
,
                :sata5 => {
                        :dfile => './sata5.vdi',
                        :size => 250, 
                        :port => 5
                }
```

После сборки системы (vagrant up):  
Собираем R5
```
mdadm -Cv /dev/md0 -l 5 -n 5 /dev/sd{b..f}
```

Прописываем в конфиг
```
sudo mkdir /etc/mdadm
echo "DEVICE partitions" | sudo tee -a /etc/mdadm/mdadm.conf
sudo mdadm --detail -sv | awk '/ARRAY/{print}' | sudo tee -a /etc/mdadm/mdadm.conf
```

Создаем GPT, 5 партиций и маунтим
```
sudo parted -s /dev/md0 mklabel gpt
sudo parted /dev/md0 mkpart primary ext4 0% 20%
sudo parted /dev/md0 mkpart primary ext4 20% 40%
sudo parted /dev/md0 mkpart primary ext4 40% 60%
sudo parted /dev/md0 mkpart primary ext4 60% 80%
sudo parted /dev/md0 mkpart primary ext4 80% 100%
for i in $(seq 1 5); do sudo mkfs.ext4 /dev/md0p$i; done
sudo mkdir -p /raid/part{1..5}
for i in $(seq 1 5); do sudo mount /dev/md0p$i /raid/part$i; done 
```

Для того, чтобы создание рейда происходило при сборке - добавляем всё это в Vagrantfile в shell provisioner



#Задание с **  

*задаем пароль для root, чтобы сделать su*  
sudo passwd  
su  
*выключаем SELINUX, чтобы он не обидился при перезагрузке по rsync на другой диск*  
nano /etc/selinux/config  
**SELINUX=disabled**  
*копируем разделы на второй диск и прописываем, что это Linux Raid*  
sfdisk -d /dev/sda | sfdisk /dev/sdb && fdisk /dev/sdb  
**t (меняем тип), fd (на Linux Raid), w (записываем изменения)**  
*создаем рейд без одного диска, создаём на нём файловую систему, делаем на него копию, меняем рут*  
mdadm --create /dev/md0 --level=1 --raid-devices=2 missing /dev/sdb1 && mkfs.xfs /dev/md0 && mount /dev/md0 /mnt/ && rsync -axu / /mnt/ && mount --bind /proc /mnt/proc && mount --bind /dev /mnt/dev && mount --bind /sys /mnt/sys && mount --bind /run /mnt/run && chroot /mnt/  
*заменям в fstab UUID sda на UUID md0*  
ls -l /dev/disk/by-uuid |grep md >> /etc/fstab && nano /etc/fstab  
*создаем конфиг, чтобы md0 не сменил имя при перезагрузке, бэкапим старый initramfs и делаем новый*  
mdadm --detail --scan > /etc/mdadm.conf && cp /boot/initramfs-$(uname -r).img /boot/initramfs-$(uname -r).img.bck && dracut /boot/initramfs-$(uname -r).img $(uname -r) --force  
*добавляем опцию в GRUB*  
nano /etc/default/grub  
**добавляем в GRUB_CMDLINE_LINUX = rd.auto=1**  
*переписываем конфиг и ставим его на sdb*  
grub2-mkconfig -o /boot/grub2/grub.cfg && grub2-install /dev/sdb  
*выходим из chroot*  
exit  
*выключаем машину*  
shutdown now  
** !при загрузке выбираем второй диск**
su
*прописываем, что sda это Linux Raid*
fdisk /dev/sda
**t (меняем тип), fd (на Linux Raid), w (записываем изменения)**
*ставим GRUB config и на sda*
grub2-install /dev/sda
*добавляем второй диск в R1*
mdadm --manage /dev/md0 --add /dev/sda1
*ждем сборки рейда через cat /proc/mdstat и перезагружаемся*

