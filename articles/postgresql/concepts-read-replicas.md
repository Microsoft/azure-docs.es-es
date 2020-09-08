---
title: 'Réplicas de lectura en Azure Database for PostgreSQL: servidor único'
description: En este artículo se describe la característica de réplica de lectura de Azure Database for PostgreSQL de servidor único.
author: rachel-msft
ms.author: raagyema
ms.service: postgresql
ms.topic: conceptual
ms.date: 08/10/2020
ms.openlocfilehash: 608740ea52cf82485bae073d9679107ac52baa28
ms.sourcegitcommit: cd0a1ae644b95dbd3aac4be295eb4ef811be9aaa
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/19/2020
ms.locfileid: "88611133"
---
# <a name="read-replicas-in-azure-database-for-postgresql---single-server"></a>Réplicas de lectura en Azure Database for PostgreSQL: servidor único

La característica de réplica de lectura permite replicar datos de un servidor Azure Database for PostgreSQL en un servidor de solo lectura. Puede crear hasta cinco réplicas desde el servidor maestro. Las réplicas se actualizan de manera asincrónica mediante la tecnología de replicación nativa del motor de PostgreSQL.

Las réplicas son nuevos servidores que se administran de forma similar a los servidores de Azure Database for PostgreSQL normales. En cada réplica de lectura, se le cobra por el proceso aprovisionado en núcleos virtuales y el almacenamiento aprovisionado en GB/mes.

Vea cómo [crear y administrar réplicas](howto-read-replicas-portal.md).

## <a name="when-to-use-a-read-replica"></a>Casos en los que utilizar las réplicas de lectura
La finalidad de la característica de réplica de lectura es ayudar a mejorar el rendimiento y la escala de las cargas de trabajo que conllevan un gran número de operaciones de lectura. Las cargas de trabajo de lectura se pueden aislar en las réplicas, mientras que las cargas de trabajo de escritura se pueden dirigir al servidor maestro.

Un caso habitual es que las cargas de trabajo de análisis e inteligencia empresarial utilicen la réplica de lectura como origen de datos para los informes.

Dado que las réplicas son de solo lectura, no reducen directamente las cargas de capacidad de escritura en el servidor maestro. Esta característica no está destinada a cargas de trabajo intensivas de escritura.

Esta característica de réplica de lectura utiliza la replicación asincrónica nativa de PostgreSQL. La característica no está diseñada para escenarios de replicación sincrónica. Habrá un retraso medible entre el servidor maestro y la réplica. Los datos de la réplica se vuelven finalmente coherentes con los datos del servidor maestro. Use esta característica con cargas de trabajo que puedan admitir este retraso.

## <a name="cross-region-replication"></a>Replicación entre regiones
Puede crear una réplica de lectura en una región distinta del servidor maestro. La replicación entre regiones puede ser útil para escenarios como el planeamiento de la recuperación ante desastres o la incorporación de datos más cerca de los usuarios.

>[!NOTE]
> Los servidores de nivel básico solo admiten la replicación de la misma región.

Puede tener un servidor maestro en cualquier [región de Azure Database for PostgreSQL](https://azure.microsoft.com/global-infrastructure/services/?products=postgresql). Un servidor maestro puede tener una réplica emparejada en su región o en las regiones de la réplica universal. En la imagen siguiente se muestran las regiones de réplica disponibles en función de la región maestra.

[ ![Regiones de réplica de lectura](media/concepts-read-replica/read-replica-regions.png)](media/concepts-read-replica/read-replica-regions.png#lightbox)

### <a name="universal-replica-regions"></a>Regiones de réplica universal
Siempre puede crear una réplica de lectura en cualquiera de las siguientes regiones, con independencia de dónde se encuentre el servidor maestro. Estas son las regiones de réplica universal:

Este de Australia, Sudeste de Australia, Centro de EE. UU., Este de Asia, Este de EE. UU. 2, Este de Japón, Oeste de Japón, Centro de Corea del Sur, Sur de Corea del Sur, Centro y norte de EE. UU., Norte de Europa, Centro-sur de EE. UU., Sudeste de Asia, Sur de Reino Unido, Oeste de Reino Unido, Oeste de Europa, Oeste de EE. UU., Oeste de EE. UU. 2, Centro-oeste de EE. UU.

### <a name="paired-regions"></a>Regiones emparejadas
Además de las regiones de réplica universal, puede crear una réplica de lectura en la región emparejada de Azure del servidor maestro. Si no conoce el par de la región, puede obtener más información en el [artículo sobre regiones emparejadas de Azure](../best-practices-availability-paired-regions.md).

Si usa réplicas entre regiones para planear la recuperación ante desastres, se recomienda que cree la réplica en la región emparejada en lugar de en una de las otras regiones. Las regiones emparejadas evitan actualizaciones simultáneas y priorizan el aislamiento físico y la residencia de datos.  

Existen limitaciones que deben considerarse: 

* Disponibilidad regional: Azure Database for PostgreSQL está disponible en Centro de Francia, Norte de Emiratos Árabes Unidos y Centro de Alemania. Sin embargo, sus regiones emparejadas no están disponibles.
    
* Pares unidireccionales: Algunas regiones de Azure se emparejan solo en una dirección. Estas regiones incluyen Oeste de la India y Sur de Brasil. 
   Esto significa que un servidor maestro de Oeste de la India puede crear una réplica en Sur de la India. Sin embargo, un servidor maestro de Sur de la India no puede crear una réplica en Oeste de la India. Esto es debido a que la región secundaria del Oeste de la India es Sur de la India, pero la región secundaria de esta última no es Oeste de la India.


## <a name="create-a-replica"></a>Creación de una réplica
Cuando se inicia el flujo de trabajo de creación de la réplica, se crea un servidor Azure Database for PostgreSQL en blanco. El nuevo servidor se rellena con los datos que estaban en el servidor maestro. El tiempo que se tarda en crear la réplica depende de la cantidad de datos en el servidor maestro y del tiempo desde la última copia de seguridad completa semanal. Puede oscilar desde unos minutos hasta varias horas.

Cada réplica se habilita para el [crecimiento automático](concepts-pricing-tiers.md#storage-auto-grow) del almacenamiento. La característica de crecimiento automático permite a la réplica mantenerse al día con los datos replicados en ella y evitar una interrupción en la replicación causada por errores de almacenamiento insuficiente.

La característica de réplica de lectura usa la replicación física de PostgreSQL (replicación no lógica). El modo de funcionamiento predeterminado es la transmisión de la replicación mediante espacios de replicación. Cuando es necesario, se usa el trasvase de registros para actualizarse.

Aprenda a [crear una réplica de lectura en Azure Portal](howto-read-replicas-portal.md).

## <a name="connect-to-a-replica"></a>Conexión a una réplica
Al crear una réplica, no se heredan las reglas de firewall o el punto de conexión de servicio de red virtual del servidor maestro. Estas reglas se deben configurar de forma independiente para la réplica.

La réplica hereda su cuenta de administrador del servidor maestro. Todas las cuentas de usuario existentes en el servidor se replican en las réplicas de lectura. Solo se puede conectar a una réplica de lectura utilizando las cuentas de usuario disponibles en el servidor maestro.

Puede conectarse a la réplica mediante su nombre de host y una cuenta de usuario válida, igual que haría en un servidor Azure Database for PostgreSQL normal. En un servidor denominado **myreplica** con el nombre de usuario administrador **myadmin**, puede conectarse a la réplica mediante psql:

```
psql -h myreplica.postgres.database.azure.com -U myadmin@myreplica -d postgres
```

Cuando se le solicite, escriba la contraseña de la cuenta de usuario.

## <a name="monitor-replication"></a>Supervisión de la replicación
Azure Database for PostgreSQL proporciona dos métricas para supervisar la replicación. Las dos métricas son **Max Lag Across Replicas** (Retraso máximo entre réplicas) y **Replica Lag** (Retraso entre réplicas). Para obtener información sobre cómo ver estas métricas, consulte la sección **Supervisión de una réplica** del [artículo de procedimientos de réplica de lectura](howto-read-replicas-portal.md).

La métrica **Retraso máximo entre réplicas** muestra el retardo en bytes entre la réplica con mayor retardo y el servidor maestro. Esta métrica está disponible únicamente en el servidor maestro.

La métrica **Replica Lag** muestra el tiempo desde la última transacción reproducida. Si no se produce ninguna transacción en el servidor maestro, la métrica refleja este retardo de tiempo. Esta métrica está disponible únicamente para los servidores de réplica. El retraso entre réplicas se calcula desde la vista `pg_stat_wal_receiver`:

```SQL
EXTRACT (EPOCH FROM now() - pg_last_xact_replay_timestamp());
```

Establezca una alerta que le informe cuando el retardo de la réplica alcance un valor que no sea aceptable para la carga de trabajo. 

Para más información, consulte el servidor maestro directamente para obtener el retardo de replicación en bytes en todas las réplicas.

En la versión 10 de PostgreSQL:

```SQL
select pg_wal_lsn_diff(pg_current_wal_lsn(), replay_lsn) 
AS total_log_delay_in_bytes from pg_stat_replication;
```

En la versión 9.6 y versiones anteriores de PostgreSQL:

```SQL
select pg_xlog_location_diff(pg_current_xlog_location(), replay_location) 
AS total_log_delay_in_bytes from pg_stat_replication;
```

> [!NOTE]
> Si se reinicia un servidor maestro o una réplica de lectura, el tiempo que tarda en reiniciarse y ponerse al día se reflejará en la métrica de retardo de la réplica.

## <a name="stop-replication"></a>Detención replicación
Puede decidir detener la replicación entre un servidor maestro y una réplica. La acción de detención reinicia la réplica para quitar su configuración de replicación. Una vez que se ha detenido la replicación entre un servidor maestro y una réplica de lectura, la réplica se convierte en un servidor independiente. Los datos del servidor independiente son los datos que estaban disponibles en la réplica en el momento en que se inició el comando para detener la replicación. El servidor independiente no alcanza al servidor maestro.

> [!IMPORTANT]
> Este servidor independiente no puede volver a convertirse en una réplica.
> Asegúrese de que la réplica tenga todos los datos que necesita antes de detener la replicación en una réplica de lectura.

Al detener la replicación, la réplica pierde todos los vínculos con su servidor maestro anterior y otras réplicas.

Aprenda a [detener la replicación en una réplica](howto-read-replicas-portal.md).

## <a name="failover"></a>Conmutación por error
No se produce ninguna conmutación por error automatizada entre el servidor maestro y la réplica. 

Dado que la replicación es asincrónica, se produce un retraso entre el maestro y la réplica. La cantidad de retraso puede verse afectada por varios factores, como lo pesada que sea la carga de trabajo que se ejecuta en el servidor maestro y la latencia entre los centros de datos. En la mayoría de los casos, el retraso de la réplica oscila entre unos segundos y un par de minutos. Puede realizar un seguimiento del retraso real de la replicación mediante la métrica *Replica Lag* (Retraso de réplica), que está disponible para cada réplica. Esta métrica muestra el tiempo desde la última transacción reproducida. Se recomienda que observe el retraso de la réplica durante un período de tiempo para identificar el retraso medio. Puede establecer una alerta sobre el retraso de la réplica, para que si se sale del intervalo esperado, puede tomar medidas.

> [!Tip]
> Si realiza la conmutación por error a la réplica, el retraso en el momento de desvincular la réplica del maestro indicará la cantidad de datos perdidos.

Cuando haya decidido que quiere conmutar por error a una réplica, realice estos pasos: 

1. Detenga la replicación en la réplica.<br/>
   Este paso es necesario para que el servidor de la réplica pueda aceptar escrituras. Como parte de este proceso, el servidor de réplica se reiniciará y se desvinculará del maestro. Una vez iniciada la detención de la replicación, el proceso de back-end tarda aproximadamente dos minutos en completarse. Consulte la sección [Detención de la replicación](#stop-replication) de este artículo para conocer las implicaciones de esta acción.
    
2. Haga que la replicación apunte a la réplica (anterior).<br/>
   Cada servidor tiene una cadena de conexión única. Actualice la aplicación para que apunte a la réplica (anterior) en lugar de al servidor maestro.
    
Una vez que la aplicación procesa correctamente las lecturas y las escrituras, ya está completa la conmutación por error. La cantidad de tiempo de inactividad que experimente su aplicación dependerá del momento en que se detecte una incidencia y se realicen los pasos 1 y 2 anteriores.


## <a name="considerations"></a>Consideraciones

En esta sección se resumen las consideraciones sobre la característica de réplica de lectura.

### <a name="prerequisites"></a>Requisitos previos
Tanto las réplicas de lectura como la [descodificación lógica](concepts-logical.md) dependen del registro de escritura previa (WAL) Postgres para obtener información. Estas dos características necesitan diferentes niveles de registro de Postgres. La descodificación lógica requiere un mayor nivel de registro que las réplicas de lectura.

Para configurar el nivel de registro adecuado, use el parámetro de soporte de replicación de Azure. El soporte de la replicación de Azure tiene tres opciones de valor:

* **Desactivado**: coloca la más mínima información en el WAL. Este valor no está disponible en la mayoría de los servidores Azure Database for PostgreSQL.  
* **Réplica**: más detallado que **Off**. Este es el nivel mínimo de registro necesario para que [las réplicas de lectura](concepts-read-replicas.md) funcionen. Esta es la configuración predeterminada en la mayoría de los servidores.
* **Lógico**: más detallado que **Réplica**. Este es el nivel mínimo de registro para que funcione la descodificación lógica. Las réplicas de lectura también funcionan con este valor.

El servidor debe reiniciarse después de un cambio de este parámetro. Internamente, este parámetro establece los parámetros Postgres `wal_level`, `max_replication_slots` y `max_wal_senders`.

### <a name="new-replicas"></a>Nuevas réplicas
Las réplicas de lectura se crean como nuevos servidores Azure Database for PostgreSQL. Un servidor no puede volver a convertirse en una réplica. No se puede crear una réplica de otra réplica de lectura.

### <a name="replica-configuration"></a>Configuración de réplicas
Una réplica se crea con la misma configuración de proceso y almacenamiento que el servidor maestro. Una vez creada una réplica, se pueden cambiar varias configuraciones, incluido el período de retención de almacenamiento y copia de seguridad.

La réplica no hereda las reglas de firewall, las reglas de red virtual ni la configuración de parámetros del servidor maestro cuando se crea o con posterioridad.

### <a name="scaling"></a>Ampliación
Escalado de núcleos virtuales o entre Uso general y Memoria optimizada:
* PostgreSQL requiere que `max_connections` la configuración en un servidor secundario sea [mayor o igual que el valor de la principal](https://www.postgresql.org/docs/current/hot-standby.html), de lo contrario, la réplica secundaria no se iniciará.
* En Azure Database for PostgreSQL, las conexiones máximas permitidas para cada servidor se corrigen en la SKU de proceso, ya que las conexiones ocupan memoria. Puede obtener más información sobre la [asignación entre max_connections y las SKU de proceso](concepts-limits.md).
* **Escalado**: En primer lugar, escale el proceso de una réplica y, a continuación, escale la principal. Esta orden impedirá que los errores no cumplan `max_connections` el requisito.
* **Reducción vertical**: En primer lugar, reduzca verticalmente el proceso principal y, a continuación, reduzca verticalmente la réplica. Si intenta escalar la réplica por debajo de la principal, se producirá un error, ya que esto infringe el`max_connections` requisito.

Escalado de almacenamiento:
* Todas las réplicas tienen habilitado el crecimiento automático de almacenamiento para evitar problemas de replicación de una réplica de almacenamiento completo. Esta configuración no se puede deshabilitar.
* También puede escalar verticalmente el almacenamiento de forma manual, como haría en cualquier otro servidor


### <a name="basic-tier"></a>Nivel Basic
Los servidores de nivel básico solo admiten la replicación de la misma región.

### <a name="max_prepared_transactions"></a>max_prepared_transactions
[PostgreSQL requiere](https://www.postgresql.org/docs/current/runtime-config-resource.html#GUC-MAX-PREPARED-TRANSACTIONS) que el valor del parámetro `max_prepared_transactions` en la réplica de lectura sea mayor o igual que el valor principal; en caso contrario, no se iniciará la réplica. Si quiere cambiar `max_prepared_transactions` en el servidor maestro, primero cámbielo en las réplicas.

### <a name="stopped-replicas"></a>Réplicas detenidas
Si decide detener la replicación entre un servidor maestro y una réplica de lectura, esta se reiniciará para aplicar el cambio. La réplica detenida se convierte en un servidor independiente que acepta las lecturas y escrituras. Este servidor independiente no puede volver a convertirse en una réplica.

### <a name="deleted-master-and-standalone-servers"></a>Servidores maestro e independiente eliminados
Cuando se elimina un servidor maestro, todas sus réplicas de lectura se convierten en servidores independientes. Las réplicas se reiniciarán para reflejar este cambio.

## <a name="next-steps"></a>Pasos siguientes
* Aprenda a [crear y administrar réplicas de lectura en Azure Portal](howto-read-replicas-portal.md).
* Aprenda a [crear y administrar réplicas de lectura en la CLI de Azure y con API REST](howto-read-replicas-cli.md).
