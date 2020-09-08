---
title: Novedades de Windows Virtual Desktop - Azure
description: Nuevas características y actualizaciones de productos para Windows Virtual Desktop.
author: Heidilohr
ms.topic: overview
ms.date: 09/02/2020
ms.author: helohr
ms.reviewer: thhickli; darank
manager: lizross
ms.custom: references_regions
ms.openlocfilehash: d698470f450f6fe903ab68334764e0918d659d7f
ms.sourcegitcommit: 5ed504a9ddfbd69d4f2d256ec431e634eb38813e
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/02/2020
ms.locfileid: "89319821"
---
# <a name="whats-new-in-windows-virtual-desktop"></a>Novedades de Windows Virtual Desktop

Windows Virtual Desktop se actualiza de forma periódica. En este artículo encontrará información sobre:

- Las actualizaciones más recientes
- Nuevas características
- Mejoras de las características existentes
- Corrección de errores

Este artículo se actualiza mensualmente. Asegúrese de volver aquí con frecuencia para mantenerse al día con las nuevas actualizaciones.

## <a name="august-2020"></a>Agosto de 2020

Estos son los cambios que se han producido en agosto de 2020:

- Hemos mejorado el rendimiento para reducir la latencia de conexión en las siguientes regiones de Azure: 

    - Reino Unido
    - Francia
    - Noruega
    - Corea del Sur

   Puede usar el [Estimador de experiencia](https://azure.microsoft.com/services/virtual-desktop/assessment/) para obtener una idea general de la forma en que estos cambios afectarán a sus usuarios.

- El cliente de Escritorio remoto de Microsoft Store (v 10.2.1522 +) ya está disponible con carácter general. Esta versión del cliente de Escritorio remoto de Microsoft Store es compatible con Windows Virtual Desktop. También hemos incorporado flujos de interfaz de usuario actualizados para mejorar las experiencias de los usuarios. Esta actualización incluye un diseño fluido, modos claros y oscuros, y muchos otros cambios atractivos. También hemos reescrito el cliente para que use el mismo motor de protocolo de escritorio remoto (RDP) subyacente que los clientes iOS, macOS y Android, lo que nos permite ofrecer nuevas características a mayor velocidad en todas las plataformas. [Descargue el cliente](https://www.microsoft.com/p/microsoft-remote-desktop/9wzdncrfj3ps?rtc=1&activetab=pivot:overviewtab) y pruébelo.

- Hemos solucionado un problema en el cliente de escritorio de Teams (versión 1.3.00.21759) en el que el cliente solo mostraba la zona horaria UTC en el chat, los canales y el calendario. El cliente actualizado muestra ahora la zona horaria de la sesión remota.

- Ahora, Azure Advisor forma parte de Windows Virtual Desktop. Al acceder a Windows Virtual Desktop a través de Azure Portal, puede ver recomendaciones para optimizar el entorno de Windows Virtual Desktop. Más información en [Azure Advisor](azure-advisor.md).

- La CLI de Azure ahora admite Windows Virtual Desktop (`az desktopvirtualization`) para ayudarle a automatizar las implementaciones de Windows Virtual Desktop. Consulte [desktopvirtualization](/cli/azure/ext/desktopvirtualization/?view=azure-cli-latest) para ver una lista de los comandos de la extensión.

- Hemos actualizado nuestras plantillas de implementación para que sean totalmente compatibles con las interfaces de Azure Resource Manager de Windows Virtual Desktop. Puede encontrar las plantillas en [GitHub](https://github.com/Azure/RDS-Templates/tree/master/ARM-wvd-templates).

- El portal de US Gov de Windows Virtual Desktop se encuentra ahora en versión preliminar pública. Para más información, consulte [nuestro anuncio](https://azure.microsoft.com/updates/windows-virtual-desktop-is-now-available-in-the-azure-government-cloud-in-preview/).

## <a name="july-2020"></a>Julio de 2020  

En julio se ha iniciado la disponibilidad con carácter general de Windows Virtual Desktop con integración de Azure Resource Management.

Esto es lo que ha cambiado con esta nueva versión: 

- La "versión Otoño 2019" se conoce ahora como "Windows Virtual Desktop (clásico)", mientras que la "versión de primavera de 2020" es ahora simplemente "Windows Virtual Desktop". Para más información, consulte [esta entrada de blog](https://azure.microsoft.com/blog/new-windows-virtual-desktop-capabilities-now-generally-available/). 

Para más información acerca de las nuevas características, consulte [esta entrada de blog](https://techcommunity.microsoft.com/t5/itops-talk-blog/windows-virtual-desktop-spring-update-enters-public-preview/ba-p/1340245). 

### <a name="autoscaling-tool-update"></a>Actualización de la herramienta de escalado automático

La versión más reciente de la herramienta de escalado automático que estaba en versión preliminar ya está disponible con carácter general. Esta herramienta usa una cuenta de Azure Automation y la aplicación lógica de Azure para apagar y reiniciar automáticamente las máquinas virtuales del host de sesión dentro de un grupo host, lo que reduce costos de infraestructura. Más información en [Escalado de hosts de sesión con Azure Automation](set-up-scaling-script.md).

### <a name="azure-portal"></a>Azure Portal

Ahora puede hacer lo siguiente con Azure Portal en Windows Virtual Desktop: 

- Asignación directa de usuarios a hosts de sesión de escritorio personal.  
- Cambio de la configuración del entorno de validación para grupos host. 

### <a name="diagnostics"></a>Diagnóstico

Hemos publicado algunas nuevas consultas precompiladas para el área de trabajo de Log Analytics. Para obtener acceso a las consultas, vaya a **Registros** y, en **Categoría**, seleccione **Windows Virtual Desktop**. Más información en [Uso de Log Analytics para la característica de diagnóstico](diagnostics-log-analytics.md).

### <a name="update-for-remote-desktop-client-for-android"></a>Actualización para el cliente de Escritorio remoto para Android

El [cliente de Escritorio remoto para Android](https://play.google.com/store/apps/details?id=com.microsoft.rdc.androidx) ahora admite las conexiones de Windows Virtual Desktop. A partir de la versión 10.0.7, el cliente de Android ofrece una nueva interfaz de usuario para mejorar la experiencia del usuario. El cliente también se integra con Microsoft Authenticator en los dispositivos Android para habilitar el acceso condicional cuando se realiza una suscripción a áreas de trabajo de Windows Virtual Desktop.  

La versión anterior del cliente de Escritorio remoto se denomina ahora "Escritorio remoto 8". Las conexiones que ya tenga en la versión anterior del cliente se transferirán sin problemas al nuevo cliente. El nuevo cliente se ha vuelto a escribir en el mismo motor de RDP Core subyacente que los clientes de iOS y macOS, lo que agiliza la versión de las nuevas características en todas las plataformas. 

### <a name="teams-update"></a>Actualización de Teams

Hemos realizado algunas mejoras en Microsoft Teams para Windows Virtual Desktop. Lo más importante es que Windows Virtual Desktop ahora admite la optimización de audio y vídeo para el cliente del escritorio de Windows. El redireccionamiento mejora la latencia, con la creación de rutas de acceso directas entre usuarios cuando estos realizan llamadas de audio o vídeo y conferencias. Una Menor distancia significa menos saltos, lo que mejora tanto la imagen como el sonido en las llamadas. Más información en [Uso de Teams en Windows Virtual Desktop](teams-on-wvd.md).

## <a name="june-2020"></a>Junio de 2020

El mes pasado, presentamos la versión preliminar de la integración de Windows Virtual Desktop con Azure Resource Manager. Esta actualización tiene muchas características nuevas y atractivas de las que nos encantaría informarle. Estas son las novedades de esta versión de Windows Virtual Desktop.

### <a name="windows-virtual-desktop-is-now-integrated-with-azure-resource-manager"></a>Windows Virtual Desktop ahora se integra con Azure Resource Manager

Windows Virtual Desktop ahora se integra en Azure Resource Manager. En la última actualización, todos los objetos de Windows Virtual Desktop pasan a ser recursos de Azure Resource Manager. Esta actualización también se integra con los controles de acceso basado en rol (Azure RBAC) de Azure. Para más información, consulte [¿Qué es Azure Resource Manager?](../azure-resource-manager/management/overview.md).

Esto es lo que hace este cambio:

- Windows Virtual Desktop ahora está integrado en Azure Portal, lo que significa que puede administrar todo directamente en el portal, no se requieren PowerShell, aplicaciones web o herramientas de terceros. Para empezar, consulte nuestro tutorial en [Creación de un grupo de hosts con Azure Portal](create-host-pools-azure-marketplace.md).

- Antes de esta actualización, solo podía publicar RemoteApps y Desktops para usuarios individuales. Con Azure Resource Manager, ahora puede publicar recursos en grupos de Azure Active Directory.

- La versión anterior de Windows Virtual Desktop tenía cuatro roles de administrador integrados que podría asignar a un inquilino o a un grupo de hosts. Estos roles están ahora en el [Control de acceso basado en rol de Azure (Azure RBAC)](../role-based-access-control/overview.md). Estos roles se pueden a todos los objetos de Azure Resource Manager de Windows Virtual Desktop, lo que le permite tener un modelo completo de delegación enriquecida.

- En esta actualización, ya no preciso ejecutar Azure Marketplace o la plantilla de GitHub varias veces para expandir un grupo de hosts. Lo único que se necesita para expandir un grupo de hosts es ir a él en Azure Portal y seleccionar **+ Agregar** para implementar hosts de sesión adicionales.

- Ahora, la implementación de los grupos de hosts está completamente integrada en [Azure Shared Image Gallery](../virtual-machines/windows/shared-image-galleries.md). Shared Image Gallery es un servicio de Azure independiente que almacena definiciones de imágenes de máquina virtual (VM), incluidas las versiones de las imágenes. También puede usar la replicación global para copiar y enviar las imágenes a otras regiones de Azure para su implementación local.

- Las funciones de supervisión que solían realizarse a través de PowerShell o de la aplicación web del servicio de diagnóstico se han desplazado a Log Analytics en Azure Portal. Ahora también tiene dos opciones para visualizar los informes. Puede ejecutar consultas Kusto y usar Workbooks para crear informes visuales.

- Ya no es preciso completar el consentimiento de Azure Active Directory (Azure AD) para usar Windows Virtual Desktop. En esta actualización, el inquilino de Azure AD de su suscripción de Azure autentica a los usuarios y proporciona controles de Azure RBAC a los administradores.


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

## <a name="client-updates"></a>Actualizaciones de clientes

Consulte estos artículos para obtener información acerca de las actualizaciones de nuestros clientes para Windows Virtual Desktop y Servicios de Escritorio remoto:

- [Windows](/windows-server/remote/remote-desktop-services/clients/windowsdesktop-whatsnew)
- [macOS](/windows-server/remote/remote-desktop-services/clients/mac-whatsnew)
- [iOS](/windows-server/remote/remote-desktop-services/clients/ios-whatsnew)
- [Android](/windows-server/remote/remote-desktop-services/clients/android-whatsnew)
- [Web](/windows-server/remote/remote-desktop-services/clients/web-client-whatsnew)

## <a name="next-steps"></a>Pasos siguientes

Obtenga información sobre el futuro de [Windows Virtual Desktop en el plan de desarrollo de Microsoft 365](https://www.microsoft.com/microsoft-365/roadmap?filters=Windows%20Virtual%20Desktop).

