---
title: Solución de problemas de desbloqueo en Azure Data Box Disk | Microsoft Docs
description: Obtenga información sobre los flujos de trabajo para solucionar problemas de la herramienta de desbloqueo con Azure Data Box Disk. Errores de la herramienta de desbloqueo con Data Box Disk Unlock
services: databox
author: alkohli
ms.service: databox
ms.subservice: disk
ms.topic: troubleshooting
ms.date: 08/05/2020
ms.author: alkohli
ms.openlocfilehash: 866cf01243983863292ada0b086f8f5b2f94e412
ms.sourcegitcommit: 4f1c7df04a03856a756856a75e033d90757bb635
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/07/2020
ms.locfileid: "87925568"
---
# <a name="troubleshoot-disk-unlocking-issues-in-azure-data-box-disk"></a>Solución de problemas de desbloqueo en Azure Data Box Disk

Este artículo se aplica a Microsoft Azure Data Box Disk y describe los flujos de trabajo que se usan para solucionar los problemas que se producen al usar la herramienta de desbloqueo. 


<!--## Query activity logs

Use the activity logs to find who unlocked and accessed the disks. Your Data Box Disk arrive on your premises in a locked state. You can use the device credentials available in the Azure portal for your order to unlock them.  

To figure out who accessed the **Device credentials** blade, you can query the Activity logs.  Any action that involves accessing **Device details > Credentials** blade is logged into the activity logs as `ListCredentials` action.

![Query Activity logs](media/data-box-logs/query-activity-log-1.png)-->


## <a name="data-box-disk-unlock-tool-errors"></a>Errores de la herramienta Data Box Disk Unlock


| Comportamiento de la herramienta o mensaje de error      | Recomendaciones                                                                             |
|-------------------------------------------------------------------------------------------------------------------------------------|------------------------------------------------------------------------------------------------------|
| No se admite la versión actual de .NET Framework. Las versiones compatibles son 4.5 y posteriores.<br><br>La herramienta se cierra con un mensaje.  | .NET 4.5 no está instalado. Instale .NET 4.5 o posterior en el equipo host donde se ejecuta la herramienta de desbloqueo de Data Box Disk.                                                                            |
| No se ha podido desbloquear ni verificar ningún volumen. Póngase en contacto con el soporte técnico de Microsoft  <br><br>La herramienta no puede desbloquear ni verificar ninguna unidad bloqueada. | La herramienta no ha podido desbloquear ninguna de las unidades bloqueadas con la clave de paso proporcionada. Póngase en contacto con el servicio de soporte técnico de Microsoft para conocer los pasos siguientes.                                                |
| Los siguientes volúmenes están desbloqueados y comprobados. <br>Letras de unidad de volumen: E:<br>No se ha podido desbloquear ningún volumen con las siguientes claves de paso: werwerqomnf, qwerwerqwdfda. <br><br>La herramienta desbloquea algunas unidades y enumera las letras de unidad correctas e incorrectas.| Parcialmente correcto. No se han podido desbloquear algunas de las unidades con la clave de paso proporcionada. Póngase en contacto con el servicio de soporte técnico de Microsoft para conocer los pasos siguientes. |
| No se han podido encontrar los volúmenes bloqueados. Compruebe que el disco recibido de Microsoft está conectado correctamente y se encuentra en estado bloqueado.          | La herramienta no puede encontrar ninguna unidad bloqueada. O las unidades están ya desbloqueadas o no se han detectado. Asegúrese de que las unidades están conectadas y bloqueadas. <br> <br>También puede ver este error si ha formateado los discos. Si ha formateado los discos, ahora no se pueden usar. Póngase en contacto con el servicio de soporte técnico de Microsoft para conocer los pasos siguientes.                                                          |
| Error irrecuperable: parámetro no válido<br>Nombre del parámetro: invalid_arg<br>USO:<br>DataBoxDiskUnlock /PassKeys:<passkey_list_separated_by_semicolon><br><br>Ejemplo: DataBoxDiskUnlock /PassKeys:passkey1;passkey2;passkey3<br>Ejemplo: DataBoxDiskUnlock /SystemCheck<br>Ejemplo: DataBoxDiskUnlock /Help<br><br>/PassKeys:       obtenga esta clave de paso del pedido de Azure Data Box Disk. La clave de paso desbloquea los discos.<br>/Help:           esta opción proporciona ayuda sobre el uso del cmdlet y ejemplos.<br>/SystemCheck:    esta opción comprueba si el sistema cumple los requisitos para ejecutar la herramienta.<br><br>Presione cualquier tecla para salir. | Parámetro no válido especificado. Los únicos parámetros permitidos son /SystemCheck, /PassKey y /Help.|


## <a name="unlock-issues-for-disks-when-using-a-windows-client"></a>Problemas en el desbloqueo de discos al usar un cliente Windows

En esta sección se detallan algunos de los principales problemas que se enfrentan durante la implementación de Data Box Disk cuando se usa un cliente Windows para la copia de datos.

### <a name="issue-could-not-unlock-drive-from-bitlocker"></a>Problema: no se pudo desbloquear la unidad desde BitLocker
 
**Causa** 

usó la contraseña en el cuadro de diálogo de BitLocker e intentó desbloquear el disco a través del cuadro de diálogo para desbloquear unidades de BitLocker. Esta acción no funcionará.

**Resolución**

Para desbloquear las unidades de Data Box Disk, debe usar la herramienta de desbloqueo de Data Box Disk y escribir la contraseña desde Azure Portal. Para más información, vaya a [Tutorial: Desempaquetado, conexión y desbloqueo de Azure Data Box Disk](data-box-disk-deploy-set-up.md#connect-to-disks-and-get-the-passkey) en la documentación.
 
### <a name="issue-could-not-unlock-or-verify-some-volumes-contact-microsoft-support"></a>Problema: no se pudieron desbloquear ni comprobar algunos volúmenes. Póngase en contacto con el soporte técnico de Microsoft
 
**Causa**

Es posible que se encuentre con el error siguiente en el registro de errores y que no pueda desbloquear ni comprobar algunos volúmenes.

`Exception System.IO.FileNotFoundException: Could not load file or assembly 'Microsoft.Management.Infrastructure, Version=1.0.0.0, Culture=neutral, PublicKeyToken=31bf3856ad364e35' or one of its dependencies. The system cannot find the file specified.`
 
Esto indica que es posible que no se encuentre la versión adecuada de Windows PowerShell en el cliente Windows.

**Resolución**

Puede instalar la [versión 5.0 de Windows PowerShell](https://www.microsoft.com/download/details.aspx?id=54616) y volver a intentar la operación.
 
Si todavía no puede desbloquear los volúmenes, copie los registros de la carpeta que tiene la herramienta de desbloqueo de Data Box Disk y [póngase en contacto con el Soporte técnico de Microsoft](data-box-disk-contact-microsoft-support.md).

## <a name="next-steps"></a>Pasos siguientes

- Aprenda a [solucionar problemas de validación](data-box-disk-troubleshoot.md).
