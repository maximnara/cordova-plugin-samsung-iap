# Samsung In-App Purchase Cordova Plugin

Комплексный плагин Cordova для внутриигровых покупок (IAP) в Samsung Galaxy Store с поддержкой управления продуктами, покупок, подписок и расширенной обработки ошибок.

---
# 💰 Приобретение плагина

**Данный плагин является платным.** Приобрести его можно на платформе Boosty по ссылке: [https://boosty.to/maximnara](https://boosty.to/maximnara)

✅ **В стоимость входит:**
- Полная версия плагина
- Помощь в установке
- Первичная техническая поддержка

---

## Возможности

- 🛍️ **Управление продуктами** - Запрос информации о продуктах и покупках пользователя
- 💳 **Процесс покупки** - Полная реализация покупок с обратными вызовами
- 🔄 **Поддержка подписок** - Управление подписками и акциями
- 🧪 **Тестовый режим** - Встроенные возможности тестирования с несколькими режимами работы
- ⚡ **Promise-based API** - Современный JavaScript Promise интерфейс
- 🎯 **Расширенная обработка ошибок** - Детальные коды ошибок и понятные сообщения
- 📱 **Система событий** - Обновления статуса покупок в реальном времени

## Установка

```bash
cordova plugin add cordova-plugin-samsung-iap
```

## Быстрый старт

```javascript
// Инициализация Samsung IAP
await SamsungIap.init({
    operationMode: SamsungIap.OPERATION_MODE_TEST, // Используйте тестовый режим для разработки
    debugMode: true
});

// Получение информации о продуктах
const products = await SamsungIap.getProducts(['product_id_1', 'product_id_2']);

// Совершение покупки
const purchase = await SamsungIap.purchaseItem('product_id_1');

// Подтверждение покупки (для расходуемых товаров)
await SamsungIap.consumeItem(purchase.data.purchaseToken);
```

## Справочник API

### Инициализация

#### `SamsungIap.init(options)`

Инициализация плагина Samsung IAP.

**Параметры:**
- `options.operationMode` (Number): Режим работы
  - `SamsungIap.OPERATION_MODE_PRODUCTION` (0) - Продакшн режим с реальными транзакциями
  - `SamsungIap.OPERATION_MODE_TEST` (1) - Тестовый режим без реальных транзакций
  - `SamsungIap.OPERATION_MODE_TEST_FAILURE` (-1) - Тестовый режим, где все запросы завершаются неудачно
- `options.debugMode` (Boolean): Включить логирование отладки

**Возвращает:** Promise<Object>

```javascript
const result = await SamsungIap.init({
    operationMode: SamsungIap.OPERATION_MODE_TEST,
    debugMode: true
});
console.log('IAP инициализирован:', result.data.isTestMode);
```

**Структура ответа:**
```javascript
{
    success: true,
    message: "Samsung IAP initialized successfully",
    data: {
        operationMode: 1,      // Используемый режим работы
        isTestMode: true       // true, если не продакшн режим
    }
}
```

### Управление продуктами

#### `SamsungIap.getProducts(productIds)`

Получение информации о продуктах из Samsung Galaxy Store.

**Параметры:**
- `productIds` (Array<String>): Массив ID продуктов для запроса

**Возвращает:** Promise<Array<Object>>

```javascript
const products = await SamsungIap.getProducts(['premium_upgrade', 'remove_ads']);
products.data.forEach(product => {
    console.log(`${product.productName}: ${product.price}`);
});
```

**Структура ответа:**
```javascript
{
    success: true,
    message: "Products retrieved successfully",
    data: [
        {
            productId: "premium_upgrade",        // ID продукта
            productName: "Premium Upgrade",      // Название продукта
            productDesc: "Remove ads and...",    // Описание продукта
            price: "99 ₽",                      // Локализованная цена
            priceCurrency: "RUB",               // Код валюты
            type: "inapp"                       // Тип продукта (inapp/subs)
        }
    ]
}
```

#### `SamsungIap.getOwnedItems(productType)`

Получение текущих покупок и подписок пользователя.

**Параметры:**
- `productType` (String): 'inapp' для расходуемых/не расходуемых товаров, 'subs' для подписок

**Возвращает:** Promise<Array<Object>>

```javascript
const ownedItems = await SamsungIap.getOwnedItems('inapp');
console.log('У пользователя есть:', ownedItems.data.length, 'товаров');
```

**Структура ответа:**
```javascript
{
    success: true,
    message: "Owned items retrieved successfully",
    data: [
        {
            productId: "premium_upgrade",           // ID продукта
            purchaseToken: "abc123...",             // Токен покупки
            purchaseTime: "2024-01-15T10:30:00Z",  // Время покупки
            jsonData: "{\"orderId\":\"...\"}",     // JSON данные покупки
            isConsumable: true                      // Является ли товар расходуемым
        }
    ]
}
```

### Процесс покупки

#### `SamsungIap.purchaseItem(productId, options)`

Инициация процесса покупки.

**Параметры:**
- `productId` (String): ID продукта для покупки
- `options` (Object, опционально):
  - `obfuscatedAccountId` (String): Опциональный обфусцированный идентификатор аккаунта
  - `obfuscatedProfileId` (String): Опциональный обфусцированный идентификатор профиля

**Возвращает:** Promise<Object>

```javascript
const purchase = await SamsungIap.purchaseItem('premium_upgrade', {
    obfuscatedAccountId: 'user_123'
});

console.log('Покупка успешна:', purchase.data.productId);
```

**Структура ответа:**
```javascript
{
    success: true,
    message: "Purchase completed successfully",
    data: {
        productId: "premium_upgrade",           // ID купленного продукта
        purchaseToken: "xyz789...",             // Токен покупки для подтверждения
        purchaseTime: "2024-01-15T10:30:00Z",  // Время покупки
        jsonData: "{\"orderId\":\"...\"}",     // JSON данные покупки от Samsung
        isConsumable: true                      // Нужно ли подтверждать покупку
    }
}
```

#### `SamsungIap.consumeItem(purchaseToken)`

Подтверждение покупки расходуемого товара.

**Параметры:**
- `purchaseToken` (String): Токен покупки для подтверждения

**Возвращает:** Promise<Object>

```javascript
await SamsungIap.consumeItem(purchase.data.purchaseToken);
console.log('Товар успешно подтвержден');
```

**Структура ответа:**
```javascript
{
    success: true,
    message: "Item consumed successfully",
    data: {
        purchaseId: "xyz789...",     // ID подтвержденной покупки
        consumed: true               // Статус подтверждения
    }
}
```

#### `SamsungIap.acknowledgeItem(purchaseToken)`

Подтверждение покупки (для не расходуемых товаров и подписок).

**Параметры:**
- `purchaseToken` (String): Токен покупки для подтверждения

**Возвращает:** Promise<Object>

```javascript
await SamsungIap.acknowledgeItem(purchase.data.purchaseToken);
console.log('Покупка подтверждена');
```

**Структура ответа:**
```javascript
{
    success: true,
    message: "Item acknowledged successfully",
    data: {
        purchaseId: "xyz789...",                                       // ID подтвержденной покупки
        acknowledged: true,                                            // Статус подтверждения
        note: "Samsung IAP automatically acknowledges purchases"       // Примечание о Samsung IAP
    }
}
```

### Управление подписками

#### `SamsungIap.getPromotionEligibility(productId)`

Проверка права пользователя на акции по подпискам.

**Параметры:**
- `productId` (String): ID продукта подписки

**Возвращает:** Promise<Array<Object>>

```javascript
const eligibility = await SamsungIap.getPromotionEligibility('premium_monthly');
eligibility.data.forEach(promo => {
    if (promo.hasPromotion) {
        console.log(`Акция доступна для ${promo.productId}: ${promo.pricing}`);
    }
});
```

**Структура ответа:**
```javascript
{
    success: true,
    message: "Promotion eligibility retrieved successfully",
    data: [
        {
            productId: "premium_monthly",        // ID продукта подписки
            pricing: "50% off for 3 months",    // Акционное предложение (если есть)
            hasPromotion: true                  // Доступна ли акция для пользователя
        }
    ]
}
```

### Обработка событий

Прослушивание событий IAP:

```javascript
document.addEventListener('samsung_iap_purchase_success', (event) => {
    console.log('Покупка завершена:', event.detail);
});

document.addEventListener('samsung_iap_error', (event) => {
    console.log('Ошибка IAP:', event.detail.userMessage);

    if (event.detail.isRecoverable) {
        // Показать пользователю возможность повторить попытку
    }
});
```

### Доступные события

- `samsung_iap_purchase_success` - Покупка успешно завершена
- `samsung_iap_purchase_failed` - Покупка не удалась
- `samsung_iap_consume_success` - Товар успешно подтвержден
- `samsung_iap_acknowledge_success` - Покупка подтверждена
- `samsung_iap_error` - Произошла общая ошибка IAP

## Обработка ошибок

Плагин предоставляет комплексную обработку ошибок с понятными сообщениями:

```javascript
try {
    const purchase = await SamsungIap.purchaseItem('invalid_product');
} catch (error) {
    if (error.isUserCancellation) {
        console.log('Пользователь отменил покупку');
    } else if (error.isRecoverable) {
        console.log('Временная ошибка, можно повторить:', error.userMessage);
    } else {
        console.log('Постоянная ошибка:', error.userMessage);
    }
}
```

**Структура ошибки:**
```javascript
{
    success: false,
    message: "Failed to get products",
    error: {
        errorCode: 1004,                           // Код ошибки Samsung IAP
        errorString: "IAP_ERROR_NETWORK_NOT_AVAILABLE",  // Строковое представление ошибки
        errorDetailsString: "Network connection failed"   // Детальное описание ошибки
    }
}
```

## Основные коды ошибок

- `IAP_ERROR_PAYMENT_IS_CANCELED` - Пользователь отменил платеж
- `IAP_ERROR_NETWORK_NOT_AVAILABLE` - Сеть недоступна
- `IAP_ERROR_ITEM_ALREADY_OWNED` - Товар уже принадлежит пользователю
- `IAP_ERROR_ITEM_NOT_FOUND` - Продукт не найден в магазине
- `IAP_ERROR_NEED_APP_UPGRADE` - Требуется обновление приложения

## Тестирование

### Настройка тестового режима

```javascript
// Инициализация в тестовом режиме
await SamsungIap.init({
    operationMode: SamsungIap.OPERATION_MODE_TEST,
    debugMode: true
});

// Тестовые покупки не будут взимать реальные деньги
const testPurchase = await SamsungIap.purchaseItem('test_product');
```

### Режим тестирования ошибок

```javascript
// Тестирование обработки ошибок
await SamsungIap.init({
    operationMode: SamsungIap.OPERATION_MODE_TEST_FAILURE
});

// Все запросы будут завершаться неудачно - полезно для тестирования обработки ошибок
```

## Лучшие практики

1. **Всегда подтверждайте/потребляйте покупки** для предотвращения двойных списаний
2. **Корректно обрабатывайте сетевые ошибки** с механизмами повторных попыток
3. **Используйте тестовый режим при разработке** чтобы избежать реальных списаний
4. **Проверяйте покупки на стороне сервера** используя проверку чеков Samsung
5. **Безопасно храните токены покупок** для будущего подтверждения/потребления

## Требования

- Android 6.0+ (API level 23+)
- Установленное приложение Samsung Galaxy Store
- Действующий аккаунт разработчика Samsung и регистрация приложения

## Устранение неполадок

### Частые проблемы

**В: Ошибка "Plugin not initialized"**
О: Убедитесь, что `SamsungIap.init()` вызван и завершен до других методов.

**В: Ошибки "Network not available"**
О: Проверьте интернет-соединение устройства и подключение к Samsung Galaxy Store.

**В: Ошибки "Item not found"**
О: Убедитесь, что ID продуктов точно совпадают с конфигурацией Samsung Galaxy Store.

**В: Покупки не работают в релизной сборке**
О: Убедитесь, что приложение подписано релизным keystore и загружено в Samsung Galaxy Store.

