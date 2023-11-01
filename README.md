# RPi4_HA_Manual_2023

## Установка HA, включая Supervised и Addons в виде контейнера на Raspberry Pi 4 x64 Lite(CLI, Not GUI)
1. Установка Portainer вместе с Docker #потом расписать _(ууупс, см. п. 3 - лучше там установить докер скриптом. UPD: выполнил п.3 - ничего не сломалось)_

2. Начинаем устанавливать зависимости в RPi по манулу:
  https://github.com/home-assistant/supervised-installer
  (может появляться сообщение New Kernel Upgrade - просто перезагружаемся и повторяем установку "проблемного" компонента)
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
3. Устанавлвиаем docker-ce: #(если пропустить, то в п.5 будет ошибка, что он не установлен)
   ``` curl -fsSL get.docker.com | sh ```
4. Устанавливаем OS-Agent: # https://github.com/home-assistant/os-agent/tree/main#using-home-assistant-supervised-on-debian
  - 1. Скачиваем последний релиз отсюда https://github.com/home-assistant/os-agent/releases (Использовать aarch64 и не использовать armv7).
  - 2. Устанавливаем командой ```sudo dpkg -i os-agent_1.0.0_linux_x86_64.deb```.
  - 3. *Можно проверить правильность установки. Следующая команда НЕ должна вызвать ошибок - ``` gdbus introspect --system --dest io.hass.os --object-path /io/hass/os ```.
5. Устанавливаем Home Assistant Supervised Debian Package: (а тут уже выбираем Raspberry Pi4 x64) 
  ```
  wget -O homeassistant-supervised.deb https://github.com/home-assistant/supervised-installer/releases/latest/download/homeassistant-supervised.deb
  sudo apt install ./homeassistant-supervised.deb
  ```
6. Готово. Заходим на ```http://<host>:8123``` (host=IP RPi4) и приступаем к первичной настройке Home Assistant.
>[!NOTE]
>Первый запуск в скриншотах - 7 ~~нажатий next~~ шагов к успеху
>https://www.home-assistant.io/getting-started/onboarding/

---
Всё бы ничего, но я забыл про сам HA:
https://www.home-assistant.io/installation/linux#install-home-assistant-container
(Пробую ставить после Supervised по манулу. UPD: Работает. Всё работает при такой корявой установке)

## Установка HA в виде контейнера в docker
 Для установки Home Assistant достаточно использовать следующую команду:
  - ```/PATH_TO_YOUR_CONFIG``` - директория в которой будет храниться конигурации
  - ```TZ=...``` - необходимо указать часовый пояс из списка https://en.wikipedia.org/wiki/List_of_tz_database_time_zones. Например, ```Europe/Moscow``` для Москва GMT+3.
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

## Обновление HA в виде контейнера:
1. Проверяем обновления:
  - ```sudo docker pull ghcr.io/home-assistant/home-assistant:stable```
_Если будет статус "Image is up to date", то далее можно не продолжать_
2. Останавливаем запущенный контейнер:
  - ```sudo docker stop homeassistant```
4. Удаляем старый контейнер:
  - ```sudo docker rm homeassistant```
5. Запускаем обновлённый контейнер:
  - ```/PATH_TO_YOUR_CONFIG``` - директория в которой будет храниться конфигурации
  - ```TZ=...``` - необходимо указать часовый пояс из списка https://en.wikipedia.org/wiki/List_of_tz_database_time_zones. Например, ```Europe/Moscow``` для Москва GMT+3.
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

> [!IMPORTANT]
> Если запущен контейнер Portainer, то Home Assistant будет жаловаться о неправильной конфигурации и отказываться устанавливать некоторые аддоны. Достаточно остановить Portainer и максимум перезагрузить контейнер Supervised(сам HA перезагружаться не будет). После установки можно включить Portainer, а в HA прожать игнорировать на данном предупреждении. Пока не изучал почему так. Благо можн за раз всё установить и больше этого не делать.

---

## Установка дополнений:
- [ ] HACS (Home Assistant Community Store - установка дополнений прямо из гитхаба) https://hacs.xyz/
- [ ] MQTT Mosquitto (MQTT-брокер)
- [ ] Zigbee2MQTT (поддержка zigbee через MQTT без проприетарных хабов\мостов, даже дешёвый USB-свисток подойдёт). Список поддерживаемых адаптеров - https://www.zigbee2mqtt.io/guide/adapters/
- [ ] File-editor(?)
- [ ] SSH(?)
