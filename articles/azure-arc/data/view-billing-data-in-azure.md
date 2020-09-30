---
title: Carga de datos de facturación en Azure y visualización en Azure Portal
description: Carga de datos de facturación en Azure y visualización en Azure Portal
services: azure-arc
ms.service: azure-arc
ms.subservice: azure-arc-data
author: twright-msft
ms.author: twright
ms.reviewer: mikeray
ms.date: 09/22/2020
ms.topic: how-to
ms.openlocfilehash: 375191bbe6f45189fba50ea927454c0ec4f64678
ms.sourcegitcommit: 53acd9895a4a395efa6d7cd41d7f78e392b9cfbe
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/22/2020
ms.locfileid: "90932222"
---
# <a name="upload-billing-data-to-azure-and-view-it-in-the-azure-portal"></a>Carga de datos de facturación en Azure y visualización en Azure Portal

> [!IMPORTANT] 
>  El uso de los servicios de datos habilitados para Azure Arc durante el período de versión preliminar no supone ningún coste. Aunque el sistema de facturación funciona de extremo a extremo, el medidor de facturación se establece en 0 USD.  Si sigue este escenario, verá entradas en la facturación para un servicio denominado actualmente **servicios de datos híbridos** y para recursos de un tipo denominado **microsoft.AzureData/`<resource type>`** . Podrá ver un registro para cada servicio de datos de Azure Arc que cree, pero cada registro se facturará por 0 USD.

[!INCLUDE [azure-arc-data-preview](../../../includes/azure-arc-data-preview.md)]

## <a name="connectivity-modes---implications-for-billing-data"></a>Modos de conectividad: implicaciones de los datos de facturación

En el futuro, habrá dos modos de ejecución de los servicios de datos habilitados para Azure Arc:

- **Conexión indirecta**: no hay ninguna conexión directa con Azure. Los datos solo se envían a Azure a través de un proceso de exportación y carga. Actualmente, todas las implementaciones de servicios de datos de Azure Arc funcionan en este modo en versión preliminar.
- **Conexión directa**: en este modo, habrá una dependencia del servicio Kubernetes habilitado para Azure Arc a fin de proporcionar una conexión directa entre Azure y el clúster de Kubernetes en el que se ejecutan los servicios de datos habilitados para Azure Arc. Esto habilitará más funcionalidades y también le permitirá usar Azure Portal y la CLI de Azure para administrar los servicios de datos habilitados para Azure Arc igual que los servicios de datos en PaaS de Azure.  Este modo de conectividad todavía no está disponible en versión preliminar, pero lo estará próximamente.

Puede obtener más información sobre la diferencia entre los [modos de conectividad](/docs/connectivity.md).

En el modo de conexión indirecta, los datos de facturación se exportan periódicamente desde el controlador de datos de Azure Arc a un archivo seguro y, después, se cargan en Azure y se procesan.  En el próximo modo de conexión directa, los datos de facturación se enviarán automáticamente a Azure aproximadamente cada hora para ofrecer una visión casi en tiempo real de los costes de los servicios. El proceso de exportación y carga de los datos en el modo de conexión indirecta también se puede automatizar mediante scripts, o bien se puede crear un servicio que lo haga automáticamente.

## <a name="upload-billing-data-to-azure"></a>Carga de datos de facturación en Azure

Para cargar los datos de facturación en Azure, primero debe ocurrir lo siguiente:

1. Cree un servicio de datos habilitado para Azure Arc si todavía no tiene uno. Por ejemplo, cree uno de los siguientes:
   - [Creación de una Instancia administrada de Azure SQL en Azure Arc](create-sql-managed-instance.md)
   - [Creación de un grupo de servidores Hiperescala de PostgreSQL habilitado para Azure Arc](create-postgresql-hyperscale-server-group.md)
1. [Cargue el inventario de recursos, los datos de uso, las métricas y los registros en Azure Monitor](upload-metrics-and-logs-to-azure-monitor.md) si aún no lo ha hecho.
1. Espere al menos dos horas desde la creación del servicio de datos para que el proceso de recopilación de telemetría de facturación pueda recopilar algunos datos de facturación.

Ejecute el comando siguiente para exportar los datos de facturación:

```console
azdata arc dc export -t usage -p usage.json
```

Por ahora, el archivo no está cifrado para que pueda ver el contenido. No dude en abrirlo en un editor de texto y ver el aspecto del contenido.

Observará que hay dos conjuntos de datos: `resources` y `data`. `resources` son el controlador de datos, los grupos de servidores Hiperescala de PostgreSQL y las instancias administradas de SQL. Los registros `resources` de los datos capturan los eventos pertinentes en el historial de un recurso: cuándo se ha creado, actualizado y eliminado. Los registros `data` capturan el número de núcleos disponibles para que los use una instancia determinada cada hora.

Ejemplo de una entrada de `resource`:

```console
    {
        "customObjectName": "<resource type>-2020-29-5-23-13-17-164711",
        "uid": "4bc3dc6b-9148-4c7a-b7dc-01afc1ef5373",
        "instanceName": "sqlInstance001",
        "instanceNamespace": "arc",
        "instanceType": "<resource>",
        "location": "eastus",
        "resourceGroupName": "production-resources",
        "subscriptionId": "482c901a-129a-4f5d-86e3-cc6b294590b2",
        "isDeleted": false,
        "externalEndpoint": "32.191.39.83:1433",
        "vCores": "2",
        "createTimestamp": "05/29/2020 23:13:17",
        "updateTimestamp": "05/29/2020 23:13:17"
    }
```

Ejemplo de una entrada de `data`:

```console
        {
          "requestType": "usageUpload",
          "clusterId": "4b0917dd-e003-480e-ae74-1a8bb5e36b5d",
          "name": "DataControllerTestName",
          "subscriptionId": "482c901a-129a-4f5d-86e3-cc6b294590b2",
          "resourceGroup": "production-resources",
          "location": "eastus",
          "uploadRequest": {
            "exportType": "usages",
            "dataTimestamp": "2020-06-17T22:32:24Z",
            "data": "[{\"name\":\"sqlInstance001\",
                       \"namespace\":\"arc\",
                       \"type\":\"<resource type>\",
                       \"eventSequence\":1, 
                       \"eventId\":\"50DF90E8-FC2C-4BBF-B245-CB20DC97FF24\",
                       \"startTime\":\"2020-06-17T19:11:47.7533333\",
                       \"endTime\":\"2020-06-17T19:59:00\",
                       \"quantity\":1,
                       \"id\":\"4BC3DC6B-9148-4C7A-B7DC-01AFC1EF5373\"}]",
           "signature":"MIIE7gYJKoZIhvcNAQ...2xXqkK"
          }
        }
```

Ejecute el comando siguiente para cargar el archivo usage.json en Azure:

```console
azdata arc dc upload -p usage.json
```

## <a name="view-billing-data-in-azure-portal"></a>Visualización de los datos de facturación en Azure Portal

Siga estos pasos para ver los datos de facturación en Azure Portal:

1. Abra Azure Portal mediante la dirección URL especial [https://aka.ms/arcdata](https://aka.ms/arcdata).
1. En el cuadro de búsqueda de la parte superior de la pantalla, escriba **Administración de costos** y haga clic en el servicio Administración de costos.
1. Haga clic en la pestaña **Análisis de costos** de la izquierda.
1. Haga clic en el botón **Coste por recurso** de la parte superior de la vista.
1. Asegúrese de que el ámbito está establecido en la suscripción en la que se hayan creado los recursos del servicio de datos.
1. Seleccione **Coste por recurso** en el menú desplegable Vista situado junto al selector de ámbito cerca de la parte superior de la vista.
1. Asegúrese de que el filtro de fecha está establecido en **Este mes** o en algún otro intervalo de tiempo que tenga sentido en función del momento en el que haya creado los recursos del servicio de datos.
1. Haga clic en **Agregar filtro** para agregar un filtro por **Tipo de recurso** = `microsoft.azuredata/<data service type>` si solo quiere filtrar por un tipo de servicio de datos habilitado para Azure Arc.
1. Ahora verá una lista de todos los recursos que se han creado y cargado en Azure. Como el medidor de facturación es 0 USD, verá que el coste siempre es 0 USD.

## <a name="download-billing-data"></a>Descarga de los datos de facturación

Puede descargar los datos de resumen de facturación directamente desde Azure Portal.

1. En la misma vista **Análisis de costos > Ver por tipo de recurso** a la que ha llegado después de seguir las instrucciones anteriores, haga clic en el botón Descargar situado junto a la parte superior.
1. Elija el tipo de archivo de descarga, Excel o CSV, y haga clic en el botón **Descargar datos**.
1. Abra el archivo en un editor adecuado para el tipo de archivo seleccionado.

## <a name="export-billing-data"></a>Exportación de los datos de facturación

También puede exportar de forma periódica datos de facturación y uso **detallados** a un contenedor de Azure Storage mediante la creación de un trabajo de exportación de facturación. Esto resulta útil si quiere ver los detalles de la facturación, como el número de horas que se ha facturado una instancia determinada durante el período de facturación.

Siga estos pasos para configurar un trabajo de exportación de facturación:

1. Haga clic en Exportaciones en la parte izquierda.
1. Haga clic en Agregar.
1. Escriba un nombre y una frecuencia de exportación, y haga clic en Siguiente.
1. Elija entre crear una cuenta de almacenamiento, o bien crearla y rellenar el formulario para especificar la cuenta de almacenamiento, el contenedor y la ruta de acceso al directorio para exportar los archivos de datos de facturación y haga clic en Siguiente.
1. Haga clic en Crear.

Los archivos de exportación de datos de facturación estarán disponibles aproximadamente en cuatro horas y se exportarán según la programación que haya especificado al crear el trabajo de exportación de facturación.

Siga estos pasos para ver los archivos de datos de facturación que se exportan:

Puede validar los archivos de datos de facturación en Azure Portal. 

> [!IMPORTANT]
> Después de crear el trabajo de exportación de facturación, espere cuatro horas antes de continuar con los pasos siguientes.

1. En el cuadro de búsqueda de la parte superior del portal, escriba **Cuentas de almacenamiento** y haga clic en **Cuentas de almacenamiento**.
3. Haga clic en la cuenta de almacenamiento que haya especificado al crear el trabajo de exportación de facturación anterior.
4. Haga clic en Contenedores en la parte izquierda.
5. Haga clic en el contenedor que haya especificado al crear el trabajo de exportación de facturación anterior.
6. Haga clic en la carpeta que haya especificado al crear el trabajo de exportación de facturación anterior.
7. Explore en profundidad los archivos y carpetas generados. y haga clic en uno de los archivos .csv generados.
8. Haga clic en el botón Descargar, para guardar el archivo en la carpeta Descargas local.
9. Abra el archivo con un visor de archivos .csv como Excel.
10. Filtre los resultados para mostrar solo las filas con el **Tipo de recurso** = `Microsoft.AzureData/<data service resource type`.
11. En la columna UsageQuantity verá el número de horas que se ha usado la instancia en el período de 24 horas actual.
