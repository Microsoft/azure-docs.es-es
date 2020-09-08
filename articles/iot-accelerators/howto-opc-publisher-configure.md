---
title: 'Configuración de OPC Publisher: Azure | Microsoft Docs'
description: En este artículo se describe cómo configurar OPC Publisher para especificar cambios en los datos de nodo de OPC UA, eventos de OPC UA para publicar y también el formato de telemetría.
author: dominicbetts
ms.author: dobett
ms.date: 06/10/2019
ms.topic: conceptual
ms.service: industrial-iot
services: iot-industrialiot
manager: philmea
ms.custom: devx-track-csharp
ms.openlocfilehash: cb27b4fec0d266bb9931b3942746b034ee3a645f
ms.sourcegitcommit: 419cf179f9597936378ed5098ef77437dbf16295
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/27/2020
ms.locfileid: "89009957"
---
# <a name="configure-opc-publisher"></a>Configuración de OPC Publisher

Puede configurar OPC Publisher para especificar:

- Los cambios de datos del nodo OPC UA para publicar.
- Los eventos de OPC UA para publicar.
- El formato de los datos de telemetría.

Puede configurar OPC Publisher mediante archivos de configuración o llamadas a métodos.

## <a name="use-configuration-files"></a>Uso de archivos de configuración

En esta sección se describen en las opciones de configuración del nodo OPC UA con archivos de configuración.

### <a name="use-a-configuration-file-to-configure-publishing-data-changes"></a>Uso de un archivo de configuración para configurar los cambios en los datos de publicación

La manera más sencilla de configurar los nodos OPC UA para publicar es con un archivo de configuración. El formato del archivo de configuración se documenta en [publishednodes.json](https://github.com/Azure/iot-edge-opc-publisher/blob/master/opcpublisher/publishednodes.json) en el repositorio.

La sintaxis del archivo de configuración ha cambiado con el tiempo. OPC Publisher todavía lee el formato anterior, pero lo convierte al formato más reciente cuando conserva la configuración.

En el ejemplo siguiente se muestra el formato del archivo de configuración:

```json
[
  {
    "EndpointUrl": "opc.tcp://testserver:62541/Quickstarts/ReferenceServer",
    "UseSecurity": true,
    "OpcNodes": [
      {
        "Id": "i=2258",
        "OpcSamplingInterval": 2000,
        "OpcPublishingInterval": 5000,
        "DisplayName": "Current time"
      }
    ]
  }
]
```

### <a name="use-a-configuration-file-to-configure-publishing-events"></a>Uso de un archivo de configuración para configurar los eventos para publicación

Para publicar eventos de OPC UA, use el mismo archivo de configuración que para los cambios de datos.

El ejemplo siguiente muestra cómo configurar la publicación para los eventos generados por el [servidor SimpleEvents](https://github.com/OPCFoundation/UA-.NETStandard-Samples/tree/master/Workshop/SimpleEvents/Server). El servidor SimpleEvents que puede encontrarse en el [repositorio de OPC Foundation](https://github.com/OPCFoundation/UA-.NETStandard-Samples) es:

```json
[
  {
    "EndpointUrl": "opc.tcp://testserver:62563/Quickstarts/SimpleEventsServer",
    "OpcEvents": [
      {
        "Id": "i=2253",
        "DisplayName": "SimpleEventServerEvents",
        "SelectClauses": [
          {
            "TypeId": "i=2041",
            "BrowsePaths": [
              "EventId"
            ]
          },
          {
            "TypeId": "i=2041",
            "BrowsePaths": [
              "Message"
            ]
          },
          {
            "TypeId": "nsu=http://opcfoundation.org/Quickstarts/SimpleEvents;i=235",
            "BrowsePaths": [
              "/2:CycleId"
            ]
          },
          {
            "TypeId": "nsu=http://opcfoundation.org/Quickstarts/SimpleEvents;i=235",
            "BrowsePaths": [
              "/2:CurrentStep"
            ]
          }
        ],
        "WhereClause": [
          {
            "Operator": "OfType",
            "Operands": [
              {
                "Literal": "ns=2;i=235"
              }
            ]
          }
        ]
      }
    ]
  }
]
```

## <a name="use-method-calls"></a>Uso de llamadas a métodos

En esta sección se describen las llamadas a métodos que se pueden usar para configurar OPC Publisher.

### <a name="configure-using-opc-ua-method-calls"></a>Configuración mediante llamadas a métodos OPC UA

OPC Publisher incluye un servidor OPC UA al que se puede acceder en el puerto 62222. Si el nombre de host es **publisher**, el punto de conexión URI es `opc.tcp://publisher:62222/UA/Publisher`.

Este punto de conexión expone los cuatro métodos siguientes:

- PublishNode
- UnpublishNode
- GetPublishedNodes
- IoT HubDirectMethod

### <a name="configure-using-iot-hub-direct-method-calls"></a>Configuración mediante llamadas a métodos directos de IoT Hub

OPC Publisher implementa las siguientes llamadas a métodos directos de IoT Hub:

- PublishNodes
- UnpublishNodes
- UnpublishAllNodes
- GetConfiguredEndpoints
- GetConfiguredNodesOnEndpoint
- GetDiagnosticInfo
- GetDiagnosticLog
- GetDiagnosticStartupLog
- ExitApplication
- GetInfo

El formato de la carga de JSON de la solicitud del método y las respuestas se definen en [opcpublisher/HubMethodModel.cs](https://github.com/Azure/iot-edge-opc-publisher/blob/master/opcpublisher/HubMethodModel.cs).

Si llama a un método desconocido en el módulo, responde con una cadena que indica que el método no se ha implementado. Puede llamar a un método desconocido para hacer ping en el módulo.

### <a name="configure-username-and-password-for-authentication"></a>Configuración de la autenticación con nombre de usuario y contraseña

Se puede establecer el modo de autenticación mediante llamadas a métodos directos de IoT Hub. La carga debe contener la propiedad **OpcAuthenticationMode**, y el nombre de usuario y la contraseña:

```csharp
{
    "EndpointUrl": "<Url of the endpoint to set authentication settings>",
    "OpcAuthenticationMode": "UsernamePassword",
    "Username": "<Username>",
    "Password": "<Password>"
    ...
}
```

El cliente de carga de IoT Hub cifra la contraseña, que se guarda en la configuración del publicador. Para volver a cambiar la autenticación a anónimo, use el método con la siguiente carga:

```csharp
{
    "EndpointUrl": "<Url of the endpoint to set authentication settings>",
    "OpcAuthenticationMode": "Anonymous"
    ...
}
```

Si la propiedad **OpcAuthenticationMode** no está establecida en la carga, la configuración de la autenticación no cambia.

## <a name="configure-telemetry-publishing"></a>Configuración de la publicación de telemetría

Cuando OPC Publisher recibe una notificación de cambio de valores en un nodo publicado, genera un mensaje con formato JSON que se envía a IoT Hub.

Puede configurar el contenido de este mensaje con formato JSON con un archivo de configuración. Si no se especifica ningún archivo de configuración con la opción `--tc`, se usa una configuración predeterminada compatible con el [acelerador de soluciones de la factoría conectada](https://github.com/Azure/azure-iot-connected-factory).

Si OPC Publisher está configurado para agrupar mensajes en lotes, se envían como matriz JSON válida.

Los datos de telemetría proceden de los siguientes orígenes:

- La configuración de OPC Publisher para el nodo
- El objeto **MonitoredItem** de la pila OPC UA sobre la que OPC Publisher recibió una notificación.
- El argumento pasado a esta notificación, que proporciona información detallada sobre el cambio de valor de los datos.

Los datos de telemetría del mensaje con formato JSON son una selección de propiedades importantes de estos objetos. Si necesita más propiedades, deberá cambiar la base del código de OPC Publisher.

La sintaxis del archivo de configuración es la que se indica a continuación:

```json
// The configuration settings file consists of two objects:
// 1) The 'Defaults' object, which defines defaults for the telemetry configuration
// 2) An array 'EndpointSpecific' of endpoint specific configuration
// Both objects are optional and if they are not specified, then publisher uses
// its internal default configuration, which generates telemetry messages compatible
// with the Microsoft Connected factory Preconfigured Solution (https://github.com/Azure/azure-iot-connected-factory).

// A JSON telemetry message for Connected factory looks like:
//  {
//      "NodeId": "i=2058",
//      "ApplicationUri": "urn:myopcserver",
//      "DisplayName": "CurrentTime",
//      "Value": {
//          "Value": "10.11.2017 14:03:17",
//          "SourceTimestamp": "2017-11-10T14:03:17Z"
//      }
//  }

// The 'Defaults' object in the sample below, are similar to what publisher is
// using as its internal default telemetry configuration.
{
    "Defaults": {
        // The first two properties ('EndpointUrl' and 'NodeId' are configuring data
        // taken from the OpcPublisher node configuration.
        "EndpointUrl": {

            // The following three properties can be used to configure the 'EndpointUrl'
            // property in the JSON message send by publisher to IoT Hub.

            // Publish controls if the property should be part of the JSON message at all.
            "Publish": false,

            // Pattern is a regular expression, which is applied to the actual value of the
            // property (here 'EndpointUrl').
            // If this key is omitted (which is the default), then no regex matching is done
            // at all, which improves performance.
            // If the key is used you need to define groups in the regular expression.
            // Publisher applies the regular expression and then concatenates all groups
            // found and use the resulting string as the value in the JSON message to
            //sent to IoT Hub.
            // This example mimics the default behaviour and defines a group,
            // which matches the conplete value:
            "Pattern": "(.*)",
            // Here some more exaples for 'Pattern' values and the generated result:
            // "Pattern": "i=(.*)"
            // defined for Defaults.NodeId.Pattern, will generate for the above sample
            // a 'NodeId' value of '2058'to be sent by publisher
            // "Pattern": "(i)=(.*)"
            // defined for Defaults.NodeId.Pattern, will generate for the above sample
            // a 'NodeId' value of 'i2058' to be sent by publisher

            // Name allows you to use a shorter string as property name in the JSON message
            // sent by publisher. By default the property name is unchanged and will be
            // here 'EndpointUrl'.
            // The 'Name' property can only be set in the 'Defaults' object to ensure
            // all messages from publisher sent to IoT Hub have a similar layout.
            "Name": "EndpointUrl"

        },
        "NodeId": {
            "Publish": true,

            // If you set Defaults.NodeId.Name to "ni", then the "NodeId" key/value pair
            // (from the above example) will change to:
            //      "ni": "i=2058",
            "Name": "NodeId"
        },

        // The MonitoredItem object is configuring the data taken from the MonitoredItem
        // OPC UA object for published nodes.
        "MonitoredItem": {

            // If you set the Defaults.MonitoredItem.Flat to 'false', then a
            // 'MonitoredItem' object will appear, which contains 'ApplicationUri'
            // and 'DisplayNode' proerties:
            //      "NodeId": "i=2058",
            //      "MonitoredItem": {
            //          "ApplicationUri": "urn:myopcserver",
            //          "DisplayName": "CurrentTime",
            //      }
            // The 'Flat' property can only be used in the 'MonitoredItem' and
            // 'Value' objects of the 'Defaults' object and will be used
            // for all JSON messages sent by publisher.
            "Flat": true,

            "ApplicationUri": {
                "Publish": true,
                "Name": "ApplicationUri"
            },
            "DisplayName": {
                "Publish": true,
                "Name": "DisplayName"
            }
        },
        // The Value object is configuring the properties taken from the event object
        // the OPC UA stack provided in the value change notification event.
        "Value": {
            // If you set the Defaults.Value.Flat to 'true', then the 'Value'
            // object will disappear completely and the 'Value' and 'SourceTimestamp'
            // members won't be nested:
            //      "DisplayName": "CurrentTime",
            //      "Value": "10.11.2017 14:03:17",
            //      "SourceTimestamp": "2017-11-10T14:03:17Z"
            // The 'Flat' property can only be used for the 'MonitoredItem' and 'Value'
            // objects of the 'Defaults' object and will be used for all
            // messages sent by publisher.
            "Flat": false,

            "Value": {
                "Publish": true,
                "Name": "Value"
            },
            "SourceTimestamp": {
                "Publish": true,
                "Name": "SourceTimestamp"
            },
            // 'StatusCode' is the 32 bit OPC UA status code
            "StatusCode": {
                "Publish": false,
                "Name": "StatusCode"
                // 'Pattern' is ignored for the 'StatusCode' value
            },
            // 'Status' is the symbolic name of 'StatusCode'
            "Status": {
                "Publish": false,
                "Name": "Status"
            }
        }
    },

    // The next object allows to configure 'Publish' and 'Pattern' for specific
    // endpoint URLs. Those will overwrite the ones specified in the 'Defaults' object
    // or the defaults used by publisher.
    // It is not allowed to specify 'Name' and 'Flat' properties in this object.
    "EndpointSpecific": [
        // The following shows how a endpoint specific configuration can look like:
        {
            // 'ForEndpointUrl' allows to configure for which OPC UA server this
            // object applies and is a required property for all objects in the
            // 'EndpointSpecific' array.
            // The value of 'ForEndpointUrl' must be an 'EndpointUrl' configured in
            // the publishednodes.json confguration file.
            "ForEndpointUrl": "opc.tcp://<your_opcua_server>:<your_opcua_server_port>/<your_opcua_server_path>",
            "EndpointUrl": {
                // We overwrite the default behaviour and publish the
                // endpoint URL in this case.
                "Publish": true,
                // We are only interested in the URL part following the 'opc.tcp://' prefix
                // and define a group matching this.
                "Pattern": "opc.tcp://(.*)"
            },
            "NodeId": {
                // We are not interested in the configured 'NodeId' value, 
                // so we do not publish it.
                "Publish": false
                // No 'Pattern' key is specified here, so the 'NodeId' value will be
                // taken as specified in the publishednodes configuration file.
            },
            "MonitoredItem": {
                "ApplicationUri": {
                    // We already publish the endpoint URL, so we do not want
                    // the ApplicationUri of the MonitoredItem to be published.
                    "Publish": false
                },
                "DisplayName": {
                    "Publish": true
                }
            },
            "Value": {
                "Value": {
                    // The value of the node is important for us, everything else we
                    // are not interested in to keep the data ingest as small as possible.
                    "Publish": true
                },
                "SourceTimestamp": {
                    "Publish": false
                },
                "StatusCode": {
                    "Publish": false
                },
                "Status": {
                    "Publish": false
                }
            }
        }
    ]
}
```

## <a name="next-steps"></a>Pasos siguientes

Ahora que ha aprendido a configurar OPC Publisher, el siguiente paso recomendado es aprender a [ejecutar OPC Publisher](howto-opc-publisher-run.md).
