## Core_Mapper

1. Как зарегистрировать новую таблицу в системе (какие классы нужно создать, какую миграцию написать).

        DbTable_NewTable
        Model_NewTable
    
        $this->createTable($this->tableName, [
            'user_id' => 'integer',
            'field' => 'character varying(64)',
            'date' => 'timestamp with time zone',
            'from' => 'text',
            'to' => 'text',
            'record_id' => 'integer'
        ]);

2. Как происходит логирование в таблицы истории? 
   Показать в коде, рассказать, как настроить. 
   Как сделать так, чтобы поле не логировалось?

        $_logTable
        'nolog' => true
        имя_таблицы_history
        это то что описывается  в свойстве $_parameters в модели?

3. Какие виды кэшей используются? Назвать классы и особенности, показать в коде.

        Core_Cache Redis

        return Zend_Registry::isRegistered('redis') ? Zend_Registry::get('redis') : Zend_Registry::get('cache');
        Zend_Registry::get('redis')

4. Как направить запрос в реплику?
        
        переключиться с реплики на основную в обе стороны через. 
        проверку instanceof Core_ReplicaPostgres и следом: switchMainDb();

5. Как выбрать через модель 1 и несколько полей, не загружая всю модель (2 варианта).

        Model_Какаято::fetchПолеПоКлючу
        Model_Какаято::select(array $columns, $primaryKeyValue)
        Model_BankSupportDepartmentUser::fetchNameById();
        Model_BankSupportDepartmentUser::select()

## Аккредитация
1. Какие таблицы аккредитаций есть? (3 штуки)
        
        customer_accreditations
        supplier_accreditations
   
        bs_user_accreditations

2. Какое отличие по хранению рассматриваемых данных в аккредитации и переаккредитации?

        аккредитации - new 
        переаккредитации - update 

## Подача заявки

1. Из каких блоков (js файлов) состоит форма подачи заявки.

        ApplicForm.js
        Application.components.ApplicMainTab
        Application.components.lotDataPanel
   
2. Какие дополнительные события происходят при подаче заявки? Какие сервисы могут повлиять на подачу?

        валидация 
   
3. Где в коде происходит скрытие информации о поставщике и цене при отображении.
   
4. Подача заявки в уторговывание? Отличие от обычного процесса

    создаеться дополнительная запись в процедрах 

        basic_procedure_id
        frm - peretorg
        '%-%'
   
5. Подача в уторговывание ППЗ. Чем отличается от обычного уторговывания.

        новая запись не создается

## Публикация процедур

1. Как выбираются способы закупок, доступные для выбора организатору

        procedure_collections
   
2. Как выбираются и подгружаются способы закупок по классификатору ЕИС

                /**
        * Справочник СПОСОБЫ ЗАКУПКИ по классификатору ЕИС
          */
          class Model_ListPurchasemethods extends Model_ListNsi
   
3. Где и как в коде проставляется позиция лота при выборе позиции плана

        /**
        * Блок данных о позиции плана закупки
        * @param array $lotPlanInfo
        * @param string $ns
        * @return array
          */
          static public function constructLotPlanInfo($lotPlanInfo, $ns = 'ns1')
   
4. Где хранятся файлы по процедуре? Как они раскидываются по формам?

        lot_documents - table
        PersonalTemplateVariables
        var procedureDocsField = {name: 'common_files', inLot: false, title: 'Файлы по процедуре'};

        data/Contract
   
5. Как установить обеспечение и задаток в процедуре. На какие поля в БД это влияет?

        finance_guarantee
        
        GuaranteeRequestService.php
        
        
        const APP_GUARANTEE_TYPE_DEPOSIT = 1; -задаток

        const REQ_PRICE_TYPE_SUM = 1;
        const REQ_PRICE_TYPE_PERCENT = 2;

        'guarantee_application' => [
        'pseudo' => 'Размер обеспечения заявки на участие',
        'validators' => array('Float', 'NullEmpty')
        ],
        'guarantee_application_extra' => [
        'pseudo' => 'Иные требования к обеспечению заявки',
        'validators' => ['Length'],
        'params' => ['max' => 2000]
        ],
        'guarantee_contract' => [
        'pseudo' => 'Размер обеспечения обязательств по договору',
        'validators' => ['NullEmpty'],

        select finance_guarantee from applications where finance_guarantee is not null
   
6. Какое поле в БД позволяет поставщикам участвовать без ЭП

        allow_supplier_without_eds (lots)

## Заявка на создание процедуры

1. Как включить функционал, какой жизненный цикл
   
2. Как создать процедуру из заявки на основании позиции плана
   
3. Параллельное согласование

## ЕИС
1. Как организовано взаимодействие с ЕИС при публикации у нас. Показать процесс, назвать основные классы.
        OosExchange
        Model_OosMessages

        Доступ к Еису из внешки:
        Нужно  добавить в Docker.dev
        WORKDIR /usr/local/share/ca-certificates/
        
        RUN openssl s_client -connect lk.eis4.roskazna.ru:443 -showcerts </dev/null 2>/dev/null \
        | sed -e '/-----BEGIN/,/-----END/!d' \
        | tee "/usr/local/share/ca-certificates/lkca.crt" >/dev/null \
        && csplit -s -z -f lkca- lkca.crt '/-----BEGIN CERTIFICATE-----/' '{*}' \
        && for file in lkca-*; do mv "$file" "$(basename "$file").crt"; done\
        && update-ca-certificates
        
        RUN openssl s_client -connect eis4.roskazna.ru:443 -showcerts </dev/null 2>/dev/null \
        | sed -e '/-----BEGIN/,/-----END/!d' \
        | tee "/usr/local/share/ca-certificates/ca.crt" >/dev/null \
        && csplit -s -z -f ca- ca.crt '/-----BEGIN CERTIFICATE-----/' '{*}' \
        && for file in ca-*; do mv "$file" "$(basename "$file").crt"; done\
        && update-ca-certificates

        Ошибка (26) фикситься созданием папок и настройкой прав, указанных значений в конфиге api.as2.incoming_folder и api.as2.outgoing_folder. Держу в курсе XD
   
2. Стягивание с ЕИС. Классы, процесс

        Крон oosRestPublish
        Model_OosExchange
        OosExchange 
        $this->placeholderContent['protocolApplications'] = $this->getOosExchange()->constructApplicationsXml($this->protocol, $this->procedure);
   
3. Где формируются пакеты, где хранятся, как отправляются файлы

        oos_messages - table
        /cron/sendOOSPlans/
        /скщт/sendOOSPackets/

## НР
1. Рассказать про интеграцию. Когда и как появляются пакеты. Как уходят.
   
2. Зачем нужен криптосервис в этом процессе, где применяется.

## Финансы
1. Основные таблицы для хранения и логирования финансовой информации. Блокировки/разблокировки/списания.
   
        transaction_log
   
2. НДС счет, счет без НДС, спецсчета.
   
3. Рассказать о взаимодействии с банковским шлюзом

        scripts/BankGateWorker.php
        abstract class WorkerDaemon
   
        Крон для отправки пакетов со стороны ЭТП в банк: 
        cron/sendBankGateMessages

        
   
4. Где в коде происходят блокировки по заявке. Какие типы блокоровок бывают?

        abstract class Model_Deposit_BlockerAbstract
        interface Model_Deposit_BlockerInterface
        function blockDeposit(
        Model_Tariff_TariffInspector
        Core_Balance::correctDeposit($this, Model_TransactionLog::DEPOSIT_BLOCKED,

        /**
        * Class Model_Deposit_NdsRateBlocker
        * Блокировка средств при подаче заявки на пролонгацию закупочного тарифа
        *   Нужен чтобы можно было гибко и в одном месте менять логику блокировки для закупочных тарифов
            */
            class Model_Deposit_RateBlocker extends Model_Deposit_BlockerAbstract {

        /**
        * Блокировка 1% от Общей стоимости заказа с НДС
          */
          class Model_Deposit_TradeFeeBlocker extends Model_Deposit_BlockerAbstract {

5. Когда происходит списание средств.

        при определении победителя и заключения договора

## Заключение договора
1. Где выбираются поставщики для заключения договора и как?

        applications
        lot_suppliers_list
   
2. Этапы перехода договора.

        contracts
        is_send_supplier
        is_last_version
   
3. Где происходит подписание файлов договора?
        
        contract_files_sign
        OrderContractContragent
        abstract class OrderContractContragent
        $contractContragent = (new OrderContractContragentFactory)->create($extract_p7z, null);
   
4. Как добавляется надпечатка?

   class Imprinter при скачивание подписанных доков для договора pdf jpeg xls

## Общее
1. Как включить подпись на площадке/отключить. Какой основной класс используется для работы с ЭП
        Конфиг 400-crypto
        'crypto->fake_crypt'
        checkCertificate(

   
2. Как работают мьютексы. Как правильно организовать блокировку между двумя экшенами меняющими один объект.

        После этого участок кода становится недоступным остальным потокам – их выполнение блокируется до тех пор, 
        пока мьютекс не будет освобождён. Освобождение должен провести поток, заблокировавший мьютекс, вызовом

        Задачей мьютекса является защита объекта от доступа к нему других потоков, 
        отличных от того, который завладел мьютексом. 
        В каждый конкретный момент только один поток может владеть объектом, защищённым мьютексом.

        abstract class WorkerDaemon
        Core_SynchronizeMutex::unlockIfNotNull($mms);


        Mutex :: create - Создать мьютекс
        Mutex :: destroy - Уничтожить мьютекс
        Mutex :: lock - Получить мьютекс
        Mutex :: trylock - Попытка получить мьютекс
        Mutex :: unlock - Освободить мьютекс


        pcntl_signal(
   
3. Где происходит проверка прав в коде.

        /library/Core/Controller/Plugin/Access.php
   
4. Как сгенерить событие в extjs и прокинуть его в другую форму. Показать на примере.

        addEvent, RelayEvent, FireEvent
   
5. Как отправить сообщение в RabbitMQ. Как разобрать. Что такое воркер, Как организована многопоточность при разборе. Рассказать про основные настройки воркера.

        MqProvider
        class Core_Amqp_MqProvider
   
        abstract class Service_Eis_ProtocolGenerator_SmspAbstract extends Service_Eis_ProtocolGenerator_Abstract
