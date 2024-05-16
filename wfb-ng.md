### Drone (Firefly):

- Клонируем репозиторий в хомяк например.
```bash
git clone -b stable https://github.com/svpcom/wfb-ng.git
```

- Устанавливаем необходимые пакеты.
```bash
sudo apt install python3-all libpcap-dev libsodium-dev python3-pip python3-pyroute2 python3-future python3-twisted python3-serial python3-all-dev iw virtualenv debhelper dh-python build-essential -y
```

- Переходим в директорию wfb-ng и запускаем сборку deb-пакета.
```bash
cd wfb-ng
sudo make deb
``````

- Переходим в директорию с собранным deb-пакетом и устанавлиаем его.
```bash
cd deb_dist
sudo dpkg -i wfb-ng_23.8.141.53708-1_arm64.deb
```

- Запускаем генератор ключей.
```bash
wfb_keygen
```

- Перемещаем ключ дрона в `/etc/`.
```bash
sudo cp drone.key /etc/
```


>[!info] Важно!
Также сгенерированный вместе с drone.key ключ для GS нужно закинуть на наземную станцию в /etc. Он называется gs.key.

- Далее нужно создать конфиг в директории `/etc`.
```bash
sudo vim /etc/wifibroadcast.cfg
```

- Добавляем туда конфигурационные данные для дрона.
```bash
[common]
wifi_channel = 161     # 161 -- radio channel @5825 MHz, range: 5815–5835 MHz, width 20MHz
                       # 1 -- radio channel @2412 Mhz, 
                       #                        # see https://en.wikipedia.org/wiki/List_of_WLAN_channels for reference
wifi_region = 'BO'     # Your country for CRDA (use BO or GY if you want max tx power)  


[drone_mavlink]
# use autopilot connected to /dev/ttyUSB0 at 115200 baud:
# peer = 'serial:ttyUSB0:115200'
# Connect to autopilot via malink-router or mavlink-proxy:
# peer = 'listen://0.0.0.0:14550'   # incoming connection
# peer = 'connect://127.0.0.1:14550'  # outgoing connection

[drone_video]
peer = 'listen://0.0.0.0:5600'  # listen for video stream (gstreamer on drone)

```

**Отключите wpa_supplicant и другие демоны в интерфейсе wlan WFB-NG!** 
Используйте `ps uaxwwww | grep wlan` для проверки.  

Дважды проверьте, что карта находится в **неуправляемом состоянии** в выводе `nmcli` и `ifconfig wlanXX` **не показывает никакого адреса, а состояние карты отключено**.

- Отредактируйте `/etc/default/wifibroadcast` и замените `wlan0` соответствующим названием интерфейса WiFi. Также добавьте в `/etc/NetworkManager/NetworkManager.conf` следующий раздел:

```bash
[keyfile]
unmanaged-devices=interface-name:wlan0
```

- Если доступно `/etc/dhcpcd.conf`, отредактируйте его и добавьте:
```bash
denyinterfaces wlan0
```
чтобы игнорировать интерфейс WFB-NG.

- Далее запускаем сервис wfb-ng и добавляем его в автозапуск.
```bash
sudo systemctl start wifibroadcast@drone
sudo systemctl enable wifibroadcast.service
```

### Наземная станция:

Делаем все то же самое, что и для дрона, только deb-пакет будет для архитектуры `amd64`.
Также пропускаем шаг с генерацией ключей, так как мы это сделали ранее на дроне и уже положили ключ станции в директорию `/etc`.

В конфиг `/etc/wifibroadcast.cfg` записываем такие настройки:
```bash
[common]
wifi_channel = 161     # 161 -- radio channel @5825 MHz, range: 5815–5835 MHz, width 20MHz
                       # 1 -- radio channel @2412 Mhz, 
                       # see https://en.wikipedia.org/wiki/List_of_WLAN_channels for reference
wifi_region = 'BO'     # Your country for CRDA (use BO or GY if you want max tx power)  

[gs_mavlink]
#peer = 'connect://127.0.0.1:14550'  # outgoing connection
# peer = 'listen://0.0.0.0:14550'   # incoming connection

[gs_video]
peer = 'connect://127.0.0.1:5600'  # outgoing connection for
                                   # video sink (QGroundControl on GS)
```

- Далее запускаем сервис wfb-ng и добавляем его в автозапуск.
```bash
sudo systemctl start wifibroadcast@gs
sudo systemctl enable wifibroadcast.service
```


### Передача тестового видео от дрона на станцию:

>[!info] Примечание
>Если не может найти енкодер x264enc, 
>то нужно установить `gstreamer1.0-plugins-ugly`
#### Кодирование / Отправка видео по UDP

В переменную `host` вбивается айпи **ПОЛУЧАТЕЛЯ**
```bash
gst-launch-1.0 videotestsrc ! video/x-raw, width=640, height=480 !  x264enc pass=qual quantizer=20 tune=zerolatency ! rtph264pay ! udpsink host=127.0.0.1 port=5600
```

#### Декодирование / Прием видео

```bash
gst-launch-1.0 -v udpsrc port=5600 caps='application/x-rtp, media=(string)video, clock-rate=(int)90000, encoding-name=(string)H264' ! rtph264depay ! avdec_h264 ! videoconvert ! autovideosink
```
