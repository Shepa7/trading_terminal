# 📊 Prediction Market Terminal 

Торговый терминал для prediction markets с полным набором ордеров и автоматическим матчингом.

---

##  Демо

**Живая версия:** https://shepa7.github.io/prediction_market_terminal/

---

##  Возможности

- MARKET ордера — Мгновенное исполнение по текущей цене ✅
- LIMIT ордера — Автоматическое исполнение при достижении цены ✅
- STOP-LIMIT ордера — Активация по стоп-цене + лимитное исполнение ✅
- Самовыкуп — Авто-закрытие противоположных позиций ✅
- Заморозка средств — Блокировка баланса при размещении ордера ✅
- P&L расчет — От фактической цены исполнения ✅
- TTL ордеров — Истечение по времени (1h, 4h, 24h, GTC) ✅
- GAP Warning — Предупреждение о рисках ✅

---

##  Исправленные баги (v2.0)

1. MARKET отменялся с NO_LIQUIDITY → Синтетический контрагент ✅
2. LIMIT ордера не исполнялись → Авто-исполнение по цене рынка ✅
3. Самовыкуп только для MARKET → Работает для всех типов ✅
4. P&L считался от lastPrice → От фактической цены исполнения ✅
5. _checkExpiry() смешивал статусы → Только EXPIRED ✅
6. getTtl() не парсил 24h/4h/1h → Корректный парсинг ✅
7. renderPositions() float-ошибки → Decimal арифметика ✅
8. placeStopLimit() не морозил средства → Заморозка при размещении ✅
9. _refund() от лимитной цены → От фактического исполнения ✅
10. render() двойной рендер → requestAnimationFrame дебаунс ✅

---

##  Тестирование

### Запуск тестов


```bash
# Базовые тесты (26 тестов)
node run_tests.js

# Расширенные тесты (48 тестов)
node run_tests-2.js

Покрытие тестов
	•	LIMIT BUY YES (4 теста) — Исполнение когда цена ≤ лимита
	•	LIMIT BUY NO (2 теста) — Исполнение по NO outcome
	•	Лучшая цена (3 теста) — Исполнение по рыночной (не лимитной)
	•	Открытые ордера (2 теста) — Появление в getUserOrders()
	•	Исполненные (3 теста) — Переход PENDING → FILLED
	•	MARKET (5 тестов) — Мгновенное исполнение без NO_LIQUIDITY
	•	Самовыкуп MARKET (3 теста) — Закрытие противоположной позиции
	•	Самовыкуп LIMIT (2 теста) — Ключевой тест!
	•	Баланс (10 тестов) — Freeze/unfreeze/refund
	•	P&L (6 тестов) — Расчет прибыли/убытка
	•	TTL/Expiry (4 теста) — Истечение ордеров
	•	Валидация (9 тестов) — MIN_LOT, PRICE_RANGE, отмена

══════════════════════════════════════════════════════
  Итого: 48 тестов   ✓ 48 прошло   ✗ 0 упало
══════════════════════════════════════════════════════

ВСЕ ТЕСТЫ ПРОШЛИ ✓

Использование
MARKET ордер

const {engine, ledger} = makeEngine(0.65);

// Купить 50 YES по рынку
const order = engine.placeMarket('trader', 'BUY', 'YES', 50);

console.log(order.status);  // 'FILLED'
console.log(ledger.getBalance('trader').shares.YES);  // 50

LIMIT ордер

// Выставить лимит на покупку по <LaTex>id_1</LaTex>0.700, Лимит $0.710
const order = engine.placeStopLimit('trader', 'BUY', 'YES', 0.700, 0.710, 10);

console.log(order.isActivated);  // false

// Цена достигла стопа
engine.onPriceTick(0.700, 'YES');

console.log(order.isActivated);  // true
console.log(order.type);  // 'limit' (превратился в лимитный)


Самовыкуп

// Включить самовыкуп (в UI это toggle)
selfBuy = true;

// Купить NO
engine.placeMarket('trader', 'BUY', 'NO', 10);

// Купить YES с самовыкупом → закроет NO позицию
engine.placeMarket('trader', 'BUY', 'YES', 10);

// Позиция NO закрыта автоматически

Архитектура

┌─────────────────────────────────────────────────────┐
│                   Frontend (HTML/CSS/JS)            │
├─────────────────────────────────────────────────────┤
│  OrderEngine (EventTarget)                          │
│  ├── BalanceLedger     # Балансы и позиции          │
│  ├── OrderBook         # Стакан заявок (FIFO)       │
│  ├── StopWatcher       # Мониторинг стоп-цен        │
│  └── UI Renderer       # Отрисовка таблиц           │
├─────────────────────────────────────────────────────┤
│  Test Suite (Node.js)                               │
│  ├── run_tests.js      # Базовые тесты              │
│  └── run_tests-2.js    # Расширенные тесты          │
└─────────────────────────────────────────────────────┘

Ключевые классы

•	OrderEngine — Основной движок ордеров
	•	BalanceLedger — Управление балансами
	•	OrderBook — Стакан заявок (FIFO)
	•	StopWatcher — Активация STOP-ордеров
	•	Decimal — Точная арифметика

API Движка
Создание ордера

engine.placeMarket(userId, side, outcome, quantity)
engine.placeLimit(userId, side, outcome, price, quantity, ttl)
engine.placeStopLimit(userId, side, outcome, stopPrice, limitPrice, quantity, ttl)

Параметры
	•	userId (string) — ID пользователя
	•	side (‘BUY’ | ‘SELL’) — Направление
	•	outcome (‘YES’ | ‘NO’) — Исход
	•	price (Decimal | number) — Цена ордера
	•	quantity (number) — Количество (мин. 10)
	•	ttl (‘GTC’ | ISO string) — Срок действия

События

engine.addEventListener('ORDER_PLACED', (e) => console.log(e.data));
engine.addEventListener('ORDER_UPDATED', (e) => console.log(e.data));
engine.addEventListener('TRADE', (e) => console.log(e.data));
engine.addEventListener('PRICE_TICK', (e) => console.log(e.data));


Установка
Локальный запуск

# 1. Клонируйте репозиторий
git clone https://github.com/shepa7/trading_terminal.git
cd trading_terminal

# 2. Откройте в браузере
open index.html

# Или через локальный сервер
python3 -m http.server 8000
# http://localhost:8000


GitHub Pages
Терминал развернут на: https://shepa7.github.io/prediction_market_terminal/

Ограничения
	•	Демо-режим — Нет подключения к реальному блокчейну
	•	Ликвидность — Синтетический контрагент при пустом стакане
	•	Один рынок — Только ETH-2026-Q1
	•	Без бэкенда — Все данные в памяти браузера






