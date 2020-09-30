---
title: Solución de problemas de inicio de sesión único basado en contraseña en Azure AD
description: Cómo solucionar incidencias relacionadas con aplicaciones de Azure AD configuradas para inicio de sesión único basado en contraseña.
author: kenwith
manager: celestedg
ms.service: active-directory
ms.subservice: app-mgmt
ms.workload: identity
ms.topic: troubleshooting
ms.date: 07/11/2017
ms.author: kenwith
ms.reviewer: asteen
ms.openlocfilehash: 436fbdde1283bd8406e631fd37b559bfe11e234e
ms.sourcegitcommit: 8a7b82de18d8cba5c2cec078bc921da783a4710e
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/28/2020
ms.locfileid: "89051883"
---
# <a name="troubleshoot-password-based-single-sign-on-in-azure-ad"></a>Solución de problemas de inicio de sesión único basado en contraseña en Azure AD

Para usar el inicio de sesión único (SSO) basado en contraseña en Aplicaciones, debe instalarse la extensión del explorador. Esta extensión se descarga automáticamente cuando se selecciona una aplicación que está configurada para el inicio de sesión único basado en contraseña. Para obtener información sobre el uso de Aplicaciones desde la perspectiva del usuario final, consulte la [ayuda del portal Aplicaciones](../user-help/my-apps-portal-end-user-access.md).

## <a name="my-apps-browser-extension-not-installed"></a>Extensión del explorador Aplicaciones no instalada
Asegúrese de que la extensión del explorador está instalada. Para obtener más información, consulte [Planeamiento de la implementación de Aplicaciones de Azure Active Directory](access-panel-deployment-plan.md). 

## <a name="single-sign-on-not-configured"></a>Inicio de sesión único no configurado
Asegúrese de que el inicio de sesión único basado en contraseña está configurado. Para obtener más información, consulte [Configuración del inicio de sesión único basado en contraseña](configure-password-single-sign-on-non-gallery-applications.md).

## <a name="users-not-assigned"></a>Usuarios no asignados
Asegúrese de que el usuario está asignado a la aplicación. Para obtener más información, consulte [Asignar un usuario o grupo a una aplicación](assign-user-or-group-access-portal.md).

## <a name="credentials-are-filled-in-but-the-extension-does-not-submit-them"></a>Se rellenan las credenciales, pero la extensión no las envía

En general, este problema sucede si el proveedor de la aplicación ha cambiado recientemente la página de inicio de sesión para agregar un campo, cambiar un identificador que se utiliza para detectar los campos de nombre de usuario y contraseña, o modificar cómo funciona la experiencia de inicio de sesión para su aplicación. Afortunadamente, en muchos casos, Microsoft puede trabajar con los proveedores de aplicaciones para resolver rápidamente estos problemas.

Aunque Microsoft tiene tecnologías para detectar automáticamente cuándo se interrumpen estas integraciones, puede que no sea posible encontrar estos problemas de forma inmediata o que se tarde algún tiempo en corregirlos. En el caso de que una de estas integraciones no funcione correctamente, abra un caso de soporte técnico para que podamos solucionar el problema lo más rápido posible.

**Si está en contacto con el proveedor de la aplicación,** pídale que se comunique con Microsoft para que podamos trabajar juntos con el fin de integrar de forma nativa su aplicación con Azure Active Directory. Para empezar puede enviar al proveedor a [Anuncio de la aplicación en la galería de aplicaciones de Azure Active Directory](../azuread-dev/howto-app-gallery-listing.md).

## <a name="credentials-are-filled-in-and-submitted-but-the-page-indicates-the-credentials-are-incorrect"></a>Las credenciales se rellenan y se envían, pero la página indica que son incorrectas

Para resolver este problema, intente primero estas cosas:

- Pídale al usuario que intente primero **iniciar sesión en el sitio web de la aplicación directamente** con las credenciales almacenadas para dicho sitio.

  * Si el inicio de sesión funciona, a continuación, indíquele al usuario que haga clic en el botón **Actualizar credenciales** situado en el **icono de la aplicación** en la sección de **aplicaciones** de [Aplicaciones](https://myapps.microsoft.com/) para actualizarlas con el nombre de usuario y la contraseña más recientes.

  * Si usted u otro administrador ha asignado las credenciales para este usuario, busque la asignación de aplicación del usuario o del grupo yendo a la pestaña **Usuarios y grupos** de la aplicación, seleccione la asignación y haga clic en el botón **Actualizar credenciales**.

- Si el usuario se asignó sus propias credenciales, pídale al usuario **que compruebe si su contraseña ha expirado en la aplicación** y si es así, **que actualice su contraseña expirada** iniciando sesión directamente en la aplicación.

  * Después de actualizar la contraseña en la aplicación, indíquele al usuario que haga clic en el botón **Actualizar credenciales** situado en el **icono de la aplicación** en la sección de **aplicaciones** de [Aplicaciones](https://myapps.microsoft.com/) para actualizarlas con el nombre de usuario y la contraseña más recientes.

  * Si usted u otro administrador ha asignado las credenciales para este usuario, busque la asignación de aplicación del usuario o del grupo yendo a la pestaña **Usuarios y grupos** de la aplicación, seleccione la asignación y haga clic en el botón **Actualizar credenciales**.

- Compruebe que la extensión de explorador Aplicaciones está habilitada y se está ejecutando en el explorador del usuario.

- Asegúrese de que los usuarios no intentan iniciar sesión en la aplicación desde Aplicaciones en los **modos incógnito, inPrivate o privado**. La extensión Aplicaciones no es compatible con estos modos.

Si las sugerencias anteriores no funcionan, puede que se haya producido un cambio en el lado de la aplicación que haya interrumpido provisionalmente la integración de la aplicación con Azure AD. Por ejemplo, esto puede ocurrir cuando el proveedor de la aplicación introduce un script en su página que tiene un comportamiento diferentes con las entradas manuales y con las automatizadas, lo que hace que una integración automatizada, como la nuestra, se interrumpa. Afortunadamente, en muchos casos, Microsoft puede trabajar con los proveedores de aplicaciones para resolver rápidamente estos problemas.

Aunque Microsoft tiene tecnologías para detectar automáticamente cuándo se interrumpen las integraciones de aplicaciones, puede que no sea posible encontrar estos problemas de forma inmediata o que se tarde algún tiempo en corregirlos. Cuando una integración no funciona correctamente, puede abrir un caso de soporte técnico para que podamos corregirlo lo antes posible. 

Además, **si está en contacto con el proveedor de la aplicación,** **pídale que cuente con nosotros** para que podamos trabajar juntos con el fin de integrar de forma nativa su aplicación con Azure Active Directory. Para empezar puede enviar al proveedor a [Anuncio de la aplicación en la galería de aplicaciones de Azure Active Directory](../azuread-dev/howto-app-gallery-listing.md).

## <a name="check-if-the-applications-login-page-has-changed-recently-or-requires-an-additional-field"></a>Compruebe si la página de inicio de sesión de la aplicación ha cambiado recientemente, o si requiere un campo adicional

Si la página de inicio de sesión de la aplicación ha cambiado de forma considerable, esto hace que a veces nuestras integraciones se interrumpan. Un ejemplo de esto es cuando un proveedor de aplicación agrega un campo de inicio de sesión, un captcha o una autenticación multifactor a sus experiencias. Afortunadamente, en muchos casos, Microsoft puede trabajar con los proveedores de aplicaciones para resolver rápidamente estos problemas.

Aunque Microsoft tiene tecnologías para detectar automáticamente cuándo se interrumpen las integraciones de aplicaciones, puede que no sea posible encontrar estos problemas de forma inmediata o que se tarde algún tiempo en corregirlos. Cuando una integración no funciona correctamente, puede abrir un caso de soporte técnico para que podamos corregirlo lo antes posible. 

Además, **si está en contacto con el proveedor de la aplicación,** **pídale que cuente con nosotros** para que podamos trabajar juntos con el fin de integrar de forma nativa su aplicación con Azure Active Directory. Para empezar puede enviar al proveedor a [Anuncio de la aplicación en la galería de aplicaciones de Azure Active Directory](../azuread-dev/howto-app-gallery-listing.md).

## <a name="capture-sign-in-fields-for-an-app"></a>Capturar los campos de inicio de sesión para una aplicación

La captura de campos de inicio de sesión solo se admite para páginas de inicio de sesión habilitadas para HTML. No se admite no en las páginas de inicio de sesión no estándar, como las que usan Adobe Flash u otras tecnologías que no compatibles con HTML.

Existen dos maneras de capturar campos de inicio de sesión para aplicaciones personalizadas:

- La **captura automática de campos de inicio de sesión** funciona bien con la mayoría de las páginas de inicio de sesión compatibles con HTML si usan *identificadores DIV conocidos* para los campos de nombre de usuario y contraseña. El código HTML en la página se extrae para encontrar identificadores DIV que cumplen determinados criterios. Esos metadatos se guardan para que puedan reproducirse en la aplicación más adelante.

- La **captura manual de campos de inicio de sesión** se usa si el proveedor de la aplicación *no etiqueta los campos de entrada de inicio de sesión*. La captura manual también se usa si el proveedor *presenta varios campos que no se pueden detectar automáticamente*. Azure Active Directory (Azure AD) puede almacenar datos de tantos campos como haya en la página de inicio de sesión, si le indica dónde se encuentran esos campos en la página.

Por lo general, si la captura automática de campos de inicio de sesión no funciona, pruebe la opción manual.

### <a name="automatically-capture-sign-in-fields-for-an-app"></a>Capturar automáticamente campos de inicio de sesión para una aplicación

Para configurar el SSO basado en contraseña con la captura automática de campos de inicio de sesión, siga estos pasos:
1. Abra [Azure Portal](https://portal.azure.com/). Inicie sesión como administrador global o coadministrador.
2. En el panel de navegación del lado izquierdo, seleccione **Todos los servicios** para abrir la extensión de Azure AD.
3. Escriba **Azure Active Directory** en el cuadro de búsqueda de filtrado y, luego, seleccione **Azure Active Directory**.
4. En el panel de navegación de Azure AD, seleccione **Aplicaciones empresariales**.
5. Seleccione **Todas las aplicaciones** para ver una lista de todas las aplicaciones.
   > [!NOTE]
   > Si no ve la aplicación que busca, use el control **Filtro** en la parte superior de la lista **Todas las aplicaciones**. Establezca la opción **Mostrar** en "Todas las aplicaciones".
6. Seleccione la aplicación para la que quiere configurar el SSO.
7. Después de que la aplicación se carga, seleccione **Inicio de sesión único** en el panel de navegación de la izquierda.
8. Seleccione el modo **Inicio de sesión con contraseña**.
9. Escriba la **URL de inicio de sesión**, que es la dirección URL de la página donde los usuarios escriben su nombre de usuario y contraseña para iniciar sesión. *Asegúrese de que los campos de inicio de sesión estén visibles en la página de la dirección URL que proporcione*.
10. Seleccione **Guardar**.
    La página se extrae automáticamente para los cuadros de entrada de nombre de usuario y contraseña. Ahora puede usar Azure AD para transmitir de forma segura las contraseñas a esa aplicación mediante la extensión de explorador Aplicaciones.

### <a name="manually-capture-sign-in-fields-for-an-app"></a>Capturar manualmente campos de inicio de sesión para una aplicación

Para capturar manualmente los campos de inicio de sesión, debe tener instalada la extensión del explorador Aplicaciones. Además, el explorador no puede estar en ejecución en modo *inPrivate*, *incógnito* ni *privado*.

Para configurar el SSO basado en contraseña para una aplicación con la captura manual de campos de inicio de sesión, siga estos pasos:
1. Abra [Azure Portal](https://portal.azure.com/). Inicie sesión como administrador global o coadministrador.
2. En el panel de navegación del lado izquierdo, seleccione **Todos los servicios** para abrir la extensión de Azure AD.
3. Escriba **Azure Active Directory** en el cuadro de búsqueda de filtrado y, luego, seleccione **Azure Active Directory**.
4. En el panel de navegación de Azure AD, seleccione **Aplicaciones empresariales**.
5. Seleccione **Todas las aplicaciones** para ver una lista de todas las aplicaciones.
   > [!NOTE] 
   > Si no ve la aplicación que busca, use el control **Filtro** en la parte superior de la lista **Todas las aplicaciones**. Establezca la opción **Mostrar** en "Todas las aplicaciones".
6. Seleccione la aplicación para la que quiere configurar el SSO.
7. Después de que la aplicación se carga, seleccione **Inicio de sesión único** en el panel de navegación de la izquierda.
8. Seleccione el modo **Inicio de sesión con contraseña**.
9. Escriba la **URL de inicio de sesión**, que es la página donde los usuarios escriben su nombre de usuario y contraseña para iniciar sesión. *Asegúrese de que los campos de inicio de sesión estén visibles en la página de la dirección URL que proporcione*.
10. Seleccione **Establecer configuración de inicio de sesión único con contraseña de *&lt;nombre de la aplicación&gt;*** .
11. Seleccione **Detectar campos de inicio de sesión manualmente**.
14. Seleccione **Aceptar**.
15. Seleccione **Guardar**.
16. Siga las instrucciones para usar Aplicaciones.


## <a name="troubleshoot-problems"></a>Solucionar problemas

### <a name="i-get-a-we-couldnt-find-any-sign-in-fields-at-that-url-error"></a>Recibo el error "No pudimos encontrar ningún campo de inicio de sesión en esa URL"

Recibe este error cuando no funciona la detección automática de campos de inicio de sesión. Para resolver el problema, pruebe la detección manual de campos de inicio de sesión. Consulte la sección [Captura manual de campos de inicio de sesión para una aplicación](#manually-capture-sign-in-fields-for-an-app) de este artículo.

### <a name="i-get-an-unable-to-save-single-sign-on-configuration-error"></a>Recibo un error "No se puede guardar la configuración del inicio de sesión único"

En contadas ocasiones, no es posible actualizar la configuración de SSO. Para resolver este problema, intente guardar la configuración de nuevo.

Si sigue apareciendo el error, abra una incidencia de soporte técnico. Incluya la información que se describe en las secciones [Ver los detalles de notificaciones del portal](#view-portal-notification-details) y [Enviar los detalles de notificaciones a un ingeniero de soporte técnico para obtener ayuda](#send-notification-details-to-a-support-engineer-to-get-help) de este artículo.

### <a name="i-cant-manually-detect-sign-in-fields-for-my-app"></a>No puedo detectar manualmente los campos de inicio de sesión para mi aplicación

Puede que observe los comportamientos siguientes cuando no funcione la detección manual:
- El proceso de captura manual parece haber funcionado, pero los campos capturados no son correctos.
- No se resaltan los campos correctos al ejecutar el proceso de captura.
- El proceso de captura lo dirige a la página de inicio de sesión de la aplicación según lo previsto, pero no ocurre nada.
- La captura manual parece funcionar, pero el SSO no se realiza cuando los usuarios acceden a la aplicación desde Aplicaciones.

Si experimenta alguno de estos problemas, haga lo siguiente:
- Asegúrese de tener *instalada y habilitada* la versión más reciente de la extensión de explorador Aplicaciones.
- Asegúrese de que el explorador no se encuentre en los modos *InPrivate*, *incógnito* ni *privado* durante el proceso de captura. La extensión Aplicaciones no es compatible con estos modos.
- Asegúrese de que los usuarios no intentan iniciar sesión en la aplicación desde Aplicaciones en los *modos incógnito*, *inPrivate* ni *privado*.
- Vuelva a intentar el proceso de captura manual. Asegúrese de que los marcadores rojos se encuentren sobre los campos correctos.
- Si el proceso de captura manual parece no responder o la página de inicio de sesión no responde, intente realizar el proceso de captura manual de nuevo. Sin embargo, esta vez, tras completar el proceso, presione la tecla F12 para abrir la consola del desarrollador del explorador. Seleccione la pestaña **Consola**. Escriba **window.location=" *&lt;escriba la dirección URL de inicio de sesión especificada al configurar la aplicación&gt;* "** y, luego, presione Entrar. De este modo, se forzará una redirección de página que finalizará el proceso de captura y almacenará los campos capturados.

## <a name="request-support"></a>Solicitud de soporte técnico 
Si recibe un mensaje de error al configurar el inicio de sesión único y asignar usuarios, abra una incidencia de soporte técnico. De la siguiente información, incluya tanta como sea posible:

-   Id. de error de correlación
-   UPN (dirección de correo electrónico del usuario)
-   TenantID
-   Tipo de explorador
-   Zona horaria y hora o período de tiempo en el que se ha producido el error
-   Seguimientos de Fiddler

### <a name="view-portal-notification-details"></a>Ver los detalles de notificaciones del portal

Para ver los detalles de cualquier notificación del portal, siga estos pasos:
1. Seleccione el icono **Notificaciones** (con forma de campana) de la esquina superior derecha de Azure Portal.
2. Seleccione una notificación que tenga un estado de *Error*. (Tienen un "!" rojo).
   > [!NOTE]
   > No se puede seleccionar notificaciones con estado *Correcto* o *En curso*.
3. Se abre el panel **Detalles de la notificación**. Lea la información para conocer más sobre el problema.
5. Si aún necesita ayuda, comparta la información con un ingeniero de soporte técnico o el grupo de producto. Seleccione el icono de **copia** a la derecha del cuadro **Copiar error** para copiar los detalles de notificación para compartirlos.

### <a name="send-notification-details-to-a-support-engineer-to-get-help"></a>Enviar los detalles de notificaciones a un ingeniero de soporte técnico para obtener ayuda

Es importante que comparta con el soporte técnico *todos* los detalles que aparecen en esta sección para que pueden ayudarle rápidamente. Para registrarlos, puede tomar una captura de pantalla o seleccionar **Copiar error**.

En la siguiente información se explica qué significa cada elemento de la notificación y se proporcionan ejemplos.

#### <a name="essential-notification-items"></a>Elementos esenciales de una notificación

- **Título**: El título descriptivo de la notificación.

   Ejemplo: *Configuración del proxy de aplicación*

- **Descripción**: Qué ha ocurrido como consecuencia de la operación.

   Ejemplo: *La dirección URL interna especificada ya se está usando en otra aplicación.*

- **Identificador de notificación**: El identificador único de la notificación.

    Ejemplo: *clientNotification-2adbfc06-2073-4678-a69f-7eb78d96b068*

- **Identificador de solicitud de cliente**: El identificador de solicitud específico generado por el explorador.

    Ejemplo: *302fd775-3329-4670-a9f3-bea37004f0bc*

- **Marca de tiempo UTC**: La marca de tiempo de cuándo tuvo lugar la notificación, en formato UTC.

    Ejemplo: *2017-03-23T19:50:43.7583681Z*

- **Identificador de transacción interno**: El identificador interno que se ha usado para buscar el error en nuestros sistemas.

    Ejemplo: **71a2f329-ca29-402f-aa72-bc00a7aca603**

- **UPN**: El usuario que ejecutó la operación.

    Ejemplo: *tperkins\@f128.info*

- **Identificador de inquilino**: El identificador único del inquilino del que forma parte el usuario que ejecutó la operación.

    Ejemplo: *7918d4b5-0442-4a97-be2d-36f9f9962ece*

- **Id. de objeto de usuario**: El identificador único del usuario que ejecutó la operación.

    Ejemplo: *17f84be4-51f8-483a-b533-383791227a99*

#### <a name="detailed-notification-items"></a>Elementos detallados de una notificación

- **Nombre para mostrar**: (Puede estar vacío) un nombre para mostrar más detallado del error.

    Ejemplo: *Configuración del proxy de aplicación*

- **Estado**: El estado específico de la notificación.

    Ejemplo: *Erróneo*

- **Identificador de objeto**: (Puede estar vacío) el identificador del objeto en el que se ejecutó la operación.

   Ejemplo: *8e08161d-f2fd-40ad-a34a-a9632d6bb599*

- **Detalles**: La descripción detallada de lo que se produjo como consecuencia de la operación.

    Ejemplo: *La dirección url interna "<https://bing.com/>" no es válida puesto que ya está en uso.*

- **Copiar error**: Le permite seleccionar el **icono de copia** a la derecha del cuadro **Copiar error** para copiar los detalles de la notificación para compartirlos con soporte técnico.

    Ejemplo:   ```{"errorCode":"InternalUrl\_Duplicate","localizedErrorDetails":{"errorDetail":"Internal url 'https://google.com/' is invalid since it is already in use"},"operationResults":\[{"objectId":null,"displayName":null,"status":0,"details":"Internal url 'https://bing.com/' is invalid since it is already in use"}\],"timeStampUtc":"2017-03-23T19:50:26.465743Z","clientRequestId":"302fd775-3329-4670-a9f3-bea37004f0bb","internalTransactionId":"ea5b5475-03b9-4f08-8e95-bbb11289ab65","upn":"tperkins@f128.info","tenantId":"7918d4b5-0442-4a97-be2d-36f9f9962ece","userObjectId":"17f84be4-51f8-483a-b533-383791227a99"}```


## <a name="next-steps"></a>Pasos siguientes
* [Serie de guías de inicio rápido sobre la administración de aplicaciones](view-applications-portal.md)
* [Planeamiento de una implementación de Aplicaciones](access-panel-deployment-plan.md)
