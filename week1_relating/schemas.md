# Схеми до week 1 (Relating)

Візуальний додаток до `joins-and-subqueries.sql`.
Діаграми у форматі Mermaid, GitHub рендерить їх автоматично.

---

## 1. Карта містків dese.db

Стрілка це місток. Тільки через містки таблиці можна склеїти.

```mermaid
erDiagram
    districts ||--o{ schools : "id = district_id"
    districts ||--o{ expenditures : "id = district_id"
    districts ||--o{ staff_evaluations : "id = district_id"
    schools ||--o{ graduation_rates : "id = school_id"

    districts {
        int id PK
        text name
        text type
        text city
        text state
        text zip
    }
    schools {
        int id PK
        int district_id FK
        text name
        text type
        text city
        text state
        text zip
    }
    expenditures {
        int id PK
        int district_id FK
        int pupils
        real per_pupil_expenditure
    }
    staff_evaluations {
        int id PK
        int district_id FK
        real evaluated
        real exemplary
        real proficient
        real needs_improvement
        real unsatisfactory
    }
    graduation_rates {
        int id PK
        int school_id FK
        real graduated
        real dropped
        real excluded
    }
```

Читати так: `districts` угорі, під нею три таблиці. Під `schools` ще одна.

---

## 2. Два випадки для ON

```mermaid
flowchart TD
    A["Чи є пряма стрілка<br/>між двома таблицями?"] -->|Так| B["ВИПАДОК А<br/>id однієї = місток другої<br/><br/>districts.id = expenditures.district_id"]
    A -->|Ні| C["Чи висять обидві<br/>під однією таблицею?"]
    C -->|Так| D["ВИПАДОК Б<br/>однаковий місток з обох боків<br/><br/>schools.district_id = expenditures.district_id"]
    C -->|Ні| E["Потрібна проміжна таблиця,<br/>ланцюжок з двох JOIN"]
```

Випадок Б був у 11.sql. Прямої стрілки між `schools` і `expenditures`
немає, але обидві висять під `districts`. Сама `districts` при цьому
в запиті не потрібна.

---

## 3. Метод трьох кроків

```mermaid
flowchart LR
    S["Умова завдання"] --> K1["КРОК 1<br/>Виписати два списки:<br/>що показати<br/>за чим фільтрувати"]
    K1 --> K2["КРОК 2<br/>Біля кожного слова<br/>назва таблиці"]
    K2 --> K3["КРОК 3<br/>FROM = таблиця першого<br/>стовпця у SELECT<br/>решта через JOIN"]
    K2 --> N["Кількість JOIN =<br/>кількість таблиць - 1"]
    K3 --> W{"Чи є в умові<br/>обмежувальне слово?"}
    W -->|Так| WY["WHERE потрібен"]
    W -->|Ні| WN["WHERE немає"]
```

Обмежувальні слова: державні, чартерні, з назвою X, які повідомили 100%,
3 і менше, більше за середнє.

**ORDER BY і LIMIT не є фільтром.** "10 найдорожчих" це `ORDER BY` плюс
`LIMIT`, а не `WHERE`.

---

## 4. Ланцюжок з трьох таблиць

Приклад: назва школи, назва округу, відсоток тих, хто кинув навчання.

```mermaid
flowchart LR
    D["districts<br/><b>name</b>"] -.->|"id = district_id"| S
    S["schools<br/><b>name</b><br/>FROM стартує тут"] -.->|"id = school_id"| G
    G["graduation_rates<br/><b>dropped</b>"]
```

`schools` стоїть посередині ланцюжка, обидві стрілки виходять з неї,
тому саме вона йде у `FROM`, а решта чіпляється по боках.

---

## 5. Що звідки береться у 12.sql

Найважчий запит pset: три таблиці плюс два підзапити.

```mermaid
flowchart TD
    subgraph SEL["SELECT"]
        A1["districts.name"]
        A2["expenditures.per_pupil_expenditure"]
        A3["staff_evaluations.exemplary"]
    end
    subgraph WH["WHERE"]
        B1["districts.type = 'Public School District'"]
        B2["per_pupil_expenditure > AVG підзапит"]
        B3["exemplary > AVG підзапит"]
    end
    subgraph ORD["ORDER BY"]
        C1["exemplary DESC"]
        C2["per_pupil_expenditure DESC"]
    end
    SEL --> WH --> ORD
```

Пастка: середнє в підзапиті рахується по **всій** таблиці,
без фільтра по типу округу. Тобто всередині підзапиту немає `WHERE`.
Якщо додати туди фільтр, запит виконається без помилки,
але поверне іншу кількість рядків.
