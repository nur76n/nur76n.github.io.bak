---
layout: post
title: "Multipath"
subtitle: "Установка и настройка"
background: '/img/posts/multipath.png'
---

## Установка

```
apt install multipath-tools
```
Включаем автозапуск и стартуем службу
```
systemctl enable multipath-tools
systemctl start multipath-tools
```
Теперь приступаем к настройке.
  
  

## Настройка
Добавляем wwid устройств в белый список
```
multipath -a "0x20150013780e8b80"
multipath -a "0x201b0013780e8b80"
```

Редактируем файл `/etc/multipath.conf `
```
blacklist {
    wwid .*
}

blacklist_exceptions {
    wwid "0x20150013780e8b80"
    wwid "0x201b0013780e8b80"
}

multipaths {
    multipath {
        wwid "0x20150013780e8b80"
        alias mpath_san2_vol1
    }

    multipath {
        wwid "0x201b0013780e8b80"
        alias mpath_san2_vol2
    }
}

defaults {
    polling_interval        2
    prio                    alua
    uid_attribute           ID_WWN
    rr_min_io               100
    user_friendly_names     yes
    path_grouping_policy    failover
    path_selector           "round-robin 0"
    no_path_retry           5
    failback                immediate
    rr_weight               priorities
}
```

Применяем настройки
```
multipathd -k"reconfigure"
```
Проверяем состояние и дебаг информацию
```
multipath -ll
multipath -v4
```
----

#### Настройка LVM
При запуске выполняется команда vgscan, которая выполнит поиск меток LVM на блочных устройствах системы с целью определения того, какие из них представляют собой физические тома, а также получения метаданных и создания списков групп томов. Имена физических томов хранятся в файле `/etc/lvm/.cache` на каждом узле в системе. Последующие команды будут обращаться к этому файлу, при этом не будет необходимости в повторном сканировании.

С помощью фильтров, определяемых в `lvm.conf`, можно управлять тем, какие устройства будут сканироваться. Фильтры представляют собой набор регулярных выражений, применяемых к именам устройств в каталоге `/dev` с целью разрешения или запрета определения блочного устройства.

Приведенные ниже примеры демонстрируют использование фильтров. Следует отметить, что некоторые примеры не являются лучшими решениями, так как регулярные выражения свободно сопоставляются с полными путями. Например, `a/.*loop.*/` соответствует не только `a/loop/`, но и `/dev/solooperation/lvol1`.

Следующий фильтр добавит все найденные устройства, что является поведением по умолчанию в случае, если фильтры не заданы.

Редактируем файл `/etc/lvm/lvm.conf`: 

Комментим эту строку
```
global_filter = [ "r|/dev/zd.*|", "r|/dev/mapper/pve-.*|" "r|/dev/mapper/.*-(vm|base)--[0-9]+--disk--[0-9]+|"]
```

И добавляем эту
```
global_filter = [ "a|/dev/sda|", "a|/dev/sdb|", "a|/dev/mapper/mpath.*|", "r|/dev/sd.*|", "r|/dev/zd.*|", "r|/dev/mapper/pve-.*|", "r|/dev/mapper/.*-(vm|base)--[0-9]+--disk--[0-9]+|"]
```
или эту (где запрещаем по модели SAN)
```
global_filter = ["r|/dev/disk/by-id/scsi-SQsan_XS3226.*|", "r|/dev/zd.*|", "r|/dev/mapper/pve-.*|", "r|/dev/mapper/.*-(vm|base)--[0-9]+--disk--[0-9]+|"]
```

```
pvscan
lsblk
```

#### Полезные ссылки

* [Статья от RedHat о фильрах LVM](https://access.redhat.com/documentation/ru-ru/red_hat_enterprise_linux/5/html/logical_volume_manager_administration/lvm_filters)
* [Корректность фильтра можно проверить на сайте](https://regex101.com/)
* [Конфигурационный файл DM-Multipath
](https://help.ubuntu.ru/wiki/%D1%80%D1%83%D0%BA%D0%BE%D0%B2%D0%BE%D0%B4%D1%81%D1%82%D0%B2%D0%BE_%D0%BF%D0%BE_ubuntu_server/%D0%BC%D0%BD%D0%BE%D0%B6%D0%B5%D1%81%D1%82%D0%B2%D0%B5%D0%BD%D0%BD%D0%BE%D0%B5_%D1%81%D0%B2%D1%8F%D0%B7%D1%8B%D0%B2%D0%B0%D0%BD%D0%B8%D0%B5_%D1%83%D1%81%D1%82%D1%80%D0%BE%D0%B9%D1%81%D1%82%D0%B2/configuration)

