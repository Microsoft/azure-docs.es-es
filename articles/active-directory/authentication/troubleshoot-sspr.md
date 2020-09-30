---
title: 'Solución de problemas del autoservicio de restablecimiento de contraseña: Azure Active Directory'
description: Aprenda a solucionar los problemas comunes del autoservicio de restablecimiento de contraseña en Azure Active Directory.
services: active-directory
ms.service: active-directory
ms.subservice: authentication
ms.topic: troubleshooting
ms.date: 08/26/2020
ms.author: iainfou
author: iainfoulds
manager: daveba
ms.reviewer: rhicock
ms.collection: M365-identity-device-management
ms.openlocfilehash: 4a02d46688fa49401684f836a7c289906affb413
ms.sourcegitcommit: 3fc3457b5a6d5773323237f6a06ccfb6955bfb2d
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/11/2020
ms.locfileid: "90030028"
---
# <a name="troubleshoot-self-service-password-reset-in-azure-active-directory"></a>Solución de problemas del autoservicio de restablecimiento de contraseña en Azure Active Directory

El autoservicio de restablecimiento de contraseña de Azure Active Directory (Azure AD) permite a los usuarios restablecer sus contraseñas en la nube.

Si tiene problemas con SSPR, puede que le sean de utilidad los pasos de resolución y los errores comunes que se describen a continuación. Si no encuentra la respuesta a un problema, [nuestros equipos de soporte técnico](#contact-microsoft-support) están siempre disponibles para ayudarle.

## <a name="sspr-configuration-in-the-azure-portal"></a>Configuración de SSPR en Azure Portal

Si tiene problemas para ver o configurar las opciones de SSPR en Azure Portal, revise los siguientes pasos de solución de problemas:

### <a name="i-dont-see-the-password-reset-section-under-azure-ad-in-the-azure-portal"></a>No veo la sección **Restablecimiento de contraseña** en Azure AD en Azure Portal.

No verá la opción **Restablecimiento de contraseña** si no tiene una licencia de Azure AD asignada al administrador que realiza la operación.

Para asignar una licencia a la cuenta de administrador en cuestión, siga los pasos para [asignar, comprobar y resolver problemas con las licencias](../users-groups-roles/licensing-groups-assign.md#step-1-assign-the-required-licenses).

### <a name="i-dont-see-a-particular-configuration-option"></a>No veo una opción de configuración determinada.

Muchos elementos de la interfaz de usuario están ocultos hasta que se necesitan. Asegúrese de que la opción está habilitada antes de buscar las opciones de configuración específicas.

### <a name="i-dont-see-the-on-premises-integration-tab"></a>No veo la pestaña **Integración local**.

La escritura diferida de contraseñas local solo es visible si ha descargado Azure AD Connect y ha configurado la característica.

Para más información, consulte [Introducción a Azure AD Connect](../hybrid/how-to-connect-install-express.md).

## <a name="sspr-reporting"></a>Informes de SSPR

Si tiene problemas con los informes de SSPR en Azure Portal, revise los siguientes pasos de resolución:

### <a name="i-dont-see-any-password-management-activity-types-in-the-self-service-password-management-audit-event-category"></a>No veo ningún tipo de actividad de administración de contraseñas en la categoría de evento de auditoría **Self-Service Password Management** (Administración de autoservicio de contraseñas).

Esto puede suceder si no tiene una licencia de Azure AD asignada al administrador que realiza la operación.

Para asignar una licencia a la cuenta de administrador en cuestión, siga los pasos para [asignar, comprobar y resolver problemas con las licencias](../users-groups-roles/licensing-groups-assign.md#step-1-assign-the-required-licenses).

### <a name="user-registrations-show-multiple-times"></a>Los registros de usuario se muestran varias veces.

Actualmente, cuando un usuario se registra, anotamos cada dato individual registrado como un evento independiente.

Si desea agregar estos datos y tener mayor flexibilidad en el modo de verlos, puede descargar el informe y abrirlos como una tabla dinámica de Excel.

## <a name="sspr-registration-portal"></a>Portal de registro de SSPR

Si los usuarios tienen problemas para registrarse en SSPR, revise los siguientes pasos de resolución:

### <a name="the-directory-isnt-enabled-for-password-reset-the-user-may-see-an-error-that-reports-your-administrator-has-not-enabled-you-to-use-this-feature"></a>El directorio no está habilitado para el restablecimiento de contraseña. El usuario puede ver un error que indica que el administrador no le permite usar esta característica.

Puede habilitar SSPR para todos los usuarios, para ninguno o para grupos de usuarios seleccionados. Actualmente solo se puede habilitar un grupo de Azure AD para SSPR mediante Azure Portal. Como parte de una implementación más amplia del autoservicio de restablecimiento de contraseña, se admiten los grupos anidados. Asegúrese de que los usuarios de los grupos que elija tienen asignadas las licencias correspondientes.

En Azure Portal, cambie la configuración de **Se habilitó el restablecimiento de contraseña del autoservicio** a *Seleccionados* o *Todos* y, luego, elija **Guardar**.

### <a name="the-user-doesnt-have-an-azure-ad-license-assigned-the-user-may-see-an-error-that-reports-your-administrator-has-not-enabled-you-to-use-this-feature"></a>El usuario no tiene asignada una licencia de Azure AD. El usuario puede ver un error que indica que el administrador no le permite usar esta característica.

Actualmente solo se puede habilitar un grupo de Azure AD para SSPR mediante Azure Portal. Como parte de una implementación más amplia del autoservicio de restablecimiento de contraseña, se admiten los grupos anidados. Asegúrese de que los usuarios de los grupos que elija tienen asignadas las licencias correspondientes. Si es necesario, revise el paso de solución de problemas anterior para habilitar SSPR.

Revise también los pasos de solución de problemas para asegurarse de que el administrador que ejecuta las opciones de configuración tiene una licencia asignada. Para asignar una licencia a la cuenta de administrador en cuestión, siga los pasos para [asignar, comprobar y resolver problemas con las licencias](../users-groups-roles/licensing-groups-assign.md#step-1-assign-the-required-licenses).

### <a name="theres-an-error-processing-the-request"></a>Se produce un error al procesar la solicitud.

Aunque los errores genéricos de registro de SSPR pueden tener varias causas, en general este error se debe a una interrupción del servicio o a un problema de configuración. Si sigue viendo este error genérico al reintentar el proceso de registro de SSPR, [póngase en contacto con el servicio de soporte técnico de Microsoft](#contact-microsoft-support) para obtener ayuda adicional.

## <a name="sspr-usage"></a>Uso de SSPR

Si usted o sus usuarios tienen problemas con el uso de SSPR, revise los siguientes escenarios y pasos de solución de problemas:

| Error | Solución |
| --- | --- |
| El directorio no está habilitado para el restablecimiento de contraseña. | En Azure Portal, cambie la configuración de **Se habilitó el restablecimiento de contraseña del autoservicio** a *Seleccionados* o *Todos* y, luego, elija **Guardar**. |
| El usuario no tiene asignada una licencia de Azure AD. | Esto puede ocurrir si no ha asignado una licencia de Azure AD al usuario deseado. Para asignar una licencia a la cuenta de administrador en cuestión, siga los pasos para [asignar, comprobar y resolver problemas con las licencias](../users-groups-roles/licensing-groups-assign.md#step-1-assign-the-required-licenses). |
| El directorio está habilitado para restablecer la contraseña, pero falta la información de autenticación del usuario o es incorrecta. | Asegúrese de que el usuario haya creado correctamente los datos de contacto en el directorio. Para obtener más información, consulte los [Datos usados en el autoservicio de restablecimiento de contraseña de Azure AD](howto-sspr-authenticationdata.md). |
| El directorio está habilitado para restablecer la contraseña, pero el usuario tiene solo una parte de los datos de contacto en el archivo cuando la directiva está configurada para requerir dos métodos de verificación. | Asegúrese de que el usuario tenga al menos dos métodos de contacto configurados correctamente. Un ejemplo es tener tanto un número de teléfono móvil *como* un número de teléfono de trabajo. |
| El directorio está habilitado para restablecer la contraseña y el usuario está configurado correctamente, pero no es posible ponerse en contacto con él. | Puede deberse a un error de servicio temporal o a que hay datos de contacto configurados incorrectamente que no hemos podido detectar como es debido. <br> <br> Si el usuario espera 10 segundos, aparece un vínculo a "Reintentar" y "Póngase en contacto con el administrador". Si el usuario selecciona "Reintentar", se vuelve a intentar la llamada. Si el usuario selecciona "Póngase en contacto con el administrador", se envía a los administradores un correo electrónico de formulario para solicitarles que realicen el restablecimiento de la contraseña para esa cuenta de usuario. |
| El usuario nunca recibe la llamada de teléfono o el SMS de restablecimiento de la contraseña. | Esto puede deberse a que se haya incluido un número de teléfono incorrecto en el directorio. Asegúrese de que el número de teléfono tenga el formato "+1 4251234567". <br> <br>El restablecimiento de la contraseña no admite extensiones, aunque especifique una en el directorio. Las extensiones se quitan antes de realizar la llamada. Use un número sin una extensión o integre la extensión en el número de teléfono en su central de conmutación (PBX). |
| El usuario nunca recibe el correo electrónico de restablecimiento de la contraseña. | La causa más común de este problema es que un filtro de correo no deseado rechace el mensaje. Compruebe la carpeta de elementos eliminados o correo no deseado para comprobar si ha recibido el correo. <br> <br> Además, asegúrese de que el usuario comprueba la cuenta de correo electrónico correcta registrada en SSPR. |
| He configurado una directiva para restablecer la contraseña, pero, cuando una cuenta de administrador usa el restablecimiento de contraseña, esa directiva no se aplica. | Microsoft administra y controla la directiva de restablecimiento de contraseña de administrador para garantizar el nivel de seguridad más alto. |
| Se impide que el usuario trate de restablecer la contraseña demasiadas veces al día. | Se usa un mecanismo de limitación automática para evitar que los usuarios intenten restablecer sus contraseñas demasiadas veces en un breve período. La limitación se produce en los siguientes escenarios: <br><ul><li>Un usuario intenta validar un número de teléfono cinco veces en una hora.</li><li>Un usuario intenta utilizar la puerta de preguntas de seguridad cinco veces en una hora.</li><li>Un usuario intenta restablecer la contraseña de la misma cuenta de usuario cinco veces en una hora.</li></ul>Si un usuario experimenta este problema, debe esperar 24 horas después del último intento. El usuario puede entonces restablecer su contraseña. |
| El usuario experimenta un error al validar su número de teléfono. | Este error se produce cuando el número de teléfono especificado no coincide con el número de teléfono archivado. Asegúrese de que el usuario escribe el número de teléfono completo, incluidos el código de área y el país, al intentar utilizar un método basado en teléfono para restablecer la contraseña. |
| Se produce un error al procesar la solicitud. | Aunque los errores genéricos de registro de SSPR pueden tener varias causas, en general este error se debe a una interrupción del servicio o a un problema de configuración. Si sigue viendo este error genérico al reintentar el proceso de registro de SSPR, [póngase en contacto con el servicio de soporte técnico de Microsoft](#contact-microsoft-support) para obtener ayuda adicional. |
| Infracción de la directiva local | La contraseña no cumple la directiva de contraseñas de Active Directory local. El usuario debe definir una contraseña que cumpla los requisitos de complejidad o seguridad. |
| La contraseña no cumple la directiva aproximada | La contraseña que se ha usado aparece en la [lista de contraseñas prohibidas](./concept-password-ban-bad.md#how-are-passwords-evaluated) y no puede usarse. El usuario debe definir una contraseña que cumpla, o supere, la directiva de la lista de contraseñas prohibidas. |

## <a name="sspr-errors-that-a-user-might-see"></a>Errores de SSPR que puede ver un usuario

Los siguientes errores y detalles técnicos se le pueden presentar a un usuario como parte del proceso SSPR. A menudo, el error no es algo que pueda resolver por sí mismo, ya que la característica SSPR debe estar habilitada, configurada o registrada para su cuenta.

Use la siguiente información para comprender el problema y lo que debe corregirse en el inquilino de Azure AD o en una cuenta de usuario individual.

| Error | Detalles | Detalles técnicos |
| --- | --- | --- |
| TenantSSPRFlagDisabled = 9 | En este momento no se puede restablecer la contraseña porque el administrador ha deshabilitado la característica en la organización. No hay ninguna otra acción que puede realizar para resolver esta situación. Póngase en contacto con el administrador y pídale que habilite esta característica.<br /><br />Para más información, vea [Ayuda, he olvidado mi contraseña de Azure AD](../user-help/active-directory-passwords-update-your-own-password.md#common-problems-and-their-solutions). | SSPR_0009: se ha detectado que el administrador no ha habilitado el restablecimiento de contraseñas. Póngase en contacto con el administrador y pídale que lo habilite en la organización. |
| WritebackNotEnabled = 10 |En este momento no se puede restablecer la contraseña porque el administrador no ha habilitado un servicio necesario para la organización. No hay ninguna otra acción que puede realizar para resolver esta situación. Póngase en contacto con el administrador y pídale que compruebe la configuración de su organización.<br /><br />Para más información sobre este servicio necesario, vea [Configuración de la escritura diferida de contraseñas](./tutorial-enable-sspr-writeback.md). | SSPR_0010: se ha detectado que no se ha habilitado la escritura diferida de contraseñas. Póngase en contacto con el administrador y pídale que la habilite. |
| SsprNotEnabledInUserPolicy = 11 | En este momento no se puede restablecer la contraseña porque el administrador no ha configurado el restablecimiento de contraseñas en la organización. No hay ninguna otra acción que puede realizar para resolver esta situación. Póngase en contacto con él y pídale que lo configure.<br /><br />Para más información sobre la configuración del restablecimiento de contraseñas, consulte [Inicio rápido: Autoservicio de restablecimiento de contraseña de Azure AD](./tutorial-enable-sspr.md). | SSPR_0011: su organización no ha definido una directiva de restablecimiento de contraseña. Póngase en contacto con el administrador y pídale que defina una directiva de restablecimiento de contraseña. |
| UserNotLicensed = 12 | En este momento no se puede restablecer la contraseña porque faltan licencias necesarias en la organización. No hay ninguna otra acción que puede realizar para resolver esta situación. Póngase en contacto con el administrador y pídale que compruebe su asignación de licencias.<br /><br />Para más información sobre licencias, vea [Requisitos de concesión de licencias del autoservicio de restablecimiento de contraseña de Azure AD](./concept-sspr-licensing.md). | SSPR_0012: su organización no tiene las licencias necesarias para realizar el restablecimiento de contraseña. Póngase en contacto con el administrador y pídale que revise las asignaciones de licencia. |
| UserNotMemberOfScopedAccessGroup = 13 | En este momento no se puede restablecer la contraseña porque el administrador no ha configurado la cuenta para que use el restablecimiento de contraseñas. No hay ninguna otra acción que puede realizar para resolver esta situación. Póngase en contacto con el administrador y pídale que configure su cuenta con el restablecimiento de contraseña.<br /><br />Para más información sobre la configuración de cuentas en cuanto al restablecimiento de contraseñas, vea [Cómo implementar correctamente el lanzamiento del restablecimiento de contraseña de autoservicio](./howto-sspr-deployment.md). | SSPR_0013: no es miembro de un grupo habilitado para restablecer contraseñas. Póngase en contacto con el administrador y pídale que lo agregue al grupo. |
| UserNotProperlyConfigured = 14 | En este momento no se puede restablecer la contraseña porque falta información necesaria de su cuenta. No hay ninguna otra acción que puede realizar para resolver esta situación. Póngase en contacto con el administrador y pídale que restablezca la contraseña por usted. Después de tener acceso a su cuenta de nuevo, debe registrar la información requerida.<br /><br />Para ello, siga los pasos que se indican en el artículo [Registro para el autoservicio de restablecimiento de contraseñas](../user-help/active-directory-passwords-reset-register.md). | SSPR_0014: se necesita información de seguridad adicional para restablecer la contraseña. Para continuar, póngase en contacto con el administrador y pídale que restablezca la contraseña. Una vez tenga acceso a su cuenta, puede registrar información de seguridad adicional en https://aka.ms/ssprsetup. El administrador puede agregar información de seguridad adicional a su cuenta siguiendo los pasos de [Establecimiento y lectura de datos de autenticación mediante PowerShell](howto-sspr-authenticationdata.md). |
| OnPremisesAdminActionRequired = 29 | En este momento no se puede restablecer la contraseña debido a un problema con la configuración del restablecimiento de contraseña de la organización. No hay ninguna otra acción que puede realizar para resolver esta situación. Póngase en contacto con el administrador y pídale que lo investigue. <br /><br />O bien<br /><br />En este momento no se puede restablecer la contraseña debido a un problema con la configuración del restablecimiento de contraseña de la organización. No hay ninguna otra acción que pueda realizar para resolver esta situación. Póngase en contacto con el administrador y pídale que lo investigue.<br /><br />Para más información sobre el posible problema, vea [Solución de problemas de escritura diferida de contraseñas](troubleshoot-sspr-writeback.md). | SSPR_0029: no se puede restablecer la contraseña debido a un error en la configuración local. Póngase en contacto con el administrador y pídale que lo investigue. |
| OnPremisesConnectivityError = 30 | En este momento no se puede restablecer la contraseña debido a problemas de conectividad con su organización. No hay ninguna acción que pueda realizar ahora mismo, pero puede que el problema se resuelva si lo intenta más tarde. Si el problema continúa, póngase en contacto con el administrador y pídale que lo investigue.<br /><br />Para más información sobre problemas de conectividad, vea [Solución de problemas con la conectividad de la escritura diferida de contraseñas](troubleshoot-sspr-writeback.md). | SSPR_0030: no se puede restablecer la contraseña debido a una mala conexión con el entorno local. Póngase en contacto con el administrador y pídale que lo investigue.|

## <a name="azure-ad-forums"></a>Foros de Azure AD

Si tiene alguna pregunta general sobre Azure AD y el autoservicio de restablecimiento de contraseña, puede pedir ayuda a la comunidad en la [Página de preguntas y respuesta de Microsoft para Azure Active Directory](/answers/topics/azure-active-directory.html). La comunidad está formada por ingenieros, jefes de producto, MVP y profesionales de TI.

## <a name="contact-microsoft-support"></a>Consultar al soporte técnico de Microsoft

Si no encuentra la respuesta a un problema, nuestros equipos de soporte técnico están siempre disponibles para ayudarle.

Para que reciba la ayuda apropiada, le pedimos que proporcione la mayor cantidad de detalles posible al abrir una incidencia. Estos detalles incluyen los siguientes:

* **Descripción general del error**: ¿Cuál es el error? ¿Qué comportamiento observó? ¿Cómo podemos reproducir el error? Proporcione tantos detalles como sea posible.
* **Página**: ¿en qué página estaba cuando se detectó el error? Incluya la dirección URL, si es posible, y una captura de pantalla de la página.
* **Código de soporte técnico**: ¿qué código de soporte técnico se generó cuando el usuario vio el error?
   * Para encontrarlo, reproduzca el error, seleccione el vínculo **Código de soporte técnico** en la parte inferior de la pantalla y envíe al ingeniero de soporte técnico el GUID resultante.

    :::image type="content" source="./media/troubleshoot-sspr-writeback/view-support-code.png" alt-text="El código de soporte técnico se encuentra en la parte inferior derecha de la ventana del explorador web.":::

  * Si se encuentra en una página sin código de soporte en la parte inferior, seleccione F12, busque el SID y el CID, y envíe estos dos resultados al ingeniero de soporte.
* **Fecha, hora y zona horaria**: incluya la fecha y la hora precisas (incluida la *zona horaria*) en que se produjo el error.
* **Id. de usuario**: ¿quién fue el usuario que vio el error? Un ejemplo es *usuario\@contoso.com*.
   * ¿Es un usuario federado?
   * ¿Es un usuario de autenticación de paso a través?
   * ¿Es un usuario con sincronización de hash de contraseña?
   * ¿Es un usuario solo de nube?
* **Licencia**: ¿el usuario tiene asignada una licencia de Azure AD?
* **Registro de eventos de aplicación**: si usa la escritura diferida de contraseñas y el error se produce en la infraestructura local, incluya una copia comprimida del registro de eventos de la aplicación desde el servidor de Azure AD Connect.

## <a name="next-steps"></a>Pasos siguientes

Para más información sobre SSPR, consulte [Funcionamiento: Autoservicio de restablecimiento de contraseña de Azure AD](concept-sspr-howitworks.md) o [¿Cómo funciona la escritura diferida del autoservicio de restablecimiento de contraseña en Azure Active Directory?](concept-sspr-writeback.md).
