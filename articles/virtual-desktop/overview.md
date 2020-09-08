---
title: ¿Qué es Windows Virtual Desktop? - Azure
description: Una información general de Windows Virtual Desktop.
author: Heidilohr
ms.topic: overview
ms.date: 08/20/2020
ms.author: helohr
manager: lizross
ms.openlocfilehash: cc5ad91c779a3445712db962fb97bab309eda973
ms.sourcegitcommit: d18a59b2efff67934650f6ad3a2e1fe9f8269f21
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/20/2020
ms.locfileid: "88661119"
---
# <a name="what-is-windows-virtual-desktop"></a>¿Qué es Windows Virtual Desktop?

Windows Virtual Desktop es un servicio de virtualización de escritorio y de aplicaciones que se ejecuta en la nube.

A continuación se indica lo que se puede hacer al ejecutar Windows Virtual Desktop en Azure:

* Configurar una implementación de Windows 10 de sesión múltiple que ofrezca toda la funcionalidad de Windows 10 con escalabilidad
* Virtualizar las aplicaciones de Microsoft 365 para la empresa y optimizarlo para ejecutarse en escenarios virtuales multiusuario
* Proporcionar escritorios virtuales de Windows 7 con actualizaciones gratuitas de seguridad ampliada
* Llevar sus aplicaciones y escritorios existentes de Servicios de Escritorio remoto (RDS) y Windows Server a cualquier equipo
* Virtualizar escritorios y aplicaciones
* Administrar escritorios y aplicaciones de Windows 10, Windows Server y Windows 7 con una experiencia de administración unificada

## <a name="introductory-video"></a>Vídeo de introducción

En este vídeo, obtendrá información sobre Windows Virtual Desktop, y conocerá no solo por qué es único, sino también cuáles son sus novedades:

<br></br><iframe src="https://www.youtube.com/embed/NQFtI3JLtaU" width="640" height="320" allowFullScreen="true" frameBorder="0"></iframe>

Vea más vídeos sobre Windows Virtual Desktop en [nuestra lista de reproducción](https://www.youtube.com/watch?v=NQFtI3JLtaU&list=PLXtHYVsvn_b8KAKw44YUpghpD6lg-EHev).

## <a name="key-capabilities"></a>Principales capacidades

Con Windows Virtual Desktop, puede configurar un entorno flexible y escalable:

* Cree un entorno de virtualización de escritorio completo en su suscripción de Azure sin tener que ejecutar ningún servidor de puerta de enlace adicional.
* Publique todos los grupos host que necesite para dar cabida a diversas cargas de trabajo.
* Aporte su propia imagen para las cargas de trabajo de producción o de prueba desde la Galería de Azure.
* Reduzca los costos con recursos de sesión múltiple agrupados. Con la nueva funcionalidad de sesión múltiple de Windows 10 Enterprise exclusiva de Windows Virtual Desktop y del rol de host de sesión de Escritorio remoto (RDSH) de Windows Server, puede reducir considerablemente el número de máquinas virtuales y la sobrecarga del sistema operativo (SO), mientras se proporcionan los mismos recursos a los usuarios.
* Asigne propiedad individual mediante escritorios personales (permanentes).

Puede implementar y administrar escritorios virtuales:

* Use las interfaces de Azure Portal, Windows Virtual Desktop PowerShell y REST para configurar los grupos de hosts, crear grupos de aplicaciones, asignar usuarios y publicar recursos.
* Publique aplicaciones de escritorio completo o aplicaciones remotas individuales desde un único grupo host, cree grupos de aplicaciones individuales para distintos conjuntos de usuarios o incluso asigne usuarios a varios grupos de aplicaciones para reducir el número de imágenes.
* A medida que administre su entorno, usará el acceso delegado integrado para asignar roles y recopilar diagnósticos con el fin de comprender diversos errores de configuración o de los usuarios.
* Use el nuevo servicio de diagnóstico para solucionar los errores.
* Administre únicamente la imagen y las máquinas virtuales y olvídese de la infraestructura. No es necesario que administre personalmente los roles de Escritorio remoto, como hace con los Servicios de Escritorio remoto, sino solo las máquinas virtuales de su suscripción de Azure.

También puede asignar y conectar a los usuarios a los escritorios virtuales:

* Una vez asignados, los usuarios pueden iniciar cualquier cliente de Windows Virtual Desktop para conectar los usuarios a sus aplicaciones y escritorios de Windows publicados. Conéctese desde cualquier dispositivo mediante una aplicación nativa de su dispositivo o por medio del cliente web HTML5 de Windows Virtual Desktop.
* Establezca de forma segura los usuarios mediante conexiones inversas al servicio, así nunca tendrá que dejar los puertos de entrada abiertos.

## <a name="requirements"></a>Requisitos

Es necesario cumplir una serie de requisitos para configurar Windows Virtual Desktop y conectar correctamente sus usuarios a los escritorios y aplicaciones de Windows.

Admitimos los siguientes sistemas operativos, por lo que debe asegurarse de que tiene las [licencias apropiadas](https://azure.microsoft.com/pricing/details/virtual-desktop/) para sus usuarios en función del escritorio y de las aplicaciones que planee implementar:

|SO|Licencia necesaria|
|---|---|
|Sesión múltiple de Windows 10 Enterprise o Windows 10 Enterprise|Microsoft 365 E3, E5, A3, A5, F3, Business Premium<br>Windows E3, E5, A3, A5|
|Windows 7 Enterprise |Microsoft 365 E3, E5, A3, A5, F3, Business Premium<br>Windows E3, E5, A3, A5|
|Windows Server 2012 R2, 2016, 2019|Licencia de acceso de cliente (CAL) de RDS con Software Assurance|

Su infraestructura necesita cumplir los siguientes requisitos para ser compatible con Windows Virtual Desktop:

* [Azure Active Directory](/azure/active-directory/).
* Una instancia de Windows Server Active Directory sincronizada con Azure Active Directory. Se puede configurar mediante Azure AD Connect (para organizaciones híbridas) o Azure AD Domain Services (para organizaciones híbridas o en la nube).
  * Una instancia de Windows Server AD sincronizada con Azure Active Directory. El usuario tiene como origen Windows Server AD y la máquina virtual con Windows Virtual Desktop está unida al dominio de Windows Server AD.
  * Una instancia de Windows Server AD sincronizada con Azure Active Directory. El usuario tiene como origen Windows Server AD y la máquina virtual con Windows Virtual Desktop está unida al dominio de Azure AD Domain Services.
  * Un dominio de Azure AD Domain Services. El usuario tiene como origen Azure Active Directory y la máquina virtual con Windows Virtual Desktop está unida al dominio de Azure AD Domain Services.
* Una suscripción a Azure, cuyo elemento primario es el mismo inquilino de Azure AD, que contenga una red virtual que contenga Windows Server Active Directory o una instancia de Azure AD DS, o esté conectada a ellos.

Requisitos de usuario para conectarse a Windows Virtual Desktop:

* El origen del usuario debe ser el mismo Active Directory que está conectado a Azure AD. Windows Virtual Desktop no admite cuentas B2B o MSA.
* El nombre principal de usuario que se usa para suscribirse a Windows Virtual Desktop debe existir en el dominio de Active Directory al que se une la máquina virtual.

Las máquinas virtuales de Azure que cree para Windows Virtual Desktop deben cumplir estos requisitos:

* Estar [unidas a un dominio estándar](../active-directory-domain-services/active-directory-ds-comparison.md) o a un [dominio híbrido](../active-directory/devices/hybrid-azuread-join-plan.md). Las máquinas virtuales no pueden estar unidas a Azure AD.
* Deben ejecutar una de las siguientes [imágenes de sistema operativo admitidas](#supported-virtual-machine-os-images).

>[!NOTE]
>Si necesita una suscripción a Azure, puede [registrarse para obtener una evaluación gratuita por un mes](https://azure.microsoft.com/free/). Si usa la versión de evaluación gratuita de Azure, debe utilizar Azure AD Domain Services para mantener sincronizada su instancia de Windows Server Active Directory con Azure Active Directory.

Para obtener una lista de direcciones URL que debe desbloquear para que la implementación de Windows Virtual Desktop funcione según lo previsto, consulte nuestra [lista de direcciones URL seguras](safe-url-list.md).

Windows Virtual Desktop consta de los escritorios y las aplicaciones de Windows que entrega a los usuarios y de la solución de administración, que Microsoft hospeda en Azure como un servicio. Se pueden implementar escritorios y aplicaciones en máquinas virtuales (VM) de cualquier región de Azure, y la solución de administración y los datos de estas VM residirán en Estados Unidos. Esto puede dar lugar a la transferencia de datos a Estados Unidos.

Para obtener un rendimiento óptimo, asegúrese de que la red cumple los requisitos siguientes:

* La latencia de ida y vuelta (RTT) desde la red del cliente hasta la región de Azure donde se han implementado grupos host debe ser inferior a 150 ms. Use el [estimador de experiencia](https://azure.microsoft.com/services/virtual-desktop/assessment) para ver el estado de la conexión y la región de Azure recomendada.
* El tráfico de red puede fluir fuera de las fronteras del país o la región si las máquinas virtuales que hospedan los escritorios y las aplicaciones se conectan al servicio de administración.
* Para optimizar el rendimiento de la red, se recomienda que las máquinas virtuales del host de sesión se coloquen en la misma región de Azure que el servicio de administración.

## <a name="supported-remote-desktop-clients"></a>Clientes compatibles de Escritorio remoto

Los clientes de Escritorio remoto siguientes admiten Windows Virtual Desktop:

* [Escritorio de Windows](connect-windows-7-10.md)
* [Web](connect-web.md)
* [macOS](connect-macos.md)
* [iOS](connect-ios.md)
* [Android](connect-android.md)

> [!IMPORTANT]
> Windows Virtual Desktop no es compatible con el cliente de Conexión de RemoteApp y Escritorio (RADC) ni con el cliente de Conexión a Escritorio remoto (MSTSC).

> [!IMPORTANT]
> Windows Virtual Desktop no es compatible actualmente con el cliente de Escritorio remoto de la tienda Windows. La compatibilidad con este cliente se agregará en una versión futura.

Para más información sobre las direcciones URL que debe desbloquear para usar los clientes, consulte la [lista de direcciones URL seguras](safe-url-list.md).

## <a name="supported-virtual-machine-os-images"></a>Imágenes de SO de máquinas virtuales admitidas

Windows Virtual Desktop admite las imágenes de SO x64 siguientes:

* Sesión múltiple de Windows 10 Enterprise, versión 1809 o superior
* Windows 10 Enterprise, versión 1809 o superior
* Windows 7 Enterprise
* Windows Server 2019
* Windows Server 2016
* Windows Server 2012 R2

Windows Virtual Desktop no es compatible con imágenes de los sistemas operativos x86 (32 bits), Windows 10 Enterprise N o Windows 10 Enterprise KN. Windows 7 tampoco admite las soluciones de perfil basadas en VHD o VHDX hospedadas en Azure Storage administrado debido a un límite de tamaño del sector.

Las opciones de automatización y de implementación disponibles dependen del sistema operativo y la versión que elija, tal como se muestra en la tabla siguiente:

|Sistema operativo|Galería de imágenes de Azure|Implementación manual de la máquina virtual|Integración de la plantilla de Azure Resource Manager|Aprovisionamiento de grupos host en Azure Marketplace|
|--------------------------------------|:------:|:------:|:------:|:------:|
|Windows 10 Enterprise (multisesión), versión 2004|Sí|Sí|Sí|Sí|
|Windows 10 Enterprise (multisesión), versión 1909|Sí|Sí|Sí|Sí|
|Windows 10 Enterprise (multisesión), versión 1903|Sí|Sí|No|No|
|Windows 10 Enterprise (multisesión), versión 1809|Sí|Sí|No|No|
|Windows 7 Enterprise|Sí|Sí|No|No|
|Windows Server 2019|Sí|Sí|No|No|
|Windows Server 2016|Sí|Sí|Sí|Sí|
|Windows Server 2012 R2|Sí|Sí|No|No|

## <a name="next-steps"></a>Pasos siguientes

Si usa Windows Virtual Desktop (clásico), puede empezar a trabajar con el tutorial de [Creación de un inquilino en Windows Virtual Desktop](./virtual-desktop-fall-2019/tenant-setup-azure-active-directory.md).

Si usa Windows Virtual Desktop con la integración de Azure Resource Manager, deberá crear un grupo de hosts. Vaya al siguiente tutorial para comenzar.

> [!div class="nextstepaction"]
> [Creación de un grupo de hosts con Azure Portal](create-host-pools-azure-marketplace.md)
