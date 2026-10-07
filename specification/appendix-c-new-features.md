[← Оглавление перевода](README.md) · [← Приложение B](appendix-b-mandatory-statements.md) · [Приложение D →](appendix-d-implementation-guidelines.md)

# Приложение C. Сводка новых функций в MQTT v5.0

Это приложение перечисляет ключевые изменения, добавленные в MQTT 5.0 по сравнению с 3.1.1. Материал не вводит новых требований, а служит путеводителем по нововведениям.

## C.1. Свойства (Properties)

Главное структурное новшество. В переменный заголовок почти всех пакетов добавлена секция **Properties** — набор пар «идентификатор + значение».

Ключевые свойства:

- **Session Expiry Interval** — время жизни сессии после отключения.
- **Receive Maximum** — управление потоком QoS 1/2.
- **Maximum Packet Size** — ограничение размера пакетов.
- **Topic Alias Maximum / Topic Alias** — сжатие имён топиков.
- **Response Topic / Correlation Data** — шаблон Request/Response.
- **User Property** — пользовательские метаданные.
- **Authentication Method / Data** — расширенная аутентификация.
- **Reason String** — текстовое описание причины.
- **Payload Format Indicator / Content Type** — типизация payload.
- **Message Expiry Interval** — время жизни сообщения.

## C.2. Коды причин (Reason Codes)

Вместо пары «Return Code 0/1/2» в CONNACK и простого «PUBACK без деталей» введены числовые **Reason Codes** для большинства пакетов:

- CONNACK, PUBACK, PUBREC, PUBREL, PUBCOMP, SUBACK, UNSUBACK, DISCONNECT, AUTH.
- Значения 0x00–0x7F — успех; 0x80–0xFF — ошибка.
- Позволяют точно определить причину отказа без закрытия соединения.

## C.3. Пакет AUTH и расширенная аутентификация

Появился пакет **AUTH** для произвольных схем аутентификации:

- SASL-подобные схемы.
- Challenge-response.
- Переаутентификация без переподключения.
- Обмен данными через Authentication Data.

## C.4. Управление сессией

Гибкое управление временем жизни сессии:

- **Session Expiry Interval** вместо Clean Session.
- Возможность изменить срок при DISCONNECT.
- Clean Start — управление началом сессии при CONNECT.

## C.5. Тематические псевдонимы (Topic Alias)

Позволяют заменить полное имя топика коротким числовым псевдонимом:

- Экономия трафика при частых публикациях.
- Область действия — одно сетевое соединение.
- Согласование через Topic Alias Maximum.

## C.6. Общие подписки (Shared Subscriptions)

Формализованы подписки с префиксом `$share/{ShareName}/{Filter}`:

- Несколько клиентов делят поток сообщений.
- Каждое сообщение — только одному клиенту группы.
- Балансировка нагрузки между потребителями.

## C.7. Request/Response

Формальный шаблон для «запрос-ответ» через pub/sub:

- Свойство **Response Topic**.
- Свойство **Correlation Data**.
- Свойство **Response Information** в CONNACK.

## C.8. Управление потоком и ресурсами

- **Receive Maximum** — ограничение неподтверждённых QoS 1/2.
- **Maximum Packet Size** — лимит размера пакета.
- **Message Expiry Interval** — устаревание сообщений.
- **Server Keep Alive** — сервер может переопределить Keep Alive.

## C.9. Улучшения QoS

- Reason Codes в PUBACK/PUBREC/PUBREL/PUBCOMP.
- Возможность прекратить QoS 2 на этапе PUBREC.
- Более точная обработка ошибок.

## C.10. Улучшения Will

- **Will Delay Interval** — задержка публикации Will.
- Собственные Properties у Will-сообщения (Message Expiry, Content Type, Response Topic и др.).
- DISCONNECT с кодом 0x04 — «Disconnect with Will Message».

## C.11. Улучшения SUBSCRIBE

- **Subscription Options** вместо единственного байта QoS:
  - **QoS** — максимальный уровень.
  - **No Local (NL)** — не получать свои публикации.
  - **Retain As Published (RAP)** — сохранять флаг RETAIN.
  - **Retain Handling** — управление отправкой retained при подписке.
- **Subscription Identifier** — числовая метка подписки.

## C.12. Перенаправление сервера

- **Server Reference** в CONNACK и DISCONNECT.
- Reason Codes 0x9C (Use another server), 0x9D (Server moved).
- Клиент может автоматически переподключаться к указанному узлу.

## C.13. Улучшения при ошибках

- Точные Reason Codes вместо закрытия соединения.
- **Reason String** для отладки.
- **Request Problem Information** — управление объёмом диагностики.
- Меньше ситуаций, требующих разрыва соединения.

## C.14. Совместимость с 3.1.1

MQTT 5.0 несовместим на уровне протокола с 3.1.1, но концептуально совместим:

- Одинаковая базовая модель (pub/sub через брокер).
- Одинаковые принципы QoS, retain, Will.
- Различия — в структуре пакетов, кодах и опциях.
- Брокер обычно поддерживает обе версии одновременно.

---

[← Оглавление перевода](README.md) · [← Приложение B](appendix-b-mandatory-statements.md) · [Приложение D →](appendix-d-implementation-guidelines.md)