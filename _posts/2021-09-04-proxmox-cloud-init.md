---
layout: post
title: "Proxmox Cloud-Init"
subtitle: "Подготовка шаблона ВМ с Cloud-Init для Proxmox."
date: 2021-09-03 00:00:00 -0400
background: '/img/posts/01.jpg'
---

#### Подготовка шаблона

Скачиваем образ cloud image Ubuntu 20.04:
```
wget https://cloud-images.ubuntu.com/focal/current/focal-server-cloudimg-amd64.img
```
Создаем ВМ, импортируем скачанный образ в local-lvm, подключаем диск в созданную машину, увеличиваем размер диска:
```
qm create 9020 --memory 2048 --net0 virtio,bridge=vmbr0,tag=18 --agent enabled=1 --cores 2 --ostype l26 --name ubuntu20-template
qm importdisk 9020 focal-server-cloudimg-amd64.img local-lvm
qm set 9020 --scsihw virtio-scsi-pci --scsi0 local-lvm:vm-9020-disk-0
qm resize 9020 scsi0 +8G
```

Добавляем устройство CloudInit Drive, устанавливаем загрузочное устройство, устанавливаем отображание консоли через xterm.js: 
```
qm set 9020 --scsi2 local-lvm:cloudinit
qm set 9020 --boot c --bootdisk scsi0
qm set 9020 --serial0 socket --vga serial0
```

---
Далее через веб интерфейс настраиваем остальные параметры.  
Во вкладке Cloud-Init:
- User: `srv-admin`
- Password: `superpassword`
- DNS domain: `corp.company.com`
- DNS servers: `10.0.1.11,10.0.1.12`
- SSH pubic key: `<открытый ключ>`
- IP Config: IPv4: `Static`, IPv4/CIDR: `10.0.8.251/24`, Gateway: `10.0.8.1`

Либо командой:
```
qm set 9020 --ciuser srv-admin --cipassword srv-password --searchdomain corp.company.kz --nameserver 10.0.1.11,10.0.1.12 --ipconfig0 ip=10.0.8.251/24,gw=10.0.8.1 --sshkeys /path/to/id_rsa.pub
```

Включаем машину 

#### Донастраиваем систему


Обновляем систему
```
sudo apt update && sudo apt dist-upgrade -y
```
Включаем bash autocompletion root-пользователя, раскомментировав эти строки в файле `/root/.bashrc`:
```
if [ -f /etc/bash_completion ] && ! shopt -oq posix; then
    . /etc/bash_completion
fi
```
Устанавливаем правильный часовой пояс, в моем случае `Asia/Almaty`
```
sudo dpkg-reconfigure tzdata
```

Конвертируем в шаблон