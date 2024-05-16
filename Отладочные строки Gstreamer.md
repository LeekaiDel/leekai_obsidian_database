[[GStreamer]]
### Передача видео по UDP
#### Кодирование / Отправка видео

В переменную `host` вбивается айпи **ПОЛУЧАТЕЛЯ**
```bash
gst-launch-1.0 videotestsrc ! video/x-raw, width=640, height=480 !  x264enc pass=qual quantizer=20 tune=zerolatency ! rtph264pay ! udpsink host=192.168.128.206 port=5600
```

#### Декодирование / Прием видео

```bash
gst-launch-1.0 -v udpsrc port=5600 caps='application/x-rtp, media=(string)video, clock-rate=(int)90000, encoding-name=(string)H264' ! rtph264depay ! avdec_h264 ! videoconvert ! autovideosink
```
