# Система "Архитектура Осмысленной Жизни 3.0": Полная архитектура

Этот документ описывает полную архитектуру системы АОЖ 3.0 на трех уровнях детализации, соответствующих методологии IDEF0.

---

## Уровень 1: Контекстная диаграмма (A-0)

Эта диаграмма представляет всю систему как единый "черный ящик", показывая ее основные входы, выходы, элементы управления и механизмы. Это взгляд на систему с высоты птичьего полета.

```mermaid
graph TD
    subgraph IDEF0_Context_Diagram [A-0: Обработка сессии пользователя]
        %% Controls (Управление - сверху)
        C1["Методология АОЖ 3.0"]
        C2["Системный промпт<br/>для AI-коуча"]
        
        %% Main Process (Основной процесс - в центре)
        P["<div style='font-size:1.2em; font-weight:bold; padding: 10px;'>Обработка сессии<br/>пользователя</div>"]

        %% Inputs (Входы - слева)
        I1["Текстовые и голосовые<br/>сообщения пользователя"]
        I2["Данные о пользователе<br/>(ID, имя, прогресс)"]

        %% Outputs (Выходы - справа)
        O1["Ответ пользователю<br/>в Telegram"]
        O2["Обновленный план<br/>и рефлексия в БД"]
        O3["Запись диалога<br/>в лог и память"]

        %% Mechanisms (Механизмы - снизу)
        M1["<b>AI-коуч:</b><br/>Mistral Cloud"]
        M2["<b>Платформа автоматизации:</b><br/>n8n"]
        M3["<b>База данных / Память:</b><br/>Supabase (Postgres)"]
        M4["<b>Транскрибация голоса:</b><br/>Nexara"]

        %% Connections
        C1 -- Управление --> P
        C2 -- Управление --> P
        
        I1 -- Вход --> P
        I2 -- Вход --> P
        
        P -- Выход --> O1
        P -- Выход --> O2
        P -- Выход --> O3
        
        P -- "Исполняется с помощью" --> M1
        P -- "Исполняется с помощью" --> M2
        P -- "Исполняется с помощью" --> M3
        P -- "Исполняется с помощью" --> M4
    end
```

---

## Уровень 2: Декомпозиция по этапам АОЖ 3.0 (A0)

Эта диаграмма "раскрывает" центральный блок из предыдущей схемы и показывает, из каких логических этапов состоит сессия с пользователем, согласно вашему документу `AOJ3.md`.

```mermaid
graph TD
    subgraph A0_Decomposition [A0: Этапы сессии "Пре-сессия и Знакомство"]
        %% Process Blocks from AOJ3.md
        A1["<b>A1: Приветствие<br/>и Настройка</b>"]
        A2["<b>A2: Диагностика</b><br/>(вопросы о настроении,<br/>энергии, вызовах и т.д.)"]
        A3["<b>A3: Синтез<br/>и Завершение</b>"]

        %% Connections
        A1 -- "Начало диалога" --> A2
        A2 -- "Ответы пользователя" --> A3

        %% Inputs
        I1["Сообщения и данные<br/>пользователя"]
        I1 -- Вход --> A1 & A2
        
        %% Outputs
        O1["Эмпатичные ответы<br/>и следующие вопросы"]
        O2["Итоговый синтез в БД<br/>и ответ пользователю"]
        
        A2 -- Выход --> O1
        A3 -- Выход --> O2

        %% Controls
        C1["Методология из AOJ3.md"]
        C1 -- Управляет --> A1 & A2 & A3
        
        %% Mechanisms
        M_All["<div style='text-align:center'><b>Общие механизмы:</b><br/>AI-коуч, n8n, Supabase, Nexara</div>"]
        M_All -- "Используются" --> A1 & A2 & A3
    end
```

---

## Уровень 3: Детализированная пошаговая схема

Эта диаграмма показывает фактический технический процесс обработки каждого сообщения пользователя, от получения до отправки ответа.

```mermaid
flowchart TD
    %% Стилевое определение для компонентов IDEF0
    classDef process fill:#f9f,stroke:#333,stroke-width:2px;
    classDef data fill:#ccf,stroke:#333,stroke-width:1px,max-width:200px;
    classDef mechanism fill:#e8e8e8,stroke:#333,stroke-width:1px;
    classDef control fill:#fcc,stroke:#333,stroke-width:1px;

    %% Входные данные
    UserInput["<b>Вход:</b> Сообщение от пользователя<br/>(текст или голос)"]:::data

    subgraph A1 [A1: Маршрутизация и подготовка]
        direction LR
        A1_Process["Проверить тип сообщения<br/>(текст/голос)"]:::process
    end
    
    UserInput --> A1_Process
    M_A1["<b>Механизм:</b><br/>n8n Switch Node"]:::mechanism
    M_A1 --> A1_Process

    subgraph A2 [A2: Транскрибация голоса]
        direction LR
        A2_Process["Преобразовать аудио в текст"]:::process
    end
    
    A1_Process -- "Если сообщение голосовое" --> A2_Process
    M_A2["<b>Механизм:</b><br/>Сервис Nexara"]:::mechanism
    M_A2 --> A2_Process
    
    TranscribedText["<b>Данные:</b><br/>Распознанный текст"]:::data
    A2_Process --> TranscribedText

    subgraph A3 [A3: Интеллектуальная обработка]
        direction LR
        A3_Process["Сгенерировать ответ<br/>на основе контекста"]:::process
    end
    
    OriginalText["<b>Данные:</b><br/>Оригинальный текст"]:::data
    A1_Process -- "Если сообщение текстовое" --> OriginalText --> A3_Process
    TranscribedText --> A3_Process

    C_A3["<b>Управление:</b><br/>Системный промпт АОЖ 3.0"]:::control
    M_A3_1["<b>Механизм:</b><br/>Языковая модель Mistral Large"]:::mechanism
    M_A3_2["<b>Механизм:</b><br/>Postgres Chat Memory"]:::mechanism
    C_A3 --> A3_Process
    M_A3_1 --> A3_Process
    M_A3_2 --> A3_Process

    subgraph A4 [A4: Постобработка и сохранение]
        direction LR
        A4_Process["Отправить ответ<br/>и сохранить результаты"]:::process
    end

    AI_Response["<b>Данные:</b><br/>Сгенерированный ответ AI"]:::data
    A3_Process --> AI_Response --> A4_Process

    M_A4_1["<b>Механизм:</b><br/>База данных Supabase"]:::mechanism
    M_A4_2["<b>Механизм:</b><br/>Telegram Bot API"]:::mechanism
    M_A4_1 --> A4_Process
    M_A4_2 --> A4_Process
    
    FinalOutput["<b>Выход:</b> Сообщение в чате<br/>и запись в БД"]:::data
    A4_Process --> FinalOutput
``` 