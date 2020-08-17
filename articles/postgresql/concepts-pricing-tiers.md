---
title: 'Planes de tarifa en Azure Database for PostgreSQL: servidor único'
description: 'En este artículo se describen las opciones de proceso y almacenamiento en Azure Database for PostgreSQL: servidor único.'
author: rachel-msft
ms.author: raagyema
ms.service: postgresql
ms.topic: conceptual
ms.date: 02/25/2020
ms.openlocfilehash: 77ce6c8112e8c694b4ccb6a657b24649437d2f07
ms.sourcegitcommit: dccb85aed33d9251048024faf7ef23c94d695145
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/28/2020
ms.locfileid: "87279243"
---
# <a name="pricing-tiers-in-azure-database-for-postgresql---single-server"></a>Limitaciones de Azure Database for PostgreSQL: servidor único

Puede crear un servidor de Azure Database for PostgreSQL en tres planes de tarifa diferentes: Básico, De uso general y Optimizado para memoria. Los planes de tarifa se diferencian por la cantidad de proceso en núcleos virtuales que se puede aprovisionar, la cantidad de memoria por núcleo virtual y la tecnología de almacenamiento usada para almacenar los datos. Todos los recursos se aprovisionan en el nivel de servidor PostgreSQL. Un servidor puede tener una o varias bases de datos.

| Recurso/nivel | **Basic** | **Uso general** | **Memoria optimizada** |
|:---|:----------|:--------------------|:---------------------|
| Generación de procesos | Gen 4, Gen 5 | Gen 4, Gen 5 | Gen 5 |
| Núcleos virtuales | 1, 2 | 2, 4, 8, 16, 32, 64 |2, 4, 8, 16, 32 |
| Memoria por núcleo virtual | 2 GB | 5 GB | 10 GB |
| Tamaño de almacenamiento | De 5 GB a 1 TB | De 5 GB a 16 TB | De 5 GB a 16 TB |
| Período de retención de copias de seguridad de base de datos | De 7 a 35 días | De 7 a 35 días | De 7 a 35 días |

Para elegir un plan de tarifa, use la siguiente tabla como punto de partida.

| Plan de tarifa | Carga de trabajo objetivo |
|:-------------|:-----------------|
| Básico | Cargas de trabajo que requieren proceso y rendimiento de E/S ligeros. Algunos ejemplos son los servidores que se usan para desarrollo o prueba, o bien las aplicaciones a pequeña escala que se usan con poca frecuencia. |
| De uso general | La mayoría de las cargas de trabajo de empresa que requieren un equilibrio entre proceso y memoria con rendimiento de E/S escalable. Por ejemplo, servidores para hospedar aplicaciones web y móviles, y otras aplicaciones empresariales.|
| Memoria optimizada | Cargas de trabajo de base de datos de alto rendimiento que requieren rendimiento en memoria para un procesamiento de transacciones más rápido y una mayor simultaneidad. Por ejemplo, servidores para procesar datos en tiempo real y aplicaciones de análisis y transacciones de alto rendimiento.|

Después de crear un servidor, el número de núcleos virtuales, la generación de hardware y el plan de tarifa (excepto hacia y desde Básico) se puede aumentar o reducir en cuestión de segundos. También puede ajustar de forma independiente la cantidad de almacenamiento y aumentar o reducir el período de retención sin que las aplicaciones experimenten tiempo de inactividad. No puede cambiar el tipo de almacenamiento de copia de seguridad. Para más información, consulte la sección [Escalado de recursos](#scale-resources).

## <a name="compute-generations-and-vcores"></a>Procesar generaciones y núcleos virtuales

Los recursos de proceso se proporcionan como núcleos virtuales, que representan la CPU lógica del hardware subyacente. Este de China 1, Norte de China 1, US DoD (centro) y US DoD (este) usan CPU lógicas Gen 4 que se basan en los procesadores Intel E5-2673 v3 (Haswell) de 2,4 GHz. Todas las demás regiones utilizan CPU lógicas Gen 5 que se basan en procesadores Intel E5-2673 v4 (Broadwell) de 2,3 GHz.

## <a name="storage"></a>Storage

El almacenamiento que se aprovisiona es la cantidad de capacidad de almacenamiento disponible para el servidor de Azure Database for PostgreSQL. El almacenamiento se usa para los archivos de base de datos, los archivos temporales, los registros de transacciones y los registros del servidor PostgreSQL. La cantidad total de almacenamiento que se aprovisiona también define la capacidad de E/S disponible para su servidor.

| Atributos de almacenamiento | **Basic** | **Uso general** | **Memoria optimizada** |
|:---|:----------|:--------------------|:---------------------|
| Tipo de almacenamiento | Almacenamiento básico | Almacenamiento de uso general | Almacenamiento de uso general |
| Tamaño de almacenamiento | De 5 GB a 1 TB | De 5 GB a 16 TB | De 5 GB a 16 TB |
| Tamaño de incremento de almacenamiento | 1 GB | 1 GB | 1 GB |
| E/S | Variable |3 IOPS/GB<br/>100 IOPS mín.<br/>20 000 IOPS máx. | 3 IOPS/GB<br/>100 IOPS mín.<br/>20 000 IOPS máx. |

> [!NOTE]
> Se admite el almacenamiento de hasta 16 TB y 20 000 IOPS en las siguientes regiones: Este de EE. UU., Este de EE. UU. 2, Centro de EE. UU., Oeste de EE. UU., Centro-norte de EE. UU., Centro-sur de EE. UU., Norte de Europa, Oeste de Europa, Sur de Reino Unido, Oeste de Reino Unido, Sudeste de Asia, Asia Pacífico, Este de Japón, Oeste de Japón, Centro de Corea del Sur, Sur de Corea del Sur, Este de Australia, Sudeste de Australia, Oeste de EE. UU. 2 y Centro-oeste de EE. UU.
>
> El resto de regiones admiten hasta 4 TB de almacenamiento y 6000 IOPS.
>

Puede agregar capacidad de almacenamiento adicional durante y después de la creación del servidor y permitir que el sistema aumente el almacenamiento automáticamente en función del consumo de almacenamiento de su carga de trabajo.

>[!NOTE]
> El almacenamiento solo se puede escalar verticalmente, no reducir.

El plan Básico no proporciona una garantía de IOPS. En los planes de tarifa Uso general y Memoria optimizada, el valor de IOPS se escala con el tamaño de almacenamiento aprovisionado en una proporción 3:1.

Puede supervisar el consumo de E/S en Azure Portal o mediante los comandos de la CLI de Azure. Las métricas pertinentes que se deben supervisar son el [límite de almacenamiento, el porcentaje de almacenamiento, el almacenamiento usado y el porcentaje de E/S](concepts-monitoring.md).

### <a name="reaching-the-storage-limit"></a>Alcance del límite de almacenamiento

Los servidores con 100 GB o menos de almacenamiento aprovisionado se marcan como de solo lectura si el almacenamiento disponible es inferior a 512 MB o el 5 % del tamaño del almacenamiento aprovisionado. Los servidores con más de 100 GB de almacenamiento aprovisionado se marcan como solo de lectura cuando el almacenamiento libre es inferior a 5 GB.

Por ejemplo, si ha aprovisionado 110 GB de almacenamiento y el uso real supera los 105 GB, el servidor se marca como de solo lectura. O bien, si ha aprovisionado 5 GB de almacenamiento, el servidor se marca como de solo lectura cuando el almacenamiento disponible se vuelva inferior a 512 MB.

Si el servidor se establece en solo lectura, todas las sesiones existentes se desconectan, y las transacciones no confirmadas se revierten. Las operaciones de escritura y las confirmaciones de transacción posteriores producirán errores. Todas las consultas de lectura subsiguientes seguirán funcionando sin interrupciones.  

Puede aumentar la cantidad de almacenamiento aprovisionado en el servidor, o iniciar una sesión nueva en modo de lectura-escritura y quitar datos para recuperar almacenamiento disponible. Al ejecutar `SET SESSION CHARACTERISTICS AS TRANSACTION READ WRITE;`, la sesión actual se establece en modo de lectura-escritura. Para evitar daños en los datos, no realice operaciones de escritura cuando el servidor aún esté en estado de solo lectura.

Le recomendamos que active el almacenamiento automático o que configure una alerta que le envíe una notificación cada vez que su almacenamiento en servidor esté cerca del umbral para que pueda evitar entrar en el estado de solo lectura. Para obtener más información, consulte la documentación sobre [cómo configurar una alerta](howto-alert-on-metric.md).

### <a name="storage-auto-grow"></a>Crecimiento automático del almacenamiento

El crecimiento automático del almacenamiento impide que el servidor se quede sin almacenamiento y se vuelva de solo lectura. Si el crecimiento automático del almacenamiento está habilitado, el almacenamiento crece automáticamente sin afectar a la carga de trabajo. Para servidores con 100 GB o menos de almacenamiento aprovisionado, el tamaño del almacenamiento aprovisionado se incrementa en 5 GB tan pronto como el almacenamiento disponible se encuentra por debajo de 1 GB o el 10 % del almacenamiento aprovisionado. Para los servidores con más de 100 GB de almacenamiento aprovisionado, el tamaño del almacenamiento aprovisionado se incrementa en un 5 % cuando el espacio de almacenamiento disponible es inferior al mayor de los dos valores de 10 GB o 5 % del tamaño de almacenamiento aprovisionado. Se aplican los límites máximos de almacenamiento según lo especificado anteriormente.

Por ejemplo, si ha aprovisionado 1000 GB de almacenamiento y el uso real supera los 950 GB, el tamaño de almacenamiento del servidor se incrementa a 1050 GB. Como alternativa, si ha aprovisionado 10 GB de almacenamiento, el tamaño del almacenamiento aumenta a 15 GB cuando queda menos de 1 GB de almacenamiento.

Recuerde que el almacenamiento solo se puede escalar verticalmente, no reducir.

## <a name="backup-storage"></a>Almacenamiento de copia de seguridad

Azure Database for PostgreSQL proporciona hasta un 100 % del almacenamiento del servidor aprovisionado como almacenamiento de copia de seguridad, sin costos adicionales. El almacenamiento de copia de seguridad que use que supere esta cantidad se cobrará en GB por mes. Por ejemplo, si aprovisiona un servidor con 250 GB de almacenamiento, tiene 250 GB de almacenamiento adicional disponible para las copias de seguridad del servidor sin ningún cargo. El almacenamiento de copias de seguridad que supere los 250 GB se cobra según el [modelo de precios](https://azure.microsoft.com/pricing/details/postgresql/). Para comprender los factores que influyen en el uso del almacenamiento de copia de seguridad, la supervisión y el control del costo del almacenamiento de copia de seguridad, puede consultar la [documentación de copia de seguridad](concepts-backup.md).

## <a name="scale-resources"></a>Escalado de recursos

Después de crear el servidor, puede cambiar los núcleos virtuales, la generación de hardware, el plan de tarifa (excepto hacia y desde Básico), la cantidad de almacenamiento y el período de retención de copia de seguridad de manera independiente. No puede cambiar el tipo de almacenamiento de copia de seguridad. El número de núcleos virtuales se pueden escalar o reducir verticalmente. El período de retención de copia de seguridad se puede escalar o reducir verticalmente de 7 a 35 días. El tamaño de almacenamiento solo se puede aumentar. El escalado de los recursos puede realizarse a través del portal o la CLI de Azure. Para ver un ejemplo de escalado con la CLI de Azure, consulte [Supervisión y escalado de un servidor de Azure Database for PostgreSQL mediante la CLI de Azure](scripts/sample-scale-server-up-or-down.md).

> [!NOTE]
> El tamaño de almacenamiento solo se puede aumentar. Tras aplicar el aumento del tamaño de almacenamiento, no puede volver a otro más pequeño.

Al cambiar el número de núcleos virtuales, la generación de hardware o el plan de tarifa, se crea una copia del servidor original con la nueva asignación de recursos de proceso. Una vez que el nuevo servidor está en funcionamiento, las conexiones se transfieren a él. Durante el breve espacio de tiempo en que el sistema cambia al nuevo servidor, no se puede establecer ninguna nueva conexión y todas las transacciones no confirmadas se revierten. Este intervalo de tiempo varía, pero en la mayoría de los casos es inferior a un minuto.

El escalado del almacenamiento y el cambio del período de retención de copia de seguridad son operaciones verdaderamente en línea. No hay ningún tiempo de inactividad y la aplicación no se ve afectada. A medida que el valor de IOPS se escala con el tamaño del almacenamiento aprovisionado, puede aumentar el número de IOPS disponibles para el servidor mediante el escalado vertical del almacenamiento.

## <a name="pricing"></a>Precios

Para conocer la información más actualizada sobre precios, consulte la [página de precios](https://azure.microsoft.com/pricing/details/PostgreSQL/) del servicio. Para ver el costo de la configuración deseada, en [Azure Portal](https://portal.azure.com/#create/Microsoft.PostgreSQLServer) se muestra el costo mensual en la pestaña **Plan de tarifa** según las opciones que seleccione. Si no tiene una suscripción de Azure, puede usar la calculadora de precios de Azure para obtener un precio estimado. En el sitio web [Calculadora de precios de Azure](https://azure.microsoft.com/pricing/calculator/), seleccione **Agregar elementos**, expanda la categoría **Bases de datos** y elija **Azure Database for PostgreSQL** para personalizar las opciones.

## <a name="next-steps"></a>Pasos siguientes

- Aprenda cómo [crear un servidor PostgreSQL en el portal](tutorial-design-database-using-azure-portal.md).
- Obtenga información acerca de los [límites de servicio](concepts-limits.md).
- Obtenga información sobre cómo [escalar horizontalmente con réplicas de lectura](howto-read-replicas-portal.md).
