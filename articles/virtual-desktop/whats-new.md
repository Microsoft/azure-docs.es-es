---
title: Novedades de Windows Virtual Desktop - Azure
description: Nuevas características y actualizaciones de productos para Windows Virtual Desktop.
services: virtual-desktop
author: Heidilohr
ms.service: virtual-desktop
ms.topic: overview
ms.date: 06/15/2020
ms.author: helohr
ms.reviewer: thhickli; darank
manager: lizross
ms.openlocfilehash: 9be1053600ab89a7879a04a7c08a44ddf3bc862a
ms.sourcegitcommit: dccb85aed33d9251048024faf7ef23c94d695145
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/28/2020
ms.locfileid: "87291233"
---
# <a name="whats-new-in-windows-virtual-desktop"></a>Novedades de Windows Virtual Desktop

Windows Virtual Desktop se actualiza de forma periódica. En este artículo encontrará información sobre:

- Las actualizaciones más recientes
- Nuevas características
- Mejoras de las características existentes
- Corrección de errores

Este artículo se actualiza mensualmente. Asegúrese de volver aquí con frecuencia para mantenerse al día con las nuevas actualizaciones.

## <a name="june-2020"></a>Junio de 2020

El mes pasado, presentamos la versión preliminar de la integración de Windows Virtual Desktop con Azure Resource Manager. Esta actualización tiene muchas características nuevas y atractivas de las que nos encantaría informarle. Estas son las novedades de esta versión de Windows Virtual Desktop.

### <a name="windows-virtual-desktop-is-now-integrated-with-azure-resource-manager-preview"></a>Windows Virtual Desktop ahora se integra con Azure Resource Manager (versión preliminar)

Windows Virtual Desktop ahora se integra en Azure Resource Manager. En la última actualización, todos los objetos de Windows Virtual Desktop pasan a ser recursos de Azure Resource Manager. Esta actualización también se integra con los controles de acceso basado en rol (RBAC) de Azure. Para más información, consulte [¿Qué es Azure Resource Manager?](../azure-resource-manager/management/overview.md).

Esto es lo que hace este cambio:

- Windows Virtual Desktop ahora está integrado en Azure Portal, lo que significa que puede administrar todo directamente en el portal, no se requieren PowerShell, aplicaciones web o herramientas de terceros. Para empezar, consulte nuestro tutorial en [Creación de un grupo de hosts con Azure Portal](create-host-pools-azure-marketplace.md).

- Antes de esta actualización, solo podía publicar RemoteApps y Desktops para usuarios individuales. Con Azure Resource Manager, ahora puede publicar recursos en grupos de Azure Active Directory.

- La versión anterior de Windows Virtual Desktop tenía cuatro roles de administrador integrados que podría asignar a un inquilino o a un grupo de hosts. Estos roles están ahora en el [control de acceso basado en rol](../role-based-access-control/overview.md) de Azure. Estos roles se pueden a todos los objetos de Azure Resource Manager de Windows Virtual Desktop, lo que le permite tener un modelo completo de delegación enriquecida.

- En esta actualización, ya no preciso ejecutar Azure Marketplace o la plantilla de GitHub varias veces para expandir un grupo de hosts. Lo único que se necesita para expandir un grupo de hosts es ir a él en Azure Portal y seleccionar **+ Agregar** para implementar hosts de sesión adicionales.

- Ahora, la implementación de los grupos de hosts está completamente integrada en [Azure Shared Image Gallery](../virtual-machines/windows/shared-image-galleries.md). Shared Image Gallery es un servicio de Azure independiente que almacena definiciones de imágenes de máquina virtual (VM), incluidas las versiones de las imágenes. También puede usar la replicación global para copiar y enviar las imágenes a otras regiones de Azure para su implementación local.

- Las funciones de supervisión que solían realizarse a través de PowerShell o de la aplicación web del servicio de diagnóstico se han desplazado a Log Analytics en Azure Portal. Ahora también tiene dos opciones para visualizar los informes. Puede ejecutar consultas Kusto y usar Workbooks para crear informes visuales.

- Ya no es preciso completar el consentimiento de Azure Active Directory (Azure AD) para usar Windows Virtual Desktop. En esta actualización, el inquilino de Azure AD de su suscripción de Azure autentica a los usuarios y proporciona controles RBAC a los administradores.


### <a name="powershell-support"></a>Compatibilidad con PowerShell

Hemos agregado nuevos cmdlets AzWvd al módulo Az de Azure PowerShell con esta actualización. Este nuevo módulo es compatible con PowerShell Core, que se ejecuta en .NET Core.

Para instalar el módulo, siga las instrucciones de [Configuración del módulo de PowerShell para Windows Virtual Desktop](powershell-module.md).

También puede ver una lista de los comandos disponibles en la [referencia de AzWvd PowerShell](/powershell/module/az.desktopvirtualization/?view=azps-4.2.0#desktopvirtualization).

Para más información acerca de las nuevas características, consulte [nuestra entrada del blog](https://techcommunity.microsoft.com/t5/itops-talk-blog/windows-virtual-desktop-spring-update-enters-public-preview/ba-p/1340245). 

### <a name="additional-gateways"></a>Puertas de enlace adicionales

Hemos agregado un nuevo clúster de puerta de enlace en Sudáfrica para reducir la latencia de conexión.

### <a name="microsoft-teams-on-windows-virtual-desktop-preview"></a>Microsoft Teams en Windows Virtual Desktop (versión preliminar)

Hemos realizado algunas mejoras en Microsoft Teams para Windows Virtual Desktop. Y lo que es más importante, Windows Virtual Desktop ahora admite el redireccionamiento de audio y visual para las llamadas. El redireccionamiento mejora la latencia, ya que crea rutas de acceso directas entre usuarios cuando llaman mediante audio o vídeo. Una Menor distancia significa menos saltos, lo que mejora tanto la imagen como el sonido en las llamadas.

Para más información, consulte nuestra [entrada del blog](https://azure.microsoft.com/updates/windows-virtual-desktop-media-optimization-for-microsoft-teams-is-now-available-in-public-preview/).

## <a name="next-steps"></a>Pasos siguientes

Obtenga información sobre el futuro de [Windows Virtual Desktop en el plan de desarrollo de Microsoft 365](https://www.microsoft.com/microsoft-365/roadmap?filters=Windows%20Virtual%20Desktop).

Consulte estos artículos para obtener información acerca de las actualizaciones de nuestros clientes para Windows Virtual Desktop y Servicios de Escritorio remoto:

- [Windows](/windows-server/remote/remote-desktop-services/clients/windowsdesktop-whatsnew)
- [macOS](/windows-server/remote/remote-desktop-services/clients/mac-whatsnew)
- [iOS](/windows-server/remote/remote-desktop-services/clients/ios-whatsnew)
- [Android](/windows-server/remote/remote-desktop-services/clients/android-whatsnew)
- [Web](/windows-server/remote/remote-desktop-services/clients/web-client-whatsnew)
