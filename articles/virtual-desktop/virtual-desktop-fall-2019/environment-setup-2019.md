---
title: 'Entorno de Windows Virtual Desktop (clásico): Azure'
description: Elementos básicos de un entorno de Windows Virtual Desktop (clásico).
author: Heidilohr
ms.topic: conceptual
ms.date: 03/30/2020
ms.author: helohr
manager: lizross
ms.openlocfilehash: 4fa281f6435013e9feb6808aab04ffa69c5f96c4
ms.sourcegitcommit: 98854e3bd1ab04ce42816cae1892ed0caeedf461
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/07/2020
ms.locfileid: "88008617"
---
# <a name="windows-virtual-desktop-classic-environment"></a>Entorno de Windows Virtual Desktop (clásico)

>[!IMPORTANT]
>Este contenido se aplica a Windows Virtual Desktop (clásico), que no admite objetos de Windows Virtual Desktop para Azure Resource Manager. Si está tratando de administrar objetos de Windows Virtual Desktop para Azure Resource Manager, consulte [este artículo](../environment-setup.md).

Windows Virtual Desktop es un servicio que ofrece a los usuarios un acceso fácil y seguro a sus escritorios virtualizados y a RemoteApps. Este tema le proporcionará más información sobre la estructura general del entorno de Windows Virtual Desktop.

## <a name="tenants"></a>Inquilinos

El inquilino de Windows Virtual Desktop es la interfaz principal para administrar su entorno de Windows Virtual Desktop. Cada inquilino de Windows Virtual Desktop debe estar asociado con la instancia de Azure Active Directory que contiene los usuarios que iniciarán sesión en el entorno. Desde el inquilino de Windows Virtual Desktop, puede comenzar a crear grupos de hosts para ejecutar las cargas de trabajo de sus usuarios.

## <a name="host-pools"></a>Grupos de host

Un grupo de hosts es una colección de máquinas virtuales de Azure que se registran en Windows Virtual Desktop como hosts de sesión cuando ejecuta el agente de Windows Virtual Desktop. Todas las máquinas virtuales de host de sesión en un grupo de hosts deben provenir de la misma imagen para poder obtener una experiencia de usuario consistente.

Un grupo de hosts puede ser uno de estos dos tipos:

- Personal, donde cada host de sesión se asigna a usuarios individuales.
- Agrupado, donde los hosts de sesión pueden aceptar conexiones de cualquier usuario autorizado a un grupo de aplicaciones dentro del grupo de hosts.

Puede configurar propiedades adicionales en el grupo de hosts para cambiar el comportamiento de equilibrio de carga, cuántas sesiones puede tener cada host de sesión y lo que el usuario puede hacer con los hosts de sesión en el grupo de hosts mientras inicia sesión en Windows Virtual Desktop. Puede controlar los recursos publicados a los usuarios a través de grupos de aplicaciones.

## <a name="app-groups"></a>Grupos de aplicaciones

Un grupo de aplicaciones es una agrupación lógica de aplicaciones instaladas en hosts de sesión del grupo de hosts. Un grupo de aplicaciones puede ser uno de estos dos tipos:

- RemoteApp, donde los usuarios acceden a las instancias de RemoteApp que usted mismo seleccione y publique individualmente en el grupo de aplicaciones.
- Escritorio, donde los usuarios acceden al escritorio completo.

De forma predeterminada, un grupo de aplicaciones de escritorio (denominado "Grupo de aplicaciones de escritorio") se crea automáticamente cada vez que crea un grupo de hosts. Puede eliminar este grupo de aplicaciones en cualquier momento. Sin embargo, no puede crear otro grupo de aplicaciones de escritorio en el grupo de hosts mientras ya exista otro grupo de aplicaciones de escritorio. Para publicar instancias de RemoteApp, debe crear un grupo de aplicaciones de RemoteApp. Puede crear varios grupos de aplicaciones de RemoteApp para adaptarse a diferentes escenarios de trabajo. Los diferentes grupos de aplicaciones de RemoteApp también pueden contener instancias de RemoteApp superpuestas.

Para publicar recursos para los usuarios, debe asignarlos a grupos de aplicaciones. Cuando asigne usuarios a los grupos de aplicaciones, tenga en cuenta lo siguiente:

- No se puede asignar un usuario a un grupo de aplicaciones de escritorio y a un grupo de aplicaciones de RemoteApp en el mismo grupo de hosts.
- Se puede asignar un usuario a varios grupos de aplicaciones dentro del mismo grupo de hosts, y su fuente será una acumulación de ambos grupos de aplicaciones.

## <a name="tenant-groups"></a>Grupos de inquilinos

En Windows Virtual Desktop, el inquilino de Windows Virtual Desktop es donde ocurre la mayor parte de la configuración e instalación. El inquilino de Windows Virtual Desktop contiene los grupos de hosts, los grupos de aplicaciones y las asignaciones de usuarios del grupo de aplicaciones. Sin embargo, puede haber ciertas situaciones en las que necesite administrar varios inquilinos de Windows Virtual Desktop a la vez, especialmente si es un proveedor de servicios en la nube (CSP) o un asociado de hospedaje. En estas situaciones, puede usar un grupo personalizado de inquilinos de Windows Virtual Desktop para ubicar a cada uno de los inquilinos de Windows Virtual Desktop de los clientes y administrar el acceso de forma centralizada. Sin embargo, si solo está administrando un único inquilino de Windows Virtual Desktop, el concepto de grupo de inquilinos no se aplica y puede continuar supervisando y administrando el inquilino que existe en el grupo de inquilinos predeterminado.

## <a name="end-users"></a>Usuarios finales

Después de asignar usuarios a sus grupos de aplicaciones, pueden conectarse a una implementación de Windows Virtual Desktop con cualquiera de los clientes de Windows Virtual Desktop.

## <a name="next-steps"></a>Pasos siguientes

Obtenga más información sobre el acceso delegado y cómo asignar roles a los usuarios en [Acceso delegado en Windows Virtual Desktop](delegated-access-virtual-desktop-2019.md).

Para información sobre cómo configurar el inquilino de Windows Virtual Desktop, consulte [Creación de un inquilino en Windows Virtual Desktop](tenant-setup-azure-active-directory.md).

Para obtener información sobre cómo conectarse a Windows Virtual Desktop, consulte uno de los siguientes artículos:

- [Conexión desde Windows 10 o Windows 7](connect-windows-7-10-2019.md)
- [Conexión desde un explorador web](connect-web-2019.md)
