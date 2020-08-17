---
title: Solución de problemas de Azure FarmBeats
description: En este artículo se describe cómo solucionar problemas de Azure FarmBeats.
author: uhabiba04
ms.topic: article
ms.date: 11/04/2019
ms.author: v-umha
ms.openlocfilehash: 110fef2dcb4c1581d49fcb776276ebd9c4ae2235
ms.sourcegitcommit: 3543d3b4f6c6f496d22ea5f97d8cd2700ac9a481
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/20/2020
ms.locfileid: "86497070"
---
# <a name="troubleshoot-azure-farmbeats"></a>Solución de problemas de Azure FarmBeats

En este artículo se proporcionan soluciones a problemas comunes de Azure FarmBeats. Para obtener ayuda adicional, póngase en contacto con el [foro de soporte técnico](https://social.msdn.microsoft.com/Forums/home?forum=ProjectFarmBeats) o envíenos un correo electrónico a farmbeatssupport@microsoft.com.

> [!NOTE]
  > Si ha instalado FarmBeats durante el mes de abril y nota que se están produciendo errores en los trabajos con un mensaje de error vacío, es posible que no se haya asignado ninguna cuota de lote a su instalación debido a que se ha dado prioridad de asistencia a organizaciones sanitarias y de seguridad de importancia crítica. Vea [aquí](https://azure.microsoft.com/blog/update-2-on-microsoft-cloud-services-continuity/) para obtener más información. Tendrá que solicitar que las máquinas virtuales se asignen a la cuenta de Batch para ejecutar los trabajos correctamente.

## <a name="install-issues"></a>Problemas de instalación

  > [!NOTE]
  > Si está reiniciando la instalación debido a un error, asegúrese de eliminar el **grupo de recursos** o todos los recursos del grupo de recursos antes de volver a desencadenar la instalación.

### <a name="invalid-sentinel-credentials"></a>Credenciales de Sentinel no válidas

Las credenciales de Sentinel proporcionadas durante la instalación no son correctas. Reinicie la instalación con las credenciales correctas.

### <a name="the-regional-account-quota-of-batch-accounts-for-the-specified-subscription-has-been-reached"></a>Se ha alcanzado la cuota de cuenta regional de las cuentas de Batch para la suscripción especificada.

Aumente la cuota o elimine las cuentas de Batch sin usar y reinicie la instalación.

### <a name="invalid-resource-group-location"></a>Ubicación del grupo de recursos no válida

Asegúrese de que el **grupo de recursos** se encuentra en la misma ubicación que la **región** especificada durante la instalación.

### <a name="other-install-issues"></a>Otros problemas de instalación

Póngase en contacto con nosotros y proporciónenos los detalles siguientes:

- Id. de suscripción
- Nombre del grupo de recursos
- Siga los pasos siguientes para adjuntar el archivo de registro para el error de implementación:

    1. En Azure Portal, vaya al **grupo de recursos**.

    2. En el lado izquierdo, seleccione **Implementaciones** en **Configuración**.

    3. En cada implementación que se muestre con el estado **Error**, seleccione entre los detalles para descargar los detalles de la implementación. Adjunte este archivo al correo.

## <a name="sensor-telemetry"></a>Telemetría del sensor

### <a name="cant-view-telemetry-data"></a>No se ven los datos de telemetría

**Síntoma**: los dispositivos o sensores están implementados, y se ha vinculado FarmBeats con el asociado del dispositivo; sin embargo, no se pueden obtener ni ver datos de telemetría en FarmBeats.

**Acción correctiva**:

1. Vaya al grupo de recursos de FarmBeats.
2. Seleccione el espacio de nombres del **Centro de eventos** ("sensor-partner-eh-namespace-xxxx"), haga clic en "Event Hubs" y, a continuación, compruebe el número de mensajes entrantes en el centro de eventos asignado al asociado.
3. Realice cualquiera de las siguientes acciones:

   - En caso de que no haya *ningún mensaje entrante*, póngase en contacto con el asociado del dispositivo.  
   - En el caso de que haya *mensajes entrantes*, póngase en contacto con nosotros y proporciónenos los registros del acelerador y el centro de datos, así como la telemetría capturada.

Para saber cómo descargar registros, vaya a la sección ["Recopilación de registros de forma manual"](#collect-logs-manually).  

### <a name="cant-view-telemetry-data-after-ingesting-historicalstreaming-data-from-your-sensors"></a>No se pueden ver los datos de telemetría después de ingerir datos históricos o de streaming de los sensores

**Síntoma**: hay dispositivos o sensores implementados, y ha creado los dispositivos o sensores en FarmBeats y ha ingerido la telemetría en EventHub, pero no puede obtener ni ver los datos de telemetría en FarmBeats.

**Acción correctiva**:

1. Asegúrese de que ha realizado correctamente el registro del asociado. Para comprobarlo, vaya a su swagger del centro de datos, vaya a /Partner API, realice una operación get y compruebe si el asociado está registrado. En caso contrario, siga estos [pasos](get-sensor-data-from-sensor-partner.md#enable-device-integration-with-farmbeats) para agregar un asociado.

2. Asegúrese de que ha utilizado el formato de mensaje de telemetría correcto:

```json
{
"deviceid": "<id of the Device created>",
"timestamp": "<timestamp in ISO 8601 format>",
"version" : "1",
"sensors": [
    {
      "id": "<id of the sensor created>",
      "sensordata": [
        {
          "timestamp": "< timestamp in ISO 8601 format >",
          "<sensor measure name (as defined in the Sensor Model)>":<value>
        },
        {
          "timestamp": "<timestamp in ISO 8601 format>",
          "<sensor measure name (as defined in the Sensor Model)>": <value>
        }
      ]
    }
 ]
}
```

### <a name="dont-have-the-azure-event-hubs-connection-string"></a>No tiene la cadena de conexión de Azure Event Hubs

**Acción correctiva**:

1. En Swagger de Datahub, vaya a la API del asociado.
2. Seleccione **Get** > **Try it out** > **Execute** (Obtener > Probar > Ejecutar).

> [!NOTE]
> Identificador del asociado del sensor que le interesa.

3. Vuelva a Partner API y seleccione **Get/\<ID>** (Obtener/).
4. Especifique el identificador del asociado del paso 3 y, luego, seleccione **Execute** (Ejecutar).

   La respuesta de la API debe tener la cadena de conexión de Event Hubs.

### <a name="device-appears-offline"></a>El dispositivo aparece sin conexión

**Síntomas**: los dispositivos están instalados y ha vinculado FarmBeats con el asociado del dispositivo. Los dispositivos están en línea y envían datos de telemetría, pero aparecen sin conexión.

**Acción correctiva** El intervalo de informes no está configurado para este dispositivo. Póngase en contacto con el fabricante del dispositivo para establecer el intervalo de informes. 

### <a name="error-deleting-a-device"></a>Error al eliminar un dispositivo

Al eliminar un dispositivo, podría encontrarse con uno de los siguientes escenarios de error comunes:  

**Mensaje**: "Se hace referencia al dispositivo en los sensores: hay uno o varios sensores asociados con el dispositivo. Elimine los sensores y, después, elimine el dispositivo".  

**Significado**: el dispositivo está asociado con varios sensores implementados en la granja.

**Acción correctiva**:  

1. elimine los sensores asociados con el dispositivo mediante Accelerator.  
2. En caso de que quiera asociar los sensores a otro dispositivo, solicite a su asociado de dispositivo que haga lo mismo.  
3. Elimine el dispositivo mediante una llamada `DELETE API` y establezca el parámetro force como *true*.  

**Mensaje**: "Se hace referencia al dispositivo en los dispositivos como ParentDeviceId: hay uno o más dispositivos que están asociados con este dispositivo como dispositivos secundarios. Elimínelos y, después, elimine este dispositivo".  

**Significado**: el dispositivo tiene otros dispositivos asociados.  

**Acción correctiva**:

1. elimine los dispositivos que están asociados a este dispositivo específico.  
2. Elimine el dispositivo específico.  

    > [!NOTE]
    > No se puede eliminar un dispositivo si hay sensores asociados. Para obtener más información sobre cómo eliminar sensores asociados, consulte la sección **Eliminación de un sensor** en [Obtención de datos de sensor de los asociados de sensor](get-sensor-data-from-sensor-partner.md).
    > Los asociados no tienen permiso para eliminar un dispositivo ni un sensor. Solo lo tienen los administradores.

## <a name="issues-with-jobs"></a>Incidencias con trabajos

### <a name="farmbeats-internal-error"></a>Error interno de FarmBeats

**Mensaje**: "Error interno de FarmBeats. Vea la Guía de solución de problemas para obtener más detalles".

**Acción correctiva** Este problema puede deberse a un error temporal en la canalización de datos. Vuelva a crear el trabajo. Si el error persiste, póngase en contacto con nosotros con el mensaje de error o los registros.

## <a name="accelerator-troubleshooting"></a>Solución de problemas del acelerador

### <a name="access-control"></a>Control de acceso

**Problema:** recibe un error mientras agrega una asignación de roles.

**Mensaje**: "No se encuentran usuarios que coincidan".

**Acción correctiva** Compruebe el identificador de correo electrónico para el que intenta agregar una asignación de roles. El identificador de correo electrónico debe coincidir exactamente con el registrado para ese usuario en Active Directory. Si el error persiste, póngase en contacto con nosotros y proporciónenos el mensaje de error o los registros.

### <a name="unable-to-log-in-to-accelerator"></a>No se puede iniciar sesión en el acelerador

**Mensaje**: "Error: no tiene autorización para llamar al servicio. Póngase en contacto con el administrador para obtener autorización".

**Acción correctiva** Pida al administrador que le autorice el acceso a la implementación de FarmBeats. Esto se puede hacer mediante una solicitud POST de las API RoleAssignment o a través de Access Control en el panel **Configuración** de Accelerator.  

Si ya se le ha concedido acceso y se encuentra con este error, actualice la página y vuelva a intentarlo. Si el error persiste, póngase en contacto con nosotros y proporciónenos el mensaje de error o los registros.

![Proyecto FarmBeats](./media/troubleshoot-azure-farmbeats/accelerator-troubleshooting-1.png)

### <a name="accelerator-issues"></a>Problemas con Accelerator  

**Problema:** ha recibido un error de Accelerator de causa indeterminada.

**Mensaje**: "Error: se ha producido un error desconocido".

**Acción correctiva** Este error se produce si deja la página inactiva durante demasiado tiempo. Actualice la página. Si el error persiste, póngase en contacto con nosotros y proporciónenos el mensaje de error o los registros.

**Problema:** Accelerator de FarmBeats no muestra la versión más reciente incluso después de actualizar FarmBeatsDeployment.

**Acción correctiva** Este error se debe a la persistencia del trabajo de servicio en el explorador. Haga lo siguiente:

1. cierre todas las pestañas del explorador que tengan abierto Accelerator y cierre la ventana del explorador.
2. Inicie una nueva instancia del explorador y vuelva a cargar el URI de Accelerator. Esta acción carga la nueva versión de Accelerator.

## <a name="sentinel-imagery-related-issues"></a>Sentinel: problemas relacionados con imágenes

### <a name="wrong-username-or-password"></a>Nombre de usuario o contraseña incorrectos

**Mensaje de error del trabajo**: "Se requiere autenticación completa para tener acceso a este recurso".

**Acción correctiva**: Realice una de las siguientes acciones:

- Siga los pasos siguientes para actualizar FarmBeats con el nombre de usuario y la contraseña correctos y vuelva a intentar el trabajo.

  **Actualización del nombre de usuario de Sentinel**

    1. Inicie sesión en el [portal de Azure](https://portal.azure.com).
    2. En el cuadro **Buscar**, busque el grupo de recursos Datahub de FarmBeats.
    3. Seleccione Storage account storage (Almacenamiento de la cuenta de almacenamiento)***** > **Containers** > **batch-prep-files** > **to_vm** > **config.ini**.
    4. Seleccione **Editar**.
    5. Actualización del nombre de usuario en la sección sentinel_account

  **Actualización de la contraseña de Sentinel**

    1. Inicie sesión en el [portal de Azure](https://portal.azure.com).
    2. En el cuadro **Buscar**, busque el grupo de recursos Datahub de FarmBeats.
    3. Seleccione keyvault-* * * * *
    4. Seleccione Directivas de acceso en Configuración.
    5. Seleccione **Agregar directiva de acceso**.
    6. Use **Administración de secretos** en Configurar a partir de una plantilla y agréguese a usted mismo a la entidad de seguridad.
    7. Seleccione **Agregar** y, después, **Guardar** en la página **Directivas de acceso**.
    8. Seleccione **Secretos** en **Configuración**.
    9. Seleccione **Sentinel-password**.
    10. Cree una nueva versión del valor y habilítela.

- Vuelva a ejecutar el trabajo con errores o ejecute un trabajo de índices por satélite para un intervalo de fechas de 5 a 7 días; luego, compruebe si el trabajo se ha realizado correctamente.

### <a name="sentinel-hub-wrongurlor-site-not-accessible"></a>Concentrador de Sentinel: Dirección URL o sitio incorrectos no accesibles

**Mensaje de error del trabajo**: "Vaya, hubo un problema. La página a la que intenta acceder no se encuentra temporalmente disponible".

**Acción correctiva**:

1. abra [Sentinel](https://scihub.copernicus.eu/dhus/) en el explorador para ver si el sitio web es accesible.
2. Si el sitio web no es accesible, compruebe si algún firewall, alguna red de empresa u otro software de bloqueo están impidiendo el acceso al sitio web y, luego, siga los pasos necesarios para permitir la dirección URL de Sentinel. 
3. Vuelva a ejecutar el trabajo con errores o ejecute un trabajo de índices por satélite para un intervalo de fechas de 5 a 7 días y, luego, compruebe si el trabajo se ha realizado correctamente.  

### <a name="sentinel-server-down-for-maintenance"></a>Servidor de Sentinel: inactivo por mantenimiento

**Mensaje de error del trabajo**: "Copernicus Open Access Hub volverá a estar operativo muy pronto". Lamentamos las molestias, estamos llevando a cabo tareas de mantenimiento en este momento. Pronto volveremos a estar en línea". 

**Acción correctiva**:

este problema puede producirse si se realizan actividades de mantenimiento en el servidor de Sentinel.

1. Si se produce un error en algún trabajo o canalización porque se están realizando tareas de mantenimiento, vuelva a enviar el trabajo más tarde. 

   Para información sobre las actividades de mantenimiento de Sentinel planeadas o no planeadas, vaya al sitio web [Copernicus Open Access Hub News](https://scihub.copernicus.eu/news/).  

2. Vuelva a ejecutar el trabajo con errores o ejecute un trabajo de índices por satélite para un intervalo de fechas de 5 a 7 días y, luego, compruebe si el trabajo se ha realizado correctamente.

### <a name="sentinel-maximum-number-of-connections-reached"></a>Sentinel: se ha alcanzado el número máximo de conexiones

**Mensaje de error del trabajo**: "el usuario '\<username>' ha alcanzado el número máximo de dos flujos simultáneos".

**Significado**: Si se produce un error en un trabajo porque se ha alcanzado el número máximo de conexiones, significa que se está usando la misma cuenta de Sentinel en varios trabajos.

**Acción correctiva**: pruebe alguna de estas soluciones:

* Espere a que finalicen los demás trabajos antes de volver a ejecutar el trabajo con errores.
* Cree una nueva cuenta de Sentinel y, a continuación, actualice el nombre de usuario y la contraseña de Sentinel en FarmBeats.

### <a name="sentinel-server-refused-connection"></a>Servidor de Sentinel: conexión rechazada

**Mensaje de error del trabajo**: "el servidor ha rechazado la conexión en: http://172.30.175.69:8983/solr/dhus".

**Acción correctiva**: este problema puede producirse si se realizan actividades de mantenimiento en el servidor de Sentinel.

1. Si se produce un error en algún trabajo o canalización porque se están realizando tareas de mantenimiento, vuelva a enviar el trabajo más tarde.

   Para información sobre las actividades de mantenimiento de Sentinel planeadas o no planeadas, vaya al sitio web [Copernicus Open Access Hub News](https://scihub.copernicus.eu/news/).  

2. Vuelva a ejecutar el trabajo con errores o ejecute un trabajo de índices por satélite para un intervalo de fechas de 5 a 7 días y, luego, compruebe si el trabajo se ha realizado correctamente.

### <a name="soil-moisture-map-has-white-areas"></a>El mapa de humedad del suelo tiene áreas blancas

**Problema:** Se ha generado el **mapa de humedad del suelo**, pero principalmente tiene áreas blancas.

**Acción correctiva**: Este problema puede producirse si los índices de satélite generados para el tiempo en el que se solicitó el mapa tienen valores de NDVI menores que 0,3. Para obtener más información, visite la [guía técnica de Sentinel](https://earth.esa.int/web/sentinel/technical-guides/sentinel-2-msi/level-2a/algorithm).

1. Vuelva a ejecutar el trabajo para otro intervalo de fechas y compruebe si los valores de NDVI de los índices de satélite son mayores que 0,3.

## <a name="collect-logs-manually"></a>Recopilación de registros de forma manual

[Instale e implemente el Explorador de Azure Storage]( https://docs.microsoft.com/azure/vs-azure-tools-storage-manage-with-storage-explorer?tabs=windows).

### <a name="collect-azure-data-factory-job-logs-or-app-service-logs-in-datahub"></a>Recopilación de los registros de trabajo de Azure Data Factory o los registros de App Service en el centro de datos

1. Inicie sesión en [Azure Portal](https://portal.azure.com).
2. En el cuadro **Buscar**, busque el grupo de recursos Datahub de FarmBeats.
3. En el panel **Grupo de recursos**, busque la cuenta de almacenamiento *datahublogs\** . Por ejemplo, *datahublogsmvxmq*.  
4. En la columna **Nombre**, seleccione la cuenta de almacenamiento para ver el panel **Cuenta de almacenamiento**.
5. En el panel **datahubblogs\*** , seleccione **Abrir en el Explorador** para ver la aplicación **Abrir Explorador de Azure Storage**.
6. En el panel izquierdo, seleccione **Contenedores de blob** y, a continuación, seleccione **job-logs** para los registros de Azure Data Factory o **appinsights-logs** para los registros de App Service.
7. Seleccione **Descargar** para descargar los registros en una carpeta local de la máquina.

    ![Proyecto FarmBeats](./media/troubleshoot-azure-farmbeats/collecting-logs-manually-1.png)

### <a name="collect-azure-data-factory-job-logs-or-app-service-logs-for-accelerator"></a>Recopilación de los registros de trabajo de Azure Data Factory o los registros de App Service para el acelerador

1. Inicie sesión en [Azure Portal](https://portal.azure.com).
2. En el cuadro de texto **Buscar**, busque el grupo de recursos Accelerator de FarmBeats.
3. En el panel **Grupo de recursos**, busque la cuenta de almacenamiento *storage\** . Por ejemplo, *storagedop4k\** .
4. Seleccione la cuenta de almacenamiento en la columna **Nombre** para ver el panel **Cuenta de almacenamiento**.
5. En el panel **storage\*** , seleccione **Abrir en el Explorador** para abrir la aplicación Explorador de Azure Storage.
6. En el panel izquierdo, seleccione **Contenedores de blob** y, a continuación, seleccione **job-logs** para los registros de Azure Data Factory o **appinsights-logs** para los registros de App Service.
7. Seleccione **Descargar** y descargue los registros en una carpeta local de la máquina.

## <a name="high-cpu-usage"></a>Uso elevado de CPU

**Error**: recibe una alerta por correo electrónico que hace referencia a una **alerta de uso elevado de CPU**.

**Acción correctiva**:

1. vaya al grupo de recursos Datahub de FarmBeats.
2. Seleccione **App Service**.  
3. Vaya a la [página de precios de App Service](https://azure.microsoft.com/pricing/details/app-service/windows/) de escalabilidad vertical y seleccione un plan de tarifa adecuado.

## <a name="weather-data-job-failures"></a>Errores de trabajos de datos meteorológicos

**Error**: Ejecuta trabajos para obtener datos del tiempo, pero se produce un error en ellos.

### <a name="collect-logs-to-troubleshoot-weather-data-job-failures"></a>Recopilación de registros para solucionar errores de trabajos de datos meteorológicos

1. Vaya al grupo de recursos de FarmBeats en Azure Portal.
2. Haga clic en el servicio Data Factory que forma parte del grupo de recursos. El servicio tendrá una etiqueta "sku: Datahub" ("SKU: centro de datos").

> [!NOTE]
> Para ver las etiquetas de los servicios en el grupo de recursos, haga clic en "Editar columnas" y agregue "Etiquetas" a la vista del grupo de recursos.

:::image type="content" source="./media/troubleshoot-Azure-farmbeats/weather-log-1.png" alt-text="Proyecto FarmBeats":::

3. En la página de información general de la factoría de datos, haga clic en **Author and Monitor** (Crear y supervisar). Se abre una pestaña nueva en el explorador. Haga clic en **Monitor**.

:::image type="content" source="./media/troubleshoot-Azure-farmbeats/weather-log-2.png" alt-text="Proyecto FarmBeats":::

4. Verá una lista de ejecuciones de canalización que forman parte de la ejecución del trabajo meteorológico. Haga clic en el trabajo para el que quiera recopilar registros.
 
:::image type="content" source="./media/troubleshoot-Azure-farmbeats/weather-log-3.png" alt-text="Proyecto FarmBeats":::

5. En la página de información general de la canalización, verá la lista de ejecuciones de actividad. Tome nota de los identificadores de ejecución de las actividades para las que quiera recopilar registros.
 
:::image type="content" source="./media/troubleshoot-Azure-farmbeats/weather-log-4.png" alt-text="Proyecto FarmBeats":::

6. Vuelva al grupo de recursos FarmBeats en Azure Portal y haga clic en la cuenta de almacenamiento con el nombre **datahublogs-XXXX**.
 
:::image type="content" source="./media/troubleshoot-Azure-farmbeats/weather-log-5.png" alt-text="Proyecto FarmBeats":::

7. Haga clic en **Contenedores** -> **adfjobs**. En el cuadro de búsqueda, escriba el identificador de ejecución del trabajo que anotó en el paso 5 anterior.
 
:::image type="content" source="./media/troubleshoot-Azure-farmbeats/weather-log-6.png" alt-text="Proyecto FarmBeats":::

8. El resultado de la búsqueda contendrá la carpeta que incluye los registros que pertenecen al trabajo. Descargue los registros y envíelos a farmbeatssupport@microsoft.com para obtener ayuda con la depuración del problema.
