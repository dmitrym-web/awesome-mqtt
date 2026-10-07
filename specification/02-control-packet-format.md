[← Оглавление перевода](README.md) · [← Раздел 1](01-introduction.md) · [Раздел 3 →](03-control-packets.md)

# 2. Формат управляющих пакетов MQTT

## 2.1. Структура управляющего пакета MQTT

Управляющий пакет MQTT состоит из трёх частей, следующих в строгом порядке:

| Часть | Наличие | Размер |
|-------|---------|--------|
| Фиксированный заголовок (Fixed Header) | во всех пакетах | 2–5 байт |
| Переменный заголовок (Variable Header) | в части пакетов | переменный |
| Полезная нагрузка (Payload) | в части пакетов | переменный |

### 2.1.1. Фиксированный заголовок (Fixed Header)

Присутствует во всех управляющих пакетах MQTT и имеет следующий формат:

```
бит      7   6   5   4      3   2   1   0
         ┌───────┬───────┐ ┌───────────────┐
байт 1   │  Тип  │ Флаги │ │               │
         └───────┴───────┘ └───────────────┘
байт 2.. │      Remaining Length            │
         └──────────────────────────────────┘
```

Размер фиксированного заголовка — от 2 до 5 байт.

#### 2.1.1.1. Тип управляющего пакета (Control Packet type)

Старшие 4 бита первого байта задают тип пакета. Возможные значения:

| Значение | Тип пакета | Направление |
|---------:|------------|-------------|
| 1 | CONNECT | Клиент → Сервер |
| 2 | CONNACK | Сервер → Клиент |
| 3 | PUBLISH | Клиент ↔ Сервер |
| 4 | PUBACK | Клиент ↔ Сервер |
| 5 | PUBREC | Клиент ↔ Сервер |
| 6 | PUBREL | Клиент ↔ Сервер |
| 7 | PUBCOMP | Клиент ↔ Сервер |
| 8 | SUBSCRIBE | Клиент → Сервер |
| 9 | SUBACK | Сервер → Клиент |
| 10 | UNSUBSCRIBE | Клиент → Сервер |
| 11 | UNSUBACK | Сервер → Клиент |
| 12 | PINGREQ | Клиент → Сервер |
| 13 | PINGRESP | Сервер → Клиент |
| 14 | DISCONNECT | Клиент → Сервер |
| 15 | AUTH | Клиент ↔ Сервер |
| 0 | Зарезервировано | — |

Значение 0 зарезервировано и **НЕ ДОЛЖНО** использоваться.

#### 2.1.1.2. Флаги (Flags)

Младшие 4 бита первого байта — это флаги, специфичные для каждого типа пакета:

| Тип пакета | Биты 3 2 1 0 | Описание |
|-----------|--------------|----------|
| CONNECT | 0 0 0 0 | Зарезервировано |
| CONNACK | 0 0 0 0 | Зарезервировано |
| PUBLISH | DUP QoS QoS RETAIN | См. раздел 3.3 |
| PUBACK | 0 0 0 0 | Зарезервировано |
| PUBREC | 0 0 0 0 | Зарезервировано |
| PUBREL | 0 0 1 0 | Зарезервировано |
| PUBCOMP | 0 0 0 0 | Зарезервировано |
| SUBSCRIBE | 0 0 1 0 | Зарезервировано |
| SUBACK | 0 0 0 0 | Зарезервировано |
| UNSUBSCRIBE | 0 0 1 0 | Зарезервировано |
| UNSUBACK | 0 0 0 0 | Зарезервировано |
| PINGREQ | 0 0 0 0 | Зарезервировано |
| PINGRESP | 0 0 0 0 | Зарезервировано |
| DISCONNECT | 0 0 0 0 | Зарезервировано |
| AUTH | 0 0 0 0 | Зарезервировано |

Если получены недопустимые флаги, получатель **ДОЛЖЕН** закрыть сетевое соединение. Это относится и к зарезервированным битам, установленным в значение, отличное от указанного.

#### 2.1.1.3. Remaining Length (оставшаяся длина)

Remaining Length — это длина переменного заголовка плюс длина полезной нагрузки. Поле начинается со второго байта фиксированного заголовка и использует **кодирование переменной длины** (Variable Byte Integer).

**Алгоритм кодирования:**

- Каждый байт кодирует 7 бит данных и 1 бит продолжения (continuation bit, старший бит).
- Если старший бит установлен в 1 — следует ещё один байт.
- Если старший бит равен 0 — это последний байт поля.
- Максимум — 4 байта, что даёт значения до 268 435 455 (256 МБ).

**Примеры кодирования:**

| Значение | Кодировка |
|----------|-----------|
| 0 | `0x00` |
| 127 | `0x7F` |
| 128 | `0x80 0x01` |
| 16 383 | `0xFF 0x7F` |
| 16 384 | `0x80 0x80 0x01` |
| 2 097 151 | `0xFF 0xFF 0x7F` |
| 2 097 152 | `0x80 0x80 0x80 0x01` |
| 268 435 455 | `0xFF 0xFF 0xFF 0x7F` |

Псевдокод декодирования:

```
multiplier = 1
value = 0
do
    encodedByte = next byte
    value = value + (encodedByte AND 127) * multiplier
    if multiplier > 128*128*128
        error: malformed Remaining Length
    multiplier = multiplier * 128
while (encodedByte AND 128) != 0
```

Получатель **ДОЛЖЕН** закрыть соединение, если поле Remaining Length закодировано более чем 4 байтами или если встречен байт продолжения после четвёртого.

### 2.1.2. Переменный заголовок (Variable Header)

Присутствует не во всех пакетах. Содержимое зависит от типа пакета. Порядок полей строго фиксирован. Все поля переменного заголовка, кроме Remaining Length, описываются отдельно для каждого типа пакета в главе 3.

В MQTT 5.0 в переменный заголовок практически всех пакетов добавлено поле **Properties** (см. 2.2).

### 2.1.3. Полезная нагрузка (Payload)

Присутствует не во всех пакетах. Если присутствует — идёт сразу после переменного заголовка. Содержимое зависит от типа пакета:

| Тип пакета | Полезная нагрузка |
|-----------|-------------------|
| CONNECT | Client Identifier, Will Topic, Will Payload, User Name, Password |
| CONNACK | нет |
| PUBLISH | Application Message |
| PUBACK | нет |
| PUBREC | нет |
| PUBREL | нет |
| PUBCOMP | нет |
| SUBSCRIBE | Topic Filters и их параметры подписки |
| SUBACK | Reason Codes |
| UNSUBSCRIBE | Topic Filters |
| UNSUBACK | Reason Codes |
| PINGREQ | нет |
| PINGRESP | нет |
| DISCONNECT | нет (кроме случая с Reason Code в свойстве) |
| AUTH | Authentication Data |

---

## 2.2. Свойства (Properties)

В MQTT 5.0 в переменный заголовок почти каждого пакета добавлена секция **Properties**. Она состоит из:

1. **Property Length** — Variable Byte Integer, длина всех свойств в байтах.
2. **Набора свойств** — список пар «идентификатор свойства + значение».

### 2.2.1. Property Length

Поле Property Length — это Variable Byte Integer (см. 2.1.1.3), задающее длину всех следующих за ним свойств. Если свойств нет, значение = 0.

### 2.2.2. Идентификаторы свойств

Каждое свойство имеет числовой идентификатор (Variable Byte Integer) и связанное значение. Полный список свойств MQTT 5.0:

| Идентификатор | Имя свойства | Тип значения | Пакеты |
|--------------:|--------------|--------------|--------|
| 0x01 | Payload Format Indicator | Byte | PUBLISH, Will |
| 0x02 | Message Expiry Interval | Four Byte Integer | PUBLISH, Will |
| 0x03 | Content Type | UTF-8 String | PUBLISH, Will |
| 0x08 | Response Topic | UTF-8 String | PUBLISH, Will |
| 0x09 | Correlation Data | Binary Data | PUBLISH, Will |
| 0x0B | Subscription Identifier | Variable Byte Integer | PUBLISH, SUBSCRIBE |
| 0x11 | Session Expiry Interval | Four Byte Integer | CONNECT, CONNACK, DISCONNECT |
| 0x12 | Assigned Client Identifier | UTF-8 String | CONNACK |
| 0x13 | Server Keep Alive | Two Byte Integer | CONNACK |
| 0x15 | Authentication Method | UTF-8 String | CONNECT, CONNACK, AUTH |
| 0x16 | Authentication Data | Binary Data | CONNECT, CONNACK, AUTH |
| 0x17 | Request Problem Information | Byte | CONNECT |
| 0x18 | Will Delay Interval | Four Byte Integer | Will |
| 0x19 | Request Response Information | Byte | CONNECT |
| 0x1A | Response Information | UTF-8 String | CONNACK |
| 0x1C | Server Reference | UTF-8 String | CONNACK, DISCONNECT |
| 0x1F | Reason String | UTF-8 String | CONNACK, PUBACK, PUBREC, PUBREL, PUBCOMP, SUBACK, UNSUBACK, DISCONNECT, AUTH |
| 0x21 | Receive Maximum | Two Byte Integer | CONNECT, CONNACK |
| 0x22 | Topic Alias Maximum | Two Byte Integer | CONNECT, CONNACK |
| 0x23 | Topic Alias | Two Byte Integer | PUBLISH |
| 0x24 | Maximum QoS | Byte | CONNACK |
| 0x25 | Retain Available | Byte | CONNACK |
| 0x26 | User Property | UTF-8 String Pair | все пакеты |
| 0x27 | Maximum Packet Size | Four Byte Integer | CONNECT, CONNACK |
| 0x28 | Wildcard Subscription Available | Byte | CONNACK |
| 0x29 | Subscription Identifier Available | Byte | CONNACK |
| 0x2A | Shared Subscription Available | Byte | CONNACK |

Свойства **НЕ ДОЛЖНЫ** встречаться в пакете более одного раза, за исключением **User Property** и **Subscription Identifier** (для них допускается многократное появление).

Получатель **ДОЛЖЕН** закрыть соединение с ошибкой, если встретит неизвестный идентификатор свойства.

### 2.2.3. Список свойств (Property list)

Свойства следуют одно за другим без разделителей. Каждое свойство = идентификатор (Variable Byte Integer) + значение (тип зависит от идентификатора).

Правила:

- Свойства **ДОЛЖНЫ** появляться в любом порядке (порядок не регламентирован).
- Общая длина всех свойств **ДОЛЖНА** совпадать со значением Property Length.
- Если Property Length не совпадает с фактической суммой длин свойств — получатель **ДОЛЖЕН** закрыть соединение с ошибкой.

---

[← Оглавление перевода](README.md) · [← Раздел 1](01-introduction.md) · [Раздел 3 →](03-control-packets.md)