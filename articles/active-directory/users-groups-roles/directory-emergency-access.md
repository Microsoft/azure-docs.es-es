---
title: 'Administración de cuentas de administrador de acceso de emergencia: Azure AD | Microsoft Docs'
description: En este artículo se describe cómo usar las cuentas de acceso de emergencia para ayudar a evitar que se bloquee inadvertidamente su acceso a la organización de Azure Active Directory (Azure AD).
services: active-directory
author: markwahl-msft
manager: daveba
ms.author: curtand
ms.date: 11/08/2019
ms.topic: conceptual
ms.service: active-directory
ms.subservice: users-groups-roles
ms.workload: identity
ms.custom: it-pro
ms.reviewer: markwahl-msft
ms.collection: M365-identity-device-management
ms.openlocfilehash: e9d4f293f3835e26def97aa2f52dd0c42d9137c7
ms.sourcegitcommit: e71da24cc108efc2c194007f976f74dd596ab013
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/29/2020
ms.locfileid: "87421729"
---
# <a name="manage-emergency-access-accounts-in-azure-ad"></a>Administración de cuentas de acceso de emergencia en Azure AD

Es importante evitar que se bloquee accidentalmente su acceso a la organización de Azure Active Directory (Azure AD) porque no podrá iniciar sesión ni activar otra cuenta de usuario como administrador. El impacto de una falta involuntaria de acceso administrativo se puede mitigar mediante la creación de dos o más *cuentas de acceso de emergencia* en la organización.

Las cuentas de acceso de emergencia tienen privilegios elevados y no se asignan a usuarios específicos. Las cuentas de acceso de emergencia se limitan a situaciones "excepcionales" o de emergencia en las que no se pueden usar las cuentas administrativas normales. Se recomienda que mantenga como objetivo la restricción del uso de las cuentas de emergencia solo para aquellos momentos en los que sean absolutamente necesarias.

En este artículo se proporcionan instrucciones para administrar las cuentas de acceso de emergencia en Azure AD.

## <a name="why-use-an-emergency-access-account"></a>¿Por qué usaría una cuenta de acceso de emergencia?

Puede que una organización necesite usar una cuenta de acceso de emergencia en las siguientes situaciones:

- Las cuentas de usuario están federadas y la federación no está disponible en este momento debido a una interrupción de la red de telefonía móvil o una interrupción del proveedor de identidades. Por ejemplo, si se ha interrumpido el host del proveedor de identidades de su entorno, puede que los usuarios no puedan iniciar sesión cuando Azure AD les redirija a su proveedor de identidades.
- Los administradores están registrados mediante Azure Multi-Factor Authentication y todos sus dispositivos individuales o los servicios están deshabilitados. Puede que los usuarios no puedan completar la autenticación multifactor para activar un rol. Por ejemplo, una interrupción de la red de telefonía móvil les impide responder a llamadas telefónicas o recibir mensajes de texto, los únicos dos mecanismos de autenticación que registraron para sus dispositivos.
- La persona con el acceso de Administrador global más reciente ha dejado la organización. Azure AD impide que se pueda eliminar la última cuenta de Administrador global, pero no impide que esta se pueda eliminar o deshabilitar de forma local. Es posible que alguna de estas situaciones impida a la organización recuperar la cuenta.
- Circunstancias imprevistas, como un desastre natural, durante las cuales puede que las redes de telefonía móvil u otras redes no estén disponibles. 

## <a name="create-emergency-access-accounts"></a>Creación de cuentas de acceso de emergencia

Cree dos o más cuentas de acceso de emergencia. Estas deben ser cuentas que estén solo en la nube, que usen el dominio \*.onmicrosoft.com y que no estén federadas ni sincronizadas desde un entorno local.

Al configurarlas, deben cumplirse los siguientes requisitos:

- Las cuentas de acceso de emergencia no se deberían asociar a ningún usuario individual de la organización. Asegúrese de que las cuentas no están conectadas a ningún teléfono móvil suministrado por un empleado, ni a tokens de hardware que viajen con empleados individuales ni con otras credenciales específicas de un empleado. Esta precaución debe incluir también los casos en los que un empleado individual pueda no estar disponible cuando se necesiten las credenciales. Es importante garantizar que todos los dispositivos registrados se almacenen en una ubicación conocida y segura que tenga varias formas de comunicarse con Azure AD.
- El mecanismo de autenticación usado para una cuenta de acceso de emergencia debe ser distinto del que se utiliza para otras cuentas administrativas, incluidas otras cuentas de acceso de emergencia.  Por ejemplo, si el inicio de sesión de administrador normal es a través de una instancia de MFA local, Azure MFA sería un mecanismo diferente.  Sin embargo, si Azure MFA es la parte principal de la autenticación para las cuentas administrativas, considere un enfoque diferente para estas, como el uso de acceso condicional con un proveedor MFA de terceros a través de [Controles personalizado](https://docs.microsoft.com/azure/active-directory/conditional-access/controls).
- El dispositivo o la credencial no deben expirar ni estar en el ámbito de una limpieza automatizada debido a la falta de uso.  
- Debe convertir la asignación del rol de Administrador global en permanente para las cuentas de acceso de emergencia. 

### <a name="exclude-at-least-one-account-from-phone-based-multi-factor-authentication"></a>Exclusión de al menos una cuenta de autenticación multifactor basada en teléfono

Para reducir el riesgo de ataques como resultado de una contraseña que se ha puesto en peligro, Azure AD recomienda que requiera autenticación multifactor a todos los usuarios individuales. Este grupo incluye a los administradores y a todos aquellos (por ejemplo, los agentes financieros) cuyas cuentas, si se ponen en peligro, tendrían un impacto significativo.

Sin embargo, al menos una de sus cuentas de acceso de emergencia no debe tener el mismo mecanismo de autenticación multifactor que las demás cuentas que no son de emergencia. Esto incluye las soluciones de autenticación multifactor de terceros. Si tiene una directiva de acceso condicional para requerir [autenticación multifactor para cada administrador](../authentication/howto-mfa-userstates.md) de Azure AD y de otras aplicaciones conectadas de software como servicio (SaaS), debería excluir las cuentas de acceso de emergencia de este requisito y configurar un mecanismo diferente. Además, debe asegurarse de que las cuentas no tienen una directiva de autenticación multifactor por usuario.

### <a name="exclude-at-least-one-account-from-conditional-access-policies"></a>Exclusión de al menos una cuenta de las directivas de acceso condicional

Durante una emergencia, no quiere que una directiva pueda bloquear el acceso para corregir un problema. Por eso, debe excluirse por lo menos una cuenta de acceso de emergencia de todas las directivas de acceso condicional.

## <a name="federation-guidance"></a>Guía de federación

Una opción adicional para las organizaciones que usan AD Domain Services, ADFS o proveedores de identidades similares para federar con Azure AD, es configurar una cuenta de acceso de emergencia cuya notificación de MFA podría proporcionarla dicho proveedor de identidades.  Por ejemplo, la cuenta de acceso de emergencia podría estar respaldada por un par de certificado y clave como el almacenado en una tarjeta inteligente.  Cuando ese usuario se autentica en Active Directory, ADFS pueden proporcionar una notificación a Azure AD para indicar que el usuario ha cumplido con los requisitos de MFA.  Incluso con este enfoque, las organizaciones todavía deben tener cuentas de acceso de emergencia basadas en la nube en caso de que no se pueda establecer la federación. 

## <a name="store-account-credentials-safely"></a>Almacenamiento seguro de las credenciales de cuenta

Las organizaciones se deben asegurar de que las credenciales de estas cuentas de acceso de emergencia estén protegidas y solo las conozcan los usuarios que están autorizados para usarlas. Algunos clientes usan una tarjeta inteligente y otros usan contraseñas. La contraseña de una cuenta de acceso de emergencia se encuentra normalmente dividida en dos o tres partes, escrita en diferentes fragmentos de papel que se almacenan en cajas de seguridad a prueba de incendios que están en ubicaciones separadas seguras.

Si usa contraseñas, asegúrese de que las cuentas tengan contraseñas seguras y que no expiren. Idealmente, las contraseñas deben tener al menos 16 caracteres y generarse aleatoriamente.

## <a name="monitor-sign-in-and-audit-logs"></a>Supervisión de registros de inicio de sesión y de auditoría

Las organizaciones deben supervisar la actividad de registro de auditoría e inicio de sesión de las cuentas de emergencia y desencadenar el envío de notificaciones a otros administradores. Al supervisar la actividad en las cuentas de emergencia, puede comprobar que estas cuentas solo se usen para pruebas o emergencias reales. Puede usar Azure Log Analytics para supervisar los registros de inicio de sesión y desencadenar alertas por SMS y correo electrónico a los administradores cuando las cuentas de emergencia inicien sesión.

### <a name="prerequisites"></a>Prerrequisitos

1. [Envíe registros de inicio de sesión de Azure AD](https://docs.microsoft.com/azure/active-directory/reports-monitoring/howto-integrate-activity-logs-with-log-analytics) a Azure Monitor.

### <a name="obtain-object-ids-of-the-break-glass-accounts"></a>Obtención de los Id. de objeto de las cuentas de emergencia

1. Inicie sesión en [Azure Portal](https://portal.azure.com) con una cuenta asignada al rol de Usuario administrador.
1. Seleccione **Azure Active Directory** > **Usuarios**.
1. Busque la cuenta de emergencia y seleccione el nombre del usuario.
1. Copie y guarde el atributo de Id. de objeto para usarlo más adelante.
1. Repita los pasos anteriores para la segunda cuenta de emergencia.

### <a name="create-an-alert-rule"></a>Crear una regla de alerta

1. Inicie sesión en [Azure Portal](https://portal.azure.com) con una cuenta asignada al rol de Colaborador de supervisión en Azure Monitor.
1. Seleccione **Todos los servicios**", escriba "log analytics" en la búsqueda y después seleccione **Áreas de trabajo de Log Analytics**.
1. Seleccione un área de trabajo.
1. En el área de trabajo, seleccione **Alertas** > **Nueva regla de alertas**.
    1. En **Recurso**, compruebe que la suscripción es la que quiere asociar con la regla de alertas.
    1. En **condición**, seleccione **Agregar**.
    1. Seleccione **Custom log search** (Búsqueda de registros personalizada) en **Nombre de señal**.
    1. En **Consulta de búsqueda**, escriba la siguiente consulta e inserte los Id. de objeto de las dos cuentas de emergencia.
        > [!NOTE]
        > Para cada cuenta de emergencia adicional que quiera incluir, agregue otro "or UserId == "ObjectGuid"" a la consulta.

        ![Agregar los Id. de objeto de las cuentas de emergencia a una regla de alertas](./media/directory-emergency-access/query-image1.png)

    1. En **Lógica de alerta**, escriba lo siguiente:

        - Basado en: Número de resultados
        - Operador: Mayor que
        - Valor del umbral: 0

    1. En **Se evaluó basándose en**, seleccione el **Periodo (en minutos)** durante el que quiere que se ejecute la consulta, así como la **Frecuencia (en minutos)** con la que quiere que se ejecute la consulta. La frecuencia debe ser menor o igual que el periodo.

        ![lógica de alerta](./media/directory-emergency-access/alert-image2.png)

    1. Seleccione **Listo**. Ahora puede ver el costo mensual estimado de esta alerta.
1. Seleccione el grupo de acciones con los usuarios a los que notificará la alerta. Si quiere crear uno, consulte [Creación de un grupo de acciones](#create-an-action-group).
1. Para personalizar la notificación por correo electrónico que se envía a los miembros del grupo de acciones, seleccione Acciones en **Personalizar las acciones**.
1. En **Detalles de alertas**, especifique el nombre de la regla de alerta y agregue una descripción opcional.
1. Establezca el **Nivel de gravedad** del evento. Se recomienda que lo establezca en **Crítico (gravedad 0)** .
1. Under **Habilitar regla tras la creación**, deje el valor como **sí**.
1. Para desactivar las alertas durante un tiempo, active la casilla **Suprimir alertas**, escriba la duración de la pausa antes de reanudar las alertas y seleccione **Guardar**.
1. Haga clic en **Crear regla de alertas**.

### <a name="create-an-action-group"></a>Creación de un grupo de acciones

1. Seleccione **Create an action group** (Crear un grupo de acciones).

    ![crear un grupo de acciones para las acciones de notificación](./media/directory-emergency-access/action-group-image3.png)

1. Escriba el nombre del grupo de acciones y un nombre corto.
1. Compruebe la suscripción y el grupo de recursos.
1. En Tipo de acción, seleccione **Correo electrónico/SMS/Push/Voz**.
1. Escriba un nombre de acción como **Notificar al administrador global**.
1. Seleccione el **Tipo de acción** como **Correo electrónico/SMS/Push/Voz**.
1. Seleccione **Editar detalles** para seleccionar los métodos de notificación que quiere configurar y escriba la información de contacto necesaria. Después, seleccione **Aceptar** para guardar los detalles.
1. Agregue las acciones adicionales que quiera desencadenar.
1. Seleccione **Aceptar**.

## <a name="validate-accounts-regularly"></a>Validación de las cuentas de forma regular

Cuando entrena a los miembros del personal en el uso de cuentas de acceso de emergencia y validar dichas cuentas, como mínimo debe seguir estos pasos a intervalos regulares:

- Asegúrese de que el personal de supervisión de seguridad sea consciente de que la actividad de comprobación de las cuentas es continua.
- Asegúrese de que el proceso para escenarios de máxima emergencia en los que se usarán estas cuentas está documentado y es actual.
- Asegúrese de que los administradores y los responsables de seguridad que podrían tener que realizar estos pasos durante una emergencia están entrenados en el proceso.
- Actualice las credenciales, en particular las contraseñas, de las cuentas de acceso de emergencia y, después, valide que las cuentas de acceso de emergencia puedan iniciar sesión y realizar tareas administrativas.
- Asegúrese de que los usuarios no han registrado la autenticación multifactor ni el restablecimiento de contraseña de autoservicio (SSPR) en ningún dispositivo de usuario individual o con información personal. 
- Si las cuentas se han registrado para la autenticación multifactor en un dispositivo, para su uso durante el inicio de sesión o la activación de rol, asegúrese de que este dispositivo sea accesible para todos los administradores que podrían tener que usarlo en caso de emergencia. Compruebe también que el dispositivo pueda comunicarse mediante al menos dos rutas de acceso de red que no compartan un modo de avería común. Por ejemplo, el dispositivo puede comunicarse con Internet a través de la red inalámbrica de la oficina y a través de una red de un proveedor de telefonía móvil.

Estos pasos deben realizarse a intervalos regulares y para cambios de claves:

- Al menos cada 90 días
- Cuando se ha producido un cambio reciente en el personal de TI, como un cambio de trabajo, una salida o un nuevo empleado.
- Cuando se han cambiado las suscripciones de Azure AD de la organización.

## <a name="next-steps"></a>Pasos siguientes

- [Protección del acceso con privilegios para las implementaciones híbridas y en la nube en Azure AD](directory-admin-roles-secure.md)
- [Adición de usuarios con Azure AD](../fundamentals/add-users-azure-active-directory.md) y [Asignación de un nuevo usuario al rol de Administrador global](../fundamentals/active-directory-users-assign-role-azure-portal.md).
- [Suscripción a Azure AD Premium](../fundamentals/active-directory-get-started-premium.md), si no lo ha hecho anteriormente.
- [Exigencia de verificación en dos pasos para un usuario](../authentication/howto-mfa-userstates.md).
- [Configuración de otras protecciones adicionales para los administradores globales de Microsoft 365](https://docs.microsoft.com/office365/enterprise/protect-your-global-administrator-accounts), si lo usa.
- [Inicio de una revisión de acceso de los administradores globales](../privileged-identity-management/pim-how-to-start-security-review.md) y [cambio de los administradores globales existentes a roles de administrador más específicos](directory-assign-admin-roles.md).
