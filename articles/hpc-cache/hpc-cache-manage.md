---
title: Administración y actualización de Azure HPC Cache
description: Administración y actualización de Azure HPC Cache mediante Azure Portal o la CLI de Azure
author: ekpgh
ms.service: hpc-cache
ms.topic: how-to
ms.date: 07/08/2020
ms.author: v-erkel
ms.openlocfilehash: 66b084cca3d1cd54362a538423988755a3d31ced
ms.sourcegitcommit: 3543d3b4f6c6f496d22ea5f97d8cd2700ac9a481
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/20/2020
ms.locfileid: "86497237"
---
# <a name="manage-your-cache"></a>Administración de la caché

En la página de información general de caché de Azure Portal se muestran los detalles del proyecto, el estado de la memoria caché y las estadísticas básicas de dicha memoria. También contiene controles para detener o iniciar la caché, eliminar la caché, vaciar los datos para el almacenamiento a largo plazo y actualizar el software.

En este artículo también se explica cómo realizar estas tareas básicas con la CLI de Azure.

Para abrir la página de información general, seleccione el recurso de caché en Azure Portal. Por ejemplo, cargue la página **Todos los recursos** y haga clic en el nombre de la memoria caché.

![Captura de pantalla de la página Información general de una instancia de Azure HPC Cache](media/hpc-cache-overview.png)

Los botones que se encuentran en la parte superior de la página pueden ayudarlo a administrar la memoria caché:

* **Iniciar** y [**Detener**](#stop-the-cache): reanuda o suspende la operación de caché
* [**Vaciar**](#flush-cached-data): escribe los datos modificados en destinos de almacenamiento
* [**Actualizar**](#upgrade-cache-software): actualiza el software de caché
* **Refrescar**: vuelve a cargar la página de información general
* [**Eliminar**](#delete-the-cache): destruye permanentemente la memoria caché

Lea más información sobre estas opciones a continuación.

Haga clic en la imagen siguiente para ver un [vídeo](https://azure.microsoft.com/resources/videos/managing-hpc-cache/) que muestra las tareas de administración de caché.

[![Miniatura de vídeo: Azure HPC Cache: Administración (haga clic para visitar la página del vídeo)](media/video-5-manage.png)](https://azure.microsoft.com/resources/videos/managing-hpc-cache/)

## <a name="stop-the-cache"></a>Detención de la caché

Puede detener la caché para reducir los costos durante un período inactivo. Mientras la caché está detenida no se le cobra por el tiempo de actividad, sino por el almacenamiento en disco asignado de la caché. (Para más información, consulte la [página de precios](https://aka.ms/hpc-cache-pricing)).

Una caché detenida no responde a las solicitudes de los clientes. Antes de detener la caché, debe desmontar los clientes.

### <a name="portal"></a>[Portal](#tab/azure-portal)

El botón **Detener** suspende una caché activa. El botón **Detener** está disponible cuando el estado de una caché es **Correcto** o **Degradado**.

![captura de pantalla de los botones superiores con el botón Detener resaltado y un mensaje emergente que describe la acción de detención y pregunta si desea continuar, y los botones Sí (valor predeterminado) y No.](media/stop-cache.png)

Después de hacer clic en Sí para confirmar la detención de la caché, esta vacía automáticamente su contenido en los destinos de almacenamiento. Este proceso puede tardar algún tiempo, pero garantiza la coherencia de los datos. Por último, el estado de la caché cambia a **Detenido**.

Para reactivar una caché detenida, haga clic en el botón **Iniciar**. No es necesario realizar ninguna confirmación.

![captura de pantalla de los botones principales con Iniciar resaltado](media/start-cache.png)

### <a name="azure-cli"></a>[CLI de Azure](#tab/azure-cli)

[!INCLUDE [cli-reminder.md](includes/cli-reminder.md)]

Suspenda temporalmente una memoria caché con el comando [az hpc-cache stop](/cli/azure/ext/hpc-cache/hpc-cache#ext-hpc-cache-az-hpc-cache-stop). Esta acción solo es válida cuando el estado de la caché es **Correcto** o **Degradado**.

La caché vacía automáticamente su contenido en los destinos de almacenamiento antes de detenerse. Este proceso puede tardar algún tiempo, pero garantiza la coherencia de los datos.

Una vez completada la acción, el estado de la caché cambia a **Detenido**.

Vuelva a activar una caché detenida con el comando [az hpc-cache start](/cli/azure/ext/hpc-cache/hpc-cache#ext-hpc-cache-az-hpc-cache-start).

Cuando se realiza el comando para iniciar o detener, la línea de comandos muestra el mensaje de estado "En ejecución" hasta que la operación se completa.

```azurecli
$ az hpc-cache start --name doc-cache0629
 - Running ..
```

Al finalizar, el mensaje se actualiza a "Finalizada" y muestra los códigos de retorno y otra información.

```azurecli
$ az hpc-cache start --name doc-cache0629
{- Finished ..
  "endTime": "2020-07-01T18:46:43.6862478+00:00",
  "name": "c48d320f-f5f5-40ab-8b25-0ac065984f62",
  "properties": {
    "output": "success"
  },
  "startTime": "2020-07-01T18:40:28.5468983+00:00",
  "status": "Succeeded"
}
```

---

## <a name="flush-cached-data"></a>Vaciado de los datos en caché

El botón **Vaciar** de la página de información general indica a la memoria caché que escriba inmediatamente todos los datos modificados que están almacenados en la memoria caché en los destinos de almacenamiento de back-end. La memoria caché guarda regularmente los datos en los destinos de almacenamiento, por lo que no es necesario hacerlo manualmente a menos que desee asegurarse de que el sistema de almacenamiento de back-end esté actualizado. Por ejemplo, puede usar **Vaciar** antes de tomar una instantánea de almacenamiento o comprobar el tamaño del conjunto de datos.

> [!NOTE]
> Durante el proceso de vaciado, la memoria caché no puede atender las solicitudes del cliente. El acceso a la memoria caché se suspende y se reanuda una vez finalizada la operación.

Cuando se inicia la operación de vaciado de la memoria caché, esta deja de aceptar solicitudes de cliente y su estado de en la página de información general cambia a **Vaciado**.

Los datos de la memoria caché se guardan en los destinos de almacenamiento adecuados. En función de la cantidad de datos que se deban vaciar, el proceso puede tardar unos minutos o más de una hora.

Una vez que todos los datos se guardan en destinos de almacenamiento, la memoria caché empieza de nuevo a tomar las solicitudes de cliente. El estado de la memoria caché vuelve a **Correcto**.

### <a name="portal"></a>[Portal](#tab/azure-portal)

Para vaciar la memoria caché, haga clic en el botón **Vaciar** y, a continuación, haga clic en **Sí** para confirmar la acción.

![Captura de pantalla de los botones superiores con el botón Vaciar resaltado, un mensaje emergente que describe la acción de vaciado y pregunta si desea continuar, y los botones Sí (valor predeterminado) y No.](media/hpc-cache-flush.png)

### <a name="azure-cli"></a>[CLI de Azure](#tab/azure-cli)

[!INCLUDE [cli-reminder.md](includes/cli-reminder.md)]

Use [az hpc-cache flush](/cli/azure/ext/hpc-cache/hpc-cache#ext-hpc-cache-az-hpc-cache-flush) para forzar a la memoria caché a escribir todos los datos modificados en los destinos de almacenamiento.

Ejemplo:

```azurecli
$ az hpc-cache flush --name doc-cache0629 --resource-group doc-rg
 - Running ..
```

Cuando finalice el vaciado, se devolverá un mensaje de operación correcta.

```azurecli
{- Finished ..
  "endTime": "2020-07-09T17:26:13.9371983+00:00",
  "name": "c22f8e12-fcf0-49e5-b897-6a6e579b6489",
  "properties": {
    "output": "success"
  },
  "startTime": "2020-07-09T17:25:21.4278297+00:00",
  "status": "Succeeded"
}
$
```

---

## <a name="upgrade-cache-software"></a>Actualización del software de caché

Si hay disponible una nueva versión de software, se activa el botón **Actualizar**. También puede ver un mensaje en la parte superior de la página sobre la actualización del software.

![Captura de pantalla de la fila superior de botones con el botón Actualizar habilitado](media/hpc-cache-upgrade-button.png)

El acceso de cliente no se interrumpe durante una actualización de software, pero el rendimiento de la memoria caché se ralentiza. Planee la actualización de software durante horas de menor uso o en un período de mantenimiento planeado.

La actualización de software puede tardar varias horas. Las memorias caché configuradas con un mayor rendimiento tardan más tiempo en actualizarse que las que tienen valores de rendimiento de pico más pequeños.

Cuando haya una actualización de software disponible, tendrá una semana aproximadamente para aplicarla manualmente. La fecha de finalización aparece en el mensaje de actualización. Si no actualiza durante ese tiempo, Azure aplica automáticamente la actualización a la memoria caché. El momento de la actualización automática no es configurable. Si le preocupa el impacto en el rendimiento de la memoria caché, debe actualizar el software usted mismo antes de que expire el período de tiempo.

Si la caché se detiene cuando se supera la fecha de finalización, actualizará automáticamente el software la próxima vez que se inicie. (Es posible que la actualización no se inicie inmediatamente, pero se iniciará en la primera hora).

### <a name="portal"></a>[Portal](#tab/azure-portal)

Haga clic en el botón **Actualizar** para comenzar la actualización de software. El estado de la memoria caché cambia a **Actualizando** hasta que se complete la operación.

### <a name="azure-cli"></a>[CLI de Azure](#tab/azure-cli)

[!INCLUDE [cli-reminder.md](includes/cli-reminder.md)]

En la CLI de Azure, se incluye nueva información sobre el software al final del informe de estado de la memoria caché. (Use [az hpc-cache show](/cli/azure/ext/hpc-cache/hpc-cache#ext-hpc-cache-az-hpc-cache-show) para comprobarlo). Busque la cadena "upgradeStatus" en el mensaje.

Use [az hpc-cache upgrade-firmware](/cli/azure/ext/hpc-cache/hpc-cache#ext-hpc-cache-az-hpc-cache-upgrade-firmware) para aplicar la actualización, si existe alguna.

Si no hay ninguna actualización disponible, esta operación no surte ningún efecto.

En este ejemplo se muestra el estado de la memoria caché (no hay ninguna actualización disponible) y los resultados del comando upgrade-firmware.

```azurecli
$ az hpc-cache show --name doc-cache0629
{
  "cacheSizeGb": 3072,
  "health": {
    "state": "Healthy",
    "statusDescription": "The cache is in Running state"
  },

<...>

  "tags": null,
  "type": "Microsoft.StorageCache/caches",
  "upgradeStatus": {
    "currentFirmwareVersion": "5.3.61",
    "firmwareUpdateDeadline": "0001-01-01T00:00:00+00:00",
    "firmwareUpdateStatus": "unavailable",
    "lastFirmwareUpdate": "2020-06-29T22:18:32.004822+00:00",
    "pendingFirmwareVersion": null
  }
}
$ az hpc-cache upgrade-firmware --name doc-cache0629
$
```

---

## <a name="delete-the-cache"></a>Eliminación de la caché

El botón **Eliminar** destruye la memoria caché. Cuando se elimina una memoria caché, todos sus recursos se destruyen y ya no incurren en cargos de cuenta.

Los volúmenes de almacenamiento de back-end que se usan como destinos de almacenamiento no se ven afectados cuando se elimina la caché. Puede agregarlos a una memoria caché futura más adelante, o bien decomisarlos por separado.

> [!NOTE]
> Azure HPC Cache no escribe automáticamente los datos modificados de la caché en los sistemas de almacenamiento de back-end antes de eliminar la caché.
>
> Para asegurarse de que todos los datos de la caché se han escrito en almacenamiento a largo plazo, [detenga la caché](#stop-the-cache) antes de eliminarla. Asegúrese de que muestra el estado **Detenido** antes eliminarla.

### <a name="portal"></a>[Portal](#tab/azure-portal)

Después de detener la memoria caché, haga clic en el botón **Eliminar** para quitar la memoria caché de forma permanente.

### <a name="azure-cli"></a>[CLI de Azure](#tab/azure-cli)

[!INCLUDE [cli-reminder.md](includes/cli-reminder.md)]

Use el comando de la CLI de Azure [az hpc-cache delete](/cli/azure/ext/hpc-cache/hpc-cache#ext-hpc-cache-az-hpc-cache-delete) para quitar la memoria caché de forma permanente.

Ejemplo:
```azurecli
$ az hpc-cache delete --name doc-cache0629
 - Running ..

<...>

{- Finished ..
  "endTime": "2020-07-09T22:24:35.1605019+00:00",
  "name": "7d3cd0ba-11b3-4180-8298-d9cafc9f22c1",
  "startTime": "2020-07-09T22:13:32.0732892+00:00",
  "status": "Succeeded"
}
$
```

---

## <a name="cache-metrics-and-monitoring"></a>Supervisión y métricas de caché

La página de información general muestra gráficos para algunas estadísticas básicas de caché: rendimiento de caché, operaciones por segundo y latencia.

![Captura de pantalla de tres gráficos de líneas que muestran las estadísticas mencionadas anteriormente para una memoria caché de ejemplo](media/hpc-cache-overview-stats.png)

Estos gráficos forman parte de las herramientas de supervisión y análisis integradas de Azure. Existen otras herramientas y alertas disponibles en las páginas del encabezado **Supervisión** de la barra lateral del portal. Obtenga más información en la sección del portal de la [documentación de Supervisión de Azure](../azure-monitor/insights/monitor-azure-resource.md#monitoring-in-the-azure-portal).

## <a name="next-steps"></a>Pasos siguientes

* Más información sobre las [herramientas de métricas y estadísticas de Azure](../azure-monitor/index.yml)
* Obtener [ayuda con la instancia de Azure HPC Cache](hpc-cache-support-ticket.md)
