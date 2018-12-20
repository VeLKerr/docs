# Управление хостами кластера

Вы можете добавлять и удалять хосты кластера, а также управлять настройками [!KEYREF PG] для отдельных кластеров.

## Получить список хостов в кластере {#list}

---

**[!TAB Консоль управления]**

1. Перейдите на страницу каталога и нажмите плитку **[!KEYREF mpg-name]**.
1. Нажмите на имя нужного кластера, затем выберите вкладку **Хосты**.


**[!TAB CLI]**

[!INCLUDE [cli-install](../../_includes/cli-install.md)]

[!INCLUDE [default-catalogue](../../_includes/default-catalogue.md)]

Чтобы получить список баз данных в кластере, выполните команду:

```
$ [!KEYREF yc-mdb-pg] host list
     --cluster-name=<имя кластера>
     
+----------------------------+--------------+---------+--------+---------------+
|            NAME            |  CLUSTER ID  |  ROLE   | HEALTH |    ZONE ID    |
+----------------------------+--------------+---------+--------+---------------+
| rc1b...mdb.yandexcloud.net | c9qp71dk1... | MASTER  | ALIVE  | ru-central1-b |
| rc1c...mdb.yandexcloud.net | c9qp71dk1... | REPLICA | ALIVE  | ru-central1-c |
+----------------------------+--------------+---------+--------+---------------+
```

Имя кластера можно запросить со [списком кластеров в каталоге](cluster-list.md#list-clusters).


**[!TAB API]**

Получить список хостов кластера можно с помощью метода [listHosts](../api-ref/Cluster/listHosts.md).

---


## Добавить хост {#add}

Количество хостов в кластерах [!KEYREF mpg-short-name] ограничено квотами на количество CPU и объем памяти, которые доступны кластерам БД в вашем облаке. Чтобы проверить используемые ресурсы, откройте страницу [Квоты](https://console.cloud.yandex.ru/?section=quotas) и найдите блок **[!KEYREF mpg-full-name]**.

---

**[!TAB Консоль управления]**

1. Перейдите на страницу каталога и нажмите плитку **[!KEYREF mpg-name]**.
1. Нажмите на имя нужного кластера и перейдите на вкладку **Хосты**. 
1. Нажмите кнопку **Добавить хост**.
1. Укажите параметры хоста:

   - зону доступности;
   - подсеть (если нужной подсети в списке нет, [создайте ее](../../vpc/operations/subnet-create.md));
   - приоритет хоста как [!KEYREF PG]-реплики;
   - источник репликации (если вы используете каскадную репликацию);
   - выберите опцию **Публичный доступ**, если хост должен быть доступен извне Облака.


**[!TAB CLI]**

[!INCLUDE [cli-install](../../_includes/cli-install.md)]

[!INCLUDE [default-catalogue](../../_includes/default-catalogue.md)]

Чтобы добавить хост в кластере:

1. Посмотрите описание команды CLI для добавления хостов:

   ```
   $ [!KEYREF yc-mdb-pg] host add --help
   ```

1. Запросите список подсетей кластера, чтобы выбрать подсеть для нового хоста:

   ```
   $ yc vpc subnet list
   
   +-----------+-----------+------------+---------------+------------------+
   |     ID    |   NAME    | NETWORK ID |     ZONE      |      RANGE       |
   +-----------+-----------+------------+---------------+------------------+
   | b0cl69... | default-c | enp6rq7... | ru-central1-c | [172.16.0.0/20]  |
   | e2lkj9... | default-b | enp6rq7... | ru-central1-b | [10.10.0.0/16]   |
   | e9b0ph... | a-2       | enp6rq7... | ru-central1-a | [172.16.32.0/20] |
   | e9b9v2... | default-a | enp6rq7... | ru-central1-a | [172.16.16.0/20] |
   +-----------+-----------+------------+---------------+------------------+
   ```

   Если нужной подсети в списке нет, [создайте ее](../../vpc/operations/subnet-create.md).

1. Выполните команду добавления хоста:

   ```
   $ [!KEYREF yc-mdb-pg] host add
        --cluster-name <имя кластера>
        --host zone-id=<зона доступности>,subnet-id=<ID подсети>
   ```
   
   [!KEYREF mpg-short-name] запустит операцию добавления хоста.
   
   Идентификатор подсети необходимо указать, если в зоне доступности больше одной подсети, в противном случае [!KEYREF mpg-short-name] автоматически выберет единственную подсеть. Имя кластера можно запросить со [списком кластеров в каталоге](cluster-list.md#list-clusters).


**[!TAB API]**

Добавить хост в кластер можно с помощью метода [addHosts](../api-ref/Cluster/addHosts.md).

---

## Изменить хост {#update}

Для каждого хоста в [!KEYREF PG]-кластере можно изменить:

- Приоритет хоста в кластере, согласно которому выбирается новый мастер при недоступности старого.
- Хост, который должен быть источником репликации для данного хоста (если вы используете [каскадную репликацию](https://www.postgresql.org/docs/10/warm-standby.html#CASCADING-REPLICATION)).

---

**[!TAB CLI]**

[!INCLUDE [cli-install](../../_includes/cli-install.md)]

[!INCLUDE [default-catalogue](../../_includes/default-catalogue.md)]

Чтобы изменить параметры [!KEYREF PG]-хоста, выполните команду:

```
$ [!KEYREF yc-mdb-pg] host update <имя хоста>
     --cluster-name <имя кластера>
     --replication-source <имя хоста-источника>
     --priority <приоритет реплики>
```

Имена хостов можно запросить со [списком хостов в кластере](#list-hosts), имя кластера — со [списком кластеров в каталоге](cluster-list.md#list-clusters).


**[!TAB API]**

Удалить хост можно с помощью метода [deleteHosts](../api-ref/Cluster/deleteHosts.md).

---

## Удалить хост {#remove}

Вы можете удалить хост из [!KEYREF PG]-кластера, если он не является единственным хостом. Чтобы заменить единственный хост, сначала создайте новый хост, а затем удалите старый.

Если хост является мастером в момент удаления, [!KEYREF mpg-short-name] автоматически назначит мастером следующую по приоритету реплику.

---

**[!TAB Консоль управления]**

1. Перейдите на страницу каталога и нажмите плитку **[!KEYREF mpg-name]**.
1. Нажмите на имя нужного кластера и выберите вкладку **Хосты**.
1. Нажмите значок ![](../../_assets/vertical-ellipsis.svg) в строке нужного хоста и выберите пункт **Удалить**.


**[!TAB CLI]**

[!INCLUDE [cli-install](../../_includes/cli-install.md)]

[!INCLUDE [default-catalogue](../../_includes/default-catalogue.md)]

Чтобы удалить хост из кластера, выполните команду:

```
$ [!KEYREF yc-mdb-pg] host delete <имя хоста>
     --cluster-name=<имя кластера>
```

Имя хоста можно запросить со [списком хостов в кластере](#list-hosts), имя кластера — со [списком кластеров в каталоге](cluster-list.md#list-clusters).


**[!TAB API]**

Удалить хост можно с помощью метода [deleteHosts](../api-ref/Cluster/deleteHosts.md).

---