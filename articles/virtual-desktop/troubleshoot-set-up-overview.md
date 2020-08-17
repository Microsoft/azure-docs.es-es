---
title: 'Introducción a la solución de problemas de Windows Virtual Desktop: Azure'
description: Información general para solucionar problemas al configurar un entorno de Windows Virtual Desktop.
author: Heidilohr
ms.topic: troubleshooting
ms.date: 06/05/2020
ms.author: helohr
manager: lizross
ms.openlocfilehash: 171322fbec099aa9fafc36c4f951213b33007976
ms.sourcegitcommit: 98854e3bd1ab04ce42816cae1892ed0caeedf461
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/07/2020
ms.locfileid: "88009365"
---
# <a name="troubleshooting-overview-feedback-and-support-for-windows-virtual-desktop"></a>Información general, comentarios y soporte técnico para la solución de problemas de Windows Virtual Desktop

>[!IMPORTANT]
>Este contenido se aplica a Windows Virtual Desktop con objetos de Windows Virtual Desktop para Azure Resource Manager. Si usa Windows Virtual Desktop (clásico) sin objetos para Azure Resource Manager, consulte [este artículo](./virtual-desktop-fall-2019/troubleshoot-set-up-overview-2019.md).

En este artículo se proporciona información general de los problemas que pueden surgir al configurar un entorno de Windows Virtual Desktop y proporciona distintas maneras de resolver los problemas.

## <a name="report-issues"></a>Informar de problemas

Para notificar problemas o sugerir características para Windows Virtual Desktop con la integración de Azure Resource Manager, visite la [comunidad técnica de Windows Virtual Desktop](https://techcommunity.microsoft.com/t5/Windows-Virtual-Desktop/bd-p/WindowsVirtualDesktop). Puede usar la comunidad tecnológica para analizar los procedimientos recomendados o sugerir y votar nuevas características.

Si realiza una publicación para solicitar ayuda o proponer una nueva característica, asegúrese de que describe el tema lo más detalladamente posible. La información detallada puede ayudar a otros usuarios a responder a su pregunta o comprender mejor la característica para la que pide un voto.

## <a name="escalation-tracks"></a>Seguimiento del escalado

Antes de hacer nada más, asegúrese de comprobar la [página de estados de Azure](https://status.azure.com/status) y [Azure Service Health](https://azure.microsoft.com/features/service-health/) para cerciorarse de que el servicio Azure funciona correctamente.

Use la siguiente tabla para identificar y resolver los problemas que puedan surgir al configurar un entorno mediante el cliente de Escritorio remoto. Una vez que el entorno esté configurado, puede usar el nuevo [servicio de diagnóstico](diagnostics-role-service.md) para identificar los problemas en escenarios comunes.

| **Problema**                                                            | **Solución propuesta**  |
|----------------------------------------------------------------------|-------------------------------------------------|
| Configuración de Express Route y de Azure Virtual Network (VNET) en el grupo del host de sesión               | [Abra una solicitud de soporte técnico de Azure](https://azure.microsoft.com/support/create-ticket/) y, a continuación, seleccione el servicio adecuado (en la categoría Redes). |
| Creación de una máquina virtual en el grupo del host de sesión cuando las plantilla de Azure Resource Manager que se proporcionan con Windows Virtual Desktop no se usan | [Abra una solicitud de soporte técnico de Azure](https://azure.microsoft.com/support/create-ticket/) y, a continuación, seleccione el servicio **Windows Virtual Desktop**. <br> <br> En caso de problemas con las plantillas de Azure Resource Manager que se proporcionan con Windows Virtual Desktop, consulte la sección Errores de plantilla de Azure Resource Manager de [Creación de grupos de host](troubleshoot-set-up-issues.md). |
| Administración del entorno del host de sesión de Windows Virtual Desktop desde Azure Portal    | [Abra una solicitud de soporte técnico de Azure](https://azure.microsoft.com/support/create-ticket/). <br> <br> En el caso de los problemas de administración cuando se usa PowerShell para Servicios de Escritorio remoto/Windows Virtual Desktop, consulte [PowerShell para Windows Virtual Desktop](troubleshoot-powershell.md) o [abra una solicitud de soporte técnico de Azure](https://azure.microsoft.com/support/create-ticket/), seleccione el servicio **Windows Virtual Desktop**, el tipo de problema **Configuración y administración** y, a continuación, el subtipo de problema **Problemas al configurar el entorno con PowerShell**. |
| Administración de la configuración de Windows Virtual Desktop asociada a los grupos de hosts y a los grupos de aplicaciones      | Consulte [PowerShell para Windows Virtual Desktop](troubleshoot-powershell.md) o [abra una solicitud de soporte técnico de Azure](https://azure.microsoft.com/support/create-ticket/), seleccione el servicio **Windows Virtual Desktop** y, a continuación, el tipo de problema adecuado.|
| Implementación y administración de contenedores de perfiles de FSLogix | Consulte la [guía de solución de problemas para productos de FSLogix](/fslogix/fslogix-trouble-shooting-ht/) y, si no logra solucionar el problema, [abra una solicitud de soporte técnico de Azure](https://azure.microsoft.com/support/create-ticket/), seleccione el servicio **Windows Virtual Desktop**, el tipo de problema **FSLogix** y, a continuación, el subtipo de problema adecuado. |
| Error de funcionamiento de los clientes de Escritorio remoto en el inicio                                                 | Consulte [Solución de problemas del cliente de Escritorio remoto](troubleshoot-client.md) y, si no logra resolver el problema, [abra una solicitud de soporte técnico de Azure](https://azure.microsoft.com/support/create-ticket/), seleccione el servicio **Windows Virtual Desktop** y, a continuación, el tipo de problema **Remote Desktop clients** (Clientes de Escritorio remoto).  <br> <br> Si es un problema de red, los usuarios deben ponerse en contacto con su administrador de red. |
| Está conectado, pero no hay ninguna fuente                                                                 | Para solucionar el problema, consulte la sección [El usuario se conecta, pero no se muestra nada (ninguna fuente)](troubleshoot-service-connection.md#user-connects-but-nothing-is-displayed-no-feed) de [Conexiones de servicios de Windows Virtual Desktop](troubleshoot-service-connection.md). <br> <br> Si sus usuarios se han asignado a un grupo de aplicaciones, [abra una solicitud de soporte técnico de Azure](https://azure.microsoft.com/support/create-ticket/), seleccione el servicio **Windows Virtual Desktop** y, a continuación, el tipo de problema **Remote Desktop clients** (Clientes de Escritorio remoto). |
| Problemas de detección de fuentes debido a la red                                            | Los usuarios deben ponerse en contacto con su administrador de red. |
| Conexión de clientes                                                                    | Consulte [Conexiones de servicios de Windows Virtual Desktop](troubleshoot-service-connection.md) y, si no se soluciona el problema, consulte [Configuración de máquina virtual de host de sesión](troubleshoot-vm-configuration.md). |
| Capacidad de respuesta del escritorio o de las aplicaciones remotas                                      | Si los problemas están asociados a una aplicación o producto específicos, póngase en contacto con el equipo responsable de dicho producto. |
| Errores o mensajes de las licencias                                                          | Si los problemas están asociados a una aplicación o producto específicos, póngase en contacto con el equipo responsable de dicho producto. |
| Problemas con los métodos de autenticación de terceros | Compruebe que su proveedor de terceros admite escenarios de Windows Virtual Desktop y contacte con ellos en relación con cualquier problema conocido. |
| Problemas en el uso de Log Analytics para Windows Virtual Desktop | Para problemas relacionados con el esquema de diagnóstico, [abra una solicitud de soporte técnico de Azure](https://azure.microsoft.com/support/create-ticket/).<br><br>Para consultas, problemas de visualización o de cualquier otro tipo en Log Analytics, seleccione el tipo de problema adecuado en Log Analytics. |
| Problemas con el uso de las aplicaciones de M365 | Póngase en contacto con el centro de administración de M365 con una de las [opciones de ayuda del centro de administración de M365](/microsoft-365/admin/contact-support-for-business-products/). |

## <a name="next-steps"></a>Pasos siguientes

- Para solucionar problemas durante la creación de un grupo de hosts en un entorno de Windows Virtual Desktop, consulte [Creación de grupos de host](troubleshoot-set-up-issues.md).
- Para solucionar problemas al configurar una máquina virtual (VM) en Windows Virtual Desktop, consulte [Configuración de la máquina virtual del host de sesión](troubleshoot-vm-configuration.md).
- Para solucionar problemas con conexiones de cliente de Windows Virtual Desktop, consulte [Conexiones de servicios de Windows Virtual Desktop](troubleshoot-service-connection.md).
- Para solucionar problemas con los clientes de Escritorio remoto, consulte [Solución de problemas del cliente de Escritorio remoto](troubleshoot-client.md).
- Para solucionar problemas al usar PowerShell con Windows Virtual Desktop, consulte [PowerShell para Windows Virtual Desktop](troubleshoot-powershell.md).
- Para más información sobre el servicio, consulte [Entorno de Windows Virtual Desktop](environment-setup.md).
- Para realizar un tutorial de solución de problemas, consulte [Tutorial: Solución de problemas de las implementaciones de plantillas de Resource Manager](../azure-resource-manager/templates/template-tutorial-troubleshoot.md).
- Para más información sobre las acciones de auditoría, consulte [Operaciones de auditoría con Resource Manager](../azure-resource-manager/management/view-activity-logs.md).
- Para obtener información sobre las acciones que permiten determinar los errores que se producen durante la implementación, consulte [Visualización de operaciones de implementación](../azure-resource-manager/templates/deployment-history.md).
