---
title: Asignación de usuarios y grupos a una aplicación | Microsoft Docs
description: Asignar usuarios a la aplicación para conceder acceso
services: active-directory
documentationcenter: ''
author: msmimart
manager: CelesteDG
ms.service: active-directory
ms.subservice: app-mgmt
ms.workload: identity
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: conceptual
ms.date: 04/26/2019
ms.author: mimart
ms.collection: M365-identity-device-management
ms.openlocfilehash: 7402778ec8170e60a623255d0d331f12d8389227
ms.sourcegitcommit: be9fcaace62709cea55beb49a5bebf4f9701f7c6
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 05/17/2019
ms.locfileid: "65824300"
---
# <a name="assign-users-and-groups-to-an-application-in-azure-active-directory"></a>Asignación de usuarios y grupos en una aplicación de Azure Active Directory
En este artículo se muestra cómo asignar usuarios o grupos a una aplicación en Azure Active Directory (Azure AD). Los usuarios deben asignarse primero a una aplicación antes de que un administrador les conceda acceso para hacer lo siguiente:

-   Acceso a una aplicación mediante la **navegación directa a la dirección URL de la aplicación** (también conocido como inicio de sesión iniciado por el proveedor de servicios).

-   Acceso a una aplicación mediante la **dirección URL de acceso de usuarios** en la página **Propiedades** de una aplicación (también conocido como inicio de sesión iniciado por un proveedor de identidades).

-   Vea como aparece una aplicación en su [panel de acceso a las aplicaciones](https://myapps.microsoft.com/) o aplicación móvil.

-   Vea como aparece una aplicación en el [iniciador de aplicaciones de Office 365](https://support.office.com/article/Meet-the-Office-365-app-launcher-79f12104-6fed-442f-96a0-eb089a3f476a).

La disponibilidad de la asignación basada en grupo viene determinada por su contrato de licencia. Solo grupos de seguridad es compatible con la asignación basada en grupo y las pertenencias a grupos anidados no se admiten actualmente.

## <a name="prerequisites"></a>Requisitos previos
Para poder asignar usuarios y grupos a una aplicación, debe requerir la asignación de usuarios. Para requerir la asignación de usuario:

1. Inicie sesión en el portal de Azure con una cuenta de administrador.
2. Haga clic en el elemento **Todos los servicios** del menú principal.
3. Elija el directorio que está usando para la aplicación.
4. Haga clic en la pestaña **Aplicaciones empresariales**.
5. Seleccione la aplicación en la lista de aplicaciones asociada a este directorio.
6. Haga clic en la pestaña **Propiedades**.
7. Cambie la opción **¿Asignación de usuarios?** a Sí.
8. Haga clic en el botón **Guardar** de la parte superior de la pantalla.

## <a name="assign-users"></a>Asignar usuarios

Para asignar uno o varios usuarios a una aplicación directamente, siga los pasos siguientes:

1.  Abra [**Azure Portal**](https://portal.azure.com/) e inicie sesión como **administrador global.**

2.  Abra la **extensión de Azure Active Directory** haciendo clic en **Todos los servicios** en la parte inferior del menú de navegación izquierdo.

3.  Escriba **"Azure Active Directory**" en el cuadro de búsqueda de filtrado y seleccione el elemento **Azure Active Directory**.

4.  Haga clic en **Aplicaciones empresariales** en el menú de navegación izquierdo de Azure Active Directory.

5.  Haga clic en **Todas las aplicaciones** para ver una lista de todas las aplicaciones.

    * Si no ve la aplicación que desea que aparezca aquí, use el control **Filtro** de la parte superior de la lista **Todas las aplicaciones** y establezca la opción **Mostrar** en **Todas las aplicaciones.**

6.  Seleccione la aplicación que desea asignar a un usuario de la lista.

7.  Cuando se cargue la aplicación, haga clic en **Usuarios y grupos** desde el menú de navegación izquierdo de la aplicación.

8.  Haga clic en el botón **Agregar** en la parte superior de la lista **Usuarios y grupos** para abrir el panel **Agregar asignación**.

9.  Haga clic en el selector **Usuarios y grupos** del panel **Agregar asignación**.

10. Escriba el **nombre completo** o la **dirección de correo electrónico** del usuario al que quiere asignar en el cuadro de búsqueda **Buscar por nombre o dirección de correo**.

11. Mantenga el puntero sobre el **usuario** en la lista para que aparezca una **casilla**. Haga clic en la casilla situada junto a la foto o el logotipo del perfil del usuario para agregar ese usuario a la lista de **seleccionados**.

12. **Opcional:** si desea **agregar más de un usuario**, escriba otro **nombre completo** o **dirección de correo electrónico** en el cuadro de búsqueda **Buscar por nombre o dirección de correo** y haga clic en la casilla para agregar ese usuario a la lista de **seleccionados**.

13. Cuando haya terminado de seleccionar usuarios, haga clic en el botón **Seleccionar** para agregarlos a la lista de usuarios y grupos que se asignarán a la aplicación.

14. **Opcional:** haga clic en el selector **Seleccionar rol** del panel **Agregar asignación** para seleccionar un rol que se asignará a los usuarios que ha seleccionado.

15. Haga clic en el botón **Asignar** para asignar la aplicación a los usuarios seleccionados.

Tras un breve período de tiempo, los usuarios seleccionados podrán iniciar estas aplicaciones mediante los métodos descritos en la sección de descripción de la solución.

## <a name="assign-groups"></a>Asignación de grupos

Para asignar uno o varios grupos a una aplicación directamente, siga estos pasos:

1.  Abra [**Azure Portal**](https://portal.azure.com/) e inicie sesión como **administrador global.**

2.  Abra la **extensión de Azure Active Directory** haciendo clic en **Todos los servicios** en la parte inferior del menú de navegación izquierdo.

3.  Escriba **"Azure Active Directory**" en el cuadro de búsqueda de filtrado y seleccione el elemento **Azure Active Directory**.

4.  Haga clic en **Aplicaciones empresariales** en el menú de navegación izquierdo de Azure Active Directory.

5.  Haga clic en **Todas las aplicaciones** para ver una lista de todas las aplicaciones.

    * Si no ve la aplicación que desea que aparezca aquí, use el control **Filtro** de la parte superior de la lista **Todas las aplicaciones** y establezca la opción **Mostrar** en **Todas las aplicaciones.**

6.  Seleccione la aplicación que desea asignar a un usuario de la lista.

7.  Cuando se cargue la aplicación, haga clic en **Usuarios y grupos** desde el menú de navegación izquierdo de la aplicación.

8.  Haga clic en el botón **Agregar** en la parte superior de la lista **Usuarios y grupos** para abrir el panel **Agregar asignación**.

9.  Haga clic en el selector **Usuarios y grupos** del panel **Agregar asignación**.

10. Escriba el **nombre completo** del grupo al que quiere asignar en el cuadro de búsqueda **Buscar por nombre o dirección de correo**.

11. Mantenga el puntero sobre el **grupo** en la lista para que aparezca una **casilla**. Haga clic en la casilla situada junto a la foto o el logotipo del perfil del grupo para agregar ese usuario a la lista de **seleccionados**.

12. **Opcional:** si desea **agregar más de un grupo**, escriba otro **nombre de grupo completo** o en el cuadro de búsqueda **Buscar por nombre o dirección de correo** y haga clic en la casilla para agregar ese grupo a la lista de **seleccionados**.

13. Cuando haya terminado de seleccionar grupos, haga clic en el botón **Seleccionar** para agregarlos a la lista de usuarios y grupos que se asignarán a la aplicación.

14. **Opcional:** haga clic en el selector **Seleccionar rol** del panel **Agregar asignación** para seleccionar un rol que se asignará a los grupos que ha seleccionado.

15. Haga clic en el botón **Asignar** para asignar la aplicación a los grupos seleccionados.

Tras un breve período de tiempo, los usuarios de los grupos seleccionados podrán iniciar estas aplicaciones mediante los métodos descritos en la sección de descripción de la solución. Si se trata de grupos dinámicos, puede que haya algo de retardo de procesamiento adicional en estas asignaciones hasta que aparezcan para los usuarios dentro de estos grupos asignados.

## <a name="enable-self-service-application-access"></a>Habilitación del acceso de autoservicio a las aplicaciones

El acceso de autoservicio a las aplicaciones es una excelente manera de permitir a los usuarios detectar automáticamente aplicaciones y, si lo desea, permitir que el grupo de negocios apruebe el acceso a esas aplicaciones. Puede permitir que el grupo de negocios administre las credenciales asignadas a esos usuarios para que puedan realizar un inicio de sesión único con contraseña en las aplicaciones directamente desde sus paneles de acceso.

Para habilitar el acceso de autoservicio a las aplicaciones, siga estos pasos:

1. Abra [**Azure Portal**](https://portal.azure.com/) e inicie sesión como **administrador global.**

2. Abra la **extensión de Azure Active Directory** haciendo clic en **Todos los servicios** en la parte inferior del menú de navegación izquierdo.

3. Escriba **"Azure Active Directory**" en el cuadro de búsqueda de filtrado y seleccione el elemento **Azure Active Directory**.

4. Haga clic en **Aplicaciones empresariales** en el menú de navegación izquierdo de Azure Active Directory.

5. Haga clic en **Todas las aplicaciones** para ver una lista de todas las aplicaciones.

   * Si no ve la aplicación que desea que aparezca aquí, use el control **Filtro** de la parte superior de la lista **Todas las aplicaciones** y establezca la opción **Mostrar** en **Todas las aplicaciones.**

6. Seleccione la aplicación en la que desea habilitar el acceso de autoservicio de la lista.

7. Cuando se cargue la aplicación, haga clic en **Autoservicio** en el menú de navegación izquierdo de la aplicación.

8. Para habilitar el acceso de autoservicio a las aplicaciones para esta aplicación, establezca la opción **¿Quiere permitir que los usuarios soliciten acceso a esta aplicación?** en **Sí.**

9. A continuación, seleccione el grupo al que se deben agregar los usuarios que solicitan acceso a esta aplicación, haga clic en el selector situado junto a la etiqueta **¿A qué grupo se deberían agregar los usuarios asignados?** y seleccione un grupo.

10. **Opcional:** si desea requerir la aprobación de la empresa antes de permitir el acceso a los usuarios, establezca la opción **¿Quiere requerir una aprobación para conceder acceso a esta aplicación?** en **Sí**.

11. **Opcional: para las aplicaciones que solo utilizan el inicio de sesión único con contraseña,** si desea permitir que los aprobadores de la empresa especifiquen las contraseñas que se envían a esta aplicación para los usuarios aprobados, establezca la opción **¿Quiere permitir que los aprobadores establezcan las contraseñas de los usuarios de esta aplicación?** en **Sí**.

12. **Opcional:** para especificar los aprobadores de la empresa que tienen permiso para aprobar el acceso a esta aplicación, haga clic en el selector situado junto a la etiqueta **¿Quién tiene permiso para aprobar el acceso a esta aplicación?** para seleccionar hasta 10 aprobadores individuales de la empresa.

    >[!NOTE]
    >No se admiten grupos.
    >
    >

13. **Opcional:** **para las aplicaciones que exponen roles**, si desea asignar usuarios aprobados de autoservicio a un rol, haga clic en el selector situado junto a la etiqueta **¿A qué rol deben asignarse los usuarios de esta aplicación?** para seleccionar el rol al que se deben asignar estos usuarios.

14. Haga clic en el botón **Guardar** de la parte superior del panel para terminar.

Cuando se haya completado la configuración de la aplicación de autoservicio, los usuarios pueden navegar a sus [paneles de acceso a las aplicaciones](https://myapps.microsoft.com/) y hacer clic en el botón **+ Agregar** para buscar las aplicaciones para las que se ha habilitado el acceso de autoservicio. Los aprobadores de la empresa pueden también ver una notificación en sus [paneles de acceso a las aplicaciones](https://myapps.microsoft.com/). Puede habilitar un correo electrónico que les informa cuando un usuario ha solicitado el acceso a una aplicación que requiere su aprobación. 

Estas aprobaciones admiten flujos de trabajo de aprobación única, lo que significa que si especifica varios aprobadores, cualquier aprobador individual puede aprobar el acceso a la aplicación.

## <a name="next-steps"></a>Pasos siguientes
[Proporcionar un inicio de sesión único a las aplicaciones con el proxy de aplicación](application-proxy-configure-single-sign-on-with-kcd.md)
