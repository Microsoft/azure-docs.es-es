---
title: Uso del acceso a la aplicación de autoservicio en Azure AD
description: Habilitación del autoservicio para que los usuarios puedan buscar aplicaciones en Azure AD
services: active-directory
author: kenwith
manager: celestedg
ms.service: active-directory
ms.subservice: app-mgmt
ms.workload: identity
ms.topic: how-to
ms.date: 07/11/2017
ms.author: kenwith
ms.reviewer: japere,asteen
ms.openlocfilehash: 0cd20d7a11bcffe9937537e3681199757a52bee5
ms.sourcegitcommit: 3fb5e772f8f4068cc6d91d9cde253065a7f265d6
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/31/2020
ms.locfileid: "89181721"
---
# <a name="how-to-use-self-service-application-access"></a>Uso del acceso de autoservicio a las aplicaciones

Para que los usuarios puedan descubrir automáticamente aplicaciones desde su página Aplicaciones, debe habilitar **Acceso de autoservicio a las aplicaciones** en las aplicaciones que quiera que los usuarios descubran automáticamente y puedan solicitar acceso.

Esta característica es una excelente manera de ahorrar tiempo y dinero como grupo de TI y es muy recomendable como parte de una implementación de aplicaciones moderna con Azure Active Directory.

Para obtener información sobre el uso de Mis aplicaciones desde la perspectiva del usuario final, consulte la [ayuda del portal Mis aplicaciones](../user-help/my-apps-portal-end-user-access.md).

Con esta característica, puede:

-   Permitir que los usuarios descubran automáticamente las aplicaciones desde la página [Mis aplicaciones](https://myapps.microsoft.com/) sin molestar al grupo de TI.
-   Agregar esos usuarios a un grupo preconfigurado para que pueda ver quién ha solicitado acceso, quitar el acceso y administrar los roles que tienen asignados.
-   Si lo desea, permita que alguien apruebe las solicitudes de acceso a la aplicación para que no tenga que hacerlo el grupo de TI.
-   También puede configurar un máximo de 10 usuarios que pueden aprobar el acceso a esta aplicación.
-   También puede permitir a un usuario establecer las contraseñas que los usuarios pueden usar para iniciar sesión en la aplicación.
-   Asignar automáticamente usuarios asignados de autoservicio directamente a un rol de aplicación.

## <a name="enable-self-service-application-access-to-allow-users-to-find-their-own-applications"></a>Habilitar el acceso de autoservicio a las aplicaciones para permitir a los usuarios buscar sus propias aplicaciones

El acceso de autoservicio a las aplicaciones es una excelente manera de permitir a los usuarios detectar automáticamente aplicaciones y, si lo desea, permitir que el grupo de negocios apruebe el acceso a esas aplicaciones. Puede permitir que el grupo de negocios administre las credenciales asignadas a esos usuarios para que puedan realizar un inicio de sesión único con contraseña en las aplicaciones directamente desde la página Mis aplicaciones.

Para habilitar el acceso de autoservicio a las aplicaciones, siga estos pasos:
1. Abra [**Azure Portal**](https://portal.azure.com/) e inicie sesión como **administrador global.**
2. Abra la **extensión de Azure Active Directory** haciendo clic en **Todos los servicios** en la parte superior del menú de navegación izquierdo principal.
3. Escriba **"Azure Active Directory**" en el cuadro de búsqueda de filtrado y seleccione el elemento **Azure Active Directory**.
4. Seleccione **Aplicaciones empresariales** en el menú de navegación izquierdo de Azure Active Directory.
5. Seleccione **Todas las aplicaciones** para ver una lista de todas las aplicaciones.
   * Si no ve la aplicación que desea que aparezca aquí, use el control **Filtro** de la parte superior de la lista **Todas las aplicaciones** y establezca la opción **Mostrar** en **Todas las aplicaciones.**
6. Seleccione la aplicación en la que desea habilitar el acceso de autoservicio de la lista.
7. Cuando se cargue la aplicación, haga clic en **Autoservicio** en el menú de navegación izquierdo de la aplicación.
8. Para habilitar el acceso de autoservicio a las aplicaciones para esta aplicación, establezca la opción **¿Quiere permitir que los usuarios soliciten acceso a esta aplicación?** en **Sí.**
9. A continuación, seleccione el grupo al que se deben agregar los usuarios que solicitan acceso a esta aplicación, seleccione el selector situado junto a la etiqueta **¿A qué grupo se deberían agregar los usuarios asignados?** y seleccione un grupo.
10. **Opcional:** si desea requerir la aprobación de la empresa antes de permitir el acceso a los usuarios, establezca la opción **¿Quiere requerir una aprobación para conceder acceso a esta aplicación?** en **Sí**.
11. **Opcional: para las aplicaciones que solo utilizan el inicio de sesión único con contraseña,** si desea permitir que los aprobadores de la empresa especifiquen las contraseñas que se envían a esta aplicación para los usuarios aprobados, establezca la opción **¿Quiere permitir que los aprobadores establezcan las contraseñas de los usuarios de esta aplicación?** en **Sí**.
12. **Opcional:** Especifique los aprobadores de negocios a los que se les permite aprobar el acceso a esta aplicación. Seleccione **¿Quién tiene permiso para aprobar el acceso a esta aplicación?** . A continuación, puede seleccionar hasta 10 aprobadores individuales de la empresa.
    * No se admiten grupos.
13. **Opcional:** **para las aplicaciones que exponen roles**, si desea asignar usuarios aprobados de autoservicio a un rol, seleccione el selector situado junto a la etiqueta **¿A qué rol deben asignarse los usuarios de esta aplicación?** para seleccionar el rol al que se deben asignar estos usuarios.
14. Seleccione el botón **Guardar** de la parte superior para finalizar.

Cuando se haya completado la configuración de la aplicación de autoservicio, navegue a sus [Mis aplicaciones](https://myapps.microsoft.com/) y haga clic en el botón **+ Agregar** para buscar las aplicaciones para las que se ha habilitado el acceso de autoservicio. Los aprobadores de la empresa pueden ver también una notificación en su página [Mis aplicaciones](https://myapps.microsoft.com/). Puede habilitar un correo electrónico que les informa cuando un usuario ha solicitado el acceso a una aplicación que requiere su aprobación. 

Estas aprobaciones solo admiten flujos de trabajo de aprobación única, lo que significa que, si especifica varios aprobadores, cualquier aprobador individual puede aprobar el acceso a la aplicación.

## <a name="things-to-check-if-self-service-isnt-working"></a>Cosas para comprobar si el autoservicio no funciona
-   Asegúrese de que se ha habilitado al usuario o grupo para solicitar el acceso de autoservicio a las aplicaciones.
-   Asegúrese de que el usuario está visitando el lugar correcto para el acceso de autoservicio a las aplicaciones. Los usuarios pueden navegar a su página [Mis aplicaciones](https://myapps.microsoft.com/) y hacer clic en el botón **+ Agregar** para buscar las aplicaciones para las que se ha habilitado el acceso de autoservicio.
-   Si ha configurado recientemente el acceso de autoservicio a las aplicaciones, intente iniciar sesión y cerrar sesión en la página Mis aplicaciones después de unos minutos para ver si los cambios del acceso de autoservicio han aparecido.

## <a name="next-steps"></a>Pasos siguientes
[Configuración de Azure Active Directory para la administración de grupos de autoservicio](../users-groups-roles/groups-self-service-management.md)
