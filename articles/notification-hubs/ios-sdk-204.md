---
title: Envío de notificaciones push a iOS con Azure Notification Hubs y la versión 2.0.4 del SDK de iOS
description: En este tutorial, aprenderá a usar Azure Notification Hubs y Apple Push Notification Service para enviar notificaciones push a dispositivos iOS.
author: sethmanheim
ms.author: sethm
ms.date: 06/19/2020
ms.topic: tutorial
ms.service: notification-hubs
ms.reviewer: thsomasu
ms.lastreviewed: 06/01/2020
ms.openlocfilehash: d89d46e3365a97d9deea8a89de2d9a1d5799cb72
ms.sourcegitcommit: 2ff0d073607bc746ffc638a84bb026d1705e543e
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/06/2020
ms.locfileid: "87836061"
---
# <a name="tutorial-send-push-notifications-to-ios-apps-using-azure-notification-hubs-version-204"></a>Tutorial: Envío de notificaciones push a aplicaciones iOS mediante Azure Notification Hubs (versión 2.0.4)

En este tutorial se muestra cómo usar Azure Notification Hubs para enviar notificaciones push a una aplicación iOS mediante la versión 2.0.4 del SDK de Azure Notification Hubs.

En este tutorial se describen los pasos siguientes:

- Cree una aplicación iOS de ejemplo.
- Conecte la aplicación iOS a Azure Notification Hubs.
- Pruebe el envío de las notificaciones push.
- Compruebe que la aplicación recibe notificaciones.

Puede descargar el código completo para este tutorial en [GitHub](https://github.com/Azure/azure-notificationhubs-ios/tree/v3-preview2/Samples).

## <a name="prerequisites"></a>Requisitos previos

Para completar este tutorial, debe cumplir los siguientes requisitos previos:

- Mac con [Xcode](https://go.microsoft.com/fwLink/p/?LinkID=266532), junto con un certificado de desarrollador válido instalado en el Llavero.
- Un iPhone o iPad con iOS versión 10 o posterior.
- El dispositivo físico registrado en el [portal de Apple](https://developer.apple.com/) y asociado con el certificado.

Antes de continuar, asegúrese de seguir el tutorial anterior sobre cómo empezar a usar [Azure Notification Hubs para aplicaciones iOS](ios-sdk-get-started.md), para instalar y configurar las credenciales push en el centro de notificaciones. Aunque no tenga ninguna experiencia previa con el desarrollo de iOS, debería poder seguir estos pasos.

> [!NOTE]
> Debido a los requisitos de configuración de las notificaciones push, debe implementar y probar estas en un dispositivo iOS físico (iPhone o iPad), en lugar de hacerlo en el emulador de iOS.

## <a name="connect-your-ios-app-to-notification-hubs"></a>Conexión de la aplicación iOS a Notification Hubs

1. En Xcode, cree un nuevo proyecto iOS y seleccione la plantilla **Single View Application** (Aplicación de vista única).

   :::image type="content" source="media/ios-sdk/image1.png" alt-text="Seleccionar plantilla":::

2. Al configurar las opciones para el nuevo proyecto, asegúrese de usar el mismo **nombre de producto** e **identificador de organización** que usó al establecer el identificador de agrupación en el portal para desarrolladores de Apple.

3. En Project Navigator (Explorador de proyectos), seleccione el nombre del proyecto en **Targets** (Destinos) y, a continuación, seleccione la pestaña **Signing & Capabilities** (Firma y funcionalidades). Asegúrese de seleccionar el **equipo** adecuado para la cuenta de desarrollador de Apple. XCode debe desplegar automáticamente el perfil de aprovisionamiento que creó anteriormente según en el identificador del conjunto.

   Si no ve el nuevo perfil de aprovisionamiento que creó en Xcode, intente actualizar los perfiles de la identidad de firma. Haga clic en **Xcode** en la barra de menús, en **Preferences** (Preferencias), en la pestaña **Account** (Cuenta), en el botón **View Details** (Ver detalles), en la identidad de firma y, por último, en el botón Refresh (Actualizar) en la esquina inferior derecha.

   :::image type="content" source="media/ios-sdk/image2.png" alt-text="Ver detalles":::

4. En la pestaña **Signing & Capabilities** (Firma y funcionalidades), seleccione **+ Capability** (Funcionalidad). Haga doble clic en **Push Notifications** (Notificaciones push) para habilitarla.

   :::image type="content" source="media/ios-sdk/image3.png" alt-text="Funcionalidad":::

5. Agregue los módulos de SDK de Azure Notification Hubs.

   Puede integrar el SDK de Azure Notification Hubs en su aplicación mediante [Cocoapods](https://cocoapods.org/) o agregando manualmente los archivos binarios al proyecto.

   - Integración a través de Cocoapods: agregue las siguientes dependencias al podfile para incluir el SDK de Azure Notification Hubs en la aplicación:

      ```ruby
      pod 'AzureNotificationHubs-iOS'
      ```

      - Ejecute pod install para instalar el pod recién definido y abra el archivo .xcworkspace.

         Si ve un error como **Unable to find a specification for AzureNotificationHubs-iOS** (No se puede encontrar una especificación para AzureNotificationHubs-iOS) mientras se ejecuta pod install, ejecute `pod repo update` para obtener los pods más recientes del repositorio de Cocoapods y, a continuación, ejecute pod install.

   - Integración a través de Carthage: agregue las siguientes dependencias al archivo Cartfile para incluir el SDK de Azure Notification Hubs en la aplicación:

      ```ruby
      github "Azure/azure-notificationhubs-ios"
      ```

      - A continuación, actualice las dependencias de compilación:

      ```shell
      $ carthage update
      ```

      Para obtener más información sobre el uso de Carthage, consulte el [repositorio de GitHub de Carthage](https://github.com/Carthage/Carthage).

   - Integración mediante la copia de los archivos binarios en el proyecto: Puede realizar la integración mediante la copia de los archivos binarios en el proyecto, como se indica a continuación:

        - Descargue el marco del [SDK de Azure Notification Hubs](https://github.com/Azure/azure-notificationhubs-android/releases) proporcionado como archivo ZIP y descomprímalo.

        - En Xcode, haga clic con el botón derecho en el proyecto y haga clic en la opción **Add Files to** (Agregar archivos a) para agregar la carpeta **WindowsAzureMessaging.framework** al proyecto de Xcode. Seleccione **Options** (Opciones) y asegúrese de que **Copy items if needed** (Copiar elementos si es necesario) esté seleccionado. A continuación, haga clic en **Add** (Agregar).

          :::image type="content" source="media/ios-sdk/image4.png" alt-text="Agregar marco":::

6. Agregue un nuevo archivo de encabezado denominado **Constants.h** al proyecto. Para ello, haga clic con el botón derecho en el nombre del proyecto y seleccione **New File...** (Nuevo archivo...). A continuación, seleccione **Header File** (Archivo de encabezado). Este archivo contiene las constantes del Centro de notificaciones. A continuación, seleccione  **Siguiente**. Asigne al archivo el nombre **Constants.h**.

7. Agregue el siguiente código al archivo Constants.h:

   ```objc
   #ifndef Constants_h
   #define Constants_h
   extern NSString* const NHInfoConnectionString;
   extern NSString* const NHInfoHubName;
   extern NSString* const NHUserDefaultTags;
   #endif /* Constants_h */
   ```

8. Agregue el archivo de implementación para Constants.h. Para ello, haga clic con el botón derecho en el nombre del proyecto y seleccione **New File...** (Nuevo archivo...). Seleccione **Objective-C File** (Archivo de Objective-C) y, a continuación, seleccione **Next** (Siguiente). Asigne al archivo el nombre **Constants.m**.

   :::image type="content" source="media/ios-sdk/image5.png" alt-text="Agregar el archivo de implementación":::

9. Abra el archivo **Constants.m** y reemplace su contenido por el código siguiente. Reemplace los marcadores de posición de literal de cadena `NotificationHubConnectionString` y `NotificationHubConnectionString` por el nombre del centro de conectividad y por **DefaultListenSharedAccessSignature** respectivamente, tal como se obtuvo anteriormente en el portal:

   ```objc
   #import <Foundation/Foundation.h>
   #import "Constants.h"

   NSString* const NHInfoConnectionString = @"NotificationHubConnectionString";
   NSString* const NHInfoHubName = @"NotificationHubName";NSString* const NHUserDefaultTags = @"notification_tags";
   ```

10. Abra el archivo **AppDelegate.h** del proyecto y reemplace su contenido por el código siguiente:

    ```objc
    #import <UIKit/UIKit.h>
    #import <WindowsAzureMessaging/WindowsAzureMessaging.h>
    #import <UserNotifications/UserNotifications.h> 

    @interface AppDelegate : UIResponder <UIApplicationDelegate,   UNUserNotificationCenterDelegate>

    @property (strong, nonatomic) UIWindow *window;

    - (void)handleRegister;
    - (void)handleUnregister;

    @end
    ```

11. En el archivo **AppDelegate.m** del proyecto, agregue las instrucciones `import` siguientes:

    ```objc
    #import "Constants.h"
    #import "NotificationDetailViewController.h"
    ```

12. También en el archivo **AppDelegate.m**, agregue la siguiente línea de código en el método `didFinishLaunchingWithOptions`, en función de su versión de iOS. Este código registra el identificador de dispositivo en APNS:

    ```objc
    [[UNUserNotificationCenter currentNotificationCenter] setDelegate:self];
    ```

13. En el mismo archivo **AppDelegate.m**, reemplace todo el código después de `didFinishLaunchingWithOptions` por el código siguiente:

    ```objc
    // Tells the app that a remote notification arrived that indicates there is data to be fetched.

    - (void)application:(UIApplication *)application didReceiveRemoteNotification:(NSDictionary *)userInfo fetchCompletionHandler:(void (^)(UIBackgroundFetchResult result))completionHandler {
    NSLog(@"Received remote (silent) notification");
    [self logNotificationDetails:userInfo];

    //
    // Let the system know the silent notification has been processed.
    //
    completionHandler(UIBackgroundFetchResultNoData);
    }

    // Tells the delegate that the app successfully registered with Apple Push Notification service (APNs).

    - (void) application:(UIApplication *)application didRegisterForRemoteNotificationsWithDeviceToken:(NSData *)deviceToken {
    NSMutableSet *tags = [[NSMutableSet alloc] init];

    // Load and parse stored tags
    NSString *unparsedTags = [[NSUserDefaults standardUserDefaults] valueForKey:NHUserDefaultTags];
    if (unparsedTags.length > 0) {
        NSArray *tagsArray = [unparsedTags componentsSeparatedByString: @","];
        [tags addObjectsFromArray:tagsArray];
    }

    // Register the device with the Notification Hub.
    // If the device has not already been registered, this will create the registration.
    // If the device has already been registered, this will update the existing registration.
    //
    SBNotificationHub* hub = [self getNotificationHub];
    [hub registerNativeWithDeviceToken:deviceToken tags:tags completion:^(NSError* error) {
        if (error != nil) {
            NSLog(@"Error registering for notifications: %@", error);
        } else {
            [self showAlert:@"Registered" withTitle:@"Registration Status"];
        }
    }];
    }

    // UNUserNotificationCenterDelegate methods
    //
    // Asks the delegate how to handle a notification that arrived while the app was running in the  foreground.

    - (void)userNotificationCenter:(UNUserNotificationCenter *)center willPresentNotification:(UNNotification *)notification withCompletionHandler:(void (^)(UNNotificationPresentationOptions options))completionHandler {
    NSLog(@"Received notification while the application is in the foreground");

    // The system calls this delegate method when the app is in the foreground. This allows the app to handle the notification
    // itself (and potentially modify the default system behavior).

    // Handle the notification by displaying custom UI.
    //
    [self showNotification:notification.request.content.userInfo];

    // Use 'options' to specify which default behaviors to enable.
    // - UNAuthorizationOptionBadge: Apply the notification's badge value to the app’s icon.
    // - UNAuthorizationOptionSound: Play the sound associated with the notification.
    // - UNAuthorizationOptionAlert: Display the alert using the content provided by the notification.
    //
    // In this case, do not pass UNAuthorizationOptionAlert because the notification was handled by the app.
    //
    completionHandler(UNAuthorizationOptionBadge | UNAuthorizationOptionSound);
    }

    // Asks the delegate to process the user's response to a delivered notification.
    //

    - (void)userNotificationCenter:(UNUserNotificationCenter *)center didReceiveNotificationResponse:(UNNotificationResponse *)response withCompletionHandler:(void(^)(void))completionHandler {
    NSLog(@"Received notification while the application is in the background");

    // The system calls this delegate method when the user taps or responds to the system notification.

    // Handle the notification response by displaying custom UI
    //
    [self showNotification:response.notification.request.content.userInfo];

    // Let the system know the response has been processed.
    //
    completionHandler();
    }

    // App logic and helpers

    - (SBNotificationHub *)getNotificationHub {
    NSString *hubName = [[NSBundle mainBundle] objectForInfoDictionaryKey:NHInfoHubName];
    NSString *connectionString = [[NSBundle mainBundle] objectForInfoDictionaryKey:NHInfoConnectionString];

    return [[SBNotificationHub alloc] initWithConnectionString:connectionString notificationHubPath:hubName];
    }

    - (void)handleRegister {
    UNUserNotificationCenter *center = [UNUserNotificationCenter currentNotificationCenter];

    UNAuthorizationOptions options =  UNAuthorizationOptionAlert | UNAuthorizationOptionSound | UNAuthorizationOptionBadge;
    [center requestAuthorizationWithOptions:(options) completionHandler:^(BOOL granted, NSError * _Nullable error) {
        if (error != nil) {
            NSLog(@"Error requesting for authorization: %@", error);
        }
    }];
    [[UIApplication sharedApplication] registerForRemoteNotifications];
    }

    - (void)handleUnregister {
    //
    // Unregister the device with the Notification Hub.
    //
    SBNotificationHub *hub = [self getNotificationHub];
    [hub unregisterNativeWithCompletion:^(NSError* error) {
        if (error != nil) {
            NSLog(@"Error unregistering for push: %@", error);
        } else {
            [self showAlert:@"Unregistered" withTitle:@"Registration Status"];
        }
    }];
    }

    - (void)logNotificationDetails:(NSDictionary *)userInfo {
    if (userInfo != nil) {
        UIApplicationState state = [UIApplication sharedApplication].applicationState;
        BOOL background = state != UIApplicationStateActive;
        NSLog(@"Received %@notification: \n%@", background ? @"(background) " : @"", userInfo);
    }
    }

    - (void)showAlert:(NSString *)message withTitle:(NSString *)title {
    if (title == nil) {
        title = @"Alert";
    }
    UIAlertController *alert = [UIAlertController alertControllerWithTitle:title message:message preferredStyle:UIAlertControllerStyleAlert];
    [alert addAction:[UIAlertAction actionWithTitle:@"OK" style:UIAlertActionStyleDefault handler:nil]];
    [[[[UIApplication sharedApplication] keyWindow] rootViewController] presentViewController:alert animated:YES completion:nil];
    }

    - (void)showNotification:(NSDictionary *)userInfo {
    [self logNotificationDetails:userInfo];

    NotificationDetailViewController *notificationDetail = [[NotificationDetailViewController alloc] initWithUserInfo:userInfo];
    [[[[UIApplication sharedApplication] keyWindow] rootViewController] presentViewController:notificationDetail animated:YES completion:nil];
    }

    @end
    ```

    Este código establece la conexión con el centro de notificaciones usando la información de conexión que especificó en **Constants.h**. Luego, proporciona el token del dispositivo al centro de notificaciones para que este pueda enviar notificaciones.

### <a name="create-notificationdetailviewcontroller-header-file"></a>Crear el archivo de encabezado NotificationDetailViewController

1. Al igual que en las instrucciones anteriores, agregue otro archivo de encabezado denominado **NotificationDetailViewController.h**. Reemplace el contenido del nuevo archivo de encabezado por el código siguiente:

   ```objc
   #import <UIKit/UIKit.h>

   NS_ASSUME_NONNULL_BEGIN

   @interface NotificationDetailViewController : UIViewController

   @property (strong, nonatomic) IBOutlet UILabel *titleLabel;
   @property (strong, nonatomic) IBOutlet UILabel *bodyLabel;
   @property (strong, nonatomic) IBOutlet UIButton *dismissButton;

   @property (strong, nonatomic) NSDictionary *userInfo;

   - (id)initWithUserInfo:(NSDictionary *)userInfo;

   @end

   NS_ASSUME_NONNULL_END
   ```

2. Agregue el archivo de implementación **NotificationDetailViewController.m**. Reemplace el contenido del archivo por el código siguiente, que implementa los métodos UIViewController:

   ```objc
   #import "NotificationDetailViewController.h"

   @interface NotificationDetailViewController ()

   @end

   @implementation NotificationDetailViewController

   - (id)initWithUserInfo:(NSDictionary *)userInfo {
    self = [super initWithNibName:@"NotificationDetail" bundle:nil];
    if (self) {
        _userInfo = userInfo;
    }
    return self;
   }

   - (void)viewDidLayoutSubviews {
    [self.titleLabel sizeToFit];
    [self.bodyLabel sizeToFit];
   }

   - (void)viewDidLoad {
    [super viewDidLoad];

    NSString *title = nil;
    NSString *body = nil;

    NSDictionary *aps = [_userInfo valueForKey:@"aps"];
    NSObject *alertObject = [aps valueForKey:@"alert"];
    if (alertObject != nil) {
        if ([alertObject isKindOfClass:[NSDictionary class]]) {
            NSDictionary *alertDict = (NSDictionary *)alertObject;
            title = [alertDict valueForKey:@"title"];
            body = [alertObject valueForKey:@"body"];
        } else if ([alertObject isKindOfClass:[NSString class]]) {
            body = (NSString *)alertObject;
        } else {
            NSLog(@"Unable to parse notification content. Unexpected format: %@", alertObject);
        }
    }

    if (title == nil) {
        title = @"<unset>";
    }

    if (body == nil) {
        body = @"<unset>";
    }

    self.titleLabel.text = title;
    self.bodyLabel.text = body;
   }

   - (IBAction)handleDismiss:(id)sender {
    [self dismissViewControllerAnimated:YES completion:nil];
   }

   @end
   ```

### <a name="viewcontroller"></a>ViewController

1. En el archivo **ViewController.h** del proyecto, agregue las instrucciones `import` siguientes:

   ```objc
   #import <WindowsAzureMessaging/WindowsAzureMessaging.h>
   #import <UserNotifications/UserNotifications.h>
   ```

2. También en **ViewController.h**, agregue las siguientes declaraciones de propiedad después de la declaración de `@interface`:

   ```objc
   @property (strong, nonatomic) IBOutlet UITextField *tagsTextField;
   @property (strong, nonatomic) IBOutlet UIButton *registerButton;
   @property (strong, nonatomic) IBOutlet UIButton *unregisterButton;
   ```

3. En el archivo de implementación **ViewController.m** del proyecto, reemplace su contenido por el código siguiente:

   ```objc
   #import "ViewController.h"
   #import "Constants.h"
   #import "AppDelegate.h"

   @interface ViewController ()

   @end

   @implementation ViewController

   // UIViewController methods

   - (void)touchesBegan:(NSSet<UITouch *> *)touches withEvent:(UIEvent *)event {
    // Simple method to dismiss keyboard when user taps outside of the UITextField.
    [self.view endEditing:YES];
   }

   - (void)viewDidLoad {
    [super viewDidLoad];

    // Load raw tags text from storage and initialize the text field
    self.tagsTextField.text = [[NSUserDefaults standardUserDefaults] valueForKey:NHUserDefaultTags];
   }

   - (IBAction)handleRegister:(id)sender {
    // Save raw tags text in storage
    [[NSUserDefaults standardUserDefaults] setValue:self.tagsTextField.text forKey:NHUserDefaultTags];

   // Delegate processing the register action to the app delegate.
   [[[UIApplication sharedApplication] delegate] performSelector:@selector(handleRegister)];
   }

   - (IBAction)handleUnregister:(id)sender {
    [[[UIApplication sharedApplication] delegate] performSelector:@selector(handleUnregister)];
   }

   @end
   ```

4. Para asegurarse de que no haya errores, compile y ejecute la aplicación en el dispositivo.

## <a name="send-test-push-notifications"></a>Prueba de envío de las notificaciones push

Puede probar a recibir notificaciones en la aplicación con la opción **Envío de prueba**  de [Azure Portal](https://portal.azure.com/). Envía una notificación push de prueba al dispositivo.

:::image type="content" source="media/ios-sdk/image6.png" alt-text="Enviar prueba":::

Las notificaciones push se envían normalmente en un servicio back-end como Mobile Apps o ASP.NET mediante una biblioteca compatible. Si no hay disponible ninguna biblioteca para su back-end, también puede usar la API REST directamente para enviar mensajes de notificación.

A continuación, presentamos una lista de algunos otros tutoriales que podría interesarle revisar para enviar notificaciones:

- Azure Mobile Apps: Para ver un ejemplo de cómo enviar notificaciones desde un back-end de Mobile Apps integrado con Notification Hubs, consulte [Incorporación de notificaciones push a la aplicación iOS](/previous-versions/azure/app-service-mobile/app-service-mobile-ios-get-started-push).
- ASP.NET:  [Uso de Notification Hubs para enviar notificaciones push a los usuarios](notification-hubs-aspnet-backend-ios-apple-apns-notification.md).
- SDK de Java para Azure Notification Hubs: Consulte [Uso de Notification Hubs desde Java](notification-hubs-java-push-notification-tutorial.md) para enviar notificaciones desde Java. Esto se probó en Eclipse para el desarrollo de Android.
- PHP: [Uso de Notification Hubs desde PHP](notification-hubs-php-push-notification-tutorial.md).

## <a name="verify-that-your-app-receives-push-notifications"></a>Compruebe que la aplicación recibe notificaciones push

Para probar las notificaciones push en iOS, debe implementar la aplicación en un dispositivo iOS físico. No puede enviar notificaciones push de Apple con el simulador de iOS.

1. Ejecute la aplicación y compruebe que el registro se realiza correctamente, luego presione **OK** (Aceptar).

   :::image type="content" source="media/ios-sdk/image7.png" alt-text="Registro":::

2. Después, envíe una notificación push de prueba desde [Azure Portal](https://portal.azure.com/), como se ha descrito en la sección anterior.

3. La notificación push se envía a todos los dispositivos registrados para recibir las notificaciones del centro de notificaciones indicado.

   :::image type="content" source="media/ios-sdk/image8.png" alt-text="Envío de prueba":::

## <a name="next-steps"></a>Pasos siguientes

En este sencillo ejemplo, se difunden notificaciones push a todos los dispositivos iOS registrados. Para aprender a enviar notificaciones push a dispositivos iOS específicos, pase al siguiente tutorial:

[Tutorial: Envío de notificaciones push a dispositivos específicos](notification-hubs-ios-xplat-segmented-apns-push-notification.md)

Para más información, consulte los siguientes artículos.

- [Introducción a Azure Notification Hubs](notification-hubs-push-notification-overview.md)
- [API de REST de Notification Hubs](/rest/api/notificationhubs/)
- [SDK de Notification Hubs para operaciones de back-end](https://www.nuget.org/packages/Microsoft.Azure.NotificationHubs/)
- [SDK de Notification Hubs en GitHub](https://github.com/Azure/azure-notificationhubs)
- [Registro en el back-end de aplicación](notification-hubs-ios-aspnet-register-user-from-backend-to-push-notification.md)
- [Administración de registros](notification-hubs-push-notification-registration-management.md)
- [Trabajo con etiquetas](notification-hubs-tags-segment-push-message.md)
- [Trabajo con plantillas personalizadas](notification-hubs-templates-cross-platform-push-messages.md)
- [Control de acceso de Service Bus con Firmas de acceso compartido](../service-bus-messaging/service-bus-sas.md)
- [Generar tokens de SAS mediante programación](/rest/api/eventhub/generate-sas-token)
- [Seguridad de Apple: cifrado común](https://developer.apple.com/security/)
- [Tiempo UNIX](https://en.wikipedia.org/wiki/Unix_time)
- [HMAC](https://en.wikipedia.org/wiki/HMAC)