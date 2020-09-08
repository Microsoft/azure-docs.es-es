---
title: 'Resolución de problemas de asignación de licencias para grupos: Azure Active Directory | Microsoft Docs'
description: Identificación y resolución de problemas de asignación de licencias cuando se utilizan licencias basadas en grupos con Azure Active Directory
services: active-directory
keywords: Licencias de Azure AD
documentationcenter: ''
author: curtand
manager: daveba
ms.service: active-directory
ms.subservice: users-groups-roles
ms.topic: how-to
ms.workload: identity
ms.date: 11/08/2019
ms.author: curtand
ms.reviewer: sumitp
ms.custom: it-pro
ms.collection: M365-identity-device-management
ms.openlocfilehash: f0d7703af48ba33edea81ca45516191266a79fa4
ms.sourcegitcommit: c5021f2095e25750eb34fd0b866adf5d81d56c3a
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/25/2020
ms.locfileid: "88799560"
---
# <a name="identify-and-resolve-license-assignment-problems-for-a-group-in-azure-active-directory"></a>Identificación y resolución de problemas de asignación de licencias de un grupo en Azure Active Directory

Las licencias basadas en grupos de Azure Active Directory (Azure AD) incorpora el concepto de usuarios en estado de error de licencias. En este artículo, se explica los motivos por los que los usuarios pueden terminar en este estado.

Cuando se asignan licencias directamente a usuarios individuales, sin usar las licencias basadas en grupos, la operación de asignación puede generar errores. Por ejemplo, al ejecutar el cmdlet de PowerShell `Set-MsolUserLicense` en un sistema del usuario, el cmdlet puede generar un error por diversos motivos relacionados con la lógica empresarial. Por ejemplo, podría haber un número insuficiente de licencias o un conflicto entre dos planes de servicio que no se puedan asignar al mismo tiempo. El problema se le notifica inmediatamente.

Cuando se usan licencias basadas en grupo, se pueden producir los mismos errores, pero se producen en segundo plano mientras el servicio de Azure AD está asignando las licencias. Por este motivo, los errores no se comunican inmediatamente. En su lugar, los errores se registran en el objeto de usuario y se notifican a través del portal de administración. Nunca se pierde la intención original de asignar la licencia al usuario, pero se registra en estado de error a efectos de futuras investigaciones y resoluciones.

## <a name="find-license-assignment-errors"></a>Búsqueda de errores de asignación de licencias

### <a name="to-find-users-in-an-error-state-in-a-group"></a>Para buscar usuarios con estado de error en un grupo

1. Abra el grupo en su página de información general y seleccione **Licencias**. Si hay usuarios con estado de error, aparece una notificación.

   ![Mensaje de notificaciones de grupo y de error](./media/licensing-groups-resolve-problems/group-error-notification.png)

1. Seleccione la notificación para abrir una lista de todos los usuarios afectados. Puede seleccionar individualmente cada usuario para ver más detalles.

   ![lista de usuarios en estado de error de licencia de grupo](./media/licensing-groups-resolve-problems/list-of-users-with-errors.png)

1. Para buscar todos los grupos que contienen al menos un error, en la hoja **Azure Active Directory** seleccione **Licencias** y, a continuación, seleccione **Información general**. Si algunos grupos requieren atención, aparece un cuadro de información.

   ![Introducción general y detalles sobre los grupos con estado de error](./media/licensing-groups-resolve-problems/group-errors-widget.png)

1. Seleccione el cuadro para ver una lista de todos los grupos con errores. Puede seleccionar cada grupo para más detalles.

   ![Información general y lista de grupos con errores](./media/licensing-groups-resolve-problems/list-of-groups-with-errors.png)

En las secciones siguientes se muestra una descripción de cada problema potencial y la manera de resolverlo.

## <a name="not-enough-licenses"></a>No hay suficientes licencias

**Problema:** No hay suficientes licencias disponibles para uno de los productos especificados en el grupo. Necesita adquirir más licencias para el producto o liberar las licencias sin usar de otros usuarios o grupos.

Para ver cuántas licencias hay disponibles, vaya a **Azure Active Directory** > **Licencias** > **Todos los productos**.

Para ver qué usuarios y grupos consumen las licencias, seleccione un producto. En **Usuarios con licencias**, puede ver una lista de todos los usuarios a los que se han asignado licencias directamente o a través de uno o varios grupos. En **Grupos con licencias**, puede ver todos los grupos que tienen ese producto asignado.

**PowerShell:** Los cmdlets de PowerShell informan de este error como _CountViolation_.

## <a name="conflicting-service-plans"></a>Planes de servicio en conflicto

**Problema:** Uno de los productos especificados en el grupo contiene un plan de servicio que entra en conflicto con otro plan de servicio que ya está asignado al usuario a través de un producto diferente. Algunos planes de servicio se configuran de tal forma que no puedan asignarse al mismo usuario como otro plan de servicio relacionado.

Considere el ejemplo siguiente. Un usuario tiene una licencia de Office 365 Enterprise *E1* asignada directamente, con todos los planes habilitados. Se ha agregado el usuario a un grupo que tiene asignado el producto Office 365 Enterprise *E3*. El producto E3 contiene planes de servicio que no pueden superponerse con los planes incluidos en E1, por lo que la asignación de licencia de grupo genera el error “Planes de servicio en conflicto”. En este ejemplo, los planes de servicio en conflicto son:

- SharePoint Online (Plan 2) entra en conflicto con SharePoint Online (Plan 1).
- Exchange Online (Plan 2) entra en conflicto con Exchange Online (Plan 1).

Para resolver este conflicto, debe deshabilitar dos de los planes. Puede deshabilitar la licencia de E1 que se ha asignado directamente al usuario. O bien, debe modificar toda la asignación de licencias de grupo y deshabilitar los planes de la licencia de E3. Como alternativa, puede quitar la licencia de E1 al usuario si es redundante en el contexto de la licencia de E3.

El administrador es la única persona competente para decidir cómo resolver el conflicto entre las licencias de productos. Azure AD no resuelve automáticamente los conflictos de licencias.

**PowerShell:** Los cmdlets de PowerShell informan de este error como _MutuallyExclusiveViolation_.

## <a name="other-products-depend-on-this-license"></a>Otros productos dependen de esta licencia

**Problema:** Uno de los productos que se especifica en el grupo contiene un plan de servicio que debe estar habilitado para otro plan de servicio, en otro producto, para funcionar. Este error se produce cuando Azure AD intenta quitar el plan del servicio subyacente. Por ejemplo, esto puede ocurrir cuando se elimina el usuario del grupo.

Para solucionar este problema, debe asegurarse de que el plan necesario todavía está asignado a los usuarios a través de algún otro método o que los servicios dependientes están deshabilitados para esos usuarios. Después, puede quitar correctamente la licencia de grupo a esos usuarios.

**PowerShell:** Los cmdlets de PowerShell informan de este error como _DependencyViolation_.

## <a name="usage-location-isnt-allowed"></a>No se permite la ubicación de uso

**Problema:** Algunos servicios de Microsoft no están disponibles en todas las ubicaciones debido a las leyes y los reglamentos locales. Antes de poder asignar una licencia a un usuario, debe especificar la propiedad **Ubicación de uso** para el usuario. Puede especificar la ubicación en la sección **Usuario** > **Perfil** > **Configuración** de Azure Portal.

Cuando Azure AD intenta asignar una licencia de grupo a un usuario cuya ubicación de uso no se admite, se produce un error y se registra en el usuario.

Para solucionar este problema, quite del grupo con licencia a los usuarios de las ubicaciones no admitidas. O bien, si los valores de ubicación de uso actual no representan la ubicación de los usuarios reales, puede modificarlos para que la próxima vez las licencias se asignen correctamente (si se admite la nueva ubicación).

**PowerShell:** Los cmdlets de PowerShell informan de este error como _ProhibitedInUsageLocationViolation_.

> [!NOTE]
> Cuando Azure AD asigna licencias de grupo, los usuarios sin ubicación de uso especificada heredan la ubicación del directorio. Se recomienda que los administradores establezcan valores de ubicación de uso correctos en los usuarios antes de utilizar licencias basadas en grupo para cumplir con la normativa y la legislación local.

## <a name="duplicate-proxy-addresses"></a>Direcciones proxy duplicadas

Si usa Exchange Online, es posible que algunos de los usuarios de la organización no estén configurados correctamente con el mismo valor de dirección de proxy. Cuando el sistema de licencias basadas en grupos intenta asignar una licencia a un usuario de este tipo, se produce un error y se muestra un mensaje que indica que la dirección proxy ya está en uso.

> [!TIP]
> Para ver si hay una dirección del proxy duplicada, ejecute el siguiente cmdlet de PowerShell en Exchange Online:
> ```
> Get-Recipient -ResultSize unlimited | where {$_.EmailAddresses -match "user@contoso.onmicrosoft.com"} | fL Name, RecipientType,emailaddresses
> ```
> Para más información acerca de este problema, consulte [Mensaje de error "la dirección del proxy ya está en uso" en Exchange Online](https://support.microsoft.com/help/3042584/-proxy-address-address-is-already-being-used-error-message-in-exchange-online). El artículo también incluye información sobre [cómo conectarse a Exchange Online mediante PowerShell remoto](/powershell/exchange/connect-to-exchange-online-powershell?view=exchange-ps).

Después de resolver los problemas de dirección del proxy para los usuarios afectados, asegúrese de forzar el procesamiento de la licencia en el grupo para asegurarse de que ahora se pueden aplicar las licencias.

## <a name="azure-ad-mail-and-proxyaddresses-attribute-change"></a>Cambio de los atributos Mail y ProxyAddresses de Azure AD

**Problema:** al actualizar la asignación de licencias en un grupo o un usuario, es posible que vea que se han cambiado los atributos Mail y ProxyAddresses de Azure AD de algunos usuarios.

Si actualiza la asignación de licencias en un usuario, se desencadenará el cálculo de dirección proxy, lo que puede provocar un cambio en los atributos de usuario. Para comprender el motivo exacto del cambio y resolver el problema, consulte este artículo [Cómo se rellena el atributo proxyAddresses en Azure AD](https://support.microsoft.com/help/3190357/how-the-proxyaddresses-attribute-is-populated-in-azure-ad).

## <a name="licenseassignmentattributeconcurrencyexception-in-audit-logs"></a>LicenseAssignmentAttributeConcurrencyException en registros de auditoría

**Problema:** El usuario tiene LicenseAssignmentAttributeConcurrencyException como asignación de licencia en los registros de auditoría.
Cuando la licencia basada en grupos intenta procesar la asignación simultánea de la misma licencia para un usuario, se registra esta excepción en el usuario. Esto suele suceder cuando un usuario es miembro de más de un grupo con la misma licencia asignada. Azure AD reintentará procesar la licencia de usuario y resolverá el problema. No es necesario que el cliente tome ninguna medida para corregir este problema.

## <a name="more-than-one-product-license-assigned-to-a-group"></a>Más de una licencia de producto asignada a un grupo

Puede asignar más de una licencia de producto a un grupo. Por ejemplo, puede asignar Office 365 Enterprise E3 y Enterprise Mobility + Security a un grupo para habilitar fácilmente todos los servicios incluidos para los usuarios.

Azure AD intenta asignar todas las licencias especificadas en el grupo a cada usuario. Si Azure AD no puede asignar uno de los productos debido a problemas de lógica de negocio, tampoco asignará las otras licencias del grupo. Un ejemplo se da si no hay suficientes licencias para todos o si hay conflictos con otros servicios que están habilitados en el usuario.

Puede ver los usuarios con los que se han producido errores de asignación y comprobar a qué productos ha afectado esta situación.

## <a name="when-a-licensed-group-is-deleted"></a>Cuando se elimina un grupo con licencia

Debe quitar todas las licencias asignadas a un grupo antes de poder eliminar el grupo. Sin embargo, quitar las licencias de todos los usuarios en el grupo puede llevar tiempo. Al quitar las asignaciones de licencias de un grupo, puede haber errores si el usuario tiene asignada una licencia dependiente o si hay un problema de conflicto de dirección de proxy que prohíbe la eliminación de la licencia. Si un usuario tiene una licencia que depende de una licencia que se va a quitar debido a la eliminación del grupo, la asignación de la licencia para el usuario se convierte de heredada a directa.

Por ejemplo, piense en un grupo que tiene asignado Office 365 E3/E5 con un plan de servicio de Skype Empresarial habilitado. Imagine también que algunos miembros del grupo tienen licencias de Audioconferencia asignadas directamente. Cuando se elimina el grupo, las licencias basadas en el grupo intentarán quitar Office 365 E3/E5 de todos los usuarios. Dado que Audioconferencia depende de Skype Empresarial, para los usuarios con Audioconferencia asignada, las licencias basadas en grupos convierten las licencias de Office 365 E3/E5 a una asignación de licencias directa.

## <a name="manage-licenses-for-products-with-prerequisites"></a>Administración de licencias para productos con requisitos previos

Algunos productos de Microsoft Online que puede tener son *complementos*. Los complementos precisan de un plan de servicio de requisitos previos habilitado para un usuario o un grupo antes de poder asignarles una licencia. Cuando se usan licencias basadas en grupo, el sistema solicita que mantenga en el mismo grupo los planes de servicios de requisitos previos y de complementos. Esto se hace para garantizar que los usuarios que se agregan al grupo puedan recibir el producto de trabajo completo. Vea el siguiente ejemplo:

Microsoft Workplace Analytics es un producto complementario. Contiene un plan de servicio único con el mismo nombre. Este plan de servicio solo se puede asignar a un usuario o grupo cuando uno de los siguientes requisitos previos se asignan también:

- Exchange Online (plan 1)
- Exchange Online (plan 2)

Si se intenta asignar este producto por sí solo a un grupo, el portal devuelve un mensaje de notificación. Al seleccionar los detalles del elemento, se muestra el siguiente mensaje de error:

  "Error en la operación de la licencia. Asegúrese de que el grupo tiene los servicios necesarios antes de agregar o quitar un servicio dependiente. **El servicio Microsoft Workplace Analytics necesita que Exchange Online (plan 2) también esté habilitado**".

Para asignar esta licencia de complemento a un grupo, es necesario asegurarse de que el grupo también contiene el plan de servicio de requisitos previos. Por ejemplo, es posible actualizar un grupo existente que ya contenga el producto Office 365 E3 completo y, a continuación, agregar al mismo el complemento.

También es posible crear un grupo independiente que contenga solo los productos mínimos necesarios para que el complemento funcione. Después se puede usar para proporcionar la licencia del producto complementario solo a los usuarios seleccionados. Según el ejemplo anterior, asignaría los siguientes productos al mismo grupo:

- Office 365 Enterprise E3, solo con el plan de servicio Exchange Online (plan 2) habilitado
- Microsoft Workplace Analytics

De ahora en adelante, cualquier usuario que se agregue a este grupo utiliza una licencia del producto E3 y una licencia del producto Workplace Analytics. Al mismo tiempo, esos usuarios pueden formar parte de otro grupo que les proporcione acceso a todo el producto E3 y solo utilizan una licencia de ese producto.

> [!TIP]
> Puede crear varios grupos para cada plan de servicio de requisitos previos. Por ejemplo, si los usuarios usan las versiones Office 365 Enterprise E1 y Office 365 Enterprise E3, puede crear dos grupos para proporcionar licencias de Microsoft Workplace Analytics: una con E1 como requisito previo y la otra con E3. Esto le permite distribuir el complemento a los usuarios de E1 y E3 sin tener que usar licencias adicionales.

## <a name="force-group-license-processing-to-resolve-errors"></a>Forzado del procesamiento de licencias del grupo para resolver errores

Dependiendo de qué pasos haya dado para resolver los errores, puede ser necesario desencadenar manualmente el procesamiento de un grupo para actualizar el estado del usuario.

Por ejemplo, si libera algunas licencias quitando asignaciones de licencia directas de usuarios, debe desencadenar el procesamiento de grupos con los que anteriormente se produjeron errores por asignar licencias íntegramente a todos los miembros. Para volver a procesar un grupo, vaya al panel del grupo, abra **Licencias** y, a continuación, seleccione el botón **Volver a procesar** en la barra de herramientas.

## <a name="force-user-license-processing-to-resolve-errors"></a>Forzado del procesamiento de licencias del usuario para resolver errores

Dependiendo de qué pasos haya dado para resolver los errores, puede ser necesario desencadenar manualmente el procesamiento de un usuario para actualizar el estado del usuario.

Por ejemplo, después de resolver el problema con la dirección proxy duplicada en un usuario afectado, debe desencadenar el procesamiento del usuario. Para volver a procesar un usuario, vaya al panel del usuario, abra **Licencias** y, a continuación, seleccione el botón **Volver a procesar** en la barra de herramientas.

## <a name="next-steps"></a>Pasos siguientes

Para más información sobre otros escenarios de administración de licencias a través de grupos, consulte lo siguiente:

* [¿En qué consisten las licencias basadas en grupos de Azure Active Directory?](../fundamentals/active-directory-licensing-whatis-azure-portal.md)
* [Asignación de licencias a un grupo en Azure Active Directory](licensing-groups-assign.md)
* [Migración de usuarios individuales con licencia a licencias basadas en grupos en Azure Active Directory](licensing-groups-migrate-users.md)
* [Cómo migrar usuarios entre diferentes licencias de productos con licencias basadas en grupos de Azure Active Directory](licensing-groups-change-licenses.md)
* [Escenarios adicionales de licencias basadas en grupos de Azure Active Directory](licensing-group-advanced.md)
* [Ejemplos de PowerShell para licencias basadas en grupos de Azure AD](licensing-ps-examples.md)