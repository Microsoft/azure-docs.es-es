---
title: 'Inicio rápido: Portal web de Metrics Advisor'
titleSuffix: Azure Cognitive Services
description: Obtenga información sobre cómo empezar a usar el portal web de Metrics Advisor.
services: cognitive-services
author: aahill
manager: nitinme
ms.service: cognitive-services
ms.subservice: metrics-advisor
ms.topic: quickstart
ms.date: 09/14/2020
ms.author: aahi
ms.openlocfilehash: 0453ec9eac2b73b5372c143b23d6db98f65e38aa
ms.sourcegitcommit: 53acd9895a4a395efa6d7cd41d7f78e392b9cfbe
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/22/2020
ms.locfileid: "90946271"
---
# <a name="quickstart-monitor-your-first-metric-using-the-web-portal"></a>Inicio rápido: Supervisión de su primera métrica con el portal web

Al aprovisionar una instancia de Metrics Advisor, puede usar las API y el área de trabajo basada en web para trabajar con el servicio. El área de trabajo basada en web se puede usar para empezar a trabajar fácil y rápidamente con el servicio. También proporciona una forma visual de configurar valores, personalizar el modelo y realizar análisis de la causa principal. 

* Incorporación de los datos de métricas
* Vista de métricas y visualizaciones
* Ajuste preciso de las configuraciones de detección
* Exploración de información de diagnóstico
* Creación y suscripción a alertas de anomalías

## <a name="prerequisites"></a>Requisitos previos

* Una suscripción a Azure: [cree una cuenta gratuita](https://azure.microsoft.com/free/cognitive-services)
* Una vez que tenga la suscripción de Azure, <a href="https://go.microsoft.com/fwlink/?linkid=2142156"  title="cree un recurso de Metrics Advisor"  target="_blank">create a Metrics Advisor resource <span class="docon docon-navigate-external x-hidden-focus"></span></a> en Azure Portal para implementar la instancia de Metrics Advisor.  

    
> [!TIP]
> * Puede que se tarde entre 10 y 30 minutos en implementar el recurso de Metrics Advisor. Haga clic en **Ir al recurso** una vez que se implemente correctamente.
> * Si desea usar la API REST para interactuar con el servicio, necesitará la clave y el punto de conexión del recurso que cree. Puede encontrarlos en la pestaña **Keys and endpoints** (Claves y puntos de conexión) del recurso creado.

En este documento se usa una instancia de SQL Database como ejemplo para crear el primer monitor.

## <a name="onboard-time-series-data"></a>Incorporación de datos de serie temporal

Metrics Advisor proporciona conectores para diferentes orígenes de datos, como SQL Database, Azure Data Explorer y Azure Table Storage. Los pasos para la conexión de datos son similares para los diferentes conectores, aunque algunos parámetros de configuración pueden variar. Consulte cómo [conectar los datos de distintos orígenes](../data-feeds-from-different-sources.md) para los parámetros necesarios para orígenes de datos específicos.

En este inicio rápido se usa una instancia de SQL Database como ejemplo. También puede introducir sus propios datos siguiendo los mismos pasos.

Para empezar, inicie sesión en el área de trabajo de Metrics Advisor con su cuenta de Active Directory. En la página de aterrizaje, seleccione el **directorio**, la **suscripción** y el **área de trabajo** que acaba de crear y, a continuación, haga clic en **Get started** (Comenzar). Una vez que se cargue la página principal de la carga de trabajo, seleccione **Add data feed** (Agregar fuente de distribución de datos) en el menú de la izquierda.

### <a name="data-schema-requirements-and-configuration"></a>Configuración y requisitos del esquema de datos

[!INCLUDE [data schema requirements](../includes/data-schema-requirements.md)]

### <a name="configure-connection-settings"></a>Configuración de las opciones de conexión

> [!TIP]
> Consulte [cómo agregar fuentes de distribución de datos](../how-tos/onboard-your-data.md) para obtener más información sobre los parámetros disponibles.

Agregue la fuente de distribución de datos conectándose a su origen de datos de serie temporal. Para empezar, seleccione los parámetros siguientes:

* **Source Type** (Tipo de origen): tipo de origen de datos donde se almacenan los datos de serie temporal.
* **Granularity** (Granularidad): intervalo entre los puntos de datos consecutivos de los datos de serie temporal, por ejemplo, anualmente, mensualmente o diariamente. La personalización más baja del intervalo admitida es de 60 segundos.
* **Ingest data since (UTC)** (Ingesta de datos desde [UTC]): hora de inicio de la primera marca de tiempo que se va a ingerir. 


A continuación, especifique la **cadena de conexión** con las credenciales del origen de datos y una **consulta** personalizada. La consulta se usa para especificar los datos que se van a ingerir y convertirlos en el esquema requerido.

[!INCLUDE [query requirements](../includes/query-requirements.md)]

:::image type="content" source="../media/connection-settings.png" alt-text="Configuración de conexión" lightbox="../media/connection-settings.png":::


### <a name="verify-the-connection-and-load-the-data-schema"></a>Comprobación de la conexión y carga del esquema de datos

Después de crear la cadena de conexión y la cadena de consulta, seleccione **Verify and get schema** (Comprobar y obtener el esquema) para comprobar la conexión y ejecutar la consulta con el fin de obtener el esquema de datos del origen de datos. Normalmente, se tarda unos segundos en función de la conexión del origen de datos. Si se produce un error en este paso, confirme lo siguiente:

1. La cadena de conexión y la consulta son correctas.
2. La instancia de Metrics Advisor puede conectarse al origen de datos si hay una configuración de firewall.

### <a name="schema-configuration"></a>Configuración del esquema

Una vez que se cargue el esquema de datos y se muestre como se ve a continuación, seleccione los campos correspondientes.


|Selección  |Descripción  |Notas  |
|---------|---------|---------|
|**Timestamp**     | La marca de tiempo de un punto de datos. Si se omite, Metrics Advisor usará la marca de tiempo cuando se ingiera el punto de datos. Para cada fuente de distribución de datos, puede especificar como máximo una columna como una marca de tiempo.        | Opcional. Debe especificarse con una columna como máximo.       |
|**Medida**     |  Los valores numéricos de la fuente de distribución de datos. Para cada fuente de distribución de datos, puede especificar varias medidas, pero debe seleccionar al menos una columna como medida.        | Debe especificarse con al menos una columna.        |
|**Dimensión**     | Los valores de las categorías. Una combinación de valores diferentes identifica una serie temporal de una única dimensión determinada; por ejemplo: Country (País), Language (Idioma) y Tenant (Inquilino). Puede seleccionar un número arbitrario de columnas como dimensiones o ningún valor. Nota: Si selecciona una columna que no es de cadena como dimensión, tenga cuidado con la explosión de dimensiones. | Opcional.        |
|**Omitir**     | Omite la columna seleccionada.        |         |


:::image type="content" source="../media/schema-configuration.png" alt-text="Configuración del esquema" lightbox="../media/schema-configuration.png":::

### <a name="automatic-roll-up-settings"></a>Configuración de la acumulación automática

> [!IMPORTANT]
> Si desea habilitar el **análisis de la causa principal** y otras funcionalidades de diagnóstico, es necesario configurar los valores de acumulación automática. Una vez habilitado, no se puede cambiar la configuración de la acumulación automática.

Metrics Advisor puede realizar automáticamente la agregación (SUM, MAX, MIN...) en cada dimensión durante la ingesta y, a continuación, crear una jerarquía que se usará en el análisis de la causa principal y en otras características de diagnóstico. Consulte cómo [configurar la acumulación automática](../how-tos/onboard-your-data.md#automatic-roll-up-settings) para obtener más información.

Proporcione un nombre personalizado a la fuente de distribución de datos, que se mostrará en el área de trabajo. Haga clic en **Enviar**. 

## <a name="tune-detection-configuration"></a>Ajuste de la configuración de detección

Una vez agregada la fuente de distribución de datos, Metrics Advisor intentará ingerir datos de métricas desde la fecha de inicio especificada. Se tardará algún tiempo en que los datos se ingieran por completo. Puede ver el estado de la ingesta haciendo clic en **Ingestion progress** (Progreso de la ingesta) en la parte superior de la página de la fuente de distribución de datos. Si se ingieren datos, Metrics Advisor aplicará la detección y continuará supervisando el origen para detectar nuevos datos.

Cuando se aplique la detección, haga clic en una de las métricas que aparecen en la fuente de distribución de datos para ver la **página de detalles de la métrica**: 
- Vista de visualizaciones de todos los sectores de serie temporal en esta métrica
- Actualización de la configuración de detección para cumplir los resultados esperados
- Configuración de la notificación de anomalías detectadas

:::image type="content" source="../media/metric-details.png" alt-text="Detalles de la métrica" lightbox="../media/metric-details.png":::

## <a name="view-the-diagnostic-insights"></a>Visualización de la información de diagnóstico

Después de ajustar la configuración de detección, las anomalías encontradas deben reflejar las anomalías reales de los datos. Metrics Advisor realiza un análisis de métricas multidimensionales, como la agrupación de anomalías, la correlación de incidentes y el análisis de la causa principal. Use estas características para analizar y diagnosticar incidentes en los datos.

Para ver la información de diagnóstico, haga clic en los puntos rojos de las visualizaciones de la serie temporal, que representan las anomalías detectadas. Aparecerá una ventana con un vínculo a la página de análisis de incidentes. 

:::image type="content" source="../media/incident-link.png" alt-text="Vínculo del incidente" lightbox="../media/incident-link.png":::

Después de hacer clic en el vínculo, se mostrará la página de análisis de incidentes, en la que se analiza la anomalía correspondiente con abundante información de diagnóstico. En la parte superior, se muestran estadísticas sobre el incidente, como la **gravedad**, las **anomalías implicadas** y la **hora de inicio** y **hora de finalización** afectadas. 

A continuación, verá la anomalía antecesora del incidente y el consejo de la causa principal automatizado. Este consejo de la causa principal automatizado se genera al analizar el árbol de incidentes de todas las anomalías relacionadas, incluidas la desviación, distribución y contribución a las anomalías principales. 

:::image type="content" source="../media/incident-diagnostic.png" alt-text="Diagnóstico de incidentes" lightbox="../media/incident-diagnostic.png":::

Sobre esta base, ya puede obtener una vista clara de lo que está ocurriendo y del impacto del incidente, así como de la causa principal potencial. De este modo, podrá tomar medidas inmediatas para resolver el incidente lo antes posible. 

También puede desplazarse por distinta información de diagnóstico con características adicionales para explorar en profundidad las anomalías por dimensión, ver anomalías similares y realizar una comparación entre las métricas. Encontrará más información en [Cómo diagnosticar un incidente](../how-tos/diagnose-incident.md). 

## <a name="get-notified-when-new-anomalies-are-found"></a>Notificación cuando se encuentren nuevas anomalías

Si desea recibir una alerta cuando se detecte una anomalía en los datos, puede crear una suscripción para una o varias de sus métricas. Metrics Advisor usa enlaces para enviar alertas. Se admiten tres tipos de enlaces: enlace de correo electrónico, webhook y Azure DevOps. Aquí se usará un webhook como ejemplo. 

### <a name="create-a-web-hook"></a>Creación de un webhook

Un webhook es el punto de entrada para notificar anomalías mediante programación desde el servicio Metrics Advisor, que llama a una API proporcionada por el usuario cuando se desencadena una alerta. Para más información sobre cómo crear un enlace, consulte la sección **Creación de un enlace** en [Procedimiento: Configuración de alertas y obtención de notificaciones mediante un enlace](../how-tos/alerts.md#create-a-hook). 

### <a name="configure-alert-settings"></a>Configurar alertas

Después de crear un enlace, una configuración de alerta determina cómo y qué notificaciones de alerta deben enviarse. Puede establecer varias configuraciones de alerta para cada métrica. Dos configuraciones importantes son **Alert for** (Alerta para), que especifica las anomalías que se van a incluir, y **Filter anomaly options** (Filtrar opciones de anomalías), que define las anomalías que se incluirán en la alerta. Vea la sección **Adición o edición de la configuración de alertas** en [Procedimiento: Configuración de alertas y obtención de notificaciones mediante un enlace](../how-tos/alerts.md#add-or-edit-alert-settings) para obtener más información.


## <a name="next-steps"></a>Pasos siguientes

- [Incorporación de las fuentes de distribución de datos](../how-tos/onboard-your-data.md)
    - [Administración de las fuentes de distribución de datos](../how-tos/manage-data-feeds.md)
    - [Configuraciones para distintos orígenes de datos](../data-feeds-from-different-sources.md)
- [Uso de la API REST o las bibliotecas de cliente](rest-api.md)
- [Configuración de métricas y ajuste de la configuración de la detección](../how-tos/configure-metrics.md)