## 1. Установка  
#### На Debian/Ubuntu/Astra Linux:  
  
```bash
sudo apt update
sudo apt install tshark
```
 Во время установки может задать вопрос о разрешении обычным пользователям перехватывать пакеты. Для удобства можно ответить «Да".

На CentOS/RHEL/Fedora: 
bash
sudo dnf install wireshark-cli   
# или                                           
sudo yum install wireshark-cli 

2. tshark  для захвата трафика на порту udp 514


Захват трафика на UDP-порту 514 - это классическая задача для анализа систем логирования (например, Syslog). Вот как это делается в tshark:

2.1. Базовый захват трафика на порту 514
Простой мониторинг в реальном времени:

bash
# Захват и отображение трафика на порту 514 в реальном времени
 sudo tshark -i any -Y "udp.port == 514" 

С сохранением в файл:
bash
# Захватить 1000 пакетов и сохранить в файл
 sudo tshark -i any -f "udp port 514" -c 1000 -w syslog_traffic.pcapng 

# Непрерывный захват до остановки Ctrl+C
 sudo tshark -i any -f "udp port 514" -w syslog_capture.pcapng 

2.2. Фильтрация с учетом направления трафика

bash
# Только входящие Syslog-сообщения (на порт 514)
 sudo tshark -i any -Y "udp.dstport == 514" 

# Только исходящие Syslog-сообщения (с порта 514)
 sudo tshark -i any -Y "udp.srcport == 514" 

# Трафик от конкретного источника к порту 514
 sudo tshark -i any -Y "ip.src == 192.168.1.100 and udp.dstport == 514" 

2.3. Анализ содержимого Syslog-сообщений
Просмотр базовой информации

bash
# Показать IP-адреса и порты
 tshark -r syslog_traffic.pcapng -Y "udp.port == 514" \
-T fields -e frame.time -e ip.src -e udp.srcport -e ip.dst -e udp.dstport 

Извлечение самого Syslog-сообщения

bash
# Показать содержимое Syslog-сообщений
 tshark -r syslog_traffic.pcapng -Y "udp.port == 514" -T fields -e data.text 

# Или в hex-формате для сырых данных
tshark -r syslog_traffic.pcapng -Y "udp.port == 514" -T fields -e data.data 

Форматированный вывод с разбором Syslog

bash
# Расширенный вывод с разбором Syslog-полей
tshark -r syslog_traffic.pcapng -Y "udp.port == 514" -T fields \
  -e frame.time \
  -e ip.src \
  -e syslog.pri \
  -e syslog.timestamp \
  -e syslog.hostname \
  -e syslog.tag \
  -e syslog.message

2.4. Продвинутые примеры для реального использования
Мониторинг в реальном времени с детальным выводом

bash
sudo tshark -i any -Y "udp.port == 514" -T fields \
  -e frame.time_relative \
  -e ip.src \
  -e syslog.facility \
  -e syslog.severity \
  -e syslog.message

Статистика по источникам Syslog

bash
# Какие IP-адреса шлют больше всего Syslog-сообщений
tshark -r syslog_traffic.pcapng -Y "udp.dstport == 514" -q -z endpoints,udp

# Или более конкретно:
tshark -r syslog_traffic.pcapng -Y "udp.port == 514» \
-T fields -e ip.src | sort | uniq -c | sort -nr

Поиск конкретных сообщений

bash
# Найти сообщения, содержащие слово "error"
tshark -r syslog_traffic.pcapng -Y "udp.port == 514 and frame contains \"error\""

# Найти сообщения от определенного хоста
tshark -r syslog_traffic.pcapng -Y \
"udp.port == 514 and syslog.hostname == \»router1\""

Захват с ограничением по времени или размеру

bash
# Захватывать 5 минут
sudo tshark -i any -f "udp port 514" -a duration:300 -w syslog_5min.pcapng

# Захватывать до достижения 10 МБ
sudo tshark -i any -f "udp port 514" -a filesize:10240 -w syslog_10mb.pcapng

2.5. Особенности для защищенного трафика (TLS/Syslog over TCP)

Если используется защищенный протокол на TCP-порту 514 или 6514:

bash
# Для TCP-based Syslog
sudo tshark -i any -Y "tcp.port == 514"

# Для Syslog over TLS (обычно порт 6514)
sudo tshark -i any -Y "tcp.port == 6514»

2.6. Полезные советы для работы с Syslog

Используйте -i any - чтобы перехватывать трафик на всех интерфейсах
Фильтр захвата vs фильтр отображения:
-f "udp port 514" - фильтр захвата (экономит ресурсы)
-Y "udp.port == 514" - фильтр отображения (более гибкий)
Для декодирования защищенного TLS-трафика может понадобиться ключ RSA:

bash
tshark -r encrypted_syslog.pcapng -o \
"ssl.keys_list: <ip>,<port>,<protocol>,<key_file>»

Автоматизация - можно использовать в скриптах для мониторинга:

bash
#!/bin/bash
tshark -i any -Y "udp.port == 514" -T fields -e syslog.message -a duration:60 | while read line; do
    echo "$(date): $line" >> /var/log/syslog_monitor.log
done

Этот подход позволит вам эффективно перехватывать и анализировать Syslog-трафик для диагностики проблем мониторинга, отладки систем логирования или аудита безопасности.
