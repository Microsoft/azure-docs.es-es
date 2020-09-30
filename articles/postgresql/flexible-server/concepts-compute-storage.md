---
title: 'Opciones de proceso y almacenamiento de Azure Database for PostgreSQL: servidor flexible'
description: 'En este artículo se explican las opciones de proceso y almacenamiento de Azure Database for PostgreSQL: servidor flexible.'
author: lfittl-msft
ms.author: lufittl
ms.service: postgresql
ms.topic: conceptual
ms.date: 09/22/2020
ms.openlocfilehash: a149d147b9817d8fde7a4fa7eb1b0e7a7eea8283
ms.sourcegitcommit: 53acd9895a4a395efa6d7cd41d7f78e392b9cfbe
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/22/2020
ms.locfileid: "90932477"
---
# <a name="compute-and-storage-options-in-azure-database-for-postgresql---flexible-server"></a>Opciones de proceso y almacenamiento de Azure Database for PostgreSQL: servidor flexible

> [!IMPORTANT]
> Azure Database for PostgreSQL: servidor flexible está en versión preliminar.

Puede crear un servidor de Azure Database for PostgreSQL en tres planes de tarifa diferentes: flexible, de uso general y optimizado para memoria. Los planes de tarifa se diferencian por la cantidad de proceso en núcleos virtuales que se puede aprovisionar, la cantidad de memoria por núcleo virtual y la tecnología de almacenamiento usada para almacenar los datos. Todos los recursos se aprovisionan en el nivel de servidor PostgreSQL. Un servidor puede tener una o varias bases de datos.

| Recurso/nivel | **Flexible** | **Uso general** | **Memoria optimizada** |
|:---|:----------|:--------------------|:---------------------|
| Núcleos virtuales | 1, 2 | 4, 8, 16, 32, 64 | 4, 8, 16, 32, 48 y 64 |
| Memoria por núcleo virtual | Variable | 4 GB | De 6,75 GB a 8 GB |
| Tamaño de almacenamiento | De 32 GB a 16 TB | De 32 GB a 16 TB | De 32 GB a 16 TB |
| Período de retención de copias de seguridad de base de datos | De 7 a 35 días | De 7 a 35 días | De 7 a 35 días |

Para elegir un plan de tarifa, use la siguiente tabla como punto de partida.

| Plan de tarifa | Carga de trabajo objetivo |
|:-------------|:-----------------|
| Flexible | Ideal para cargas de trabajo que no necesitan toda la CPU continuamente. |
| Uso general | La mayoría de las cargas de trabajo de empresa que requieren un equilibrio entre proceso y memoria con rendimiento de E/S escalable. Por ejemplo, servidores para hospedar aplicaciones web y móviles, y otras aplicaciones empresariales.|
| Memoria optimizada | Cargas de trabajo de base de datos de alto rendimiento que requieren rendimiento en memoria para un procesamiento de transacciones más rápido y una mayor simultaneidad. Por ejemplo, servidores para procesar datos en tiempo real y aplicaciones de análisis y transacciones de alto rendimiento.|

Después de crear un servidor, el nivel de proceso, el número de núcleos virtuales y el tamaño de almacenamiento se pueden aumentar o reducir en cuestión de segundos. También se puede aumentar o reducir de forma independiente el período de retención de copia de seguridad. Para más información, consulte la sección [Escalado de recursos](#scale-resources).

## <a name="compute-tiers-vcores-and-server-types"></a>Niveles de proceso, núcleos virtuales y tipos de servidor

Los recursos de proceso se pueden seleccionar en función del nivel, los núcleos virtuales y el tamaño de la memoria. Los núcleos virtuales representan la CPU lógica del hardware subyacente.

Las especificaciones detalladas de los tipos de servidores disponibles son las siguientes:

| Nombre de SKU             | Núcleos virtuales | Tamaño de memoria | IOPS máximas admitidas | Ancho de banda de E/S máximo admitido |
|----------------------|--------|-------------|--------------------|-----------------------------|
| **Flexible**        |        |             |                    |                             |
| B1ms                 | 1      | 2 GiB       | 640                | 15 MiB/s                  |
| B2s                  | 2      | 4 GiB       | 1280               | 15 MiB/s                  |
| **Uso general**  |        |             |                    |                             |
| D2s_v3               | 2      | 8 GiB       | 3200               | 48 MiB/s                  |
| D4s_v3               | 4      | 16 GiB      | 6400               | 96 MiB/s                  |
| D8s_v3               | 8      | 32 GiB      | 12800              | 192 MiB/s                 |
| D16s_v3              | 16     | 64 GiB      | 18000              | 384 MiB/s                 |
| D32s_v3              | 32     | 128 GB     | 18000              | 750 MiB/s                 |
| D48s_v3              | 48     | 192 GiB     | 18000              | 750 MiB/s                 |
| D64s_v3              | 64     | 256 GiB     | 18000              | 750 MiB/s                 |
| **Memoria optimizada** |        |             |                    |                             |
| E2s_v3               | 2      | 16 GiB      | 3200               | 48 MiB/s                  |
| E4s_v3               | 4      | 32 GiB      | 6400               | 96 MiB/s                  |
| E8s_v3               | 8      | 64 GiB      | 12800              | 192 MiB/s                 |
| E16s_v3              | 16     | 128 GB     | 18000              | 384 MiB/s                 |
| E32s_v3              | 32     | 256 GiB     | 18000              | 750 MiB/s                 |
| E48s_v3              | 48     | 384 GiB     | 18000              | 750 MiB/s                 |
| E64s_v3              | 64     | 432 GiB     | 18000              | 750 MiB/s                 |

## <a name="storage"></a>Storage

El almacenamiento que se aprovisiona es la cantidad de capacidad de almacenamiento disponible para el servidor de Azure Database for PostgreSQL. El almacenamiento se usa para los archivos de base de datos, los archivos temporales, los registros de transacciones y los registros del servidor PostgreSQL. La cantidad total de almacenamiento que se aprovisiona también define la capacidad de E/S disponible para su servidor.

El almacenamiento está disponible en los siguientes tamaños fijos:

| Tamaño del disco | E/S |
|:---|:---|
| 32 GiB | Aprovisionado de 120, máximo de 3500 |
| 64 GiB | Aprovisionado de 240, máximo de 3500 |
| 128 GB | Aprovisionado de 500, máximo de 3500 |
| 256 GiB | Aprovisionado de 1100, máximo de 3500 |
| 512 GB | Aprovisionado de 2300, máximo de 3500 |
| 1 TiB | 5\.000 |
| 2 TiB | 7 500 |
| 4 TiB | 7 500 |
| 8 TiB | 16 000 |
| 16 TiB | 18 000 |

Tenga en cuenta que las IOPS también están restringidas por el tipo de máquina virtual. Aunque puede seleccionar cualquier tamaño de almacenamiento independientemente del tipo de servidor, es posible que no pueda usar todas las IOPS que proporciona el almacenamiento, especialmente si elige un servidor con un número pequeño de núcleos virtuales.

Puede agregar capacidad de almacenamiento adicional durante y después de la creación del servidor.

>[!NOTE]
> El almacenamiento solo se puede escalar verticalmente, no reducir.

Puede supervisar el consumo de E/S en Azure Portal o mediante los comandos de la CLI de Azure. Las métricas pertinentes que se deben supervisar son el [límite de almacenamiento, el porcentaje de almacenamiento, el almacenamiento usado y el porcentaje de E/S](concepts-monitoring.md).

### <a name="maximum-iops-for-your-configuration"></a>Número máximo de IOPS para la configuración

|Nombre de SKU            |Tamaño de almacenamiento en GiB                       |32 |64 |128 |256 |512  |1024|2 048|4 096|8192 |16 384|
|--------------------|------------------------------------------|---|---|----|----|-----|-----|-----|-----|------|------|
|                    |Número máximo de IOPS                              |120|240|500 |1100|2300 |5000 |7500 |7500 |16000 |18000 |
|**Flexible**       |                                          |   |   |    |    |     |     |     |     |      |      |
|B1ms                |640 IOPS                                  |120|240|500 |640*|640* |640* |640* |640* |640*  |640*  |
|B2s                 |1280 IOPS                                 |120|240|500 |1100|1280*|1280*|1280*|1280*|1280* |1280* |
|**Uso general** |                                          |   |   |    |    |     |     |     |     |      |      |
|D2s_v3              |3200 IOPS                                 |120|240|500 |1100|2300 |3200*|3200*|3200*|3200* |3200* |
|D4s_v3              |6400 IOPS                                |120|240|500 |1100|2300 |5000 |6400*|6400*|6400* |6400* |
|D8s_v3              |12 800 IOPS                               |120|240|500 |1100|2300 |5000 |7500 |7500 |12 800*|12 800*|
|D16s_v3             |18 000 IOPS                               |120|240|500 |1100|2300 |5000 |7500 |7500 |16000 |18000 |
|D32s_v3             |18 000 IOPS                               |120|240|500 |1100|2300 |5000 |7500 |7500 |16000 |18000 |
|D48s_v3             |18 000 IOPS                               |120|240|500 |1100|2300 |5000 |7500 |7500 |16000 |18000 |
|D64s_v3             |18 000 IOPS                               |120|240|500 |1100|2300 |5000 |7500 |7500 |16000 |18000 |
|**Memoria optimizada**|                                          |   |   |    |    |     |     |     |     |      |      |
|E2s_v3              |3200 IOPS                                 |120|240|500 |1100|2300 |3200*|3200*|3200*|3200* |3200* |
|E4s_v3              |6400 IOPS                                |120|240|500 |1100|2300 |5000 |6400*|6400*|6400* |6400* |
|E8s_v3              |12 800 IOPS                               |120|240|500 |1100|2300 |5000 |7500 |7500 |12 800*|12 800*|
|E16s_v3             |18 000 IOPS                               |120|240|500 |1100|2300 |5000 |7500 |7500 |16000 |18000 |
|E32s_v3             |18 000 IOPS                               |120|240|500 |1100|2300 |5000 |7500 |7500 |16000 |18000 |
|E48s_v3             |18 000 IOPS                               |120|240|500 |1100|2300 |5000 |7500 |7500 |16000 |18000 |
|E64s_v3             |18 000 IOPS                               |120|240|500 |1100|2300 |5000 |7500 |7500 |16000 |18000 |

Si están marcadas con un \*, las IOPS están limitadas por el tipo de máquina virtual seleccionado. De lo contrario, las IOPS están limitadas por el tamaño de almacenamiento seleccionado.

### <a name="maximum-io-bandwidth-mibsec-for-your-configuration"></a>Ancho de banda de E/S máximo (MiB/s) para la configuración

|Nombre de SKU            |Tamaño del almacenamiento, GiB                             |32 |64 |128 |256 |512  |1024|2 048|4 096|8192 |16 384|
|--------------------|----------------------------------------------|---|---|----|----|-----|-----|-----|-----|------|------|
|                    |**Ancho de banda de almacenamiento, MiB/s**                |25 |50 |100 |125 |150  |200  |250  |250  |500   |750   |
|**Flexible**       |                                              |   |   |    |    |     |     |     |     |      |      |
|B1ms                |10 MiB/s                                    |10*|10*|10* |10* |10*  |10*  |10*  |10*  |10*   |10*   |
|B2s                 |15 MiB/s                                    |15*|15*|15* |15* |15*  |15*  |15*  |15*  |15*   |15*   |
|**Uso general** |                                              |   |   |    |    |     |     |     |     |      |      |
|D2s_v3              |48 MiB/s                                    |25 |48*|48* |48* |48*  |48*  |48*  |48*  |48*   |48*   |
|D4s_v3              |96 MiB/s                                    |25 |50 |96* |96* |96*  |96*  |96*  |96*  |96*   |96*   |
|D8s_v3              |192 MiB/s                                   |25 |50 |100 |125 |150  |192* |192* |192* |192*  |192*  |
|D16s_v3             |384 MiB/s                                   |25 |50 |100 |125 |150  |200  |250  |250  |384*  |384*  |
|D32s_v3             |750 MiB/s                                   |25 |50 |100 |125 |150  |200  |250  |250  |500   |750   |
|D48s_v3             |750 MiB/s                                   |25 |50 |100 |125 |150  |200  |250  |250  |500   |750   |
|D64s_v3             |750 MiB/s                                   |25 |50 |100 |125 |150  |200  |250  |250  |500   |750   |
|**Memoria optimizada**|                                              |   |   |    |    |     |     |     |     |      |      |
|E2s_v3              |48 MiB/s                                    |25 |48*|48* |48* |48*  |48*  |48*  |48*  |48*   |48*   |
|E4s_v3              |96 MiB/s                                    |25 |50 |96* |96* |96*  |96*  |96*  |96*  |96*   |96*   |
|E8s_v3              |192 MiB/s                                   |25 |50 |100 |125 |150  |192* |192* |192* |192*  |192*  |
|E16s_v3             |384 MiB/s                                   |25 |50 |100 |125 |150  |200  |250  |250  |384*  |384*  |
|E32s_v3             |750 MiB/s                                   |25 |50 |100 |125 |150  |200  |250  |250  |500   |750   |
|E48s_v3             |750 MiB/s                                   |25 |50 |100 |125 |150  |200  |250  |250  |500   |750   |
|E64s_v3             |750 MiB/s                                   |25 |50 |100 |125 |150  |200  |250  |250  |500   |750   |

Si está marcado con un \*, el ancho de banda de E/S está limitado por el tipo de máquina virtual seleccionado. De lo contrario, el ancho de banda de E/S está limitado por el tamaño de almacenamiento seleccionado.

### <a name="reaching-the-storage-limit"></a>Alcance del límite de almacenamiento

Cuando se alcanza el límite de almacenamiento, el servidor comienza a devolver errores y evita cualquier modificación adicional. Esto también puede producir problemas con otras actividades operativas, como las copias de seguridad y el archivo de WAL.

Se recomienda supervisar de forma activa el espacio en disco en uso y aumentar el tamaño del disco en previsión de cualquier situación de insuficiencia de almacenamiento. Puede configurar una alerta que avise cuando el almacenamiento del servidor se acerque al límite para poder evitar problemas de falta de espacio en disco. Para obtener más información, consulte la documentación sobre [cómo configurar una alerta](howto-alert-on-metrics.md).

### <a name="storage-auto-grow"></a>Crecimiento automático del almacenamiento

El crecimiento automático del almacenamiento todavía no está disponible para los servidores flexibles.

## <a name="backup"></a>Copia de seguridad

El servicio realiza automáticamente copias de seguridad del servidor. Puede seleccionar un período de retención de entre 7 y 35 días. Obtenga más información sobre las copias de seguridad en el [artículo de conceptos](concepts-backup-restore.md).

## <a name="scale-resources"></a>Escalado de recursos

Después de crear el servidor, puede cambiar los núcleos virtuales, el nivel de proceso, la cantidad de almacenamiento y el período de retención de copia de seguridad de manera independiente. El número de núcleos virtuales se pueden escalar o reducir verticalmente. El período de retención de copia de seguridad se puede escalar o reducir verticalmente de 7 a 35 días. El tamaño de almacenamiento solo se puede aumentar. El escalado de los recursos puede realizarse a través del portal o la CLI de Azure.

> [!NOTE]
> El tamaño de almacenamiento solo se puede aumentar. Tras aplicar el aumento del tamaño de almacenamiento, no puede volver a otro más pequeño.

Al cambiar el número de núcleos virtuales o el nivel de proceso, el servidor se reinicia para que se aplique el nuevo tipo de servidor. Durante el breve espacio de tiempo en que el sistema cambia al nuevo servidor, no se puede establecer ninguna nueva conexión y todas las transacciones no confirmadas se revierten. Este intervalo de tiempo varía, pero en la mayoría de los casos es inferior a un minuto. El escalado del almacenamiento funciona de la misma manera y también requiere un reinicio breve.

El cambio del período de retención de copia de seguridad es una operación en línea.

## <a name="pricing"></a>Precios

Para conocer la información más actualizada sobre precios, consulte la [página de precios](https://azure.microsoft.com/pricing/details/PostgreSQL/) del servicio. Para ver el costo de la configuración deseada, en [Azure Portal](https://portal.azure.com/#create/Microsoft.PostgreSQLServer) se muestra el costo mensual en la pestaña **Plan de tarifa** según las opciones que seleccione. Si no tiene una suscripción de Azure, puede usar la calculadora de precios de Azure para obtener un precio estimado. En el sitio web [Calculadora de precios de Azure](https://azure.microsoft.com/pricing/calculator/), seleccione **Agregar elementos**, expanda la categoría **Bases de datos** y elija **Azure Database for PostgreSQL** para personalizar las opciones.

## <a name="next-steps"></a>Pasos siguientes

- Aprenda cómo [crear un servidor PostgreSQL en el portal](how-to-manage-server-portal.md).
- Obtenga información acerca de los [límites de servicio](concepts-limits.md).
