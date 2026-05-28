# Короткий мануал запуска компьютера Repka Pi 4 сразу на SSH без подключения к монитору. 
=======
# Инструкция по запуску Repka Pi с SSH без монитора

Исходные данные:
 - Repka Pi 4 Optimal (2 ГБ ОЗУ, eMMC 64 ГБ)
 - Флешка microSD
 - Доступ к Wi-Fi сети
 - Компьютер с Ubuntu для подготовки флешки

У меня два компьютера с win и ubuntu как менять файлы на флешке через win я не нашел, поэтому пришлось слегка с буМном полясать с перетаскиванием флешек.
Данная инструкция не решение проблемы. Возможно какие-то опции будут лишние, делитесь своим опытом, поправим инструкцию.

## Шаг 1: Подготовка флешки через Rufus
Записываем образ Armbian на флешку (процесс стандартный, опускаем).

## Шаг 2: Создание файлов на флешке для SSH и Wi-Fi
После записи образа флешка монтируется в /media/username/RepkaPi-4-OS/

```shell
# Переходим в boot-директорию
cd /media/user/RepkaPi-4-OS/boot


# Создаём пустой файл ssh (активирует SSH-сервер при загрузке)
sudo touch ssh

# Создаём конфиг Wi-Fi
sudo nano wpa_supplicant.conf
```

#### Содержимое wpa_supplicant.conf:

```
ctrl_interface=DIR=/var/run/wpa_supplicant GROUP=netdev
update_config=1
country=RU

network={
    ssid="ИМЯ_СЕТИ"
    psk="ПАРОЛЬ"
    key_mgmt=WPA-PSK
}
```

## Шаг 3: Отключение GUI и настройка автоподключения Wi-Fi

Эти шаги мы делали после первого входа, но лучше сделать их заранее на флешке:

```shell
# Создаём конфиг для NetworkManager (автоподключение к Wi-Fi)
sudo mkdir -p /media/user/RepkaPi-4-OS/etc/NetworkManager/system-connections

sudo tee /media/user/RepkaPi-4-OS/etc/NetworkManager/system-connections/WiFi.nmconnection > /dev/null << 'EOF'
[connection]
id=ИМЯ_СЕТИ
type=wifi
interface-name=wlan0
autoconnect=true
autoconnect-priority=10

[wifi]
mode=infrastructure
ssid=ИМЯ_СЕТИ

[wifi-security]
key-mgmt=wpa-psk
psk=ПАРОЛЬ

[ipv4]
method=auto

[ipv6]
method=auto
EOF

sudo chmod 600 /media/user/RepkaPi-4-OS/etc/NetworkManager/system-connections/WiFi.nmconnection

# Отключаем графический интерфейс GDM
sudo rm -f /media/user/RepkaPi-4-OS/etc/systemd/system/display-manager.service
sudo ln -sf /dev/null /media/user/RepkaPi-4-OS/etc/systemd/system/gdm.service

# Устанавливаем консольный режим по умолчанию
sudo ln -sf /lib/systemd/system/multi-user.target /media/user/RepkaPi-4-OS/etc/systemd/system/default.target

# Включаем SSH в автозагрузку
sudo ln -sf /lib/systemd/system/ssh.service /media/user/RepkaPi-4-OS/etc/systemd/system/multi-user.target.wants/ssh.service
```

## Шаг 4: Другие настройки (опционально)

```shell
# Фиксированный IP (если нужен)
# Создаём /media/user/RepkaPi-4-OS/etc/network/interfaces.d/eth0
# Или настраиваем через NetworkManager позже

# Включаем последовательную консоль (на случай проблем)
sudo sed -i 's/console=serial0,115200/console=ttyS0,115200/' /media/user/RepkaPi-4-OS/boot/armbianEnv.txt
```

## Шаг 5: Завершение подготовки

```shell
# Размонтируем флешку
cd ~
sudo umount /media/user/RepkaPi-4-OS
sync
```

Вставляем флешку в Repka Pi, подключаем питание.

## Шаг 6: Поиск в сети и подключение
Ждём 2-3 минуты. Находим IP через роутер (192.168.0.1 → DHCP-клиенты) и подключаемся:

```shell
ssh root@НАЙДЕННЫЙ_IP
# Пароль: 1234
```

Дальше лучше сменить пароль и имя пользователя. 
