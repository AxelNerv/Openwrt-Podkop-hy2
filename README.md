# Podkop Lists — домены и подсети для OpenWrt + Hysteria 2

Готовые списки для [**Podkop**](https://github.com/itdoginfo/podkop) на OpenWrt-роутерах.

Протестировано на **ASUS TUF-AX4200** и **Cudy TR3000** (256 MB flash), OpenWrt 24.10+, Podkop v0.7.17+, Sing-box 1.13.11+.

---

## 📋 Файлы

| Файл | Секция | Что содержит |
|------|--------|--------------|
| [`user-domains.txt`](./user-domains.txt) | main | Все домены **кроме Discord** |
| [`user-subnets.txt`](./user-subnets.txt) | main | Все подсети **кроме Discord** |
| [`discord-domains.txt`](./discord-domains.txt) | discord | Только домены Discord |
| [`discord-subnets.txt`](./discord-subnets.txt) | discord | Только подсети Discord |

Такое разделение позволяет направить Discord через отдельный прокси (например РУ VPN для низкого пинга), а весь остальной трафик через основной.

---

## 🚀 Быстрый старт

### Требования

- OpenWrt 24.10+
- Podkop v0.7.17+
- Hysteria 2 / VLESS или другой прокси уже настроен

### Шаг 1. Community-lists

В Podkop → **Секции → main** включи галочки:

- `cloudflare`
- `meta`
- `russia_inside`
- `telegram`
- `twitter`

> `discord` из community **не включай** — используем отдельную секцию

### Шаг 2. Секция main — основной прокси

В поле **Внешние списки доменов**:
```
https://raw.githubusercontent.com/AxelNerv/Openwrt-Podkop-hy2/main/user-domains.txt
```

В поле **Внешние списки подсетей**:
```
https://raw.githubusercontent.com/AxelNerv/Openwrt-Podkop-hy2/main/user-subnets.txt
```

### Шаг 3. Секция discord — отдельный прокси

Добавь новую секцию в Podkop → **Секции → добавить**.

Укажи подключение к нужному прокси (например РУ VPN).

В поле **Внешние списки доменов**:
```
https://raw.githubusercontent.com/AxelNerv/Openwrt-Podkop-hy2/main/discord-domains.txt
```

В поле **Внешние списки подсетей**:
```
https://raw.githubusercontent.com/AxelNerv/Openwrt-Podkop-hy2/main/discord-subnets.txt
```

### Шаг 4. Рекомендуемые настройки

В Podkop → **Настройки**:

| Параметр | Значение |
|----------|----------|
| `DNS type` | `DoH` |
| `Main DNS` | `1.1.1.1` |
| `Bootstrap DNS` | `8.8.8.8` |
| `DNS rewrite TTL` | `30` |
| `Disable QUIC` | ✅ |
| `Download lists via proxy` | ✅ |
| `Update interval` | `12h` |

### Шаг 5. Save & Apply → подожди 15–30 секунд

### Шаг 6. Проверка

```bash
nslookup linkedin.com
```
IP из диапазона `198.18.x.x` — работает. Реальный IP — не проксируется.

---

## 🗂️ Что покрыто в main

- **Spotify** — расширенный (включая Rich Presence для Discord)
- **TikTok** — полный набор включая региональные API
- **Xiaomi Mi Home / IoT**
- **Figma**
- **AI** — Claude, ChatGPT, Gemini, Perplexity, Midjourney, Suno, ElevenLabs, DeepSeek и другие
- **GitHub** — включая Copilot и Codespaces
- **Стриминг** — Netflix, HBO Max, Disney+, Hulu, Crunchyroll, Twitch, SoundCloud, Deezer, Tidal
- **Игры** — EA/Apex, Blizzard, Ubisoft, Rockstar, Epic, Bungie, Mihoyo, CD Projekt, Bethesda, 2K, Square Enix, Minecraft, VRChat, FACEIT
- **Dev / SaaS** — Atlassian, Notion, Slack, Zoom, Vercel, Netlify, Heroku, Supabase
- **Adobe, JetBrains, Microsoft, Apple, Dropbox**
- **Proton, Signal, Tor, Mullvad, Tutanota**
- **Платежи** — PayPal, Stripe, Wise, Revolut, Visa, Mastercard
- **Крипто** — Binance, Bybit, Coinbase, Kraken, OKX, MEXC и другие
- **Новости** — BBC, Reuters, NYT, Bloomberg, Guardian, Meduza, Mediazona и другие
- **Обучение** — Coursera, Udemy, edX, Duolingo, Skillshare
- **LinkedIn, Reddit, Medium, Substack**

---

## ⚠️ Важные нюансы

**Discord** — вынесен в отдельную секцию. Подсеть `104.16.0.0/12` обязательна — без неё пинг в голосовых каналах 5000+.

**TikTok** — домены `-ru` (`frontier-ru`, `libra-ru` и т.д.) **не включены** — они ведут на российские CDN и ломают работу приложения.

**Steam** — домены для скачивания не включены, пинг к игровым серверам через VPN вырастет. Оставлены только `valvesoftware.com`, `csgo.com`, `dota2.com`.

---

## 🐛 Troubleshooting

```bash
# Логи
logread | grep -E "podkop|sing-box" | tail -50

# Сбросить FakeIP кэш
rm /tmp/sing-box/cache.db && /etc/init.d/podkop restart

# Проверить домен в списке
grep discord /tmp/podkop/*.txt 2>/dev/null | head -5

# Сброс DNS на Windows
ipconfig /flushdns
```

---

## 📊 Совместимость

| Роутер | Flash | RAM | Статус |
|--------|-------|-----|--------|
| ASUS TUF-AX4200 | 256 MB | 512 MB | ✅ |
| Cudy TR3000 (256 MB) | 256 MB | 256 MB | ✅ |
| Cudy TR3000 (128 MB) | 128 MB | 256 MB | ⚠️ впритык |

---

## 🔗 Ссылки

- [Podkop](https://github.com/itdoginfo/podkop)
- [Allow-domains](https://github.com/itdoginfo/allow-domains)
- [Sing-box](https://sing-box.sagernet.org/)
- [Hysteria 2](https://v2.hysteria.network/)
