---
title: Configuración de alertas de Metrics Advisor
titleSuffix: Azure Cognitive Services
description: En este artículo se explica cómo configurar las alertas de Metrics Advisor mediante enlaces de correo electrónico, webhooks y enlaces de Azure DevOps.
services: cognitive-services
author: aahill
manager: nitinme
ms.service: cognitive-services
ms.subservice: metrics-advisor
ms.topic: conceptual
ms.date: 09/14/2020
ms.author: aahi
ms.openlocfilehash: 870b412a1fa5d362ac7b5be1df6c92d95c2936ca
ms.sourcegitcommit: bdd5c76457b0f0504f4f679a316b959dcfabf1ef
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/22/2020
ms.locfileid: "90977783"
---
# <a name="how-to-configure-alerts-and-get-notifications-using-a-hook"></a>Procedimiento: Configuración de alertas y obtención de notificaciones mediante enlaces

Una vez que Metrics Advisor detecta una anomalía, se desencadena, mediante un enlace, una notificación de alerta basada en la configuración de alertas. Un valor de alerta se puede usar con varias configuraciones de detección; hay varios parámetros disponibles para personalizar la regla de alerta.

## <a name="create-a-hook"></a>Creación de un enlace

Metrics Advisor admite tres tipos diferentes de enlaces: enlaces de correo electrónico, webhooks y enlaces de Azure DevOps. Puede elegir el que mejor se adapte a su caso concreto.       

### <a name="email-hook"></a>Enlace de correo electrónico

> [!Note]
> Los administradores de recursos de Metrics Advisor deben configurar los valores de configuración del correo electrónico y especificar la información relacionada con SMTP en Metrics Advisor antes de poder enviar alertas de anomalías. El administrador del grupo de recursos o el administrador de la suscripción debe asignar al menos un rol de *administrador de Cognitive Services Metrics Advisor* en la pestaña control de acceso del recurso de Metrics Advisor. 

Para crear un enlace de correo electrónico, están disponibles los siguientes parámetros. 

Un enlace de correo electrónico es el canal por el que se envían las alertas de anomalías a las direcciones de correo electrónico especificadas en la sección **Email to** (Enviar correo electrónico a). Se enviarán dos tipos de correos electrónicos de alerta. Las alertas de *fuentes de distribución de datos no disponibles* y los *informes de incidentes* que contienen una o varias anomalías. 

|Parámetro |Descripción  |
|---------|---------|
| Nombre | Nombre del enlace de correo electrónico |
| Email to (Enviar correo electrónico a)| Direcciones de correo electrónico a las que se enviarían las alertas|
| Vínculo externo | Campo opcional que habilita una redirección personalizada; por ejemplo, para las notas de solución de problemas. |
| Título personalizado de la alerta de anomalía | La plantilla de título admite `${severity}`, `${alertSettingName}`, `${datafeedName}`, `${metricName}`, `${detectConfigName}`, `${timestamp}`, `${topDimension}`, `${incidentCount}` y `${anomalyCount}`.

Después de hacer clic en **OK** (Aceptar), se creará un enlace de correo electrónico. Puede usarlo en cualquier configuración de alertas para recibir alertas de anomalías. 

### <a name="web-hook"></a>Webhook

> [!Note]
> * Use el método de solicitud **POST**.
> * El cuerpo de la solicitud será similar a lo siguiente:  
    `{"timestamp":"2019-09-11T00:00:00Z","alertSettingGuid":"49635104-1234-4c1c-b94a-744fc920a9eb"}`
> * Cuando se crea o modifica un webhook, se llama a la API como una prueba con un cuerpo de solicitud vacío. La API debe devolver un código HTTP 200.

Un webhook es el punto de entrada de toda la información disponible en el servicio Metrics Advisor y llama a una API proporcionada por el usuario cuando se desencadena una alerta. Todas las alertas se pueden enviar a través de un webhook.

Para crear un webhook, tendrá que agregar la siguiente información:

|Parámetro |Descripción  |
|---------|---------|
|Punto de conexión     | La dirección de la API a la que se llamará cuando se desencadene una alerta.        |
|Nombre de usuario y contraseña | Para autenticarse en la dirección de la API. Déjelo en blanco si no se necesita autenticación.         |
|Encabezado     | Los encabezados personalizados de la llamada API.        |

:::image type="content" source="../media/alerts/create-web-hook.png" alt-text="Ventana de creación de webhooks":::

Cuando se envía una notificación a través de un webhook, puede usar las siguientes API para obtener los detalles de la alerta. Establezca los valores de *timestamp* y *alertSettingGuid* en el servicio de API al que se enviarán y, a continuación, use las siguientes consultas: 
- `query_alert_result_anomalies`
- `query_alert_result_incidents`

### <a name="azure-devops"></a>Azure DevOps

Metrics Advisor también permite crear automáticamente un elemento de trabajo en Azure DevOps para realizar un seguimiento de los problemas o errores cuando se detecta alguna anomalía. Todas las alertas se pueden enviar a través de enlaces de Azure DevOps.

Para crear un enlace de Azure DevOps, deberá agregar la siguiente información:

|Parámetro |Descripción  |
|---------|---------|
| Nombre | Un nombre para el enlace. |
| Organización | La organización a la que pertenece su instancia de Azure DevOps. |
| Project | El proyecto específico en Azure DevOps. |
| Token de acceso |  Un token para autenticarse en DevOps. | 

> [!Note]
> Debe conceder permisos de escritura si desea que Metrics Advisor cree elementos de trabajo basados en alertas de anomalías. Después de crear enlaces, puede usarlos en cualquiera de las configuraciones de alertas. Administre los enlaces en la página **hook settings** (configuración de enlaces).

## <a name="add-or-edit-alert-settings"></a>Adición o edición de la configuración de alertas

Vaya a la página de detalles de métricas para buscar la sección **Alert settings** (Configuración de alertas), que se encuentra en la esquina inferior izquierda de la página de detalles de métricas. Se muestran todos los valores de configuración de alertas que se aplican a la configuración de detección seleccionada. Cuando se crea una configuración de detección, no hay ningún valor de alerta y, por tanto, no se envía ninguna alerta.  
Puede usar los iconos de **add** (agregar), **edit** (editar) y **delete** (eliminar) para modificar la configuración de alertas.

:::image type="content" source="../media/alerts/alert-setting.png" alt-text="Elemento de menú configuración de alertas":::

Haga clic en los botones **add** (agregar) o **edit** (editar) para que aparezca una ventana para agregar o editar la configuración de alertas.

:::image type="content" source="../media/alerts/edit-alert.png" alt-text="Adición o edición de la configuración de alertas":::

**Alert setting name** (Nombre del valor de alerta): El nombre de este valor de alerta. Se mostrará en el título del correo electrónico de alerta.

**Hooks** (Enlaces): La lista de enlaces a los que se van a enviar alertas.

La sección marcada en la captura de pantalla anterior son los valores de configuración de una configuración de detección. Puede establecer diferentes valores de configuración de alertas para diferentes configuraciones de detección. Elija la configuración de destino en la lista desplegable de esta ventana. 

### <a name="filter-settings"></a>Configuración de filtros 

A continuación se muestran los valores de configuración de filtro de una configuración de detección.

**Alert For** (Alerta para) tiene cuatro opciones para filtrar anomalías.

* **Anomalies in all series** (Anomalías en todas las series): Todas las anomalías se incluirán en la alerta.         
* **Anomalies in the series group** (Anomalías en el grupo de series): Se filtran las series por valores de dimensión. Establezca valores específicos para algunas dimensiones. Las anomalías solo se incluirán en la alerta cuando la serie coincida con el valor especificado.       
* **Anomalies in favorite series** (Anamolías en series favoritas): En la alerta solo se incluirán las series marcadas como favoritas.        |
* **Anomalies in top N of all series** (Anomalías en las N series principales de todas las series): Este filtro se usa en caso de que solo le interese la serie cuyo valor se encuentra en las N principales. Se devolverán algunas marcas de tiempo y se comprobará si el valor de las series de esta marca de tiempo se encuentra en las N principales. Si el recuento correspondiente a "en las N principales" es mayor que el número especificado, la anomalía se incluirá en una alerta.        |

**Filter anomaly options** (Filtrar opciones de anomalías) es un filtro adicional con las siguientes opciones:

- **severity** (gravedad): La anomalía solo se incluirá cuando la gravedad de la anomalía esté dentro del intervalo especificado.
- **Snooze** (Posponer): Detenga temporalmente las alertas de anomalías en los N siguientes puntos (período) cuando se desencadenen en una alerta.
    - **snooze type** (tipo de aplazamiento): Cuando se establece en **Series**, una anomalía desencadenada solo pospondrá su serie. En el caso de **Metric** (Métrica), una anomalía desencadenada pospondrá todas las series de esta métrica.
    - **snooze number** (posponer número): el número de puntos (período) que se van a posponer.
    - **reset for non-successive** (restablecer para las no sucesivas): Cuando se selecciona, una anomalía desencadenada solo pospondrá las siguientes N anomalías sucesivas. Si uno de los siguientes puntos de datos no es una anomalía, el aplazamiento se restablecerá desde ese punto; si no se selecciona, una anomalía desencadenada pospondrá los próximos N puntos (período), incluso si los puntos de datos sucesivos no son anomalías.
- **value** (valor) (opcional): Se filtra por valor. La anomalía se incluirá solo si los valores de puntos cumplen la condición. Si usa el valor correspondiente de otra métrica, los nombres de dimensión de las dos métricas deberían ser coherentes.

Las anomalías no filtradas se enviarán en una alerta.

### <a name="add-cross-metric-settings"></a>Adición de valores de configuración entre métricas

Haga clic en **+ Add cross-metric settings** (+ Agregar valores de configuración entre métricas) en la página de valores de configuración de alertas para agregar otra sección.

El selector **Operator** (Operador) es la relación lógica de cada sección, para determinar si envían una alerta.


|Operador  |Descripción  |
|---------|---------|
|y     | Solo se envía una alerta si una serie coincide con cada sección de alerta y todos los puntos de datos son anomalías. Si las métricas tienen diferentes nombres de dimensión, nunca se desencadenará una alerta.         |
|O BIEN     | Envíe la alerta si al menos una sección contiene anomalías.         |

:::image type="content" source="../media/alerts/alert-setting-operator.png" alt-text="Operador para varias secciones de configuración de alertas":::

## <a name="next-steps"></a>Pasos siguientes

- [Ajuste de la detección de anomalías mediante comentarios](anomaly-feedback.md)
- [Diagnóstico de incidentes](diagnose-incident.md)
- [Configuración de métricas y ajuste de la configuración de la detección](configure-metrics.md)