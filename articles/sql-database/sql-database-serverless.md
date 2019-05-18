---
title: Azure SQL Database sin servidor (versión preliminar)| Microsoft Docs
description: En este artículo se describe el nuevo nivel de proceso sin servidor y se compara con el nivel de proceso aprovisionado existente.
services: sql-database
ms.service: sql-database
ms.subservice: service
ms.custom: ''
ms.devlang: ''
ms.topic: conceptual
author: oslake
ms.author: moslake
ms.reviewer: sstein, carlrab
manager: craigg
ms.date: 05/11/2019
ms.openlocfilehash: 72552f6335f3ad6742679708a639634362c49c0b
ms.sourcegitcommit: be9fcaace62709cea55beb49a5bebf4f9701f7c6
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 05/17/2019
ms.locfileid: "65823312"
---
# <a name="sql-database-serverless-preview"></a>SQL Database sin servidor (versión preliminar)

## <a name="what-is-the-serverless-compute-tier"></a>Qué es el nivel de proceso sin servidor

SQL Database sin servidor (versión preliminar) es un nivel de proceso que factura por la cantidad de proceso utilizada por una sola base de datos por segundo. Este nivel de proceso sin servidor ofrece una relación entre precio y rendimiento optimizada para bases de datos únicas con patrones de uso intermitentes e impredecibles, que pueden permitirse alguna demora en el calentamiento de los recursos de proceso después de períodos de inactividad.

Una base de datos en el nivel de proceso sin servidor se parametriza con el intervalo de proceso que puede usar y una demora de pausa automática.

![facturación sin servidor](./media/sql-database-serverless/serverless-billing.png)

### <a name="performance"></a>Rendimiento

- `MinVcore` y `MaxVcore` son parámetros configurables que definen el intervalo de capacidad de proceso disponible para la base de datos. Los límites de memoria y E/S son proporcionales al intervalo de núcleos virtuales especificado.  
- La demora de pausa automática es un parámetro configurable que define el período de tiempo de que la base de datos debe estar inactiva antes de detenerse automáticamente. La base de datos se reanuda automáticamente con el siguiente inicio de sesión.

### <a name="pricing"></a>Precios

- La factura total de una base de datos sin servidor es la suma de la factura de proceso y la factura de almacenamiento.
La facturación de proceso se basa en la cantidad de núcleos virtuales usados y memoria usada por segundo.
- El mínimo de recursos facturado se basa en el mínimo de núcleos virtuales y el mínimo de memoria.
- Mientras la base de datos está en pausa, solo se factura el almacenamiento.

## <a name="scenarios"></a>Escenarios

Este nivel de proceso sin servidor ofrece una relación entre precio y rendimiento optimizada para bases de datos únicas con patrones de uso intermitentes e impredecibles, que pueden permitirse alguna demora en el calentamiento de los recursos de proceso después de períodos de inactividad. En cambio, el nivel de proceso aprovisionado ofrece una relación entre precio y rendimiento optimizada para bases de datos únicas o agrupadas con mayor uso medio que no pueden permitirse ninguna demora en el calentamiento de los recursos de proceso.

### <a name="scenarios-well-suited-for-serverless-compute"></a>Escenarios adecuados para el proceso sin servidor

- Bases de datos únicas con patrones de uso impredecibles e intermitentes intercalados con períodos de inactividad, que pueden beneficiarse del ahorro en los precios que ofrece la facturación por segundo para la cantidad de proceso utilizada.
- Bases de datos únicas con una demanda de recursos difícil de predecir y clientes que prefieren delegar en el servicio el dimensionamiento de los recursos de proceso.
- Bases de datos únicas en el nivel de proceso aprovisionado que cambian con frecuencia los niveles de rendimiento.

### <a name="scenarios-well-suited-for-provisioned-compute"></a>Escenarios adecuados para el proceso aprovisionado

- Bases de datos únicas con un uso de proceso más considerable y habitual a lo largo del tiempo.
- Bases de datos que no pueden tolerar compensaciones de rendimiento resultantes de recortes de memoria más frecuentes o de una demora en la reanudación automática desde un estado de pausa.
- Varias bases de datos con patrones de uso impredecibles e intermitentes que se pueden consolidar en un único servidor y usan grupos elásticos para una mejor optimización de precios.

## <a name="comparison-with-provisioned-compute-tier"></a>Comparación con el nivel de proceso aprovisionado

La tabla siguiente resume las diferencias entre el nivel de proceso sin servidor y el nivel de proceso aprovisionado:

| | **Proceso sin servidor** | **Proceso aprovisionado** |
|:---|:---|:---|
|**Escenario de uso principal**| Bases de datos con uso impredecible e intermitente, intercalado con períodos de inactividad. | Bases de datos o grupos elásticos con uso más habitual.|
| **Trabajo de administración del rendimiento** |Inferior|Superior|
|**Escalado de proceso**|Automático|Manual|
|**Capacidad de respuesta del proceso**|Menor después de períodos de inactividad|Inmediato|
|**Granularidad de facturación**|Por segundo|Por hora|

## <a name="purchasing-model-and-service-tier"></a>Modelo de compra y nivel de servicio

SQL Database sin servidor solo se admite actualmente en el nivel de uso general en hardware de generación 5 en el modelo de compra de núcleos virtuales.

## <a name="autoscaling"></a>Escalado automático

### <a name="scaling-responsiveness"></a>Escalado de la capacidad de respuesta

En general, las bases de datos se ejecutan en una máquina con capacidad suficiente para satisfacer la demanda de recursos sin interrupciones para cualquier cantidad de proceso solicitada dentro de los límites establecidos por el valor `maxVcores`. En ocasiones, se produce automáticamente un equilibrio de carga si la máquina no puede satisfacer la demanda de recursos en cuestión de minutos. La base de datos permanece en línea durante el equilibrio de carga excepto durante un breve período al final de la operación cuando se deshabilitan las conexiones.

### <a name="memory-management"></a>Administración de memoria

La memoria para bases de datos sin servidor se reclama con mayor frecuencia que la de las bases de datos aprovisionadas. Este comportamiento es importante para controlar costos en el nivel de proceso sin servidor. A diferencia del proceso aprovisionado, la memoria de la caché de SQL se reclama desde una base de datos sin servidor cuando el uso de CPU o de la memoria caché es bajo.

## <a name="autopause-and-autoresume"></a>Pausa y reanudación automáticas

### <a name="autopause"></a>Pausa automática

La pausa automática se desencadena si todas las condiciones siguientes se cumplen durante la demora de pausa automática:

- Número de sesiones = 0
- CPU = 0 (para la carga de trabajo de usuario en ejecución en el grupo de usuarios)

Hay disponible una opción para deshabilitar la pausa automática si se desea.

### <a name="autoresume"></a>Reanudación automática

La reanudación automática se desencadena si se cumple cualquiera de las siguientes condiciones en cualquier momento:

|Característica|Desencadenamiento de reanudación automática|
|---|---|
|Autenticación y autorización|Iniciar sesión|
|Detección de amenazas|Habilitación o deshabilitación de la configuración de detección de amenazas en el nivel de base de datos o servidor<br>Modificación de la configuración de detección de amenazas en el nivel de base de datos o servidor|
|Detección y clasificación de datos|Adición, modificación, eliminación o visualización de las etiquetas de confidencialidad|
|Auditoría|Visualización de registros de auditoría<br>Actualización o visualización de la directiva de auditoría|
|Enmascaramiento de datos|Adición, modificación, eliminación o visualización de reglas de enmascaramiento de datos|
|Cifrado de datos transparente|Visualización del estado de cifrado de datos transparente|
|Almacén de datos de consulta (rendimiento)|Modificación o visualización de la configuración del almacén de consulta, ajuste automático|
|Ajuste automático|Aplicación y comprobación de recomendaciones de ajuste automático, como la indexación automática|
|Copia de base de datos|Creación de base de datos como copia<br>Exportación a un archivo BACPAC|
|Sincronización de datos SQL|Sincronización entre la base de datos central y las bases de datos miembro que se ejecutan según una programación configurable o bien de forma manual|
|Modificación de algunos metadatos de base de datos|Adición de nuevas etiquetas de base de datos<br>Cambio del máximo de núcleos virtuales, el mínimo de núcleos virtuales y la demora de pausa automática|
|SQL Server Management Studio (SSMS)|Al usar SSMS versión 18 y abrir una nueva ventana de consulta para cualquier base de datos en el servidor se reanudará cualquier base de datos en pausa automática en el mismo servidor. Este comportamiento no se produce si se usa SSMS versión 17.9.1 con IntelliSense deshabilitado.|

### <a name="connectivity"></a>Conectividad

Si una base de datos sin servidor está en pausa, la primera vez que se inicie sesión se reanudará la base de datos y se devolverá un error con el código 40613 que indica que la base de datos no está disponible. Una vez que se reanude la base de datos, será necesario intentar iniciar sesión de nuevo para establecer la conectividad. No es necesario modificar los clientes de la base de datos con lógica de reintento de conexión.

### <a name="latency"></a>Latencia

La latencia para pausar o reanudar automáticamente una base de datos sin servidor suele ser de 1 minuto.

### <a name="feature-support"></a>Compatibilidad de características

Las características siguientes no admiten la pausa y reanudación automáticas. Es decir, si se utiliza cualquiera de las siguientes características, la base de datos permanecerá en línea, independientemente de la duración de la inactividad de la base de datos:

- Replicación geográfica (replicación geográfica activa y grupos de conmutación por error automáticos)
- Retención de copia de seguridad a largo plazo (LTR)
- La base de datos de sincronización utilizada en la sincronización de datos SQL.


## <a name="on-boarding-into-the-serverless-compute-tier"></a>Incorporación del nivel de proceso sin servidor

La creación de una nueva base de datos o el cambio de una base de datos existente a un nivel de proceso sin servidor siguen el mismo patrón que la creación de una nueva base de datos en el nivel de proceso aprovisionado y constan de los dos pasos siguientes:

1. Especifique el nombre del objetivo de servicio. El objetivo de servicio preceptúa el nivel de servicio, la generación de hardware y el máximo de núcleos virtuales. La siguiente tabla muestra las opciones de objetivo de servicio:

   |Nombre del objetivo de servicio|Nivel de servicio|Generación de hardware|Máximo de núcleos virtuales|
   |---|---|---|---|
   |GP_S_Gen5_1|Uso general|Gen5|1|
   |GP_S_Gen5_2|Uso general|Gen5|2|
   |GP_S_Gen5_4|Uso general|Gen5|4|

2. Opcionalmente, especifique el mínimo de núcleos virtuales y la demora de pausa automática para cambiar sus valores predeterminados. En la siguiente tabla se muestran los valores disponibles para estos parámetros.

   |Parámetro|Opciones de valores|Valor predeterminado|
   |---|---|---|---|
   |Mínimo de núcleos virtuales|Cualquier valor entre {0,5, 1, 2, 4} que no supere el máximo de núcleos virtuales|0,5 núcleos virtuales|
   |Demora de pausa automática|Mín.: 360 minutos (6 horas)<br>Máx.: 10 080 minutos (7 días)<br>Incrementos: 60 minutos<br>Deshabilitar pausa automática: -1|360 minutos|

> [!NOTE]
> Actualmente no es posible usar T-SQL para mover una base de datos existente al nivel de proceso sin servidor o cambiar su tamaño de proceso, pero esto sí puede realizarse mediante Azure Portal o PowerShell.

### <a name="create-new-database-using-the-azure-portal"></a>Creación de una nueva base de datos mediante Azure Portal

Consulte [Quickstart: Creación de una base de datos única en Azure SQL Database con Azure Portal](sql-database-single-database-get-started.md).

### <a name="create-new-database-using-powershell"></a>Creación de una nueva base de datos mediante PowerShell

En el ejemplo siguiente se crea una nueva base de datos en el nivel de proceso sin servidor definida por el objetivo de servicio denominado GP_S_Gen5_4 con los valores predeterminados para el mínimo de núcleos virtuales y la demora de pausa automática.

El nivel de proceso sin servidor requiere una versión más reciente de PowerShell que la que está disponible actualmente en la galería, así que debe ejecutar `Update-Module Az.Sql` para obtener los cmdlets más recientes compatibles con el nivel de proceso sin servidor.

```powershell
New-AzSqlDatabase `
  -ResourceGroupName $resourceGroupName `
  -ServerName $serverName `
  -DatabaseName $databaseName `
  -ComputeModel Serverless `
  -Edition GeneralPurpose `
  -ComputeGeneration Gen5 `
  -MinVcore 0.5 `
  -MaxVcore 2 `
  -AutoPauseDelay 720
```

### <a name="move-existing-database-into-the-serverless-compute-tier"></a>Cambio de una base de datos existente al nivel de proceso sin servidor

En el siguiente ejemplo se mueve una base de datos única existente desde el nivel de proceso aprovisionado al nivel de proceso sin servidor. En este ejemplo se especifica explícitamente el mínimo de núcleos virtuales, el máximo de núcleos virtuales y la demora de pausa automática.

```powershell
Set-AzSqlDatabase
  -ResourceGroupName $resourceGroupName `
  -ServerName $serverName `
  -DatabaseName $databaseName `
  -Edition GeneralPurpose `
  -ComputeModel Serverless `
  -ComputeGeneration Gen5 `
  -MinVcore 1 `
  -MaxVcore 4 `
  -AutoPauseDelay 1440
```

### <a name="move-a-database-out-of-the-serverless-compute-tier"></a>Cambio de una base de datos del nivel de proceso sin servidor al nivel de proceso aprovisionado

Una base de datos sin servidor se puede mover a un nivel de proceso aprovisionado de la misma manera que se mueve una base de datos de proceso aprovisionado a un nivel de proceso sin servidor.

## <a name="modify-serverless-configuration-parameters"></a>Modificación de los parámetros de configuración sin servidor

### <a name="maximum-vcores"></a>Máximo de núcleos virtuales

El máximo de núcleos virtuales se modifica mediante el comando [Set-AzSqlDatabase](https://docs.microsoft.com/powershell/module/az.sql/set-azsqldatabase) en PowerShell con el argumento `MaxVcore`.

### <a name="minimum-vcores"></a>Mínimo de núcleos virtuales

Modificación de los núcleos virtuales mínimos se realiza mediante el [conjunto AzSqlDatabase](https://docs.microsoft.com/powershell/module/az.sql/set-azsqldatabase) comando en PowerShell mediante el `MinVcore` argumento.

### <a name="autopause-delay"></a>Demora de pausa automática

Modificar el retraso pausarautomáticamente se realiza mediante el [conjunto AzSqlDatabase](https://docs.microsoft.com/powershell/module/az.sql/set-azsqldatabase) comando en PowerShell mediante el `AutoPauseDelay` argumento.

## <a name="monitor-serverless-database"></a>Supervisión de la base de datos sin servidor

### <a name="resources-used-and-billed"></a>Recursos utilizados y facturados

Los recursos de una base de datos sin servidor están encapsulados con las siguientes entidades:

#### <a name="app-package"></a>Paquete de aplicaciones

El paquete de aplicaciones es el límite de administración de recursos más externo de una base de datos, independientemente de si la base de datos se encuentra en un nivel de proceso sin servidor o aprovisionado. El paquete de aplicaciones contiene la instancia de SQL y los servicios externos que en conjunto abarcan todos los recursos de usuario y del sistema que utiliza una base de datos en SQL Database. Entre los ejemplos de servicios externos se incluyen R y la búsqueda de texto completo. Generalmente, la instancia de SQL domina el uso de recursos global en el paquete de aplicaciones.

#### <a name="user-resource-pool"></a>Grupo de recursos de usuario

El grupo de recursos de usuario es el límite de administración de recursos más interno de una base de datos, independientemente de si la base de datos se encuentra en un nivel de proceso sin servidor o aprovisionado. El grupo de recursos de usuario abarca la CPU y la E/S para cargas de trabajo de usuario generadas por consultas de DDL (por ejemplo, CREATE, ALTER, etc.) y DML (por ejemplo, SELECT, INSERT, UPDATE, DELETE, etc.). Por lo general, estas consultas representan la proporción de uso dentro del paquete de aplicaciones más importante.

### <a name="metrics"></a>Métricas

|Entidad|Métrica|DESCRIPCIÓN|Unidades|
|---|---|---|---|
|Paquete de aplicaciones|app_cpu_percent|Porcentaje de núcleos virtuales utilizado por la aplicación con respecto al máximo de núcleos virtuales permitido para la aplicación.|Porcentaje|
|Paquete de aplicaciones|app_cpu_billed|La cantidad de procesos que se facturan para la aplicación durante el período de informe. El importe pagado durante este período es el producto de esta métrica por el precio de la unidad de núcleo virtual. <br><br>Los valores de esta métrica se determinan al agregar en el tiempo el máximo de CPU utilizada y la memoria usada por segundo. Si la cantidad utilizada es menor que la cantidad mínima aprovisionada definida por el mínimo de núcleos virtuales y la memoria mínima, se factura la cantidad mínima aprovisionada. Para comparar la CPU y la memoria con fines de facturación, la memoria se normaliza en unidades de núcleos virtuales cambiando la escala de la cantidad de GB de memoria en 3 GB por núcleo virtual.|Segundos de núcleo virtual|
|Paquete de aplicaciones|app_memory_percent|Porcentaje de memoria utilizada por la aplicación con respecto a la memoria máxima permitida para la aplicación.|Porcentaje|
|Grupo de usuarios|cpu_percent|Porcentaje de núcleos virtuales utilizado por la carga de trabajo de usuario con respecto al máximo de núcleos virtuales permitido para la carga de trabajo de usuario.|Porcentaje|
|Grupo de usuarios|data_IO_percent|Porcentaje de IOPS de datos utilizado por la carga de trabajo de usuario con respecto al máximo de IOPS de datos permitido para la carga de trabajo de usuario.|Porcentaje|
|Grupo de usuarios|log_IO_percent|Porcentaje de MB/s de registro utilizado por la carga de trabajo de usuario con respecto al máximo de MB/s de registro permitido para la carga de trabajo de usuario.|Porcentaje|
|Grupo de usuarios|workers_percent|Porcentaje de trabajos utilizado por la carga de trabajo de usuario con respecto al máximo de trabajos permitido para la carga de trabajo de usuario.|Porcentaje|
|Grupo de usuarios|sessions_percent|Porcentaje de sesiones utilizado por la carga de trabajo de usuario con respecto al máximo de sesiones permitido para la carga de trabajo de usuario.|Porcentaje|
____

> [!NOTE]
> Las métricas en Azure Portal están disponibles en el panel de base de datos para una base de datos única en **Supervisión**.

### <a name="pause-and-resume-status"></a>Estado de pausa y reanudación

En Azure Portal, se muestra el estado de la base de datos en el panel de información general del servidor que enumera las bases de datos que contiene. El estado de la base de datos también se muestra en el panel de información general de la base de datos.

Con el siguiente comando de PowerShell puede consultar el estado de pausa y reanudación de una base de datos:

```powershell
Get-AzSqlDatabase `
  -ResourceGroupName $resourcegroupname `
  -ServerName $servername `
  -DatabaseName $databasename `
  | Select -ExpandProperty "Status"
```

## <a name="resource-limits"></a>Límites de recursos

Para ver los límites de recursos, consulte [Nivel de proceso sin servidor](sql-database-vCore-resource-limits-single-databases.md#serverless-compute-tier).

## <a name="billing"></a>Facturación

La cantidad de proceso que se factura es el máximo de CPU y memoria usado en cada segundo. Si la cantidad de CPU y memoria usadas es inferior a la cantidad mínima aprovisionada para cada una, se factura la cantidad aprovisionada. Para comparar la CPU y la memoria con fines de facturación, la memoria se normaliza en unidades de núcleos virtuales cambiando la escala de la cantidad de GB de memoria en 3 GB por núcleo virtual.

- **Recurso facturado**: CPU y memoria
- **Importe facturado ($)**: precio de la unidad de núcleo virtual * máx. (mínimo de núcleos virtuales, núcleos virtuales usados, GB de memoria mínima * 1/3, GB de memoria usada * 1/3) 
- **Frecuencia de facturación**: Por segundo

El precio de la unidad de núcleo virtual es el costo por núcleo virtual por segundo. Consulte la [página de precios de Azure SQL Database](https://azure.microsoft.com/pricing/details/sql-database/single/) para conocer los precios de unidad específicos de una región determinada.

La cantidad de proceso facturada se expone mediante la métrica siguiente:

- **Métrica**: app_cpu_billed (segundos de núcleo virtual)
- **Definición**: máx. (mínimo de núcleos virtuales, núcleos virtuales usados, GB de memoria mínima * 1/3, GB de memoria usada * 1/3)
- **Frecuencia de informes**: Por minuto

Esta cantidad se calcula cada segundo y se agrega en un intervalo de 1 minuto.

Considere la posibilidad de una base de datos sin servidor configurado con núcleo virtual de 1 minuto y número de 4 núcleos virtuales.  Esto corresponde a aproximadamente 3 GB de memoria de mínimo y máximo de 12 GB de memoria.  Supongamos que el retraso de pausa automática se establece en 6 horas y la carga de trabajo de la base de datos está activa durante las primeras horas 2 de un período de 24 horas e inactiva en caso contrario.    

En este caso, la base de datos se factura por proceso y almacenamiento durante las primeras 8 horas.  Aunque la base de datos está iniciando inactiva después de la hora de 2, todavía se factura para el cálculo en las subsiguientes 6 horas, según el proceso mínimos aprovisionado mientras la base de datos está en línea.  Sólo almacenamiento se factura durante el resto del período de 24 horas mientras está en pausa la base de datos.

Más concretamente, la factura de proceso en este ejemplo se calcula como sigue:

|Intervalo de tiempo|núcleos virtuales que usa cada segundo.|Utilizar GB por segundo|Calcular la factura de dimensión|segundos de la memoria con núcleo virtual se factura en un intervalo de tiempo|
|---|---|---|---|---|
|0:00-1:00|4|9|núcleos virtuales que se usa|4 núcleos virtuales * 3600 segundos = 14400 segundos de memoria con núcleo virtual|
|1:00-2:00|1|12|Memoria usada|12 Gb * 1/3 * 3600 segundos = 14400 segundos de memoria con núcleo virtual|
|2:00-8:00|0|0|Memoria mínima aprovisionado|3 Gb * 1/3 * 21600 segundos = 21600 segundos de la memoria con núcleo virtual|
|8:00-24:00|0|0|No hay ningún proceso de factura mientras está en pausa|núcleo virtual 0 segundos|
|Segundos de núcleo virtual total facturadas más de 24 horas||||núcleo virtual 50400 segundos|

Suponga que el precio de la unidad de proceso es 0,000073 $/núcleo virtual/segundo.  A continuación, la factura para este período de 24 horas de proceso es el producto de los segundos de precio y núcleo virtual de unidad proceso facturadas: $0.000073/vCore/second * 50400 segundos de la memoria con núcleo virtual = $3,68

## <a name="available-regions"></a>Regiones disponibles

El nivel de proceso sin servidor está disponible en todas las regiones excepto las siguientes: Centro de Australia, Este de China, Norte de China, Sur de Francia, Centro de Alemania, Nordeste de Alemania, India occidental, Sur de Corea del Sur, Oeste de Sudáfrica, Norte de Reino Unido, Sur de Reino Unido, Oeste de Reino Unido y Centro-oeste de EE. UU.

## <a name="next-steps"></a>Pasos siguientes

- Para comenzar, consulte [Inicio rápido: Creación de una base de datos única en Azure SQL Database con Azure Portal](sql-database-single-database-get-started.md).
- Para ver los límites de recursos, consulte [Límites de recursos del nivel de proceso sin servidor](sql-database-vCore-resource-limits-single-databases.md#serverless-compute-tier).
