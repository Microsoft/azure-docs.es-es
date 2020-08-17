---
title: Preguntas frecuentes sobre la sesión múltiple de Windows 10 Enterprise - Azure
description: Preguntas frecuentes y procedimientos recomendados para usar la sesión múltiple de Windows 10 Enterprise para Windows Virtual Desktop.
author: Heidilohr
ms.topic: conceptual
ms.date: 02/19/2020
ms.author: helohr
manager: lizross
ms.openlocfilehash: 91c7c19fddab2a4372f183c90532b24e774277d1
ms.sourcegitcommit: 98854e3bd1ab04ce42816cae1892ed0caeedf461
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/07/2020
ms.locfileid: "88008056"
---
# <a name="windows-10-enterprise-multi-session-faq"></a>Preguntas frecuentes sobre la sesión múltiple de Windows 10 Enterprise

En este artículo se responden preguntas frecuentes y se explican procedimientos recomendados para la sesión múltiple de Windows 10 Enterprise.

## <a name="what-is-windows-10-enterprise-multi-session"></a>¿Qué es la sesión múltiple de Windows 10 Enterprise?

La sesión múltiple de Windows 10 Enterprise, anteriormente conocida como Windows 10 Enterprise para escritorios virtuales (EVD), es un nuevo host de sesión de Escritorio remoto que permite varias sesiones interactivas simultáneas, lo que anteriormente solo era posible con Windows Server. Esta funcionalidad proporciona a los usuarios una experiencia conocida de Windows 10, mientras que el departamento de TI puede beneficiarse de las ventajas en los costos de la sesión múltiple y usar licencias de Windows por usuario existentes en lugar de licencias de acceso de cliente (CAL) de RDS. Para más información sobre licencias y precios, consulte [Precios de Windows Virtual Desktop](https://azure.microsoft.com/pricing/details/virtual-desktop/).

## <a name="how-many-users-can-simultaneously-have-an-interactive-session-on-windows-10-enterprise-multi-session"></a>¿Cuántos usuarios pueden tener simultáneamente una sesión interactiva en la sesión múltiple de Windows 10 Enterprise?

Cuántas sesiones interactivas que pueden estar activas al mismo tiempo dependen de los recursos de hardware del sistema (vCPU, memoria, disco y vGPU), cómo utilizan los usuarios las aplicaciones mientras tienen iniciada una sesión y cómo es la carga de trabajo del sistema. Se recomienda que valide el rendimiento del sistema para saber cuántos usuarios puede tener en la sesión múltiple de Windows 10 Enterprise. Para más información, consulte [Precios de Windows Virtual Desktop](https://azure.microsoft.com/pricing/details/virtual-desktop/).

## <a name="why-does-my-application-report-windows-10-enterprise-multi-session-as-a-server-operating-system"></a>¿Por qué mi aplicación indica la sesión múltiple de Windows 10 Enterprise como un sistema operativo Windows Server?

La sesión múltiple de Windows 10 Enterprise es una edición virtual de Windows 10 Enterprise. Una de las diferencias es que este sistema operativo indica [ProductType](/windows/win32/cimwin32prov/win32-operatingsystem) con un valor de 3, el mismo valor que Windows Server. Esta propiedad garantiza la compatibilidad del sistema operativo con las herramientas de administración de RDSH existentes, con aplicaciones de RDSH compatibles con sesión múltiple y con optimizaciones de rendimiento del sistema principalmente de bajo nivel para entornos de RDSH. Algunos instaladores de aplicaciones pueden bloquear la instalación en la sesión múltiple de Windows 10 en función de si detectan que ProductType está establecido en Client. Si la aplicación no se instala, póngase en contacto con el proveedor de la aplicación para obtener una versión actualizada.

## <a name="can-i-run-windows-10-enterprise-multi-session-on-premises"></a>¿Es posible ejecutar la sesión múltiple de Windows 10 Enterprise en un entorno local?

La sesión múltiple de Windows 10 Enterprise no se puede ejecutar en entornos de producción locales porque está optimizada para el servicio Windows Virtual Desktop para Azure. La ejecución de la sesión múltiple de Windows 10 Enterprise fuera de Azure para fines de producción supone un incumplimiento del contrato de licencia. La sesión múltiple de Windows 10 Enterprise no se activará en servicios de administración de claves (KMS) locales.

## <a name="how-do-i-customize-the-windows-10-enterprise-multi-session-image-for-my-organization"></a>¿Cómo puedo personalizar la imagen de la sesión múltiple de Windows 10 Enterprise para mi organización?

Puede iniciar una máquina virtual en Azure con la sesión múltiple de Windows 10 Enterprise de Windows 10 y personalizarla mediante la instalación de aplicaciones de línea de negocio, usar el comando sysprep/generalize y, a continuación, crear una imagen en Azure Portal.

Para empezar, cree una VM en Azure con la sesión múltiple de Windows 10 Enterprise. En lugar de iniciar la máquina virtual en Azure, puede descargar el disco duro virtual directamente. Después, podrá usar el disco duro virtual que ha descargado para crear una nueva máquina virtual de generación 1 en un equipo con Windows 10 con Hyper-V habilitado.

Personalice la imagen según sus necesidades instalando aplicaciones de línea de negocio y prepare la imagen con sysprep. Cuando haya terminado de personalizarla, cargue la imagen en Azure con el disco duro virtual dentro. Después, obtenga Windows Virtual Desktop de Azure Marketplace y úselo para implementar un nuevo grupo de hosts con la imagen personalizada.

## <a name="how-do-i-manage-windows-10-enterprise-multi-session-after-deployment"></a>¿Cómo administrar la sesión múltiple de Windows 10 Enterprise después de la implementación?

Puede usar cualquier herramienta de configuración compatible, pero se recomienda Configuration Manager 1906 porque es compatible con la sesión múltiple de Windows 10 Enterprise. Actualmente se está trabajando para garantizar la compatibilidad con Microsoft Intune.

## <a name="can-windows-10-enterprise-multi-session-be-azure-active-directory-ad-joined"></a>¿Puede unirse la sesión múltiple de Windows 10 Enterprise a Azure Active Directory (AD)?

Actualmente, se admite unir la sesión múltiple de Windows 10 Enterprise a Azure AD híbrido. Una vez unida la sesión múltiple de Windows 10 Enterprise a un dominio, use el objeto de directiva de grupo existente para habilitar el registro de Azure AD. Para más información, consulte [Planeamiento de la implementación de la unión a Azure Active Directory híbrido](../active-directory/devices/hybrid-azuread-join-plan.md).

## <a name="where-can-i-find-the-windows-10-enterprise-multi-session-image"></a>¿Dónde se puede encontrar la imagen de la sesión múltiple de Windows 10 Enterprise?

La sesión múltiple de Windows 10 Enterprise se encuentra en la galería de Azure. Para encontrarla, vaya a Azure Portal y busque la versión Windows 10 Enterprise para escritorios virtuales. En el caso de una imagen integrada con aplicaciones de Microsoft 365 para empresas, vaya al Azure Portal y busque **Microsoft Windows 10 + Aplicaciones de Microsoft 365 para empresas**.

## <a name="which-windows-10-enterprise-multi-session-image-should-i-use"></a>¿Qué imagen de sesión múltiple de Windows 10 Enterprise debo usar?

La galería de Azure tiene varias versiones, entre las que se incluyen la sesión múltiple de Windows 10 Enterprise, versión 1809, y la sesión múltiple de Windows 10 Enterprise, versión 1903. Se recomienda usar la versión más reciente para un mejor rendimiento y fiabilidad.

## <a name="which-windows-10-enterprise-multi-session-versions-are-supported"></a>¿Qué versiones de sesión múltiple de Windows 10 Enterprise se admiten?

Se admiten la versión 1809 y versiones posteriores de la sesión múltiple de Windows 10 Enterprise, y están disponibles en la galería de Azure. Estas versiones siguen la misma directiva de ciclo de vida de compatibilidad que Windows 10 Enterprise, lo que significa que la versión de marzo es compatible durante 18 meses y la versión de septiembre, durante 30 meses.

## <a name="which-profile-management-solution-should-i-use-for-windows-10-enterprise-multi-session"></a>¿Qué solución de administración de perfiles debo usar para la sesión múltiple de Windows 10 Enterprise?

Se recomienda usar contenedores de perfiles de FSLogix al configurar Windows 10 Enterprise en entornos no persistentes u otros escenarios que requieran un perfil almacenado centralmente. FSLogix garantiza que el perfil de usuario esté disponible y actualizado para cada sesión de usuario. También se recomienda usar el contenedor de perfiles de FSLogix para almacenar un perfil de usuario en cualquier recurso compartido de SMB con los permisos adecuados, pero puede almacenar perfiles de usuario en el almacenamiento de blobs en páginas de Azure, si es necesario. Los usuarios de Windows Virtual Desktop pueden usar FSLogix sin costo adicional.

Para obtener más información sobre cómo configurar un contenedor de perfiles de FSLogix, consulte [Configurar el contenedor de perfiles de FSLogix](create-host-pools-user-profile.md#configure-the-fslogix-profile-container).

## <a name="which-license-do-i-need-to-access-windows-10-enterprise-multi-session"></a>¿Qué licencia se necesita para acceder a la sesión múltiple de Windows 10 Enterprise?

Para ver una lista completa de licencias aplicables, consulte [Precios de Windows Virtual Desktop](https://azure.microsoft.com/pricing/details/virtual-desktop/).

## <a name="why-do-my-apps-disappear-after-i-sign-out"></a>¿Por qué las aplicaciones desaparecen después de cerrar sesión?

Esto sucede porque usa la sesión múltiple de Windows 10 Enterprise con una solución de administración de perfiles como FSLogix. La solución de administrador o de perfiles ha configurado el sistema para eliminar los perfiles de usuario cuando estos cierren sesión. Esta configuración significa que cuando el sistema elimina su perfil de usuario después de cerrar la sesión, también quita todas las aplicaciones que haya instalado durante la sesión. Si desea mantener las aplicaciones instaladas, deberá pedir al administrador que aprovisione estas aplicaciones para todos los usuarios del entorno de Windows Virtual Desktop.

## <a name="how-do-i-make-sure-apps-dont-disappear-when-users-sign-out"></a>¿Cómo asegurarse de que las aplicaciones no desaparecen cuando los usuarios cierran sesión?

La mayoría de los entornos virtualizados están configurados de forma predeterminada para evitar que los usuarios instalen aplicaciones adicionales en sus perfiles. Si quiere asegurarse de que una aplicación no desaparece cuando el usuario cierra sesión en Windows Virtual Desktop, tiene que aprovisionar la aplicación para todos los perfiles de usuario de su entorno. Para más información sobre cómo aprovisionar las aplicaciones, consulte estos recursos.

- [Publicar aplicaciones integradas en Windows Virtual Desktop](publish-apps.md)
- [Opciones de la línea de comandos de mantenimiento de paquetes de aplicaciones DISM](https://docs.microsoft.com/windows-hardware/manufacture/desktop/dism-app-package--appx-or-appxbundle--servicing-command-line-options)
- [Add-AppxProvisionedPackage](https://docs.microsoft.com/powershell/module/dism/add-appxprovisionedpackage?view=win10-ps)

## <a name="how-do-i-make-sure-users-dont-download-and-install-apps-from-the-microsoft-store"></a>¿Cómo asegurarse de que los usuarios no descarguen e instalen aplicaciones de Microsoft Store?

Puede deshabilitar la aplicación Microsoft Store para asegurarse de que los usuarios no descarguen aplicaciones adicionales más allá de las aplicaciones que ya ha aprovisionado para ellos.

Para deshabilitar la aplicación de la tienda:

1. Cree una nueva directiva de grupo.
2. Seleccione **Configuración del equipo** > **Plantillas administrativas** > **Componentes de Windows**.
3. Seleccione **Tienda**.
4. Seleccione la **aplicación de la tienda**.
5. Seleccione **Deshabilitado** y, después, **Aceptar**.
6. Seleccione **Aplicar**.

## <a name="next-steps"></a>Pasos siguientes

Para más información sobre Windows Virtual Desktop y la sesión múltiple de Windows 10 Enterprise:

- Lea la [documentación sobre Windows Virtual Desktop](overview.md).
- Visite el espacio de [TechCommunity sobre Windows Virtual Desktop](https://techcommunity.microsoft.com/t5/Windows-Virtual-Desktop/bd-p/WindowsVirtualDesktop).
- Configure la implementación de Windows Virtual Desktop con los [tutoriales de Windows Virtual Desktop](./virtual-desktop-fall-2019/tenant-setup-azure-active-directory.md).
