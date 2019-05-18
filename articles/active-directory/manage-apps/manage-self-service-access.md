---
title: Configuración de la asignación a la aplicación de autoservicio | Microsoft Docs
description: Habilitar el acceso de autoservicio a las aplicaciones para permitir a los usuarios buscar sus propias aplicaciones
services: active-directory
documentationcenter: ''
author: msmimart
manager: CelesteDG
ms.assetid: ''
ms.service: active-directory
ms.subservice: app-mgmt
ms.workload: identity
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: article
ms.date: 04/02/2018
ms.author: mimart
ms.collection: M365-identity-device-management
ms.openlocfilehash: 66b842fc1b438e2d0046b4359712c3537f568e5a
ms.sourcegitcommit: be9fcaace62709cea55beb49a5bebf4f9701f7c6
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 05/17/2019
ms.locfileid: "65824428"
---
# <a name="how-to-configure-self-service-application-assignment"></a>Configuración de la asignación a la aplicación de autoservicio

Antes de que los usuarios puedan detectar automáticamente las aplicaciones de su panel de acceso, debe habilitar el **acceso de autoservicio a las aplicaciones** de cualquier aplicación que desee permitir que los usuarios puedan detectar automáticamente y solicitar acceso a ella.

Esta característica es una excelente manera de ahorrar tiempo y dinero como grupo de TI y es muy recomendable como parte de una implementación de aplicaciones moderna con Azure Active Directory.

Con esta característica, puede:

-   Permitir que los usuarios detecten automáticamente las aplicaciones desde el [panel de acceso a las aplicaciones](https://myapps.microsoft.com/) sin molestar al grupo de TI.

-   Agregar esos usuarios a un grupo preconfigurado para que pueda ver quién ha solicitado acceso, quitar el acceso y administrar los roles que tienen asignados.

-   Si lo desea, puede permitir que un aprobador de la empresa apruebe las solicitudes de acceso a la aplicación para que no tenga que hacerlo el grupo de TI.

-   También puede configurar un máximo de 10 usuarios que pueden aprobar el acceso a esta aplicación.

-   Permitir que un aprobador de la empresa establezca las contraseñas que los usuarios pueden usar para iniciar sesión en la aplicación directamente desde el [panel de acceso a las aplicaciones](https://myapps.microsoft.com/) del aprobador.

-   Asignar automáticamente usuarios asignados de autoservicio directamente a un rol de aplicación.

## <a name="enable-self-service-application-access-to-allow-users-to-find-their-own-applications"></a>Habilitar el acceso de autoservicio a las aplicaciones para permitir a los usuarios buscar sus propias aplicaciones

El acceso de autoservicio a las aplicaciones es una excelente manera de permitir a los usuarios detectar automáticamente aplicaciones y, si lo desea, permitir que el grupo de negocios apruebe el acceso a esas aplicaciones. Puede permitir que el grupo de negocios administre las credenciales asignadas a esos usuarios para que puedan realizar un inicio de sesión único con contraseña en las aplicaciones directamente desde sus paneles de acceso.

Para habilitar el acceso de autoservicio a las aplicaciones, siga estos pasos:

1.  Abra [**Azure Portal**](https://portal.azure.com/) e inicie sesión como **administrador global**.

2.  Abra la **extensión de Azure Active Directory** haciendo clic en **Todos los servicios** en la parte inferior del menú de navegación izquierdo.

3.  Escriba **"Azure Active Directory**" en el cuadro de búsqueda de filtrado y seleccione el elemento **Azure Active Directory**.

4.  Haga clic en **Aplicaciones empresariales** en el menú de navegación izquierdo de Azure Active Directory.

5.  Haga clic en **Todas las aplicaciones** para ver una lista de todas las aplicaciones.

    * Si no ve la aplicación que desea que aparezca aquí, use el control **Filtro** de la parte superior de la lista **Todas las aplicaciones** y establezca la opción **Mostrar** en **Todas las aplicaciones.**

6.  Seleccione la aplicación en la que desea habilitar el acceso de autoservicio de la lista.

7.  Cuando se cargue la aplicación, haga clic en **Autoservicio** en el menú de navegación izquierdo de la aplicación.

8.  Para habilitar el acceso de autoservicio a las aplicaciones para esta aplicación, establezca la opción **¿Quiere permitir que los usuarios soliciten acceso a esta aplicación?** en **Sí.**

9.  A continuación, seleccione el grupo al que se deben agregar los usuarios que solicitan acceso a esta aplicación, haga clic en el selector situado junto a la etiqueta **¿A qué grupo se deberían agregar los usuarios asignados?** y seleccione un grupo.
  
    > [!NOTE]
    > No se admite el uso de grupos sincronizados desde el entorno local para el grupo al que se deben agregar los usuarios que solicitan acceso a esta aplicación.
  
10. **Opcional:** si desea requerir la aprobación de la empresa antes de permitir el acceso a los usuarios, establezca la opción **¿Quiere requerir una aprobación para conceder acceso a esta aplicación?** en **Sí**.

11. **Opcional: para las aplicaciones que solo utilizan el inicio de sesión único con contraseña,** si desea permitir que los aprobadores de la empresa especifiquen las contraseñas que se envían a esta aplicación para los usuarios aprobados, establezca la opción **¿Quiere permitir que los aprobadores establezcan las contraseñas de los usuarios de esta aplicación?** en **Sí**.

12. **Opcional:** para especificar los aprobadores de la empresa que tienen permiso para aprobar el acceso a esta aplicación, haga clic en el selector situado junto a la etiqueta **¿Quién tiene permiso para aprobar el acceso a esta aplicación?** para seleccionar hasta 10 aprobadores individuales de la empresa.

     > [!NOTE]
     > No se admiten grupos.
     >
     >

13. **Opcional:** **para las aplicaciones que exponen roles**, si desea asignar usuarios aprobados de autoservicio a un rol, haga clic en el selector situado junto a la etiqueta **¿A qué rol deben asignarse los usuarios de esta aplicación?** para seleccionar el rol al que se deben asignar estos usuarios.

14. Haga clic en el botón **Guardar** de la parte superior de la hoja para terminar.

Cuando se haya completado la configuración de la aplicación de autoservicio, los usuarios pueden navegar a sus [paneles de acceso a las aplicaciones](https://myapps.microsoft.com/) y hacer clic en el botón **+ Agregar** para buscar las aplicaciones para las que se ha habilitado el acceso de autoservicio. Los aprobadores de la empresa pueden también ver una notificación en sus [paneles de acceso a las aplicaciones](https://myapps.microsoft.com/). Puede habilitar un correo electrónico que les informa cuando un usuario ha solicitado el acceso a una aplicación que requiere su aprobación. 

Estas aprobaciones admiten flujos de trabajo de aprobación única, lo que significa que si especifica varios aprobadores, cualquier aprobador individual puede aprobar el acceso a la aplicación.

## <a name="next-steps"></a>Pasos siguientes
[Configuración de Azure Active Directory para la administración de grupos de autoservicio](../users-groups-roles/groups-self-service-management.md)
