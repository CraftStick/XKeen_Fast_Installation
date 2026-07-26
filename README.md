# Быстрая установка XKeen

Короткая инструкция для быстрого развёртывания XKeen на роутере Keenetic/Netcraze. Здесь только необходимый минимум — все нюансы, опциональные настройки и решение проблем смотрите в полной документации автора (ссылки внизу).

> [!NOTE]
> Если ранее вы уже пытались настроить XKeen по другим гайдам — рекомендуется сбросить роутер до заводских, отформатировать флешку в **EXT4** и пройти установку с нуля по этой инструкции.

---

## Оглавление
- [1. Подготовка USB-накопителя (EXT4)](#1-подготовка-usb-накопителя-ext4)
- [2. Компоненты роутера](#2-компоненты-роутера)
- [3. Установка Entware](#3-установка-entware)
- [4. Установка XKeen](#4-установка-xkeen)
- [5. Предварительные настройки роутера](#5-предварительные-настройки-роутера)
- [6. Настройка Xray](#6-настройка-xray)
- [7. Запуск](#7-запуск)
- [8. DNS-over-TLS / DNS-over-HTTPS](#8-dns-over-tls--dns-over-https)
- [9. (Опционально) Блокировка QUIC — UDP 80/443](#9-опционально-блокировка-quic--udp-порты-80-и-443)
- [Полная документация и полезные ссылки](#полная-документация-и-полезные-ссылки)

---

## 1. Подготовка USB-накопителя (EXT4)

Для работы OPKG диск должен быть отформатирован в **EXT4**. Отформатировать можно, например, бесплатной **Paragon Partition Manager Free** или **AOMEI Partition Assistant Standard**.

![Paragon Partition Manager Free](https://github.com/Corvus-Malus/XKeen-docs/raw/main/images/Light/Paragon-Partition-Manager-Free-Light.png)

> **Рекомендация:** при форматировании создайте раздел **SWAP** (обязательно первым) объёмом 512 МБ — 1 ГБ. Keenetic активирует его только при использовании 95% RAM. Также отключите сжатие RAM в «Параметры системы».

**macOS:** проще всего использовать [Keenetic Entware Flash](https://github.com/MaxXxaM/keenetic-entware-flash) — он одной командой создаёт SWAP + EXT4 с установщиком Entware.

> **Важно:** накопитель EXT4 не читается в Windows напрямую. При необходимости используйте драйвер [ext2fsd](https://www.ext2fsd.com/).

После форматирования подключите накопитель к USB-порту роутера — он должен появиться на странице «Приложения → Диски и принтеры». Если диск не виден, проверьте, что установлен компонент «Файловая система Ext».

> Перед установкой рекомендуется сделать резервную копию прошивки и настроек роутера (Общие настройки → Файлы).

![Резервная копия Keenetic](https://github.com/Corvus-Malus/XKeen-docs/raw/main/images/Light/Keenetic-backup-Light.jpg)

---

## 2. Компоненты роутера

В разделе **Параметры системы → Изменить набор компонентов** установите:

- [x] Интерфейс USB
- [x] Файловая система Ext
- [x] Общий доступ к файлам и принтерам по протоколу SMB
- [x] Поддержка открытых пакетов
- [x] Прокси-сервер DNS-over-TLS
- [x] Прокси-сервер DNS-over-HTTPS
- [x] Модули ядра подсистемы Netfilter
- [ ] Сервер SSH — **должен быть выключен** (Entware использует собственный SSH-сервер)

> В KeeneticOS 5+ компонент «Протокол IPv6» включён всегда и отключить его в наборе компонентов нельзя.

![Компоненты Keenetic](https://github.com/Corvus-Malus/XKeen-docs/raw/main/images/Light/Keenetic-components-Light.jpg)

---

## 3. Установка Entware

**Выбор архитектуры установщика** (по модели роутера):

- **mipsel** — [mipsel-installer.tar.gz](https://bin.entware.net/mipselsf-k3.4/installer/mipsel-installer.tar.gz)
  Keenetic/Netcraze: 4G (KN-1212), Omni (KN-1410), Extra (KN-1710/1711/1713), Giga (KN-1010/1011), Ultra (KN-1810), Viva (KN-1910/1912/1913), Giant (KN-2610), Hero 4G (KN-2310/2311), Hopper (KN-3810).
  Zyxel Keenetic: II / III, Extra, Extra II, Giga II / III, Omni, Omni II, Viva, Ultra, Ultra II.

- **mips** — [mips-installer.tar.gz](https://bin.entware.net/mipssf-k3.4/installer/mips-installer.tar.gz)
  Keenetic/Netcraze: Ultra SE (KN-2510), Giga SE (KN-2410), DSL (KN-2010), Skipper DSL (KN-2112), Duo (KN-2110), Hopper DSL (KN-3610).
  Zyxel Keenetic: DSL, LTE, VOX.

- **aarch64** — [aarch64-installer.tar.gz](https://bin.entware.net/aarch64-k3.10/installer/aarch64-installer.tar.gz)
  Keenetic/Netcraze: Peak (KN-2710), Ultra (KN-1811), Ultra (NC-1812), Giga (KN-1012), Hopper (KN-3811), Hopper SE (KN-3812).

**Шаги (на примере mipsel):**

1. В корне раздела диска создайте папку **install** и положите туда установочный архив (напр. `mipsel-installer.tar.gz`). Доступ к диску по сети — через SMB.

![Диски и принтеры](https://github.com/Corvus-Malus/XKeen-docs/raw/main/images/Light/Keenetic-apps-Light.jpg)

2. В веб-интерфейсе перейдите на страницу **OPKG**, в поле «Накопитель» выберите ваш EXT4-раздел, нажмите **Сохранить**. Поле сценария `initrc` оставьте пустым — оно само сменится на `/opt/etc/init.d/rc.unslung`.

![OPKG](https://github.com/Corvus-Malus/XKeen-docs/raw/main/images/Light/Keenetic-OPKG-Light.jpg)

3. В «Диагностика → Системный журнал» дождитесь сообщения об успешной установке Entware (этапы 1/5 … 5/5).

4. Подключитесь по SSH (**PuTTY** / **Termius** / **MobaXterm**): IP роутера, порт **22** (порт **222** используется только если установлен компонент «Сервер SSH»). Логин `root`, пароль `keenetic`.

![PuTTY](https://github.com/Corvus-Malus/XKeen-docs/raw/main/images/Light/Putty-Light.png)

5. Смените пароль root командой `passwd`, затем обновите пакеты:

```bash
opkg update
opkg upgrade
```

---

## 4. Установка XKeen

**Рекомендуемый способ** (от пользователя root):

```bash
opkg update && opkg upgrade && opkg install curl tar && cd /tmp
sh -c "$(curl -sSL https://raw.githubusercontent.com/jameszeroX/XKeen/main/install.sh)"
```

Либо через jsDelivr (если GitHub недоступен):

```bash
opkg update && opkg upgrade && opkg install curl tar && cd /tmp
sh -c "$(curl -sSL https://cdn.jsdelivr.net/gh/jameszeroX/XKeen@main/install.sh)"
```

Во время установки выберите:

- **GeoIP** → `1. Установить отсутствующие GeoIP`
- **GeoSite** → `1. Установить отсутствующие GeoSite`
- **Автообновления** → `1. Включить отсутствующие задачи`, затем задайте расписание (напр. ежедневно в 00:00).

По завершении вы увидите: `Установка окончена`.

> Для роутеров **Keenetic Skipper 4G (KN-2910)** и **4G (KN-1212)** в актуальном форке совместимый бинарник Xray ставится сразу — подменять его вручную больше не нужно.

---

## 5. Предварительные настройки роутера

> [!IMPORTANT]
> **Хотите гнать весь трафик через XKeen без разбора по устройствам — просто НЕ создавайте политику.** Если политики с именем `XKeen` нет, проксирование автоматически применяется ко **всем** клиентам роутера, а раздел ниже можно пропустить. Политика нужна только если вы хотите включать обход выборочно, для отдельных устройств.

### Создание политики (только для выборочного проксирования)

1. **Приоритеты подключений → Политики доступа в интернет** → создайте политику **`XKeen`**, способ доступа — «Отметить провайдера или нескольких».
2. **Приоритеты подключений → Применение политик** → добавьте в политику цели **Клиент / Сеть** (те устройства, которым нужен обход).

![Политики доступа](https://github.com/Corvus-Malus/XKeen-docs/raw/main/images/Light/Keenetic-interface-priorities-Light.png)

![Применение политик](https://github.com/Corvus-Malus/XKeen-docs/raw/main/images/Light/Keenetic-policy-consumers-Light.png)

---

### Перенос сервисов Keenetic с 443 порта

Нужно для режима TProxy; для Hybrid — не обязательно.

В CLI роутера (`192.168.1.1/a`):

```bash
ip http ssl port 8443
system configuration save
```

Допустимые порты: `5083 | 5443 | 8083 | 8443 | 65083`. После переноса KeenDNS будет доступен по новому порту, напр. `xxxx.keenetic.link:8443`.

![webcli](https://github.com/Corvus-Malus/XKeen-docs/raw/main/images/Light/webcli-Light.jpg)

---

## 6. Настройка Xray

Конфиги лежат в `/opt/etc/xray/configs/`. Нужны три файла: `03_inbounds.json`, `04_outbounds.json`, `05_routing.json`.

![Каталог configs](https://github.com/Corvus-Malus/XKeen-docs/raw/main/images/Light/Explorer-configs-Light.jpg)

### 03_inbounds.json — режим работы

Выберите один режим:

- [Mixed](https://github.com/Corvus-Malus/XKeen/releases/latest/download/03_inbounds.json) — UDP через TProxy, TCP через Redirect (баланс).
- [TProxy](https://github.com/Corvus-Malus/XKeen/releases/latest/download/03_inbounds_tproxy.json) — TCP+UDP, лучший вариант для игр и стриминга.
- [Redirect](https://github.com/Corvus-Malus/XKeen/releases/latest/download/03_inbounds_redirect.json) — только TCP.

![03_inbounds](https://github.com/Corvus-Malus/XKeen-docs/raw/main/images/Dark/03-inbounds-Dark.png)

---

### 04_outbounds.json — подключение к VPS

[Шаблон](https://github.com/Corvus-Malus/XKeen/releases/latest/download/04_outbounds.json). Заполните под свой VPS:

- `tag` — например `vless-reality`
- `protocol` — `vless`
- `address` — IP вашего VPS
- `port` — `443`
- `fingerprint`, `serverName` — как в 3X-UI (напр. `chrome`, `yahoo.com`)
- `id`, `publicKey`, `shortId` — из инфо соединения в 3X-UI (`pbk`=publicKey, `fp`=fingerprint, `sni`=serverName, `sid`=shortId)

![3X-UI](https://github.com/Corvus-Malus/XKeen-docs/raw/main/images/Dark/3X-UI-Dark.png)

Проще всего собрать outbound через генератор — вставьте ссылку подключения из 3X-UI:
[XKeen Config Generator](https://corvus-malus.github.io/XKeen-Config-Generator/)

![Config Generator](https://github.com/Corvus-Malus/XKeen-docs/raw/main/images/Dark/XKeen-Config-Generator-Dark.png)

---

### 05_routing.json — правила маршрутизации

Выберите вариант:

- [Вариант 1](https://github.com/Corvus-Malus/XKeen/releases/latest/download/05_routing.json) — через VPS идут только указанные IP/домены (Google, Twitter, TikTok и т.п.), всё остальное — напрямую.
- [Вариант 2](https://github.com/Corvus-Malus/XKeen-docs/releases/latest/download/05_routing.json) — напрямую идут `.ru/.su/.рф` и торренты, всё остальное — через VPS.

Правила можно собрать в [XKeen Routing Generator](https://xray-routing-generator.netlify.app/).

![05_routing](https://github.com/Corvus-Malus/XKeen-docs/raw/main/images/Dark/05-routing-Dark.png)

Способы GeoIP/GeoSite — автоматические базы адресов, обновляются через xkeen. Типы совпадений в правилах: частичное (`vk.com`), `regexp:`, `domain:` (поддомены), `full:` (точное).

---

## 7. Запуск

```bash
xkeen -start
```

![xkeen start](https://github.com/Corvus-Malus/XKeen-docs/raw/main/images/Dark/xkeen-start-Dark.png)

Основные команды: `xkeen -stop`, `xkeen -restart`, `xkeen -status`.

> **Про нагрузку на CPU:** Xray обрабатывает весь трафик, и слабый процессор может не тянуть большой транзит. Чтобы снизить нагрузку, можно ограничить работу портами 80 и 443:
> ```bash
> xkeen -ap 443,80
> ```
> В форке 1.1.3.9+ порты задаются через файл `/opt/etc/xkeen/port_proxying.lst`.

---

## 8. DNS-over-TLS / DNS-over-HTTPS

Для шифрования DNS-запросов. Отключите «транзит DNS» и **не** игнорируйте DNS провайдера.

### DNS-over-TLS (DoT)

- CloudFlare: `1.1.1.1` `cloudflare-dns.com`
- Google: `8.8.8.8` `dns.google`
- AdGuard: `94.140.14.14` `dns.adguard-dns.com`

---

### DNS-over-HTTPS (DoH)

- CloudFlare: `https://cloudflare-dns.com/dns-query`
- Google: `https://dns.google/dns-query`
- AdGuard: `https://dns.adguard-dns.com/dns-query`

![DNS](https://github.com/Corvus-Malus/XKeen-docs/raw/main/images/Light/DNS.png)

---

## 9. (Опционально) Блокировка QUIC — UDP-порты 80 и 443

Браузеры используют **QUIC** (HTTP/3 поверх UDP). Такой трафик может идти мимо прокси и ломать доступ к части сайтов (например, ChatGPT). Чтобы этого не было, заблокируйте UDP на портах **443** и **80** — браузер откатится на TCP, и маршрутизация XKeen отработает корректно.

**Сетевые правила → Межсетевой экран → Добавить правило:**

- **Включить правило** — ✅
- **Действие** — `Запретить`
- **IP-адрес источника** — `Любой`
- **IP-адрес назначения** — `Любой`
- **Номер порта источника** — `Любой`
- **Протокол** — `UDP`
- **Номер порта назначения** — `Равен` → `443`
- **Расписание работы** — `Работает постоянно`

Нажмите **Сохранить**, затем создайте **второе точно такое же правило для порта 80**.

![Правило межсетевого экрана](https://github.com/Corvus-Malus/XKeen-docs/raw/main/images/Light/Keenetic-Bridge0-Light.png)

> Правило создаётся на активном подключении (напр. «Подключение Ethernet»). Если сайты, наоборот, перестанут открываться — временно отключите эти правила и проверьте.

---

## Полная документация и полезные ссылки

> ⚠️ Это **быстрая установка** — рассмотрен только базовый путь. Тонкости (SWAP вручную, замена/обновление ядра Xray, исключения маршрутизации при нескольких туннелях, фикс голоса в Discord, доступ к ChatGPT, бэкап флешки, полный список консольных команд, решение проблем с SSH и т.д.) описаны в полной документации автора. Если что-то пошло не так или нужна тонкая настройка — смотрите там.

- **Полная документация XKeen (Corvus-Malus):** https://github.com/Corvus-Malus/XKeen
- **Форк XKeen (jameszeroX) + Wiki:** https://github.com/jameszeroX/XKeen — раздел [Configuration](https://github.com/jameszeroX/XKeen/wiki/Configuration)
- **FAQ по XKeen:** https://jameszero.net/faq-xkeen.htm
- **Инструкция от автора XKeen (Skrill0):** https://xskrill.notion.site/XKeen-c9f0f2a5018743b59eb81bd6fccdf25a
- **Телеграм-чат XKeen:** https://t.me/+SZWOjSlvYpdlNmMy
- **Русскоязычный чат Project VLESS:** https://t.me/projectVless

Авторы: XKeen — [@Skrill_zerro](https://t.me/Skrill_zerro), форк — [jameszeroX](https://github.com/jameszeroX).

---

## ⭐ Поддержка

Если инструкция помогла - поставьте звёздочку, буду рад 🤗

И не забудьте про оригинал — авторов XKeen тоже поддержите звездой.
