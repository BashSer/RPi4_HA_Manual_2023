# RPi4_HA_Manual_2023

  > [!NOTE]  
  > Вы можете просто загрузить готовый образ Home Assistant OS из этого же инсталлера и весь накопитель данных будет размечен под готовую операционную систему со всеми возможностями. Просто включить и перейти по `homeassistant:8123` или `homeassistant.local:8123` или `http://X.X.X.X:8123` (замените X.X.X.X на IP-адрес Raspberry Pi).


## Table of Contents:
* [Установка Raspberry Pi OS Lite (64-bit, CLI, Not GUI) на Raspberry Pi 3/4/400/5 x64](#iso)
* [Обновление и базовая конфигурация OS](#OS)
* [Установка HA, включая Supervised и Addons в виде контейнера на Raspberry Pi 4 x64 Lite(CLI, Not GUI)](#software)
  * [Возникшие проблемы](#problems)
* [Установка Addons](#addons)
  * [HACS (Home Assistant Community Store)](#HACS)
  * [MQTT Mosquitto](#MQTT)
  * [Zigbee2MQTT](#zigbee)
  * [Node-RED](#node-red)
  * [ESPhome](#esphome)
  * [MQTT-io](#mqtt-io)
  * []()
  * []()
* [Установка интеграций](#integrations)

---

<a name="iso"></a>
## Установка Raspberry Pi OS Lite (64-bit, CLI, Not GUI) на Raspberry Pi 3/4/400/5 x64 
  1. Скачиваем и запускаем:
     
     https://www.raspberrypi.com/software/
     
     ![image](https://github.com/BashSer/RPi4_HA_Manual_2023/assets/37932617/97e75b93-86fe-4f04-bd65-ec59339ed30d)

  2. Выбираем Raspberry Pi OS Lite (64-bit), выбираем накопитель данных и записываем:
     
     ![image](https://github.com/BashSer/RPi4_HA_Manual_2023/assets/37932617/43f57fa7-a905-4872-8fa1-33b250614c7f)

  >[!NOTE]
  >В дополнительных настройках включить SSH к OS и Wi-Fi:
  >![image](https://github.com/BashSer/RPi4_HA_Manual_2023/assets/37932617/dda00ac6-f8d9-436d-a823-1522ed64b94d)
  
  3. Установить накопитель данных в Raspberry Pi 3/4/400/5, включить и оставить на 2-3 минуты(RPi4 8Gb), пока она инициализируется. После чего можно подключиться по SSH.

---

<a name="OS"></a>
## Обновление и базовая конфигурация OS:
Обновление репозиториев, установка с авто-соглашением и перезагрузка:
```
sudo apt update && sudo apt upgrade -y
sudo reboot now
```
_добавить про перепрошивку?_

---


<a name="software"></a>
### Установка HA, включая Supervised и Addons в виде контейнеров на Raspberry Pi 4 x64 Lite(CLI, Not GUI)

  1. Устанавлвиаем docker-ce:
  ```
  curl -fsSL get.docker.com | sudo sh
  sudo reboot now
  ```
  2. Устанавливаем все необходимые зависимости в RPi OS:
    https://github.com/home-assistant/supervised-installer
  ```
  sudo apt install \
  apparmor \
  cifs-utils \
  curl \
  dbus \
  jq \
  libglib2.0-bin \
  lsb-release \
  network-manager \
  nfs-common \
  systemd-journal-remote \
  systemd-resolved \
  udisks2 \
  wget -y
  ```

### Установка HA в виде контейнера в docker
  Для установки Home Assistant достаточно использовать следующую команду:
  - ```/PATH_TO_YOUR_CONFIG``` *- директория в которой будет храниться конфигурации*
  - ```TZ=...``` *- необходимо указать часовый пояс из списка https://en.wikipedia.org/wiki/List_of_tz_database_time_zones. Например, ```Europe/Moscow``` для Москва GMT+3.*
```
sudo docker run -d \
  --name homeassistant \
  --privileged \
  --restart=unless-stopped \
  -e TZ=Europe/Moscow \
  -v /PATH_TO_YOUR_CONFIG:/config \
  --network=host \
  ghcr.io/home-assistant/home-assistant:stable
```

#### Обновление HA в виде контейнера:
  1. Проверяем обновления:
```
sudo docker pull ghcr.io/home-assistant/home-assistant:stable
```
  _Если будет статус "Image is up to date", то далее можно не продолжать_
  
  2. Останавливаем запущенный контейнер:
```
sudo docker stop homeassistant
```
  3. Удаляем старый контейнер:
```
sudo docker rm homeassistant
```
  4. Запускаем обновлённый контейнер командой из установки выше.


### Устанавливаем OS-Agent по инструкции https://github.com/home-assistant/os-agent/tree/main#using-home-assistant-supervised-on-debian:

  1. Скачиваем последний релиз отсюда:
    
  https://github.com/home-assistant/os-agent/releases
    
  *(Скачать aarch64 и не использовать armv7 в данном случае)*
```
wget https://github.com/home-assistant/os-agent/releases/download/1.6.0/os-agent_1.6.0_linux_aarch64.deb
```
    
  2. Устанавливаем командой:
```
sudo dpkg -i os-agent_1.6.0_linux_aarch64.deb
```
  >[!NOTE]
  >Можно проверить правильность установки
  >
  >Следующая команда НЕ должна вызвать ошибок:
  >```
  >gdbus introspect --system --dest io.hass.os --object-path /io/hass/os
  >```

### Устанавливаем Home Assistant Supervised Debian Package:
>[!IMPORTANT]
>Тут уже выбираем Raspberry Pi4 x64
```
wget -O homeassistant-supervised.deb https://github.com/home-assistant/supervised-installer/releases/latest/download/homeassistant-supervised.deb
sudo apt install ./homeassistant-supervised.deb
sudo reboot now
```

Готово. Заходим на ```http://<host>:8123``` (host=IP RPi4) и приступаем к первичной настройке Home Assistant.

> [!NOTE]
>Первый запуск в скриншотах - 7 ~~нажатий next~~ шагов к успеху
>https://www.home-assistant.io/getting-started/onboarding/

---

<a name="problems"></a>
###Возникшие проблемы:

> [!NOTE]
> Если в HA будет предупреждение `Unsupported system - AppArmor issues`, то это можно починить так:
> ```
>sudo nano /boot/cmdline.txt
> ```
> в конце дописать `lsm=apparmor`, сохранить <kbd>Ctrl</kbd> + <kbd>S</kbd>, закрыть <kbd>Ctrl</kbd> + <kbd>X</kbd> и выполнить полную перезагрузку:
> ```
> sudo reboot now
> ```

> [!IMPORTANT]
> Если запущен контейнер Portainer, то Home Assistant будет жаловаться о неправильной конфигурации и отказываться устанавливать некоторые аддоны. Достаточно остановить Portainer и максимум перезагрузить контейнер Supervised(сам HA перезагружаться не будет). После установки можно включить Portainer, а в HA прожать игнорировать на данном предупреждении. Пока не изучал почему так. Благо можн за раз всё установить и больше этого не делать.

---

<a name="addons"></a>
## Установка дополнений:
>[!WARNING]
>Не забудьте включить "Расширенный режим" в настройках профиля.

<a name="HACS"></a>
- [x] HACS (Home Assistant Community Store)
  https://hacs.xyz/
  >[!NOTE]
  >Требуется учётная запись github
  - Настройки -> Дополнения
  - Внизу справа Магазин дополнений
  - Найти `Terminal & SSH`
  - Установить, Веб-интерфейс
  - `wget -O - https://get.hacs.xyz | bash -` (<kbd>CTRL</kbd> + <kbd>SHIFT</kbd> + <kbd>V</kbd>)
  - Перезагрузить Home Assistant
  - Зайти в веб-интерфейс Home Assistant
  >[!WARNING]
  >Обновить страницу с чисткой кэша <kbd>CTRL</kbd> + <kbd>F5</kbd>
  - Настройки -> Устройства и службы
  - Внизу справа Добавить интеграцию
  - Найти `HACS`
  - Ознакомиться со всеми пунктами и Подтвердить
  - Открыть ссылку подключения github и ввести предоставленный код авторизации в своей учётной записи
  - Готово. Теперь в боковй меню появился пункт `HACS`

<a name="MQTT"></a>
- [X] MQTT Mosquitto (MQTT-брокер)
  - Настройки -> Дополнения
  - Внизу справа Магазин дополнений
  - Найти `Mosquitto broker`
  - Установить, Запустить
  >[!NOTE]
  > | Port | Description |
  > | :----: | :---------: |
  > | `1883` | Normal MQTT |
  > | `1884` | MQTT over WebSocket |
  > | `8883` | Normal MQTT with SSL |


<a name="zigbee"></a>
- [X] Zigbee2MQTT
  >[!NOTE]
  >https://www.zigbee2mqtt.io/
  >https://github.com/zigbee2mqtt/hassio-zigbee2mqtt#installation
  - Настройки -> Дополнения
  - Внизу справа Магазин дополнений
  - Наверху справа открыть меню в виде трёх точек и выбрать Репозитории
  - Вставить ссылку `https://github.com/zigbee2mqtt/hassio-zigbee2mqtt` и Добавить
  >[!WARNING]
  >Обновить страницу с чисткой кэша <kbd>CTRL</kbd> + <kbd>F5</kbd>
  - Настройки -> Дополнения
  - Внизу справа Магазин дополнений
  - Найти `Zigbee2MQTT`(master ветка, стабильная, редко обновляется) или `Zigbee2MQTT Edge`(dev ветка, часто обновляется, удобно во время тестов при решении Issue)
  - Установить, Конфигурация
  - В блоке `serial:` вписать порт USB в который установлен zigbee-свисток `port: /dev/ttyUSB0`
  - Запустить

<a name="file-editor"></a>
- [X] File-editor
  - Настройки -> Дополнения
  - Внизу справа Магазин дополнений
  - Найти `File editor`
  - Установить
  - Готово
  >[!NOTE]
  >Можно включить отоборажение на боковой панели слева

<a name="node-red"></a>
- [ ] Node-RED
  >[!WARNING]
  > Требуется SSL ?
  - Настройки -> Дополнения
  - Внизу справа Магазин дополнений
  - Найти `Node-RED`
  - Установить

<a name="esphome"></a>
- [ ] ESPHome(?)(Раньше для ПЕРВОГО запуска требовалась ESP подключенная по USB - поправили?)


<a name="mqtt-io"></a>
- [ ] MQTT-IO(что-то новое?)


- [ ] AppDaemon(?)(Python Apps and Dashboard)
- [ ] Duck DNS(?)
- [ ] Let's Encrypt(?)
- [ ] Rclone Backup(?)


<a name="integrations"></a>
- [ ] 
