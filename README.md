# Podkop Lists — домены и подсети для OpenWrt + Hysteria 2

Готовые списки доменов и подсетей для [**Podkop**](https://github.com/itdoginfo/podkop) на OpenWrt-роутерах. Покрывают заблокированные в РФ сервисы, ушедшие с рынка РФ сервисы, а также домены которые не работают «по умолчанию» через community-lists (Xiaomi Mi Home, Spotify для Discord Rich Presence, EA/Apex Legends, Figma и другие).

Протестировано на **ASUS TUF-AX4200** и **Cudy TR3000** (с 256 MB flash), OpenWrt 24.10+, Podkop v0.7.14+, Sing-box 1.12.22+.

## 📋 Что внутри

| Файл | Что содержит | Строк |
|------|--------------|-------|
| [`user-domains.txt`](./user-domains.txt) | Домены заблокированных/ушедших сервисов + критичные API-endpoints | ~550 |
| [`user-subnets.txt`](./user-subnets.txt) | IP-подсети тех же сервисов (Google Cloud, EA, Blizzard, Netflix и др.) | ~250 |

Оба файла — в формате **text mode** для полей `User domains text` и `User subnets text` в Podkop. Поддерживают `//`-комментарии и секции.

## 🎯 Community-lists (галочки в Podkop)

Эти списки должны быть **включены** в секции (Sections → main → Community lists). Домены/подсети этих сервисов **не дублированы** в моих файлах:

- ✅ `meta` (Facebook/Instagram/WhatsApp)
- ✅ `telegram`
- ✅ `twitter` (X)
- ✅ `youtube`
- ✅ `cloudflare`
- ✅ `tiktok` (но расширен в `user-domains.txt`, т.к. community не покрывает все CDN)

Если ты **отключил** какой-то из этих community-lists — соответствующие домены/подсети нужно добавить вручную (я их не включил в файлы чтобы не было конфликтов).

## 🚀 Быстрый старт

### Требования

- OpenWrt 24.10 или новее
- Podkop v0.7.14 или новее ([установка](https://github.com/itdoginfo/podkop#установка))
- Hysteria 2 / VLESS / другой прокси-сервер уже настроен
- Минимум 30 MB свободного flash-памяти на роутере

### Шаг 1. Открой Podkop

В браузере: `http://192.168.1.1` → **Services → Podkop → Sections → main**

### Шаг 2. Включи Community-lists

Отметь галочки для списков из секции выше (discord, meta, telegram, twitter, youtube, cloudflare, tiktok).

### Шаг 3. Вставь домены

1. Открой файл [`user-domains.txt`](./user-domains.txt) в raw-виде (кнопка **Raw** на GitHub)
2. Выдели весь текст (`Ctrl+A`) → скопируй (`Ctrl+C`)
3. В Podkop установи `Custom Domain List Type` → **Text**
4. В поле **User Domains (text)** вставь скопированный текст
5. Сохрани

### Шаг 4. Вставь подсети

1. Открой файл [`user-subnets.txt`](./user-subnets.txt) в raw-виде
2. Выдели весь текст → скопируй
3. Установи `Custom Subnet List Type` → **Text**
4. В поле **User Subnets (text)** вставь скопированный текст

### Шаг 5. Рекомендуемые настройки Podkop

В Podkop → **Settings**:

| Параметр | Значение | Зачем |
|----------|----------|-------|
| `DNS rewrite TTL` | `30` | Быстрая реакция на смену endpoint (особенно для Spotify/Discord Rich Presence) |
| `Disable QUIC` | ✅ включено | QUIC через TPROXY нестабилен с Hysteria 2 |
| `Download lists via proxy` | ✅ включено | Community-lists с GitHub скачиваются через VPN (если GitHub заблокирован) |
| `DNS type` | `DoH` (DNS-over-HTTPS) | Провайдеры в РФ часто блокируют UDP:53 |
| `Main DNS` | `1.1.1.1` или `8.8.8.8` | Cloudflare или Google DNS |
| `Bootstrap DNS` | `8.8.8.8` | Для начального резолва DoH-сервера |

### Шаг 6. Save & Apply

Нажми **Save & Apply** в Podkop. Подожди ~15-30 секунд (sing-box перезагрузит конфиг и скачает community-lists).

### Шаг 7. Проверка

В Podkop открой **Diagnostic → Global check** — все пункты должны быть ✅ зелёными.

Практический тест — зайди с устройства за роутером на:
- `linkedin.com`
- `claude.ai`
- `chatgpt.com`
- `netflix.com`
- `medium.com`

Все должны открываться без VPN-клиента на устройстве.

## 🗂️ Что покрыто в списках

<details>
<summary><b>Категории доменов</b> (клик чтобы развернуть)</summary>

- **Spotify** — расширенный набор (региональные spclient'ы, dealer WebSocket'ы, CDN) для работы Discord Rich Presence
- **TikTok** — расширенный (не все CDN покрыты community)
- **Xiaomi Mi Home / IoT** — требуется для работы приложения Mi Home через Podkop
- **Google сервисы** — DNS, Ads/Analytics/Tag Manager, шрифты, Firebase, Cloud APIs, региональные домены
- **Figma** — для работы (дизайн)
- **AI-сервисы** — Claude/Anthropic, OpenAI/ChatGPT, Gemini, Perplexity, Midjourney, Runway, Suno, ElevenLabs, DeepSeek, HuggingFace, Poe и другие
- **Соцсети** — LinkedIn, Reddit, Medium, Substack, Pinterest, Tumblr, Quora, Stack Overflow
- **Стриминг видео** — Netflix, HBO Max, Disney+, Hulu, Crunchyroll, Apple TV+, Paramount+
- **Стриминг музыки** — SoundCloud, Deezer, Tidal, Apple Music, Bandcamp
- **Twitch**
- **Игры** — EA/Apex Legends, Blizzard, Riot (LoL/Valorant), Ubisoft, Rockstar, Epic Games, Roblox, Minecraft
- **Dev / SaaS** — GitHub (+Copilot), Atlassian (Jira/Trello/Bitbucket), Notion, Slack, Zoom, Vercel, Netlify, Heroku, Fly.io, Supabase
- **Adobe, JetBrains, Microsoft VSCode, Dropbox, Box** — ушедшие лицензии
- **Proton, Signal, Tor, Mullvad, Tutanota** — приватность
- **Платежи** — PayPal, Stripe, Wise, Revolut, SWIFT, Visa, Mastercard
- **Крипто-биржи** — Binance, Bybit, Coinbase, Kraken, OKX, MEXC и др.
- **Новости** — BBC, Reuters, NYT, WSJ, Bloomberg, Guardian, Euronews, Meduza, Novaya, Currenttime, TheIns, Mediazona и другие
- **Обучение** — Coursera, Udemy, edX, Duolingo, Skillshare, Masterclass
- **Travel / Shopping** — Airbnb, Booking, eBay, Etsy, Envato

</details>

<details>
<summary><b>Категории подсетей</b> (клик чтобы развернуть)</summary>

- **Google DNS** (`8.8.8.0/24`, `8.8.4.0/24`) — провайдеры РФ часто блокируют
- **Google Cloud Platform** — хостинг Spotify, Pokemon GO и тысячи других сервисов
- **Google основные подсети** — поиск, Gmail, Maps, Drive, Docs
- **EA / Apex Legends / Respawn** — включая EA Multiplay серверы
- **Blizzard Battle.net**
- **Riot Games** (LoL, Valorant матч-серверы)
- **Rockstar Games**
- **Ubisoft**
- **Spotify** (дополнительные подсети к GCP)
- **LinkedIn**
- **Netflix** (актуальные 11 блоков)
- **Twitch**
- **BBC**
- **ProtonVPN / ProtonMail**
- **TikTok региональные CDN**

Закомментированы по умолчанию (раскомментируй если нужно):
- **DigitalOcean** — если не используешь community-list `digitalocean`
- **Hetzner** — если не используешь community-list `hetzner`
- **OVH** — если не используешь community-list `ovh`
- **Fastly** (`151.101.0.0/16`) — для Reddit, но затрагивает тысячи других сайтов
- **Apple** (`17.0.0.0/8`) — весь Apple, может замедлить обновления iOS/macOS

</details>

##  Кастомизация

### Добавить свой сервис

В Podkop, в поле User Domains (text) допиши новую секцию:

```
// ===== Мой сервис =====
example.com
api.example.com
```

И нажми **Save & Apply**.

### Убрать категорию

Если не хочешь пропускать через VPN, например, стриминг музыки — найди секцию `// ===== Стриминг музыки =====` и закомментируй (добавь `//` в начало) или удали строки.

### Проверить что домен идёт через VPN

С устройства за роутером:
```bash
nslookup linkedin.com
```

Если в ответе IP из диапазона `198.18.0.0/15` — **работает** (это FakeIP от sing-box). Реальный IP означает что домен НЕ проксируется.

## 🐛 Troubleshooting

### Sing-box не стартует после добавления

Проверь логи:
```bash
logread | grep -E "podkop|sing-box" | tail -50
```

Скорее всего опечатка в домене или невалидный CIDR в подсетях. Ищи строку с ошибкой.

### Какой-то сайт не открывается хотя должен

1. Сбрось DNS-кэш на устройстве:
   - Windows: `ipconfig /flushdns`
   - macOS: `sudo killall -HUP mDNSResponder`
   - Linux: `sudo systemd-resolve --flush-caches`
2. Сбрось кэш FakeIP на роутере:
   ```bash
   ssh root@192.168.1.1
   rm /tmp/sing-box/cache.db
   service podkop restart
   ```

### Spotify Rich Presence в Discord тормозит

Убедись что:
- TTL = 30 в настройках Podkop
- QUIC отключён
- Discord перезапущен **полностью** (выйти из трея и открыть заново) после изменения списков

### Какая-то подсеть ломает работу (трафик идёт не туда)

Закомментируй её в `user-subnets.txt` (добавь `//` в начало), обнови в Podkop, Save & Apply.

Частые кандидаты на отключение:
- `17.0.0.0/8` (Apple)
- `151.101.0.0/16` (Fastly — слишком широкий)
- Подсети Hetzner/OVH/DigitalOcean если они перекрываются с community-lists

## 📊 Совместимость

| Роутер | Flash | RAM | Протестировано |
|--------|-------|-----|----------------|
| ASUS TUF-AX4200 | 256 MB | 512 MB | ✅ |
| Cudy TR3000 (256 MB) | 256 MB | 256 MB | ✅ |
| Cudy TR3000 (128 MB) | 128 MB | 256 MB | ⚠️ впритык, может не хватить места |

На роутерах со 128 MB flash **рекомендую ставить только список доменов**, подсети пропустить (или оставить только критичные — Google Cloud, EA).

## Вклад

Если заметил что какой-то сервис не работает через VPN или наоборот зачем-то идёт через VPN (и не должен) — открой [Issue](../../issues) или Pull Request.

Приветствуется:
- Добавление новых заблокированных/ушедших сервисов
- Обновление подсетей (IP-блоки меняются со временем)
- Фидбек по работе конкретных игр (особенно Apex Legends)

## 📜 Лицензия

Свободное использование и модификация. Author: AxelNerv

## 🔗 Полезные ссылки

- [Podkop на GitHub](https://github.com/itdoginfo/podkop)
- [Allow-domains — community-lists от itdoginfo](https://github.com/itdoginfo/allow-domains)
- [Документация Podkop](https://podkop.net/docs/)
- [OpenWrt](https://openwrt.org/)
- [Sing-box](https://sing-box.sagernet.org/)
- [Hysteria 2](https://v2.hysteria.network/)

---
