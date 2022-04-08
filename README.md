# Linux: Послеустановочные шаги

## Установка earlyoom

Arch Linux, Manjaro Linux

```sh
sudo pacman -S earlyoom
sudo systemctl enable earlyoom.service
sudo systemctl start earlyoom.service
```

Debian GNU/Linux, Ubuntu, Linux Mint

```sh
sudo apt install earlyoom
sudo systemctl enable earlyoom.service
sudo systemctl start earlyoom.service
```

## Запретить устройствам выводить компьютер из спящего режима

Создать файл `/etc/systemd/system/acpi-wake.service`:

```
[Unit]
Description=ACPI Wake Service

[Service]
Type=oneshot
ExecStart=/bin/sh -c "for i in $(cat /proc/acpi/wakeup|grep enabled|awk '{print $1}'|xargs); do [ '$i' != PWRB ] && echo $i|tee /proc/acpi/wakeup;done"

[Install]
WantedBy=multi-user.target
```

Включить сервис

```sh
systemctl start acpi-wake.service
systemctl enable acpi-wake.service
```

Этот сервис запрещает всем устройствам, кроме кнопки питания (`PWRB`, название может отличаться), выводить компьютер из спящего режима. Список всех устройств с такой возможностью можно посмотреть командой `cat /proc/acpi/wakeup`

Источник: https://ubuntu-mate.community/t/power-management-acpi-wake-up-settings/17344

## Настроить параметры виртуальной памяти

Для компьютеров с ОЗУ > 8 ГиБ можно уменьшить значения `vm.dirty_ratio` и `vm.dirty_background_ratio`

Для этого нужно создать файл `/etc/sysctl.d/60-dirty.conf` со следующим содержимым:

```
vm.dirty_background_ratio=5
vm.dirty_ratio=10
```

Источники:

- https://habr.com/ru/company/selectel/blog/498526/#comment_21533708
- https://www.suse.com/support/kb/doc/?id=000017857
- https://lonesysadmin.net/2013/12/22/better-linux-disk-caching-performance-vm-dirty_ratio/
