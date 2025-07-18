# ProjectScribeRelease

Проект состоит из двух частей:
1) Worker на cloudflare сервере общается с gemini, позволяет использовать несколько ключей без бана
2) Клиент, который собственно и конвертит книжки

# Установка (черновой вариант)

## GCP
1) Создаем новый гугл акк (у меня банов не было, на всякий). На андроиде если симку вытащить, то можно создать 1 акк раз в пару месяцев без номера телефона
2) https://console.cloud.google.com/ -> заходим, отказываемся от $300
3) CTRL + O -> New project -> Вводим непалевное название
4) Повторяем пункт 3 до упора (10-15 раз)
5) https://aistudio.google.com/apikey -> заходим создаем новый ключ на каждый проект -> сохраняем в блокнотик

## CF
6) Перейдите https://github.com/vadash/ProjectScribeRelease/releases
7) Скачайте последний `openai-gemini-mod-public.7z` сохраните из него любой zip файл, они все одинаковые по функционалу
8) Ознакомьтесть с гайдом https://www.cloudflare.com/ru-ru/developer-platform/products/workers/
9) Создайте worker https://dash.cloudflare.com/ -> Compute (workers) -> Workers & Pages -> Тут создаем новый worker типа pages и заливаем zip из пункта 7, будет ошибка
10) Идем в Settings -> Compatibility flags -> добавляем `nodejs_compat`
11) Добавляем от KEY1 до KEY15 из пункта 5 (цифры не важны, главное чтобы с KEYS начиналось)

<img width="1251" height="1167" alt="image" src="https://github.com/user-attachments/assets/937651b5-0855-4c98-b24c-bf51ffb3c29e" />

12) Повторно закачиваем zip файл, ошибки не будет, будет зеленая галка

## Client

Ставим NET 9, ffmpeg скачиваем шарим в PATH

Env создаем

```
OpenAi__OpenAiApiKey="sk-proj-REDACTED"
Tts__GeminiTtsApiKey="sk-proj-REDACTED"
```

В конфиг json прописываем путь до библиотеки и worker

