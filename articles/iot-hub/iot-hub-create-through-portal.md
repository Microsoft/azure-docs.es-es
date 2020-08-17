---
title: Uso de Azure Portal para crear un centro de IoT | Microsoft Docs
description: Describe sobre cómo crear, administrar y eliminar los centros de IoT Hub de Azure a través de Azure Portal. Incluye información sobre los niveles de precios, el escalado, la seguridad y la configuración de la mensajería.
author: robinsh
ms.service: iot-hub
services: iot-hub
ms.topic: conceptual
ms.date: 09/06/2018
ms.author: robinsh
ms.custom:
- 'Role: Cloud Development'
ms.openlocfilehash: 8f7555129f5f514d1954ac56cf87872a85ca91d0
ms.sourcegitcommit: a76ff927bd57d2fcc122fa36f7cb21eb22154cfa
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/28/2020
ms.locfileid: "87326739"
---
# <a name="create-an-iot-hub-using-the-azure-portal"></a>Creación de una instancia de IoT Hub mediante Azure Portal

[!INCLUDE [iot-hub-resource-manager-selector](../../includes/iot-hub-resource-manager-selector.md)]

En este artículo se describe cómo crear y administrar centros de IoT mediante [Azure Portal](https://portal.azure.com).

Para seguir los pasos de este tutorial, necesita una suscripción a Azure. Si no tiene una suscripción a Azure, cree una [cuenta gratuita](https://azure.microsoft.com/free/?WT.mc_id=A261C142F) antes de empezar.

## <a name="create-an-iot-hub"></a>Crear un centro de IoT

[!INCLUDE [iot-hub-include-create-hub](../../includes/iot-hub-include-create-hub.md)]

## <a name="change-the-settings-of-the-iot-hub"></a>Cambio de la configuración del Centro de IoT

Puede cambiar la configuración de un centro de IoT existente después de crearlo desde el panel IoT Hub (Centro de IoT).

![Captura de pantalla que muestra la configuración de IoT Hub](./media/iot-hub-create-through-portal/iot-hub-settings-panel.png)

Estas son algunas de las propiedades que puede establecer para un centro de IoT:

**Pricing and scale** (Precios y escala): puede usar esta propiedad para migrar a un nivel diferente o establecer el número de unidades de IoT Hub. 

**Operations monitoring** (Supervisión de operaciones): active o desactive las distintas categorías de supervisión, como el registro de eventos relacionados con los mensajes del dispositivo a la nube o con los mensajes de la nube al dispositivo.

**IP Filter** (Filtro de IP): especifique un intervalo de direcciones IP que el centro de IoT va a aceptar o rechazar.

**Propiedades**: proporciona la lista de propiedades que puede copiar y usar en otra parte, como el identificador de recursos, el grupo de recursos, la ubicación, etc.

### <a name="shared-access-policies"></a>Directivas de acceso compartido

También puede ver o modificar la lista de directivas de acceso compartido; para ello, haga clic en **Shared access policies** (Directivas de acceso compartido) en la sección **Settings** (Configuración). Estas directivas definen los permisos que tienen los dispositivos y servicios para conectarse a IoT Hub. 

Haga clic en **Add** (Agregar) para abrir la hoja **Add a shared access policy** (Agregar una directiva de acceso compartido).  Aquí puede escribir el nombre de la nueva directiva y los permisos que quiere asociar a esta directiva, tal como se muestra en la siguiente ilustración:

![Captura de pantalla muestra la adición de una directiva de acceso compartido](./media/iot-hub-create-through-portal/iot-hub-add-shared-access-policy.png)

* Las directivas **Lectura del Registro** y **Escritura del Registro** conceden derechos de acceso de lectura y escritura para el registro de identidades. Estos permisos se usan en los servicios de back-end en la nube para administrar las identidades de los dispositivos. Si selecciona la opción de escritura, se elegirá automáticamente la opción de lectura.

* La directiva **Conexión del servicio** concede permiso de acceso a los puntos de conexión de servicio. Los servicios de back-end en la nube usan este permiso para enviar y recibir mensajes de los dispositivos, así como para actualizar y leer los datos del dispositivo gemelo y del módulo gemelo.

* La directiva **Conexión del dispositivo** concede permisos para enviar y recibir mensajes con los puntos de conexión del dispositivo de IoT Hub. Los dispositivos usan este permiso para enviar y recibir mensajes desde un centro de IoT, actualizar y leer datos del dispositivo gemelo y del módulo gemelo y realizar cargas de archivos.

Haga clic en **Crear** para agregar la directiva recién creada a la lista existente.

Para más información sobre el acceso que concede cada permiso específico, consulte [Permisos de IoT Hub](./iot-hub-devguide-security.md#iot-hub-permissions).

## <a name="register-a-new-device-in-the-iot-hub"></a>Registro de un nuevo dispositivo en el centro de IoT

[!INCLUDE [iot-hub-include-create-device](../../includes/iot-hub-include-create-device.md)]

## <a name="message-routing-for-an-iot-hub"></a>Enrutamiento de mensajes para un centro de IoT

Haga clic en **Message Routing** (Enrutamiento de mensajes) en **Messaging** (Mensajería) para ver el panel de enrutamiento de mensajes, donde puede definir rutas y puntos de conexión personalizados para el centro. [Message routing](iot-hub-devguide-messages-d2c.md) (Enrutamiento de mensajes) le permite administrar cómo se envían los datos desde los dispositivos hasta los puntos de conexión. El primer paso es agregar una nueva ruta. Luego, puede agregar un punto de conexión existente a la ruta o crear uno de los tipos admitidos, como almacenamiento de blobs. 

![Panel de enrutamiento de mensajes](./media/iot-hub-create-through-portal/iot-hub-message-routing.png)

### <a name="routes"></a>Rutas

La pestaña de rutas es la primera pestaña del panel de enrutamiento de mensajes. Para agregar una nueva ruta, haga clic en **+Add** (+Agregar). Verá la siguiente pantalla: 

![Captura de pantalla que muestra la incorporación de una nueva ruta](./media/iot-hub-create-through-portal/iot-hub-add-route-storage-endpoint.png)

Escriba un nombre para el centro. El nombre debe ser único dentro de la lista de rutas de ese centro. 

En **Endpoint** (Punto de conexión) puede seleccionar uno de la lista desplegable o agregar uno nuevo. En este ejemplo, ya están disponibles una cuenta de almacenamiento y un contenedor. Para agregarlos como punto de conexión, haga clic en **+Add** (+Agregar) junto a la lista desplegable de puntos de conexión y seleccione **Blob Storage**. En la pantalla siguiente se muestra dónde se especifican la cuenta de almacenamiento y el contenedor.

![Captura de pantalla que muestra la incorporación de un punto de conexión de almacenamiento para la regla de enrutamiento](./media/iot-hub-create-through-portal/iot-hub-routing-add-storage-endpoint.png)

Haga clic en **Pick a container** (Seleccionar un contenedor) para seleccionar la cuenta de almacenamiento y el contenedor. Cuando haya seleccionado esos campos, volverá al panel del punto de conexión. Use los valores predeterminados en el resto de los campos y **Create** (Crear) para crear el punto de conexión de la cuenta de almacenamiento y agregarlo a las reglas de enrutamiento.

En **Data source** (Origen de datos), seleccione Device Telemetry Messages (Mensajes de telemetría de dispositivo). 

A continuación, agregue una consulta de enrutamiento. En este ejemplo, los mensajes que tienen una propiedad de la aplicación denominada `level` con un valor igual a `critical` se enrutan a la cuenta de almacenamiento.

![Captura de pantalla que muestra cómo se guarda una nueva regla de enrutamiento](./media/iot-hub-create-through-portal/iot-hub-add-route.png)

Haga clic en **Save** (Guardar) para guardar la regla de enrutamiento. Volverá al panel de enrutamiento de mensajes y se mostrará la nueva regla de enrutamiento.

### <a name="custom-endpoints"></a>Puntos de conexión personalizados

Haga clic en la pestaña **Custom endpoints** (Puntos de conexión personalizados). Verá los puntos de conexión personalizados ya creados. Desde aquí, puede agregar nuevos puntos de conexión o eliminar los existentes. 

> [!NOTE]
> Si elimina una ruta, no elimina los puntos de conexión asignados a ella. Para eliminar un punto de conexión, haga clic en la pestaña de puntos de conexión personalizados, seleccione el punto de conexión que quiere eliminar y haga clic en el botón para eliminar.
>

Puede leer más sobre los puntos de conexión personalizados en [Referencia: Puntos de conexión de IoT Hub](iot-hub-devguide-endpoints.md).

Puede definir hasta 10 puntos de conexión personalizados para un centro de IoT. 

Para ver un ejemplo completo de cómo usar puntos de conexión personalizados con el enrutamiento, consulte [Enrutamiento de mensajes con IoT Hub](tutorial-routing.md).

## <a name="find-a-specific-iot-hub"></a>Búsqueda de un centro de IoT específico

Existen dos maneras de buscar un centro de IoT específico en su suscripción:

1. Si sabe el grupo de recursos al que pertenece, haga clic en **Resource groups** (Grupos de recursos) y seleccione el grupo de recursos de la lista. En la pantalla del grupo de recursos se muestran todos los recursos de ese grupo, incluidos los centros de IoT. Haga clic en el centro que busca.

2. Haga clic en **All resources** (Todos los recursos). En el panel **All resources** (Todos los recursos), hay una lista desplegable que acepta como valor predeterminado `All types`. Haga clic en la lista desplegable y desactive la opción `Select all`. Busque `IoT Hub` y márquelo. Haga clic en el cuadro de lista desplegable para cerrarlo; las entradas se filtran y solo se muestran sus centros de IoT.

## <a name="delete-the-iot-hub"></a>Eliminación del Centro de IoT

Para eliminar un centro de Iot, busque el que quiere eliminar y luego haga clic en el botón **Delete** (Eliminar) debajo de su nombre.

## <a name="next-steps"></a>Pasos siguientes

Siga estos vínculos para más información sobre la administración de Azure IoT Hub:

* [Enrutamiento de mensajes con IoT Hub](tutorial-routing.md)
* [Métricas de IoT Hub](iot-hub-metrics.md)
* [Supervisión de operaciones](iot-hub-operations-monitoring.md)