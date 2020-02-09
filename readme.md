## OTUS-Linux-Lesson 2
# Задание основное и * (без **)    

Добавляем диск в Vagrantfile 
```
,
                :sata5 => {
                        :dfile => './sata5.vdi',
                        :size => 250, 
                        :port => 5
                }
```

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
