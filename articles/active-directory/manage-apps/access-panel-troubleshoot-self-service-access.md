---
title: Problemas al usar el acceso de autoservicio a las aplicaciones | Microsoft Docs
description: Solucionar problemas relacionados con el acceso de autoservicio a las aplicaciones
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
ms.date: 07/11/2017
ms.author: mimart
ms.reviewer: japere,asteen
ms.collection: M365-identity-device-management
ms.openlocfilehash: a981dfb1d72c21eccf2ad7119ea219114ed15aed
ms.sourcegitcommit: 36c50860e75d86f0d0e2be9e3213ffa9a06f4150
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 05/16/2019
ms.locfileid: "65784273"
---
# <a name="problem-using-self-service-application-access"></a>Problemas al usar el acceso de autoservicio a las aplicaciones

El acceso de autoservicio a las aplicaciones es una excelente manera de permitir a los usuarios detectar automáticamente aplicaciones y, si lo desea, permitir que el grupo de negocios apruebe el acceso a esas aplicaciones. Puede permitir que el grupo de negocios administre las credenciales asignadas a esos usuarios para que puedan realizar un inicio de sesión único con contraseña en las aplicaciones directamente desde sus paneles de acceso.

Antes de que los usuarios puedan detectar automáticamente las aplicaciones de su panel de acceso, debe habilitar el **acceso de autoservicio a las aplicaciones** de cualquier aplicación que desee permitir que los usuarios puedan detectar automáticamente y solicitar acceso a ella.

## <a name="general-issues-to-check-first"></a>Problemas generales para comprobar primero

-   Asegúrese de que el acceso de autoservicio a las aplicaciones está configurado correctamente. Consulte "Configuración del acceso de autoservicio a las aplicaciones".

-   Asegúrese de que se ha habilitado al usuario o grupo para solicitar el acceso de autoservicio a las aplicaciones.

-   Asegúrese de que el usuario está visitando el lugar correcto para el acceso de autoservicio a las aplicaciones. Los usuarios pueden navegar a sus [paneles de acceso a las aplicaciones](https://myapps.microsoft.com/) y hacer clic en el botón **+ Agregar** para buscar las aplicaciones para las que se ha habilitado el acceso de autoservicio.

-   Si ha configurado recientemente el acceso de autoservicio a las aplicaciones, intente iniciar sesión y cerrar sesión en el panel de acceso del usuario después de unos minutos para ver si los cambios del acceso de autoservicio han aparecido.

## <a name="how-to-configure-self-service-application-access"></a>Configuración del acceso de autoservicio a las aplicaciones

Para habilitar el acceso de autoservicio a las aplicaciones, siga estos pasos:

1. Abra [**Azure Portal**](https://portal.azure.com/) e inicie sesión como **administrador global**.

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
    > No se admiten grupos.
    >
    >

13. **Opcional:** **para las aplicaciones que exponen roles**, si desea asignar usuarios aprobados de autoservicio a un rol, haga clic en el selector situado junto a la etiqueta **¿A qué rol deben asignarse los usuarios de esta aplicación?** para seleccionar el rol al que se deben asignar estos usuarios.

14. Haga clic en el botón **Guardar** de la parte superior de la hoja para terminar.

Cuando se haya completado la configuración de la aplicación de autoservicio, los usuarios pueden navegar a sus [paneles de acceso a las aplicaciones](https://myapps.microsoft.com/) y hacer clic en el botón **+ Agregar** para buscar las aplicaciones para las que se ha habilitado el acceso de autoservicio. Los aprobadores de la empresa pueden también ver una notificación en sus [paneles de acceso a las aplicaciones](https://myapps.microsoft.com/). Puede habilitar un correo electrónico que les informa cuando un usuario ha solicitado el acceso a una aplicación que requiere su aprobación. 

Estas aprobaciones solo admiten flujos de trabajo de aprobación única, lo que significa que, si especifica varios aprobadores, cualquier aprobador individual puede aprobar el acceso a la aplicación.

## <a name="if-these-troubleshooting-steps-do-not-resolve-the-issue"></a>Si después de seguir estos pasos el problema no se ha resuelto, 

abra una incidencia de soporte técnico con la información siguiente si está disponible:

-   Id. de error de correlación

-   UPN (dirección de correo electrónico del usuario)

-   TenantID

-   Tipo de explorador

-   Zona horaria y hora/período de tiempo durante el que se produce el error

-   Seguimientos de Fiddler

## <a name="next-steps"></a>Pasos siguientes
[Configuración de Azure Active Directory para la administración de grupos de autoservicio](../users-groups-roles/groups-self-service-management.md)
