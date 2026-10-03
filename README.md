# ЛУЧ v1.7

Голосовой ассистент для Arch Linux: распознавание речи, синтез голоса, локальные и
облачные LLM, управление компьютером и **Android-телефоном**, веб-интерфейс.

Версия 1.7 — первая с полноценным Android-клиентом: телефон подключается к тому же
веб-серверу, ассистент управляет им голосом и видит его местоположение.

---

## Что умеет

| Возможность | Модуль |
|---|---|
| Распознавание речи (faster-whisper локально или Google) | `main.py` |
| Голосовой пропуск — ассистент реагирует только на ваш тембр | `main.py` (speechbrain ECAPA) |
| Синтез речи (Silero TTS, русские голоса) | `main.py` |
| Локальные модели через Ollama | `main.py` |
| Облачные модели: OpenCode Zen, любой OpenAI-совместимый провайдер | `main.py` |
| Управление ПК: терминал, блокировка, таймеры, поиск | `work_fuctions.py` |
| Навигация, геокодирование, поиск мест рядом | `work_fuctions.py` |
| Управление Android-телефоном и обмен сообщениями | `device_control.py` |
| Android-приложение (Kotlin) | `android/` |
| Веб-интерфейс и REST API | `web_server.py`, `static/`, `index.html` |
| Терминальное меню и GUI-монитор | `console_ui.py`, `gui_launcher.py` |

---

## Структура

```
v1.7/
├── main.py                 ядро: голосовой цикл, STT/TTS, работа с LLM
├── web_server.py           FastAPI: REST API, HTTPS, статика
├── work_fuctions.py        инструменты ассистента (shell, навигация, таймеры)
├── device_control.py       реестр устройств и очередь команд для Android
├── settings.py             загрузка/сохранение settings.json
├── console_ui.py           терминальное меню выбора (rich)
├── gui_launcher.py         GUI-монитор и запуск ядра (PySide6)
├── index.html, static/     веб-интерфейс
└── android/                исходники Android-приложения (Kotlin) и APK
```

---

## Установка

```bash
python -m venv ai_env
source ai_env/bin/activate

# torch/torchaudio для CUDA — из отдельного индекса
pip install torch torchaudio --index-url https://download.pytorch.org/whl/cu130

pip install faster-whisper speechbrain sounddevice torch torchaudio \
    silero-tts ollama rich PySide6 fastapi uvicorn httpx
```

---

## Запуск

```bash
# ядро + веб-интерфейс
python main.py

# GUI-монитор (запускает ядро сам)
python gui_launcher.py
```

При первом запуске ассистент попросит произнести длинную фразу — это эталон
вашего голоса для голосового пропуска (файл `my_voice_profile.wav`).

Веб-интерфейс: `http://<ip>:1337`, защищённый (нужен для микрофона в браузере):
`https://<ip>:1338`.

---

## Управление телефоном

Телефон ставит APK, указывает адрес сервера и регистрируется в `device_control.py`.
После этого ИИ получает команды:

| Команда | Действие |
|---|---|
| `phone_status` | состояние телефона |
| `phone_wake` | разбудить экран |
| `phone_volume` | громкость вверх/вниз |
| `phone_brightness` | яркость в процентах |
| `phone_torch` | фонарик |
| `phone_toast` | всплывающее уведомление |
| `phone_open_settings` | открыть настройки |
| `phone_open_app` | открыть приложение по названию |
| `phone_apps` | список установленных приложений |
| `phone_clipboard` | прочитать или записать буфер обмена |
| `phone_dial` | открыть набор с номером |
| `phone_share` | передать текст в мессенджер |
| `phone_stop_ring` | снять входящий звонок |

Также доступны `phone_battery`, `phone_location`, `phone_notify`, `phone_ring`,
`phone_vibrate`, `phone_open_url`.

---

## Безопасность

> `settings.json`, `luch-key.pem` и `memory.txt` содержат ключи в открытом виде и
> исключены из git. Не публикуйте их.

Аутентификация веб-API в этой версии **не обязательна** — это исправлено в 1.8.
Не выставляйте порт 1337/1338 в интернет.

---

## Android

Исходники: `android/app/src/main/java/proekt/luch/app/`, готовый APK:
`android/LUCH-1.7.apk`.

Приложение умеет работать через обычный HTTP — отдельный сертификат не нужен.
