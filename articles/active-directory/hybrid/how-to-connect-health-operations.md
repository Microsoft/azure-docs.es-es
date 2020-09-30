---
title: Operaciones de Azure Active Directory Connect Health
description: Este artículo describe las operaciones adicionales que pueden realizarse una vez que haya implementado Azure AD Connect Health.
services: active-directory
documentationcenter: ''
author: zhiweiwangmsft
manager: daveba
ms.assetid: 86cc3840-60fb-43f9-8b2a-8598a9df5c94
ms.service: active-directory
ms.subservice: hybrid
ms.workload: identity
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: how-to
ms.date: 07/18/2017
ms.author: billmath
ms.collection: M365-identity-device-management
ms.openlocfilehash: 4a83d1a4bdc2496f874a963b8858192273260cc8
ms.sourcegitcommit: c94a177b11a850ab30f406edb233de6923ca742a
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/01/2020
ms.locfileid: "89276276"
---
# <a name="azure-active-directory-connect-health-operations"></a>Operaciones de Azure Active Directory Connect Health
Este tema describe las distintas operaciones que se pueden realizar mediante Azure Active Directory (Azure AD) Connect Health.

## <a name="enable-email-notifications"></a>Habilitación de notificaciones de correo electrónico
Puede configurar el servicio de Azure AD Connect Health para enviar notificaciones por correo electrónico cuando las alertas indiquen que el estado de la infraestructura de identidad no es correcto. Esto se produce cuando se genera una alerta y cuando se resuelve.

![Captura de pantalla de la configuración de notificaciones de correo electrónico de Azure AD Connect Health](./media/how-to-connect-health-operations/email_noti_discover.png)

> [!NOTE]
> Las notificaciones de correo electrónico están habilitadas de forma predeterminada.
>

### <a name="to-enable-azure-ad-connect-health-email-notifications"></a>Para habilitar las notificaciones de correo electrónico de Azure AD Connect Health
1. Abra la hoja **Alertas** del servicio para el que desea recibir una notificación de correo electrónico.
2. Haga clic en el botón **Configuración de notificaciones** de la barra de acciones.
3. En el modificador de la opción Notificación de correo electrónico, seleccione **ON** (Activado).
4. Seleccione la casilla si desea que todos los administradores globales reciban notificaciones de correo electrónico.
5. Si desea recibir notificaciones de correo electrónico en otras direcciones de correo electrónico, puede especificarlas en el cuadro **Destinatario de correo electrónico adicional**. Para quitar una dirección de correo electrónico de esta lista, haga clic con el botón derecho en la entrada y seleccione **Eliminar**.
6. Para finalizar los cambios, haga clic en **Guardar**. Los cambios surtirán efecto después de guardar.

>[!NOTE] 
> Cuando hay problemas para procesar solicitudes de sincronización en nuestro servicio de back-end, este servicio envía un correo electrónico de notificación con los detalles del error a las direcciones de correo electrónico de los contactos administrativos de su inquilino. Hemos recibido comentarios de clientes en los que afirman que, en determinados casos, el volumen de estos mensajes es excesivo, por lo que estamos cambiando la forma en que se envían. 
>
> En lugar de enviar un mensaje para cada error de sincronización cada vez que se produce, enviaremos un resumen diario de todos los errores que el servicio back-end ha devuelto. De este modo, los clientes pueden procesar estos errores de una manera más eficaz y reduce el número de mensajes de error duplicados.
>
> Tenemos previsto que este cambio se implemente el 15 de enero de 2020.

## <a name="delete-a-server-or-service-instance"></a>Eliminación de una instancia de servidor o servicio

>[!NOTE] 
> Para los pasos de eliminación, es necesaria una licencia de Azure AD premium.

En algunos casos, es posible que desee dejar de supervisar un servidor. Siga estas instrucciones para quitar un servidor del Servicio de Azure AD Connect Health.

Cuando elimine un servidor, tenga en cuenta lo siguiente:

* Esta acción detiene cualquier recopilación de datos de ese servidor. Este servidor se quitará del servicio de supervisión. Después de esta acción, no podrá ver nuevas alertas ni datos de análisis de uso o supervisión de este servidor.
* Esta acción no desinstala el agente de mantenimiento del servidor. Si no ha desinstalado el agente de mantenimiento antes de realizar este paso, es posible que vea eventos de error en el servidor relacionados con el agente de mantenimiento.
* Esta acción no elimina los datos que ya se recopilaron desde este servidor. Esos datos se eliminan de acuerdo con la directiva de retención de datos de Azure.
* Tras realizar esta acción, si desea empezar la supervisión del mismo servidor de nuevo, desinstale el agente de estado y vuelva a instalarlo en este servidor.

### <a name="delete-a-server-from-the-azure-ad-connect-health-service"></a>Eliminación de un servidor del servicio de Azure AD Connect Health

>[!NOTE] 
> Para los pasos de eliminación, es necesaria una licencia de Azure AD premium.

Azure AD Connect Health para Servicios de federación de Active Directory (AD FS) y Azure AD Connect (Sync):

1. Seleccione el nombre del servidor que se va a quitar para abrir la hoja **Servidor** en la hoja **Lista de servidores**.
2. En la hoja **Servidor**, en la barra de acciones, haga clic en **Eliminar**.
![Captura de pantalla de eliminación de servidor de Azure AD Connect Health](./media/how-to-connect-health-operations/DeleteServer2.png)
3. Confirme el nombre del servidor; para ello, escríbalo en el cuadro de confirmación.
4. Haga clic en **Eliminar**.

Azure AD Connect Health para Azure Active Directory Domain Services:

1. Abra el panel de **controladores de dominio**.
2. Seleccione el controlador de dominio que va a quitar.
3. En la barra de acciones, haga clic en **Eliminar selección**.
4. Confirme la acción para eliminar el servidor.
5. Haga clic en **Eliminar**.

### <a name="delete-a-service-instance-from-azure-ad-connect-health-service"></a>Eliminación de una instancia de servicio del Servicio de Azure AD Connect Health
En algunos casos, es posible que desee quitar una instancia de servicio. Siga estas instrucciones para quitar una instancia de servicio del Servicio de Azure AD Connect Health.

Cuando elimine una instancia de servicio, tenga en cuenta lo siguiente:

* Esta acción quitará la instancia de servicio actual del servicio de supervisión.
* Esta acción no desinstalará ni quitará el agente de mantenimiento de ninguno de los servidores que se supervisaron como parte de esta instancia de servicio. Si no ha desinstalado el agente de mantenimiento antes de realizar este paso, es posible que vea eventos de error en los servidores relacionados con el agente de mantenimiento.
* Todos los datos de esta instancia de servicio se eliminarán según lo estipulado en la directiva de retención de datos de Azure.
* Tras realizar esta acción, si desea empezar a supervisar el servicio, desinstale el agente de estado y vuelva a instalarlo en todos los servidores. Tras realizar esta acción, si desea empezar la supervisión del mismo servidor de nuevo, desinstale el agente de estado y regístrelo en ese servidor.

#### <a name="to-delete-a-service-instance-from-the-azure-ad-connect-health-service"></a>Eliminación de una instancia de servicio del Servicio de Azure AD Connect Health
1. Seleccione el identificador del servicio (nombre de la granja) que desea quitar para abrir la hoja **Servicio** en la hoja **Lista de Servicios**. 
2. En la hoja **Servicio**, en la barra de acciones, haga clic en **Eliminar**. 
![Captura de pantalla de eliminación de servicio de Azure AD Connect Health](./media/how-to-connect-health-operations/DeleteServer.png)
3. Confirme el nombre del servicio. Para ello, escríbalo en el cuadro de confirmación (por ejemplo: sts.contoso.com).
4. Haga clic en **Eliminar**.
   <br><br>

[//]: # (Inicio de la sección de RBAC)
## <a name="manage-access-with-azure-rbac"></a>Administración del acceso con Azure RBAC
El [control de acceso basado en rol (RBAC de Azure)](../../role-based-access-control/role-assignments-portal.md) para Azure AD Connect Health proporciona acceso a usuarios o grupos distintos a los administradores globales. Azure RBAC asigna roles a los usuarios y grupos previstos, y proporciona un mecanismo para limitar los administradores globales dentro del directorio.

### <a name="roles"></a>Roles
Azure AD Connect Health admite los siguientes roles integrados:

| Role | Permisos |
| --- | --- |
| Propietario |Los propietarios pueden *administrar el acceso* (por ejemplo, asignar roles a un usuario o grupo), *ver toda la información* (por ejemplo, ver las alertas) desde el portal y *cambiar la configuración* (por ejemplo, notificaciones de correo electrónico) dentro de Azure AD Connect Health. <br>De forma predeterminada, a los administradores globales de Azure AD se les asigna este rol y esto no se puede cambiar. |
| Colaborador |Los colaboradores pueden *ver toda la información* (por ejemplo, ver las alertas) desde el portal y *cambiar la configuración* (por ejemplo, notificaciones de correo electrónico) dentro de Azure AD Connect Health. |
| Lector |Los lectores pueden *ver toda la información* (por ejemplo, ver las alertas) desde el portal dentro de Azure AD Connect Health. |

Todos los demás roles (como "Administradores de acceso de usuario" o "Usuarios del laboratorio DevTest"), incluso si están disponibles en la experiencia del portal, no afectan al acceso dentro de Azure AD Connect Health.

### <a name="access-scope"></a>Ámbito de acceso
Azure AD Connect Health admite la administración de acceso a dos niveles:

* **Todas las instancias de servicio**: ruta de acceso recomendada en la mayoría de los casos. Controla el acceso a todas las instancias de servicio (por ejemplo, a una granja de servidores de AD FS) en todos los tipos de rol que se estén supervisando mediante Azure AD Connect Health.
* **Instancia de servicio**: en algunos casos, puede que necesite separar el acceso según los tipos de rol o por una instancia de servicio. En este caso, puede administrar el acceso en el nivel de instancia de servicio.  

El permiso se concede si un usuario final tiene acceso al nivel de directorio o de instancia de servicio.

### <a name="allow-users-or-groups-access-to-azure-ad-connect-health"></a>Cómo permitir el acceso a los usuarios o grupos a Azure AD Connect Health
Se muestra cómo hacerlo en los pasos siguientes.
#### <a name="step-1-select-the-appropriate-access-scope"></a>Paso 1: Seleccionar el ámbito de acceso adecuado
Para permitir a un usuario acceder al nivel de *todas las instancias de servicio* dentro de Azure AD Connect Health, abra la hoja principal en Azure AD Connect Health.<br>

#### <a name="step-2-add-users-and-groups-and-assign-roles"></a>Paso 2: Agregar usuarios, grupos y asignar roles
1. En la sección **Configurar**, haga clic en **Usuarios**.<br>
   ![Captura de pantalla de la barra lateral de recursos de Azure AD Connect Health](./media/how-to-connect-health-operations/startRBAC.png)
2. Seleccione **Agregar**.
3. En el panel **Seleccionar rol**, seleccione un rol (por ejemplo, **Propietario**).<br>
   ![Captura de pantalla del menú de configuración de Azure AD Connect Health y Azure RBAC](./media/how-to-connect-health-operations/RBAC_add.png)
4. Escriba el nombre o identificador del usuario o grupo de destino. Puede seleccionar uno o más usuarios o grupos al mismo tiempo. Haga clic en **Seleccionar**.
   ![Captura de pantalla de la lista de roles de Azure AD Connect Health y Azure RBAC](./media/how-to-connect-health-operations/RBAC_select_users.png)
5. Seleccione **Aceptar**.<br>
6. Después de finalizar la asignación de roles, los usuarios y grupos aparecerán en la lista.<br>
   ![Captura de pantalla de Azure AD Connect Health y Azure RBAC, y nuevos usuarios resaltados](./media/how-to-connect-health-operations/RBAC_user_list.png)

Ahora los usuarios y grupos de la lista tienen acceso, según sus roles asignados.

> [!NOTE]
> * Los administradores globales siempre tienen acceso total a todas las operaciones, pero las cuentas de administrador global no figurarán en la lista anterior.
> * No se admite la característica "Invitar a usuarios" en Azure AD Connect Health.
>
>

#### <a name="step-3-share-the-blade-location-with-users-or-groups"></a>Paso 3: Compartir la ubicación de la hoja con usuarios o grupos
1. Después de asignar permisos, un usuario puede acceder a Azure AD Connect Health yendo [aquí](https://aka.ms/aadconnecthealth).
2. Una vez en la hoja, el usuario puede anclar dicha hoja o diferentes partes de esta al panel. Solo tiene que hacer clic en el icono **Anclar al panel**.<br>
   ![Captura de pantalla de la hoja de anclaje de Azure RBAC y Azure AD Connect Health, con el icono de anclaje resaltado](./media/how-to-connect-health-operations/RBAC_pin_blade.png)

> [!NOTE]
> Un usuario con el rol de "Lector" asignado no puede obtener la extensión de Azure AD Connect Health de Azure Marketplace. El usuario no puede realizar la operación "crear" necesaria para realizar esta acción. Aún así, este usuario todavía puede obtener la hoja visitando el vínculo anterior. Para usos posteriores, el usuario puede anclar la hoja en el panel.
>
>

### <a name="remove-users-or-groups"></a>Eliminación de usuarios o grupos
Puede quitar un usuario o un grupo agregado a Azure RBAC y Azure AD Connect Health. Simplemente haga clic con el botón derecho en el usuario o grupo y seleccione **Quitar**.<br>
![Captura de pantalla de Azure AD Connect Health y Azure RBAC, con la opción Quitar resaltada](./media/how-to-connect-health-operations/RBAC_remove.png)

[//]: # (Fin de la sección de RBAC)

## <a name="next-steps"></a>Pasos siguientes
* [Azure AD Connect Health](./whatis-azure-ad-connect.md)
* [Instalación del Agente de Azure AD Connect Health](how-to-connect-health-agent-install.md)
* [Uso de Azure AD Connect Health con AD FS](how-to-connect-health-adfs.md)
* [Uso de Azure AD Connect Health para sincronización](how-to-connect-health-sync.md)
* [Uso de Azure AD Connect Health con AD DS](how-to-connect-health-adds.md)
* [Preguntas más frecuentes de Azure AD Connect Health](reference-connect-health-faq.md)
* [Historial de versiones de Azure AD Connect Health](reference-connect-health-version-history.md)