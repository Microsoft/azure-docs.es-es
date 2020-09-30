---
title: Cambiar la configuración de la aplicación de Azure IoT Central | Microsoft Docs
description: Como administrador, aprenderá a administrar su aplicación de Azure IoT Central al cambiar el nombre de la aplicación o la dirección URL, cargar una imagen y eliminar una aplicación.
author: viv-liu
ms.author: viviali
ms.date: 11/27/2019
ms.topic: how-to
ms.service: iot-central
services: iot-central
manager: peterpr
ms.openlocfilehash: 37a3725fb7684a9e87f1936ef0ce8cc79de03894
ms.sourcegitcommit: 1b320bc7863707a07e98644fbaed9faa0108da97
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/09/2020
ms.locfileid: "89594669"
---
# <a name="change-iot-central-application-settings"></a>Cambiar la configuración de la aplicación de IoT Central



En este artículo se describe cómo un administrador puede administrar la aplicación al cambiar el nombre de la aplicación y la dirección URL, cargar la imagen y eliminar una aplicación en la aplicación de Azure IoT Central.

Para tener acceso a la sección **Administración** y poder usarla, debe disponer del rol **Administrador** para la aplicación Azure IoT Central. Si crea una aplicación Azure IoT Central, se le asigna automáticamente el rol **Administrator** para esa aplicación.

## <a name="change-application-name-and-url"></a>Cambio del nombre y la dirección URL de la aplicación

En la página **Configuración de la aplicación** puede cambiar el nombre y la dirección URL de la aplicación, después, seleccione **Guardar**.

![Página Configuración de la aplicación](media/howto-administer/image0-a.png)

Si el administrador crea un tema personalizado para la aplicación, esta página incluye una opción para ocultar el **nombre de la aplicación** en la interfaz de usuario. Esta opción resulta útil si el logotipo de la aplicación en el tema personalizado incluye el nombre de la aplicación. Para más información, consulte [Personalización de la interfaz de usuario de Azure IoT Central](./howto-customize-ui.md).

> [!Note]
> Si cambia la dirección URL, la dirección URL anterior la puede tomar otro cliente de Azure IoT Central. En ese caso, usted ya no podrá usarla. Cuando cambie la dirección URL, la dirección antigua ya no funcionará y tendrá que notificar a los usuarios la nueva URL que deben usar.

## <a name="delete-an-application"></a>Eliminar una aplicación

Use el botón **Eliminar** para quitar permanentemente la aplicación de IoT Central. Con esta acción se eliminan de manera permanente todos los datos asociados a la aplicación.

> [!Note]
> Para eliminar una aplicación, también debe tener permisos para eliminar recursos en la suscripción de Azure que haya elegido al crear la aplicación. Para obtener más información, vea el artículo sobre el [uso del control de acceso basado en rol para administrar el acceso a los recursos de la suscripción de Azure](https://docs.microsoft.com/azure/active-directory/role-based-access-control-configure).

## <a name="manage-programmatically"></a>Administración de manera programática

Hay paquetes de SDK de Azure Resource Manager SDK de IoT Central disponibles para Node, Python, C#, Ruby, Java y Go. Puede usar estos paquetes para crear, enumerar, actualizar o eliminar aplicaciones de IoT Central. Los paquetes incluyen aplicaciones auxiliares para administrar la autenticación y el control de errores.

Encontrará ejemplos de cómo usar los SDK de Azure Resource Manager en [https://github.com/Azure-Samples/azure-iot-central-arm-sdk-samples](https://github.com/Azure-Samples/azure-iot-central-arm-sdk-samples).

Para obtener más información, consulte los siguientes repositorios y paquetes de GitHub:

| Idioma | Repositorio | Paquete |
| ---------| ---------- | ------- |
| Nodo | [https://github.com/Azure/azure-sdk-for-node](https://github.com/Azure/azure-sdk-for-node) | [https://www.npmjs.com/package/azure-arm-iotcentral](https://www.npmjs.com/package/azure-arm-iotcentral)
| Python |[https://github.com/Azure/azure-sdk-for-python](https://github.com/Azure/azure-sdk-for-python) | [https://pypi.org/project/azure-mgmt-iotcentral](https://pypi.org/project/azure-mgmt-iotcentral)
| C# | [https://github.com/Azure/azure-sdk-for-net](https://github.com/Azure/azure-sdk-for-net) | [https://www.nuget.org/packages/Microsoft.Azure.Management.IotCentral](https://www.nuget.org/packages/Microsoft.Azure.Management.IotCentral)
| Ruby | [https://github.com/Azure/azure-sdk-for-ruby](https://github.com/Azure/azure-sdk-for-ruby) | [https://rubygems.org/gems/azure_mgmt_iot_central](https://rubygems.org/gems/azure_mgmt_iot_central)
| Java | [https://github.com/Azure/azure-sdk-for-java](https://github.com/Azure/azure-sdk-for-java) | [https://search.maven.org/search?q=a:azure-mgmt-iotcentral](https://search.maven.org/search?q=a:azure-mgmt-iotcentral)
| Go | [https://github.com/Azure/azure-sdk-for-go](https://github.com/Azure/azure-sdk-for-go) | [https://github.com/Azure/azure-sdk-for-go](https://github.com/Azure/azure-sdk-for-go)

## <a name="next-steps"></a>Pasos siguientes

Ahora que ha aprendido a administrar la aplicación de Azure IoT Central, el siguiente paso sugerido es aprender a [administrar usuarios y roles](howto-manage-users-roles.md) en Azure IoT Central.
