# Практика 4. Анализ сетевого трафика

[English version](README.en.md)

**70 минут, Wireshark.** Проходим по записи настоящего заражения компьютера (троян Trickbot, 2017 год): находим программу под видом картинки,
проверку внешнего адреса, нестандартные порты и маяк. Во второй записи — DNS-туннель. Преподаватель показывает каждый шаг на проекторе,
вы повторяете у себя. Программировать не нужно.

## 0. Запустить Wireshark

Откройте Codespace из этого репозитория (**Code → Codespaces → Create codespace on main**) или склонируйте его в уже открытый Codespace:

```bash
git clone https://github.com/ObeWan99/P4-traffic-analysis.git
cd P4-traffic-analysis
```

Запустите Wireshark — он работает в контейнере и открывается в браузере:

```bash
docker compose up -d
```

Первый запуск скачивает образ (около 2 ГБ) — 2–3 минуты. Затем вкладка **Ports** → порт **3000** → **Open in Browser**.
В новой вкладке браузера — окно Wireshark.

Открыть запись: **File → Open**, в поле имени файла ввести `/pcap/c2-beacon.pcap` → **Open**. Вторая запись — `/pcap/dns-tunnel.pcap`.

> Внутри `c2-beacon.pcap` — настоящая вредоносная программа для Windows. **Смотреть можно, сохранять что-либо из трафика нельзя:**
> меню **File → Export Objects** не трогать.

## Фильтры занятия

Фильтр вводится в строку над списком пакетов и применяется клавишей **Enter**. Красная строка — опечатка.

| Шаг | Что смотрим | Фильтр или меню |
|---|---|---|
| 1 | Кто с кем говорил | **Statistics → Conversations**, вкладка **IPv4** |
| 2 | Что скачивал компьютер | `http.request` |
| 3 | Что за «картинка» | правой кнопкой по строке → **Follow → TCP Stream** |
| 4 | Что программа сделала первым делом | `dns.flags.response == 0` · `http.request.uri == "/raw"` |
| 5 | Куда подключается | `tcp.flags.syn == 1 && tcp.flags.ack == 0` |
| 5 | Ритм соединений | **View → Time Display Format → Seconds Since Previous Displayed Packet**, затем `ip.dst == 67.21.90.105 && tcp.flags.syn == 1 && tcp.flags.ack == 0` |
| 5 | Ещё один сервер | `ip.dst == 72.211.215.68 && tcp.flags.syn == 1` |
| 6 | Из чего состоит трафик | файл `/pcap/dns-tunnel.pcap`, **Statistics → Protocol Hierarchy** |
| 6 | Запросы к одному домену | `dns.flags.response == 0 && dns.qry.name contains "onetwogleb"` |
| 6 | Все остальные запросы | `dns.flags.response == 0 && !(dns.qry.name contains "onetwogleb") && !mdns` |
| 6 | Ритм запросов | **Statistics → I/O Graphs** |

После **Follow → TCP Stream** в строке остаётся `tcp.stream eq …` — сотрите его и нажмите Enter.

## Итог занятия

Таблица «наблюдение → гипотеза → как проверили → подтверждено?» по всем находкам. Заполняем вместе в конце занятия.

## Файлы

| Файл | Содержимое |
|---|---|
| `pcap/c2-beacon.pcap` | запись заражения компьютера, 21 минута |
| `pcap/dns-tunnel.pcap` | запись второго компьютера, 27 минут |
| `pcap/SOURCES.md` | откуда взяты записи и их контрольные суммы |
| `compose.yml` | Wireshark в браузере |

Остановить Wireshark: `docker compose down`.
