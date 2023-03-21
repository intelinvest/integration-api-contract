# integration-api-contract
API contract for develop integrations

Контракт описывает соглашение, которого должно придерживаться приложение/модуль
для интеграции сторонних API для получения транзакций. 

Это может быть:
* API крипто-биржи
* API брокера
* API блокчейна
* API любого друго сервиса

# API endpoints
Модуль должен реализовывать данные методы API

## /status
должен реализовывать GET запрос и отдавать http-код 200, если модуль запущен и функционирует в нормальном режиме
Формат ответа и тело ответа не нужны


## /input/credentials
должен реализовывать POST запрос, в качестве request-body принимается json-объект с данными полями

ниже представлены модели и примеры, написанные на `java`

```
public class Credentials {

    /** ключ доступа к стороннему API (может быть не задан) */
    private String accessKey;
    
    /** секретный ключ доступа к стороннему API (может быть не задан) */
    private String secretKey;
    
    /** адрес кошелька, идентификатор счета (который интегрируется) */
    private String address;
    
    /** пароль для доступа к стороннему API (может быть не задан) */
    private String password;
    
    /** дата начала выборки транзакций (может быть не задана) */
    private String startDate;
    
    /** уникальный идентификатор запроса (необходим для служебных действий, например, для сохранения оригинального ответа от стороннего API) */
    private String requestId;
}
``` 

Набор полей, которые должны быть указаны в запросе, определяет модуль.
Например, для интеграции с API блокчейна чаще всего требуется только адрес кошелька.
Для интеграции с API крипто-биржи может потребоваться сочетание accessKey, secretKey и password

`startDate` - может бы не передано в запросе, тогда модуль сам решает с какой начальной даты запрашивать транзакции.
Так как это зависит от API к которому пишется модуль.

## Структура ответа

В ответе должны содержаться все необходимые данные по транзакциям.

### DealsImportResult - результат обработки транзакций

```
DealsImportResult {

    /** Список ошибок при обработке транзакций */
    private final List<DealImportError> errors = new ArrayList<>();
    
    /** Список обработанных транзакций */
    private final List<ImportTradeDataHolder> transactions = new ArrayList<>();
    
    /** Список данных по активам/бумагам (необходимые для создания сущности в системе) */
    private final List<AssetModel> assetMetaData = new ArrayList<>();
    
    /** Список балансов по заданному аккаунту */
    private Map<String, BigDecimal> currentMoneyRemainders = new HashMap<>();
    
    /** Глобальная ошибка импорта */
    private String generalError;
    
    /** Признак валидности ответа (используется когда нет сделок, и ответ из API пустой) */
    private boolean reportValid = false;
    
    /** Дополнительные инструкции для обработки транзаций */
    private ParseInstruction parseInstruction;
}    
 ```
 
 errors - заполняется ошибками при парсинге транзакции (может быть не заполнен)
 transactions - транзакции, которые удалось преобразовать
 assetMetaData - данные для создания актива в системе (может быть не заполнен)
 currentMoneyRemainders - данные по балансам (позициям) аккаунта (может быть не заполнен)
 generalError - глобальная ошибка (может быть не задана)
 parseInstruction - инструкции для парсинга (может быть не заполнен)
 
 
 
 ### DealImportError - сущность ошибки обработки транзакции
 
 ```
 public class DealImportError {

    /** текст ошибки */
    private String message;
    
    /** дата транзакции с которой произошла ошибка (при наличии) */
    @JsonFormat(shape = JsonFormat.Shape.STRING, pattern = "yyyy-MM-dd")
    private Date dealDate;
    
    /** уникальный идентификатор бумаги/актива в транзакции с которой произошла ошибка (при наличии) */
    private String dealTicker;
    
    /** код валюты актива/бумаги (при наличии) */
    private String currency;
}
```


### ImportTradeDataHolder - сущность с полями транзакции

```
public class ImportTradeDataHolder {

    /** цена из транзакции */
    private BigDecimal price;
    
    /** тип операции */
    private Operation operation;
    
    /** дата транзакции */
    private Date date;
    
    /** количество актива/бумаг */
    private BigDecimal quantity;
    
    /** комиссия */
    private BigDecimal fee = BigDecimal.ZERO;
    
    /** заметка (при наличии) */
    private String note;
    
    /** список алиасов актива/бумаги */
    private List<ShareAlias> shareAliases = new ArrayList<>();
    
    /** сумма транзакции */
    private BigDecimal sum;
    
    /** НКД (для транзакций по облигациям, может быть не задано) */
    private BigDecimal nkd;
    
    /** номинал (для транзакций по облигациям, может быть не задано) */
    private BigDecimal facevalue;
    
    /** код валюты актива/бумаги */
    private String currency;
    
    /** код валюту комиссии */
    private String feeCurrency;
    
    /** сумма транзакции (заполняется только для транзакций с денежными средствами) */
    private BigDecimal moneyAmount;
    
    /** данные по связанной сделке */
    private ImportTradeDataHolder linked;
    
    /** уникальный идентификатор транзакции (при наличии) */
    protected String tradeSystemId;
    
    /** признак указания НКД или дивиденда на 1 бумагу */
    private boolean perOne = true;
    
    /** предпочтительный тип сделки, может быть не указан */
    private TradePreferredType tradePreferredType;
}
``` 


### ShareAlias - данные для идентификации актива/бумаги

```
public class ShareAlias {
    
    /** уникальный идентификатор бумаги (Тикер) (при наличии) */
    private String ticker;
    
    /** уникальный код бумаги (ISIN) (при наличии) */
    private String isin;
    
    /** алиас бумаги (при наличии) */
    private String alias;
    
    /** предпочитаемый тип бумаги (может быть не указан) */
    private ShareType preferredType;
    
    /** биржа бумаги (код, название) (при налиичии) */
    private String exchange;
}
```


### ShareType - тип бумаги

```
public enum ShareType {

    STOCK,
    BOND,
    ASSET,
    CURRENCY;
}
````

### Operation - тип операции транзакции

```
public enum Operation {

    BUY,
    SELL,
    COUPON,
    DIVIDEND,
    AMORTIZATION,
    INCOME,
    LOSS,
    CURRENCY_BUY,
    CURRENCY_SELL,
    SHARE_IN,
    SHARE_OUT,
    DIVIDEND_SHARE,
    DEPOSIT,
    WITHDRAW,
    REPAYMENT,
    CALCULATION;
}
```
* CALCULATION -  системный общий тип для Дивиденда, Купона, Погашения
* SHARE_IN, SHARE_OUT - можно указывать если поле price в транзации не указано
* BUY, SELL - применяются для транзакций с активом/бумагой
* DEPOSIT, WITHDRAW, LOSS, INCOME - для транзакций с денежными средствами


### AssetModel - сущность описывающая актив/бумагу из транзакции

```
public class AssetModel {

    /** Тип актива */
    private AssetCategory category;

    /** Код актива (может быть ticker/isin или еще какой-то однозначно идентифицирующий актив во внешней системе) */
    private String ticker;

    /** Валюта актива */
    private String currency;

    /** Название актива */
    private String name;

    /** Цена актива (текущая) */
    @Nullable
    private BigDecimal price;

    /** Заметка */
    @Nullable
    private String note;

    /** Количество значащих разрядов в цене */
    private int decimals = 9;

    /** Признак закрепления актива за пользователем, а не за системой */
    private boolean userDefined;

    /** Метаданные по активу */
    @JsonRawValue
    private String metaData;
}
```


### AssetCategory - категория актива/бумаги

```
public enum AssetCategory {

    STOCK,
    BOND,
    MONEY,
    ETF,
    FUTURE,
    OPTION,
    METALL,
    REALTY,
    CURRENCY,
    CRYPTO_CURRENCY,
    NFT,
    OTHER;
}
```


### TradePreferredType - предпочитаемый тип создаваемой транзации

```
public enum TradePreferredType {

    MoneyTrade,
    MoneyDepositTrade,
    CurrencyTrade,
    MoneyWithdrawTrade,
    MoneyIncomeTrade,
    MoneyLossTrade,
    DividendTrade,
    CouponTrade,
    AmortizationTrade
}
```


### ParseInstruction - дополнительные инстркции для парсинга транзакций

```
public class ParseInstruction {

    private boolean correctQuantityByShareLotSize;
    private boolean correctQuantityByAmountAndPrice;
    private boolean holdOriginFeeCurrency;
    private boolean calculateQuantity;
    private boolean calculateNkdPerOne;
    private boolean calculateFacevalue;
    private boolean calculateBondPrice;
}
```

# Общие положения

* модуль может быть написан на любом ЯП
* модуль должен иметь Docker файл и уметь работать как самостоятельно, так из контейнера


# Пример на spring-boot

## StatusEndpoint
```
@RestController
@RequestMapping("status")
public class StatusEndpoint {

    @GetMapping()
    public ResponseEntity<Void> status() {
        return ResponseEntity.ok().build();
    }
}
```

## EntryEndpoint

```
@RestController
@RequiredArgsConstructor
@RequestMapping("/input")
public class EntryEndpoint {

    private final EtherscanService etherscanService;

    @PostMapping("credentials")
    public ResponseEntity<DealsImportResult> processIntegration(@Valid @RequestBody Credentials credentials) {
        EtherTransactionReport etherTransactionReport = etherscanService.getReport(credentials.getAddress());
        EtherscanApiImporter etherscanApiImporter = new EtherscanApiImporter(etherTransactionReport);
        DealsImportResult result = etherscanApiImporter.process();
        return ResponseEntity.ok(result);
    }
}
```

## EtherscanService

Реализует логику для получения данных из API Etherscan, с учетом ограничения на количество 
запросов в секунду. Например, получаем закомиченные транзакции (https://docs.etherscan.io/api-endpoints/accounts#get-a-list-of-normal-transactions-by-address)


## EtherscanApiImporter

Реализует логику по обработке списка транзакций полученных из сервиса, приводя их к модели контакта описанной выше
