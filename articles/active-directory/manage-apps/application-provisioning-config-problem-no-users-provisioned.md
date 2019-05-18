---
title: No se aprovisionan usuarios en una aplicación de la galería de Azure AD | Microsoft Docs
description: Cómo solucionar problemas habituales que surgen cuando no se ve aparecer a usuarios en una aplicación de la galería de Azure AD que ha configurado para el aprovisionamiento de usuarios con Azure AD
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
ms.topic: conceptual
ms.date: 09/20/2018
ms.author: mimart
ms.reviewer: asteen
ms.collection: M365-identity-device-management
ms.openlocfilehash: eaeb97f88c2482cb9d091afb1c205e9b09a85ce0
ms.sourcegitcommit: 36c50860e75d86f0d0e2be9e3213ffa9a06f4150
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 05/16/2019
ms.locfileid: "65784585"
---
# <a name="no-users-are-being-provisioned-to-an-azure-ad-gallery-application"></a>No se aprovisionan usuarios en una aplicación de la galería de Azure AD
Una vez que se ha configurado el aprovisionamiento automático de una aplicación (incluida la comprobación de que las credenciales de la aplicación proporcionadas a Azure AD para conectarse a la aplicación son válidas), se aprovisionan los usuarios y los grupos en la aplicación. El aprovisionamiento se determina mediante lo siguiente:

-   Qué usuarios y grupos se han **asignado** a la aplicación. Para más información sobre la asignación, consulte [Asignación de un usuario o un grupo a una aplicación empresarial en Azure Active Directory](assign-user-or-group-access-portal.md).
-   Si las **asignaciones de atributos** están habilitadas o no, y configuradas para sincronizar atributos válidos de Azure AD a la aplicación. Para más información sobre las asignaciones de atributos, consulte [Personalización de asignaciones de atributos de aprovisionamiento de usuarios para aplicaciones SaaS en Azure Active Directory de usuarios](customize-application-attributes.md).
-   Si hay o no presente un **filtro de ámbito** que filtre usuarios en función de valores de atributo concretos. Para más información sobre los filtros de ámbito, consulte [Aprovisionamiento de aplicaciones basado en atributos con filtros de ámbito](define-conditional-rules-for-provisioning-user-accounts.md).
  
Si observa que no se están aprovisionando usuarios, consulte los registros de auditoría de Azure AD. Busque las entradas del registro para un usuario específico.

Puede acceder a los registros de auditoría de aprovisionamiento en Azure Portal, en la pestaña **Azure Active Directory &gt; Aplicaciones empresariales &gt; \[Nombre de la aplicación\] &gt; Registros de auditoría**. Filtre los registros en la categoría **Aprovisionamiento de cuentas** para ver solo los eventos de aprovisionamiento para esa aplicación. Puede buscar usuarios por el "identificador de coincidencia" que se configuró para ellos en las asignaciones de atributos. Por ejemplo, si configuró el "nombre principal de usuario" o la "dirección de correo electrónico" como atributo de coincidencia en el lado de Azure AD y el usuario que no se aprovisiona tiene el valor "audrey@contoso.com", busque "audrey@contoso.com" en los registros de auditoría y revise las entradas devueltas.

Los registros de auditoría de aprovisionamiento registran todas las operaciones realizadas por el servicio de aprovisionamiento, lo que incluye consultar a Azure AD sobre los usuarios asignados que están en el ámbito de aprovisionamiento, consultar en la aplicación de destino la existencia de esos usuarios y comparar los objetos de usuario entre el sistema. Después, agregue, actualice o deshabilite la cuenta de usuario en el sistema de destino en función de la comparación.

## <a name="general-problem-areas-with-provisioning-to-consider"></a>Áreas problemáticas generales con el aprovisionamiento que tener en cuenta
A continuación, se muestra una lista de las áreas problemáticas generales en las que puede profundizar si tiene idea de por dónde empezar.

- [No parece que el servicio de aprovisionamiento se inicie](#provisioning-service-does-not-appear-to-start)
- [Los registros de auditoría indican que hay usuarios omitidos y no aprovisionados, aunque estén asignados](#audit-logs-say-users-are-skipped-and-not-provisioned-even-though-they-are-assigned)

## <a name="provisioning-service-does-not-appear-to-start"></a>No parece que el servicio de aprovisionamiento se inicie
Si establece **Estado de aprovisionamiento** en **Activado** en la sección **Azure Active Directory &gt; Aplicaciones empresariales &gt; \[Nombre de la aplicación\] &gt;Aprovisionamiento** de Azure Portal. Sin embargo, no se muestran más detalles de estado en esa página después de recargas posteriores; es probable que el servicio se esté ejecutando pero que no haya completado aún una sincronización inicial. Compruebe los **registros de auditoría** descritos antes para determinar qué operaciones está realizando el servicio y si se han producido errores.

>[!NOTE]
>Una sincronización inicial puede tardar desde 20 minutos hasta varias horas, según el tamaño del directorio de Azure AD y el número de usuarios en el ámbito de aprovisionamiento. Las sincronizaciones posteriores a la inicial son más rápidas, ya que el servicio de aprovisionamiento almacena marcas de agua que representan el estado de ambos sistemas tras la sincronización inicial. La sincronización inicial mejora el rendimiento de las posteriores.
>


## <a name="audit-logs-say-users-are-skipped-and-not-provisioned-even-though-they-are-assigned"></a>Los registros de auditoría indican que hay usuarios omitidos y no aprovisionados, aunque estén asignados

Cuando un usuario se muestra como "Omitido" en los registros de auditoría, es importante que lea los detalles ampliados en el mensaje del registro para determinar la razón. Algunas razones y soluciones habituales son:

- **Se ha configurado un filtro de ámbito** **que está filtrando al usuario por un valor de atributo**. Para más información sobre los filtros de ámbito, consulte los [filtros de ámbito](define-conditional-rules-for-provisioning-user-accounts.md).
- **El usuario no está autorizado de forma efectiva.** Si ve un mensaje de error de este tipo, se debe a que hay un problema con el registro de asignación de usuarios almacenado en Azure AD. Para corregir este problema, desasigne el usuario (o grupo) de la aplicación y vuelva a asignarlo. Para más información sobre la asignación, consulte [Asignación de acceso a usuarios o grupos](assign-user-or-group-access-portal.md).
- **Un atributo obligatorio falta o no se ha llenado para un usuario.** Una cuestión importante que tener en cuenta al configurar el aprovisionamiento es revisar y configurar las asignaciones de atributos y los flujos de trabajo que definen qué propiedades de usuario (o de grupo) fluyen de Azure AD a la aplicación. Esta configuración incluye la opción de "propiedad de coincidencia" que se usa para identificar de forma exclusiva y emparejar a usuarios y grupos entre ambos sistemas. Para más información sobre este importante proceso, consulte [Personalización de asignaciones de atributos de aprovisionamiento de usuarios para aplicaciones SaaS en Azure Active Directory de usuarios](customize-application-attributes.md).
- **Asignación de atributos para grupos:** Aprovisionamiento del nombre del grupo y los detalles del grupo, además de los miembros, si se admite para algunas aplicaciones. Puede habilitar o deshabilitar esta funcionalidad habilitando o deshabilitando la **Asignación** para los objetos de grupo que se muestran en la pestaña **Aprovisionamiento**. Si los grupos de aprovisionamiento están habilitados, asegúrese de revisar las asignaciones de atributos para asegurarse de que se use un campo apropiado para el "identificador de coincidencia". El identificador de coincidencia puede ser el alias de correo electrónico o el nombre para mostrar. El grupo y sus miembros no se han aprovisionado si la propiedad de coincidencia está vacía o no se ha rellenado para un grupo en Azure AD.

## <a name="next-steps"></a>Pasos siguientes

[Sincronización de Azure AD Connect: Descripción del aprovisionamiento declarativo](../hybrid/concept-azure-ad-connect-sync-declarative-provisioning.md)
