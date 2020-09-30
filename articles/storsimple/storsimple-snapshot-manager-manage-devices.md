---
title: Administración de dispositivos con StorSimple Snapshot Manager | Microsoft Docs
description: Describe cómo usar el complemento MMC para Administrador de instantáneas StorSimple para conectar y administrar dispositivos StorSimple.
services: storsimple
documentationcenter: ''
author: alkohli
manager: timlt
editor: ''
ms.assetid: 966ecbe3-a7fa-4752-825f-6694dd949946
ms.service: storsimple
ms.devlang: NA
ms.topic: how-to
ms.tgt_pltfrm: NA
ms.workload: NA
ms.date: 06/05/2017
ms.author: alkohli
ms.openlocfilehash: dbb7e835d110bcb6cb81fd5425a2aafbb578e692
ms.sourcegitcommit: 814778c54b59169c5899199aeaa59158ab67cf44
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/13/2020
ms.locfileid: "90054857"
---
# <a name="use-storsimple-snapshot-manager-to-connect-and-manage-storsimple-devices"></a>Uso de Administrador de instantáneas StorSimple para conectar y administrar dispositivos StorSimple
## <a name="overview"></a>Información general
Puede usar los nodos en el panel **Ámbito** de Administrador de instantáneas StorSimple para comprobar los datos importados de dispositivo de StorSimple y actualizar dispositivos de almacenamiento conectados. Además, al hacer clic en el nodo **Dispositivos**, puede ver una lista de dispositivos conectados y, en el panel **Resultados**, la información de estado correspondiente.

![Dispositivos conectados](./media/storsimple-snapshot-manager-manage-devices/HCS_SSM_connect_devices.png)

**Ilustración 1: Dispositivo conectado a Administrador de instantáneas StorSimple** 

Dependiendo de su selección de **Vista**, el panel **Resultados** muestra la siguiente información sobre cada dispositivo. Para obtener más información acerca de cómo configurar una vista, vaya a [Menú Ver](storsimple-use-snapshot-manager.md#view-menu).

| Columna Resultados | Descripción |
|:--- |:--- |
| Nombre |El nombre del dispositivo tal como está configurado en el Portal de Azure clásico |
| Modelo |El número de modelo del dispositivo |
| Versión |La versión del software instalado en el dispositivo |
| Status |Si el dispositivo está disponible |
| Última sincronización |Fecha y hora cuando el dispositivo se sincronizó por última vez |
| Nº de serie |El número de serie del dispositivo |

Si hace clic con el botón derecho en el nodo **Dispositivos** en el panel **Ámbito**, puede seleccionar entre las siguientes acciones:

* Incorporación o remplazo de un dispositivo
* Conexión de un dispositivo y comprobación de las importaciones
* Actualización de los dispositivos conectados

Si hace clic en el nodo **Dispositivos** y, luego, hace clic con el botón derecho en el nombre del dispositivo en el panel **Resultados**, puede seleccionar entre las siguientes acciones:

* Autenticar un dispositivo
* Ver los detalles de dispositivo
* Actualizar un dispositivo
* Eliminación de una configuración de dispositivo
* Cambiar una contraseña de dispositivo

> [!NOTE]
> Todas estas acciones también están disponibles en el panel **Acciones** .


Este tutorial explica cómo usar Administrador de instantáneas StorSimple para conectarse y administrar dispositivos y realizar las siguientes tareas:

* Incorporación o remplazo de un dispositivo
* Conexión de un dispositivo y comprobación de las importaciones
* Actualización de los dispositivos conectados
* Autenticar un dispositivo
* Ver los detalles de dispositivo
* Actualización de un dispositivo individual
* Eliminación de una configuración de dispositivo
* Cambio de una contraseña de dispositivo caducada
* Reemplazo de un dispositivo con errores

> [!NOTE]
> Para obtener información general acerca del uso de la interfaz del Administrador de instantáneas StorSimple, vaya a [Interfaz de usuario del Administrador de instantáneas StorSimple](storsimple-use-snapshot-manager.md).


## <a name="add-or-replace-a-device"></a>Incorporación o remplazo de un dispositivo
Utilice el procedimiento siguiente para agregar o reemplazar un dispositivo StorSimple.

#### <a name="to-add-or-replace-a-device"></a>Para agregar o reemplazar un dispositivo
1. Haga clic en el icono del escritorio para iniciar Administrador de instantáneas StorSimple.
2. En el panel **Ámbito**, haga clic con el botón derecho en el nodo **Dispositivos** y, después, haga clic en **Configurar un dispositivo**. Aparece el cuadro de diálogo **Configurar un dispositivo** .
   
    ![Configuración de un dispositivo StorSimple](./media/storsimple-snapshot-manager-manage-devices/HCS_SSM_config_device.png) 
3. En la lista desplegable **Dispositivo** , seleccione la dirección IP del dispositivo o el dispositivo virtual. 
4. En la casilla **Contraseña** , escriba la contraseña de StorSimple Snapshot Manager que creó para el dispositivo en el Portal de Azure clásico. Haga clic en **OK**. Administrador de instantáneas StorSimple busca el dispositivo que ha identificado. 
   
   * Si el dispositivo está disponible, Administrador de instantáneas StorSimple agrega una conexión.
   * Si el dispositivo no está disponible por alguna razón, Administrador de instantáneas StorSimple devuelve un mensaje de error. Haga clic en **Aceptar** para cerrar el mensaje de error y, luego, haga clic en **Cancelar** para cerrar el cuadro de diálogo **Configurar un dispositivo**.

## <a name="connect-a-device-and-verify-imports"></a>Conexión de un dispositivo y comprobación de las importaciones
Utilice el siguiente procedimiento para conectar un dispositivo de StorSimple y compruebe que se importan todos los grupos de volúmenes existentes que tienen asociadas copias de seguridad.

#### <a name="to-connect-a-device-and-verify-imports"></a>Para conectar un dispositivo y comprobar las importaciones
1. Para conectar un dispositivo en Administrador de instantáneas StorSimple, siga las instrucciones en Agregar o reemplazar un dispositivo. Cuando se conecta a un dispositivo, Administrador de instantáneas StorSimple responde como sigue:
   
   * Si el dispositivo no está disponible por alguna razón, Administrador de instantáneas StorSimple devuelve un mensaje de error. 
   
   * Si el dispositivo está disponible, Administrador de instantáneas StorSimple agrega una conexión. Al seleccionar el dispositivo, aparece en el panel **Resultados** y el campo de estado indica que el dispositivo está **Disponible**. Administrador de instantáneas StorSimple importa los grupos de volúmenes configurados para el dispositivo, siempre que los grupos de volúmenes tengan asociadas copias de seguridad. Las directivas de copia de seguridad no se importan. Los grupos de volúmenes que no tienen copias de seguridad asociadas no se importan.
2. Haga clic en el icono del escritorio para iniciar Administrador de instantáneas StorSimple.
3. Haga clic con el botón derecho en el primer nodo del panel **Ámbito** y, luego, haga clic en **Alternar vista de importaciones**.
   
    ![Selección de Alternar visualización de importaciones](./media/storsimple-snapshot-manager-manage-devices/HCS_SSM_Toggle_Imports_Display.png) 
4. El cuadro de diálogo **Alternar visualización de importaciones** aparece y muestra el estado de los grupos de volúmenes y las copias de seguridad importados. Haga clic en **OK**.

Después de que se hayan importado correctamente los grupos de volúmenes y las copias de seguridad, puede usar Administrador de instantáneas StorSimple para administrarlos, tal como administraría los grupos de volúmenes y copias de seguridad que haya creado y configurado con Administrador de instantáneas StorSimple. 

## <a name="refresh-connected-devices"></a>Actualización de los dispositivos conectados
Utilice el siguiente procedimiento para sincronizar los dispositivos StorSimple conectados con Administrador de instantáneas StorSimple.

#### <a name="to-refresh-connected-devices"></a>Para actualizar los dispositivos conectados
1. Haga clic en el icono del escritorio para iniciar Administrador de instantáneas StorSimple.
2. En el panel **Ámbito**, haga clic con el botón derecho en **Dispositivos** y, luego, haga clic en **Actualizar dispositivos**. Esto sincroniza los dispositivos conectados con Administrador de instantáneas StorSimple para que pueda ver los grupos de volúmenes y copias de seguridad, incluidas las adiciones recientes. 
   
    ![Actualización de los dispositivos StorSimple](./media/storsimple-snapshot-manager-manage-devices/HCS_SSM_Refresh_devices.png)

La acción **Actualizar dispositivos** recupera los nuevos grupos de volúmenes y las copias de seguridad asociadas de los dispositivos conectados. A diferencia de la acción **Volver a examinar volúmenes** disponible para el nodo **Volúmenes**, **Actualizar dispositivos** no restaura el registro de copia de seguridad.

## <a name="authenticate-a-device"></a>Autenticar un dispositivo
Utilice el procedimiento siguiente para autenticar un dispositivo de StorSimple con Administrador de instantáneas StorSimple.

#### <a name="to-authenticate-a-device"></a>Para autenticar un dispositivo
1. Haga clic en el icono del escritorio para iniciar Administrador de instantáneas StorSimple.
2. En el panel **Ámbito**, haga clic en **Dispositivos**.
3. En el panel **Resultados**, haga clic con el botón derecho en el nombre del dispositivo y, luego, haga clic en **Autenticar**.
4. Aparece el cuadro de diálogo **Autenticar** . Escriba la contraseña del dispositivo y, a continuación, haga clic en **Aceptar**.
   
    ![Cuadro de diálogo autenticar](./media/storsimple-snapshot-manager-manage-devices/HCS_SSM_Authenticate.png) 

## <a name="view-device-details"></a>Ver los detalles de dispositivo
Utilice el procedimiento siguiente para ver los detalles de un dispositivo de StorSimple y, si es necesario, vuelva a sincronizar el dispositivo con Administrador de instantáneas StorSimple.

#### <a name="to-view-and-resynchronize-device-details"></a>Para ver y volver a sincronizar los detalles del dispositivo
1. Haga clic en el icono del escritorio para iniciar Administrador de instantáneas StorSimple.
2. En el panel **Ámbito**, haga clic en **Dispositivos**.
3. En el panel **Resultados**, haga clic con el botón derecho en el nombre del dispositivo y, luego, haga clic en **Detalles**.

4. Aparecerá el cuadro de diálogo **Detalles del dispositivo**. Este cuadro muestra el nombre, modelo, versión, número de serie, estado, nombre calificado iSCSI (IQN) de destino y última fecha y hora de sincronización.

* Haga clic en **Resincronizar** para sincronizar el dispositivo.
* Haga clic en **Aceptar** o **Cancelar** para cerrar el cuadro de diálogo.
  
  ![Detalles del dispositivo](./media/storsimple-snapshot-manager-manage-devices/HCS_SSM_Device_details.png) 

## <a name="refresh-an-individual-device"></a>Actualización de un dispositivo individual
Utilice el procedimiento siguiente para volver a sincronizar un dispositivo individual de StorSimple con Administrador de instantáneas StorSimple.

#### <a name="to-refresh-a-device"></a>Para actualizar un dispositivo
1. Haga clic en el icono del escritorio para iniciar Administrador de instantáneas StorSimple. 
2. En el panel **Ámbito**, haga clic en **Dispositivos**. 
3. En el panel **Resultados**, haga clic con el botón derecho en el nombre del dispositivo y, luego, haga clic en **Actualizar dispositivo**. Esto sincroniza el dispositivo con Administrador de instantáneas StorSimple.

## <a name="delete-a-device-configuration"></a>Eliminación de una configuración de dispositivo
Utilice el procedimiento siguiente para eliminar una configuración individual de dispositivo de StorSimple desde Administrador de instantáneas StorSimple.

#### <a name="to-delete-a-device-configuration"></a>Para eliminar una configuración de dispositivo
1. Haga clic en el icono del escritorio para iniciar Administrador de instantáneas StorSimple.
2. En el panel **Ámbito**, haga clic en **Dispositivos**. 
3. En el panel **Resultados**, haga clic con el botón derecho en el nombre del dispositivo y, luego, haga clic en **Eliminar**. 
4. Aparece el mensaje siguiente. Haga clic en **Sí** para eliminar la configuración o haga clic en **No** para cancelar la eliminación.
   
    ![Eliminación de la configuración del dispositivo](./media/storsimple-snapshot-manager-manage-devices/HCS_SSM_DeleteDevice.png)

## <a name="change-an-expired-device-password"></a>Cambio de una contraseña de dispositivo caducada
Para autenticar a un dispositivo de StorSimple con Administrador de instantáneas StorSimple tiene que escribir una contraseña. Esta contraseña se configura cuando se usar la interfaz de Windows PowerShell para configurar el dispositivo. Sin embargo, la contraseña puede expirar. Si esto ocurre, puede usar el Portal de Azure clásico para cambiar la contraseña. A continuación, ya que el dispositivo se configuró en Administrador de instantáneas StorSimple antes de que la contraseña caducara, tiene que volver a autenticar el dispositivo en Administrador de instantáneas StorSimple.

#### <a name="to-change-the-expired-password"></a>Para cambiar la contraseña caducada
1. En el Portal de Azure clásico, inicie el servicio StorSimple Manager.
2. Haga clic en **Dispositivos** > **Configurar** para el dispositivo.
3. Desplácese hacia abajo hasta la sección de Administrador de instantáneas StorSimple. Escriba una contraseña que tenga 14 o 15 caracteres. Asegúrese de que la contraseña contiene una combinación de mayúsculas, minúsculas, números y caracteres especiales.
4. Vuelva a escribir la contraseña para confirmarla.
5. Haga clic en **Guardar** en la parte inferior de la página.

#### <a name="to-re-authenticate-the-device"></a>Para volver a autenticar el dispositivo
1. Inicie Administrador de instantáneas StorSimple.
2. En el panel **Ámbito**, haga clic en **Dispositivos**. Aparecerá una lista de dispositivos configurados en el panel **Resultados** .
3. Seleccione el dispositivo y, a continuación, haga clic con el botón derecho en **Autenticar**.
4. En la ventana **Autenticar** , escriba la nueva contraseña.
5. Seleccione el dispositivo, haga clic con el botón derecho y seleccione **Actualizar dispositivo**. Esto sincroniza el dispositivo con Administrador de instantáneas StorSimple.

## <a name="replace-a-failed-device"></a>Reemplazo de un dispositivo con errores
Si un dispositivo StorSimple falla y se sustituye por un dispositivo en espera (conmutación por error), siga estos pasos para conectar con el nuevo dispositivo y ver las copias de seguridad asociadas.

#### <a name="to-connect-to-a-new-device-after-failover"></a>Para conectarse a un nuevo dispositivo después de la conmutación por error
1. Volver a configurar la conexión iSCSI al nuevo dispositivo. Para obtener instrucciones, consulte "Paso 7: Montar, inicializar y formatear un volumen" en [Implementar el dispositivo de StorSimple local](storsimple-8000-deployment-walkthrough-u2.md).

> [!NOTE]
> Si el nuevo dispositivo de StorSimple tiene la misma dirección IP que el antiguo, puede conectarse con la configuración anterior.


1. Detenga el servicio de administración de Microsoft StorSimple:
   
   1. Inicie el Administrador del servidor.
   2. En el panel Administrador del servidor, en el menú **Herramientas**, seleccione **Servicios**.
   3. En la ventana **Servicios**, seleccione **Microsoft StorSimple Management Service**.
   4. En el panel derecho, en **Microsoft StorSimple Management Service**, haga clic en **Detener el servicio**.
2. Quitar la información de configuración relacionada con el dispositivo anterior:
   
   1. En el Explorador de archivos, vaya a C:\ProgramData\Microsoft\StorSimple\BACatalog.
   2. Elimine los archivos de la carpeta BACatalog.
3. Reinicie el servicio de administración de Microsoft StorSimple:
   
   1. En el panel Administrador del servidor, en el menú **Herramientas**, seleccione **Servicios**.
   2. En la ventana **Servicios**, seleccione **Microsoft StorSimple Management Service**.
   3. En el panel derecho, en **Microsoft StorSimple Management Service**, haga clic en **Reiniciar el servicio**.
4. Inicie Administrador de instantáneas StorSimple.
5. Para configurar el nuevo dispositivo StorSimple, complete los pasos que encontrará en Paso 2: Conexión a un dispositivo StorSimple en [Implementación del Administrador de instantáneas StorSimple](storsimple-snapshot-manager-deployment.md).
6. Haga clic con el botón derecho en el nodo de nivel superior en el panel **Ámbito** (StorSimple Snapshot Manager en el ejemplo) y, luego, haga clic en **Alternar vista de importaciones**. 
7. Aparece un mensaje cuando los grupos de volúmenes importados y las copias de seguridad son visibles en Administrador de instantáneas StorSimple. Haga clic en **OK**.

## <a name="next-steps"></a>Pasos siguientes
* Obtenga más información sobre el [uso de Snapshot Manager de StorSimple para administrar la solución de StorSimple](storsimple-snapshot-manager-admin.md).
* Obtenga más información sobre el [uso de Snapshot Manager de StorSimple para ver y administrar volúmenes de copia de seguridad](storsimple-snapshot-manager-manage-volumes.md).

