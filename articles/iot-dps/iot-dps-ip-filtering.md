---
title: Filtros de conexión IP de Azure IoT DPS | Microsoft Docs
description: Describe cómo usar el filtrado de IP para bloquear las conexiones de direcciones IP específicas de su instancia de Azure IoT DPS. Puede bloquear conexiones de direcciones IP concretas o de intervalos.
author: wesmc7777
ms.author: wesmc
ms.service: iot-dps
services: iot-dps
ms.topic: conceptual
ms.date: 03/12/2020
ms.openlocfilehash: 580c378df5fc3912aa540b5d85adf99bc42605e0
ms.sourcegitcommit: 3543d3b4f6c6f496d22ea5f97d8cd2700ac9a481
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/20/2020
ms.locfileid: "86511949"
---
# <a name="use-azure-iot-dps-ip-connection-filters"></a>Uso de filtros de conexión IP de Azure IoT DPS

La seguridad es un aspecto importante de cualquier solución de IoT. En ciertas ocasiones necesitará especificar explícitamente las direcciones IP desde las que se pueden conectar los dispositivos como parte de la configuración de seguridad. La característica de *filtro IP* para una instancia de Azure IoT Hub Device Provisioning Service (DPS) le permite configurar reglas para rechazar o aceptar tráfico de direcciones IPv4 específicas.

## <a name="when-to-use"></a>Cuándo se usa

Hay dos casos específicos en los que resulta útil bloquear las conexiones a un punto de conexión de DPS para determinadas direcciones IP:

* La instancia de DPS debe recibir tráfico solo de un intervalo concreto de direcciones IP y rechazar todas las demás. Por ejemplo, cuando se usa DPS con [Azure ExpressRoute](https://azure.microsoft.com/documentation/articles/expressroute-faqs/#supported-services) para crear conexiones privadas entre una instancia de DPS y los dispositivos.

* Cuando necesite rechazar el tráfico de direcciones IP que el administrador de DPS haya identificado como sospechosas.

## <a name="how-filter-rules-are-applied"></a>Cómo se aplican las reglas de filtro

Las reglas de filtro IP se aplican en el nivel de instancia de DPS. Por lo tanto, las reglas de filtro IP se aplican a todas las conexiones de los dispositivos y aplicaciones de back-end mediante un protocolo admitido.

Cualquier intento de conexión desde una dirección IP que coincida con una regla IP de rechazo en su instancia de DSP recibe un código de estado 401 no autorizado y la descripción. El mensaje de respuesta no menciona la regla IP.

## <a name="default-setting"></a>Valor predeterminado

De forma predeterminada, la cuadrícula de **filtro IP** del portal para DPS está vacía. Este ajuste predeterminado indica que la instancia de DPS acepta conexiones de cualquier dirección IP. Esta configuración predeterminada es equivalente a una regla que acepta el intervalo de direcciones IP 0.0.0.0/0.

![Configuración predeterminada de filtro IP de IoT DPS](./media/iot-dps-ip-filtering/ip-filter-default.png)

## <a name="add-or-edit-an-ip-filter-rule"></a>Incorporación o edición de una regla de filtro IP

Para agregar una regla de filtro IP, seleccione **+ Agregar regla de filtro IP**.

![Adición de una regla de filtro IP a una instancia de IoT DPS](./media/iot-dps-ip-filtering/ip-filter-add-rule.png)

Después de seleccionar **Agregar regla de filtro IP**, rellene los campos.

![Después de seleccionar Agregar regla de filtro IP](./media/iot-dps-ip-filtering/ip-filter-after-selecting-add.png)

* Escriba un **nombre** para la regla de filtro IP. Debe ser una cadena única de hasta 128 caracteres alfanuméricos que no distinga mayúsculas de minúsculas. Solo se aceptan los caracteres alfanuméricos de 7 bits ASCII más `{'-', ':', '/', '\', '.', '+', '%', '_', '#', '*', '?', '!', '(', ')', ',', '=', '@', ';', '''}`.

* Proporcione una única dirección IPv4 o un bloque de direcciones IP en la notación CIDR. Por ejemplo, en notación CIDR, 192.168.100.0/22 representa las direcciones IPv4 de 1024 de 192.168.100.0 a 192.168.103.255.

* Seleccione un **Permitir** o **Bloquear** como **acción** para la regla de filtro IP.

Después de rellenar los campos, seleccione **Guardar** para guardar la regla. Se mostrará una alerta que le informará de que la actualización está en curso.

![Notificación acerca de cómo guardar una regla de filtro IP](./media/iot-dps-ip-filtering/ip-filter-save-new-rule.png)

La opción **Agregar** está deshabilitada cuando se alcanza el máximo de 10 reglas de filtro IP.

Para editar una regla existente, seleccione los datos que quiere cambiar, realice el cambio y, a continuación, seleccione **Guardar** para guardar la edición.

> [!NOTE]
> El rechazo de las direcciones IP puede evitar que otros servicios de Azure interactúen con la instancia de DPS.

## <a name="delete-an-ip-filter-rule"></a>Eliminación de una regla de filtro IP

Para eliminar una regla de filtro IP, seleccione el icono de la papelera de esa fila y, a continuación, seleccione **Guardar**. Se quita la regla y se guarda el cambio.

![Eliminación de una regla de filtro IP de IoT DPS](./media/iot-dps-ip-filtering/ip-filter-delete-rule.png)


## <a name="update-ip-filter-rules-in-code"></a>Actualización de reglas de filtro IP en el código

También puede recuperar y modificar el filtro IP de DPS mediante el punto de conexión REST del proveedor de recursos de Azure. Consulte `properties.ipFilterRules` en [método createorupdate](https://docs.microsoft.com/rest/api/iot-dps/iotdpsresource/createorupdate).

Actualmente no se admite la actualización de las reglas de filtro IP de DPS con la CLI de Azure o Azure PowerShell, pero se puede realizar con plantillas de Azure Resource Manager. Consulte [Información general sobre plantillas de Azure Resource Manager](../azure-resource-manager/templates/overview.md) para obtener instrucciones sobre el uso de las plantillas de Resource Manager. Los ejemplos de plantillas siguientes muestran cómo crear, editar y eliminar reglas de filtro de IP de DPS.

### <a name="add-an-ip-filter-rule"></a>Adición de una regla de filtro IP

En el ejemplo de plantilla siguiente se crea una nueva regla de filtro IP denominada "AllowAll" que acepta todo el tráfico.

```json
{
    "$schema": "https://schema.management.azure.com/schemas/2015-01-01/deploymentTemplate.json#", 
    "contentVersion": "1.0.0.0", 
    "parameters": {
        "iotDpsName": {
            "type": "string",
            "defaultValue": "[resourceGroup().name]",
            "minLength": 3,
            "metadata": {
                "description": "Specifies the name of the IoT DPS service."
            }
        }, 
        "location": {
            "type": "string",
            "defaultValue": "[resourceGroup().location]",
            "metadata": {
                "description": "Location for Iot DPS resource."
            }
        }        
    }, 
    "variables": {
        "iotDpsApiVersion": "2020-01-01"
    }, 
    "resources": [
        {
            "type": "Microsoft.Devices/provisioningServices",
            "apiVersion": "[variables('iotDpsApiVersion')]",
            "name": "[parameters('iotDpsName')]",
            "location": "[parameters('location')]",
            "sku": {
                "name": "S1",
                "tier": "Standard",
                "capacity": 1
            },
            "properties": {
                "IpFilterRules": [
                    {
                        "FilterName": "AllowAll",
                        "Action": "Accept",
                        "ipMask": "0.0.0.0/0"
                    }
                ]
            }            
        }
    ]
}
```

Actualice los atributos de la regla de filtro IP de la plantilla en función de sus requisitos.

| Atributo                | Descripción |
| ------------------------ | ----------- |
| **FilterName**           | Escriba un nombre para la regla de filtro IP. Debe ser una cadena única de hasta 128 caracteres alfanuméricos que no distinga mayúsculas de minúsculas. Solo se aceptan los caracteres alfanuméricos ASCII de 7 bits más {'-', ':', '/', '\', '.', '+', '%', '_', '#', '*', '?', '!', '(', ')', ',', '=', '@', ';', '''}. |
| **Acción**               | Los valores aceptados son **Aceptar** o **Rechazar** como la acción de la regla de filtro IP. |
| **ipMask**               | Proporcione una única dirección IPv4 o un bloque de direcciones IP en la notación CIDR. Por ejemplo, en notación CIDR, 192.168.100.0/22 representa las direcciones IPv4 de 1024 de 192.168.100.0 a 192.168.103.255. |


### <a name="update-an-ip-filter-rule"></a>Actualización de una regla de filtro IP

En el ejemplo de plantilla siguiente se actualiza la regla de filtro IP denominada "AllowAll", que se mostró anteriormente, para rechazar todo el tráfico.

```json
{ 
    "$schema": "https://schema.management.azure.com/schemas/2015-01-01/deploymentTemplate.json#",  
    "contentVersion": "1.0.0.0",  
    "parameters": { 
        "iotDpsName": { 
            "type": "string", 
            "defaultValue": "[resourceGroup().name]", 
            "minLength": 3, 
            "metadata": { 
                "description": "Specifies the name of the IoT DPS service." 
            } 
        },  
        "location": { 
            "type": "string", 
            "defaultValue": "[resourceGroup().location]", 
            "metadata": { 
                "description": "Location for Iot DPS resource." 
            } 
        }        
    },  
    "variables": { 
        "iotDpsApiVersion": "2020-01-01" 
    },  
    "resources": [ 
        { 
            "type": "Microsoft.Devices/provisioningServices", 
            "apiVersion": "[variables('iotDpsApiVersion')]", 
            "name": "[parameters('iotDpsName')]", 
            "location": "[parameters('location')]", 
            "sku": { 
                "name": "S1", 
                "tier": "Standard", 
                "capacity": 1 
            }, 
            "properties": { 
                "IpFilterRules": [ 
                    { 
                        "FilterName": "AllowAll", 
                        "Action": "Reject", 
                        "ipMask": "0.0.0.0/0" 
                    } 
                ] 
            }             
        } 
    ] 
}
```

### <a name="delete-an-ip-filter-rule"></a>Eliminación de una regla de filtro IP

En el ejemplo de plantilla siguiente se eliminan todas las reglas de filtro IP para la instancia de DPS.

```json
{ 
    "$schema": "https://schema.management.azure.com/schemas/2015-01-01/deploymentTemplate.json#",  
    "contentVersion": "1.0.0.0",  
    "parameters": { 
        "iotDpsName": { 
            "type": "string", 
            "defaultValue": "[resourceGroup().name]", 
            "minLength": 3, 
            "metadata": { 
                "description": "Specifies the name of the IoT DPS service." 
            } 
        },  
        "location": { 
            "type": "string", 
            "defaultValue": "[resourceGroup().location]", 
            "metadata": { 
                "description": "Location for Iot DPS resource." 
            } 
        }        
    },  
    "variables": { 
        "iotDpsApiVersion": "2020-01-01" 
    },  
    "resources": [ 
        { 
            "type": "Microsoft.Devices/provisioningServices", 
            "apiVersion": "[variables('iotDpsApiVersion')]", 
            "name": "[parameters('iotDpsName')]", 
            "location": "[parameters('location')]", 
            "sku": { 
                "name": "S1", 
                "tier": "Standard", 
                "capacity": 1 
            }, 
            "properties": { 
            }             
        } 
    ] 
}
```



## <a name="ip-filter-rule-evaluation"></a>Evaluación de las reglas de filtro IP

Las reglas de filtro IP se aplican en orden y la primera regla que coincida con la dirección IP determina la acción de aceptar o rechazar.

Por ejemplo, si desea aceptar las direcciones del intervalo 192.168.100.0/22 y rechazar todas las demás, la primera regla de la cuadrícula debe aceptar el intervalo de direcciones 192.168.100.0/22. La siguiente regla debe rechazar todas las direcciones mediante el intervalo 0.0.0.0/0.

Puede cambiar el orden de las reglas de filtro IP en la cuadrícula al hacer clic en los tres puntos verticales situados al principio de las filas y mediante el método arrastrar y colocar.

Para guardar el nuevo orden de reglas de filtro IP, haga clic en **Guardar**.

![Cambio del orden de las reglas de filtro IP de DPS](./media/iot-dps-ip-filtering/ip-filter-rule-order.png)

## <a name="next-steps"></a>Pasos siguientes

Para explorar aún más la administración de DPS, consulte:

* [Descripción de las direcciones IP de IoT DPS](iot-dps-understand-ip-address.md)
* [Uso de la CLI de Azure y de la extensión de IoT para administrar IoT Hub Device Provisioning Service](how-to-manage-dps-with-cli.md)
* [Control de acceso al servicio Azure IoT Hub Device Provisioning](how-to-control-access.md)
