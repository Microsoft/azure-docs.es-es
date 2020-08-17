---
title: Filtros de conexión IP de Azure IoT Hub | Microsoft Docs
description: Describe cómo usar el filtrado de IP para bloquear las conexiones de direcciones IP específicas de su instancia de Azure IoT Hub. Puede bloquear conexiones de direcciones IP concretas o de intervalos.
author: robinsh
ms.service: iot-hub
services: iot-hub
ms.topic: conceptual
ms.date: 05/25/2020
ms.author: robinsh
ms.openlocfilehash: 1ba3c89ea4f964f9e6fd5f902aab29a83a058f25
ms.sourcegitcommit: 3d79f737ff34708b48dd2ae45100e2516af9ed78
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/23/2020
ms.locfileid: "87074723"
---
# <a name="use-ip-filters"></a>Uso de filtros IP

La seguridad es un aspecto importante de cualquier solución de IoT basada en Azure IoT Hub. En ciertas ocasiones necesitará especificar explícitamente las direcciones IP desde las que se pueden conectar los dispositivos como parte de la configuración de seguridad. La característica de *filtro IP* le permite configurar reglas para rechazar o aceptar tráfico de direcciones IPv4 específicas.

## <a name="when-to-use"></a>Cuándo se usa

Hay dos casos específicos cuando resulta útil bloquear los puntos de conexión de IoT Hub para determinadas direcciones IP:

* IoT Hub debe recibir tráfico solo de un intervalo concreto de direcciones IP y rechazar todo lo demás. Por ejemplo, cuando se usa IoT Hub con [Azure ExpressRoute](https://azure.microsoft.com/documentation/articles/expressroute-faqs/#supported-services) para crear conexiones privadas entre una instancia de IoT Hub y la infraestructura local.

* Cuando necesite rechazar el tráfico de direcciones IP que el administrador de IoT Hub haya identificado como sospechosas.

## <a name="how-filter-rules-are-applied"></a>Cómo se aplican las reglas de filtro

Las reglas de filtro IP se aplican en el nivel de servicio de IoT Hub. Por lo tanto, las reglas de filtro IP se aplican a todas las conexiones de los dispositivos y aplicaciones de back-end mediante un protocolo admitido. Sin embargo, los clientes que leen directamente desde el [punto de conexión compatible con el centro de eventos integrado](iot-hub-devguide-messages-read-builtin.md) (no a través de la cadena de conexión IoT Hub) no están enlazados a las reglas de filtro IP. 

Cualquier intento de conexión desde una dirección IP que coincida con una regla IP de rechazo en su centro de IoT recibe un código de estado 401 no autorizado y la descripción. El mensaje de respuesta no menciona la regla IP. Rechazar direcciones de IP puede evitar que otros servicios de Azure (por ejemplo, Azure Stream Analytics, Azure Virtual Machines o el Explorador de dispositivos de Azure Portal) interactúen con el centro de IoT.

> [!NOTE]
> Si usa Azure Stream Analytics (ASA) para leer mensajes de una instancia de IoT Hub con el filtrado IP habilitado, use el punto de conexión y el nombre compatibles con el centro de eventos del centro de IoT para agregar manualmente una [entrada de flujo de datos de Event Hub](../stream-analytics/stream-analytics-define-inputs.md#stream-data-from-event-hubs) en ASA.

## <a name="default-setting"></a>Valor predeterminado

De forma predeterminada, la cuadrícula de **filtro IP** del portal para un centro de IoT está vacía. Este ajuste predeterminado indica que el centro acepta conexiones de cualquier dirección IP. Esta configuración predeterminada es equivalente a una regla que acepta el intervalo de direcciones IP 0.0.0.0/0.

Para ir a la página Configuración de filtro IP, seleccione **Redes**, **Acceso público** y, a continuación, **elija rangos IP seleccionados**:

:::image type="content" source="media/iot-hub-ip-filtering/ip-filter-default.png" alt-text="Configuración predeterminada de filtro de direcciones IP de IoT Hub":::

## <a name="add-or-edit-an-ip-filter-rule"></a>Incorporación o edición de una regla de filtro IP

Para agregar una regla de filtro IP, seleccione **+ Agregar regla de filtro IP**.

:::image type="content" source="./media/iot-hub-ip-filtering/ip-filter-add-rule.png" alt-text="Incorporación de una regla de filtro IP a una instancia de IoT Hub":::

Después de seleccionar **Agregar regla de filtro IP**, rellene los campos.

:::image type="content" source="./media/iot-hub-ip-filtering/ip-filter-after-selecting-add.png" alt-text="Después de seleccionar Agregar regla de filtro IP":::

* Escriba un **nombre** para la regla de filtro IP. Debe ser una cadena única de hasta 128 caracteres alfanuméricos que no distinga mayúsculas de minúsculas. Solo se aceptan los caracteres alfanuméricos de 7 bits ASCII más `{'-', ':', '/', '\', '.', '+', '%', '_', '#', '*', '?', '!', '(', ')', ',', '=', '@', ';', '''}`.

* Proporcione una única dirección IPv4 o un bloque de direcciones IP en la notación CIDR. Por ejemplo, en notación CIDR, 192.168.100.0/22 representa las direcciones IPv4 de 1024 de 192.168.100.0 a 192.168.103.255.

* Seleccione un **Permitir** o **Bloquear** como **acción** para la regla de filtro IP.

Después de rellenar los campos, seleccione **Guardar** para guardar la regla. Se mostrará una alerta que le informará de que la actualización está en curso.

:::image type="content" source="./media/iot-hub-ip-filtering/ip-filter-save-new-rule.png" alt-text="Notificación acerca de cómo guardar una regla de filtro IP":::

La opción **Agregar** está deshabilitada cuando se alcanza el máximo de 10 reglas de filtro IP.

Para editar una regla existente, seleccione los datos que quiere cambiar, realice el cambio y, a continuación, seleccione **Guardar** para guardar la edición.

## <a name="delete-an-ip-filter-rule"></a>Eliminación de una regla de filtro IP

Para eliminar una regla de filtro IP, seleccione el icono de la papelera de esa fila y, a continuación, seleccione **Guardar**. Se quita la regla y se guarda el cambio.

:::image type="content" source="./media/iot-hub-ip-filtering/ip-filter-delete-rule.png" alt-text="Eliminación de una regla de filtro IP de IoT Hub":::

## <a name="retrieve-and-update-ip-filters-using-azure-cli"></a>Recuperación y actualización de los filtros IP mediante la CLI de Azure

Los filtros IP de IoT Hub se pueden recuperar y actualizar con la [CLI de Azure](https://docs.microsoft.com/cli/azure/?view=azure-cli-latest).

Para recuperar los filtros IP actuales de IoT Hub, ejecute:

```azurecli-interactive
az resource show -n <iothubName> -g <resourceGroupName> --resource-type Microsoft.Devices/IotHubs
```

Esto devolverá un objeto JSON, donde se enumeran los filtros IP existentes bajo la clave `properties.ipFilterRules`:

```json
{
...
    "properties": {
        "ipFilterRules": [
        {
            "action": "Reject",
            "filterName": "MaliciousIP",
            "ipMask": "6.6.6.6/6"
        },
        {
            "action": "Allow",
            "filterName": "GoodIP",
            "ipMask": "131.107.160.200"
        },
        ...
        ],
    },
...
}
```

Para agregar un nuevo filtro IP de IoT Hub, ejecute:

```azurecli-interactive
az resource update -n <iothubName> -g <resourceGroupName> --resource-type Microsoft.Devices/IotHubs --add properties.ipFilterRules "{\"action\":\"Reject\",\"filterName\":\"MaliciousIP\",\"ipMask\":\"6.6.6.6/6\"}"
```

Para quitar un filtro IP existente en IoT Hub, ejecute:

```azurecli-interactive
az resource update -n <iothubName> -g <resourceGroupName> --resource-type Microsoft.Devices/IotHubs --add properties.ipFilterRules <ipFilterIndexToRemove>
```

Tenga en cuenta que `<ipFilterIndexToRemove>` tiene que corresponder al orden de filtros IP de `properties.ipFilterRules` de su instancia de IoT Hub.

## <a name="retrieve-and-update-ip-filters-using-azure-powershell"></a>Recuperación y actualización de los filtros IP mediante Azure PowerShell

[!INCLUDE [updated-for-az](../../includes/updated-for-az.md)]

Los filtros IP de IoT Hub se pueden recuperar y establecer con [Azure PowerShell](/powershell/azure/).

```powershell
# Get your IoT Hub resource using its name and its resource group name
$iothubResource = Get-AzResource -ResourceGroupName <resourceGroupNmae> -ResourceName <iotHubName> -ExpandProperties

# Access existing IP filter rules
$iothubResource.Properties.ipFilterRules |% { Write-host $_ }

# Construct a new IP filter
$filter = @{'filterName'='MaliciousIP'; 'action'='Reject'; 'ipMask'='6.6.6.6/6'}

# Add your new IP filter rule
$iothubResource.Properties.ipFilterRules += $filter

# Remove an existing IP filter rule using its name, e.g., 'GoodIP'
$iothubResource.Properties.ipFilterRules = @($iothubResource.Properties.ipFilterRules | Where 'filterName' -ne 'GoodIP')

# Update your IoT Hub resource with your updated IP filters
$iothubResource | Set-AzResource -Force
```

## <a name="update-ip-filter-rules-using-rest"></a>Actualización de las reglas de filtro IP con REST

También puede recuperar y modificar el filtro IP de IoT Hub mediante el punto de conexión REST del proveedor de recursos de Azure. Consulte `properties.ipFilterRules` en [método createorupdate](https://docs.microsoft.com/rest/api/iothub/iothubresource/createorupdate).

## <a name="ip-filter-rule-evaluation"></a>Evaluación de las reglas de filtro IP

Las reglas de filtro IP se aplican en orden y la primera regla que coincida con la dirección IP determina la acción de aceptar o rechazar.

Por ejemplo, si desea aceptar las direcciones del intervalo 192.168.100.0/22 y rechazar todas las demás, la primera regla de la cuadrícula debe aceptar el intervalo de direcciones 192.168.100.0/22. La siguiente regla debe rechazar todas las direcciones mediante el intervalo 0.0.0.0/0.

Puede cambiar el orden de las reglas de filtro IP en la cuadrícula al hacer clic en los tres puntos verticales situados al principio de las filas y mediante el método arrastrar y colocar.

Para guardar el nuevo orden de reglas de filtro IP, haga clic en **Guardar**.

:::image type="content" source="media/iot-hub-ip-filtering/ip-filter-rule-order.png" alt-text="Cambiar el orden de las reglas de filtro IP de IoT Hub":::

## <a name="next-steps"></a>Pasos siguientes

Para explorar aún más las funcionalidades de IoT Hub, consulte:

* [Métricas de IoT Hub](iot-hub-metrics.md)
