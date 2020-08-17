---
title: 'Preguntas frecuentes sobre el autoservicio de restablecimiento de contraseña: Azure Active Directory'
description: Preguntas más frecuentes sobre autoservicio de restablecimiento de contraseña de Azure AD
services: active-directory
ms.service: active-directory
ms.subservice: authentication
ms.topic: how-to
ms.date: 07/20/2020
ms.author: iainfou
author: iainfoulds
manager: daveba
ms.reviewer: rhicock
ms.collection: M365-identity-device-management
ms.openlocfilehash: 2eff16e0d1aa242ed4ca8b427c7af8e8defa7804
ms.sourcegitcommit: 3d79f737ff34708b48dd2ae45100e2516af9ed78
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/23/2020
ms.locfileid: "87051293"
---
# <a name="self-service-password-reset-frequently-asked-questions"></a>Preguntas frecuentes sobre el autoservicio de restablecimiento de contraseña

Estas son algunas preguntas frecuentes sobre todos los aspectos relacionados con el autoservicio de restablecimiento de contraseña.

Si tiene alguna pregunta general sobre Azure Active Directory (Azure AD) y el autoservicio de restablecimiento de contraseña (SSPR) y no encuentra ninguna respuesta aquí, puede pedir ayuda a la comunidad en la[Página de preguntas y respuestas de Microsoft sobre Azure Active Directory](https://docs.microsoft.com/answers/topics/azure-active-directory.html). La comunidad está formada por ingenieros, jefes de producto, MVP y profesionales de TI.

Estas preguntas más frecuentes se dividen en las siguientes secciones:

* [Preguntas sobre el registro de restablecimiento de contraseña](#password-reset-registration)
* [Preguntas sobre el restablecimiento de contraseña](#password-reset)
* [Preguntas sobre el cambio de contraseña](#password-change)
* [Preguntas sobre los informes de la administración de contraseñas](#password-management-reports)
* [Preguntas sobre la escritura diferida de contraseñas](#password-writeback)

## <a name="password-reset-registration"></a>Registro de restablecimiento de contraseña

* **P:  ¿Mis usuarios pueden registrar sus propios datos de restablecimiento de contraseña?**

  > **R:** Sí. Siempre que el restablecimiento de contraseña esté habilitado y los usuarios cuenten con licencia, pueden ir al portal de registro de restablecimiento de contraseña (https://aka.ms/ssprsetup) para registrar su información de autenticación. Los usuarios también pueden registrarse a través del Panel de acceso (https://myapps.microsoft.com). Para ello, deben seleccionar su imagen de perfil y, después, **Perfil** y la opción  **Registrarme para restablecer la contraseña**.
  >
  > Si habilita el [registro combinado](concept-registration-mfa-sspr-combined.md), los usuarios pueden registrarse tanto en SSPR como en Azure Multi-Factor Authentication al mismo tiempo.
* **P:  Si habilito el restablecimiento de contraseña de un grupo y decido habilitarla para todo el mundo, ¿deben volver a registrarse mis usuarios?**

  > **R:** No. No es necesario que los usuarios cuyos datos de autenticación estén rellenados vuelvan a registrarse.
  >
  >
* **P:  ¿Puedo definir los datos de restablecimiento de contraseña en nombre de mis usuarios?**

  > **R:** Sí, puede hacerlo en Azure AD Connect, PowerShell, [Azure Portal](https://portal.azure.com) o el [Centro de administración de Microsoft 365](https://admin.microsoft.com). Para obtener más información, consulte los [Datos usados en el autoservicio de restablecimiento de contraseña de Azure AD](howto-sspr-authenticationdata.md).
  >
  >
* **P:  ¿Puedo sincronizar localmente los datos de las preguntas de seguridad?**

  > **R:** No es posible actualmente.
  >
  >
* **P:  ¿Mis usuarios pueden registrar datos de forma que otros usuarios no puedan verlos?**

  > **R:** Sí. Cuando los usuarios registran datos a través del portal de registro de restablecimiento de contraseña, se guardan en campos de autenticación privados que solo son visibles para los administradores globales y para el propio usuario.
  >
  >
* **P:  ¿Mis usuarios deben registrarse antes de poder utilizar el restablecimiento de contraseña?**

  > **R:** No. Si define información de autenticación suficiente en nombre de sus usuarios, estos no tendrán que registrarse. El restablecimiento de contraseña funciona siempre que tenga los datos con formato correcto almacenados en los campos adecuados del directorio.
  >
  >
* **P:  ¿Puedo sincronizar o definir los campos Teléfono de autenticación, Correo electrónico de autenticación o Teléfono de autenticación alternativo en nombre de mis usuarios?**

  > **R:** Los campos que puede establecer un administrador global se definen en el artículo [Requisitos de datos de SSPR](howto-sspr-authenticationdata.md).
  >
  >
* **P:  ¿Cómo determina el portal de registro cuáles son las opciones que tiene que mostrar a mis usuarios?**

  > **R:** En el portal de registro de restablecimiento de contraseña solo se muestran las opciones habilitadas para los usuarios. Estas opciones se encuentran en la sección **Políticas para restablecer la contraseña del usuario** de la pestaña **Configurar** del directorio. Por ejemplo, si no habilita las preguntas de seguridad, los usuarios no podrán registrarse para obtener esa opción.
  >
  >
* **P:  ¿Cuándo se considera que un usuario está registrado?**

  > **R:** Se considera que un usuario está registrado en SSPR si ha registrado al menos el **número de métodos requeridos para el restablecimiento** de una contraseña establecido en [Azure Portal](https://portal.azure.com).
  >
  >

## <a name="password-reset"></a>Restablecimiento de contraseña

* **P:  ¿Se impide que los usuarios realicen varios intentos para restablecer una contraseña en un breve período de tiempo?**

  > **R:** Sí, existen características de seguridad integradas en el restablecimiento de contraseña a fin de ofrecer protección frente al uso indebido. 
  >
  > Los usuarios solo pueden intentar cinco restablecimientos de contraseña en un período de 24 horas antes de que se les bloquee durante 24 horas. 
  >
  > Los usuarios pueden intentar validar un número de teléfono, enviar un SMS o validar preguntas y respuestas de seguridad solo cinco veces en un período de una hora antes de que se les bloquee durante 24 horas. 
  >
  > Los usuarios pueden enviar un correo electrónico un máximo de 10 veces en un período de 10 minutos antes de que se les bloquee durante 24 horas.
  >
  > Los contadores se restablecen una vez que un usuario restablece su contraseña.
  >
  >
* **P:  ¿Cuánto tiempo debo esperar para recibir un correo electrónico, un SMS o una llamada telefónica para el restablecimiento de contraseña?**

  > **R:** Los mensajes de correo electrónico, los mensajes SMS y las llamadas telefónicas se recibirán en menos de un minuto. Lo más común es que tarden entre 5 y 20 segundos.
  > Si no recibe la notificación en este período de tiempo:
  > * Compruebe la carpeta de correo no deseado.
  > * Compruebe que el número o el correo electrónico de contacto sean los esperados.
  > * Compruebe que los datos de autenticación del directorio tengan el formato correcto, por ejemplo: +1 4255551234 o *usuario\@contoso.com*. 
* **P:  ¿Qué idiomas admite el restablecimiento de contraseña?**

  > **R:** La interfaz de usuario del restablecimiento de contraseña, los mensajes SMS y las llamadas de voz están localizados en los mismos idiomas que admite Office 365.
  >
  >
* **P:  ¿En qué partes de la experiencia de restablecimiento de contraseña aparece mi marca si defino los objetos de personalización de marca de mi organización en la pestaña de configuración del directorio?**

  > **R:** El portal de restablecimiento de contraseña muestra el logotipo de su organización y también le permite configurar el vínculo "Póngase en contacto con el administrador" para dirigirlo a una URL o un correo electrónico personalizado. Todo correo electrónico enviado por el proceso de restablecimiento de contraseña incluye el logotipo, los colores y el nombre de su organización en el cuerpo del correo electrónico y se personaliza a partir de una configuración específica.
  >
  >
* **P:  ¿Cómo puedo informar a mis usuarios sobre dónde tienen que ir para restablecer sus contraseñas?**

  > **R:** Pruebe algunas de las sugerencias de nuestro artículo sobre la [implementación del autoservicio de restablecimiento de contraseña (SSPR)](howto-sspr-deployment.md#plan-communications).
  >
  >
* **P:  ¿Puedo usar esta página desde un dispositivo móvil?**

  > **R:** Sí, esta página funciona en dispositivos móviles.
  >
  >
* **P:  ¿Se admite el desbloqueo de cuentas locales de Active Directory cuando los usuarios restablecen sus contraseñas?**

  > **R:** Sí. Cuando un usuario restablece la contraseña y se ha implementado la escritura diferida de contraseñas mediante Azure AD Connect, esa cuenta de usuario se desbloquea automáticamente al restablecer la contraseña.
  >
  >
* **P:  ¿Cómo puedo integrar directamente el restablecimiento de contraseña en la experiencia de inicio de sesión de escritorio de mi usuario?**

  > **R:** Si es cliente de Azure AD Premium, puede instalar Microsoft Identity Manager sin costo adicional e implementar la solución de restablecimiento de contraseña local.
  >
  >
* **P:  ¿Puedo establecer distintas preguntas de seguridad para diferentes configuraciones regionales?**

  > **R:** No es posible actualmente.
  >
  >
* **P:  ¿Cuántas preguntas se pueden configurar en la opción de autenticación de preguntas de seguridad?**

  > **R:** Puede configurar hasta 20 preguntas de seguridad personalizadas en [Azure Portal](https://portal.azure.com).
  >
  >
* **P:  ¿Qué longitud pueden tener las preguntas de seguridad?**

  > **R:** Las preguntas de seguridad pueden tener entre 3 y 200 caracteres.
  >
  >
* **P:  ¿Qué longitud pueden tener las respuestas a las preguntas de seguridad?**

  > **R:** Las respuestas pueden tener entre 3 y 40 caracteres.
  >
  >
* **P:  ¿Se rechazarán las respuestas duplicadas a las preguntas de seguridad?**

  > **R:** Sí, rechazaremos las respuestas duplicadas a las preguntas de seguridad.
  >
  >
* **P:  ¿Puede un usuario registrar la misma pregunta de seguridad más de una vez?**

  > **R:** No. Después de que un usuario registre una pregunta específica, no podrá volver a registrarla.
  >
  >
* **P:  ¿Es posible establecer un límite mínimo de preguntas de seguridad para registro y restablecimiento?**

  > **R:** Sí, es posible establecer un límite para el registro y otro para el restablecimiento. Se pueden requerir entre tres y cinco preguntas de seguridad tanto para el registro como para el restablecimiento.
  >
  >
* **P:  He configurado mi directiva para que requiera que los usuarios utilicen preguntas de seguridad para realizar el restablecimiento, pero parece que los administradores de Azure están configurados de otra forma.**

  > **R:** Este es un comportamiento esperado. Microsoft exige una directiva segura de restablecimiento de contraseña de dos puertas de forma predeterminada para cualquier rol de administrador de Azure. De este modo se evita que los administradores usen preguntas de seguridad. Para obtener más información sobre esta directiva, vea el artículo [Restricciones y directivas de contraseñas en Azure Active Directory](concept-sspr-policy.md).
  >
  >
* **P:  Si un usuario ha registrado más del número máximo de preguntas necesarias para el restablecimiento, ¿cómo se seleccionan las preguntas de seguridad durante el proceso de restablecimiento?**

  > **R:** De manera aleatoria se seleccionan *N* preguntas de seguridad del número total de preguntas para las cuales se ha registrado un usuario, donde *N* es la cantidad establecida para la opción **Número de preguntas necesarias para el restablecimiento**. Por ejemplo, si un usuario ha registrado cinco preguntas de seguridad pero solo se requieren tres para restablecer una contraseña, se seleccionarán aleatoriamente tres de las cinco preguntas para presentarlas tras el restablecimiento. Si el usuario se equivoca al responder las preguntas, el proceso de selección volverá a ejecutarse para evitar la repetición (hammering) de las preguntas.
  >
  >
* **P:  ¿Durante cuánto tiempo es válido el código de acceso de un solo uso que se recibe por correo electrónico o mensaje SMS?**

  > **R:** La duración de la sesión de restablecimiento de contraseña es de 15 minutos. Desde el inicio de la operación de restablecimiento de contraseña, el usuario tiene 15 minutos para restablecer la contraseña. Los códigos de acceso de un solo uso de SMS y correo electrónico son válidos por 5 minutos durante la sesión de restablecimiento de contraseña.
  >
  >
* **P:  ¿Puedo impedir que los usuarios restablezcan su contraseña?**

  > **R:** Sí, si usa un grupo para habilitar SSPR, puede quitar a un usuario individual del grupo que permite a los usuarios restablecer su contraseña. Si el usuario es un administrador global, conservará la capacidad de restablecer la contraseña (esta opción no se puede deshabilitar).
  >
  >

## <a name="password-change"></a>Cambio de contraseña

* **P:  ¿Dónde deberían ir los usuarios para cambiar las contraseñas?**

  > **R:** Los usuarios pueden cambiar sus contraseñas en cualquier lugar en que vean su icono o imagen de perfil, como en la esquina superior derecha de las experiencias del portal de [Office 365](https://portal.office.com) o el [Panel de acceso](https://myapps.microsoft.com). Los usuarios pueden cambiar las contraseñas en la [página de perfil del Panel de acceso](https://account.activedirectory.windowsazure.com/r#/profile). Los usuarios también deben cambiar automáticamente sus contraseñas en la página de inicio de sesión de Azure AD en caso de que hayan expirado. Por último, los usuarios pueden navegar directamente al [portal de cambio de contraseñas de Azure AD](https://account.activedirectory.windowsazure.com/ChangePassword.aspx) si quieren cambiar las contraseñas.
  >
  >
* **P:  ¿Los usuarios pueden recibir una notificación en el portal de Office cuando expira la contraseña local?**

  > **R:** Sí, es posible si usa los Servicios de federación de Active Directory (AD FS). Si usa AD FS, siga las instrucciones que se muestran en el artículo en el que se detalla el [envío de notificaciones de directivas de contraseña con AD FS](https://technet.microsoft.com/windows-server-docs/identity/ad-fs/operations/configure-ad-fs-to-send-password-expiry-claims?f=255&MSPPError=-2147217396). No es posible si usa la sincronización de hash de contraseña. No sincronizamos las directivas de contraseña de directorios locales, por lo que no es posible publicar notificaciones de expiración en las experiencias en la nube. En cualquier caso, también es posible [notificar a los usuarios cuyas contraseñas están a punto de expirar a través de PowerShell](https://social.technet.microsoft.com/wiki/contents/articles/23313.notify-active-directory-users-about-password-expiry-using-powershell.aspx).
  >
  >
* **P:  ¿Puedo impedir que los usuarios cambien su contraseña?**

  > **R:** No se pueden bloquear los cambios de contraseña de usuarios que estén solo en la nube. En el caso de los usuarios locales, puede seleccionar la opción **El usuario no puede cambiar la contraseña**. Los usuarios seleccionados no podrán cambiar su contraseña.
  >
  >

## <a name="password-management-reports"></a>Informes de administración de contraseñas

* **P:  ¿Cuánto tiempo demoran en aparecer los datos en los informes de la administración de contraseñas?**

  > **R:** Los datos se mostrarán en los informes de administración de contraseñas en un plazo de entre cinco y diez minutos. En algunas instancias, es posible que tarden hasta una hora.
  >
  >
* **P:  ¿Cómo puedo filtrar los informes de administración de contraseñas?**

  > **R:** Para filtrar los informes de administración de contraseñas, seleccione la lupa pequeña que se muestra en el extremo derecho de las etiquetas de columna, cerca de la parte superior del informe. Si quiere realizar un filtrado más completo, puede descargar el informe a Excel y crear una tabla dinámica.
  >
  >
* **P: ¿Cuál es el número máximo de eventos que se almacenan en los informes de administración de contraseñas?**

  > **R:** Hasta 75 000 eventos de restablecimiento de contraseña o de registro de restablecimiento de contraseña se almacenan en los informes de administración de contraseñas, los que abarcan los últimos 30 días. Estamos trabajando para ampliar este número e incluir más eventos.
  >
  >
* **P:  ¿Hasta cuándo se remontan los informes de administración de contraseñas?**

  > **R:** Los informes de administración de contraseñas muestran las operaciones generadas durante los últimos 30 días. Por ahora, si desea archivar estos datos, puede descargar periódicamente los informes y guardarlos en una ubicación independiente.
  >
  >
* **P:  ¿Existe un número máximo de filas que pueden aparecer en los informes de administración de contraseñas?**

  > **R:** Sí. Pueden mostrarse un máximo de 75 000 filas en cualquiera de los informes de administración de contraseñas, ya sea que se muestren en la interfaz de usuario o se descarguen.
  >
  >
* **P:  ¿Existe una API para acceder a los datos de informes de registro o de restablecimiento de contraseña?**

  > **R:** Sí. Para saber cómo puede acceder a los datos de informes de restablecimiento de contraseña, consulte la [referencia de API REST de Azure Log Analytics](/rest/api/loganalytics/).
  >
  >

## <a name="password-writeback"></a>escritura diferida de contraseñas

* **P:  ¿Cómo funciona en segundo plano la escritura diferida de contraseñas?**

  > **R:** Vea el artículo [Funcionamiento de la escritura diferida de contraseñas](howto-sspr-writeback.md) para obtener una explicación de lo que ocurre cuando se habilita la escritura diferida de contraseñas, además de cómo fluyen los datos por el sistema para volver al entorno local.
  >
  >
* **P:  ¿Cuánto tarda en funcionar la escritura diferida de contraseñas? ¿Existe un retraso en la sincronización como ocurre con la sincronización de hash de contraseñas?**

  > **R:** El servicio de Escritura diferida de contraseñas es inmediato. Se trata de una canalización sincrónica que funciona radicalmente distinto a la sincronización de hash de contraseña. La escritura diferida de contraseñas permite que los usuarios obtengan comentarios en tiempo real sobre el éxito de su operación de cambio o restablecimiento de contraseña. El tiempo promedio para la escritura diferida correcta de una contraseña es de menos de 500 ms.
  >
  >
* **P:  Si mi cuenta local está deshabilitada, ¿en qué afecta esto a mi acceso y mi cuenta en la nube?**

  > **R:** Si el identificador local está deshabilitado, el acceso y el identificador de la nube también se deshabilitarán durante el próximo intervalo de sincronización a través de Azure AD Connect. De forma predeterminada, la sincronización se realiza cada 30 minutos.
  >
  >
* **P:  Si mi cuenta local está restringida por una directiva de contraseñas de Active Directory local, ¿SSPR se atiene a esta directiva si cambio mi contraseña?**

  > **R:** Sí, SSPR se basa en la directiva de contraseñas de Active Directory local y se rige por esta. Esta directiva incluye la directiva de contraseñas de dominio de Active Directory estándar, así como las directivas de contraseñas definidas específicas que se aplican a un usuario.
  >
  >
* **P:  ¿Para qué tipos de cuentas funciona la escritura diferida de contraseñas?**

  > **R:** La escritura diferida de contraseñas funciona con cuentas de usuario que están sincronizadas entre Active Directory local y Azure AD, lo que incluye hash de contraseña federado sincronizado y usuarios de autenticación de paso a través.
  >
  >
* **P:  ¿La escritura diferida de contraseñas aplica las directivas de contraseñas de mi dominio?**

  > **R:** Sí. La escritura diferida de contraseñas aplica la vigencia, el historial, la complejidad y los filtros de contraseñas, además de cualquier otra restricción que pueda aplicar sobre las contraseñas en su dominio local.
  >
  >
* **P:  ¿Es segura la escritura diferida de contraseñas?  ¿Cómo puedo estar seguro de no ser víctima del ataque de un hacker?**

  > **R:** Sí, la escritura diferida de contraseñas es segura. Para más información sobre los distintos niveles de seguridad que implementa el servicio de escritura diferida de contraseñas, consulte la sección [Seguridad de la escritura diferida de contraseñas](concept-sspr-writeback.md#password-writeback-security) del artículo [Información general sobre la escritura diferida de contraseñas](howto-sspr-writeback.md).
  >
  >

## <a name="next-steps"></a>Pasos siguientes

* [¿Cómo se realiza un lanzamiento correcto de SSPR?](howto-sspr-deployment.md)
* [Restablecimiento o modificación de la contraseña](../user-help/active-directory-passwords-update-your-own-password.md)
* [Registro para el autoservicio de restablecimiento de contraseñas](../user-help/active-directory-passwords-reset-register.md)
* [¿Tiene alguna pregunta acerca de las licencias?](concept-sspr-licensing.md)
* [¿Qué datos usa SSPR y cuáles se deben rellenar en lugar de los usuarios?](howto-sspr-authenticationdata.md)
* [¿Qué métodos de autenticación están disponibles para los usuarios?](concept-sspr-howitworks.md#authentication-methods)
* [¿Cuáles son las opciones de directiva con SSPR?](concept-sspr-policy.md)
* [¿Qué es la escritura diferida de contraseñas y por qué nos interesa?](howto-sspr-writeback.md)
* [¿Cómo se informa sobre la actividad de SSPR?](howto-sspr-reporting.md)
* [¿Cuáles son todas las opciones en SSPR y qué significan?](concept-sspr-howitworks.md)
* [Creo que algo se ha roto. ¿Cómo se solucionan problemas en SSPR?](active-directory-passwords-troubleshoot.md)
