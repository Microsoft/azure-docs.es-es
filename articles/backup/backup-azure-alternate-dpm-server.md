---
title: Recuperación de datos de una instancia de Azure Backup Server
description: Recupere los datos que protegió en un almacén de Recovery Services desde cualquier Azure Backup Server registrado en dicho almacén.
ms.topic: conceptual
ms.date: 07/09/2019
ms.openlocfilehash: 0cce45fbaaedad4cfcee0cda768aae2fa294312b
ms.sourcegitcommit: 3246e278d094f0ae435c2393ebf278914ec7b97b
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/02/2020
ms.locfileid: "89378073"
---
# <a name="recover-data-from-azure-backup-server"></a>Recuperación de datos de Azure Backup Server

Puede usar Azure Backup Server para recuperar los datos de los que creó una copia de seguridad en un almacén de Recovery Services. El proceso para hacerlo está integrado en la consola de administración de Azure Backup Server y es similar al flujo de trabajo de recuperación de otros componentes de Azure Backup.

> [!NOTE]
> Este artículo se puede aplicar a [System Center Data Protection Manager 2012 R2 con UR7 o posterior](https://support.microsoft.com/kb/3065246), en combinación con el [agente de Azure Backup más reciente](https://aka.ms/azurebackup_agent).
>
>

Para recuperar datos de una instancia de Azure Backup Server:

1. En la pestaña **Recuperación** de la consola de administración de Azure Backup Server, seleccione **"Agregar DPM externo"** (en la parte superior izquierda de la pantalla).

    ![Agregar DPM externo](./media/backup-azure-alternate-dpm-server/add-external-dpm.png)
2. Descargue las nuevas **credenciales del almacén** asociado al **Azure Backup Server** en donde se recuperan los datos, elija el Azure Backup Server en la lista de servidores registrados en el almacén de Recovery Services y escriba la **frase de contraseña de cifrado** asociada al servidor cuyos datos se recuperan.

    ![Credenciales externas de DPM](./media/backup-azure-alternate-dpm-server/external-dpm-credentials.png)

   > [!NOTE]
   > Solo las instancias de Azure Backup Server asociadas al mismo almacén de registro pueden recuperar datos unos de otros.
   >
   >

    Una vez agregado correctamente el Azure Backup Server externo, puede examinar los datos del servidor externo y el Azure Backup Server local en la pestaña **Recuperación**.
3. Vaya a la lista disponible de servidores de producción protegidos por el Azure Backup Server externo y seleccione el origen de datos adecuado.

    ![Ir al servidor DPM externo](./media/backup-azure-alternate-dpm-server/browse-external-dpm.png)
4. Seleccione el **mes y año** en la lista desplegable **Puntos de recuperación**, seleccione la **fecha de recuperación** en que se creó el punto de recuperación y seleccione la **hora de recuperación**.

    Aparece una lista de archivos y carpetas en el panel inferior que se puede examinar y recuperar en cualquier ubicación.

    ![Puntos de recuperación del servidor DPM externo](./media/backup-azure-alternate-dpm-server/external-dpm-recoverypoint.png)
5. Haga clic con el botón derecho en el elemento apropiado y seleccione **Recuperar**.

    ![Recuperación del DPM externo](./media/backup-azure-alternate-dpm-server/recover.png)
6. Revise la **Selección de recuperación**. Compruebe los datos y la hora de la copia de seguridad que se va a recuperar, así como el origen desde el que se creó la copia de seguridad. Si la selección no es correcta, elija **Cancelar** a fin de volver a la pestaña de recuperación para seleccionar el punto de recuperación adecuado. Si la selección es correcta, elija **Siguiente**.

    ![Resumen de recuperación del DPM externo](./media/backup-azure-alternate-dpm-server/external-dpm-recovery-summary.png)
7. Seleccione **Recuperar en una ubicación alternativa**. **Examinar** para ir a la ubicación correcta para la recuperación.

    ![Ubicación alternativa de la recuperación del DPM externo](./media/backup-azure-alternate-dpm-server/external-dpm-recovery-alternate-location.png)
8. Elija una de las siguientes opciones: **Crear copia**, **Omitir** o **Sobrescribir**.

   * **Crear copia**: crea una copia del archivo si hay un conflicto de nombres.
   * **Omitir**: si hay un conflicto de nombres, no recupera el archivo, sino que deja el archivo original.
   * **Sobrescribir**: si hay un conflicto de nombres, sobrescribe la copia existente del archivo.

     Elija la opción adecuada para **Restaurar seguridad**. Puede aplicar la configuración de seguridad del equipo de destino donde se están recuperando los datos o la configuración de seguridad aplicable al producto en el momento en que se creó el punto de recuperación.

     Identifique si se envía una **notificación** una vez que se completa correctamente la recuperación.

     ![Notificaciones de recuperación del DPM externo](./media/backup-azure-alternate-dpm-server/external-dpm-recovery-notifications.png)
9. En la pantalla **Resumen** se muestran las opciones elegidas hasta el momento. Una vez que selecciona **"Recuperar"** , los datos se recuperan en la ubicación local adecuada.

    ![Resumen de las opciones de recuperación del DPM externo](./media/backup-azure-alternate-dpm-server/external-dpm-recovery-options-summary.png)

   > [!NOTE]
   > El trabajo de recuperación se puede supervisar en la pestaña **Supervisión** del Azure Backup Server.
   >
   >

    ![Supervisión de la recuperación](./media/backup-azure-alternate-dpm-server/monitoring-recovery.png)
10. Puede seleccionar **Borrar DPM externo** en la pestaña **Recuperación** del servidor DPM para quitar la vista del servidor DPM externo.

    ![Borrar DPM externo](./media/backup-azure-alternate-dpm-server/clear-external-dpm.png)

## <a name="troubleshooting-error-messages"></a>Solución de mensajes de error

| No. | Mensaje de error | Pasos para solucionar problemas |
|:---:|:--- |:--- |
| 1. |Este servidor no está registrado en el almacén especificado por las credenciales del almacén. |**Causa:** Este error aparece cuando el archivo de credenciales del almacén seleccionado no pertenece al almacén de Recovery Services asociado al Azure Backup Server en el que se intenta la recuperación. <br> **Resolución:** Descargue el archivo de credenciales del almacén de Recovery Services en el que está registrado el Azure Backup Server. |
| 2. |Los datos recuperables no están disponibles o el servidor seleccionado no es un servidor DPM. |**Causa:** No hay ninguna otra instancia de Azure Backup Server registrada en el almacén de Recovery Services, los servidores aún no han cargado los metadatos o el servidor seleccionado no es una instancia de Azure Backup Server (que utilice un servidor o cliente de Windows). <br> **Resolución:** Si hay otras instancias de Azure Backup Server registradas en el almacén de Recovery Services, asegúrese de que está instalado el agente de Azure Backup más reciente. <br>Si hay otras instancias de Azure Backup Server registradas en el almacén de Recovery Services, espere un día después de la instalación para iniciar el proceso de recuperación. Por la noche se cargarán en la nube los metadatos de todas las copias de seguridad protegidas. Los datos estarán disponibles para la recuperación. |
| 3. |Ningún otro servidor DPM está registrado en este almacén. |**Causa:** No hay ningún otro Azure Backup Server registrado en el almacén desde el que se intenta realizar la recuperación.<br>**Resolución:** Si hay otras instancias de Azure Backup Server registradas en el almacén de Recovery Services, asegúrese de que está instalado el agente de Azure Backup más reciente.<br>Si hay otras instancias de Azure Backup Server registradas en el almacén de Recovery Services, espere un día después de la instalación para iniciar el proceso de recuperación. El trabajo nocturno carga los metadatos de todas las copias de seguridad protegidas en la nube. Los datos estarán disponibles para la recuperación. |
| 4. |La frase de contraseña de cifrado especificada no coincide con la frase de contraseña asociada al siguiente servidor: **\<server name>** |**Causa:** La frase de contraseña de cifrado que se usa en el proceso de cifrado de los datos del Azure Backup Server que se recuperan no coincide con la proporcionada. El agente no puede descifrar los datos y se produce un error en la recuperación.<br>**Resolución:** Especifique la misma frase de contraseña de cifrado asociada a Azure Backup Server cuyos datos se están recuperando. |

## <a name="next-steps"></a>Pasos siguientes

Lea las otras preguntas más frecuentes:

* [Preguntas comunes](backup-azure-vm-backup-faq.md) sobre las copias de seguridad de máquinas virtuales de Azure
* [Preguntas comunes](backup-azure-file-folder-backup-faq.md) sobre el agente de Azure Backup
