# Установка форка в Homebridge (Docker)

Инструкция по установке этого форка (`no9821nda/homebridge-fibaro-home-center`)
вместо оригинального npm-пакета в Homebridge, запущенный в официальном
Docker-образе [`homebridge/homebridge`](https://hub.docker.com/r/homebridge/homebridge).

Форк устанавливается напрямую с GitHub: при установке npm клонирует репозиторий
и скрипт `prepare` автоматически собирает плагин из исходников (`dist/`).

## Требования

- Официальный Docker-образ Homebridge, плагины в томе `/var/lib/homebridge`.
- **Node.js ≥ 20.19** внутри контейнера (нужен для `require(ESM)`-совместимости
  с homebridge 2). Проверка — шаг 2 ниже.
- **git** внутри контейнера (нужен npm для установки из GitHub; в официальном
  образе присутствует).

## Установка

1. Зайти в контейнер (подставьте имя своего контейнера):

   ```bash
   docker exec -it homebridge bash
   ```

2. Проверить окружение:

   ```bash
   node -v    # должно быть >= 20.19 (или 22.12+, 24+)
   git --version
   ```

   Если Node старее — обновите образ и пересоздайте контейнер
   (`docker pull homebridge/homebridge:latest`); том с конфигом сохранится.

3. Установить форк в директорию плагинов:

   ```bash
   cd /var/lib/homebridge
   npm install --save github:no9821nda/homebridge-fibaro-home-center#v3.3.0
   ```

   Тег `#v3.3.0` фиксирует конкретную версию. Для установки текущего `main`
   уберите суффикс с тегом.

4. Перезапустить Homebridge — из веб-UI или с хоста:

   ```bash
   docker restart homebridge
   ```

## Проверка

- В веб-UI на вкладке «Плагины» у `homebridge-fibaro-home-center` должна
  отображаться версия **3.3.0** (версия форка; в npm — 3.2.2).
- В логах Homebridge при старте — обычная инициализация платформы `FibaroHC`
  и строка `Successfully logged in.`.

## Почему конфигурацию менять не нужно

Имя пакета и имя платформы (`FibaroHC`) совпадают с оригиналом, поэтому:

- существующий блок в `config.json` продолжает работать без изменений;
- кэшированные аксессуары и автоматизации HomeKit сохраняются;
- форк просто замещает оригинальный пакет в `node_modules`.

Запись в `/var/lib/homebridge/package.json` (флаг `--save`) переживает
пересоздание контейнера: стартовый скрипт официального образа сам
доустанавливает пакеты из него.

## Обновление форка

После изменений в репозитории (коммит в `main` или новый тег):

```bash
docker exec -it homebridge bash
cd /var/lib/homebridge
npm install --save github:no9821nda/homebridge-fibaro-home-center        # свежий main
# или, для конкретного тега:
npm install --save github:no9821nda/homebridge-fibaro-home-center#v3.4.0
docker restart homebridge
```

> Кнопка «Update» в веб-UI для git-установок не работает — обновление
> всегда выполняется командой выше. Откат на оригинал при этом UI не
> предложит: версия форка (3.3.0) выше опубликованной в npm (3.2.2).

## Возврат к оригинальному плагину

```bash
cd /var/lib/homebridge
npm install --save homebridge-fibaro-home-center@latest
docker restart homebridge
```

## Диагностика

| Симптом | Причина / решение |
|---|---|
| `npm ERR! ... git` при установке | В контейнере нет git — обновите образ или установите вручную (`apt-get update && apt-get install -y git`). |
| Ошибка `ERR_REQUIRE_ESM` / плагин не загружается | Node в контейнере старее 20.19 — обновите образ. |
| Долгая установка | Нормально: `prepare` компилирует TypeScript при установке из git. |
| После пересоздания контейнера плагин «пропал» | Убедитесь, что устанавливали с `--save` и том `/var/lib/homebridge` смонтирован. |
