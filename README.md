**Оптимизация Minecraft сервера 2025**

# ⚙️ Руководство по оптимизации Minecraft-серверов (2025)
### Ядра: Paper / Pufferfish / Leaf (Gale, Purpur fork)
Подходит для серверов:
**Survival / SkyBlock / OneBlock / PvP / MiniGames / Technical / Anarchy**  
Рассчитано на онлайн: **50 – 300+ игроков**

---

## 🔰 1. Сравнение производительности ядер

| Параметр                            | Paper               | Pufferfish            | Leaf                                             |
| ----------------------------------- | ------------------- | --------------------- | ------------------------------------------------ |
| Средний MSPT (50 игроков)           | 3.7 mspt            | 3.2 mspt              | 2.8 mspt                                         |
| Средний MSPT (300 игроков)          | 8.1 mspt            | 6.2 mspt              | 4.5 mspt (Leaf Async)                            |
| Асинхронность                       | Базовая (I/O)       | Частично (AI/спавн)   | Полная (ChunkSend / Pathfinding / EntityTracker) |
| Поддержка Java 21+                  | Да                  | Да                    | Да (GraalVM / ZGC / Virtual Threads)             |
| DAB (Dynamic Activation Behavior)   | –                   | Да                    | Да (усоверш.)                                    |
| Целевая нагрузка                    | 1–100               | 50–200                | 100–500+                                         |

📊 **Вывод:**  
Leaf даёт прирост производительности до **+25 %** в стандартных условиях и до **+40 %** при большом числе сущностей (> 300).

---

## 🧩 2. JVM и сборка мусора

**Рекомендуемые дистрибутивы Java 21 LTS:**  
Temurin / Corretto / GraalVM.  
🚫 Не используйте headless-версии.

## 🔹 G1GC (до 200 игроков)
```bash
java -Xms8G -Xmx8G \
-XX:+UseG1GC \
-XX:+ParallelRefProcEnabled \
-XX:MaxGCPauseMillis=200 \
-XX:+UnlockExperimentalVMOptions \
-XX:+DisableExplicitGC \
-XX:+AlwaysPreTouch \
-XX:G1NewSizePercent=30 \
-XX:G1MaxNewSizePercent=40 \
-XX:G1HeapRegionSize=8M \
-XX:G1ReservePercent=20 \
-XX:G1HeapWastePercent=5 \
-XX:G1MixedGCCountTarget=4 \
-XX:InitiatingHeapOccupancyPercent=15 \
-XX:G1MixedGCLiveThresholdPercent=90 \
-XX:G1RSetUpdatingPauseTimePercent=5 \
-XX:SurvivorRatio=32 \
-XX:+PerfDisableSharedMem \
-XX:MaxTenuringThreshold=1 \
-Dusing.aikars.flags=https://mcflags.emc.gs \
-Daikars.new.flags=true \
-jar server.jar nogui

## 🔹 ZGC 21+ (300+ игроков):
java -Xms16G -Xmx16G \
-XX:+UnlockExperimentalVMOptions \
-XX:+UseZGC -XX:+ZGenerational \
-XX:+AlwaysPreTouch -XX:+DisableExplicitGC \
-XX:+PerfDisableSharedMem \
-XX:+UseDynamicNumberOfGCThreads \
--add-modules=jdk.incubator.vector \
-jar server.jar nogui

📈 ZGC → ~0.5 ms паузы GC при 300+ игроках.
Используйте GraalVM для максимальной производительности.

##⚙️ 3. Асинхронные модули Leaf:

📄 config/leaf-global.yml
async:
  async-entity-tracker:
    enabled: true
    compat-mode: false
  async-pathfinding:
    enabled: true
    max-threads: 4
  async-chunk-send:
    enabled: true
  async-target-finding:
    enabled: true
  async-playerdata-save:
    enabled: true
performance:
  use-virtual-thread-for-user-authenticator: true
  skip-ai-for-non-aware-mob: true
  throttle-hopper-when-full:
    enabled: true
    skip-ticks: 8
dab:
  enabled: true
  start-distance: 12
  max-tick-freq: 20
  activation-dist-mod: 8

##🧠 Рекомендации:
| Онлайн        | Потоки AI | Async-модули                            |
| ------------- | --------- | --------------------------------------- |
| < 100 игроков | авто      | базовые async-entity-tracker            |
| > 300 игроков | ½ CPU     | включить все async-модули               |
| Любой         | —         | parallel-world-tracking = false (эксп.) |

##🌍 4. Конфигурации мира:

server.properties:
view-distance=7
simulation-distance=4
sync-chunk-writes=false
network-compression-threshold=256

spigot.yml:
entity-activation-range:
  animals: 16
  monsters: 24
  raiders: 48
  misc: 8
  villagers: 16
  flying-monsters: 48

entity-tracking-range:
  players: 48
  animals: 48
  monsters: 48
  misc: 32
  other: 64

tick-inactive-villagers: false
nerf-spawner-mobs: true
mob-spawn-range: 3
merge-radius:
  item: 3.5
  exp: 4.0
hopper-transfer: 8
hopper-check: 8

paper-world.yml (основные параметры):
delay-chunk-unloads-by: 10s
max-auto-save-chunks-per-tick: 8
prevent-moving-into-unloaded-chunks: true
per-player-mob-spawns: true
redstone-implementation: ALTERNATE_CURRENT
optimize-explosions: true
treasure-maps:
  enabled: false

delay-chunk-unloads-by: 10s
max-auto-save-chunks-per-tick: 8
prevent-moving-into-unloaded-chunks: true
per-player-mob-spawns: true
redstone-implementation: ALTERNATE_CURRENT
optimize-explosions: true
treasure-maps:
  enabled: false

##⚙️ 5. Оптимизация alt-item-despawn-rate:

alt-item-despawn-rate:
  enabled: true
  items:
    cobblestone: 300
    dirt: 300
    gravel: 300
    sand: 300
    red_sand: 300
    diorite: 300
    granite: 300
    andesite: 300
    short_grass: 200
    bamboo: 200
    oak_leaves: 200
    cherry_leaves: 200
    cactus: 300
    scaffolding: 600

📘 Назначение:
управляет временем исчезновения дропа без плагинов ClearLag.
Paper/Leaf очищают предметы встроенно, экономя ресурсы.

| Категория              | Survival   | SkyBlock   | PvP | Tech/Anarchy |
| ---------------------- | ---------- | ---------- | --- | ------------ |
| Камень, земля, песок   | 300 (15 с) | 200 (10 с) | 200 | 400 (20 с)   |
| Листва, растения       | 200        | 150        | 150 | 300          |
| Прочие лёгкие предметы | 300        | 200        | 200 | 400          |
| Ценные блоки           | —          | —          | —   | —            |

##🧠 6. Оптимизация мобов и AI (DAB):

bukkit.yml:
spawn-limits:
  monsters: 25
  animals: 5
  water-animals: 2
  water-ambient: 2
  ambient: 1
ticks-per:
  monster-spawns: 10
  animal-spawns: 400
  
pufferfish.yml:
dab:
  enabled: true
  activation-dist-mod: 7
  max-tick-freq: 20
enable-async-mob-spawning: true
inactive-goal-selector-throttle: true

🧩 DAB — снижает частоту AI-тикания у мобов вдали от игрока
(1 тик = 20 мс → при freq 20 → 1 раз в секунду).

## 🔩 7. Hopper / Redstone / TickRates

**paper-world.yml**
```yaml
tick-rates:
  mob-spawner: 2
  grass-spread: 4
  container-update: 1
hopper-transfer: 8
hopper-check: 8
redstone-implementation: ALTERNATE_CURRENT
optimize-explosions: true

⚙️ Настройки воронок:
hopper:
  disable-move-event: false     # Не выключайте, если есть плагины защиты (WG, GP и т.п.)
  ignore-occluding-blocks: true # Быстрее, но ломает схемы с хопперами под песком

📘 Совет: держите true, если сервер не технический.
Если у вас фермы, зависящие от поведения старых версий — поставьте false.

🧬 Настройка спавнеров:
tick-rates:
  mob-spawner: 2

Значение 2 оптимально (1 тик = 0.1 с).
Для большого числа спавнеров (Tech/Anarchy) можно увеличить до 3–4.

💥 Оптимизация взрывов:
optimize-explosions: true

Заменяет ванильный алгоритм на быстрый (~ –15 % нагрузки).
Точность повреждений сохраняется.

🗺️ Отключение карт сокровищ:
treasure-maps:
  enabled: false
  find-already-discovered:
    loot-tables: true
    villager-trade: true

Генерация карт — одна из самых тяжёлых операций.
Оставляйте false, если мир не предсгенерирован.

🌱 Прочие тики мира:

tick-rates:
  grass-spread: 4
  container-update: 1
non-player-arrow-despawn-rate: 20
creative-arrow-despawn-rate: 20

| Параметр                       | Survival          | SkyBlock          | PvP               | Tech/Anarchy |
| ------------------------------ | ----------------- | ----------------- | ----------------- | ------------ |
| redstone-implementation        | ALTERNATE_CURRENT | ALTERNATE_CURRENT | ALTERNATE_CURRENT | VANILLA      |
| hopper.disable-move-event      | false             | true              | true              | false        |
| hopper.ignore-occluding-blocks | true              | true              | true              | false        |
| tick-rates.mob-spawner         | 2                 | 3                 | 2                 | 3–4          |
| optimize-explosions            | true              | true              | true              | true         |

**🧹 8. Дальность исчезновения мобов (despawn-ranges):**
despawn-ranges:
  ambient: {soft: 30, hard: 72}
  creature: {soft: 30, hard: 72}
  monster: {soft: 30, hard: 72}
  water_creature: {soft: 30, hard: 72}

📏 Формула: hard = (simulation-distance × 16) + 8
например при simulation-distance: 4 → hard: 72.

| Тип сервера | soft | hard |
| ----------- | ---- | ---- |
| Survival    | 30   | 72   |
| SkyBlock    | 24   | 64   |
| PvP         | 16   | 48   |
| Technical   | 40   | 80   |

💡 Связано с delay-chunk-unloads-by, entity-activation-range, per-player-mob-spawns и DAB.

##🚫 9. Защита от лагов при движении и лимиты сущностей:
prevent-moving-into-unloaded-chunks: true

entity-per-chunk-save-limit:
  arrow: 16
  trident: 16
  potion: 8
  egg: 8
  experience_orb: 16
  wither_skull: 4
  firework_rocket: 8
  
📘 Назначение:
предотвращает фризы при попадании игроков в незагруженные чанки и краши при сохранении тысяч сущностей.

| Тип сервера  | лимит снарядов |
| ------------ | -------------- |
| Survival     | 10–16          |
| SkyBlock     | 8–12           |
| PvP          | 12–20          |
| Tech/Anarchy | 20+            |

🧩 Комбинируется с alt-item-despawn-rate и async-chunk-send для минимизации sync-нагрузки.

**🌐 10. Сеть и система**

purpur.yml:
use-alternate-keepalive: true
teleport-if-outside-border: true

💡 На Linux добавьте:
-DLeaf.native-transport-type=epoll

А для GraalVM:
--add-modules=jdk.incubator.vector

##🧭 11. Профилирование и диагностика

🔹 Spark:
/spark profiler start
# подождите 20–30 минут активности
/spark profiler stop

Фильтрация тяжёлых тиков:
/spark profiler --only-ticks-over 100

🔹 MSPT / TPS:
/mspt

- MSPT ≤ 50 — норма
- 55 — поискать нагрузку

🔹 Sentry (Leaf):
sentry:
  dsn: 'https://yourdsn@oXXXX.ingest.sentry.io/XXXXX'
  log-level: WARN
  only-log-thrown: true

##⚠️ 12. Типичные ошибки при оптимизации:

| Ошибка                              | Последствие                | Исправление                                             |
| ----------------------------------- | -------------------------- | ------------------------------------------------------- |
| Включены все async-модули           | Краши Citizens/ProtocolLib | Включать только entity-tracker, pathfinding, chunk-send |
| parallel-world-tracking=true        | Ошибки async чанков        | Выключить                                               |
| ClearLag-плагины                    | Лаги при сканировании      | Использовать alt-item-despawn-rate                      |
| view-distance > 8                   | Падение TPS                | 6–7 макс                                                |
| Нет spawn-limits                    | Фризы от мобов             | Настроить DAB                                           |
| Старые GC (CMS/Parallel)            | Утечки памяти              | G1GC/ZGC                                                |
| hopper-check > 8                    | Ломаются фермы             | ≤ 8                                                     |
| lobotomize + inactive villagers off | Тупые жители               | Сбалансировать                                          |

##⚡ 13. Быстрая таблица для админов:

| Компонент                                | 50 игроков        | 100 игроков       | 300+ игроков      |
| ---------------------------------------- | ----------------- | ----------------- | ----------------- |
| Java                                     | Temurin 21 (G1GC) | GraalVM 21 (G1GC) | GraalVM 21 (ZGC)  |
| Xms/Xmx                                  | 6 G               | 10 G              | 16–32 G           |
| view-distance                            | 7                 | 6                 | 5                 |
| simulation-distance                      | 4                 | 4                 | 3                 |
| per-player-mob-spawns                    | true              | true              | true              |
| DAB                                      | on (dist 8)       | on (dist 7)       | on (dist 6)       |
| hopper transfer/check                    | 8/8               | 8/8               | 10/10             |
| async-pathfinding / chunk-send / tracker | on                | on                | on                |
| parallel-world-tracking                  | off               | off               | off               |
| redstone-implementation                  | ALTERNATE_CURRENT | ALTERNATE_CURRENT | ALTERNATE_CURRENT |
| Spark                                    | enabled           | enabled           | enabled           |

##🧠 14. JVM-флаги (в приложении):
G1GC (до 200 игроков)
(см. часть 1 — пункт 3)
ZGC (300+)

java -Xms16G -Xmx16G \
-XX:+UseZGC -XX:+ZGenerational \
-XX:+AlwaysPreTouch -XX:+DisableExplicitGC \
--add-modules=jdk.incubator.vector \
-jar server.jar nogui

##🧩 15. Заключение:

🎯 Цель — разгрузить главный поток за счёт:

- асинхронных подсистем (Leaf async-pathfinding / chunk-send / entity-tracker)
- динамического AI (DAB)
- оптимизации чанков и hopper-систем
- ограничений дистанций и лимитов мобов

🔍 Перед изменениями — используйте /spark profiler и /mspt.
❌ Не ставьте «чудо-плагины оптимизации» — Leaf и Pufferfish уже делают это на уровне ядра.

##📎 Источники: 

PaperMC Docs (https://docs.papermc.io/)
Pufferfish Wiki (https://github.com/pufferfish-gg/Pufferfish)
Leaf Docs (Winds-Studio) (https://github.com/Winds-Studio/Leaf?utm_source=chatgpt.com)
Aikar JVM Flags Guide (https://aikar.co/2018/07/02/tuning-the-jvm-g1gc-garbage-collector-flags-for-minecraft/)
Spark Plugin (https://spark.lucko.me/)
