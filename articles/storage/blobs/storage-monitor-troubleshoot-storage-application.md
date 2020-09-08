---
title: Supervisión de una aplicación de almacenamiento en la nube en Azure y solucionar sus problemas | Microsoft Docs
description: Use herramientas de diagnóstico, métricas y alertas para solucionar problemas de una aplicación en la nube y supervisar esta.
author: normesta
ms.service: storage
ms.subservice: blobs
ms.topic: tutorial
ms.date: 07/20/2018
ms.author: normesta
ms.reviewer: fryu
ms.openlocfilehash: 8554a78112d197ef8174ac9d18147d301745165e
ms.sourcegitcommit: 62717591c3ab871365a783b7221851758f4ec9a4
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/22/2020
ms.locfileid: "83652159"
---
# <a name="monitor-and-troubleshoot-a-cloud-storage-application"></a>Supervisión de una aplicación de almacenamiento en la nube y solución de sus problemas

Este tutorial es la parte número cuatro y última de una serie. Aprenderá a supervisar una aplicación de almacenamiento en la nube y solucionar sus problemas.

En la parte número cuatro de la serie, se aprende a:

> [!div class="checklist"]
> * Activar el registro y las métricas
> * Habilitar las alertas para errores de autorización
> * Ejecutar el tráfico de prueba con tokens SAS incorrectos
> * Descargar y analizar registros

[Azure Storage Analytics](../common/storage-analytics.md) proporciona el registro y los datos de métricas para una cuenta de almacenamiento. Estos datos proporcionan información sobre el estado de la cuenta de almacenamiento. Para recopilar datos de análisis de Azure Storage, puede configurar el registro, las métricas y las alertas. Este proceso implica activar el registro, configurar las métricas y habilitar las alertas.

El registro y las métricas de cuentas de almacenamiento están habilitados en la pestaña **Diagnósticos** en Azure Portal. Los registros de almacenamiento permiten grabar en la cuenta de almacenamiento detalles de solicitudes correctas e incorrectas. Estos registros no solo le permiten ver detalles de lectura y escritura, sino que también le permitirán eliminar operaciones de tablas, colas y blobs de Azure. También le permiten ver los motivos de solicitudes con error, como los tiempos de espera, la limitación y los errores de autorización.

## <a name="log-in-to-the-azure-portal"></a>Iniciar sesión en Azure Portal

Inicie sesión en el [Azure Portal](https://portal.azure.com)

## <a name="turn-on-logging-and-metrics"></a>Activar el registro y las métricas

En el menú izquierdo, seleccione **Grupos de recursos**, seleccione **myResourceGroup** y luego seleccione la cuenta de almacenamiento en la lista de recursos.

En **Configuración de diagnóstico (clásica)** establezca **Estado** en **Activado**. Asegúrese de que todas las opciones que aparecen en **Propiedades del blob** estén habilitadas.

Cuando haya terminado, haga clic en **Guardar**.

![Panel Diagnósticos](media/storage-monitor-troubleshoot-storage-application/enable-diagnostics.png)

## <a name="enable-alerts"></a>Habilitación de alertas

Las alertas proporcionan una manera de enviar correos electrónicos a los administradores o de desencadenar un webhook basándose en una métrica de superación de umbral. En este ejemplo, se habilita una alerta para la métrica `SASClientOtherError`.

### <a name="navigate-to-the-storage-account-in-the-azure-portal"></a>Vaya a la cuenta de almacenamiento en Azure Portal.

En la sección **Supervisión**, seleccione **Alertas (clásico)** .

Seleccione **Agregar alerta de métrica (clásica)** y complete el formulario **Agregar regla** rellenando la información necesaria. En la lista desplegable **Métrica**, seleccione `SASClientOtherError`. Para permitir que la alerta se desencadene tras el primer error, en la lista desplegable **Condición** seleccione **Mayor o igual que**.

![Panel Diagnósticos](media/storage-monitor-troubleshoot-storage-application/add-alert-rule.png)

## <a name="simulate-an-error"></a>Simulación de un error

Para simular una alerta válida, puede intentar solicitar un blob inexistente en su cuenta de almacenamiento. El comando siguiente requiere un nombre de contenedor de almacenamiento. Puede utilizar el nombre de un contenedor existente o crear uno nuevo para este ejemplo.

Reemplace los marcadores de posición por valores reales (asegúrese de que `<INCORRECT_BLOB_NAME>` está establecido en un valor que no existe) y ejecute el comando.

```azurecli-interactive
sasToken=$(az storage blob generate-sas \
    --account-name <STORAGE_ACCOUNT_NAME> \
    --account-key <STORAGE_ACCOUNT_KEY> \
    --container-name <CONTAINER_NAME> \
    --name <INCORRECT_BLOB_NAME> \
    --permissions r \
    --expiry `date --date="next day" +%Y-%m-%d`)

curl https://<STORAGE_ACCOUNT_NAME>.blob.core.windows.net/<CONTAINER_NAME>/<INCORRECT_BLOB_NAME>?$sasToken
```

La siguiente imagen es una alerta de ejemplo que se basa en el error simulado ejecutado con el ejemplo anterior.

 ![Alerta de ejemplo](media/storage-monitor-troubleshoot-storage-application/email-alert.png)

## <a name="download-and-view-logs"></a>Descarga y vista de registros

Los registros de almacenamiento guardan datos en un conjunto de blobs en un contenedor de blobs denominado **$logs** en su cuenta de almacenamiento. Este contenedor no se muestra si enumera todos los contenedores de blobs de su cuenta, pero puede ver su contenido si se accede a él directamente.

En este escenario, se utiliza [Microsoft Message Analyzer](https://technet.microsoft.com/library/jj649776.aspx) para interactuar con su cuenta de almacenamiento de Azure.

### <a name="download-microsoft-message-analyzer"></a>Descarga de Microsoft Message Analyzer

Descargue [Microsoft Message Analyzer](https://docs.microsoft.com/message-analyzer/installing-and-upgrading-message-analyzer) e instale la aplicación.

Inicie la aplicación y elija **File (Archivo)**  > **Open (Abrir)**  > **From Other File Sources (Desde otros orígenes de archivo)** .

En el cuadro de diálogo **File Selector** (Selector de archivos), seleccione **+ Add Azure Connection** (Agregar conexión de Azure). Especifique el **nombre de cuenta de almacenamiento** y la **clave de cuenta** y haga clic en **Aceptar**.

![Microsoft Message Analyzer - Cuadro de diálogo Agregar conexión de almacenamiento de Azure](media/storage-monitor-troubleshoot-storage-application/figure3.png)

Cuando esté conectado, expanda los contenedores de la vista de árbol de almacenamiento para ver los blobs de registro. Seleccione el registro más reciente y haga clic en **Aceptar**.

![Microsoft Message Analyzer - Cuadro de diálogo Agregar conexión de almacenamiento de Azure](media/storage-monitor-troubleshoot-storage-application/figure4.png)

En el cuadro de diálogo **New Session** (Nueva sesión), haga clic en **Start** (Iniciar) para ver el registro.

Cuando el registro se abra, podrá ver los eventos de almacenamiento. Como puede ver en la siguiente imagen, se desencadenó `SASClientOtherError` en la cuenta de almacenamiento. Para información adicional sobre el registro de almacenamiento, visite [Azure Storage Analytics](../common/storage-analytics.md).

![Microsoft Message Analyzer - Visualización de eventos](media/storage-monitor-troubleshoot-storage-application/figure5.png)

[Explorador de Storage](https://azure.microsoft.com/features/storage-explorer/) es otra herramienta que se puede usar para interactuar con sus cuentas de almacenamiento, incluido el contenedor **$logs** y los registros contenidos en él.

## <a name="next-steps"></a>Pasos siguientes

En la parte número cuatro y última de la serie, aprendió a supervisar la cuenta de almacenamiento y solucionar sus problemas. Mas concretamente, aprendió a:

> [!div class="checklist"]
> * Activar el registro y las métricas
> * Habilitar las alertas para errores de autorización
> * Ejecutar el tráfico de prueba con tokens SAS incorrectos
> * Descargar y analizar registros

Siga este vínculo para ver ejemplos de almacenamiento previamente creados.

> [!div class="nextstepaction"]
> [Ejemplos de script de almacenamiento de Azure](storage-samples-blobs-cli.md)