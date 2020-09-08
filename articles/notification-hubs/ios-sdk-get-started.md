---
title: Envío de notificaciones push a aplicaciones iOS mediante Azure Notification Hubs y el SDK de iOS
description: En este tutorial, aprenderá a usar Azure Notification Hubs y Apple Push Notification Service para enviar notificaciones push a dispositivos iOS.
author: sethmanheim
ms.author: sethm
ms.date: 08/10/2020
ms.topic: tutorial
ms.service: notification-hubs
ms.reviewer: thsomasu
ms.lastreviewed: 06/01/2020
ms.openlocfilehash: 7cdf095898bfe85e6f3b14fa1dcdb7b0c94ccde6
ms.sourcegitcommit: 1a0dfa54116aa036af86bd95dcf322307cfb3f83
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/10/2020
ms.locfileid: "88042452"
---
# <a name="tutorial-send-push-notifications-to-ios-apps-using-azure-notification-hubs"></a>Tutorial: Envío de notificaciones push a aplicaciones iOS mediante Azure Notification Hubs

En este tutorial se muestra cómo configurar Azure Notification Hubs y cómo configurar las credenciales para enviar notificaciones push a un dispositivo iOS mediante [Apple Push Notification Service (APNs)](https://developer.apple.com/library/content/documentation/NetworkingInternet/Conceptual/RemoteNotificationsPG/APNSOverview.html#//apple_ref/doc/uid/TP40008194-CH8-SW1). 

La realización de este tutorial es un requisito previo para los siguientes tutoriales de aplicaciones iOS basadas en Objective-C y Swift. En él se desarrollan los pasos siguientes:

- Generar el archivo de solicitud de firma de certificado.
- Solicitar notificaciones push a la aplicación.
- Crear un perfil de aprovisionamiento para la aplicación.
- Crear un Centro de notificaciones
- Configurar el centro de notificaciones con la información de APNs.

## <a name="prerequisites"></a>Requisitos previos

Para completar este tutorial, deberá tener una cuenta de Azure activa. En caso de no tener cuenta, puede crear una de evaluación gratuita en tan solo unos minutos. Para obtener más información, consulte [Evaluación gratuita de Azure](https://azure.microsoft.com/free/).

También necesitará los siguientes elementos:

- Una cuenta de [desarrollador de Apple](https://developer.apple.com/) activa.
- Mac con [Xcode](https://go.microsoft.com/fwLink/p/?LinkID=266532), junto con un certificado de desarrollador válido instalado en el Llavero.
- Un iPhone o iPad con iOS versión 10 o posterior.
- El dispositivo físico registrado en el [portal de Apple](https://developer.apple.com/) y asociado con el certificado.

Asegúrese de leer la [introducción a Azure Notification Hubs](notification-hubs-push-notification-overview.md) si no está familiarizado con el servicio.

> [!NOTE]
> El centro de notificaciones se configurará para usar el modo de autenticación de espacio aislado únicamente. No debe usar este modo de autenticación para las cargas de trabajo de producción.

## <a name="generate-the-certificate-signing-request-file"></a>Generación del archivo de solicitud de firma de certificado

El Servicio de notificaciones push de Apple (APNS) usa certificados para autenticar las notificaciones push. Siga estas instrucciones para crear el certificado de inserción necesario para enviar y recibir notificaciones. Para obtener más información sobre estos conceptos, consulte la documentación oficial de [Apple Push Notification Service](https://developer.apple.com/library/archive/documentation/NetworkingInternet/Conceptual/RemoteNotificationsPG/APNSOverview.html).

Genere el archivo de solicitud de firma de certificado (CSR) que utiliza Apple para generar un certificado push firmado:

1. En el Mac, ejecute la herramienta Acceso a llaves. Se puede activar desde la carpeta **Utilities** (Utilidades) o la carpeta **Other** (Otros) en el panel de inicio.

2. Seleccione **Keychain Access** (Acceso a Llaveros), expanda **Certificate Assistant** (Asistente para certificados) y, después, seleccione **Request a Certificate from a Certificate Authority** (Solicitar un certificado a una entidad de certificación).

   :::image type="content" source="media/ios-sdk-get-started/image1.png" alt-text="Acceso a Llaveros":::

   > [!NOTE]
   > De forma predeterminada, Keychain Access (Acceso con cadena de claves) selecciona el primer elemento de la lista. Esto puede ser un problema si está en la categoría **Certificates** (Certificados) y **Apple Worldwide Developer Relations Certification Authority** (Entidad de certificación de Apple Worldwide Developer Relations) no es el primer elemento de la lista. Antes de generar la solicitud de firma de certificado, asegúrese de que tiene un elemento que no sea clave o de que esté seleccionada la clave **Apple Worldwide Developer Relations Certification Authority** (Entidad de certificación de Apple Worldwide Developer Relations).

3. Seleccione un valor en **User Email Address** (Dirección de correo electrónico de usuario), escriba un valor en **Common Name** (Nombre común), asegúrese de que especifica **Saved to disk** (Guardar en disco) y, finalmente, seleccione **Continue** (Continuar). Deje en blanco **CA Email Address** (Dirección de correo de la entidad de certificación), ya que no es obligatorio.

   :::image type="content" source="media/ios-sdk-get-started/image2.png" alt-text="Información obligatoria del certificado":::

4. Escriba un nombre para el archivo CSR en **Save As** (Guardar como), seleccione la ubicación en **Where** (Dónde) y, a continuación, seleccione **Save** (Guardar).

   :::image type="content" source="media/ios-sdk-get-started/image3.png" alt-text="Elegir nombre de archivo":::

   Esta acción guarda el archivo CSR en la ubicación seleccionada. La ubicación predeterminada es **Desktop** (Escritorio). Recuerde la ubicación seleccionada para este archivo.

A continuación, se registrará la aplicación en Apple, se habilitarán las notificaciones push y se cargará el archivo CSR exportado para crear un certificado de inserción.

## <a name="register-your-app-for-push-notifications"></a>Registro de la aplicación para notificaciones push

Para enviar notificaciones push a una aplicación iOS, registre la aplicación en Apple y también regístrese para el uso de notificaciones push.

1. Si todavía no ha registrado la aplicación, vaya al [portal de aprovisionamiento de iOS](https://go.microsoft.com/fwlink/p/?LinkId=272456) en el centro para desarrolladores de Apple. Inicie sesión en el portal con su identificador de Apple y seleccione **Identifiers** (Identificadores). Luego, seleccione **+**  para registrar una nueva aplicación.

   :::image type="content" source="media/ios-sdk-get-started/image4.png" alt-text="Página de identificadores de aplicación":::

2. En la pantalla **Register a New Identifier** (Registrar un nuevo identificador), seleccione el botón de radio **App IDs** (Identificadores de aplicación). A continuación, seleccione **Continue** (Continuar).

   :::image type="content" source="media/ios-sdk-get-started/image5.png" alt-text="Página de registro de nuevo identificador":::

3. Actualice los tres campos siguientes de la nueva aplicación y, después, seleccione  **Continue** (Continuar):

   - **Descripción**: escriba un nombre descriptivo para la aplicación.
   - **Identificador de agrupación**: escriba un identificador de agrupación con el formato **Identificador de organización.Nombre de producto**, como se mencionó en la  [Guía de distribución de aplicaciones](https://help.apple.com/xcode/mac/current/#/dev91fe7130a). Los valores de **Organization Identifier** (Identificador de organización) y **Product Name** (Nombre de producto) deben coincidir con el identificador de la organización y el nombre del producto que se usa al crear el proyecto Xcode. En la captura de pantalla que aparece a continuación, se usa el valor de **NotificationHubs** como identificador de una organización y el valor **GetStarted** como el nombre del producto. Asegúrese de que el valor de **Bundle Identifier** (Identificador de agrupación) coincida con el del proyecto de Xcode para que Xcode use el perfil de publicación correcto.

      :::image type="content" source="media/ios-sdk-get-started/image6.png" alt-text="Registrar el identificador de la aplicación":::

   - **Notificaciones push**: Active la opción **Push Notifications** (Notificaciones push) en la sección **Capabilities** (Funcionalidades).

      :::image type="content" source="media/ios-sdk-get-started/image7.png" alt-text="Registrar un nuevo identificador de aplicación":::

      De esta forma, se genera el identificador de la aplicación y se solicita que confirme la información. Seleccione **Continue** (Continuar) y, después, **Register** (Registrar) para confirmar el nuevo identificador de aplicación.

      :::image type="content" source="media/ios-sdk-get-started/image8.png" alt-text="Confirmar el nuevo identificador de aplicación":::

      Tras seleccionar **Register** (Registrar), verá el nuevo identificador de aplicación como un elemento de línea en la página **Certificates, Identifiers & Profiles** (Certificados, identificadores y perfiles).

4. En esta página, **** en **Identifiers** (Identificadores), busque el elemento de línea con el identificador de aplicación que acaba de crear y seleccione su fila para mostrar la pantalla **Edit your App ID Configuration** (Editar la configuración del id. de aplicación).

## <a name="create-a-certificate-for-notification-hubs"></a>Creación de un certificado para Notification Hubs

> [!NOTE]
> Con el lanzamiento de iOS 13, solo puede recibir notificaciones silenciosas mediante la autenticación basada en token. Si usa la autenticación basada en certificados para las credenciales de APNs, debe cambiar al uso de la autenticación basada en token.

Para que el centro de notificaciones funcione con  **APNs**, se requiere un certificado. Esto se puede hacer de dos maneras:

- Cree un archivo **.p12** que se pueda cargar directamente en Notification Hubs.

- Cree un archivo **.p8** que se pueda usar para la [autenticación basada en tokens](notification-hubs-push-notification-http2-token-authentication.md) (enfoque más reciente).

La segunda opción tiene varias ventajas en comparación con el uso de certificados, como se documenta en  [Autenticación basada en token (HTTP/2) para APNs](notification-hubs-push-notification-http2-token-authentication.md). Sin embargo, se incluyen los pasos para ambas estrategias.

### <a name="option-1-create-a-p12-push-certificate-that-can-be-uploaded-directly-to-notification-hubs"></a>Opción 1: Creación de un certificado push .p12 que se pueda cargar directamente en Notification Hubs

1. Desplácese hacia abajo hasta la opción activada **Push Notifications** (Notificaciones push) y, luego, seleccione **Configure** (Configurar) para crear el certificado.

   :::image type="content" source="media/ios-sdk-get-started/image9.png" alt-text="Id. de aplicación":::

2. Aparece la ventana **Apple Push Notification service SSL Certificates** (Certificados SSL de Apple Push Notification Service). Seleccione el botón **Create Certificate** (Crear certificado) en la sección **Development SSL Certificate** (Certificado SSL de desarrollo).

   :::image type="content" source="media/ios-sdk-get-started/image10.png" alt-text="Crear certificado":::

   Se muestra la pantalla **Create a new Certificate** (Crear un nuevo certificado).

   > [!NOTE]
   > Este tutorial usa un certificado de desarrollo. Se usa el mismo proceso cuando se registra un certificado de producción. Asegúrese usar el mismo tipo de certificado al enviar notificaciones.

3. Seleccione **Choose File** (Elegir archivo), vaya a la ubicación donde guardó el archivo CSR de la primera tarea y, después, haga doble clic en el nombre del certificado para cargarlo. A continuación, seleccione **Continue** (Continuar).

4. Una vez que el portal cree el certificado, seleccione el botón **Download** (Descargar). Guarde el certificado y recuerde la ubicación donde se ha guardado.

   :::image type="content" source="media/ios-sdk-get-started/image11.png" alt-text="Descargar el certificado":::

   El certificado se descarga y se guarda en la carpeta **Downloads** (Descargas).

   :::image type="content" source="media/ios-sdk-get-started/image12.png" alt-text="Buscar el archivo de certificado":::

   De manera predeterminada, el certificado de desarrollo descargado se denomina **aps_development.cer**.

5. Haga doble clic en el certificado push descargado **aps\_development.cer**. Esta operación instala el certificado nuevo en la cadena de llaves, tal como se muestra en la siguiente imagen:

   :::image type="content" source="media/ios-sdk-get-started/image13.png" alt-text="Acceso a Llaveros":::

   Aunque el nombre del certificado puede ser diferente, llevará el prefijo **Apple Development iOS Push Services** (Servicios push de desarrollo de Apple iOS).

6. En Keychain Access (Acceso a Llaveros), haga clic en el nuevo certificado push que creó, en la categoría **Certificates** (Certificados). Seleccione **Export** (Exportar), asigne un nombre al archivo, seleccione el formato **.p12format** y, después, haga clic en **Save** (Guardar).

   :::image type="content" source="media/ios-sdk-get-started/image14.png" alt-text="Exportar un certificado":::

   De manera opcional, puede decidir proteger el certificado con una contraseña. Haga clic en **OK** (Aceptar) si quiere omitir la creación de una contraseña. Anote el nombre de archivo y la ubicación del certificado p12 exportado. Se usan para habilitar la autenticación con APNs.

   > [!NOTE]
   > El nombre y la ubicación del archivo .p12 pueden ser diferentes a los que se ilustran en este tutorial.

### <a name="option-2-create-a-p8-certificate-that-can-be-used-for-token-based-authentication"></a>Opción 2: Creación de un certificado .p8 que se pueda usar para autenticación basada en tokens

1. Anote los siguientes datos:

   - **App ID Prefix** (Prefijo de identificador de aplicación) (es un  **identificador de equipo**)
   - **Identificador de lote**

2. De vuelta en **Certificates, Identifiers & Profiles** (Certificados, identificadores y perfiles), haga clic en  **Keys** (Claves). Si ya tiene configurada una clave para **APNs**, puede volver a usar el certificado .p8 que descargó justo después de crearlo. En tal caso, puede omitir los pasos 3 a 5.

3. Haga clic en el botón **+**  (o en el botón **Create a key** [Crear una clave]) para crear una nueva clave.

4. Proporcione un valor de **Key Name** (Nombre de clave) adecuado, active la opción **Apple Push Notifications service (APNs)**  y, a continuación, haga clic en **Continue** (Continuar), seguido de  **Register** (Registrar) en la pantalla siguiente.

5. Haga clic en **Download** (Descargar) y, a continuación, mueva el archivo **.p8** (con el prefijo `AuthKey_`) a un directorio local seguro; después, haga clic en  **Done** (Listo).

   > [!IMPORTANT]
   > Asegúrese de mantener el archivo .p8 en un lugar seguro (y guardar una copia de seguridad). Después de descargar la clave, no se puede volver a descargar, ya que se quita la copia del servidor.

6. En **Keys** (Claves), haga clic en la clave que acaba de crear (o en una clave existente si decide utilizarla en su lugar).

7. Anote el valor de **Key ID** (Id. de clave).

8. Abra el certificado .p8 en una aplicación adecuada, como [Visual Studio Code](https://code.visualstudio.com/) y anote el valor de la clave. Se trata del valor que aparece entre **-----BEGIN PRIVATE KEY-----**  y **-----END PRIVATE KEY-----** .

   ```p8
   -----BEGIN PRIVATE KEY-----
   <key_value>
   -----END PRIVATE KEY-----
   ```

   Este es el valor de token que se usará más adelante para configurar Notification Hubs.

Al completar estos pasos, debe tener la siguiente información para usarla más adelante en la  [configuración de un centro de notificaciones con información de APNs](#configure-the-notification-hub-with-apns-information):

- **Identificador de equipo** (consulte el paso 1)
- **Identificador de agrupación** (consulte el paso 1)
- **Identificador de clave** (consulte el paso 7)
- **Valor de token** (el valor de la clave del certificado .p8, consulte el paso 8)

## <a name="create-a-provisioning-profile"></a>Creación de un perfil de aprovisionamiento

1. Vuelva al [portal de aprovisionamiento de iOS](https://go.microsoft.com/fwlink/p/?LinkId=272456), seleccione **Certificates, Identifiers & Profiles** (Certificados, identificadores y perfiles), seleccione **Profiles** (Perfiles) en el menú izquierdo y, luego, seleccione **+**  para crear un perfil. Aparece la pantalla **Register a New Provisioning Profile** (Registrar un nuevo perfil de aprovisionamiento).

2. En **Development** (Desarrollo), seleccione **iOS App Development** (Desarrollo de aplicaciones de iOS) como tipo de perfil de aprovisionamiento y, después, haga clic en **Continue** (Continuar).

   :::image type="content" source="media/ios-sdk-get-started/image15.png" alt-text="Lista de perfiles de aprovisionamiento":::

3. Una vez hecho esto, seleccione el identificador de la aplicación que acaba de crear en la lista desplegable **App ID** (Id. de la aplicación) y, a continuación, seleccione **Continue** (Continuar).

   :::image type="content" source="media/ios-sdk-get-started/image16.png" alt-text="Seleccionar identificador de la aplicación":::

4. En la ventana **Select certificates** (Seleccionar certificados), seleccione el certificado de desarrollo que se usa para la firma del código y, después, seleccione **Continue** (Continuar). Este certificado no es el certificado push que creó. Si no hay uno, debe crearlo. Si hay un certificado, vaya al paso siguiente. Para crear un certificado de desarrollo si no existe uno:

   1. Si ve **No Certificates are available** (No hay certificados disponibles), seleccione **Create Certificate** (Crear certificado).
   2. En la sección  **Software**, seleccione **Apple Development** (Desarrollo de Apple). A continuación, seleccione **Continue** (Continuar).
   3. En la pantalla **Create a New Certificate** (Crear un nuevo certificado), seleccione **Choose File** (Elegir archivo).
   4. Vaya al certificado de **Certificate Signing Request** (Solicitud de firma de certificado) que creó anteriormente, selecciónelo y, luego, elija **Open** (Abrir).
   5. Seleccione **Continue** (Continuar).
   6. Descargue el certificado de desarrollo y recuerde la ubicación donde lo guarda.

5. Vuelva a la página **Certificates, Identifiers & Profiles** (Certificados, identificadores y perfiles), seleccione **Profiles** (Perfiles) en el menú izquierdo y, luego, seleccione **+**  para crear un perfil. Aparece la pantalla **Register a New Provisioning Profile** (Registrar un nuevo perfil de aprovisionamiento).

6. En la ventana **Select certificates** (Seleccionar certificados), seleccione el certificado de desarrollo que acaba de crear. A continuación, seleccione **Continue** (Continuar).

7. Después, seleccione los dispositivos que va a usar para la prueba y elija **Continue** (Continuar).

8. Para terminar, elija un nombre para el perfil en **Provisioning Profile Name** (Nombre del perfil de aprovisionamiento) y seleccione  **Generate** (Generar).

   :::image type="content" source="media/ios-sdk-get-started/image17.png" alt-text="Elegir el nombre del perfil de aprovisionamiento":::

9. Cuando se cree el perfil de aprovisionamiento, seleccione **Download** (Descargar). Recuerde la ubicación donde lo guarda.

10. Vaya a la ubicación del perfil de aprovisionamiento y haga doble clic en él para instalarlo en la máquina de desarrollo de Xcode.

## <a name="create-a-notification-hub"></a>Creación de un centro de notificaciones

En esta sección, va a crear un centro de notificaciones y configurar la autenticación con APNs mediante el certificado push .p12 o la autenticación basada en tokens. Si desea usar un centro de notificaciones ya creado, puede omitir los pasos hasta el paso 5.

1. Inicie sesión en  [Azure Portal](https://portal.azure.com/).

2. Seleccione **Todos los servicios** en el menú de la izquierda y, a continuación, seleccione **Notification Hubs** en la sección **Móvil** . Seleccione el icono de estrella junto al nombre del servicio para agregarlo a la sección **FAVORITOS** en el menú de la izquierda. Tras agregar **Notification Hubs** a **FAVORITOS**, selecciónelo.

   :::image type="content" source="media/ios-sdk-get-started/image18.png" alt-text="Azure Portal":::

3. En la página **Notification Hubs**, seleccione **Agregar** en la barra de herramientas.

   :::image type="content" source="media/ios-sdk-get-started/image19.png" alt-text="Agregar botón de la barra de herramientas":::

4. En la página **Notification Hubs**, lleve a cabo los siguientes pasos:

   1. Escriba un nombre en **Centro de notificaciones**.
   2. Escriba un nombre en **Crear un nuevo espacio de nombres**. Un espacio de nombres contiene uno o varios centros de notificaciones.
   3. Seleccione un valor de la lista desplegable **Ubicación**. Este valor especifica la ubicación en la que quiere crear el centro de notificaciones.
   4. Seleccione un grupo de recursos existente en  **Grupo de recursos** o bien cree uno nuevo.
   5. Seleccione  **Crear**.

   :::image type="content" source="media/ios-sdk-get-started/image20.png" alt-text="Establecer las propiedades":::

5. Seleccione **Notificaciones** (el icono de campana) y, luego, **Ir al recurso**. También puede actualizar la lista en la página **Notification Hubs** y seleccionar su centro.

   :::image type="content" source="media/ios-sdk-get-started/image21.png" alt-text="Notificaciones del portal":::

6. Seleccione **Directivas de acceso** de la lista. Observe que las dos cadenas de conexión están disponibles. Las necesitará más adelante para gestionar las notificaciones push.

   > [!IMPORTANT]
   > No use la directiva **DefaultFullSharedAccessSignature** en la aplicación. Está pensada para usarse solamente en el back-end.

   :::image type="content" source="media/ios-sdk-get-started/image22.png" alt-text="Cadenas de conexión":::

## <a name="configure-the-notification-hub-with-apns-information"></a>Configurar el centro de notificaciones con la información de APNs

En **Servicios de notificaciones**, seleccione **Apple (APNS)** y siga los pasos adecuados en función del enfoque elegido anteriormente en la sección [Creación de un certificado para Notification Hubs](#create-a-certificate-for-notification-hubs).

> [!NOTE]
> Use **Producción** como **Modo de aplicación** solo si desea enviar notificaciones push a los usuarios que compraron la aplicación en la tienda.

### <a name="option-1-use-a-p12-push-certificate"></a>Opción 1: Uso de un certificado push .p12

1. Seleccione **Certificado**.

2. Seleccione el icono del archivo.

3. Seleccione el archivo .p12 que exportó antes y, luego, **Abrir**.

4. Si es necesario, especifique la contraseña correcta.

5. Seleccione el modo **Espacio aislado**.

   :::image type="content" source="media/ios-sdk-get-started/image23.png" alt-text="Configuración":::

6. Seleccione  **Guardar**.

### <a name="option-2-use-token-based-authentication"></a>Opción 2: Uso de la autenticación basada en tokens

1. Seleccione **Token**.

2. Escriba los siguientes valores adquiridos anteriormente:

   - **Identificador de clave**
   - **Identificador de lote**
   - **Identificador de equipo**
   - **Token**

3. Elija **Espacio aislado**.

4. Seleccione  **Guardar**.

Ahora ya ha configurado el centro de notificaciones con APNs. Tiene también la cadena de conexión para registrar la aplicación y enviar notificaciones push.

## <a name="next-steps"></a>Pasos siguientes

En este tutorial, creó y configuró un centro de notificaciones en Azure y lo configuró para permitir el envío de notificaciones a la aplicación mediante Apple Push Notification Service (APNs). A continuación, crearemos una aplicación iOS de ejemplo e integraremos el SDK de Azure Notification Hubs para que pueda recibir notificaciones push enviadas a través de Azure Portal. Continúe en el siguiente tutorial en función del lenguaje que elija:

- [Tutorial: Integración con una aplicación iOS mediante Swift]()
