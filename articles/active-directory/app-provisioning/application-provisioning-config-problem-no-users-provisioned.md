---
title: Los usuarios no se aprovisionan en mi aplicación
description: Cómo solucionar problemas habituales que surgen cuando no se ve aparecer a usuarios en una aplicación de la galería de Azure AD que ha configurado para el aprovisionamiento de usuarios con Azure AD
services: active-directory
author: kenwith
manager: celestedg
ms.service: active-directory
ms.subservice: app-provisioning
ms.workload: identity
ms.topic: how-to
ms.date: 04/20/2020
ms.author: kenwith
ms.reviewer: arvinh
ms.openlocfilehash: d07450b61558814a99bf85668c55899c8cf9d389
ms.sourcegitcommit: 3bf69c5a5be48c2c7a979373895b4fae3f746757
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/14/2020
ms.locfileid: "88234795"
---
# <a name="no-users-are-being-provisioned"></a>No se está aprovisionando ningún usuario 
>[!NOTE]
>El 16/04/2020 cambiamos el comportamiento de los usuarios a los que se asigna el rol de acceso predeterminado. Vea la sección siguiente para obtener más detalles. 
>
Una vez que se ha configurado el aprovisionamiento automático de una aplicación (incluida la comprobación de que las credenciales de la aplicación proporcionadas a Azure AD para conectarse a la aplicación son válidas), se aprovisionan los usuarios y los grupos en la aplicación. El aprovisionamiento se determina mediante lo siguiente:

-   Qué usuarios y grupos se han **asignado** a la aplicación. Tenga en cuenta que no se admite el aprovisionamiento de grupos anidados o grupos de Office 365. Para más información sobre la asignación, consulte [Asignación de un usuario o un grupo a una aplicación empresarial en Azure Active Directory](../manage-apps/assign-user-or-group-access-portal.md).
-   Si las **asignaciones de atributos** están habilitadas o no, y configuradas para sincronizar atributos válidos de Azure AD a la aplicación. Para más información sobre las asignaciones de atributos, consulte [Personalización de asignaciones de atributos de aprovisionamiento de usuarios para aplicaciones SaaS en Azure Active Directory de usuarios](customize-application-attributes.md).
-   Si hay o no presente un **filtro de ámbito** que filtre usuarios en función de valores de atributo concretos. Para más información sobre los filtros de ámbito, consulte [Aprovisionamiento de aplicaciones basado en atributos con filtros de ámbito](../app-provisioning/define-conditional-rules-for-provisioning-user-accounts.md).
  
  
Si observa que no se están aprovisionando usuarios, consulte [Registros de aprovisionamiento (versión preliminar)](../reports-monitoring/concept-provisioning-logs.md?context=azure/active-directory/manage-apps/context/manage-apps-context) en Azure AD. Busque las entradas del registro para un usuario específico.

Para acceder a los registros de aprovisionamiento en Azure Portal, seleccione **Azure Active Directory** &gt; **Aplicaciones empresariales** &gt; **Registros de aprovisionamiento (versión preliminar)** en la sección **Actividad**. Puede buscar los datos de aprovisionamiento por el nombre del usuario o el identificador en el sistema de origen o en el sistema de destino. Para más información, consulte [Registros de aprovisionamiento (versión preliminar)](../reports-monitoring/concept-provisioning-logs.md?context=azure/active-directory/manage-apps/context/manage-apps-context). 

Los registros de aprovisionamiento registran todas las operaciones realizadas por el servicio de aprovisionamiento, lo que incluye consultar a Azure AD sobre los usuarios asignados que están en el ámbito de aprovisionamiento, consultar en la aplicación de destino la existencia de esos usuarios y comparar los objetos de usuario entre el sistema. Después, agregue, actualice o deshabilite la cuenta de usuario en el sistema de destino en función de la comparación.

## <a name="general-problem-areas-with-provisioning-to-consider"></a>Áreas problemáticas generales con el aprovisionamiento que tener en cuenta
A continuación, se muestra una lista de las áreas problemáticas generales en las que puede profundizar si tiene idea de por dónde empezar.

- [No parece que el servicio de aprovisionamiento se inicie](#provisioning-service-does-not-appear-to-start)
- [Los registros de aprovisionamiento indican que hay usuarios omitidos y no aprovisionados, aunque estén asignados](#provisioning-logs-say-users-are-skipped-and-not-provisioned-even-though-they-are-assigned)

## <a name="provisioning-service-does-not-appear-to-start"></a>No parece que el servicio de aprovisionamiento se inicie
Si establece **Estado de aprovisionamiento** en **Activado** en la sección **Azure Active Directory &gt; Aplicaciones empresariales &gt;\[Nombre de la aplicación\] &gt;Aprovisionamiento** de Azure Portal. Sin embargo, no se muestran más detalles de estado en esa página después de recargas posteriores; es probable que el servicio se esté ejecutando pero que no haya completado aún un ciclo inicial. Consulte los **Registros de aprovisionamiento (versión preliminar)** descritos antes para determinar qué operaciones está realizando el servicio y si se han producido errores.

>[!NOTE]
>Un ciclo inicial puede tardar desde 20 minutos hasta varias horas, según el tamaño del directorio de Azure AD y el número de usuarios en el ámbito de aprovisionamiento. Las sincronizaciones posteriores al ciclo inicial son más rápidas, ya que el servicio de aprovisionamiento almacena marcas de agua que representan el estado de ambos sistemas tras el ciclo inicial. El ciclo inicial mejora el rendimiento de las sincronizaciones posteriores.
>


## <a name="provisioning-logs-say-users-are-skipped-and-not-provisioned-even-though-they-are-assigned"></a>Los registros de aprovisionamiento indican que hay usuarios omitidos y no aprovisionados, aunque estén asignados

Cuando un usuario se muestra como "Omitido" en los registros de aprovisionamiento, es importante consultar la pestaña **Pasos** del registro para determinar la razón. Algunas razones y soluciones habituales son:

- **Se ha configurado un filtro de ámbito** **que está filtrando al usuario por un valor de atributo**. Para más información sobre los filtros de ámbito, consulte los [filtros de ámbito](../app-provisioning/define-conditional-rules-for-provisioning-user-accounts.md).
- **El usuario no está autorizado de forma efectiva.** Si ve un mensaje de error de este tipo, se debe a que hay un problema con el registro de asignación de usuarios almacenado en Azure AD. Para corregir este problema, desasigne el usuario (o grupo) de la aplicación y vuelva a asignarlo. Para más información sobre la asignación, consulte [Asignación de acceso a usuarios o grupos](../manage-apps/assign-user-or-group-access-portal.md).
- **Un atributo obligatorio falta o no se ha llenado para un usuario.** Una cuestión importante que tener en cuenta al configurar el aprovisionamiento es revisar y configurar las asignaciones de atributos y los flujos de trabajo que definen qué propiedades de usuario (o de grupo) fluyen de Azure AD a la aplicación. Esta configuración incluye la opción de "propiedad de coincidencia" que se usa para identificar de forma exclusiva y emparejar a usuarios y grupos entre ambos sistemas. Para más información sobre este importante proceso, consulte [Personalización de asignaciones de atributos de aprovisionamiento de usuarios para aplicaciones SaaS en Azure Active Directory de usuarios](customize-application-attributes.md).
- **Asignación de atributos para grupos:** Aprovisionamiento del nombre del grupo y los detalles del grupo, además de los miembros, si se admite para algunas aplicaciones. Puede habilitar o deshabilitar esta funcionalidad habilitando o deshabilitando el valor de **Asignación** para los objetos de grupo que se muestran en la pestaña **Aprovisionamiento**. Si los grupos de aprovisionamiento están habilitados, asegúrese de revisar las asignaciones de atributos para asegurarse de que se use un campo apropiado para el "identificador de coincidencia". El identificador de coincidencia puede ser el alias de correo electrónico o el nombre para mostrar. El grupo y sus miembros no se han aprovisionado si la propiedad de coincidencia está vacía o no se ha rellenado para un grupo en Azure AD.
## <a name="provisioning-users-assigned-to-the-default-access-role"></a>Aprovisionamiento de usuarios a los que se asigna el rol de acceso predeterminado
El rol predeterminado en una aplicación de la galería se denomina rol de "acceso predeterminado". Históricamente, los usuarios a los que se asigna este rol no se aprovisionan y se marcan como omitidos en los [registros de aprovisionamiento](../reports-monitoring/concept-provisioning-logs.md) por "no estar autorizados de forma efectiva". 

**Comportamiento de las configuraciones de aprovisionamiento creadas después del 16/04/2020:** los usuarios a los que se asigne el rol de acceso predeterminado se evaluarán igual que los usuarios de los demás roles. Un usuario que tenga asignado el acceso predeterminado no se omitirá como "no autorizado de forma efectiva". 

**Comportamiento de las configuraciones de aprovisionamiento creadas antes del 16/04/2020:** durante los próximos 3 meses, el comportamiento continuará igual que hoy. Los usuarios con el rol de acceso predeterminado se omitirán por no estar autorizados de forma efectiva. Después de julio de 2020, el comportamiento será uniforme para todas las aplicaciones. No se omitirá el aprovisionamiento de usuarios con el rol de acceso predeterminado por "no estar autorizados de forma efectiva". Este cambio lo realizará Microsoft, sin necesidad de que el cliente realice ninguna acción. Si desea asegurarse de que estos usuarios se sigan omitiendo incluso después de este cambio, aplique los filtros de ámbito adecuados o anule la asignación del usuario de la aplicación para garantizar que queden fuera del ámbito.  

Si tiene preguntas acerca de estos cambios, póngase en contacto con provisioningfeedback@microsoft.com.
## <a name="next-steps"></a>Pasos siguientes

[Sincronización de Azure AD Connect: Descripción del aprovisionamiento declarativo](../hybrid/concept-azure-ad-connect-sync-declarative-provisioning.md)