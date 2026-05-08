# Podkop Lists — домены и подсети для OpenWrt + Hysteria 2

Готовые списки доменов и подсетей для [**Podkop**](https://github.com/itdoginfo/podkop) на OpenWrt-роутерах. Покрывают заблокированные в РФ сервисы, ушедшие с рынка РФ сервисы, а также домены которые не работают через community-lists по умолчанию.

Протестировано на **ASUS TUF-AX4200** и **Cudy TR3000** (256 MB flash), OpenWrt 24.10+, Podkop v0.7.14+, Sing-box 1.12.22+.

---

## 📋 Файлы

| Файл | Строк |
|------|-------|
| [`user-domains.txt`](./user-domains.txt) | ~600 |
| [`user-subnets.txt`](./user-subnets.txt) | ~120 |

---

## 🚀 Быстрый старт

### Требования

- OpenWrt 24.10+
- Podkop v0.7.14+ ([установка](https://github.com/itdoginfo/podkop#установка))
- Hysteria 2 / VLESS или другой прокси уже настроен

### Шаг 1. Открой Podkop

`http://192.168.1.1` → **Services → Podkop → Sections → main**

### Шаг 2. Включи Community-lists

Обязательно включи галочки:

- `cloudflare`
- `discord` *(включи, даже если добавляешь Discord через внешний список)*
- `meta`
- `russia_inside`
- `telegram`
- `twitter`

### Шаг 3. Подключи списки через внешние URL

Это самый надёжный способ — роутер скачивает файлы сам напрямую с GitHub.

В Podkop → **Sections → main**:

1. `Custom Domain List Type` → **Disabled** (текстовое поле не нужно)
2. `Custom Subnet List Type` → **Disabled**
3. В поле **External domain lists** добавь:
   ```
   https://raw.githubusercontent.com/AxelNerv/Openwrt-Podkop-hy2/main/user-domains.txt
   ```
4. В поле **External subnet lists** добавь:
   ```
   https://raw.githubusercontent.com/AxelNerv/Openwrt-Podkop-hy2/main/user-subnets.txt
   ```
5. Убедись что включена опция **Download lists via proxy** — иначе GitHub может не скачаться без VPN

### Шаг 4. Рекомендуемые настройки

В Podkop → **Settings**:

| Параметр | Значение | Зачем |
|----------|----------|-------|
| `DNS type` | `DoH` | Провайдеры РФ блокируют UDP:53 |
| `Main DNS` | `1.1.1.1` | Cloudflare DoH |
| `Bootstrap DNS` | `8.8.8.8` | Для начального резолва DoH |
| `DNS rewrite TTL` | `30` | Быстрая реакция на смену IP |
| `Disable QUIC` | ✅ | QUIC нестабилен с Hysteria 2 через TPROXY |
| `Download lists via proxy` | ✅ | GitHub скачивается через VPN |
| `Update interval` | `12h` | Автообновление списков |

### Шаг 5. Save & Apply

Нажми **Save & Apply**. Подожди 15–30 секунд.

### Шаг 6. Проверка

В Podkop → **Diagnostic → Global check** — все пункты должны быть ✅.

Практический тест — с устройства за роутером:
```bash
nslookup linkedin.com
```
Если IP из диапазона `198.18.x.x` — **работает** (FakeIP от sing-box). Реальный IP — домен не проксируется.

---

## 🗂️ Что покрыто

<details>
<summary><b>Домены</b></summary>

- **Spotify** — расширенный (spclient'ы, dealer WebSocket'ы, CDN) для Discord Rich Presence
- **TikTok** — полный набор включая российские и региональные API-эндпоинты
- **Xiaomi Mi Home / IoT**
- **Figma**
- **AI-сервисы** — Claude, ChatGPT, Gemini, Perplexity, Midjourney, Suno, ElevenLabs, DeepSeek, HuggingFace и другие
- **Discord** — полный набор (основные домены, CDN, gateway, голосовые регионы)
- **GitHub** — включая Copilot и Codespaces
- **Стриминг** — Netflix, HBO Max, Disney+, Hulu, Crunchyroll, Apple TV+, SoundCloud, Deezer, Tidal, Twitch
- **Игры** — EA/Apex Legends, Blizzard/Battle.net, Ubisoft, Rockstar, Epic Games, Bungie/Destiny, Mihoyo/Hoyoverse, CD Projekt RED, Bethesda, 2K Games, Square Enix, Minecraft, VRChat
- **Dev / SaaS** — Atlassian, Notion, Slack, Zoom, Vercel, Netlify, Heroku, Fly.io, Supabase
- **Adobe, JetBrains, Microsoft, Apple, Dropbox**
- **Proton, Signal, Tor, Mullvad, Tutanota**
- **Платежи** — PayPal, Stripe, Wise, Revolut, Visa, Mastercard
- **Крипто** — Binance, Bybit, Coinbase, Kraken, OKX, MEXC и другие
- **Новости** — BBC, Reuters, NYT, Bloomberg, Guardian, Meduza, Mediazona и другие
- **Обучение** — Coursera, Udemy, edX, Duolingo, Skillshare
- **LinkedIn, Reddit, Medium, Substack**

</details>

<details>
<summary><b>Подсети</b></summary>

- **Google DNS** — `8.8.8.0/24`, `8.8.4.0/24`
- **Google Cloud Platform** — хостинг Spotify, Pokemon GO и других сервисов
- **TikTok / ByteDance** — AS396986, Volcengine CDN, Akamai ноды
- **Discord** — AS49544 (voice/media), Cloudflare gateway (`162.159.128.0/22`), `104.16.0.0/12`
- **Microsoft / Windows Update** — AS8075, Office 365, Azure CDN
- **EA / Apex Legends** — AS35995
- **Blizzard / Battle.net** — AS57976
- **Ubisoft, Rockstar, Bungie, Mihoyo**
- **Netflix** — 11 актуальных блоков
- **LinkedIn, BBC, ProtonVPN, Stripe**

Закомментированы (раскомментируй если нужно):
- `17.0.0.0/8` — весь Apple
- `151.101.0.0/16` — Fastly (Reddit + тысячи других)
- Cloudflare полный список
- AWS CloudFront
- DigitalOcean, Hetzner, OVH

</details>

---

## ⚠️ Важные нюансы

**Discord** — community-list `discord` включи обязательно, даже если в внешних списках уже есть Discord. Подсеть `104.16.0.0/12` добавлена специально для фикса пинга 5000+ в голосовых каналах.

**TikTok** — домены с суффиксом `-ru` (`frontier-ru`, `libra-ru` и т.д.) намеренно **не включены** — они ведут на российские CDN-ноды и их проксирование ломает работу приложения.

**Steam** — основные домены закомментированы. Игровые серверы Steam лучше не проксировать — пинг вырастет. Оставлены только `valvesoftware.com`, `csgo.com`, `dota2.com`.

---

## 🐛 Troubleshooting

**Список не применяется / sing-box не стартует**
```bash
logread | grep -E "podkop|sing-box" | tail -50
```

**Сбросить FakeIP кэш**
```bash
rm /tmp/sing-box/cache.db
/etc/init.d/podkop restart
```

**Проверить что домен попал в список**
```bash
grep tiktok /tmp/podkop/*.txt 2>/dev/null | head -5
```

**Сброс DNS-кэша на устройстве**
- Windows: `ipconfig /flushdns`
- macOS: `sudo killall -HUP mDNSResponder`

---

## 📊 Совместимость

| Роутер | Flash | RAM | Статус |
|--------|-------|-----|--------|
| ASUS TUF-AX4200 | 256 MB | 512 MB | ✅ |
| Cudy TR3000 (256 MB) | 256 MB | 256 MB | ✅ |
| Cudy TR3000 (128 MB) | 128 MB | 256 MB | ⚠️ впритык |

На роутерах со 128 MB flash рекомендую подключать только домены, подсети пропустить или оставить минимум.

---

## 🔗 Ссылки

- [Podkop](https://github.com/itdoginfo/podkop)
- [Allow-domains (community-lists)](https://github.com/itdoginfo/allow-domains)
- [Sing-box](https://sing-box.sagernet.org/)
- [Hysteria 2](https://v2.hysteria.network/)
- [OpenWrt](https://openwrt.org/)
