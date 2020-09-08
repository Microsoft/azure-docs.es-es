---
title: 'Solución de problemas de la administración de derechos: Azure AD'
description: Aprenda sobre algunos elementos que debe comprobar para solucionar los problemas de administración de derechos de Azure Active Directory.
services: active-directory
documentationCenter: ''
author: barclayn
manager: daveba
editor: markwahl-msft
ms.service: active-directory
ms.workload: identity
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: troubleshooting
ms.subservice: compliance
ms.date: 06/17/2020
ms.author: barclayn
ms.reviewer: markwahl-msft
ms.collection: M365-identity-device-management
ms.openlocfilehash: 216cff03ac6ce64dee9aae1e9daa4a86385eeb0e
ms.sourcegitcommit: 9c3cfbe2bee467d0e6966c2bfdeddbe039cad029
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/24/2020
ms.locfileid: "88783338"
---
# <a name="troubleshoot-azure-ad-entitlement-management"></a>Solución de problemas de la administración de derechos de Azure AD

En este artículo se describen algunos elementos que debe comprobar que le ayudarán a solucionar problemas de administración de derechos de Azure Active Directory (Azure AD).

## <a name="administration"></a>Administración

* Si recibe un mensaje acceso denegado al configurar la administración de derechos y es administrador global, asegúrese de que el directorio tiene [licencia de Azure AD Premium P2 (o EMS E5)](entitlement-management-overview.md#license-requirements).

* Si recibe un mensaje de acceso denegado al crear o ver los paquetes de acceso y es miembro de un grupo de creadores de catálogos, debe [crear un catálogo](entitlement-management-catalog-create.md) antes de crear el primer paquete de acceso.

## <a name="resources"></a>Recursos

* Los roles de las aplicaciones se definen en la propia aplicación y se administran en Azure AD. Si una aplicación no tiene ningún rol de recurso, la administración de derechos asigna a los usuarios un rol de **Acceso predeterminado**.

    Tenga en cuenta que en Azure Portal también se pueden mostrar las entidades de servicio de los servicios que no se pueden seleccionar como aplicaciones.  En concreto, **Exchange Online** y **SharePoint Online** son servicios, no aplicaciones que tengan roles de recursos en el directorio, por lo que no pueden incluirse en un paquete de acceso.  En su lugar, use las licencias de grupo para establecer una licencia adecuada para un usuario que necesite acceder a esos servicios.

* Para que un grupo sea un recurso en un paquete de acceso, debe poder modificarse en Azure AD.  Los grupos que se originan en una instancia local de Active Directory no pueden asignarse como recursos porque su propietario o los atributos de miembro no se pueden cambiar en Azure AD.   Los grupos que se originan en Exchange Online como grupos de distribución tampoco se pueden modificar en Azure AD. 

* Las bibliotecas de documentos de SharePoint Online y los documentos individuales no se pueden agregar como recursos.  En su lugar, cree un [grupo de seguridad de Azure AD](../fundamentals/active-directory-groups-create-azure-portal.md), incluya ese grupo y un rol de sitio en el paquete de acceso y, en SharePoint Online, use ese grupo para controlar el acceso a la biblioteca de documentos o al documento.

* Si hay usuarios que ya se han asignado a un recurso que desee administrar con un paquete de acceso, asegúrese de que los usuarios están asignados al paquete de acceso con una directiva adecuada. Por ejemplo, puede incluir un grupo en un paquete de acceso que ya tenga usuarios en el grupo. Si esos usuarios del grupo requieren acceso continuado, deben tener una directiva adecuada para los paquetes de acceso, de manera que no se queden sin acceso de grupo. Para asignar el paquete de acceso, puede pedir a los usuarios que soliciten el paquete de acceso que contiene el recurso o asignárselo directamente. Para más información, consulte [Modificación de la configuración de solicitudes y aprobación de un paquete de acceso](entitlement-management-access-package-request-policy.md).

* Cuando se quita un miembro de un equipo, también se quita del grupo de Microsoft 365. Puede pasar algún tiempo hasta que se elimine la funcionalidad de chat del equipo. Para más información, consulte [Pertenencia a grupos](/microsoftteams/office-365-groups#group-membership).

* Asegúrese de que el directorio no esté configurado para ubicaciones multigeográficas. La administración de derechos no admite actualmente ubicaciones multigeográficas en SharePoint Online. Los sitios de SharePoint Online deben estar en la ubicación geográfica predeterminada para regirse por la administración de derechos. Para más información, consulte [Capacidades multigeográficas de OneDrive y SharePoint Online](/Office365/Enterprise/multi-geo-capabilities-in-onedrive-and-sharepoint-online-in-office-365).

## <a name="access-packages"></a>Paquetes de acceso

* Si intenta eliminar una directiva o un paquete de acceso y ve un mensaje de error que indica que hay asignaciones activas, si no ve ningún usuario con asignaciones, compruebe si los usuarios eliminados recientemente todavía tienen asignaciones. Durante el período de 30 días después de la eliminación de un usuario, se puede restaurar la cuenta de usuario.   

## <a name="external-users"></a>Usuarios externos

* Cuando un usuario externo quiera solicitar acceso a un paquete de acceso, asegúrese de que use el **vínculo del portal Mi acceso** correspondiente al paquete de acceso. Para más información, consulte [Uso compartido de un vínculo para solicitar un paquete de acceso](entitlement-management-access-package-settings.md). Si un usuario externo visita simplemente **myaccess.microsoft.com** y no usa el vínculo completo del portal Mi acceso, dicho usuario verá que los paquetes de acceso están disponibles para él en su propia organización y no en la de usted.

* Si un usuario externo no puede solicitar acceso a un paquete de acceso o no puede acceder a los recursos, asegúrese de comprobar la [configuración de usuarios externos](entitlement-management-external-users.md#settings-for-external-users).

* Si un nuevo usuario externo que no ha iniciado sesión en el directorio antes recibe un paquete de acceso que incluya un sitio de SharePoint Online, este se mostrará como que no se ha entregado totalmente hasta que la cuenta esté aprovisionada en SharePoint Online. Para más información sobre la configuración de uso compartido, consulte [Revisión de la configuración de uso compartido externo de SharePoint Online](entitlement-management-external-users.md#review-your-sharepoint-online-external-sharing-settings).

## <a name="requests"></a>Requests

* Cuando un usuario desee solicitar acceso a un paquete de acceso, asegúrese de que está utilizando el **vínculo del portal Mi acceso** para el paquete de acceso. Para más información, consulte [Uso compartido de un vínculo para solicitar un paquete de acceso](entitlement-management-access-package-settings.md).

* Si abre el portal Mi acceso con el explorador establecido en el modo InPrivate o incógnito, se podría producir un conflicto con el comportamiento de inicio de sesión. Se recomienda no usar ninguno de estos modos en el explorador cuando visite el portal Mi acceso.

* Cuando un usuario que aún no está en el directorio inicia sesión en el portal Mi acceso para solicitar un paquete de acceso, asegúrese de que se autentique con su cuenta profesional. La cuenta profesional puede ser una cuenta del directorio de recursos o de un directorio incluido en una de las directivas del paquete de acceso. Si la cuenta de usuario no es una cuenta profesional o si el directorio donde se autentica no está incluido en la directiva, el usuario no verá el paquete de acceso. Para más información, consulte [Solicitud de acceso a un paquete acceso](entitlement-management-request-access.md).

* Si un usuario tiene un bloqueo de inicio de sesión al directorio de recursos, no podrá acceder a la solicitud de acceso del portal Mi acceso. Para que pueda solicitar el acceso, debe eliminar el bloqueo de inicio de sesión de su perfil. Para quitar el bloqueo de inicio de sesión en Azure Portal, haga clic en **Azure Active Directory**, en **Usuarios**, en el usuario y, finalmente, en **Perfil**. Edite la sección **Configuración** y cambie **Bloquear inicio de sesión** a **No**. Para más información, consulte [Incorporación o actualización de la información de perfil de un usuario mediante Azure Active Directory](../fundamentals/active-directory-users-profile-azure-portal.md).  También puede comprobar si el usuario se bloqueó debido a una [directiva de protección de identidad](../identity-protection/howto-identity-protection-remediate-unblock.md).

* En el portal Mi acceso, si un usuario es tanto solicitante como aprobador, no verá su solicitud para un paquete de acceso en la página **Aprobaciones**. Este comportamiento es deliberado: un usuario no puede aprobar sus propias solicitudes. Asegúrese de que el paquete de acceso que solicita tiene otros aprobadores configurados en la directiva. Para más información, consulte [Modificación de la configuración de solicitudes y aprobación de un paquete de acceso](entitlement-management-access-package-request-policy.md).

### <a name="view-a-requests-delivery-errors"></a>Visualización de errores de entrega de una solicitud

**Rol necesario:** Administrador global, administrador de usuarios, propietario del catálogo o administrador de paquetes de acceso.

1. En Azure Portal, haga clic en **Azure Active Directory** y, luego, en **Gobernanza de identidades**.

1. En el menú de la izquierda, haga clic en **Paquetes de acceso** y luego abra el paquete de acceso.

1. Haga clic en **Solicitudes**.

1. Seleccione la solicitud que quiere ver.

    Si la solicitud tiene errores de entrega, su estado será **Undelivered** (Sin enviar) y su subestado, **Entregado parcialmente**.

    Si hay errores de entrega, en el panel de detalles de la solicitud se mostrará un recuento de los errores de entrega.

1. Haga clic en el recuento para ver todos los errores de entrega de la solicitud.

### <a name="reprocess-a-request"></a>Nuevo procesamiento de una solicitud

Si se encuentra un error después de desencadenar una solicitud de reprocesamiento de un paquete de acceso, debe esperar mientras el sistema vuelve a procesar la solicitud. El sistema vuelve a intentar el procesamiento varias veces durante varias horas, por lo que no se puede forzar el reprocesamiento durante este tiempo. 

Solo se pueden volver a procesar las solicitudes que tienen el estado **Error en la entrega** o **Entregado parcialmente** y una fecha de finalización inferior a una semana. De lo contrario, el botón **Reprocesar** estará atenuado.

![Botón Reprocesar atenuado](./media/entitlement-management-troubleshoot/cancel-reprocess-grayedout.png)

- Si el error se corrige durante la ventana de pruebas, el estado de la solicitud cambiará a **Entrega**. La solicitud se volverá a procesar sin acciones adicionales por parte del usuario.

- Si el error no se corrige durante la ventana de pruebas, el estado de la solicitud puede ser **Error en la entrega** o **Entregado parcialmente**. A continuación, puede usar el botón **Reprocesar**. Tendrá siete días para volver a procesar la solicitud.

**Rol necesario:** Administrador global, administrador de usuarios, propietario del catálogo o administrador de paquetes de acceso.

1. En Azure Portal, haga clic en **Azure Active Directory** y, luego, en **Gobernanza de identidades**.

1. En el menú de la izquierda, haga clic en **Paquetes de acceso** y luego abra el paquete de acceso.

1. Haga clic en **Solicitudes**.

1. Haga clic en la solicitud que desee procesar de nuevo.

1. En el panel de detalles de la solicitud, haga clic en **Volver a procesar solicitud**.

    ![Nuevo procesamiento de una solicitud que no se ha ejecutado correctamente](./media/entitlement-management-troubleshoot/reprocess-request.png)

### <a name="cancel-a-pending-request"></a>Cancelación de una solicitud pendiente

Solo se pueden cancelar las solicitudes pendientes que aún no se hayan entregado o en cuya entrega se hayan producido errores. De lo contrario, el botón **Cancelar** estará atenuado.

**Rol necesario:** Administrador global, administrador de usuarios, propietario del catálogo o administrador de paquetes de acceso.

1. En Azure Portal, haga clic en **Azure Active Directory** y, luego, en **Gobernanza de identidades**.

1. En el menú de la izquierda, haga clic en **Paquetes de acceso** y luego abra el paquete de acceso.

1. Haga clic en **Solicitudes**.

1. Haga clic en la solicitud que desee cancelar.

1. En el panel de detalles de la solicitud, haga clic en **Cancelar solicitud**.

## <a name="multiple-policies"></a>Varias directivas

* La administración de derechos sigue los procedimientos recomendados de privilegios mínimos. Cuando un usuario solicita acceso a un paquete de acceso al que se aplican varias directivas, la administración de derechos incluye lógica para garantizar que las directivas más estrictas o más específicas tengan prioridad sobre las directivas genéricas. Si una directiva es genérica, la administración de derechos podría no mostrarla al solicitante o podría seleccionar automáticamente una directiva más estricta.

* Por ejemplo, piense en un paquete de acceso con dos directivas para los empleados internos en los que ambas directivas se aplican al solicitante. La primera directiva es para usuarios específicos que incluyen el solicitante. La segunda directiva es para todos los usuarios de un directorio del que es miembro el solicitante. En este escenario, se selecciona automáticamente la primera directiva para el solicitante porque es más estricta. Al solicitante no se le ofrece la opción de seleccionar la segunda directiva.

* Cuando se aplican varias directivas, la directiva que se selecciona automáticamente o las directivas que se muestran al solicitante se basan en la siguiente lógica de prioridad:

    | Prioridad de directiva | Ámbito |
    | --- | --- |
    | P1 | Usuarios y grupos específicos del directorio u organizaciones conectadas específicas |
    | P2 | Todos los miembros del directorio (excepto los invitados) |
    | P3 | Todos los usuarios del directorio (incluidos los invitados) u organizaciones conectadas específicas |
    | P4 | Todas las organizaciones conectadas o todos los usuarios (todas las organizaciones conectadas y todos los usuarios externos nuevos) |
    
    Si alguna directiva está en una categoría de prioridad más alta, se omiten las categorías de prioridad inferior. Para ver un ejemplo de cómo se muestran al solicitante varias directivas con la misma prioridad, consulte [Selección de una directiva](entitlement-management-request-access.md#select-a-policy).

## <a name="next-steps"></a>Pasos siguientes

- [Administración del acceso para usuarios externos](entitlement-management-external-users.md)
- [Visualización de los informes sobre el acceso de los usuarios en la administración de derechos](entitlement-management-reports.md)