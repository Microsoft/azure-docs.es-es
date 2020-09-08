---
title: Métricas personalizadas en Azure Monitor (versión preliminar)
description: Obtenga información sobre las métricas personalizadas en Azure Monitor y cómo se modelan.
author: ancav
ms.author: ancav
services: azure-monitor
ms.topic: conceptual
ms.date: 06/01/2020
ms.subservice: metrics
ms.openlocfilehash: 73c9b2bf8cf88ca5e8576c451c9d9ac5f0eae8a3
ms.sourcegitcommit: 628be49d29421a638c8a479452d78ba1c9f7c8e4
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/20/2020
ms.locfileid: "88639909"
---
# <a name="custom-metrics-in-azure-monitor-preview"></a>Métricas personalizadas en Azure Monitor (versión preliminar)

A medida que implementa recursos y aplicaciones en Azure, querrá empezar a recopilar datos de telemetría para obtener conclusiones detalladas sobre su rendimiento y mantenimiento. Azure pone algunas métricas a su disposición de manera estándar. Estas métricas se denominan [estándar o de plataforma](./metrics-supported.md). Sin embargo, están limitadas por naturaleza. 

Es posible que quiera recopilar algunos indicadores de rendimiento personalizados o métricas específicas del negocio para obtener conclusiones más detalladas. Estas métricas **personalizadas** pueden recopilarse a través de los datos de telemetría de la aplicación, un agente que se ejecute en sus recursos de Azure o, incluso, fuera del sistema de supervisión y enviarse directamente a Azure Monitor. Una vez publicadas en Azure Monitor, puede examinar, consultar y generar una alerta sobre métricas personalizadas para sus recursos y aplicaciones de Azure en paralelo con las métricas estándar emitidas por Azure.

Las métricas personalizadas en Azure Monitor están actualizadas en la versión preliminar pública. 

## <a name="methods-to-send-custom-metrics"></a>Métodos para enviar métricas personalizadas

Las métricas personalizadas pueden enviarse a Azure Monitor a través de distintos métodos:
- Instrumente la aplicación con el SDK de Azure Application Insights y envíe datos de telemetría personalizados a Azure Monitor. 
- Instale el agente de Azure Monitor (versión preliminar) en la [máquina virtual de Azure Windows o Linux](azure-monitor-agent-overview.md) y use una [regla de recopilación de datos](data-collection-rule-azure-monitor-agent.md) para enviar contadores de rendimiento a las métricas de Azure Monitor.
- Instale la extensión Windows Azure Diagnostics (WAD) en su [máquina virtual Azure](collect-custom-metrics-guestos-resource-manager-vm.md), [conjunto de escalado de máquinas virtuales](collect-custom-metrics-guestos-resource-manager-vmss.md), [máquina virtual clásica](collect-custom-metrics-guestos-vm-classic.md) o [Cloud Services clásico](collect-custom-metrics-guestos-vm-cloud-service-classic.md), y envíe los contadores de rendimiento a Azure Monitor. 
- Instale el [agente de InfluxData Telegraf](collect-custom-metrics-linux-telegraf.md) en su máquina virtual Linux de Azure y envíe las métricas mediante el complemento de salida de Azure Monitor.
- Envíe las métricas personalizadas [directamente a la API REST de Azure Monitor](./metrics-store-custom-rest-api.md), `https://<azureregion>.monitoring.azure.com/<AzureResourceID>/metrics`.

## <a name="pricing-model-and-retention"></a>Modelo de precios y retención

Consulte la [página de tarifas de Azure Monitor](https://azure.microsoft.com/pricing/details/monitor/) para más información sobre cuándo se habilitará la facturación para las consultas de métricas y las métricas personalizadas. Los detalles de precios específicos de todas las métricas, incluidas las métricas personalizadas y las consultas de métricas, están disponibles en esta página. En resumen, no hay ningún costo por ingerir métricas estándar (métricas de plataforma) en el almacén de métricas de Azure Monitor, pero las métricas personalizadas tendrán un costo cuando sean de disponibilidad general. Las consultas de la API de métricas supondrán un costo.

Las métricas personalizadas se conservan durante el [mismo tiempo que las métricas de plataforma](data-platform-metrics.md#retention-of-metrics). 

> [!NOTE]  
> Las métricas enviadas a Azure Monitor mediante el SDK de Application Insights se facturan como datos de registro ingeridos. Los cargos de métricas adicionales solo tienen un costo si se ha seleccionado la característica Application Insights [Habilitar la creación de alertas sobre las dimensiones de las métricas personalizadas](../app/pre-aggregated-metrics-log-metrics.md#custom-metrics-dimensions-and-pre-aggregation). Esta casilla envía datos a la base de datos de métricas de Azure Monitor mediante la API de métricas personalizadas para permitir unas alertas más complejas.  Obtenga más información sobre el [modelo de tarifas de Application Insights](../app/pricing.md#pricing-model) y las [tarifas en su región](https://azure.microsoft.com/pricing/details/monitor/).


## <a name="how-to-send-custom-metrics"></a>Cómo enviar métricas personalizadas

Al enviar las métricas personalizadas a Azure Monitor, cada punto de datos (o valor) notificado debe incluir la siguiente información:

### <a name="authentication"></a>Authentication
Para enviar métricas personalizadas a Azure Monitor, la entidad que envía la métrica debe tener un token de Azure Active Directory (Azure AD) válido en el encabezado de **portador** de la solicitud. Existen algunas formas admitidas para adquirir un token de portador válido:
1. [Identidades administradas para recursos de Azure](../../active-directory/managed-identities-azure-resources/overview.md). Proporciona una identidad a un recurso de Azure mismo, por ejemplo, una máquina virtual. Managed Service Identity (MSI) está diseñado para proporcionar permisos a los recursos para llevar a cabo determinadas operaciones. Un ejemplo es permitir que un recurso emita métricas sobre sí mismo. Se puede conceder a un recurso, o a su MSI, permisos de **Publicador de métricas de supervisión** en otro recurso. Con este permiso, el MSI puede emitir métricas también para otros recursos.
2. [Entidad de servicio de Azure AD](../../active-directory/develop/app-objects-and-service-principals.md). En este escenario, en una aplicación de Azure AD, o servicio, pueden asignarse permisos para emitir métricas sobre un recurso de Azure.
Para autenticar la solicitud, Azure Monitor valida el token de aplicación mediante las claves públicas de Azure AD. El rol **Supervisión del publicador de métricas** existente ya tiene este permiso. Está disponible en Azure Portal. La entidad de servicio, en función de para qué recursos vaya a emitir las métricas personalizadas, puede recibir el rol **Supervisión del publicador de métricas** en el ámbito necesario. Algunos ejemplos son una suscripción, un grupo de recursos o un recurso específico.

> [!TIP]  
> Cuando solicite un token de Azure AD para emitir métricas personalizadas, asegúrese de que la audiencia o el recurso para el que se solicita el token sea `https://monitoring.azure.com/`. Asegúrese de incluir la barra diagonal "/" final.

### <a name="subject"></a>Asunto
Esta propiedad captura para cuál identificador de recurso de Azure se notifica la métrica personalizada. Esta información se codificarán en la dirección URL de la llamada API que se realiza. Cada API solo puede enviar los valores de métrica para un único recurso de Azure.

> [!NOTE]  
> No se pueden emitir métricas personalizadas con el identificador de recurso de un grupo de recursos o una suscripción.


### <a name="region"></a>Region
Esta propiedad captura en qué región de Azure se implementa el recurso para el que se van a emitir las métricas. Las métricas se deben emitir al mismo punto de conexión regional de Azure Monitor que la región donde se implementa el recurso. Por ejemplo, las métricas personalizadas para una máquina virtual que se implementa en el Oeste de EE. UU. deben enviarse al punto de conexión de Azure Monitor regional WestUS. La información de región también está codificada en la dirección URL de la llamada API.

> [!NOTE]  
> Durante la versión preliminar pública, las métricas personalizadas solo están disponibles en un subconjunto de las regiones de Azure. En una sección posterior de este artículo se incluye una lista de regiones admitidas.
>
>

### <a name="timestamp"></a>Timestamp
Cada punto de datos que se envía a Azure Monitor debe marcarse con una marca de tiempo. Esta marca de tiempo captura la fecha y hora en que se midió o recopiló el valor de la métrica. Azure Monitor acepta datos de métrica con marcas de tiempo de hasta 20 minutos en el pasado y hasta 5 minutos en el futuro. La marca de tiempo debe estar en formato ISO 8601.

### <a name="namespace"></a>Espacio de nombres
Los espacios de nombres son una manera de clasificar o agrupar las métricas similares. Mediante el uso de espacios de nombres se puede conseguir el aislamiento entre los grupos de métricas que puedan estar recopilando diferentes conclusiones o indicadores de rendimiento. Por ejemplo, podría tener un espacio de nombres denominado **contosomemorymetrics** que realice el seguimiento de las métricas de uso de memoria que perfile una aplicación. Otro espacio de nombres denominado **contosoapptransaction** podría realizar un seguimiento de todas las métricas sobre las transacciones de usuario en la aplicación.

### <a name="name"></a>Nombre
**Nombre** es el nombre de la métrica que se está notificando. Normalmente, el nombre es lo suficientemente descriptivo como para ayudar a identificar lo que se está midiendo. Un ejemplo es una métrica que mide el número de bytes de memoria utilizados en una máquina virtual determinada. Podría tener un nombre de métrica como **Bytes de memoria en uso**.

### <a name="dimension-keys"></a>Claves de dimensión
Una dimensión es un par de valor o clave que ayuda a describir características adicionales sobre la métrica que se recopila. Mediante el uso de características adicionales, puede recopilar más información acerca de la métrica, lo que permite obtener conclusiones más detalladas. Por ejemplo, la métrica **Bytes de memoria en uso** podría tener una clave de dimensión denominada **Proceso** que capture el número de bytes de memoria que consume cada proceso en una máquina virtual. Con esta clave, puede filtrar la métrica para ver cuánta memoria usan los procesos específicos o para identificar los cinco procesos principales por uso de memoria.
Las dimensiones son opcionales, no todas las métricas pueden tener dimensiones. Una métrica personalizada puede tener hasta 10 dimensiones.

### <a name="dimension-values"></a>Valores de dimensión
Al informar de un punto de datos de métrica, para cada clave de dimensión en la métrica notificada, hay un valor de dimensión correspondiente. Por ejemplo, si quisiera informar de la memoria utilizada por ContosoApp en la máquina virtual:

* El nombre de la métrica sería **Bytes de memoria en uso**.
* La clave de dimensión sería **Proceso**.
* El valor de la dimensión sería **ContosoApp.exe**.

Al publicar un valor de métrica, solo puede especificar un único valor de dimensión por clave de dimensión. Si recopila el mismo uso de memoria para varios procesos en la máquina virtual, puede informar sobre varios valores de métricas para esa marca de tiempo. Cada valor de métrica especificaría un valor de dimensión diferente para la clave de dimensión **Proceso**.
Las dimensiones son opcionales, no todas las métricas pueden tener dimensiones. Si en una publicación de métrica se definen las claves de dimensión, los valores de dimensión correspondientes serán obligatorios.

### <a name="metric-values"></a>Valores de métrica
Azure Monitor almacena todas las métricas a intervalos de granularidad de un minuto. Somos conscientes de que, durante un minuto determinado, es posible que una métrica tenga que ser muestreada varias veces. Un ejemplo es el uso de CPU. O podría ser necesario medirlo para muchos eventos discretos. Un ejemplo son las latencias de transacción de inicio de sesión. Para limitar el número de valores sin procesar que tenga que emitir y pagar en Azure Monitor, puede agregar previamente de manera local y emitir los valores:

* **Mínimo**: el valor mínimo observado de todas las muestras y medidas durante el minuto.
* **Máximo**: el valor máximo observado de todas las muestras y medidas durante el minuto.
* **Suma**: el sumatorio de todos los valores observados de todas las muestras y medidas durante el minuto.
* **Recuento**: el número de muestras y medidas tomadas durante el minuto.

Por ejemplo, si hubo cuatro transacciones de inicio de sesión a la aplicación durante un minuto determinado, las latencias medidas resultantes para cada una podrían ser las siguientes:

|Transacción 1|Transacción 2|Transacción 3|Transacción 4|
|---|---|---|---|
|7 ms|4 ms|13 ms|16 ms|
|

La publicación de métricas resultante en Azure Monitor sería la siguiente:
* Mín.: 4
* Máx.: 16
* Suma: 40
* Recuento: 4

Si la aplicación no puede agregar previamente de manea local y debe emitir cada muestra o evento discretos inmediatamente después de la recopilación, puede emitir los valores de medida sin procesar. Por ejemplo, cada vez que se produzca una transacción de inicio de sesión en la aplicación, publica una métrica en Azure Monitor con solo una única medida. Por lo tanto, para una transacción de inicio de sesión que llevó 12 ms, la publicación de la métrica sería la siguiente:
* Mín.: 12
* Máx.: 12
* Suma: 12
* Recuento: 1

Con este proceso, puede emitir varios valores para la misma combinación de métrica más dimensión durante un minuto dado. Azure Monitor luego toma todos los valores sin procesar emitidos para un minuto dado y los suma.

### <a name="sample-custom-metric-publication"></a>Publicación de métricas personalizadas de ejemplo
En el ejemplo siguiente, creará una métrica personalizada llamada **Memoria de bytes en uso** en el espacio de nombres de métrica **Perfil de memoria** para una máquina virtual. La métrica tiene una única dimensión denominada **Proceso**. Para la marca de tiempo dada, se emiten valores de métricas para dos procesos diferentes:

```json
{
    "time": "2018-08-20T11:25:20-7:00",
    "data": {

      "baseData": {

        "metric": "Memory Bytes in Use",
        "namespace": "Memory Profile",
        "dimNames": [
          "Process"        ],
        "series": [
          {
            "dimValues": [
              "ContosoApp.exe"
            ],
            "min": 10,
            "max": 89,
            "sum": 190,
            "count": 4
          },
          {
            "dimValues": [
              "SalesApp.exe"
            ],
            "min": 10,
            "max": 23,
            "sum": 86,
            "count": 4
          }
        ]
      }
    }
  }
```
> [!NOTE]  
> Application Insights, la extensión de diagnóstico, y el agente InfluxData Telegraf ya están configurados para emitir los valores de métrica frente al punto de conexión regional correcto y transportar todas las propiedades anteriores en cada emisión.
>
>

## <a name="custom-metric-definitions"></a>Definiciones de métricas personalizadas
No es necesario definir previamente una métrica personalizada en Azure Monitor antes de emitirla. Cada punto de datos de métrica publicado contiene el espacio de nombres, el nombre y la información de la dimensión. Por tanto, la primera vez que se emite una métrica personalizada para Azure Monitor, se crea automáticamente una definición de métrica. Esta definición de métrica luego puede detectarse en cualquier recurso con respecto al que se emitió la métrica a través de las definiciones de métricas.

> [!NOTE]  
> Azure Monitor todavía no permite definir **Unidades** para una métrica personalizada.

## <a name="using-custom-metrics"></a>Uso de métricas personalizadas
Una vez que las métricas personalizadas se envían a Azure Monitor, puede examinarlas con Azure Portal y consultarlas mediante las API REST de Azure Monitor. También puede crear alertas con ellas para avisarle cuando se cumplen ciertas condiciones.

> [!NOTE]
> Debe ser un rol de lector o colaborador para ver las métricas personalizadas.

### <a name="browse-your-custom-metrics-via-the-azure-portal"></a>Exploración de las métricas personalizadas a través de Azure Portal
1.    Vaya a [Azure Portal](https://portal.azure.com).
2.    Seleccione el panel **Monitor**.
3.    Seleccione **Métricas**.
4.    Seleccione un recurso con el que haya emitido métricas personalizadas.
5.    Seleccione el espacio de nombres de métrica para la métrica personalizada.
6.    Seleccione la métrica personalizada.

## <a name="supported-regions"></a>Regiones admitidas
Durante la versión preliminar pública, la capacidad de publicar métricas personalizadas solo está disponible en un subconjunto de las regiones de Azure. Esta restricción implica que las métricas solo pueden publicarse para recursos de una de las regiones admitidas. En la tabla siguiente se enumera el conjunto de regiones de Azure admitidas para las métricas personalizadas. También se muestran los puntos de conexión correspondientes para cuyos recursos en esas regiones se deben publicar métricas:

|Región de Azure |Prefijo del punto de conexión regional|
|---|---|
| **EE. UU. y Canadá** | |
|Centro-Oeste de EE. UU. | https:\//westcentralus.monitoring.azure.com |
|Oeste de EE. UU. 2       | https:\//westus2.monitoring.azure.com |
|Centro-Norte de EE. UU | https:\//northcentralus.monitoring.azure.com
|Centro-sur de EE. UU.| https:\//southcentralus.monitoring.azure.com |
|Centro de EE. UU.      | https:\//centralus.monitoring.azure.com |
|Centro de Canadá | https:\//canadacentral.monitoring.azure.com |
|Este de EE. UU.| https:\//eastus.monitoring.azure.com |
|Este de EE. UU. 2 | https:\//eastus2.monitoring.azure.com |
| **Europa** | |
|Norte de Europa    | https:\//northeurope.monitoring.azure.com |
|Oeste de Europa     | https:\//westeurope.monitoring.azure.com |
|Sur de Reino Unido | https:\//uksouth.monitoring.azure.com
|Centro de Francia | https:\//francecentral.monitoring.azure.com |
| **África** | |
|Norte de Sudáfrica | https:\//southafricanorth.monitoring.azure.com |
| **Asia** | |
|Centro de la India | https:\//centralindia.monitoring.azure.com |
|Este de Australia | https:\//australiaeast.monitoring.azure.com |
|Japón Oriental | https:\//japaneast.monitoring.azure.com |
|Sudeste de Asia  | https:\//southeastasia.monitoring.azure.com |
|Este de Asia | https:\//eastasia.monitoring.azure.com |
|Centro de Corea del Sur   | https:\//koreacentral.monitoring.azure.com |

## <a name="latency-and-storage-retention"></a>Latencia y retención del almacenamiento

La adición de una nueva métrica o de una nueva dimensión a una métrica pueden tardar de 2 a 3 minutos en aparecer. Una vez en el sistema, los datos deben aparecer en menos de 30 segundos el 99 % del tiempo. 

Si elimina una métrica o quita una dimensión, el cambio puede tardar de una semana a un mes en eliminarse del sistema.

## <a name="quotas-and-limits"></a>Cuotas y límites
Azure Monitor impone los siguientes límites de uso a las métricas personalizadas:

|Category|Límite|
|---|---|
|Serie temporal activa/suscripciones/región|50.000|
|Claves de dimensión por métrica|10|
|Longitud de cadena de espacios de nombres de métricas, nombres de métricas, claves de dimensión y valores de dimensión|256 caracteres|

Una serie temporal activa se define como cualquier combinación única de métrica, clave de dimensión o valor de dimensión para los que se hayan publicado valores de métrica en las últimas 12 horas.

## <a name="next-steps"></a>Pasos siguientes
Use métricas personalizadas desde distintos servicios: 
 - [Máquinas virtuales](collect-custom-metrics-guestos-resource-manager-vm.md)
 - [Conjunto de escalado de máquinas virtuales](collect-custom-metrics-guestos-resource-manager-vmss.md)
 - [Azure Virtual Machines (clásico)](collect-custom-metrics-guestos-vm-classic.md)
 - [Máquina virtual Linux que usa el agente Telegraf](collect-custom-metrics-linux-telegraf.md)
 - [REST API](./metrics-store-custom-rest-api.md)
 - [Cloud Services clásico](collect-custom-metrics-guestos-vm-cloud-service-classic.md)
 

