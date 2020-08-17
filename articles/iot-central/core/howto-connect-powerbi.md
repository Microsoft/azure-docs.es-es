---
title: Visualización de los datos de Azure IoT Central en un panel de Power BI | Microsoft Docs
description: Use la solución de Power BI para Azure IoT Central a fin de visualizar y analizar los datos de IoT Central.
ms.service: iot-central
services: iot-central
author: viv-liu
ms.author: viviali
ms.date: 10/4/2019
ms.topic: conceptual
ms.openlocfilehash: f996bb2d5126ef038ca872aee1f1893979a5229b
ms.sourcegitcommit: 877491bd46921c11dd478bd25fc718ceee2dcc08
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/02/2020
ms.locfileid: "87081005"
---
# <a name="visualize-and-analyze-your-azure-iot-central-data-in-a-power-bi-dashboard"></a>Visualización y análisis de los datos de Azure IoT Central en un panel de Power BI

*Este tema se aplica a administradores y desarrolladores de soluciones.*

:::image type="content" source="media/howto-connect-powerbi/iot-continuous-data-export.png" alt-text="Canalización de las soluciones de Power BI":::

Use la solución de Power BI para Azure IoT Central V3 a fin de crear un panel eficaz de Power BI y supervisar el rendimiento de los dispositivos IoT. En el panel de Power BI, puede:

- Realizar un seguimiento de la cantidad de datos que se envían a lo largo del tiempo
- Comparar volúmenes de datos entre diferentes flujos de telemetría
- Filtrar por los datos enviados por dispositivos específicos
- Visualización de los datos de telemetría más recientes en una tabla

Esta solución configura una canalización que lee datos de la cuenta de Azure Blob Storage de la [Exportación de datos continua](howto-export-data-blob-storage.md). La canalización utiliza Azure Functions, Azure Data Factory y Azure SQL Database para procesar y transformar los datos. Puede visualizar y analizar los datos en un informe de Power BI que puede descargar como archivo PBIX. Todos los recursos se crean en la suscripción de Azure, para que pueda personalizar cada componente de acuerdo con sus necesidades.

## <a name="prerequisites"></a>Requisitos previos

Recuerde que para completar los pasos de esta guía paso a paso, necesita una suscripción activa a Azure. Si no tiene una suscripción a Azure, cree una [cuenta gratuita](https://azure.microsoft.com/free/?WT.mc_id=A261C142F) antes de empezar.

Para configurar la solución, son necesarios los recursos siguientes:

- Una versión 3 de la aplicación de IoT Central. Para obtener información sobre cómo comprobar la versión de la aplicación, vea [Acerca de la aplicación](./howto-get-app-info.md). Para obtener información sobre cómo crear una aplicación de IoT Central, vea [Creación de una aplicación de Azure IoT Central](./quick-deploy-iot-central.md).
- Exportación de datos continua configurada para exportar datos de telemetría, dispositivos y plantillas de dispositivo a Azure Blog Storage. Para obtener más información, consulte [Exportación de datos de IoT a destinos en Azure](howto-export-data.md).
  - Asegúrese de que solo la aplicación de IoT Central exporte datos al contenedor de blobs.
  - Los [dispositivos deben enviar mensajes codificados por JSON](../../iot-hub/iot-hub-devguide-messages-d2c.md). Los dispositivos deben especificar `contentType:application/JSON` y `contentEncoding:utf-8` o `contentEncoding:utf-16` o `contentEncoding:utf-32` en las propiedades del sistema de mensajes.
- Power BI Desktop (versión más reciente). Consulte [Descargas de Power BI](https://powerbi.microsoft.com/downloads/).
- Power BI Pro (si desea compartir el panel con otros usuarios).

> [!NOTE]
> Si usa una versión 2 de la aplicación de IoT Central, consulte [Visualización y análisis de los datos de Azure IoT Central en un panel de Power BI](https://docs.microsoft.com/previous-versions/azure/iot-central/core/howto-connect-powerbi) en el sitio de documentación de versiones anteriores.

## <a name="install"></a>Instalar

Para configurar la canalización, vaya a la página [Solución de Power BI para Azure IoT Central V3](https://appsource.microsoft.com/product/web-apps/iot-central.power-bi-solution-iot-central) en el sitio de **Microsoft AppSource**. Seleccione **Obtenerla ahora** y siga las instrucciones.

Al abrir el archivo PBIX, asegúrese de leer y seguir las instrucciones de la portada. En estas instrucciones se describe cómo conectar el informe a la base de datos SQL.

## <a name="report"></a>Informe

El archivo PBIX contiene el informe **Devices and Telemetry** (Dispositivos y telemetría), que muestra una vista histórica de los datos de telemetría enviados por los dispositivos. Proporciona un desglose de los distintos tipos de telemetría y también muestra los datos de telemetría más recientes enviados por los dispositivos.

:::image type="content" source="media/howto-connect-powerbi/report.png" alt-text="Informe Dispositivos y telemetría de Power BI":::

## <a name="pipeline-resources"></a>Recursos de canalización

Puede acceder a todos los recursos de Azure que componen la canalización en Azure Portal. Todos los recursos se encuentran en el grupo de recursos que creó al configurar la canalización.

:::image type="content" source="media/howto-connect-powerbi/azure-deployment.png" alt-text="Vista del grupo de recursos en Azure Portal":::

En la lista siguiente se describe el rol de cada recurso en la canalización:

### <a name="azure-functions"></a>Azure Functions

La aplicación de Azure Functions se activa cada vez IoT Central escribe un nuevo archivo en el almacenamiento de blobs. Las funciones extraen datos de los blobs de telemetría, dispositivos y plantillas de dispositivo para rellenar las tablas SQL intermedias que usa Azure Data Factory.

### <a name="azure-data-factory"></a>Azure Data Factory

Azure Data Factory se conecta a la SQL Database como servicio vinculado. Ejecuta los procedimientos almacenados para procesar los datos y los almacena en las tablas de análisis.

Azure Data Factory se ejecuta cada 15 minutos para transformar el último lote de datos que se va a cargar en las tablas SQL (que es el número mínimo actual del **desencadenador de la ventana de saltos de tamaño constante**).

### <a name="azure-sql-database"></a>Azure SQL Database

Azure Data Factory genera un conjunto de tablas de análisis para Power BI. Puede explorar estos esquemas en Power BI y usarlos para crear sus propias visualizaciones.

## <a name="estimated-costs"></a>Costos estimados

La [solución de Power BI para Azure IoT Central V3](https://appsource.microsoft.com/product/web-apps/iot-central.power-bi-solution-iot-central) en el sitio de Microsoft AppSource incluye un vínculo a un estimador de costos para los recursos que implemente.

## <a name="next-steps"></a>Pasos siguientes

Ahora que ha aprendido a usar visualizar los datos en Power BI, el siguiente paso que se recomienda es aprender a [administrar los dispositivos](howto-manage-devices.md).
