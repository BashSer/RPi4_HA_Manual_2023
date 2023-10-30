# RPi4_HA_Manual_2023
Установка HA в виде контейнера на Raspberry Pi 4 x64 Lite(CLI, No GUI)

1. Установка Portainer вместе с Docker #потом расписать (ууупс, см. п. 3 - лучше там установить докер скриптом. UPD: выполнил п.3 - ничего не сломалось)

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
  4.1. Скачиваем последний релиз отсюда https://github.com/home-assistant/os-agent/releases (Использовать aarch64 и не использовать armv7).
  4.2. Устанавливаем командой ```sudo dpkg -i os-agent_1.0.0_linux_x86_64.deb```.
  4.3. *Можно проверить правильность установки. Следующая команда НЕ должна вызвать ошибок - ``` gdbus introspect --system --dest io.hass.os --object-path /io/hass/os ```.
5. Устанавливаем Home Assistant Supervised Debian Package: (а тут уже выбираем Raspberry Pi4 x64) 
  ```
  wget -O homeassistant-supervised.deb https://github.com/home-assistant/supervised-installer/releases/latest/download/homeassistant-supervised.deb
  sudo apt install ./homeassistant-supervised.deb
  ```
6. Готово. Заходим на ```http://<host>:8123``` (host=IP RPi4) и приступаем к первичной настройке Home Assistant.

---
Всё бы ничего, но я забыл про сам HA:
https://www.home-assistant.io/installation/linux#install-home-assistant-container
(Пробую ставить после Supervised по манулу. Внизу подставлен часовой пояс Москва GMT+3. Искать тут https://en.wikipedia.org/wiki/List_of_tz_database_time_zones, например ```Europe/Moscow```)
(UPD: Работает. Работает при такой корявой установке)
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
