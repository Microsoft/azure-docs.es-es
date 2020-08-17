---
title: Configuración de notificaciones push en Azure Notification Hubs | Microsoft Docs
description: Aprenda a configurar Azure Notification Hubs en Azure Portal mediante la configuración del sistema de notificación de plataforma (PNS).
services: notification-hubs
author: sethmanheim
manager: femila
ms.service: notification-hubs
ms.workload: mobile
ms.topic: quickstart
ms.date: 06/22/2020
ms.author: sethm
ms.reviewer: thsomasu
ms.lastreviewed: 02/14/2019
ms.custom: devx-track-azurecli
ms.openlocfilehash: c8e2372e22c9db3aa5ea8b0bd953f18a2c01fbe2
ms.sourcegitcommit: 5a37753456bc2e152c3cb765b90dc7815c27a0a8
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/04/2020
ms.locfileid: "87761032"
---
# <a name="quickstart-set-up-push-notifications-in-a-notification-hub"></a>Inicio rápido: Configuración de notificaciones push en un centro de notificaciones

Azure Notification Hubs proporciona un motor de inserción que es fácil de usar y que se escala horizontalmente. Use Notification Hubs para enviar notificaciones a cualquier plataforma (iOS, Android, Windows, Baidu), desde cualquier back-end (en la nube o local). Para más información, consulte [¿Qué es Azure Notification Hubs?](notification-hubs-push-notification-overview.md).

En este inicio rápido, se usará la configuración del sistema de notificación de plataforma (PNS) en Notification Hubs para configurar las notificaciones push en varias plataformas. El inicio rápido muestra los pasos a seguir en Azure Portal. [Google Firebase Cloud Messaging](?tabs=azure-cli#google-firebase-cloud-messaging-fcm) incluye instrucciones sobre el uso de la CLI de Azure.

Si aún no ha creado un centro de notificaciones, cree uno ahora. Para más información, consulte [Creación de un centro de notificaciones de Azure en Azure Portal](create-notification-hub-portal.md) o [Creación de un centro de notificaciones de Azure mediante la CLI de Azure](create-notification-hub-azure-cli.md).

## <a name="apple-push-notification-service"></a>Apple Push Notification Service

Para configurar Apple Push Notification Service (APNS):

1. En Azure Portal, en la página **Centro de notificaciones**, seleccione **Apple (APNS)** en el menú izquierdo.

1. Para **Modo de autenticación**, seleccione **Certificado** o **Token**.

   a. Si ha seleccionado **Certificado**:
   * Seleccione el icono de archivo, y luego el archivo *.p12* que desea cargar.
   * Escriba una contraseña.
   * Seleccione el modo **Espacio aislado**. O para enviar notificaciones push a los usuarios que compraron la aplicación en la tienda, seleccione el modo **Producción**.

     ![Captura de pantalla de una configuración del certificado APNS en Azure Portal](./media/notification-hubs-ios-get-started/notification-hubs-apple-config-cert.png)

   b. Si selecciona **Token**:

   * Especifique los valores **Id. de clave**, **Id. de agrupación**, **Id. de equipo** y **Token**.
   * Seleccione el modo **Espacio aislado**. O para enviar notificaciones push a los usuarios que compraron la aplicación en la tienda, seleccione el modo **Producción**.

     ![Captura de pantalla de una configuración del token APNS en Azure Portal](./media/configure-notification-hub-portal-pns-settings/notification-hubs-apple-config-token.png)

Para más información, consulte [Envío de notificaciones push a aplicaciones iOS mediante Azure Notification Hubs](ios-sdk-get-started.md).

## <a name="google-firebase-cloud-messaging-fcm"></a>Google Firebase Cloud Messaging (FCM)

# <a name="portal"></a>[Portal](#tab/azure-portal)

Para configurar las notificaciones push para Google FCM:

1. En Azure Portal, en la página **Centro de notificaciones**, seleccione **Google (GCM/FCM)** en el menú izquierdo.
2. Pegue la **clave de API** del proyecto de Google FCM que guardó anteriormente.
3. Seleccione **Guardar**.

   ![Captura de pantalla que muestra cómo configurar Notification Hubs para Google FCM](./media/notification-hubs-android-push-notification-google-fcm-get-started/fcm-server-key.png)

Cuando haya completado estos pasos, una alerta le indica que el centro de notificaciones se ha actualizado correctamente. El botón **Save** (Guardar) está deshabilitado.

# <a name="azure-cli"></a>[CLI de Azure](#tab/azure-cli)

### <a name="prerequisites"></a>Requisitos previos

Antes de empezar, debe disponer de lo siguiente:

* La [CLI de Azure](/cli/azure/install-azure-cli), versión 2.0.67 o cualquier versión posterior.

* La [extensión para Notification Hubs](/cli/azure/ext/notification-hub/notification-hub) de la CLI de Azure.
* La **clave de API** de un proyecto de Google Firebase Cloud Messaging (FCM).

### <a name="set-up-push-notifications-for-google-fcm"></a>Configuración de notificaciones push para Google FCM

1. Use el comando [az notification-hub credential gcm update](/cli/azure/ext/notification-hub/notification-hub/credential/gcm#ext-notification-hub-az-notification-hub-credential-gcm-update) para agregar la clave de API de Google al centro de notificaciones.

   ```azurecli
   az notification-hub credential gcm update --resource-group spnhubrg --namespace-name spnhubns    --notification-hub-name spfcmtutorial1nhub --google-api-key myKey
   ```

2. La aplicación Android necesita una cadena de conexión para conectar con el centro de notificaciones.  Use el comando [az notification-hub authorization-rule list](/cli/azure/ext/notification-hub/notification-hub/authorization-rule#ext-notification-hub-az-notification-hub-authorization-rule-list) para enumerar las directivas de acceso disponibles.  Use el comando [az notification-hub authorization-rule list-keys](/cli/azure/ext/notification-hub/notification-hub/authorization-rule#ext-notification-hub-az-notification-hub-authorization-rule-list-keys) para obtener las cadenas de conexión de la directiva de acceso.  Especifique el valor de **primaryConnectionString** o **secondaryConnectionString** en el parámetro `--query` para obtener la cadena de conexión principal directamente.

   ```azurecli
   #list access policies for a notification hub
   az notification-hub authorization-rule list --resource-group spnhubrg --namespace-name spnhubns --notification-hub-name spfcmtutorial1nhub --output table

   #list keys and connection strings for a notification hub access policy
   az notification-hub authorization-rule list-keys --resource-group spnhubrg --namespace-name spnhubns --notification-hub-name spfcmtutorial1nhub --name myAccessPolicyName --output json

   #get the primaryConnectionString for an access policy
   az notification-hub authorization-rule list-keys --resource-group spnhubrg --namespace-name spnhubns --notification-hub-name spfcmtutorial1nhub --name myAccessPolicyName --query primaryConnectionString
   ```

3. Use el comando [az notification-hub test-send](/cli/azure/ext/notification-hub/notification-hub#ext-notification-hub-az-notification-hub-test-send) para probar el envío de mensajes a la aplicación Android.

   ```azurecli
   #test with message body
   az notification-hub test-send --resource-group spnhubrg --namespace-name spnhubns --notification-hub-name spfcmtutorial1nhub --notification-format gcm --message "my message body"

   #test with JSON string
   az notification-hub test-send --resource-group spnhubrg --namespace-name spnhubns --notification-hub-name spfcmtutorial1nhub --notification-format gcm --payload "{\"data\":{\"message\":\"my JSON string\"}}"
   ```

Obtenga referencias de la CLI de Azure para otras plataformas con el comando [az notification-hub credential](/cli/azure/ext/notification-hub/notification-hub/credential).

Para más información sobre el envío de notificaciones a una aplicación Android, consulte [Envío de notificaciones push a dispositivos Android mediante Firebase](notification-hubs-android-push-notification-google-fcm-get-started.md).

## <a name="windows-push-notification-service"></a>Servicios de notificaciones de inserción de Windows

Para configurar Servicios de notificaciones de inserción de Windows (WNS):

1. En Azure Portal, en la página **Centro de notificaciones**, seleccione **Windows (WNS)** en el menú izquierdo.
2. Especifique los valores de **SID de paquete** y **Clave de seguridad**.
3. Seleccione **Guardar**.

   ![Captura de pantalla que muestra los cuadros de SID del paquete y Clave de seguridad](./media/notification-hubs-windows-store-dotnet-get-started/notification-hub-configure-wns.png)

Para más información, consulte [Envío de notificaciones a aplicaciones de Plataforma universal de Windows mediante Azure Notification Hubs](notification-hubs-windows-store-dotnet-get-started-wns-push-notification.md).

## <a name="microsoft-push-notification-service-for-windows-phone"></a>Servicio de notificaciones push de Microsoft para Windows Phone

Para configurar el Servicio de notificaciones push de Microsoft (MPNS) para Windows Phone:

1. En Azure Portal, en la página **Centro de notificaciones**, seleccione **Windows Phone (MPNS)** en el menú izquierdo.
1. Habilite las notificaciones push sin autenticar o autenticadas:

   a. Para habilitar las notificaciones push no autenticadas, seleccione **Habilitar notificaciones de inserción sin autenticar** > **Guardar**.

      ![Captura de pantalla que muestra cómo habilitar las notificaciones push no autenticadas](./media/notification-hubs-windows-phone-get-started/azure-portal-unauth.png)

   b. Para habilitar las notificaciones push no autenticadas:
      * Seleccione **Cargar certificado** en la barra de herramientas.
      * Seleccione el icono de archivo y el archivo de certificado.
      * Escriba la contraseña del certificado.
      * Seleccione **Aceptar**.
      * En la página **Windows Phone (MPNS)** , seleccione **Guardar**.

Para más información, consulte [Notificaciones push a aplicaciones de Windows Phone mediante Notification Hubs](notification-hubs-windows-mobile-push-notifications-mpns.md).

## <a name="baidu-android-china"></a>Baidu (Android China)

Para configurar las notificaciones push para Baidu:

1. En Azure Portal, en la página **Centro de notificaciones**, seleccione **Baidu (Android China)** en el menú izquierdo.
2. Escriba la **clave de API** que ha obtenido en la consola de Baidu en el proyecto Baidu Cloud Push.
3. Escriba la **clave secreta** que ha obtenido en la consola de Baidu, en el proyecto Baidu Cloud Push.
4. Seleccione **Guardar**.

    ![Captura de pantalla de Notification Hubs que muestra la configuración de Baidu (Android China) para las notificaciones push](./media/notification-hubs-baidu-get-started/AzureNotificationServicesBaidu.png)

Cuando haya completado estos pasos, una alerta le indica que el centro de notificaciones se ha actualizado correctamente. El botón **Save** (Guardar) está deshabilitado.

Para más información, consulte [Introducción a Notification Hubs con Baidu](notification-hubs-baidu-china-android-notifications-get-started.md).

## <a name="next-steps"></a>Pasos siguientes

En este inicio rápido, ha aprendido cómo configurar los valores para los sistemas de notificaciones de distintas plataformas para un centro de notificaciones en Azure Portal.

Para más información acerca de cómo enviar notificaciones push a varias plataformas, consulte estos tutoriales:

* [Envío de notificaciones push a aplicaciones iOS mediante Azure Notification Hubs](ios-sdk-get-started.md)
* [Envío de notificaciones a dispositivos Android mediante Notification Hubs y Google FCM](notification-hubs-android-push-notification-google-fcm-get-started.md)
* [Envío de notificaciones a una aplicación de UWP que se ejecuta en un dispositivo Windows](notification-hubs-windows-store-dotnet-get-started-wns-push-notification.md)
* [Envío de notificaciones a una aplicación Windows Phone 8 mediante MPNS](notification-hubs-windows-mobile-push-notifications-mpns.md)
* [Envío de notificaciones mediante Notification Hubs y Baidu Cloud Push](notification-hubs-baidu-china-android-notifications-get-started.md)
