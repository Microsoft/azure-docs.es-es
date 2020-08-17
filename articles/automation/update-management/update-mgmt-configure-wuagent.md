---
title: Configuración de los parámetros de Windows Update para Azure Automation Update Management
description: En este artículo se explica cómo configurar los valores de Windows Update para trabajar con Azure Automation Update Management.
services: automation
ms.subservice: update-management
ms.date: 05/04/2020
ms.topic: conceptual
ms.openlocfilehash: aa4e49d0e79704b6fc9f19eb1b736b2358cf3a14
ms.sourcegitcommit: cee72954f4467096b01ba287d30074751bcb7ff4
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/30/2020
ms.locfileid: "87449816"
---
# <a name="configure-windows-update-settings-for-azure-automation-update-management"></a>Configuración de los parámetros de Windows Update para Azure Automation Update Management

Azure Automation Update Management se basa en el [cliente de Windows Update](/windows/deployment/update/windows-update-overview) para descargar e instalar las actualizaciones de Windows. Hay configuraciones específicas que el cliente de Windows Update utiliza al conectarse a Windows Server Update Services (WSUS) o Windows Update. Muchas de estas configuraciones se pueden administrar con:

- Editor de directivas de grupo local
- Directiva de grupo
- PowerShell
- Edición directa del Registro

Update Management respeta muchas de las configuraciones especificadas para controlar el cliente de Windows Update. Si usa la configuración para habilitar las actualizaciones que no son de Windows, Update Management también administrará dichas actualizaciones. Si quiere habilitar la descarga de actualizaciones antes de que se lleve a cabo una implementación de actualizaciones, las implementaciones de actualizaciones pueden ejecutarse de manera más rápida y eficaz, con menos probabilidades de exceder la ventana de mantenimiento.

Para encontrar recomendaciones adicionales sobre cómo configurar WSUS en la suscripción de Azure y mantener actualizadas las máquinas virtuales de Windows de forma segura, revise [Planeamiento de la implementación para actualizar máquinas virtuales Windows en Azure con WSUS](/azure/architecture/example-scenario/wsus/).

## <a name="pre-download-updates"></a>Actualizaciones previas a la descarga

Para configurar la descarga automática de las actualizaciones sin instalarlas automáticamente, puede usar una directiva de grupo para [Configurar actualizaciones automáticas](/windows-server/administration/windows-server-update-services/deploy/4-configure-group-policy-settings-for-automatic-updates##configure-automatic-updates) en 3. Esta configuración habilita la descarga de las actualizaciones necesarias en segundo plano y le notifica que las actualizaciones están listas para instalarse. De este modo, Update Management mantiene el control de las programaciones, pero permite descargar actualizaciones fuera de la ventana de mantenimiento de Update Management. Este comportamiento evita errores `Maintenance window exceeded` en Update Management.

Puede habilitar esta configuración en PowerShell:

```powershell
$WUSettings = (New-Object -com "Microsoft.Update.AutoUpdate").Settings
$WUSettings.NotificationLevel = 3
$WUSettings.Save()
```

## <a name="configure-reboot-settings"></a>Configuración de las opciones de reinicio

Las claves del Registro que se enumeran en [Configuración de actualizaciones automáticas mediante la edición del Registro](/windows/deployment/update/waas-wu-settings#configuring-automatic-updates-by-editing-the-registry) y [Claves del Registro usadas para administrar reinicios](/windows/deployment/update/waas-restart#registry-keys-used-to-manage-restart) pueden hacer que las máquinas se reinicien, incluso si ha especificado **No reiniciar nunca** en la configuración de **Implementación de actualizaciones**. Debe configurar estas claves del Registro para adaptarse a su entorno.

## <a name="enable-updates-for-other-microsoft-products"></a>Habilitar actualizaciones de otros productos de Microsoft

De forma predeterminada, el cliente de Windows Update está configurado para proporcionar actualizaciones solo para Windows. Si habilita la opción **Ofrecerme actualizaciones para otros productos de Microsoft cuando actualice Windows**, también recibirá actualizaciones para otros productos, incluidas las revisiones de seguridad para Microsoft SQL Server u otro software de Microsoft. Puede configurar esta opción si ha descargado y copiado los [archivos de plantillas administrativas](https://support.microsoft.com/help/3087759/how-to-create-and-manage-the-central-store-for-group-policy-administra) más recientes disponibles para Windows 2016 y versiones posteriores.

Si tiene máquinas que ejecutan Windows Server 2012 R2, no puede configurar esta opción mediante directiva de grupo. Ejecute el comando de PowerShell siguiente en estas máquinas:

```powershell
$ServiceManager = (New-Object -com "Microsoft.Update.ServiceManager")
$ServiceManager.Services
$ServiceID = "7971f918-a847-4430-9279-4a52d1efe18d"
$ServiceManager.AddService2($ServiceId,7,"")
```

## <a name="make-wsus-configuration-settings"></a>Establecer valores de configuración de WSUS

Update Management es compatible con la configuración de WSUS. Puede especificar orígenes para examinar y descargar actualizaciones mediante las instrucciones de [Ubicación del servicio Microsoft Update en la intranet](/windows/deployment/update/waas-wu-settings#specify-intranet-microsoft-update-service-location). De forma predeterminada, el cliente de Windows Update está configurado para descargar las actualizaciones desde Windows Update. Cuando se especifica un servidor de WSUS como origen para las máquinas, si las actualizaciones no se aprueban en WSUS, se produce un error en la implementación de la actualización. 

Para restringir las máquinas al servicio de actualización interno, establezca [No conectar a ubicaciones de Internet de Windows Update](/windows-server/administration/windows-server-update-services/deploy/4-configure-group-policy-settings-for-automatic-updates#do-not-connect-to-any-windows-update-internet-locations). 

## <a name="next-steps"></a>Pasos siguientes

Programe una implementación de actualizaciones siguiendo las instrucciones de [Administración de actualizaciones y revisiones para las máquinas virtuales](update-mgmt-manage-updates-for-vm.md).
