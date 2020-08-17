---
title: Compatibilidad de Azure IoT Hub con redes virtuales
description: Cómo usar el patrón de conectividad de redes virtuales con IoT Hub
services: iot-hub
author: jlian
ms.service: iot-fundamentals
ms.topic: conceptual
ms.date: 06/16/2020
ms.author: jlian
ms.openlocfilehash: 7776345ee4e02baa2d2a6ae7bc08389aa94bd09b
ms.sourcegitcommit: 3d56d25d9cf9d3d42600db3e9364a5730e80fa4a
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/03/2020
ms.locfileid: "87534488"
---
# <a name="iot-hub-support-for-virtual-networks-with-private-link-and-managed-identity"></a>Compatibilidad de IoT Hub con redes virtuales mediante Private Link e identidad administrada

De forma predeterminada, los nombres de host de IoT Hub se asignan a un punto de conexión público con una dirección IP enrutable públicamente a través de Internet. Diferentes clientes comparten este punto de conexión público de IoT Hub, que es accesible para todos los dispositivos de IoT a través de redes de área extensa y de redes locales.

![Punto de conexión público de IoT Hub](./media/virtual-network-support/public-endpoint.png)

De forma similar, las características de IoT Hub, entre las que se incluyen el [enrutamiento de mensajes](./iot-hub-devguide-messages-d2c.md), [carga de archivos](./iot-hub-devguide-file-upload.md) e [importación/exportación en bloque de dispositivos](./iot-hub-bulk-identity-mgmt.md), requieren conectividad de IoT Hub a un recurso de Azure propiedad del cliente a través de su punto de conexión público. Estas rutas de conectividad conforman colectivamente el tráfico de salida de IoT Hub a los recursos del cliente.

Es posible que desee restringir la conectividad a los recursos de Azure (incluido IoT Hub) para que solo se realice a través de una red virtual de su propiedad y bajo su control. Estas razones incluyen las siguientes:

* Introducción del aislamiento de red para IoT Hub evitando la exposición de la conectividad a la red pública de Internet.

* Habilitación de una experiencia de conectividad privada desde los recursos de la red local, lo que garantiza que sus datos y tráfico se transmiten directamente a la red troncal de Azure.

* Prevención de ataques de exfiltración de redes locales confidenciales. 

* Seguimiento de los patrones de conectividad generales de Azure establecidos mediante [puntos de conexión privados](../private-link/private-endpoint-overview.md).

En este artículo se describe cómo lograr estos objetivos mediante el uso de [Azure Private Link](../private-link/private-link-overview.md) para la conectividad de entrada a IoT Hub y el uso de la excepción de servicios de Microsoft de confianza para la conectividad de salida de IoT Hub a otros recursos de Azure.

## <a name="ingress-connectivity-to-iot-hub-using-azure-private-link"></a>Conectividad de entrada a IoT Hub mediante Azure Private Link

Un punto de conexión privado es una dirección IP privada asignada en una red virtual propiedad del cliente a través de la cual se puede acceder a un recurso de Azure. Con Azure Private Link, puede configurar un punto de conexión privado para su instancia de IoT Hub, para permitir así que los servicios que operan en la red virtual accedan a IoT Hub sin necesidad de enviar el tráfico al punto de conexión público de IoT Hub. Del mismo modo, los dispositivos locales pueden usar la [red privada virtual (VPN)](../vpn-gateway/vpn-gateway-about-vpngateways.md) o el emparejamiento de [ExpressRoute](https://azure.microsoft.com/services/expressroute/) para obtener conectividad con la red virtual y la instancia de IoT Hub (a través de su punto de conexión privado). Como resultado, puede restringir o bloquear por completo la conectividad a los puntos de conexión públicos de su instancia de IoT Hub mediante el uso de [filtros IP de IoT Hub](./iot-hub-ip-filtering.md) y la [configuración del enrutamiento para que no envíe datos al punto de conexión integrado](#built-in-event-hub-compatible-endpoint-doesnt-support-access-over-private-endpoint). Este enfoque mantiene la conectividad con la instancia de IoT Hub usando el punto de conexión privado para los dispositivos. El objetivo principal de esta configuración es para los dispositivos de una red local. No se recomienda esta configuración para los dispositivos implementados en una red de área extensa.

![Punto de conexión público de IoT Hub](./media/virtual-network-support/virtual-network-ingress.png)

Antes de continuar, asegúrese de que se cumplen los requisitos previos siguientes:

* Ha [creado una red virtual de Azure](../virtual-network/quick-create-portal.md) con una subred en la que se creará el punto de conexión privado.

* En el caso de los dispositivos que operan en redes locales, configure la [red privada virtual (VPN)](../vpn-gateway/vpn-gateway-about-vpngateways.md) o el emparejamiento privado de [ExpressRoute](https://azure.microsoft.com/services/expressroute/) en su red virtual de Azure.

### <a name="set-up-a-private-endpoint-for-iot-hub-ingress"></a>Configuración de un punto de conexión privado para la entrada de IoT Hub

El punto de conexión privado funciona para API de dispositivo de IoT Hub (como los mensajes de dispositivo a nube), así como para API de servicio (como la creación y la actualización de dispositivos).

1. En Azure Portal, seleccione **Redes**, **Conexiones de punto de conexión privado** y haga clic en **+ Punto de conexión privado**.

    :::image type="content" source="media/virtual-network-support/private-link.png" alt-text="Captura de pantalla que muestra dónde agregar un punto de conexión privado para IoT Hub":::

1. Indique la suscripción, el grupo de recursos, el nombre y la región donde desea crear el nuevo punto de conexión privado. Lo más adecuado es que el punto de conexión privado se cree en la misma región que el centro de conectividad.

1. Haga clic en **Siguiente: Recurso** y proporcione la suscripción para el recurso de IoT Hub; y seleccione **"Microsoft.Devices/IotHubs"** como tipo de recurso, su nombre de IoT Hub como **recurso** e **iotHub** como subrecurso de destino.

1. Haga clic en **Siguiente: Configuración** y proporcione la red virtual y la subred donde se creará el punto de conexión privado. Si lo desea, seleccione la opción para la integración con la zona DNS privada de Azure.

1. Haga clic en **Siguiente: Etiquetas** y, de manera opcional, proporcione las etiquetas del recurso.

1. Haga clic en **Revisar y crear** para crear el recurso de Private Link.

### <a name="built-in-event-hub-compatible-endpoint-doesnt-support-access-over-private-endpoint"></a>El punto de conexión integrado compatible con Event Hubs no admite el acceso a través de un punto de conexión privado

El [punto de conexión integrado compatible con Event Hubs](iot-hub-devguide-messages-read-builtin.md) no admite el acceso a través de un punto de conexión privado. Cuando se configura un punto de conexión privado de IoT Hub, es solo para la conectividad de entrada. El consumo de datos del punto de conexión integrado compatible con Event Hubs solo se puede realizar a través de la red pública de Internet. 

El [filtro IP](iot-hub-ip-filtering.md) de IoT Hub tampoco controla el acceso público al punto de conexión integrado. Para bloquear por completo el acceso a la red pública a su instancia de IoT Hub, debe hacer lo siguiente: 

1. Configurar el acceso del punto de conexión privado para IoT Hub
1. [Desactivar el acceso a la red pública](iot-hub-public-network-access.md) o usar el filtro IP para bloquear todas las direcciones IP
1. Dejar de usar el punto de conexión integrado de Event Hubs mediante la [configuración del enrutamiento para que no le envíe datos](iot-hub-devguide-messages-d2c.md)
1. Desactivar la [ruta de reserva](iot-hub-devguide-messages-d2c.md#fallback-route)
1. Configurar la salida a otros recursos de Azure mediante un [servicio de confianza de Microsoft](#egress-connectivity-from-iot-hub-to-other-azure-resources)

### <a name="pricing-for-private-link"></a>Precios de Private Link

Para más información sobre los precios, consulte [Precios de Azure Private Link](https://azure.microsoft.com/pricing/details/private-link).

## <a name="egress-connectivity-from-iot-hub-to-other-azure-resources"></a>Conectividad de salida de IoT Hub a otros recursos de Azure

IoT Hub puede conectar con los recursos de Azure Blob Storage, Event Hubs, Service Bus para el [enrutamiento de mensajes](./iot-hub-devguide-messages-d2c.md), la [carga de archivos](./iot-hub-devguide-file-upload.md) y la [importación/exportación en bloque de dispositivos](./iot-hub-bulk-identity-mgmt.md) a través del punto de conexión público de los recursos. Al enlazar el recurso a una red virtual, se bloquea la conectividad al recurso de forma predeterminada. Como resultado, esta configuración impide que las instancias de IoT Hub trabajen en el envío de datos a los recursos. Para corregir este problema, habilite la conectividad desde el recurso de IoT Hub a su recurso de cuenta de almacenamiento, Event Hubs o Service Bus a través de la opción de **servicio de confianza de Microsoft**.

### <a name="turn-on-managed-identity-for-iot-hub"></a>Activación de la identidad administrada para IoT Hub

Para que otros servicios puedan encontrar la instancia de IoT Hub como un servicio de confianza de Microsoft, la instancia debe tener una identidad administrada asignada por el sistema.

1. Vaya a **Identidad** en el portal de IoT Hub.

1. En **Estado**, seleccione **Activado** y, a continuación, haga clic en **Guardar**.

    :::image type="content" source="media/virtual-network-support/managed-identity.png" alt-text="Captura de pantalla que muestra cómo activar la identidad administrada para IoT Hub":::

### <a name="assign-managed-identity-to-your-iot-hub-at-creation-time-using-arm-template"></a>Asignación de identidad administrada a IoT Hub en el momento de la creación con la plantilla de ARM

Para asignar una identidad administrada a su instancia de IoT Hub en el momento del aprovisionamiento de recursos, use la plantilla de ARM siguiente:

```json
{
  "$schema": "https://schema.management.azure.com/schemas/2015-01-01/deploymentTemplate.json#",
  "contentVersion": "1.0.0.0",
  "resources": [
    {
      "type": "Microsoft.Devices/IotHubs",
      "apiVersion": "2020-03-01",
      "name": "<provide-a-valid-resource-name>",
      "location": "<any-of-supported-regions>",
      "identity": {
        "type": "SystemAssigned"
      },
      "sku": {
        "name": "<your-hubs-SKU-name>",
        "tier": "<your-hubs-SKU-tier>",
        "capacity": 1
      }
    },
    {
      "type": "Microsoft.Resources/deployments",
      "apiVersion": "2018-02-01",
      "name": "updateIotHubWithKeyEncryptionKey",
      "dependsOn": [
        "<provide-a-valid-resource-name>"
      ],
      "properties": {
        "mode": "Incremental",
        "template": {
          "$schema": "https://schema.management.azure.com/schemas/2019-04-01/deploymentTemplate.json#",
          "contentVersion": "0.9.0.0",
          "resources": [
            {
              "type": "Microsoft.Devices/IotHubs",
              "apiVersion": "2020-03-01",
              "name": "<provide-a-valid-resource-name>",
              "location": "<any-of-supported-regions>",
              "identity": {
                "type": "SystemAssigned"
              },
              "sku": {
                "name": "<your-hubs-SKU-name>",
                "tier": "<your-hubs-SKU-tier>",
                "capacity": 1
              }
            }
          ]
        }
      }
    }
  ]
}
```

Después de sustituir los valores del recurso `name`, `location`, `SKU.name` y `SKU.tier`, puede usar la CLI de Azure para implementar el recurso en un grupo de recursos existente mediante:

```azurecli-interactive
az deployment group create --name <deployment-name> --resource-group <resource-group-name> --template-file <template-file.json>
```

Una vez creado el recurso, puede recuperar la instancia de Managed Service Identity asignada al concentrador mediante la CLI de Azure:

```azurecli-interactive
az resource show --resource-type Microsoft.Devices/IotHubs --name <iot-hub-resource-name> --resource-group <resource-group-name>
```

### <a name="pricing-for-managed-identity"></a>Precios de la identidad administrada

La característica de excepción de servicios de confianza de Microsoft es gratuita. Los cargos por las cuentas de almacenamiento aprovisionadas, las instancias de Event Hubs o los recursos de Service Bus se aplican por separado.

### <a name="egress-connectivity-to-storage-account-endpoints-for-routing"></a>Conectividad de salida a los puntos de conexión de la cuenta de almacenamiento para el enrutamiento

IoT Hub puede enrutar mensajes a una cuenta de almacenamiento propiedad del cliente. Para permitir que la funcionalidad de enrutamiento acceda a una cuenta de almacenamiento mientras se aplican las restricciones de firewall, IoT Hub debe tener una [identidad administrada](#turn-on-managed-identity-for-iot-hub). Después de aprovisionar una identidad administrada, realice los pasos siguientes para conceder el permiso RBAC a la identidad de recursos de la instancia de IoT Hub para acceder a su cuenta de almacenamiento.

1. En Azure Portal, vaya a la pestaña **Control de acceso (IAM)** de la cuenta de almacenamiento y haga clic en **Agregar** en la sección **Agregar una asignación de roles**.

2. Seleccione **Colaborador de datos de Storage Blob** ([*no* Colaborador ni Colaborador de la cuenta de almacenamiento](../storage/common/storage-auth-aad-rbac-portal.md#azure-roles-for-blobs-and-queues)) como **rol**, el **usuario, grupo o entidad de servicio de Azure AD** en **Asignando acceso a** y seleccione el nombre del recurso de IoT Hub en la lista desplegable. Haga clic en el botón **Save** (Guardar).

3. Vaya a la pestaña **Firewalls y redes virtuales** de la cuenta de almacenamiento y habilite la opción **Allow access from selected networks** (Permitir acceso desde redes seleccionadas). En la lista **Excepciones**, active la casilla situada junto a **Permitir que los servicios de Microsoft de confianza accedan a esta cuenta de almacenamiento**. Haga clic en el botón **Save** (Guardar).

4. En la página de recursos de IoT Hub, vaya a la pestaña **Enrutamiento de mensajes**.

5. Vaya a la sección **Puntos de conexión personalizados** y haga clic en **Agregar**. Seleccione **Almacenamiento** como el tipo de punto de conexión.

6. En la página que aparece, proporcione un nombre para el punto de conexión, seleccione el contenedor que quiere usar en Blob Storage, y proporcione la codificación y el formato de nombre de archivo. Seleccione **Asignado por el sistema** como **Tipo de autenticación** para su punto de conexión de almacenamiento. Haga clic en el botón **Crear**.

Ahora el punto de conexión de almacenamiento personalizado está configurado para usar la identidad asignada por el sistema del concentrador y tiene permiso para acceder al recurso de almacenamiento a pesar de las restricciones de firewall. Ahora puede usar este punto de conexión para configurar una regla de enrutamiento.

### <a name="egress-connectivity-to-event-hubs-endpoints-for-routing"></a>Conectividad de salida a los puntos de conexión de Event Hubs para el enrutamiento

IoT Hub se pueden configurar para enrutar mensajes a un espacio de nombres de Event Hubs propiedad del cliente. Para permitir que la funcionalidad de enrutamiento acceda a un recurso de Event Hubs mientras se aplican las restricciones de firewall, IoT Hub debe tener una identidad administrada. Después de crear la identidad administrada, realice los pasos siguientes para conceder el permiso RBAC a la identidad de recursos de la instancia de IoT Hub para acceder a las instancias de Event Hubs.

1. En Azure Portal, vaya a la pestaña **Control de acceso (IAM)** de su instancia de Event Hubs y haga clic en **Agregar** en la sección **Agregar una asignación de roles**.

2. Seleccione **Remitente de los datos de Azure Event Hubs** como **rol**, el **usuario, grupo o entidad de servicio de Azure AD** en **Asignando acceso a** y seleccione el nombre del recurso de IoT Hub en la lista desplegable. Haga clic en el botón **Save** (Guardar).

3. Vaya a la pestaña **Firewalls y redes virtuales** de su instancia de Event Hubs y habilite la opción **Allow access from selected networks** (Permitir acceso desde redes seleccionadas). En la lista **Excepciones**, active la casilla situada junto a **Allow trusted Microsoft services to access event hubs** (Permitir que los servicios de Microsoft de confianza accedan a esta instancia de Event Hubs). Haga clic en el botón **Save** (Guardar).

4. En la página de recursos de IoT Hub, vaya a la pestaña **Enrutamiento de mensajes**.

5. Vaya a la sección **Puntos de conexión personalizados** y haga clic en **Agregar**. Seleccione **Event Hubs** como el tipo de punto de conexión.

6. En la página que aparece, proporcione un nombre para el punto de conexión, seleccione el espacio de nombres y la instancia de Event Hubs, y haga clic en el botón **Crear**.

Ahora el punto de conexión de Event Hubs personalizado está configurado para usar la identidad asignada por el sistema del concentrador y tiene permiso para acceder al recurso de Event Hubs a pesar de las restricciones de firewall. Ahora puede usar este punto de conexión para configurar una regla de enrutamiento.

### <a name="egress-connectivity-to-service-bus-endpoints-for-routing"></a>Conectividad de salida a los puntos de conexión de Service Bus para el enrutamiento

IoT Hub se pueden configurar para enrutar mensajes a un espacio de nombres de Service Bus propiedad del cliente. Para permitir que la funcionalidad de enrutamiento acceda a un recurso de Service Bus mientras se aplican las restricciones de firewall, IoT Hub debe tener una identidad administrada. Después de aprovisionar la identidad administrada, realice los pasos siguientes para conceder el permiso RBAC a la identidad de recursos de la instancia de IoT Hub para acceder al recurso de Service Bus.

1. En Azure Portal, vaya a la pestaña **Control de acceso (IAM)** de su instancia de Service Bus y haga clic en **Agregar** en la sección **Agregar una asignación de roles**.

2. Seleccione **Remitente de los datos de Azure Service Bus** como **rol**, el **usuario, grupo o entidad de servicio de Azure AD** en **Asignando acceso a** y seleccione el nombre del recurso de IoT Hub en la lista desplegable. Haga clic en el botón **Save** (Guardar).

3. Vaya a la pestaña **Firewalls y redes virtuales** de su instancia de Service Bus y habilite la opción **Allow access from selected networks** (Permitir acceso desde redes seleccionadas). En la lista **Excepciones**, active la casilla situada junto a **Allow trusted Microsoft services to access this service bus** (Permitir que los servicios de Microsoft de confianza accedan a esta instancia de Service Bus). Haga clic en el botón **Save** (Guardar).

4. En la página de recursos de IoT Hub, vaya a la pestaña **Enrutamiento de mensajes**.

5. Vaya a la sección **Puntos de conexión personalizados** y haga clic en **Agregar**. Seleccione **Cola de Service Bus** o **Tema de Service Bus** (según corresponda) como el tipo de punto de conexión.

6. En la página que aparece, proporcione un nombre para el punto de conexión, y seleccione el espacio de nombres y la cola o el tema de Service Bus (según corresponda). Haga clic en el botón **Crear**.

Ahora el punto de conexión de Service Bus personalizado está configurado para usar la identidad asignada por el sistema del concentrador y tiene permiso para acceder al recurso de Service Bus a pesar de las restricciones de firewall. Ahora puede usar este punto de conexión para configurar una regla de enrutamiento.

### <a name="egress-connectivity-to-storage-accounts-for-file-upload"></a>Conectividad de salida a las cuentas de almacenamiento para la carga de archivos

La característica de carga de archivos de IoT Hub permite a los dispositivos cargar archivos en una cuenta de almacenamiento propiedad del cliente. Para permitir que la carga de archivos funcione, tanto los dispositivos como IoT Hub deben tener conectividad con la cuenta de almacenamiento. Si se aplican restricciones de firewall a la cuenta de almacenamiento, los dispositivos deben usar cualquiera de los mecanismos de la cuenta de almacenamiento admitida (como [puntos de conexión privados](../private-link/create-private-endpoint-storage-portal.md), [puntos de conexión de servicio](../virtual-network/virtual-network-service-endpoints-overview.md) o [configuración directa de firewall](../storage/common/storage-network-security.md)) para obtener conectividad. Del mismo modo, si se aplican restricciones de firewall en la cuenta de almacenamiento, IoT Hub se debe configurar para acceder al recurso de almacenamiento a través de la excepción de servicios de confianza de Microsoft. Con este propósito, la instancia de IoT Hub debe tener una identidad administrada. Después de aprovisionar una identidad administrada, realice los pasos siguientes para conceder el permiso RBAC a la identidad de recursos de la instancia de IoT Hub para acceder a su cuenta de almacenamiento.

1. En Azure Portal, vaya a la pestaña **Control de acceso (IAM)** de la cuenta de almacenamiento y haga clic en **Agregar** en la sección **Agregar una asignación de roles**.

2. Seleccione **Colaborador de datos de Storage Blob** ([*no* Colaborador ni Colaborador de la cuenta de almacenamiento](../storage/common/storage-auth-aad-rbac-portal.md#azure-roles-for-blobs-and-queues)) como **rol**, el **usuario, grupo o entidad de servicio de Azure AD** en **Asignando acceso a** y seleccione el nombre del recurso de IoT Hub en la lista desplegable. Haga clic en el botón **Save** (Guardar).

3. Vaya a la pestaña **Firewalls y redes virtuales** de la cuenta de almacenamiento y habilite la opción **Allow access from selected networks** (Permitir acceso desde redes seleccionadas). En la lista **Excepciones**, active la casilla situada junto a **Permitir que los servicios de Microsoft de confianza accedan a esta cuenta de almacenamiento**. Haga clic en el botón **Save** (Guardar).

4. En la página de recursos de IoT Hub, vaya a la pestaña **Carga de archivos**.

5. En la página que se muestra, seleccione el contenedor que quiere usar en Blob Storage, y establezca los valores que desee en **Configuración de notificación de archivos**, **TTL de SAS**, **TTL predeterminado** y **Número máximo de entregas**. Seleccione **Asignado por el sistema** como **Tipo de autenticación** para su punto de conexión de almacenamiento. Haga clic en el botón **Crear**.

Ahora el punto de conexión de almacenamiento para la carga de archivos está configurado para usar la identidad asignada por el sistema del concentrador y tiene permiso para acceder al recurso de almacenamiento a pesar de las restricciones de firewall.

### <a name="egress-connectivity-to-storage-accounts-for-bulk-device-importexport"></a>Conectividad de salida a las cuentas de almacenamiento para la importación/exportación masiva de dispositivos

IoT Hub admite la funcionalidad para [importar/exportar](./iot-hub-bulk-identity-mgmt.md) información de dispositivos de forma masiva desde y hacia una instancia de Storage Blob proporcionada por el cliente. Para permitir que la característica de importación/exportación masiva funcione, tanto los dispositivos como IoT Hub deben tener conectividad con la cuenta de almacenamiento.

Esta funcionalidad requiere conectividad de IoT Hub a la cuenta de almacenamiento. Para acceder a un recurso de Service Bus mientras se aplican las restricciones de firewall, IoT Hub debe tener una identidad administrada. Después de aprovisionar la identidad administrada, realice los pasos siguientes para conceder el permiso RBAC a la identidad de recursos de la instancia de IoT Hub para acceder al recurso de Service Bus.

1. En Azure Portal, vaya a la pestaña **Control de acceso (IAM)** de la cuenta de almacenamiento y haga clic en **Agregar** en la sección **Agregar una asignación de roles**.

2. Seleccione **Colaborador de datos de Storage Blob** ([*no* Colaborador ni Colaborador de la cuenta de almacenamiento](../storage/common/storage-auth-aad-rbac-portal.md#azure-roles-for-blobs-and-queues)) como **rol**, el **usuario, grupo o entidad de servicio de Azure AD** en **Asignando acceso a** y seleccione el nombre del recurso de IoT Hub en la lista desplegable. Haga clic en el botón **Save** (Guardar).

3. Vaya a la pestaña **Firewalls y redes virtuales** de la cuenta de almacenamiento y habilite la opción **Allow access from selected networks** (Permitir acceso desde redes seleccionadas). En la lista **Excepciones**, active la casilla situada junto a **Permitir que los servicios de Microsoft de confianza accedan a esta cuenta de almacenamiento**. Haga clic en el botón **Save** (Guardar).

Ahora puede usar la API REST de Azure IoT para [crear trabajos de importación y exportación](https://docs.microsoft.com/rest/api/iothub/service/jobclient/getimportexportjobs) a fin de obtener información sobre cómo usar la funcionalidad de importación y exportación en bloque. Tenga en cuenta que tendrá que proporcionar el objeto `storageAuthenticationType="identityBased"` en el cuerpo de la solicitud y usar `inputBlobContainerUri="https://..."` y `outputBlobContainerUri="https://..."` como las direcciones URL de entrada y salida de la cuenta de almacenamiento, respectivamente.

Los SDK de Azure IoT Hub también admiten esta funcionalidad en el administrador del Registro del cliente del servicio. El fragmento de código siguiente muestra cómo iniciar un trabajo de importación o exportación al usar el SDK de C#.

```csharp
// Call an import job on the IoT Hub
JobProperties importJob = 
await registryManager.ImportDevicesAsync(
  JobProperties.CreateForImportJob(inputBlobContainerUri, outputBlobContainerUri, null, StorageAuthenticationType.IdentityBased), 
  cancellationToken);

// Call an export job on the IoT Hub to retrieve all devices
JobProperties exportJob = 
await registryManager.ExportDevicesAsync(
    JobProperties.CreateForExportJob(outputBlobContainerUri, true, null, StorageAuthenticationType.IdentityBased),
    cancellationToken);
```

Para usar esta versión de los SDK de Azure IoT con compatibilidad con redes virtuales para C#, Java y Node.js:

1. Cree una variable de entorno denominada `EnableStorageIdentity` y establezca su valor en `1`.

2. Descargue el SDK:  [Java](https://aka.ms/vnetjavasdk) | [C#](https://aka.ms/vnetcsharpsdk) | [Node.js](https://aka.ms/vnetnodesdk)
 
Para Python, descargue nuestra versión limitada de GitHub.

1. Vaya a la [página de la versión de GitHub](https://aka.ms/vnetpythonsdk).

2. Descargue el archivo siguiente, que encontrará en la parte inferior de la página de la versión, bajo el encabezado denominado **assets**.
    > *azure_iot_hub-2.2.0_limited-py2.py3-none-any.whl*

3. Abra un terminal y navegue hasta la carpeta que contiene el archivo descargado.

4. Ejecute el siguiente comando para instalar el SDK de servicio de Python con compatibilidad con redes virtuales:
    > pip install ./azure_iot_hub-2.2.0_limited-py2.py3-none-any.whl


## <a name="next-steps"></a>Pasos siguientes

Use los vínculos siguientes para obtener más información sobre las características de IoT Hub:

* [Enrutamiento de mensajes](./iot-hub-devguide-messages-d2c.md)
* [Carga de archivos](./iot-hub-devguide-file-upload.md)
* [Importación/exportación masiva de dispositivos](./iot-hub-bulk-identity-mgmt.md) 
