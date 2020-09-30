---
title: 'Solución de problemas de la escritura diferida del autoservicio de restablecimiento de contraseña: Azure Active Directory'
description: Aprenda a solucionar los problemas comunes de la escritura diferida del autoservicio de restablecimiento de contraseña en Azure Active Directory.
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
ms.openlocfilehash: 50e202d26574c0fc8adfeb7f73eb150ebb1781af
ms.sourcegitcommit: f8d2ae6f91be1ab0bc91ee45c379811905185d07
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/10/2020
ms.locfileid: "89663353"
---
# <a name="troubleshoot-self-service-password-reset-writeback-in-azure-active-directory"></a>Solución de problemas de la escritura diferida del autoservicio de restablecimiento de contraseña en Azure Active Directory

El autoservicio de restablecimiento de contraseña de Azure Active Directory (Azure AD) permite a los usuarios restablecer sus contraseñas en la nube. La escritura diferida de contraseñas es una característica que se habilita con [Azure AD Connect](../hybrid/whatis-hybrid-identity.md) y que permite que los cambios de contraseña en la nube se escriban en diferido en un directorio local existente en tiempo real.

Si tiene problemas con la escritura diferida de SSPR, puede que le sean de utilidad los pasos de resolución y los errores comunes que se describen a continuación. Si no encuentra la respuesta a un problema, [nuestros equipos de soporte técnico](#contact-microsoft-support) están siempre disponibles para ayudarle.

## <a name="troubleshoot-connectivity"></a>Solución de problemas de conectividad

Si tiene problemas con la escritura diferida de contraseñas en Azure AD Connect, revise los siguientes pasos que pueden ayudar a resolverlos. Para recuperar el servicio, se recomienda seguir estos pasos en orden:

* [Confirmación de la conectividad de la red](#confirm-network-connectivity)
* [Reinicio del servicio Azure AD Connect Sync](#restart-the-azure-ad-connect-sync-service)
* [Deshabilitar y volver a habilitar la característica de escritura diferida de contraseña](#disable-and-re-enable-the-password-writeback-feature)
* [Instalación de la última versión de Azure AD Connect](#install-the-latest-azure-ad-connect-release)
* [Solución de problemas con la escritura diferida de contraseñas](#common-password-writeback-errors)

### <a name="confirm-network-connectivity"></a>Confirmación de la conectividad de la red

El punto de error más común es la configuración incorrecta del firewall o los puertos del proxy y los tiempos de inactividad.

En el caso de la versión *1.1.443.0* y superior de Azure AD Connect, se necesita el acceso de *HTTPS de salida* a las siguientes direcciones:

* *\*.passwordreset.microsoftonline.com*
* *\*.servicebus.windows.net*

Si necesita más detalles, consulte la [lista de intervalos de dirección IP del centro de datos de Microsoft Azure](https://www.microsoft.com/download/details.aspx?id=41653). Esta lista se actualiza todos los miércoles y entra en vigor el lunes siguiente.

Para más información, consulte los [requisitos previos de conectividad para Azure AD Connect](../hybrid/how-to-connect-install-prerequisites.md).

### <a name="restart-the-azure-ad-connect-sync-service"></a>Reinicio del servicio Azure AD Connect Sync

Para resolver problemas de conectividad u otros problemas transitorios con el servicio, realice los pasos siguientes para reiniciar el servicio Azure AD Connect Sync:

1. Como administrador en el servidor que ejecuta Azure AD Connect, seleccione **Inicio**.
1. Escriba *services.msc* en el campo de búsqueda y seleccione **Entrar**.
1. Busque la entrada *Microsoft Azure AD Sync*.
1. Haga clic con el botón derecho en la entrada del servicio, seleccione **Reiniciar** y espere a que se complete la operación.

    :::image type="content" source="./media/troubleshoot-sspr-writeback/service-restart.png" alt-text="Reinicio del servicio de sincronización de Azure AD mediante la GUI" border="false":::

Con estos pasos, se restablecerá la conexión con Azure AD y se resolverán los problemas de conectividad.

Si al reiniciar el servicio Azure AD Connect Sync no se resuelve el problema, pruebe a deshabilitar y volver a habilitar la característica de escritura diferida de contraseñas en la sección siguiente.

### <a name="disable-and-re-enable-the-password-writeback-feature"></a>Deshabilitar y volver a habilitar la característica de escritura diferida de contraseñas

Para seguir solucionando problemas, realice estos pasos para deshabilitar y volver a habilitar la característica de escritura diferida de contraseñas:

1. Como administrador en el servidor que ejecuta Azure AD Connect, abra el **asistente para configuración de Azure AD Connect**.
1. En **Conectarse a Azure AD**, escriba las credenciales de administrador global de Azure AD.
1. En **Conectarse a AD DS**, escriba las credenciales de administrador de Active Directory Domain Services local.
1. En **Identificación de forma exclusiva de usuarios**, seleccione el botón **Siguiente**.
1. En **Características opcionales**, desactive la casilla **Escritura diferida de contraseñas**.
1. Seleccione **Siguiente** en el resto de las páginas del cuadro de diálogo sin cambiar nada hasta llegar a la página **Listo para configurar**.
1. Compruebe que en la página **Listo para configurar** la opción *Escritura diferida de contraseñas* aparece como *Deshabilitada*. Seleccione el botón verde **Configurar** para confirmar los cambios.
1. En **Finalizado**, desactive la opción **Sincronizar ahora** y, a continuación, seleccione **Finalizar** para cerrar el asistente.
1. Vuelva a abrir el **asistente para configuración de Azure AD Connect**.
1. Repita los pasos del 2 al 8, pero, esta vez, seleccione la opción *Escritura diferida de contraseñas* en la pantalla **Características opcionales** para volver a habilitar el servicio.

Con estos pasos, se restablecerá la conexión con Azure AD y se resolverán los problemas de conectividad.

Si el problema no se soluciona después de deshabilitar y volver a habilitar la característica de escritura diferida de contraseñas, vuelva a instalar Azure AD Connect en la siguiente sección.

### <a name="install-the-latest-azure-ad-connect-release"></a>Instalación de la última versión de Azure AD Connect

Con la reinstalación de Azure AD Connect se pueden resolver los problemas de configuración y conectividad entre Azure AD y el entorno local de Active Directory Domain Services. Este paso solo se recomienda después de intentar los pasos anteriores para comprobar y solucionar los problemas de conectividad.

> [!WARNING]
> Si ha personalizado las reglas de sincronización predefinidas, *realice una copia de seguridad de ellas antes de continuar con la actualización y, cuando haya terminado, vuelva a implementarlas manualmente*.

1. Descargue la versión más reciente de AD Connect del [Centro de descarga de Microsoft](https://go.microsoft.com/fwlink/?LinkId=615771).
1. Puesto que ya ha instalado Azure AD Connect, realice una actualización local para actualizar la instalación de Azure AD Connect a la versión más reciente.

    Ejecute el paquete descargado y siga las instrucciones en pantalla para actualizar Azure AD Connect.

Con estos pasos, se restablecerá la conexión con Azure AD y se resolverán los problemas de conectividad.

Si con la instalación de la versión más reciente del servidor de Azure AD Connect no se resuelve el problema, intente deshabilitar y volver a habilitar la escritura diferida de contraseñas como último paso después de instalar la versión más reciente.

## <a name="verify-that-azure-ad-connect-has-the-required-permissions"></a>Comprobación de que Azure AD Connect tiene los permisos necesarios

Azure AD Connect necesita el permiso **Restablecer contraseña** de AD DS para realizar la escritura diferida de contraseñas. Para averiguar si Azure AD Connect tiene el permiso necesario para una cuenta de usuario de AD DS local determinada, use la característica **Permisos efectivos de Windows**:

1. Inicie sesión en el servidor Azure AD Connect e inicie **Synchronization Service Manager** seleccionando **Inicio**  > **Synchronization Service**.
1. En la pestaña **Conectores** seleccione el conector **Active Directory Domain Services** local y, a continuación, seleccione **Propiedades**.

    :::image type="content" source="./media/troubleshoot-sspr-writeback/synchronization-service-manager.png" alt-text="Reinicio del servicio de sincronización de Azure AD mediante la GUI" border="false":::
  
1. En la ventana emergente, seleccione **Connect to Active Directory Forest** (Conectar con el bosque de Active Directory) y anote el valor de la propiedad **Nombre de usuario**. Esta propiedad es la cuenta de AD DS que Azure AD Connect usa para realizar la sincronización de directorios.

    Para que Azure AD Connect realice la escritura diferida de contraseñas, la cuenta de AD DS debe tener permiso para restablecer la contraseña. En los pasos siguientes se comprobarán los permisos de esta cuenta de usuario.

    :::image type="content" source="./media/troubleshoot-sspr-writeback/synchronization-service-manager-properties.png" alt-text="Reinicio del servicio de sincronización de Azure AD mediante la GUI" border="false":::
  
1. Inicie sesión en un controlador de dominio local e inicie la aplicación **Usuarios y equipos de Active Directory**.
1. Seleccione **Vista** y asegúrese de que la opción **Características avanzadas** está habilitada.  

    :::image type="content" source="./media/troubleshoot-sspr-writeback/view-advanced-features.png" alt-text="Reinicio del servicio de sincronización de Azure AD mediante la GUI" border="false":::
  
1. Busque la cuenta de usuario de AD DS que quiere comprobar. Haga clic con el botón derecho en el nombre de la cuenta y seleccione **Propiedades**.  
1. En la ventana emergente, vaya a la pestaña **Seguridad** y seleccione **Avanzada**.  
1. En la ventana emergente **Advanced Security Settings for Administrator** (Configuración de seguridad avanzada para el administrador), vaya a la pestaña **Acceso efectivo**.
1. Elija **Seleccionar un usuario**, la cuenta de AD DS que usa Azure AD Connect (consulte el paso 3) y, luego, **Ver acceso efectivo**.

    :::image type="content" source="./media/troubleshoot-sspr-writeback/view-effective-access.png" alt-text="Reinicio del servicio de sincronización de Azure AD mediante la GUI" border="false":::
  
1. Desplácese hacia abajo y busque **Restablecer contraseña**. Si la entrada está activada, significa que la cuenta de AD DS tiene permiso para restablecer la contraseña de la cuenta de usuario de Active Directory seleccionada.  

    :::image type="content" source="./media/troubleshoot-sspr-writeback/check-permissions.png" alt-text="Reinicio del servicio de sincronización de Azure AD mediante la GUI" border="false":::

## <a name="common-password-writeback-errors"></a>Errores comunes de la escritura diferida de contraseñas

Pueden producirse los siguientes problemas más específicos con la escritura diferida de contraseñas. Si experimenta alguno de estos errores, revise la solución propuesta y compruebe si la escritura diferida de contraseñas funciona correctamente.

| Error | Solución |
| --- | --- |
| El servicio de restablecimiento de contraseña no se inicia de forma local. Aparece el error 6800 en el registro de eventos de aplicación de la máquina de Azure AD Connect. <br> <br> Después de la incorporación, los usuarios federados, con autenticación de paso a través o con sincronización de hash de contraseña no pueden restablecer sus contraseñas. | Cuando está habilitada la escritura diferida de contraseñas, el motor de sincronización llama a la biblioteca de escritura diferida para realizar la configuración (incorporación) comunicándose con el servicio de incorporación de la nube. Los errores detectados durante la incorporación o al iniciar el punto de conexión de Windows Communication Foundation (WCF) para la escritura diferida de contraseñas producirán otros errores en el registro de eventos de la máquina de Azure AD Connect. <br> <br> Durante el reinicio del servicio Azure AD Sync (ADSync), si se configuró la escritura diferida, se inicia el punto de conexión de WCF. Sin embargo, si se produce un error en el inicio del punto de conexión, se registra el evento 6800 y se deja que se inicie el servicio de sincronización. La presencia de este evento significa que el punto de conexión de la escritura diferida de contraseñas no se ha iniciado. Los detalles del registro de eventos para este evento (6800) junto con sus entradas generadas por el componente PasswordResetService indican por qué el punto de conexión no se ha podido iniciar. Revise estos errores del registro de eventos e intente volver a iniciar Azure AD Connect si la escritura diferida de contraseñas sigue sin funcionar. Si el problema persiste, intente deshabilitar la escritura diferida de contraseñas y vuelva a habilitarla.
| Cuando un usuario intenta restablecer una contraseña o desbloquear una cuenta con la escritura diferida de contraseñas habilitada, se produce un error en la operación. <br> <br> Además, verá un evento en el registro de eventos de Azure AD Connect que contiene: "Synchronization Engine returned an error hr=800700CE, message=The filename or extension is too long" (El motor de sincronización devolvió un error hr=800700CE, mensaje=El nombre de archivo o la extensión son demasiado largos) después de la operación de desbloqueo. | Busque la cuenta de Active Directory para Azure AD Connect y restablezca la contraseña para que no contenga más de 256 caracteres. Luego, abra el **Servicio de sincronización** desde el menú **Inicio**. Vaya a **Conectores** y busque el **Conector Active Directory**. Selecciónelo y, a continuación, seleccione **Propiedades**. Vaya a la página **Credenciales** y escriba la nueva contraseña. Seleccione **Aceptar** para cerrar la página. |
| En el último paso del proceso de instalación de Azure AD Connect, se generará un error que indica que no se ha podido configurar la escritura diferida de contraseñas. <br> <br> El registro de eventos de la aplicación de Azure AD Connect contiene el error 32009 con el texto "Error getting auth token" (Error al obtener el token de autorización). | Este error se produce en los dos casos siguientes: <br><ul><li>Ha especificado una contraseña incorrecta para la cuenta de administrador global proporcionada al principio del proceso de instalación de Azure AD Connect.</li><li>Ha tratado de usar un usuario federado para la cuenta de administrador global especificada al principio del proceso de instalación de Azure AD Connect.</li></ul> Para corregir este error, asegúrese de que no esté usando una cuenta federada para el administrador global que especificó al principio del proceso de instalación y que la contraseña especificada sea correcta. |
| El registro de eventos de la máquina de Azure AD Connect contiene el error 32002 que se genera al ejecutar PasswordResetService. <br> <br> El error reza como sigue: "Error Connecting to ServiceBus. The token provider was unable to provide a security token (Error al conectar con ServiceBus: El proveedor de tokens no pudo proporcionar un token de seguridad). | El entorno local no es capaz de conectarse al punto de conexión de la instancia de Azure Service Bus en la nube. Este error se debe normalmente a una regla de firewall que bloquea una conexión saliente a una dirección de puerto o web determinada. Vea [Requisitos previos de conectividad](../hybrid/how-to-connect-install-prerequisites.md) para más información. Una vez que haya actualizado estas reglas, reinicie el servidor de Azure AD Connect y la escritura diferida de contraseñas debería empezar a funcionar de nuevo. |
| Después de trabajar un tiempo, los usuarios federados, con autenticación de paso a través o con sincronización de hash de contraseña no pueden restablecer sus contraseñas. | En algunos casos excepcionales, el servicio de escritura diferida de contraseñas puede no volver a iniciarse cuando se reinicia Azure AD Connect. En estos casos, compruebe primero si la escritura diferida de contraseñas está habilitada en el entorno local. Para ello, use el asistente de Azure AD Connect o PowerShell. Si la característica aparece como habilitada, intente habilitarla o deshabilitarla de nuevo. Si este paso de solución de problemas no funciona, pruebe a desinstalar Azure AD Connect por completo y volver a instalarlo. |
| Los usuarios federados, con autenticación de paso a través o con sincronización de hash de contraseña que intenten restablecer sus contraseñas verán un error después de intentar enviar su contraseña. El error indica que ha habido un problema del servicio. <br ><br> Además, durante las operaciones de restablecimiento de contraseña, podría ver un error relacionado con que al agente de administración se le denegó el acceso a los registros de eventos locales. | Si ve estos errores en el registro de eventos, confirme que la cuenta del Agente de administración de Active Directory (ADMA) (que se especificó en el asistente en el momento de la configuración) tiene los permisos necesarios para la escritura diferida de contraseñas. <br> <br> Una vez que se concede este permiso, puede tardar hasta una hora en poder usarse a través de la tarea en segundo plano `sdprop`, en el controlador de dominio (DC). <br> <br> Para que el restablecimiento de contraseña funcione, el permiso debe quedar marcado en el descriptor de seguridad del objeto de usuario cuya contraseña se está restableciendo. Hasta que este permiso se muestra en el objeto de usuario, el restablecimiento de la contraseña seguirá generando un mensaje de acceso denegado. |
| Los usuarios federados, con autenticación de paso a través o con sincronización de hash de contraseña que intenten restablecer sus contraseñas verán un error después de enviar su contraseña. El error indica que ha habido un problema del servicio. <br> <br> Además de este problema, durante las operaciones de restablecimiento de contraseña, puede aparecer un error en los registros de eventos del servicio Azure AD Connect con el mensaje “No se encontró el objeto”. | Este error suele indicar que el motor de sincronización no puede encontrar el objeto de usuario en el espacio del conector de Azure AD o en el objeto del espacio del conector de Azure AD o del metaverso vinculado (MV). <br> <br> Para solucionar este problema, asegúrese de que el usuario realmente se sincroniza desde el entorno local a Azure AD a través de la instancia actual de Azure AD Connect e inspeccione el estado de los objetos de los espacios de conector y MV. Confirme que el objeto de Servicios de certificados de Active Directory (AD CS) está conectado al objeto de MV mediante la regla "Microsoft.InfromADUserAccountEnabled.xxx".|
| Los usuarios federados, con autenticación de paso a través o con sincronización de hash de contraseña que intenten restablecer sus contraseñas verán un error después de enviar su contraseña. El error indica que ha habido un problema del servicio. <br> <br> Además de este problema, durante las operaciones de restablecimiento de contraseña, puede recibir un error en los registros de eventos del servicio Azure AD Connect que indique que "se encontraron varias coincidencias". | Significa que el motor de sincronización ha detectado que el objeto de MV está conectado a más de un objeto de AD CS a través de "Microsoft.InfromADUserAccountEnabled.xxx". Esto significa que el usuario tiene una cuenta habilitada en más de un bosque. Este escenario no se admite para la escritura diferida de contraseñas. |
| Error de configuración en las operaciones con contraseñas. El registro de eventos de la aplicación contiene el error 6329 de Azure AD Connect con el texto: 0x8023061f (The operation failed because password synchronization is not enabled on this Management Agent) (Error en la operación porque no está habilitada la sincronización de contraseñas en este agente de administración). | Este error se produce si se cambia la configuración de Azure AD Connect para agregar un nuevo bosque de Active Directory (o para quitar y volver a agregar un bosque existente) después de que ya se haya habilitado la escritura diferida de contraseñas. Se producirá un error en las operaciones de contraseñas para los usuarios de estos bosques recién agregados. Para corregir el problema, deshabilite la característica de escritura diferida de contraseñas después de que se hayan completado los cambios de configuración del bosque y vuelva a habilitarla entonces. |

## <a name="password-writeback-event-log-error-codes"></a>Códigos de error de registro de eventos de escritura diferida de contraseñas

Un procedimiento recomendado para solucionar problemas con la escritura diferida de contraseñas consiste en inspeccionar el registro de eventos de la aplicación en la máquina de Azure AD Connect. Este registro de eventos contiene eventos de dos orígenes para la escritura diferida de contraseñas. El origen *PasswordResetService* describe las operaciones y los problemas relacionados con el funcionamiento de la escritura diferida de contraseñas. El origen *ADSync* describe las operaciones y los problemas relacionados con la configuración de contraseñas en el entorno de Active Directory Domain Services.

### <a name="if-the-source-of-the-event-is-adsync"></a>Si el origen del evento es ADSync

| Código | Nombre o mensaje | Descripción |
| --- | --- | --- |
| 6329 | BAIL: MMS(4924) 0x80230619: "A restriction prevents the password from being changed to the current one specified" (Una restricción impide que la contraseña se modifique por la que ha especificado actualmente). | Este evento se produce cuando el servicio de escritura diferida de contraseñas intenta establecer una contraseña en su directorio local que no cumple los requisitos del dominio en cuanto a antigüedad, historial, complejidad o filtrado de la contraseña. <br> <br> Si tiene una vigencia mínima de la contraseña y ha cambiado recientemente la contraseña dentro de ese margen de tiempo, no puede volver a cambiarla hasta que alcance la duración especificada en el dominio. Para las pruebas, la vigencia mínima debe establecerse en 0. <br> <br> Si tiene habilitados los requisitos del historial de contraseñas, debe seleccionar una contraseña que no se haya utilizado las últimas *N* veces, donde *N* es el valor del historial de contraseñas. Si selecciona una contraseña que se haya usado en las últimas *N* veces, verá un error en este caso. Para las pruebas, el historial mínimo debe establecerse en 0. <br> <br> Si tiene requisitos de complejidad de contraseña, todos ellos se aplican cuando el usuario intenta cambiar o restablecer una contraseña. <br> <br> Si tiene habilitados filtros de contraseña y un usuario selecciona una contraseña que no cumple los criterios de filtrado, se producirá un error en la operación de restablecimiento o modificación. |
| 6329 | MMS(3040): admaexport.cpp(2837): The server doesn't contain the LDAP password policy control (El servidor no contiene el control de directiva de contraseñas LDAP). | Este problema se produce si el control LDAP_SERVER_POLICY_HINTS_OID (1.2.840.113556.1.4.2066) no está habilitado en los controladores de dominio. Para usar la característica de escritura diferida de contraseña, debe habilitar el control. Para ello, los controladores de dominio deben estar en Windows Server 2008R2 o una versión posterior. |
| HR 8023042 | El motor de sincronización devolvió un error hr=80230402, mensaje=Error al intentar obtener un objeto debido a que hay entradas duplicadas con el mismo delimitador. | Este error se produce cuando se habilita el mismo identificador de usuario en varios dominios. Por ejemplo se produciría si se están sincronizando los bosques de cuentas y recursos, y tienen el mismo identificador de usuario que está habilitado en cada bosque. <br> <br> Este error también puede ocurrir si usa un atributo delimitador que no sea único, como un alias o UPN, y dos usuarios comparten el mismo. <br> <br> Para resolver este problema, asegúrese de no tener ningún usuario duplicado dentro de los dominios y de estar utilizando un atributo delimitador único para cada usuario. |

### <a name="if-the-source-of-the-event-is-passwordresetservice"></a>El origen del evento es PasswordResetService

| Código | Nombre o mensaje | Descripción |
| --- | --- | --- |
| 31001 | PasswordResetStart | Este evento indica que el servicio local detectó una solicitud de restablecimiento de contraseña para un usuario federado, con autenticación de paso a través o con sincronización de hash de contraseñas, que se origina en la nube. Este evento es el primero en cada operación de escritura diferida de contraseñas. |
| 31002 | PasswordResetSuccess | Este evento indica que un usuario seleccionó una contraseña nueva durante una operación de restablecimiento de contraseña. Determinamos que dicha contraseña cumple los requisitos de las contraseñas corporativas. La escritura diferida de la contraseña se realizó correctamente en el entorno de Active Directory local. |
| 31003 | PasswordResetFail | Este evento indica que un usuario seleccionó una contraseña y esta llegó correctamente al entorno local. Pero, al intentar establecer la contraseña en el entorno de Active Directory local, se produjo un error. Esto puede ocurrir por varios motivos: <br><ul><li>La contraseña del usuario no cumple los requisitos de antigüedad, historial, complejidad o filtro del dominio. Para resolver este problema, cree una contraseña nueva.</li><li>La cuenta de servicio de ADMA no tiene los permisos adecuados para establecer la nueva contraseña en la cuenta de usuario en cuestión.</li><li>La cuenta de usuario está en un grupo protegido, como el de administradores de empresa o dominio, lo que impide la ejecución de operaciones de establecimiento de contraseñas.</li></ul>|
| 31004 | OnboardingEventStart | Este evento se produce si habilita la escritura diferida de contraseñas con Azure AD Connect y hemos iniciado la incorporación en la organización del servicio web de escritura diferida de contraseñas. |
| 31005 | OnboardingEventSuccess | Este evento indica que el proceso de incorporación se realizó correctamente y que la funcionalidad de escritura diferida de contraseñas está lista para usarse. |
| 31006 | ChangePasswordStart | Este evento indica que el servicio local detectó una solicitud de cambio de contraseña para un usuario federado, con autenticación de paso a través o con sincronización de hash de contraseña, que se origina en la nube. Este evento es el primero de cada operación de escritura diferida de cambio de contraseña. |
| 31007 | ChangePasswordSuccess | Indica que un usuario seleccionó una nueva contraseña durante una operación de cambio de contraseña, determinamos que esta contraseña cumple los requisitos corporativos de las contraseñas y que esa contraseña se ha escrito en diferido correctamente en el entorno de Active Directory local. |
| 31008 | ChangePasswordFail | Este evento indica que un usuario seleccionó una contraseña y que esta ha llegado correctamente al entorno local, pero, al intentar establecer la contraseña en el entorno de Active Directory local, se produjo un error. Esto puede ocurrir por varios motivos: <br><ul><li>La contraseña del usuario no cumple los requisitos de antigüedad, historial, complejidad o filtro del dominio. Para resolver este problema, cree una contraseña nueva.</li><li>La cuenta de servicio de ADMA no tiene los permisos adecuados para establecer la nueva contraseña en la cuenta de usuario en cuestión.</li><li>La cuenta de usuario está en un grupo protegido, como administradores del dominio o de empresa, lo que impide la ejecución de operaciones de conjunto de contraseñas.</li></ul> |
| 31009 | ResetUserPasswordByAdminStart | El servicio local detectó una solicitud de restablecimiento de contraseña para un usuario federado, con autenticación de paso a través o con sincronización de hash de contraseña, originada en el administrador en nombre de un usuario. Este evento es el primero en una operación de escritura diferida de restablecimiento de contraseña que fue iniciada por el administrador. |
| 31010 | ResetUserPasswordByAdminSuccess | El administrador seleccionó una contraseña nueva durante una operación de restablecimiento de contraseña iniciada por él. Determinamos que dicha contraseña cumple los requisitos de las contraseñas corporativas. La escritura diferida de la contraseña se realizó correctamente en el entorno de Active Directory local. |
| 31011 | ResetUserPasswordByAdminFail | El administrador seleccionó una contraseña en nombre del usuario. La contraseña ha llegado correctamente al entorno local. Pero, al intentar establecer la contraseña en el entorno de Active Directory local, se produjo un error. Esto puede ocurrir por varios motivos: <br><ul><li>La contraseña del usuario no cumple los requisitos de antigüedad, historial, complejidad o filtro del dominio. Para resolver este problema, pruebe con una contraseña nueva.</li><li>La cuenta de servicio de ADMA no tiene los permisos adecuados para establecer la nueva contraseña en la cuenta de usuario en cuestión.</li><li>La cuenta de usuario está en un grupo protegido, como administradores del dominio o de empresa, lo que impide la ejecución de operaciones de conjunto de contraseñas.</li></ul>  |
| 31012 | OffboardingEventStart | Este evento se produce si habilita la escritura diferida de contraseñas con Azure AD Connect e indica que se inició la externalización en su organización del servicio web de escritura diferida de contraseñas. |
| 31013| OffboardingEventSuccess| Este evento indica que el proceso de externalización se realizó correctamente y que la funcionalidad de escritura diferida de contraseñas se ha deshabilitado también correctamente. |
| 31014| OffboardingEventFail| Este evento indica que el proceso de retirada no se realizó correctamente. Puede deberse a un error de permisos en la cuenta de administrador en la nube o local especificada durante la configuración. El error también puede producirse si está intentando utilizar un administrador global en la nube federado al deshabilitar la escritura diferida de contraseñas. Para solucionar este problema, compruebe los permisos administrativos y asegúrese de que no se utilice ninguna cuenta federada al configurar la funcionalidad de escritura diferida de contraseñas.|
| 31015| WriteBackServiceStarted| Este evento indica que el servicio de escritura diferida de contraseñas se ha iniciado correctamente. Está listo para aceptar las solicitudes de administración de contraseñas de la nube.|
| 31016| WriteBackServiceStopped| Este evento indica que el servicio de escritura diferida de contraseñas se ha detenido. Todas las solicitudes de administración de contraseñas de la nube fracasarán.|
| 31017| AuthTokenSuccess| Este evento indica que se ha recuperado correctamente un token de autorización para el administrador global especificado durante la instalación de Azure AD Connect para iniciar el proceso de incorporación o externalización.|
| 31018| KeyPairCreationSuccess| Este evento indica que se creó correctamente la clave de cifrado de contraseña. Esta clave se usa para que las contraseñas cifradas de la nube se envíen al entorno local.|
| 32000| UnknownError| Este evento indica que se produjo un error desconocido durante una operación de administración de contraseñas. Observe el texto de excepción en el evento para obtener más detalles. Si tiene problemas, pruebe a deshabilitar y volver a habilitar la escritura diferida de contraseñas. Si así no se soluciona el problema, incluya una copia del registro de eventos junto con el identificador de seguimiento especificado al abrir una solicitud de soporte técnico.|
| 32001| ServiceError| Este evento indica que se ha producido un error al conectarse a la instancia de restablecimiento de contraseñas de la nube. Este error suele producirse cuando el servicio local no pudo conectarse al servicio web de restablecimiento de contraseña.|
| 32002| ServiceBusError| Este evento indica que se ha producido un error al conectarse a la instancia de Service Bus del inquilino. Esto puede deberse a que está bloqueando las conexiones salientes en el entorno local. Compruebe el firewall para asegurarse de que permite conexiones a través del TCP 443 y a https://ssprdedicatedsbprodncu.servicebus.windows.net, y vuelva a intentarlo. Si sigue teniendo problemas, pruebe a deshabilitar y volver a habilitar la escritura diferida de contraseñas.|
| 32003| InPutValidationError| Este evento indica que la entrada transferida a la API del servicio web no era válida. Vuelva a intentarlo.|
| 32004| DecryptionError| Este evento indica que se ha producido un error al descifrar la contraseña que ha llegado de la nube. Podría deberse a una falta de coincidencia de la clave de descifrado entre el servicio en la nube y su entorno local. Para resolver este problema, deshabilite y vuelva a habilitar la escritura diferida de contraseñas en su entorno local.|
| 32005| ConfigurationError| Durante la incorporación, guardamos la información específica del inquilino en un archivo de configuración en su entorno local. Este evento indica que se ha producido un error al guardar este archivo o que cuando se inició el servicio hubo un error en la lectura del archivo. Para corregir este problema, pruebe a deshabilitar y volver a habilitar la escritura diferida de contraseñas para forzar la reescritura del archivo de configuración.|
| 32007| OnBoardingConfigUpdateError| Durante la incorporación, enviamos datos de la nube al servicio de restablecimiento de contraseñas local. Esos datos, a continuación, se escriben en un archivo en memoria antes de enviarse al servicio de sincronización para almacenarse de forma segura en el disco. Este evento indica un problema con la escritura o actualización de los datos en memoria. Para corregir este problema, pruebe a deshabilitar y volver a habilitar la escritura diferida de contraseñas para forzar la reescritura de este archivo de configuración.|
| 32008| ValidationError| Este evento indica que hemos recibido una respuesta no válida desde el servicio web de restablecimiento de contraseña. Para solucionar este problema, pruebe a deshabilitar y volver a habilitar la escritura diferida de contraseñas.|
| 32009| AuthTokenError| Este evento indica que no se ha podido obtener un token de autorización para la cuenta de administrador global especificada durante la instalación de Azure AD Connect. Este error puede deberse a un nombre de usuario o contraseña especificados para la cuenta de administrador global. También puede producirse si la cuenta de administrador global especificada está federada. Para corregir este problema, vuelva a ejecutar la configuración con el nombre de usuario y la contraseña correctos, y asegúrese de que el administrador es una cuenta administrada (solo en la nube o con la sincronización de contraseñas).|
| 32010| CryptoError| Este evento indica que se produjo un error al generar la clave de cifrado de contraseña o al descifrar una contraseña que llega desde el servicio en la nube. Este error probablemente indica un problema con su entorno. Consulte los detalles de su registro de eventos para obtener más información y resolver este problema. También puede intentar deshabilitar y volver a habilitar el servicio de escritura diferida de contraseñas.|
| 32011| OnBoardingServiceError| Este evento indica que el servicio local no pudo comunicarse correctamente con el servicio web de restablecimiento de contraseña para iniciar el proceso de incorporación. Puede ocurrir como resultado de una regla de firewall o si hay un problema al obtener un token de autenticación para el inquilino. Para corregir este problema, asegúrese de que no esté bloqueando las conexiones salientes a través del TCP 443 y TCP 9350-9354 o a https://ssprdedicatedsbprodncu.servicebus.windows.net. Asegúrese también de que la cuenta de administrador de Azure AD que usa para la incorporación no está federada.|
| 32013| OffBoardingError| Este evento indica que el servicio local no pudo comunicarse correctamente con el servicio web de restablecimiento de contraseña para iniciar el proceso de externalización. Puede ocurrir como resultado de una regla de firewall o si hay un problema al obtener un token de autorización para el inquilino. Para solucionar este problema, asegúrese de que no esté bloqueando las conexiones salientes a través de 443 o a https://ssprdedicatedsbprodncu.servicebus.windows.net, y que la cuenta de administrador de Azure Active Directory utilizada para la externalización no esté federada.|
| 32014| ServiceBusWarning| Este evento indica que tenemos que tratar de volver a establecer la conexión a la instancia de Service Bus del inquilino. En condiciones normales, esto no debe ser un problema, pero si este evento se produce muchas veces, considere la posibilidad de comprobar la conexión de red a Service Bus, especialmente si se trata de una conexión de ancho de banda bajo o de latencia alta.|
| 32015| ReportServiceHealthError| Para supervisar el estado de su servicio de escritura diferida de contraseñas, enviamos datos de latido a nuestro servicio web de restablecimiento de contraseñas cada cinco minutos. Este evento indica que se ha producido un error al enviar esta información de latido al servicio web en la nube. Esta información de estado no incluye datos personales y se trata únicamente de un latido y estadísticas de servicio básicas para que podamos ofrecer información de estado del servicio en la nube.|
| 33001| ADUnKnownError| Este evento indica que se produjo un error desconocido que devuelve Active Directory. Compruebe en el registro de eventos de servidor de Azure AD Connect si hay eventos del origen ADSync para obtener más información.|
| 33002| ADUserNotFoundError| Este evento indica que el usuario que está intentando restablecer o cambiar una contraseña no se encontró en el directorio local. Esto puede ocurrir cuando el usuario se ha eliminado en el entorno local pero no en la nube. Este error también puede producirse si hay un problema con la sincronización. Compruebe los registros de sincronización, así como los últimos detalles de ejecución de la sincronización para más información.|
| 33003| ADMutliMatchError| Si una solicitud de restablecimiento o cambio de contraseña se origina desde la nube, usamos el delimitador de la nube especificado durante el proceso de configuración de Azure AD Connect para determinar cómo vincular tal solicitud de nuevo a un usuario en el entorno local. Este evento indica que hemos encontrado dos usuarios en el directorio local con el mismo atributo delimitador de la nube. Compruebe los registros de sincronización, así como los últimos detalles de ejecución de la sincronización para más información.|
| 33004| ADPermissionsError| Este evento indica que la cuenta de servicio del Agente de administración de Active Directory (ADMS) no tiene los permisos adecuados en la cuenta en cuestión para establecer una nueva contraseña. Asegúrese de que la cuenta de ADMA en el bosque del usuario tiene los permisos de restablecimiento de contraseña en todos los objetos del bosque. Para más información sobre cómo establecer los permisos, consulte el paso 4: Configure los permisos adecuados de Active Directory. Este error también puede producirse si el atributo AdminCount del usuario está establecido en 1.|
| 33005| ADUserAccountDisabled| Este evento indica que hemos intentado restablecer o cambiar una contraseña de una cuenta deshabilitada en el entorno local. Habilite la cuenta y vuelva a intentarlo.|
| 33006| ADUserAccountLockedOut| Este evento indica que hemos intentado restablecer o cambiar una contraseña de una cuenta bloqueada en el entorno local. Los bloqueos se aplican cuando un usuario intenta modificar o restablecer la contraseña demasiadas veces en poco tiempo. Desbloquee la cuenta y vuelva a intentarlo.|
| 33007| ADUserIncorrectPassword| Este evento indica que el usuario ha especificado una contraseña incorrecta actual al realizar una operación de cambio de contraseña. Especifique la contraseña actual correcta e inténtelo de nuevo.|
| 33008| ADPasswordPolicyError| Este evento se produce cuando el servicio de escritura diferida de contraseñas intenta establecer una contraseña en su directorio local que no cumple los requisitos del dominio en cuanto a antigüedad, historial, complejidad o filtrado de la contraseña. <br> <br> Si tiene una vigencia mínima de la contraseña y ha cambiado recientemente la contraseña dentro de ese margen de tiempo, no puede volver a cambiarla hasta que alcance la duración especificada en el dominio. Para las pruebas, la vigencia mínima debe establecerse en 0. <br> <br> Si tiene habilitados los requisitos del historial de contraseñas, debe seleccionar una contraseña que no se haya utilizado en las últimas *N* veces, donde *N* es la configuración del historial de contraseñas. Si selecciona una contraseña que se haya usado en las últimas *N* veces, verá un error en este caso. Para las pruebas, el historial mínimo debe establecerse en 0. <br> <br> Si tiene requisitos de complejidad de contraseña, todos ellos se aplican cuando el usuario intenta cambiar o restablecer una contraseña. <br> <br> Si tiene habilitados filtros de contraseña y un usuario selecciona una contraseña que no cumple los criterios de filtrado, se producirá un error en la operación de restablecimiento o modificación.|
| 33009| ADConfigurationError| Este evento indica que se ha producido un problema al escribir la contraseña en diferido en el directorio local por un problema de configuración con Active Directory. Para más información sobre el error producido, compruebe el registro de eventos de la aplicación de la máquina de Azure AD Connect para ver los mensajes del servicio ADSync.|

## <a name="azure-ad-forums"></a>Foros de Azure AD

Si tiene alguna pregunta general sobre Azure AD y el autoservicio de restablecimiento de contraseña, puede pedir ayuda a la comunidad en la [Página de preguntas y respuesta de Microsoft para Azure Active Directory](/answers/topics/azure-active-directory.html). La comunidad está formada por ingenieros, jefes de producto, MVP y profesionales de TI.

## <a name="contact-microsoft-support"></a>Consultar al soporte técnico de Microsoft

Si no encuentra la respuesta a un problema, nuestros equipos de soporte técnico están siempre disponibles para ayudarle.

Para que reciba la ayuda apropiada, le pedimos que proporcione la mayor cantidad de detalles posible al abrir una incidencia. Estos detalles incluyen los siguientes:

* **Descripción general del error**: ¿Cuál es el error? ¿Qué comportamiento observó? ¿Cómo podemos reproducir el error? Proporcione tantos detalles como sea posible.
* **Página**: ¿en qué página estaba cuando se detectó el error? Incluya la dirección URL, si es posible, y una captura de pantalla de la página.
* **Código de soporte técnico**: ¿qué código de soporte técnico se generó cuando el usuario vio el error?
   * Para encontrarlo, reproduzca el error, seleccione el vínculo **Código de soporte técnico** en la parte inferior de la pantalla y envíe al ingeniero de soporte técnico el GUID resultante.

    :::image type="content" source="./media/troubleshoot-sspr-writeback/view-support-code.png" alt-text="Reinicio del servicio de sincronización de Azure AD mediante la GUI":::

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
