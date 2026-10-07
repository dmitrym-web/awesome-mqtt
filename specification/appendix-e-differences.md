[← Оглавление перевода](README.md) · [← Приложение D](appendix-d-implementation-guidelines.md)

# Приложение E. Различия между MQTT 3.1.1 и MQTT 5.0

Это приложение перечисляет изменения между версиями 3.1.1 (2014) и 5.0 (2019). Материал — обзорный, не нормативный.

## E.1. Формат пакетов

### E.1.1. Свойства (Properties)

- **3.1.1:** свойства отсутствуют.
- **5.0:** в переменный заголовок почти всех пакетов добавлена секция Properties (Property Length + список).

### E.1.2. Remaining Length

- **3.1.1:** максимум 268 435 455 байт.
- **5.0:** без изменений.

### E.1.3. Флаги пакетов

- **3.1.1:** для CONNECT, PUBLISH и др. действуют те же правила.
- **5.0:** добавлены требования при получении недопустимых флагов — закрытие соединения.

## E.2. CONNECT

- **3.1.1:** поля Protocol Name, Protocol Level, Connect Flags, Keep Alive. Payload: Client ID, Will Topic, Will Message, User Name, Password.
- **5.0:** добавлена секция Properties.
  - Session Expiry Interval вместо Clean Session.
  - Receive Maximum, Maximum Packet Size, Topic Alias Maximum.
  - Request Response Information, Request Problem Information.
  - Authentication Method/Data.
- **3.1.1:** флаг Clean Session.
- **5.0:** флаг Clean Start + свойство Session Expiry Interval.

## E.3. CONNACK

- **3.1.1:** Return Code (0–5), Session Present Flag.
- **5.0:** Reason Code (0x00–0xFF), Session Present Flag, Properties:
  - Assigned Client Identifier, Server Keep Alive.
  - Maximum QoS, Retain Available, Wildcard Subscription Available.
  - Subscription Identifiers Available, Shared Subscription Available.
  - Response Information, Server Reference, Reason String, User Property.

## E.4. PUBLISH

- **3.1.1:** Topic Name, Packet Identifier (QoS > 0), Payload. Флаги DUP, QoS, RETAIN.
- **5.0:** добавлены Properties:
  - Payload Format Indicator, Message Expiry Interval, Content Type.
  - Response Topic, Correlation Data, Topic Alias.
  - Subscription Identifier, User Property.

## E.5. Подтверждения

### E.5.1. PUBACK / PUBREC

- **3.1.1:** только Packet Identifier.
- **5.0:** Reason Code (опционально) + Properties.

### E.5.2. PUBREL / PUBCOMP

- **3.1.1:** только Packet Identifier.
- **5.0:** Reason Code + Properties.
- **3.1.1:** в PUBREL могут быть любые флаги? Нет — тоже `0010`.
- **5.0:** то же самое.

## E.6. SUBSCRIBE

- **3.1.1:** Packet Identifier + список пар (Topic Filter, QoS Byte).
- **5.0:** Packet Identifier + Properties (Subscription Identifier, User Property) + список пар (Topic Filter, Subscription Options).
- **5.0:** Subscription Options включает QoS, NL, RAP, Retain Handling.

## E.7. SUBACK

- **3.1.1:** Packet Identifier + список Return Codes (0x00, 0x01, 0x02, 0x80).
- **5.0:** Packet Identifier + Properties + список Reason Codes (более детальные).

## E.8. UNSUBSCRIBE

- **3.1.1:** Packet Identifier + список Topic Filters.
- **5.0:** Packet Identifier + Properties + список Topic Filters.

## E.9. UNSUBACK

- **3.1.1:** только Packet Identifier.
- **5.0:** Packet Identifier + Properties + список Reason Codes.

## E.10. DISCONNECT

- **3.1.1:** без содержимого, только для нормального отключения.
- **5.0:** Reason Code + Properties:
  - Session Expiry Interval (изменение при отключении).
  - Reason String, User Property.
  - Server Reference (перенаправление сервера).
- **5.0:** Reason Code 0x04 = Disconnect with Will Message.

## E.11. AUTH

- **3.1.1:** отсутствует.
- **5.0:** новый пакет для расширенной аутентификации.

## E.12. Сессия

- **3.1.1:** Clean Session = 0 — сессия хранится бессрочно. Clean Session = 1 — удаляется при отключении.
- **5.0:** Session Expiry Interval — гибкое управление временем жизни сессии.

## E.13. Will

- **3.1.1:** Will Topic, Will Message, Will QoS, Will Retain.
- **5.0:** добавлены Will Properties:
  - Will Delay Interval.
  - Payload Format Indicator, Message Expiry Interval.
  - Content Type, Response Topic, Correlation Data, User Property.

## E.14. Обработка ошибок

- **3.1.1:** в основном закрытие соединения.
- **5.0:** Reason Codes в подтверждениях, DISCONNECT с кодом, Reason String для диагностики.

## E.15. Подписки

- **3.1.1:** QoS — единственная опция.
- **5.0:** QoS + NL + RAP + Retain Handling.
- **5.0:** Subscription Identifier.
- **5.0:** Shared Subscriptions.

## E.16. Поток данных

- **3.1.1:** нет управления потоком.
- **5.0:** Receive Maximum, Maximum Packet Size.

## E.17. Псевдонимы топиков

- **3.1.1:** нет.
- **5.0:** Topic Alias.

## E.18. Request/Response

- **3.1.1:** не формализовано, реализуется вручную.
- **5.0:** Response Topic, Correlation Data, Response Information.

## E.19. Перенаправление сервера

- **3.1.1:** нет.
- **5.0:** Server Reference + Reason Codes 0x9C, 0x9D.

## E.20. Совместимость

- **Протокол:** несовместим на уровне пакетов.
- **Модель:** одинаковая (pub/sub, QoS, retain, Will).
- **Сосуществование:** брокер может поддерживать обе версии на одном порту, различая их по Protocol Level.

---

[← Оглавление перевода](README.md) · [← Приложение D](appendix-d-implementation-guidelines.md)