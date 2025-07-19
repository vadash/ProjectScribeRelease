# ProjectScribeRelease

Проект состоит из двух частей:
1) Worker на cloudflare сервере общается с gemini нейросеткой, позволяет использовать несколько ключей без бана
2) Клиент, который собственно и конвертит книжки

Конвертация проходит в несколько этапов: составление голосового прортрета для главных персонажей (голос всеравно не будет 1 в 1, но будет похоже), разбитие книги на блоки и преобразование книги в transcript готовый для tts. Примерно 1 из 50 блоков гемини отказывается озвучивать (больше для smut и порно), тут после 3го фейла включается наш любимый edge TTS одноголосый. Потом идет постобработка (нормализация, удаление тишины) и склейка в opus

# Пример

Сперва послушайте пример и решите стоит ли оно того https://github.com/vadash/ProjectScribeRelease/releases/download/6.4/khameleon10-part001.opus

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
12) Добавляем PASS случайный - это ваш ключ доступа к серверу

<img width="1251" height="1167" alt="image" src="https://github.com/user-attachments/assets/937651b5-0855-4c98-b24c-bf51ffb3c29e" />

13) Повторно закачиваем zip файл, ошибки не будет, будет зеленая галка (после каждого изменения ключей нужно повторить)

## Client

1) Ставим NET 9 https://dotnet.microsoft.com/en-us/download/dotnet/thank-you/runtime-desktop-9.0.7-windows-x64-installer (на XP не запустится)
2) Ставим ffmpeg https://github.com/BtbN/FFmpeg-Builds/releases `ffmpeg-master-latest-win64-gpl.zip` распаковываем в папку (например, `C:\portable\ffmpeg\bin`)
3) Добавляем в PATH

<img width="222" height="274" alt="image" src="https://github.com/user-attachments/assets/25f8533d-f661-4b8a-a634-a10b41238afb" />

4) Скачиваем https://github.com/vadash/ProjectScribeRelease/releases ProjectScribe_XXX.7z
5) Распаковываем в папку (например, `C:\portable\ProjectScribe`)
6) `.env` создаем в корне, меняем `sk-proj-REDACTED` на ключ (PASS) из 12 пункта

```
OpenAi__OpenAiApiKey="sk-proj-REDACTED"
Tts__GeminiTtsApiKey="sk-proj-REDACTED"
```
7) Редактируем `config.json` `OpenAiApiEndpoint` (например, `https://gemini-openai-adapter-public.pages.dev/v1/chat/completions/1`) и `GeminiTtsApiEndpoint` (например, `https://gemini-openai-adapter-public.pages.dev/tts/1`). Эти адреса берем из worker settings

<img width="1348" height="417" alt="image" src="https://github.com/user-attachments/assets/4d79420c-b8d3-4408-bbba-02b494874d9a" />

Тут же меняем путь до библиотеки `BaseLibraryDirectory` (два слеша не забываем)

# Настройка

Осуществляется через редактирование `config.json`

## ffmpeg

Две интересные опции - удаляет тишину больше Х сек и нормализация громкости. Обе включены по умолчанию

## CF

Можно регулировать число потоков обработки `MaxConcurrentTtsThreads` и размер блока `TextBlockTargetCharCount`. На бесплатном плане хорошо работает 2 и 1600. Второй можно немного снизить до 1400 если много timeout ошибок. Больше 1600 не поставить, так как cloudflare worker имеет жесткий timeout 100 sec, а гемини ттс не поддерживает streaming

## Промпты

В папке `prompts` их 4. Они не оптимальны, можно переделать под свои задачи

# Планы

Добавить txt, fb2 на вход

Добавить словарь

# QnA

## Все пошло по П????, хочу сбросить сохраненый прогресс конвертации

1) Удаляем папку data из `ProjectScribe`
2) Удаляем папки из библиотеки кроме 001 and 100

<img width="257" height="292" alt="image" src="https://github.com/user-attachments/assets/faeee82a-062e-4ed7-a8e1-f56378521cc1" />

## Если файл не конвертится возможно epub битый, парсер слабоват

Ставим Sigil https://sigil-ebook.com/sigil/download открываем файл, он ругается, нажимаем ок, сохраняем

