---
title: 'Aprovisionamiento y administración de un entorno Gen2: Azure Time Series | Microsoft Docs'
description: Aprenda a aprovisionar y administra un entorno de Azure Time Series Insights Gen2.
author: shipra1mishra
ms.author: shmishr
manager: diviso
ms.workload: big-data
ms.service: time-series-insights
services: time-series-insights
ms.topic: conceptual
ms.date: 07/08/2020
ms.custom: seodec18
ms.openlocfilehash: e71c426fee62187bb680c7b8bd1af92d87e85274
ms.sourcegitcommit: dccb85aed33d9251048024faf7ef23c94d695145
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/28/2020
ms.locfileid: "87288812"
---
# <a name="provision-and-manage-azure-time-series-insights-gen2"></a>Aprovisionamiento y administración de Azure Time Series Insights Gen2

En este artículo se describe cómo crear y administrar un entorno de Azure Time Series Insights Gen2 con [Azure Portal](https://portal.azure.com/).

## <a name="overview"></a>Información general

Al aprovisionar un entorno de Azure Time Series Insights Gen2, creará estos recursos de Azure:

* Un entorno de Azure Time Series Insights Gen2 que sigue el modelo de precios de pago por uso.
* Una cuenta de Azure Storage.
* Un almacenamiento intermedio opcional para consultas más rápidas e ilimitadas.

> [!TIP]
> * Aprenda [a planificar su entorno ](./time-series-insights-update-plan.md).
> * Obtenga información sobre cómo [agregar un origen de centro de eventos](./time-series-insights-how-to-add-an-event-source-eventhub.md) o cómo [agregar un origen de IoT Hub](./time-series-insights-how-to-add-an-event-source-iothub.md).

Aprenderá a:

1.  Asociar cada entorno de Azure Time Series Insights Gen2 con un origen del evento. También proporcionará una propiedad de identificador de marca de tiempo y un grupo de consumidores único para asegurarse de que el entorno tiene acceso a los eventos correspondientes.

1. Una vez completado el aprovisionamiento, puede modificar sus directivas de acceso y otros atributos del entorno para adaptarse a sus necesidades empresariales.

   > [!NOTE]
   > El primer paso es opcional al aprovisionar un entorno. Si omite este paso, debe adjuntar un origen del evento al entorno más adelante para que los datos puedan empezar a fluir a su entorno y se pueda tener acceso a ellos a través de la consulta.

## <a name="create-the-environment"></a>Creación del entorno

Para crear un entorno de Azure Time Series Insights Gen2:
1. Cree un recurso de Azure Time Series Insights en *Internet de las cosas* en [Azure Portal](https://portal.azure.com/).

1. Seleccione **Gen2(L1)** como el **Nivel**. Proporcione un nombre de entorno y elija el grupo de suscripciones y el grupo de recursos que se va a usar. A continuación, seleccione una ubicación compatible para hospedar el entorno.

   [![Cree una instancia de Azure Time Series Insights.](media/v2-update-manage/create-and-manage-configuration.png)](media/v2-update-manage/create-and-manage-configuration.png#lightbox)

1. Introduzca un id. de serie temporal.

    > [!NOTE]
    > * El identificador de serie temporal es *inmutable* y *distingue mayúsculas de minúsculas*. (Una vez establecido, no se puede cambiar).
    > * Los identificadores de serie temporal pueden tener hasta *tres* claves. Considérelo como una clave principal en una base de datos, que representa de forma única cada sensor de dispositivo que enviaría datos a su entorno. Puede ser una propiedad o una combinación de hasta tres propiedades.
    > * Más información sobre [cómo elegir un identificador de serie temporal](time-series-insights-update-how-to-id.md)

1. Para crear una cuenta de Azure Storage, seleccione un nombre de cuenta de almacenamiento, un tipo de cuenta y designe una opción de [replicación](https://docs.microsoft.com/azure/storage/common/redundancy-migration?tabs=portal). Al hacerlo, se crea automáticamente una cuenta de Azure Storage. De manera predeterminada, se creará la cuenta de [uso general v2](https://docs.microsoft.com/azure/storage/common/storage-account-overview). La cuenta se crea en la misma región que el entorno de Azure Time Series Insights Gen2 que seleccionó anteriormente. De manera alternativa, también puede traer su propio almacenamiento (BYOS) a través de una [plantilla de Resource Manager](./time-series-insights-manage-resources-using-azure-resource-manager-template.md) cuando cree un nuevo entorno de Azure Time Series Gen2. 

1. **(Opcional)** Habilite el almacenamiento intermedio para el entorno si quiere consultas más rápidas e ilimitadas sobre los datos más recientes del entorno. También puede crear o eliminar un almacenamiento intermedio a través de la opción **Configuración de almacenamiento** en el panel de navegación de la izquierda, después de crear un entorno de Azure Time Series Insights Gen2.

1. **(Opcional)** Ahora puede agregar un origen del evento. También puede esperar hasta que se haya aprovisionado la instancia.

   * Azure Time Series Insights admite [Azure IoT Hub](./time-series-insights-how-to-add-an-event-source-iothub.md) y [Azure Event Hubs](./time-series-insights-how-to-add-an-event-source-eventhub.md) como opciones de origen del evento. Si bien solo puede agregar un origen del evento único en el momento de crear el entorno, puede agregar otro origen del evento más adelante. 
   
     Puede seleccionar un grupo de consumidores existente o crear uno nuevo cuando agregue el origen del evento. Tenga en cuenta que el origen del evento requiere un único grupo de consumidores para que el entorno lea los datos en él.

   * Elija la propiedad Timestamp adecuada. De manera predeterminada, Azure Time Series Insights usa el tiempo de puesta en cola del mensaje para cada origen del evento.

     > [!TIP]
     > Es posible que el tiempo de puesta en cola del mensaje no sea la mejor opción de configuración para usar en escenarios de carga de datos históricos o de eventos por lotes. En tales casos, asegúrese de comprobar correctamente la decisión de usar o no usar una propiedad Timestamp.

     [![Pestaña Configuración de origen de eventos](media/v2-update-manage/create-and-manage-event-source.png)](media/v2-update-manage/create-and-manage-event-source.png#lightbox)

1. Confirme que el entorno se ha aprovisionado y configurado de la forma deseada.

    [![Pestaña Revisar y crear](media/v2-update-manage/create-and-manage-review-and-confirm.png)](media/v2-update-manage/create-and-manage-review-and-confirm.png#lightbox)

## <a name="manage-the-environment"></a>Administrar el entorno

Puede administrar su entorno de Azure Time Series Insights Gen2 con Azure Portal. Hay algunas diferencias clave entre un entorno de Gen2 y los entornos de Gen1 S1 o Gen2 S2 que debe tener en cuenta al administrar el entorno a través de Azure Portal:

* La hoja **Resumen** del entorno Gen2 de Azure Portal tiene los cambios siguientes:

  * La capacidad se quita porque no se aplica a los entornos Gen2.
  * Se agrega la propiedad **Identificador de serie temporal**. Determina cómo se particionan los datos.
  * Se eliminan los conjuntos de datos de referencia.
  * La dirección URL mostrada lo dirige al [Explorador de Azure Time Series Insights](./time-series-insights-update-explorer.md).
  * Su nombre de la cuenta de Azure Storage se agrega a la lista.

* Se quita la hoja **Configurar** de Azure Portal porque las unidades de escalado no se aplican a los entornos de Azure Time Series Insights Gen2. Sin embargo, puede usar **Configuración de almacenamiento** para configurar el almacenamiento intermedio recién incorporado.

* Se quita la hoja **Datos de referencia** de Azure Portal en Azure Time Series Insights Gen2 porque el concepto de los datos de referencia se ha reemplazado por el [modelo de serie temporal (TSM)](/azure/time-series-insights/concepts-model-overview).

[![Entorno de Azure Time Series Insights Gen2 en Azure Portal](media/v2-update-manage/create-and-manage-overview-confirm.png)](media/v2-update-manage/create-and-manage-overview-confirm.png#lightbox)

## <a name="next-steps"></a>Pasos siguientes

- Para más información sobre los entornos de Gen2 y de disponibilidad general de Azure Time Series Insights, lea [Planeamiento del entorno](./time-series-insights-update-plan.md).

- Obtenga información sobre cómo [agregar un origen de Event Hub](./time-series-insights-how-to-add-an-event-source-eventhub.md).

- Configure un [origen de centro de IoT](./time-series-insights-how-to-add-an-event-source-iothub.md).
