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

## Отключить запрос пароля при монтировании разделов

Создать файл `/etc/polkit-default-privs.local` со следующим содержимым:

```
org.freedesktop.udisks2.filesystem-mount-system auth_admin:auth_admin:yes
```

На некоторых системах расположение файла может быть другим, например `/etc/polkit-default-privs/local`

Выполнить в терминале команду:

```sh
sudo set_polkit_default_privs
```

Источники:

- https://forums.opensuse.org/showthread.php/494354-Authentication-is-required-to-mount-the-device
- https://en.opensuse.org/openSUSE:Security_Documentation#Configuration_of_Polkit_Settings
- https://forums.opensuse.org/showthread.php/504610-PolicyKit-gui
- https://cialu.net/stop-asking-password-when-mounting-another-internal-hard-drive-on-linux/

## Включить дробное масштабирование для сеанса GNOME Wayland

Выполнить в терминале команду:

```sh
gsettings set org.gnome.mutter experimental-features "['scale-monitor-framebuffer']"
```

Затем перезапустить сеанс

Источник: https://wiki.archlinux.org/title/HiDPI#Wayland

## Включить поддержку Wayland в приложениях на базе Chromium

### Браузеры

Установить флаг `chrome://flags/#ozone-platform-hint` в `Auto` или `Wayland`

### Electron-приложения

Для каждого приложения по отдельности - с помощью флагов `--enable-features=UseOzonePlatform --ozone-platform=wayland`

Для всех приложений - с помощью файла `${XDG_CONFIG_HOME}/electron-flags.conf` (обычно `~/.config/electron-flags.conf`):

```
--enable-features=UseOzonePlatform
--ozone-platform=wayland
```

Источники:

- https://wiki.archlinux.org/title/wayland#Per_application
- https://wiki.archlinux.org/title/wayland#Per_user
- https://wiki.archlinux.org/title/chromium#Native_Wayland_support
