---
title: Creación de una aplicación de Azure IoT Central | Microsoft Docs
description: Cree una nueva aplicación de Azure IoT Central. Cree la aplicación mediante el plan de precios gratuito o uno de los planes de precios estándar.
author: viv-liu
ms.author: viviali
ms.date: 07/30/2020
ms.topic: quickstart
ms.service: iot-central
services: iot-central
ms.custom: mvc
manager: corywink
ms.openlocfilehash: 4b939505f807385f235def2606d0f29564f5d08f
ms.sourcegitcommit: 1b2d1755b2bf85f97b27e8fbec2ffc2fcd345120
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/04/2020
ms.locfileid: "87552584"
---
# <a name="create-an-azure-iot-central-application"></a>Creación de una aplicación de Azure IoT Central

En este inicio rápido se muestra cómo crear una aplicación de Azure IoT Central.

## <a name="create-an-application"></a>Crear una aplicación

Navegue al sitio de [Compilación Azure IoT Central](https://aka.ms/iotcentral). Después, inicie sesión con una cuenta Microsoft personal, profesional o educativa.

Puede crear una nueva aplicación desde la lista de plantillas de IoT Central relevantes del sector, lo que le ayudaría a empezar a trabajar rápidamente, o bien partir de cero con una plantilla de **Aplicaciones personalizadas**. En este inicio rápido se usa la plantilla **Aplicación personalizada**.

Para crear una aplicación de Azure IoT Central a partir de la plantilla **Aplicación personalizada**:

1. Vaya a la página **Compilar**:

    ![Página Compilación de una aplicación de IoT](media/quick-deploy-iot-central/iotcentralcreate-new-application.png)

1. Elija **Aplicaciones personalizadas** y asegúrese de que está seleccionada la plantilla **Aplicación personalizada**.

1. Azure IoT Central sugiere automáticamente un **nombre de aplicación** en función de la plantilla de aplicación que ha seleccionado. Puede usar este nombre o escribir su propio nombre descriptivo de la aplicación.

1. Azure IoT Central también genera un único prefijo de **dirección URL de aplicación**, según el nombre de la aplicación. Esta dirección URL se usa para tener acceso a la aplicación. Si quiere, cambie el prefijo de esta URL por algo más fácil de recordar.

    ![Página de creación de una aplicación de Azure IoT Central](media/quick-deploy-iot-central/iotcentralcreate-custom.png)

    ![Información acerca de la facturación de Azure IoT Central](media/quick-deploy-iot-central/iotcentralcreate-billinginfo.png)

    > [!NOTE]
    > Si ha eligió **Aplicación personalizada** en la página anterior, verá la lista desplegable **Plantilla de aplicación**. Es posible que la muestre otras plantillas que la organización ha puesto a su disposición. 

    >[!IMPORTANT]
    >La plantilla **Aplicación personalizada (heredada)** (V2) se ha retirado, ya que todas las funcionalidades disponibles previamente en la plantilla de aplicación heredada ya están disponibles en la plantilla **Aplicación personalizada** (V3), que es más reciente. 
    
1. Elija si desea crear esta aplicación mediante el plan de precios de evaluación gratuita durante siete días o uno de los planes de precios Estándar:

    - Las aplicaciones que se crean mediante el plan *gratuito* no tienen costo durante siete días y admiten un máximo de cinco dispositivos. En cualquier momento antes de que expiren puede convertirlas para que usen un plan de precios estándar.
    - Las aplicaciones que se crean con un plan *estándar* se facturan por dispositivo y se pueden elegir el plan de precios **Estándar 1** o **Estándar 2** y los dos primeros dispositivos son gratis. Obtenga más información sobre los planes de precios gratuito y estándar en la [página de precios de Azure IoT Central](https://azure.microsoft.com/pricing/details/iot-central/). Si crea una aplicación que use un plan de precios estándar, tendrá que seleccionar su *directorio*, *suscripción de Azure* y *ubicación*:
        - *Directorio* es la instancia de Azure Active Directory en que se crea la aplicación. Azure Active Directory contiene identidades de usuario, credenciales y otra información de la organización. Si no tiene Azure Active Directory, se crea uno automáticamente al crear una suscripción de Azure.
        - Una *suscripción de Azure* permite crear instancias de los servicios de Azure. IoT Central aprovisiona los recursos de su suscripción. Si no tiene una suscripción a Azure, puede crear la de forma gratuita en la [página de suscripción a Azure](https://aka.ms/createazuresubscription). Después de crear la suscripción a Azure, vuelva a la página **Nueva aplicación**. La nueva suscripción aparece en el cuadro de lista desplegable **Suscripción de Azure**.
        - *Ubicación* es la [zona geográfica](https://azure.microsoft.com/global-infrastructure/geographies/) en la que desea crear la aplicación. Normalmente, se debe elegir la ubicación más cercana físicamente a los dispositivos para un rendimiento óptimo. Una vez que elija una ubicación, no puede mover la aplicación a otra más adelante.

1. Revise los Términos y Condiciones y seleccione **Crear** en la parte inferior de la página. Después de unos minutos, IoT Central aplicación está lista para usarse:

    ![Aplicación de Azure IoT Central](media/quick-deploy-iot-central/iotcentral-application.png)

## <a name="next-steps"></a>Pasos siguientes

En esta guía de inicio rápido, ha creado una aplicación de IoT Central. Este es el siguiente paso sugerido para obtener más información sobre IoT Central:

> [!div class="nextstepaction"]
> [Agregar un dispositivo simulado a la aplicación IoT Central](./quick-create-simulated-device.md)

Si es un desarrollador de dispositivos y desea profundizar en algún código, el siguiente paso sugerido es:
> [!div class="nextstepaction"]
> [Creación y conexión de un aplicación cliente a una aplicación de Azure IoT Central](./tutorial-connect-device-nodejs.md)
