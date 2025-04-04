# neurostory_architecture
Добро пожаловать в репозиторий архитектуры проекта "Нейростори" — инновационной текстовой RPG с динамической генерацией контента на основе искусственного интеллекта.

# Краткое описание проекта (полное ниже)

## Описание проекта
"Нейростори" — это мультимедийная ролевая игра, в которой игроки создают уникальные миры, приключения и персонажей, а AI-модели генерируют текст, изображения, аудио и анимации в реальном времени. Основные цели проекта:  

- Обеспечить персонализированный игровой опыт с нелинейным повествованием.  
- Снизить затраты на разработку по сравнению с традиционными RPG.  
- Поддерживать сообщество через моддинг и мультиплеерные функции.  
- Проект ориентирован на платформу Steam с перспективой портирования на консоли, мобильные устройства и Steam Deck.  

## Архитектура системы
Архитектура "Нейростори" построена на модульном и асинхронном подходе, обеспечивающем гибкость, масштабируемость и высокую производительность. Она включает следующие ключевые компоненты:  

### Основные уровни  
Клиентская часть (Unity Client): Интерфейс игрока, реализованный на Unity.  
Серверная часть (FastAPI Server): Ядро системы, управляющее логикой и запросами.  
Очереди (Queues): Асинхронная обработка задач генерации контента.  
AI-модули (AI Modules): Генерация текста, изображений, речи и анимаций.  
Базы данных (Databases): Хранение данных пользователей, миров, приключений и NPC.  
Мультиплеер (Multiplayer): Координация совместной игры.  
Audio Search: Поиск звуков окружения из локальной базы и FreeSound API.  

## Компоненты
### Клиентская часть:
- Player: Игрок, отправляющий команды через UI.  
- User Interface (UI): Графический интерфейс для ввода и отображения.  
- Unity Controller (UC): Логика клиента, преобразующая команды в запросы.  
- Network Adapter (NA): Связь с сервером через REST API с JWT-аутентификацией.  
### Серверная часть (FastAPI):
- API Gateway: Точка входа для всех запросов.
- Authentication Module: Регистрация и вход с JWT.
- World Creation Module: Генерация миров (текст, карты, изображения).
- Adventure Creation Module: Создание приключений (шаги, NPC, аудио).
- Action Handler Module: Обработка действий игрока.
- Dialogue Manager Module: Управление диалогами с NPC.
- Character Manager Module: Создание и управление персонажами.
- Party Manager Module: Координация групп игроков и NPC.
- Queue Manager: Управление асинхронными задачами.
- Data Saver: Сохранение данных в базы.
- Bug Manager Module: Обработка багов.
- Settings Manager Module: Настройка генерации (модели, промпты).  

### Очереди:
LLM Queue, TTI Queue, TTS Queue, Audio Queue для асинхронной обработки.  

### AI-модули:
- GPT-4o: Генерация текстов.
- Midjourney v6 / DALL-E 3: Генерация изображений.
- OpenAI TTS / XTTSv2: Генерация речи.
- Azgaar: Генерация карт.
- DepthFlow: Генерация анимаций.
- Audio Search:
  - Локальный банк (2000 звуков) и FreeSound API.

### Базы данных:
Users DB, Worlds DB, Adventures DB, Characters DB, NPCs DB, Game DB, Bugs DB.  

### Мультиплеер:
Multiplayer Server и Coordinator для синхронизации.  

## Краткая визуализация архитектуры
Краткая архитектура представлена в Mermaid-диаграмме.
```mermaid
graph TD
    P[Игрок] --> UI[UI]
    UI --> UC[Контроллер]
    UC --> NA[Сеть]
    NA --> API[API Gateway]
    subgraph FastAPI[Сервер]
        API --> AUTH[Аутентификация]
        API --> WC[Создание мира]
        API --> AC[Создание приключения]
        API --> QM[Очереди]
    end
    subgraph Queues[Очереди]
        QM --> LQ[Текст]
        QM --> TQ[Изображения]
        QM --> TSQ[Речь]
    end
    subgraph AI[AI-модули]
        LQ --> LLM[GPT-4o]
        TQ --> TTI1[Midjourney]
        TSQ --> TTS1[OpenAI TTS]
    end
    API --> NA --> UC --> UI --> P
  ```

# Полное описание проекта (полное ниже)

## Полное описание архитектуры системы
### 1. Общая структура
- Архитектура делится на следующие ключевые уровни:
- Клиентская часть (Unity Client): интерфейс взаимодействия игрока с системой.
- Серверная часть (FastAPI Server): ядро системы, обрабатывающее запросы и управляющее логикой.
- Очереди (Queues): управление асинхронными задачами генерации контента.
- AI-модули (AI Modules): генерация текста, изображений, речи и анимации.
- Базы данных (Databases): хранение данных пользователей, миров, приключений, персонажей и NPC.
- Мультиплеер (Multiplayer): координация совместной игры.
- Audio Search: поиск звуков окружения.
### 2. Компоненты системы
#### 2.1. Клиентская часть (Unity Client)
- Player (Игрок): конечный пользователь, который взаимодействует с системой через интерфейс. Игрок может регистрироваться, входить в систему, создавать миры, приключения, персонажей, управлять действиями и вести диалоги с NPC.
- User Interface (UI): графический интерфейс в Unity, через который игрок отправляет команды (например, создание мира, выбор действия) и получает результаты (текст, изображения, аудио).
- Unity Controller (UC): логика клиентской части, преобразующая команды игрока в HTTP-запросы и отображающая ответы сервера через UI.
- Network Adapter (NA): модуль связи с сервером через REST API, использующий JWT-токены для аутентификации.
**Поток данных: Игрок → UI → UC → NA → Сервер.**
#### 2.2. Серверная часть (FastAPI Server)
- Сервер на FastAPI выступает шлюзом и координатором всех операций. Он включает следующие модули:
- API Gateway (API): точка входа для всех клиентских запросов. Принимает HTTP-запросы (GET/POST) и распределяет их по соответствующим модулям. Использует JWT для проверки авторизации.
- Authentication Module (AUTH): отвечает за регистрацию (register) и вход (login) пользователей. Создаёт и проверяет JWT-токены, сохраняя данные в Users DB.
- World Creation Module (WC): управляет созданием игровых миров. Генерирует описание мира, карту (через Azgaar) и изображение (Midjourney/DALL-E 3), сохраняя данные в Worlds DB.
- Adventure Creation Module (AC): создает приключения. Генерирует начальный шаг, NPC, изображения и аудио, используя контекст мира из Worlds DB и сохраняя состояние в Adventures DB и Game DB.
- Action Handler Module (AH): обрабатывает действия игрока в приключении. Генерирует следующий шаг, варианты действий, изображения и аудио, обновляя состояние в Game DB.
- Dialogue Manager Module (DM): управляет диалогами с NPC. Генерирует текст диалога, озвучку и сохраняет состояние в NPCs DB и Game DB.
- Character Manager Module (CM): отвечает за создание, обновление и генерацию изображений персонажей. Данные хранятся в Characters DB.
- Party Manager Module (PM): управляет группами игроков и NPC. Координирует состав группы и синхронизирует данные через Game DB.
- Queue Manager (QM): координирует асинхронные задачи генерации (текст, изображения, аудио), распределяя их по очередям (LLM Queue, TTI Queue, TTS Queue, Audio Queue).
- Data Saver (DS): модуль сохранения данных в соответствующие базы (Users DB, Worlds DB, Adventures DB, Characters DB, NPCs DB, Game DB, Bugs DB).
- Bug Manager Module (BM): регистрирует и предоставляет информацию о багах, сохраняя их в Bugs DB.
- Settings Manager Module (SM): управляет настройками генерации, такими как выбор модели изображений, варианты промптов и вес персонажей в Midjourney.
**Поток данных: API → Модули → DS → Базы данных / QM → Очереди.**
#### 2.3. Очереди (Queues)
Очереди обеспечивают асинхронную обработку задач генерации:
- LLM Queue (LQ): задачи генерации текста (описания, шаги, диалоги, NPC).
- TTI Queue (TQ): задачи генерации изображений (миры, локации, персонажи, NPC).
- TTS Queue (TSQ): задачи генерации речи (озвучка шагов, диалогов).
- Audio Queue (AQ): задачи поиска звуков окружения.
**Поток данных: QM → Очереди → AI-модули → QM → Модули сервера.**
#### 2.4. AI-модули (AI Modules)
Модули искусственного интеллекта генерируют контент:
- GPT-4o (LLM): генерация текстов (описания миров, шаги приключений, диалоги NPC).
- Midjourney v6 (TTI1): генерация изображений (миры, локации, персонажи, NPC) с поддержкой весов персонажей.
- DALL-E 3 (TTI2): альтернативная генерация изображений.
- OpenAI TTS (TTS1): генерация речи для озвучки шагов и диалогов.
- XTTSv2 (TTS2): альтернативная генерация речи с настройкой голоса.
- Azgaar (MG): генерация карт для миров с учётом параметров (климат, государства, расы).
- DepthFlow (AG): генерация анимаций для действий игрока.
**Поток данных: Очереди → AI-модули → QM.**
#### 2.5. Audio Search
- Local Sound Bank (LB): локальная база из 2000 звуков с тегами для окружения.
- FreeSound API (FS): внешний API для поиска звуков, если нужного нет в локальной базе.
**Поток данных: AQ → LB → FS (при необходимости) → AQ → AH/AC/DM.**
#### 2.6. Базы данных (Databases)
- Users DB (UD): данные пользователей (логины, пароли, настройки языка, баланс).
- Worlds DB (WD): информация о мирах (название, климат, правительства, карта, изображения).
- Adventures DB (AD): данные приключений (локации, квесты, шаги, NPC).
- Characters DB (CD): информация о персонажах (имя, характеристики, изображения).
- NPCs DB (ND): данные NPC (имя, описание, диалоги, отношения).
- Game DB (GS): текущее состояние игры (позиция, шаги, группа).
- Bugs DB (BD): записи о багах (ошибка, описание, пользователь).
**Поток данных: DS → Базы данных → Модули сервера.**
#### 2.7. Мультиплеер (Multiplayer)
- Multiplayer Server (MS): сервер для обработки мультиплеерных запросов.
- Multiplayer Coordinator (MC): синхронизация данных между игроками через Game DB.
**Поток данных: API → MS → MC → GS → MC → MS → API.**
### 3. Основные процессы
#### 3.1. Регистрация и вход
- Игрок отправляет запрос на регистрацию/вход через UI.
- AUTH проверяет данные в Users DB, возвращает JWT-токен через API.
#### 3.2. Создание мира
- WC принимает параметры (название, климат, правительства) через API.
- Генерирует описание (LLM), карту (MG) и изображение (TTI1/TTI2) через QM.
- Сохраняет данные в Worlds DB через DS.
#### 3.3. Создание приключения
- AC загружает контекст мира из Worlds DB.
- Генерирует первый шаг (LLM), изображения (TTI1/TTI2), NPC и аудио (TTS1/TTS2, AQ).
- Сохраняет данные в Adventures DB и Game DB.
#### 3.4. Обработка действий
- AH загружает состояние из Game DB.
- Генерирует следующий шаг (LLM), изображение (TTI1/TTI2), анимацию (AG) и аудио (TTS1/TTS2, AQ).
- Обновляет Game DB.
#### 3.5. Диалог с NPC
- DM загружает данные NPC из NPCs DB.
- Генерирует диалог (LLM) и озвучку (TTS1/TTS2).
- Сохраняет состояние в Game DB.
#### 3.6. Управление персонажами и группой
- CM создаёт/обновляет персонажей, сохраняя их в Characters DB.
- PM управляет группой, синхронизируя данные через Game DB.
#### 3.7. Обработка багов и настроек
- BM регистрирует баги в Bugs DB.
- SM обновляет настройки генерации в Users DB и влияет на очереди через QM.
#### 3.8. Мультиплеер
- MS и MC синхронизируют действия игроков через Game DB.
### 4. Преимущества архитектуры
- Модульность: каждый компонент изолирован, что упрощает разработку и масштабирование.
- Асинхронность: очереди позволяют обрабатывать тяжёлые задачи (генерацию контента) без блокировки.
- Гибкость: поддержка двух моделей изображений (Midjourney, DALL-E 3) и речи (OpenAI TTS, XTTSv2).
- Расширяемость: легко добавить новые AI-модули или функции мультиплеера.
### 5. Ограничения
- Зависимость от внешних API: FreeSound и AI-модели могут быть точками отказа.
- Очереди: многопоточность усложняет отслеживание прогресса задач.

## Полная визуализация архитектуры
Полная архитектура представлена в Mermaid-диаграмме.
```mermaid
---
config:
  layout: elk
  look: neo
---
flowchart TD
 subgraph Player["Игрок"]
        P["Player"]
  end
 subgraph UnityClient["Клиентская часть - Unity"]
        UI["User Interface"]
        UC["Unity Controller"]
        NA["Network Adapter"]
  end
 subgraph FastAPIServer["Серверная часть - FastAPI"]
        API["API Gateway"]
        AUTH["Authentication Module"]
        WC["World Creation Module"]
        AC["Adventure Creation Module"]
        AH["Action Handler Module"]
        DM["Dialogue Manager Module"]
        CM["Character Manager Module"]
        PM["Party Manager Module"]
        QM["Queue Manager"]
        DS["Data Saver"]
        BM["Bug Manager Module"]
        SM["Settings Manager Module"]
  end
 subgraph Queues["Очереди"]
        LQ["LLM Queue"]
        TQ["TTI Queue"]
        TSQ["TTS Queue"]
        AQ["Audio Queue"]
  end
 subgraph AIModules["AI-модули"]
        LLM["GPT-4o - Text Generation"]
        TTI1["Midjourney v6 - Image Generation"]
        TTI2["DALL-E 3 - Image Generation"]
        TTS1["OpenAI TTS - Speech Generation"]
        TTS2["XTTSv2 - Speech Generation"]
        MG["Azgaar - Map Generation"]
        AG["DepthFlow - Animation Generation"]
  end
 subgraph AudioSearch["Audio Search"]
        LB["Local Sound Bank - 2000 Tagged Sounds"]
        FS["FreeSound API"]
  end
 subgraph Databases["Базы данных"]
        UD["Users DB"]
        WD["Worlds DB"]
        AD["Adventures DB"]
        CD["Characters DB"]
        ND["NPCs DB"]
        GS["Game DB"]
        BD["Bugs DB"]
  end
 subgraph Multiplayer["Мультиплеер"]
        MS["Multiplayer Server"]
        MC["Multiplayer Coordinator"]
  end
    P -- Регистрация / Вход / Действие --> UI
    UI -- Команды игрока --> UC
    UC -- "HTTP-запросы" --> NA
    NA -- REST API + JWT --> API
    API -- Регистрация / Логин --> AUTH
    AUTH -- Создание / Проверка JWT --> UD
    UD -- Токен --> AUTH
    AUTH -- Успешный логин --> API
    API -- Создание мира --> WC
    API -- Создание приключения --> AC
    API -- Обработка действия --> AH
    API -- Управление диалогом --> DM
    API -- Управление персонажами --> CM
    API -- Управление группой --> PM
    API -- Сообщить о баге --> BM
    API -- Настройки генерации --> SM
    WC -- Параметры мира --> DS
    DS -- Название, климат, правительства --> WD
    WC -- Описание мира --> QM
    QM -- Очередь текста --> LQ
    LQ -- Генерация текста --> LLM
    LLM -- Текст мира --> QM
    QM -- Описание --> WC
    WC -- Запрос карты --> MG
    MG -- Карта мира --> WC
    WC -- Запрос изображения --> TQ
    TQ -- Генерация изображения --> TTI1 & TTI2
    TTI1 -- Изображение мира --> QM
    TTI2 -- Изображение мира --> QM
    QM -- Изображение --> WC & AC & AH & CM
    WC -- Сохранение мира --> DS
    DS -- Данные мира --> WD
    WC -- Ответ клиенту --> API
    AC -- Контекст мира --> WD
    WD -- Данные мира --> AC
    AC -- Параметры приключения --> DS
    DS -- Локация, квест, NPC --> AD
    AC -- Первый шаг --> QM
    LLM -- Текст шага --> QM
    QM -- Шаг --> AC & AH
    AC -- Изображение локации --> TQ
    TTI1 -- Изображение локации --> QM
    TTI2 -- Изображение локации --> QM
    AC -- Создание NPC --> QM
    LQ -- Генерация NPC --> LLM
    LLM -- Данные NPC --> QM
    QM -- NPC --> AC
    AC -- Изображение NPC --> TQ
    TTI1 -- Изображение NPC --> QM
    TTI2 -- Изображение NPC --> QM
    QM -- Изображение NPC --> AC
    AC -- Озвучка шага --> TSQ
    TTS1 -- Аудио шага --> QM
    TTS2 -- Аудио шага --> QM
    QM -- Аудио --> AC & AH & DM
    AC -- Звук окружения --> AQ
    AQ -- Поиск звука --> LB
    LB -- Звук найден --> AQ
    LB -- Звук не найден --> FS
    FS -- Звук из API --> AQ
    AQ -- Аудио окружения --> AC & AH
    AC -- Сохранение приключения --> DS
    DS -- Шаг, NPC --> AD
    DS -- Состояние --> GS
    AC -- Ответ клиенту --> API
    AH -- Контекст игры --> GS
    GS -- Состояние, персонажи --> AH
    AH -- Следующий шаг --> QM
    LQ -- Генерация шага --> LLM
    LLM -- Текст и варианты --> QM
    AH -- Изображение сцены --> TQ
    TTI1 -- Изображение сцены --> QM
    TTI2 -- Изображение сцены --> QM
    AH -- Анимация действия --> AG
    AG -- Анимация сцены --> AH
    AH -- Озвучка шага --> TSQ
    AH -- Звук окружения --> AQ
    AH -- Сохранение шага --> DS
    DS -- Обновленное состояние --> GS
    AH -- Ответ клиенту --> API
    DM -- Контекст NPC --> ND
    ND -- Данные NPC --> DM
    DM -- Запрос диалога --> QM
    LQ -- Генерация диалога --> LLM
    LLM -- Текст диалога --> QM
    QM -- Диалог --> DM
    DM -- Озвучка диалога --> TSQ
    TSQ -- Генерация речи --> TTS1 & TTS2
    TTS1 -- Аудио диалога --> QM
    TTS2 -- Аудио диалога --> QM
    DM -- Сохранение состояния --> DS
    DM -- Ответ клиенту --> API
    CM -- Создание / Обновление --> DS
    DS -- Данные персонажа --> CD
    CM -- Изображение персонажа --> TQ
    TTI1 -- Изображение персонажа --> QM
    TTI2 -- Изображение персонажа --> QM
    CM -- Ответ клиенту --> API
    PM -- Создание / Обновление --> DS
    DS -- Данные группы --> GS
    PM -- Добавление NPC --> ND
    ND -- NPC в группу --> PM
    PM -- Добавление персонажей --> CD
    CD -- Персонажи в группе --> PM
    PM -- Ответ клиенту --> API
    BM -- Создание бага --> DS
    DS -- Данные бага --> BD
    BM -- Получение багов --> BD
    BD -- Список багов --> BM
    BM -- Ответ клиенту --> API
    SM -- Установка модели --> DS
    DS -- Настройки --> UD
    SM -- Варианты промптов --> QM
    QM -- Обновление очередей --> LQ & TQ
    SM -- Ответ клиенту --> API
    API -- Мультиплеерный запрос --> MS
    MS -- Координация --> MC
    MC -- Синхронизация --> GS
    GS -- Обновленное состояние --> MC
    MC -- Ответ серверу --> MS
    MS -- Ответ клиенту --> API
    API -- Текст, изображение, аудио --> NA
    NA -- Данные --> UC
    UC -- Отображение --> UI
    UI -- Игровой экран --> P
  ```


