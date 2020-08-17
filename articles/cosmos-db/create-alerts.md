---
title: Creación de alertas para Azure Cosmos DB mediante Azure Monitor
description: Aprenda a configurar alertas para Azure Cosmos DB mediante Azure Monitor.
author: SnehaGunda
ms.author: sngun
ms.service: cosmos-db
ms.subservice: cosmosdb-sql
ms.devlang: dotnet
ms.topic: how-to
ms.date: 07/16/2020
ms.openlocfilehash: 84c2ad3a24d944db6a55f3f21e8a2a0c4084d033
ms.sourcegitcommit: 3d79f737ff34708b48dd2ae45100e2516af9ed78
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/23/2020
ms.locfileid: "87095589"
---
# <a name="create-alerts-for-azure-cosmos-db-using-azure-monitor"></a>Creación de alertas para Azure Cosmos DB mediante Azure Monitor

Las alertas se utilizan para configurar pruebas periódicas con el fin de supervisar la disponibilidad y la capacidad de respuesta de los recursos de Azure Cosmos DB. Las alertas pueden enviarle una notificación en forma de un correo electrónico o ejecutar una función de Azure cuando una de las métricas alcance el umbral o si se registra un evento específico en el registro de actividad.

Se pueden recibir alertas en función de las métricas o de los eventos del registro de actividad de la su cuenta de Azure Cosmos:

* **Métricas**: la alerta se desencadena cuando el valor de una métrica especificada cruza el umbral que se ha asignado. Por ejemplo, cuando el número total de unidades de solicitud consumidas supera las 1000 RU/s. La alerta se desencadena tanto la primera vez que se cumple la condición como después, cuando dicha condición deja de cumplirse. Consulte el artículo [Referencia de datos de supervisión](monitor-cosmos-db-reference.md#metrics) para ver las diferentes métricas disponibles en Azure Cosmos DB.

* **Eventos del registro de actividad**: esta alerta se desencadena cuando se produce un evento determinado. Por ejemplo, cuando se accede a las claves de la cuenta de Azure Cosmos o estas se actualizan.

Puede configurar alertas en el panel de Azure Cosmos DB o en el servicio Azure Monitor en Azure Portal. Ambas interfaces ofrecen las mismas opciones. En este artículo se muestra cómo configurar alertas para Azure Cosmos DB mediante Azure Monitor.

## <a name="create-an-alert-rule"></a>Crear una regla de alerta

En esta sección se muestra cómo crear una alerta cuando se recibe un código de estado HTTP 429, lo que sucede cuando la tasa de solicitudes está limitada. Por ejemplo, podría recibir una alerta cuando haya 100 o más solicitudes con tasa limitada. En este artículo se muestra cómo configurar una alerta para este escenario mediante el código de estado HTTP. También puede usar pasos similares para configurar otros tipos de alertas; solo tiene que elegir una condición diferente en función de sus requisitos.

1. Inicie sesión en [Azure Portal](https://portal.azure.com/).

1. Seleccione **Monitor** en la barra de navegación izquierda y, luego, elija **Alertas**.

1. Seleccione el botón Nueva regla de alertas para abrir el panel Crear regla de alertas.  

1. Rellene la sección **Ámbito**:

   * Abra el panel **Seleccionar recurso** y configure lo siguiente:

   * Elija el nombre de la **suscripción**.

   * Seleccione **Cuentas de Azure Cosmos DB** como **tipo de recurso**.

   * La **ubicación** de la cuenta de Azure Cosmos.

   * Después de rellenar los detalles, se muestra una lista de cuentas de Azure Cosmos en el ámbito seleccionado. Elija aquella para la que quiere configurar alertas y seleccione **Listo**.

1. Rellene la sección **Condición**:

   * Abra el panel **Seleccionar condición** para abrir la página **Configure signal logic** (Configurar lógica de señal) y configure la siguiente información:

   * Seleccione una señal. El **tipo de señal** puede ser una **métrica** o un **registro de actividad**. En este escenario, elija **Métricas**. El motivo es que quiere recibir una alerta cuando haya problemas de limitación de la tasa en la métrica de las unidades de solicitud totales.

   * Seleccione **Todas** en el **servicio de supervisión**

   * Elija un **nombre de señal**. Para recibir una alerta de códigos de estado HTTP, elija la señal **Total Request Units** (Unidades de solicitud totales).

   * En la pestaña siguiente, puede definir la lógica para desencadenar una alerta y usar el gráfico para ver las tendencias de la cuenta de Azure Cosmos. La métrica **Total Request Units** (Unidades de solicitud totales) admite dimensiones. Estas dimensiones le permiten filtrar por la métrica. Si no selecciona ninguna dimensión, se omite este valor.

   * Elija **StatusCode** como **nombre de dimensión**. Seleccione **Agregar valor personalizado** y establezca el código de estado en 429.

   * En **Lógica de alerta**, establezca **Umbral** en **Estático**. El umbral estático usa un valor de umbral definido por el usuario para evaluar la regla, mientras que los umbrales dinámicos usan algoritmos de aprendizaje automático integrados para aprender continuamente el patrón de comportamiento de la métrica y calcular los umbrales automáticamente.

   * Establezca el **operador** en **Mayor que**, **Tipo de agregación** en **Total** y **Valor del umbral** en **100**. Con esta lógica, si el cliente ve más de 100 solicitudes con un código de estado 429, se desencadena la alerta. También puede configurar el tipo de agregación, la granularidad de la agregación y la frecuencia de evaluación en función de sus requisitos.

   * Después de rellenar el formulario, seleccione **Listo**. En la captura de pantalla siguiente se muestran los detalles de la lógica de la alerta:

     :::image type="content" source="./media/create-alerts/configure-alert-logic.png" alt-text="Configuración de la lógica para recibir alertas correspondientes a solicitudes 429 con tasa limitada":::

1. Rellene la sección **Grupo de acciones**:

   * En el panel **Crear regla**, seleccione un grupo de acciones existente o cree uno. Un grupo de acciones le permite definir la acción que se va a realizar cuando se produce una condición de alerta. En este ejemplo, cree un grupo de acciones para recibir una notificación por correo electrónico cuando se desencadene la alerta. Abra el panel **Add action group** (Agregar grupo de acciones) y rellene los detalles siguientes:

   * **Nombre del grupo de acciones**: el nombre del grupo de acciones debe ser único dentro de un grupo de recursos.

   * **Nombre corto**: el nombre corto del grupo de acciones; este valor se incluye en las notificaciones por correo electrónico y SMS para identificar qué grupo de acciones fue el origen de la notificación.

   * Elija la suscripción y el grupo de recursos en el que se creará este grupo de acciones.  

   * Proporcione un nombre para la acción y seleccione **Email/SMS message/Push/Voice** (Correo electrónico, SMS, mensaje de inserción o voz) en **Tipo de acción**. En la captura de pantalla siguiente se muestran los detalles del tipo de acción:

     :::image type="content" source="./media/create-alerts/configure-alert-action-type.png" alt-text="Configuración del tipo de acción, por ejemplo, una notificación por correo electrónico para recibir la alerta":::

1. Rellene la sección **Alert rule details** (Detalles de la regla de alertas):

   * Defina un nombre para la regla, proporcione una descripción opcional, el nivel de gravedad de la regla, elija si desea habilitar la regla tras su creación y, luego, seleccione **Crear regla de alertas** para crear la regla de alertas de la métrica.

Después de crear la alerta, estará activa al cabo de 10 minutos.

## <a name="common-alerting-scenarios"></a>Escenarios comunes de uso de alertas

A continuación, se muestran algunos escenarios en los que puede usar alertas:

* Cuando se actualizan las claves de una cuenta de Azure Cosmos.
* Cuando el uso de datos o de índices de un contenedor, una base de datos o una región supera un número determinado de bytes.
* Cuando el consumo normalizado de RU/s es mayor que un porcentaje determinado. La métrica de consumo normalizado de RU proporciona el uso máximo de rendimiento dentro de un conjunto de réplicas. Para información, consulte el artículo [Cómo supervisar RU/s normalizadas](monitor-normalized-request-units.md).  
* Cuando se agrega, se quita o se queda sin conexión una región.
* Cuando se crea, elimina o actualiza una base de datos o un contenedor.
* Cuando cambia el rendimiento de la base de datos o del contenedor.

## <a name="next-steps"></a>Pasos siguientes

* Cómo [supervisar la métrica de RU/s normalizadas](monitor-normalized-request-units.md) en el contenedor de Azure Cosmos.
* Cómo [supervisar el rendimiento o el uso de unidades de solicitud](monitor-request-unit-usage.md) de una operación en Azure Cosmos DB.