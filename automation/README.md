# Автоматизация

Набор автоматических задач для роутера **OpenWrt + podkop**: обновления, обслуживание, профилактика.

Всё здесь живёт **вне пакета podkop** — в `/root/` и `/etc/crontabs/root`. Пакетный менеджер удаляет только файлы своего пакета, поэтому обновление podkop эти скрипты **не сносит**.

| Задача | Расписание | Зачем |
|---|---|---|
| [Автообновление podkop](#1-автообновление-podkop--luci-app-podkop) | Пн 03:30 | Всегда свежая версия без ручной проверки |
| [Сброс fakeip-кэша](#2-ежедневный-сброс-fakeip-кэша) | Ежедневно 05:00 | Лечит «старое грузится, новое нет» (TikTok и т.п.) |
| [Перезагрузка роутера](#3-еженедельная-перезагрузка-роутера) | Пн 04:00 | Профилактика от деградации скорости |

> **sing-box сознательно не обновляется автоматически.** Это ядро маршрутизации: неудачное обновление оставляет роутер без интернета, а откатывать приходится руками. Его лучше обновлять осознанно и когда есть доступ к устройству.

---

## 1. Автообновление podkop + luci-app-podkop

### Что делает

Раз в неделю скрипт:

1. Спрашивает у GitHub последнюю версию podkop
2. Сравнивает с установленной — **если совпадает, просто выходит**, ничего не трогая
3. Если вышла новее — скачивает `podkop` и `luci-app-podkop`, ставит через `opkg` (**конфиг сохраняется**), убирает за собой временные файлы
4. Пишет результат в лог с датой; лог сам обрезается, чтобы не разрастался
5. Проверяет, на месте ли его строка в cron — **и восстанавливает её, если затёрли**

### Шаг 1. Создать скрипт

```sh
cat > /root/podkop-autoupdate.sh << 'EOF'
#!/bin/sh
# Еженедельное автообновление podkop + luci-app-podkop
# Живёт ВНЕ пакета podkop -> обновления podkop этот файл не трогают.
# sing-box намеренно НЕ обновляется.

LOG=/root/podkop-update.log
API=https://api.github.com/repos/itdoginfo/podkop/releases/latest
CRON=/etc/crontabs/root
LINE='30 3 * * 1 /root/podkop-autoupdate.sh'

log() { echo "$(date '+%F %H:%M') $*" >> "$LOG"; }

if ! grep -qF '/root/podkop-autoupdate.sh' "$CRON" 2>/dev/null; then
	echo "$LINE" >> "$CRON"
	/etc/init.d/cron restart >/dev/null 2>&1
	log "строка cron восстановлена"
fi

if [ -f "$LOG" ] && [ "$(wc -c < "$LOG")" -gt 51200 ]; then
	tail -n 100 "$LOG" > "$LOG.tmp" && mv "$LOG.tmp" "$LOG"
fi

CUR=$(opkg list-installed podkop 2>/dev/null | awk '{print $3}' | sed 's/^v//; s/-r[0-9]*$//')
JSON=$(wget -qO- "$API" 2>/dev/null)
LATEST=$(echo "$JSON" | grep -o '"tag_name": *"[^"]*"' | head -1 | cut -d'"' -f4)

if [ -z "$LATEST" ]; then
	log "GitHub недоступен - пропуск"
	exit 0
fi

if [ "$CUR" = "$LATEST" ]; then
	log "актуально ($CUR) - обновление не требуется"
	exit 0
fi

log "есть обновление: $CUR -> $LATEST"

PK=$(echo "$JSON" | grep -o 'https://[^"]*\.ipk' | grep '/podkop-v' | head -1)
LA=$(echo "$JSON" | grep -o 'https://[^"]*\.ipk' | grep '/luci-app-podkop-v' | head -1)

if [ -z "$PK" ] || [ -z "$LA" ]; then
	log "не нашёл .ipk в релизе - пропуск"
	exit 0
fi

cd /tmp || exit 0
if ! wget -qO pk.ipk "$PK"; then
	log "ошибка скачивания podkop"
	rm -f pk.ipk
	exit 0
fi
if ! wget -qO la.ipk "$LA"; then
	log "ошибка скачивания luci-app-podkop"
	rm -f pk.ipk la.ipk
	exit 0
fi

opkg install /tmp/pk.ipk /tmp/la.ipk >> "$LOG" 2>&1
NEW=$(opkg list-installed podkop 2>/dev/null | awk '{print $3}')
log "установлено: podkop $NEW"

rm -f /tmp/pk.ipk /tmp/la.ipk /tmp/luci-indexcache* 2>/dev/null
exit 0
EOF
```

### Шаг 2. Права, расписание, защита от перепрошивки

```sh
chmod +x /root/podkop-autoupdate.sh; grep -qF '/root/podkop-autoupdate.sh' /etc/crontabs/root || echo '30 3 * * 1 /root/podkop-autoupdate.sh' >> /etc/crontabs/root; /etc/init.d/cron enable; /etc/init.d/cron restart; grep -qF '/root/podkop-autoupdate.sh' /etc/sysupgrade.conf 2>/dev/null || printf '/root/podkop-autoupdate.sh\n/root/podkop-update.log\n' >> /etc/sysupgrade.conf
```

Добавление в `sysupgrade.conf` означает, что скрипт переживёт даже перепрошивку роутера.

### Шаг 3. Проверить

```sh
/root/podkop-autoupdate.sh; cat /root/podkop-update.log; crontab -l
```

Ожидаемый вывод:

```
2026-07-31 11:34 строка cron восстановлена
2026-07-31 11:34 актуально (0.7.21) - обновление не требуется
```

и строка `30 3 * * 1 /root/podkop-autoupdate.sh` в расписании. Значит всё работает.

### Шпаргалка

Посмотреть историю обновлений:

```sh
cat /root/podkop-update.log
```

Проверить прямо сейчас, не дожидаясь понедельника:

```sh
/root/podkop-autoupdate.sh
```

Отключить совсем — удалять **и строку, и файл**, иначе скрипт вернёт себя в расписание:

```sh
sed -i '/podkop-autoupdate/d' /etc/crontabs/root; rm -f /root/podkop-autoupdate.sh; /etc/init.d/cron restart
```

### Изменить время запуска

`30 3 * * 1` = понедельник 03:30. Формат: `минуты часы * * день_недели`, где день недели: `0` — воскресенье, `1` — понедельник, ... `6` — суббота.

Менять нужно **в двух местах**: в переменной `LINE=` внутри скрипта **и** в `/etc/crontabs/root`. Иначе самовосстановление вернёт старое время.

> **Совет.** Если на роутере включена еженедельная перезагрузка — ставьте обновление за полчаса до неё. Тогда ребут аккуратно подхватит новую версию.

---

## 2. Ежедневный сброс fakeip-кэша

### Симптом

Сервис частично работает: **старый (закэшированный) контент открывается, а новый не грузится**. Чаще всего это TikTok — старые видео играют, новые бесконечно крутятся. Остальное при этом работает.

### Причина

sing-box хранит fakeip-привязки в `/tmp/sing-box/cache.db`. Сервисы вроде TikTok часто меняют IP серверов, и старые записи начинают указывать на мёртвые адреса.

### Установка

```sh
grep -q 'sing-box/cache.db' /etc/crontabs/root 2>/dev/null || echo '0 5 * * * rm -f /tmp/sing-box/cache.db && /etc/init.d/podkop restart' >> /etc/crontabs/root; /etc/init.d/cron enable; /etc/init.d/cron restart
```

Разовое ручное лечение, если приспичило прямо сейчас:

```sh
rm -f /tmp/sing-box/cache.db && /etc/init.d/podkop restart
```

После этого выгрузить приложение из списка запущенных и открыть заново.

---

## 3. Еженедельная перезагрузка роутера

Профилактика: на части роутеров со временем деградирует скорость или подтекает память, и ребут это лечит. Раз в неделю ночью — незаметно.

```sh
grep -q '/sbin/reboot' /etc/crontabs/root 2>/dev/null || echo '0 4 * * 1 /sbin/reboot' >> /etc/crontabs/root; /etc/init.d/cron enable; /etc/init.d/cron restart
```

---

## Итоговое расписание

```
Пн 03:30        обновление podkop + luci-app-podkop
Пн 04:00        перезагрузка роутера
Ежедневно 05:00 сброс fakeip-кэша + рестарт podkop
Каждый час :13  обновление списков (podkop добавляет сам)
```

Цепочка выстроена намеренно: сначала обновление, затем чистый ребут, затем сброс кэша — к утру роутер полностью свежий.

Проверить, что всё на месте:

```sh
crontab -l
```

---

## Оговорки

- Рассчитано на OpenWrt с пакетным менеджером **opkg** (23.05 / 24.10). На OpenWrt 25.x и новее используется `apk`: в скрипте заменить `opkg list-installed podkop` на `apk list -I podkop`, а `opkg install` на `apk add --allow-untrusted`.
- Роутеру нужен доступ к `api.github.com` и `github.com`. Если они блокируются провайдером — домены должны идти через прокси podkop.
- Скрипт обновляет **только podkop и luci-app-podkop**. sing-box, прошивка и прочие пакеты не затрагиваются.
- Все задачи безопасны при повторном запуске: строки в cron не дублируются, а автообновление при отсутствии новой версии просто выходит.
