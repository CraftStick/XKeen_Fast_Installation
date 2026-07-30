# Быстрая установка XKeen

Компактная пошаговая инструкция по установке XKeen на роутер Keenetic/Netcraze. Только рабочий путь — нюансы и решение проблем в полной документации автора (ссылки внизу).

> [!NOTE]
> Если раньше уже пробовали ставить XKeen по другим гайдам — сбросьте роутер, отформатируйте флешку в **EXT4** и начните с нуля.

---

## 1. Флешка (EXT4)

Флешку нужно отформатировать в **EXT4**.

**Способ 1 — прямо в роутере** (KeeneticOS 5.1.1+, можно удалённо): сначала в **Параметры системы → Изменить набор компонентов** установите компонент **«Утилиты для файловой системы ext»** (поиск: `утили`). 

Затем вставьте флешку → **Накопители и устройства** → напротив раздела **⋮ → Форматировать** → файловая система **EXT4** → Подтвердить. Флешку в компьютер вставлять не нужно.

![Накопители и устройства — Форматировать](https://github.com/CraftStick/XKeen_Fast_Installation/blob/main/images/format-router-1.jpg?raw=true)

![Форматирование раздела в EXT4](https://github.com/CraftStick/XKeen_Fast_Installation/blob/main/images/format-router-2.jpg?raw=true)

**Способ 2 — на компьютере** (если версия старая или нет пункта форматирования): Paragon Partition Manager Free (Windows) или [Keenetic Entware Flash](https://github.com/MaxXxaM/keenetic-entware-flash) (macOS, делает SWAP + EXT4 одной командой).

![Paragon Partition Manager Free](https://github.com/Corvus-Malus/XKeen-docs/raw/main/images/Light/Paragon-Partition-Manager-Free-Light.png)

> Желательно первым разделом создать **SWAP** 512 МБ — 1 ГБ (при форматировании на компе).

После форматирования флешка должна появиться в «Накопители и устройства».

---

## 2. Компоненты роутера

**Параметры системы → Изменить набор компонентов** — включите:

- [x] Интерфейс USB · Файловая система Ext · Сервер SMB
- [x] Поддержка открытых пакетов
- [x] Прокси-сервер DNS-over-TLS · DNS-over-HTTPS
- [x] Модули ядра подсистемы Netfilter
- [ ] **Сервер SSH — выключить** (Entware ставит свой)

![Компоненты Keenetic](https://github.com/Corvus-Malus/XKeen-docs/raw/main/images/Light/Keenetic-components-Light.jpg)

---

## 3. Установка Entware

Архитектура установщика по модели:

- **mipsel** — [installer](https://bin.entware.net/mipselsf-k3.4/installer/mipsel-installer.tar.gz): 4G (KN-1212), Omni (KN-1410), Extra (KN-1710/1711/1713), Giga (KN-1010/1011), Ultra (KN-1810), Viva (KN-1910/1912/1913), Giant (KN-2610), Hero 4G (KN-2310/2311), Hopper (KN-3810); Zyxel Keenetic II/III, Extra, Extra II, Giga II/III, Omni, Omni II, Viva, Ultra, Ultra II.
- **mips** — [installer](https://bin.entware.net/mipssf-k3.4/installer/mips-installer.tar.gz): Ultra SE (KN-2510), Giga SE (KN-2410), DSL (KN-2010), Skipper DSL (KN-2112), Duo (KN-2110), Hopper DSL (KN-3610); Zyxel Keenetic DSL, LTE, VOX.
- **aarch64** — [installer](https://bin.entware.net/aarch64-k3.10/installer/aarch64-installer.tar.gz): Peak (KN-2710), Ultra (KN-1811), Ultra (NC-1812), Giga (KN-1012), Hopper (KN-3811), Hopper SE (KN-3812).

1. Положите архив установщика в папку **install** в корне флешки (доступ по SMB).
2. **OPKG** → в «Накопитель» выберите EXT4-раздел → **Сохранить** (`initrc` оставьте пустым).

![OPKG](https://github.com/Corvus-Malus/XKeen-docs/raw/main/images/Light/Keenetic-OPKG-Light.jpg)

3. Дождитесь в «Диагностика → Системный журнал» установки Entware (этапы 1/5 … 5/5).
4. Подключитесь по SSH (PuTTY / Termius): IP роутера, порт **22**, логин `root`, пароль `keenetic`.
5. Смените пароль (`passwd`) и обновите пакеты:

```bash
opkg update && opkg upgrade
```

---

## 4. Установка XKeen

От root:

```bash
opkg update && opkg upgrade && opkg install curl tar && cd /tmp
sh -c "$(curl -sSL https://raw.githubusercontent.com/jameszeroX/XKeen/main/install.sh)"
```

В процессе выберите: **GeoIP** → `1`, **GeoSite** → `1`, **Автообновления** → `1` + расписание. В конце — `Установка окончена`.

---

## 5. Настройки роутера

> [!IMPORTANT]
> **Нужен обход для ВСЕХ устройств — не создавайте политику.** Без политики `XKeen` проксирование включается на всех клиентов автоматически. Политика нужна только для выборочного обхода.

**Выборочно (по желанию):** Приоритеты подключений → Политики → создать `XKeen` → Применение политик → добавить нужные устройства.

**Перенос сервисов с 443** (нужно для TProxy) — в CLI (`192.168.1.1/a`):

```bash
ip http ssl port 8443
system configuration save
```

![Политики доступа](https://github.com/Corvus-Malus/XKeen-docs/raw/main/images/Light/Keenetic-interface-priorities-Light.png)

---

## 6. Настройка Xray

Конфиги в `/opt/etc/xray/configs/` — нужны три файла.

![Каталог configs](https://github.com/Corvus-Malus/XKeen-docs/raw/main/images/Light/Explorer-configs-Light.jpg)

**03_inbounds.json** — режим: [Mixed](https://github.com/Corvus-Malus/XKeen/releases/latest/download/03_inbounds.json) (баланс) · [TProxy](https://github.com/Corvus-Malus/XKeen/releases/latest/download/03_inbounds_tproxy.json) (игры/стриминг) · [Redirect](https://github.com/Corvus-Malus/XKeen/releases/latest/download/03_inbounds_redirect.json) (только TCP).

**04_outbounds.json** — [шаблон](https://github.com/Corvus-Malus/XKeen/releases/latest/download/04_outbounds.json). Проще собрать через [Config Generator](https://corvus-malus.github.io/XKeen-Config-Generator/) — вставьте VLESS ссылку из 3X-UI.

> [!NOTE]
> 🎯 **У вас еще нет VLESS-ссылки из 3X-UI и не знаете что это?**
> 
> Посмотрите [пошаговое руководство по поднятию VPS](https://www.youtube.com/watch?v=zXt3ThtVy0M&t=982s) на основе отличного видео от EasyNetwork.

![Config Generator](https://github.com/Corvus-Malus/XKeen-docs/raw/main/images/Dark/XKeen-Config-Generator-Dark.png)

**05_routing.json** — [Вариант 1](https://github.com/Corvus-Malus/XKeen/releases/latest/download/05_routing.json) (через VPS только заблокированное) или [Вариант 2](https://github.com/Corvus-Malus/XKeen-docs/releases/latest/download/05_routing.json) (RU напрямую, остальное через VPS). Правила — в [Routing Generator](https://xray-routing-generator.netlify.app/).

![05_routing](https://github.com/Corvus-Malus/XKeen-docs/raw/main/images/Dark/05-routing-Dark.png)

**Продвинутый вариант** — базы от автора форка [zkeen-ip](https://github.com/jameszeroX/zkeen-ip) (IP-подсети) + [zkeen-domains](https://github.com/jameszeroX/zkeen-domains) (домены). Готовый пример routing с этими базами (заблокированное, Discord, крупные CDN/хостеры через VLESS, остальное — direct) лежит в README [zkeen-ip](https://github.com/jameszeroX/zkeen-ip). Актуальный `zkeenip.dat` — на [странице релизов](https://github.com/jameszeroX/zkeen-ip/releases/latest).

---

## 7. Запуск

```bash
xkeen -start
```

Команды: `xkeen -stop` · `-restart` · `-status`.

![xkeen start](https://github.com/Corvus-Malus/XKeen-docs/raw/main/images/Dark/xkeen-start-Dark.png)

---

## 8. DNS (DoT / DoH)

Отключите транзит DNS и не игнорируйте DNS провайдера.

- **DoT:** `1.1.1.1` cloudflare-dns.com · `8.8.8.8` dns.google · `94.140.14.14` dns.adguard-dns.com
- **DoH:** `https://cloudflare-dns.com/dns-query` · `https://dns.google/dns-query` · `https://dns.adguard-dns.com/dns-query`

![DNS](https://github.com/Corvus-Malus/XKeen-docs/raw/main/images/Light/DNS.png)

---

## 9. (Опционально) Блокировка QUIC — UDP 80/443

QUIC (HTTP/3 поверх UDP) может идти мимо прокси и ломать часть сайтов (ChatGPT и т.п.). Заблокируйте UDP 443 и 80 — браузер откатится на TCP.

**Сетевые правила → Межсетевой экран → Добавить правило:** Действие `Запретить`, Протокол `UDP`, Порт назначения `Равен` → `443`. Затем **второе правило для порта 80**.

![Правило межсетевого экрана](https://github.com/Corvus-Malus/XKeen-docs/raw/main/images/Light/Keenetic-Bridge0-Light.png)

---

## Полная документация

> ⚠️ Это быстрая установка — только базовый путь. Тонкости (обновление ядра Xray, исключения маршрутизации, фикс Discord/ChatGPT, бэкап, полный список команд) — в доке автора.

- **Полная дока XKeen (Corvus-Malus):** https://github.com/Corvus-Malus/XKeen
- **Форк + Wiki (jameszeroX):** https://github.com/jameszeroX/XKeen
- **FAQ:** https://jameszero.net/faq-xkeen.htm
- **Телеграм-чат XKeen:** https://t.me/+SZWOjSlvYpdlNmMy

Авторы: XKeen — [@Skrill_zerro](https://t.me/Skrill_zerro), форк — [jameszeroX](https://github.com/jameszeroX).

---

<div align="center">

Если было полезно — поставьте ⭐, это очень помогает!

И не забудьте про оригинал — авторов XKeen тоже поддержите звездой.

</div>
