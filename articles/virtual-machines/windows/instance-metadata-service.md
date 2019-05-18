---
title: Servicio de metadatos de instancia de Azure | Microsoft Docs
description: Interfaz rESTful para obtener información sobre el proceso, red y próximos eventos de mantenimiento de la máquina virtual de Windows.
services: virtual-machines-windows
documentationcenter: ''
author: KumariSupriya
manager: harijayms
editor: ''
tags: azure-resource-manager
ms.service: virtual-machines-windows
ms.devlang: na
ms.topic: article
ms.tgt_pltfrm: vm-windows
ms.workload: infrastructure-services
ms.date: 04/25/2019
ms.author: sukumari
ms.reviewer: azmetadata
ms.openlocfilehash: 160d494eea4bd597725a4e7c21ad9b763502bee6
ms.sourcegitcommit: 36c50860e75d86f0d0e2be9e3213ffa9a06f4150
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 05/16/2019
ms.locfileid: "65792091"
---
# <a name="azure-instance-metadata-service"></a>Servicio de metadatos de instancia de Azure

El servicio de metadatos de instancia de Azure proporciona información sobre instancias de máquina virtual en ejecución que pueden usarse para administrar y configurar las máquinas virtuales.
Esto incluye información como SKU, configuración de red y eventos de mantenimiento próximos. Para obtener más información sobre qué tipo de información está disponible, consulte [las API de metadatos](#metadata-apis).

El servicio de metadatos de instancia de Azure es un punto de conexión REST al que pueden tener acceso todas las máquinas virtuales IaaS creadas a través de [Azure Resource Manager](https://docs.microsoft.com/rest/api/resources/).
El punto de conexión está disponible en una dirección IP no enrutable conocida (`169.254.169.254`) a la que solo se puede tener acceso desde dentro de la máquina virtual.

> [!IMPORTANT]
> Este servicio está **disponible con carácter general** en todas las regiones de Azure.  Recibe actualizaciones periódicas para exponer información nueva sobre instancias de máquina virtual. Esta página refleja la actualizada [las API de metadatos](#metadata-apis) disponibles.

## <a name="service-availability"></a>Disponibilidad del servicio

El servicio está disponible con carácter general en las regiones de Azure. Puede que no todas las versiones de API estén disponibles en todas las regiones de Azure.

Regiones                                        | ¿Disponibilidad?                                 | Versiones compatibles
-----------------------------------------------|-----------------------------------------------|-----------------
[Todas las regiones globales de Azure disponibles con carácter general](https://azure.microsoft.com/regions/)     | Disponibilidad general | 2017-04-02, 2017-08-01, 2017-12-01, 2018-02-01, 2018-04-02, 2018-10-01
[Azure Government](https://azure.microsoft.com/overview/clouds/government/)              | Disponibilidad general | 2017-04-02, 2017-08-01, 2017-12-01, 2018-02-01, 2018-04-02, 2018-10-01
[Azure en China](https://www.azure.cn/)                                                     | Disponibilidad general | 2017-04-02, 2017-08-01, 2017-12-01, 2018-02-01, 2018-04-02, 2018-10-01
[Azure Alemania](https://azure.microsoft.com/overview/clouds/germany/)                    | Disponibilidad general | 2017-04-02, 2017-08-01, 2017-12-01, 2018-02-01, 2018-04-02, 2018-10-01
[Centro-oeste de EE. UU. público](https://azure.microsoft.com/regions/)                           | Disponibilidad general | 2017-04-02, 2017-08-01, 2017-12-01, 2018-02-01, 2018-04-02, 2018-10-01, 2019-02-01

Esta tabla cambia cuando hay actualizaciones del servicio o cuando hay nuevas versiones compatibles disponibles.

> [!NOTE]
> 2019-02-01 se implementa actualmente y estará disponible en otras regiones en breve.

Para probar el servicio de metadatos de instancia, cree una máquina virtual desde [Azure Resource Manager](https://docs.microsoft.com/rest/api/resources/) o [Azure Portal](https://portal.azure.com) en las regiones anteriores, y siga los ejemplos siguientes.

## <a name="usage"></a>Uso

### <a name="versioning"></a>Control de versiones

El servicio de metadatos de instancia tiene versiones. Las versiones son obligatorias y la versión actual de Global Azure es la `2018-10-01`. Las versiones compatibles actuales son (2017-04-02, 2017-08-01, 2017-12-01, 2018-02-01, 2018-04-02, 2018-10-01).

A medida que se agreguen versiones más recientes, todavía se podrá acceder a las versiones anteriores por motivos de compatibilidad si los scripts tienen dependencias en formatos de datos específicos.

Cuando se especifica ninguna versión, se devuelve un error con una lista de las versiones más recientes compatibles.

> [!NOTE]
> La respuesta es una cadena JSON. La respuesta de ejemplo siguiente se ha impreso correctamente para mejorar la legibilidad.

**Solicitud**

```bash
curl -H Metadata:true "http://169.254.169.254/metadata/instance"
```

**Respuesta**

```json
{
    "error": "Bad request. api-version was not specified in the request. For more information refer to aka.ms/azureimds",
    "newest-versions": [
        "2018-10-01",
        "2018-04-02",
        "2018-02-01"
    ]
}
```

### <a name="using-headers"></a>Uso de encabezados

Al consultar el servicio de metadatos de instancia, debe proporcionar el encabezado `Metadata: true` para asegurarse de que la solicitud no se ha redirigido de manera involuntaria.

### <a name="retrieving-metadata"></a>Recuperar metadatos

Los metadatos de instancia están disponibles para ejecutar máquinas virtuales creadas o administradas mediante [Azure Resource Manager](https://docs.microsoft.com/rest/api/resources/). Para tener acceso a todas las categorías de datos de una instancia de máquina virtual, use la solicitud siguiente:

```bash
curl -H Metadata:true "http://169.254.169.254/metadata/instance?api-version=2017-08-01"
```

> [!NOTE]
> Todas las consultas de metadatos de instancia distinguen mayúsculas de minúsculas.

### <a name="data-output"></a>Salida de datos

De forma predeterminada, el servicio de metadatos de instancia devuelve datos en formato JSON (`Content-Type: application/json`). Pero las distintas API devuelven los datos en formatos diferentes si se solicita.
En la tabla siguiente se muestra una referencia de otros formatos de datos que las API pueden admitir.

API | Formato predeterminado de datos | Otros formatos
--------|---------------------|--------------
/instance | json | text
/scheduledevents | json | ninguno
/attested | JSON | ninguno

Para acceder a un formato de respuesta que no sea el predeterminado, especifique el formato solicitado como un parámetro de cadena de consulta en la solicitud. Por ejemplo:

```bash
curl -H Metadata:true "http://169.254.169.254/metadata/instance?api-version=2017-08-01&format=text"
```

> [!NOTE]
> Para los nodos hoja el `format=json` no funciona. Para estas consultas `format=text` debe especificarse explícitamente si el formato predeterminado es json.

### <a name="security"></a>Seguridad

El punto de conexión del servicio de metadatos de instancia solo es accesible desde la instancia de máquina virtual en ejecución en una dirección IP no enrutable. Además, el servicio rechaza cualquier solicitud que tenga un encabezado `X-Forwarded-For`.
Las solicitudes tienen que incluir también un encabezado `Metadata: true` para garantizar que la solicitud actual esté dirigida directamente y no como parte de un redireccionamiento accidental.

### <a name="error"></a>Error

Si no se encuentra un elemento de datos o hay una solicitud con formato incorrecto, el servicio de metadatos de instancia devuelve errores HTTP estándar. Por ejemplo:

Código de estado HTTP | Motivo
----------------|-------
200 OK |
400 - Solicitud incorrecta | Falta `Metadata: true` encabezado o falta el formato al consultar un nodo hoja
405 No encontrado | El elemento solicitado no existe
405 Método no permitido | Solo se admiten solicitudes `GET` y `POST`
429 Demasiadas solicitudes | La API es compatible actualmente con un máximo de cinco consultas por segundo
500 Error de servicio     | Vuelva a intentarlo más tarde

### <a name="examples"></a>Ejemplos

> [!NOTE]
> Todas las respuestas de las API son cadenas JSON. Todas las respuestas de ejemplo siguientes son impreso para mejorar la legibilidad.

#### <a name="retrieving-network-information"></a>Recuperación de información de red

**Solicitud**

```bash
curl -H Metadata:true "http://169.254.169.254/metadata/instance/network?api-version=2017-08-01"
```

**Respuesta**

> [!NOTE]
> La respuesta es una cadena JSON. La respuesta de ejemplo siguiente se ha impreso correctamente para mejorar la legibilidad.

```json
{
  "interface": [
    {
      "ipv4": {
        "ipAddress": [
          {
            "privateIpAddress": "10.1.0.4",
            "publicIpAddress": "X.X.X.X"
          }
        ],
        "subnet": [
          {
            "address": "10.1.0.0",
            "prefix": "24"
          }
        ]
      },
      "ipv6": {
        "ipAddress": []
      },
      "macAddress": "000D3AF806EC"
    }
  ]
}

```

#### <a name="retrieving-public-ip-address"></a>Recuperación de dirección IP pública

```bash
curl -H Metadata:true "http://169.254.169.254/metadata/instance/network/interface/0/ipv4/ipAddress/0/publicIpAddress?api-version=2017-08-01&format=text"
```

#### <a name="retrieving-all-metadata-for-an-instance"></a>Recuperación de todos los metadatos para una instancia

**Solicitud**

```bash
curl -H Metadata:true "http://169.254.169.254/metadata/instance?api-version=2018-10-01"
```

**Respuesta**

> [!NOTE]
> La respuesta es una cadena JSON. La respuesta de ejemplo siguiente se ha impreso correctamente para mejorar la legibilidad.

```json
{
  "compute": {
    "azEnvironment": "AZUREPUBLICCLOUD",
    "location": "westus",
    "name": "jubilee",
    "offer": "Windows-10",
    "osType": "Windows",
    "placementGroupId": "",
    "plan": {
        "name": "",
        "product": "",
        "publisher": ""
    },
    "platformFaultDomain": "1",
    "platformUpdateDomain": "1",
    "provider": "Microsoft.Compute",
    "publicKeys": [],
    "publisher": "MicrosoftWindowsDesktop",
    "resourceGroupName": "myrg",
    "sku": "rs4-pro",
    "subscriptionId": "xxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxx",
    "tags": "Department:IT;Environment:Prod;Role:WorkerRole",
    "version": "17134.345.59",
    "vmId": "13f56399-bd52-4150-9748-7190aae1ff21",
    "vmScaleSetName": "",
    "vmSize": "Standard_D1",
    "zone": "1"
  },
  "network": {
    "interface": [
      {
        "ipv4": {
          "ipAddress": [
            {
              "privateIpAddress": "10.1.2.5",
              "publicIpAddress": "X.X.X.X"
            }
          ],
          "subnet": [
            {
              "address": "10.1.2.0",
              "prefix": "24"
            }
          ]
        },
        "ipv6": {
          "ipAddress": []
        },
        "macAddress": "000D3A36DDED"
      }
    ]
  }
}
```

#### <a name="retrieving-metadata-in-windows-virtual-machine"></a>Recuperación de metadatos en una máquina virtual con Windows

**Solicitud**

Los metadatos de instancia se pueden recuperar en Windows a través del programa `curl`:

```bash
curl -H @{'Metadata'='true'} http://169.254.169.254/metadata/instance?api-version=2018-10-01 | select -ExpandProperty Content
```

O a través del cmdlet de PowerShell `Invoke-RestMethod`:

```powershell

Invoke-RestMethod -Headers @{"Metadata"="true"} -URI http://169.254.169.254/metadata/instance?api-version=2018-10-01 -Method get
```

**Respuesta**

> [!NOTE]
> La respuesta es una cadena JSON. La respuesta de ejemplo siguiente se ha impreso correctamente para mejorar la legibilidad.

```json
{
  "compute": {
    "azEnvironment": "AZUREPUBLICCLOUD",
    "location": "westus",
    "name": "SQLTest",
    "offer": "SQL2016SP1-WS2016",
    "osType": "Windows",
    "placementGroupId": "",
    "plan": {
        "name": "",
        "product": "",
        "publisher": ""
    },
    "platformFaultDomain": "0",
    "platformUpdateDomain": "0",
    "provider": "Microsoft.Compute",
    "publicKeys": [],
    "publisher": "MicrosoftSQLServer",
    "resourceGroupName": "myrg",
    "sku": "Enterprise",
    "subscriptionId": "xxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxx",
    "tags": "Department:IT;Environment:Test;Role:WebRole",
    "version": "13.0.400110",
    "vmId": "453945c8-3923-4366-b2d3-ea4c80e9b70e",
    "vmScaleSetName": "",
    "vmSize": "Standard_DS2",
    "zone": ""
  },
  "network": {
    "interface": [
      {
        "ipv4": {
          "ipAddress": [
            {
              "privateIpAddress": "10.0.1.4",
              "publicIpAddress": "X.X.X.X"
            }
          ],
          "subnet": [
            {
              "address": "10.0.1.0",
              "prefix": "24"
            }
          ]
        },
        "ipv6": {
          "ipAddress": []
        },
        "macAddress": "002248020E1E"
      }
    ]
  }
}
```

## <a name="metadata-apis"></a>API de metadatos

#### <a name="the-following-apis-are-available-through-the-metadata-endpoint"></a>Las API siguientes están disponibles a través del extremo de metadatos:

Datos | DESCRIPCIÓN | Versión de introducción
-----|-------------|-----------------------
attested | Consulte [Datos atestiguados](#attested-data) | 01 de octubre de 2018
identidad | Identidades administradas de recursos de Azure. Consulte [Obtener un token de acceso](../../active-directory/managed-identities-azure-resources/how-to-use-vm-token.md) | 2018-02-01
instancia | Consulte [API de instancia](#instance-api) | 2017-04-02
scheduledevents | Consulte [Scheduled Events](scheduled-events.md). | 2017-08-01

#### <a name="instance-api"></a>Instancia de API
##### <a name="the-following-compute-categories-are-available-through-the-instance-api"></a>Las siguientes categorías de proceso están disponibles a través de la API de instancia:

> [!NOTE]
> A través del extremo de metadatos, las siguientes categorías se accede a través de la instancia y los procesos

Datos | DESCRIPCIÓN | Versión de introducción
-----|-------------|-----------------------
azEnvironment | Entorno de Azure donde se ejecuta la VM. | 01 de octubre de 2018
customData | Consulte [datos personalizados](#custom-data) | 2019-02-01
ubicación | La región de Azure donde se ejecuta la máquina virtual | 2017-04-02
Nombre | Nombre de la máquina virtual | 2017-04-02
offer | Ofrecen información para la imagen de máquina virtual y se implementa sólo está presente para las imágenes de la Galería de imágenes de Azure | 2017-04-02
osType | Linux o Windows | 2017-04-02
placementGroupId | [Grupo de selección de ubicación](../../virtual-machine-scale-sets/virtual-machine-scale-sets-placement-groups.md) de su conjunto de escalado de máquina virtual | 2017-08-01
plan | [Planear](https://docs.microsoft.com/rest/api/compute/virtualmachines/createorupdate#plan) que contiene el nombre, el producto y el publicador para una máquina virtual si su una imagen de Marketplace de Azure | 2018-04-02
platformUpdateDomain |  El [dominio de actualización](manage-availability.md) en que se ejecuta la máquina virtual. | 2017-04-02
platformFaultDomain | El [dominio de error](manage-availability.md) en que se ejecuta la máquina virtual. | 2017-04-02
proveedor | Proveedor de la máquina virtual | 01 de octubre de 2018
publicKeys | [Colección de claves públicas](https://docs.microsoft.com/rest/api/compute/virtualmachines/createorupdate#sshpublickey) asignado a la máquina virtual y las rutas de acceso | 2018-04-02
publisher | Publicador de la imagen de VM | 2017-04-02
resourceGroupName | [Grupo de recursos](../../azure-resource-manager/resource-group-overview.md) para su máquina virtual | 2017-08-01
sku | SKU específica de la imagen de VM | 2017-04-02
subscriptionId | Suscripción de Azure para la máquina virtual | 2017-08-01
tags | [Etiquetas](../../azure-resource-manager/resource-group-using-tags.md) para su máquina virtual  | 2017-08-01
version | Versión de la imagen de máquina virtual | 2017-04-02
vmId | [Identificador único](https://azure.microsoft.com/blog/accessing-and-using-azure-vm-unique-id/) para la máquina virtual. | 2017-04-02
vmScaleSetName | [Nombre del conjunto de escalado de máquina virtual](../../virtual-machine-scale-sets/virtual-machine-scale-sets-overview.md) de conjunto de escalado de la máquina virtual | 2017-12-01
vmSize | [Tamaño de VM](sizes.md) | 2017-04-02
zona | [Zona de disponibilidad](../../availability-zones/az-overview.md) de la máquina virtual | 2017-12-01

##### <a name="the-following-network-categories-are-available-through-the-instance-api"></a>Las siguientes categorías de red están disponibles a través de la API de instancia:

> [!NOTE]
> A través del extremo de metadatos, las siguientes categorías se accede a través de la instancia/red/interfaz

Datos | DESCRIPCIÓN | Versión de introducción
-----|-------------|-----------------------
ipv4/privateIpAddress | Dirección IPv4 local de la máquina virtual | 2017-04-02
ipv4/publicIpAddress | Dirección IPv4 pública de la máquina virtual | 2017-04-02
subnet/address | Dirección de subred de la máquina virtual | 2017-04-02
subnet/prefix | Prefijo de la subred, ejemplo, 24 | 2017-04-02
ipv6/ipAddress | Dirección IPv6 local de la máquina virtual | 2017-04-02
macAddress | Dirección de MAC de la VM | 2017-04-02

## <a name="attested-data"></a>Datos atestiguados

Instance Metadata responde en el punto de conexión HTTP en 169.254.169.254. Parte del escenario que sirve Instance Metadata Service está destinado a proporcionar la garantía de que los datos respondidos vienen de Azure. Firmamos una parte de esta información para que las imágenes de Marketplace puedan tener la seguridad de que su imagen se ejecuta en Azure.

### <a name="example-attested-data"></a>Datos atestiguados de ejemplo

> [!NOTE]
> Todas las respuestas de las API son cadenas JSON. Las siguientes respuestas de ejemplo se han impreso correctamente para facilitar su lectura.

 **Solicitud**

 ```bash
curl -H Metadata:true "http://169.254.169.254/metadata/attested/document?api-version=2018-10-01&nonce=1234567890"

```

Api-version es un campo obligatorio y la versión compatible con los datos atestiguados es la 2018-10-01.
Nonce es una cadena de 10 dígitos opcional que se proporciona. Nonce se puede usar para realizar un seguimiento de la solicitud y, si no se proporciona, la marca de tiempo UTC actual se devuelve en la cadena codificada de respuesta.

 **Respuesta**

> [!NOTE]
> La respuesta es una cadena JSON. La respuesta de ejemplo siguiente se ha impreso correctamente para mejorar la legibilidad.

 ```json
{
 "encoding":"pkcs7","signature":"MIIEEgYJKoZIhvcNAQcCoIIEAzCCA/8CAQExDzANBgkqhkiG9w0BAQsFADCBugYJKoZIhvcNAQcBoIGsBIGpeyJub25jZSI6IjEyMzQ1NjY3NjYiLCJwbGFuIjp7Im5hbWUiOiIiLCJwcm9kdWN0IjoiIiwicHVibGlzaGVyIjoiIn0sInRpbWVTdGFtcCI6eyJjcmVhdGVkT24iOiIxMS8yMC8xOCAyMjowNzozOSAtMDAwMCIsImV4cGlyZXNPbiI6IjExLzIwLzE4IDIyOjA4OjI0IC0wMDAwIn0sInZtSWQiOiIifaCCAj8wggI7MIIBpKADAgECAhBnxW5Kh8dslEBA0E2mIBJ0MA0GCSqGSIb3DQEBBAUAMCsxKTAnBgNVBAMTIHRlc3RzdWJkb21haW4ubWV0YWRhdGEuYXp1cmUuY29tMB4XDTE4MTEyMDIxNTc1N1oXDTE4MTIyMDIxNTc1NlowKzEpMCcGA1UEAxMgdGVzdHN1YmRvbWFpbi5tZXRhZGF0YS5henVyZS5jb20wgZ8wDQYJKoZIhvcNAQEBBQADgY0AMIGJAoGBAML/tBo86ENWPzmXZ0kPkX5dY5QZ150mA8lommszE71x2sCLonzv4/UWk4H+jMMWRRwIea2CuQ5RhdWAHvKq6if4okKNt66fxm+YTVz9z0CTfCLmLT+nsdfOAsG1xZppEapC0Cd9vD6NCKyE8aYI1pliaeOnFjG0WvMY04uWz2MdAgMBAAGjYDBeMFwGA1UdAQRVMFOAENnYkHLa04Ut4Mpt7TkJFfyhLTArMSkwJwYDVQQDEyB0ZXN0c3ViZG9tYWluLm1ldGFkYXRhLmF6dXJlLmNvbYIQZ8VuSofHbJRAQNBNpiASdDANBgkqhkiG9w0BAQQFAAOBgQCLSM6aX5Bs1KHCJp4VQtxZPzXF71rVKCocHy3N9PTJQ9Fpnd+bYw2vSpQHg/AiG82WuDFpPReJvr7Pa938mZqW9HUOGjQKK2FYDTg6fXD8pkPdyghlX5boGWAMMrf7bFkup+lsT+n2tRw2wbNknO1tQ0wICtqy2VqzWwLi45RBwTGB6DCB5QIBATA/MCsxKTAnBgNVBAMTIHRlc3RzdWJkb21haW4ubWV0YWRhdGEuYXp1cmUuY29tAhBnxW5Kh8dslEBA0E2mIBJ0MA0GCSqGSIb3DQEBCwUAMA0GCSqGSIb3DQEBAQUABIGAld1BM/yYIqqv8SDE4kjQo3Ul/IKAVR8ETKcve5BAdGSNkTUooUGVniTXeuvDj5NkmazOaKZp9fEtByqqPOyw/nlXaZgOO44HDGiPUJ90xVYmfeK6p9RpJBu6kiKhnnYTelUk5u75phe5ZbMZfBhuPhXmYAdjc7Nmw97nx8NnprQ="
}
```

> El blob de firma es una versión con la firma [pkcs7](https://aka.ms/pkcs7) del documento. Contiene el certificado usado para firmar, junto con detalles de la máquina virtual como vmld, nonce y timeStamp para la creación y expiración del documento y la información del plan acerca de la imagen. La información del plan solo se rellena para las imágenes de Azure Marketplace. El certificado se puede extraer de la respuesta y usarse para validar que la respuesta es válida y viene de Azure.

#### <a name="retrieving-attested-metadata-in-windows-virtual-machine"></a>Recuperación de metadatos atestiguados en una máquina virtual Windows

 **Solicitud**

Los metadatos de instancia se pueden recuperar en Windows a través de la utilidad `curl` de PowerShell:

 ```bash
curl -H @{'Metadata'='true'} "http://169.254.169.254/metadata/attested/document?api-version=2018-10-01&nonce=1234567890" | select -ExpandProperty Content
```

 O bien a través del cmdlet `Invoke-RestMethod`:

 ```powershell
Invoke-RestMethod -Headers @{"Metadata"="true"} -URI "http://169.254.169.254/metadata/attested/document?api-version=2018-10-01&nonce=1234567890" -Method get
```

Api-version es un campo obligatorio y la versión compatible con los datos atestiguados es la 2018-10-01.
Nonce es una cadena de 10 dígitos opcional que se proporciona. Nonce se puede usar para realizar un seguimiento de la solicitud y, si no se proporciona, la marca de tiempo UTC actual se devuelve en la cadena codificada de respuesta.

 **Respuesta**

> [!NOTE]
> La respuesta es una cadena JSON. La respuesta de ejemplo siguiente se ha impreso correctamente para mejorar la legibilidad.

 ```json
{
 "encoding":"pkcs7","signature":"MIIEEgYJKoZIhvcNAQcCoIIEAzCCA/8CAQExDzANBgkqhkiG9w0BAQsFADCBugYJKoZIhvcNAQcBoIGsBIGpeyJub25jZSI6IjEyMzQ1NjY3NjYiLCJwbGFuIjp7Im5hbWUiOiIiLCJwcm9kdWN0IjoiIiwicHVibGlzaGVyIjoiIn0sInRpbWVTdGFtcCI6eyJjcmVhdGVkT24iOiIxMS8yMC8xOCAyMjowNzozOSAtMDAwMCIsImV4cGlyZXNPbiI6IjExLzIwLzE4IDIyOjA4OjI0IC0wMDAwIn0sInZtSWQiOiIifaCCAj8wggI7MIIBpKADAgECAhBnxW5Kh8dslEBA0E2mIBJ0MA0GCSqGSIb3DQEBBAUAMCsxKTAnBgNVBAMTIHRlc3RzdWJkb21haW4ubWV0YWRhdGEuYXp1cmUuY29tMB4XDTE4MTEyMDIxNTc1N1oXDTE4MTIyMDIxNTc1NlowKzEpMCcGA1UEAxMgdGVzdHN1YmRvbWFpbi5tZXRhZGF0YS5henVyZS5jb20wgZ8wDQYJKoZIhvcNAQEBBQADgY0AMIGJAoGBAML/tBo86ENWPzmXZ0kPkX5dY5QZ150mA8lommszE71x2sCLonzv4/UWk4H+jMMWRRwIea2CuQ5RhdWAHvKq6if4okKNt66fxm+YTVz9z0CTfCLmLT+nsdfOAsG1xZppEapC0Cd9vD6NCKyE8aYI1pliaeOnFjG0WvMY04uWz2MdAgMBAAGjYDBeMFwGA1UdAQRVMFOAENnYkHLa04Ut4Mpt7TkJFfyhLTArMSkwJwYDVQQDEyB0ZXN0c3ViZG9tYWluLm1ldGFkYXRhLmF6dXJlLmNvbYIQZ8VuSofHbJRAQNBNpiASdDANBgkqhkiG9w0BAQQFAAOBgQCLSM6aX5Bs1KHCJp4VQtxZPzXF71rVKCocHy3N9PTJQ9Fpnd+bYw2vSpQHg/AiG82WuDFpPReJvr7Pa938mZqW9HUOGjQKK2FYDTg6fXD8pkPdyghlX5boGWAMMrf7bFkup+lsT+n2tRw2wbNknO1tQ0wICtqy2VqzWwLi45RBwTGB6DCB5QIBATA/MCsxKTAnBgNVBAMTIHRlc3RzdWJkb21haW4ubWV0YWRhdGEuYXp1cmUuY29tAhBnxW5Kh8dslEBA0E2mIBJ0MA0GCSqGSIb3DQEBCwUAMA0GCSqGSIb3DQEBAQUABIGAld1BM/yYIqqv8SDE4kjQo3Ul/IKAVR8ETKcve5BAdGSNkTUooUGVniTXeuvDj5NkmazOaKZp9fEtByqqPOyw/nlXaZgOO44HDGiPUJ90xVYmfeK6p9RpJBu6kiKhnnYTelUk5u75phe5ZbMZfBhuPhXmYAdjc7Nmw97nx8NnprQ="
}
```

> El blob de firma es una versión con la firma [pkcs7](https://aka.ms/pkcs7) del documento. Contiene el certificado usado para firmar, junto con detalles de la máquina virtual como vmld, nonce y timeStamp para la creación y expiración del documento y la información del plan acerca de la imagen. La información del plan solo se rellena para las imágenes de Azure Marketplace. El certificado se puede extraer de la respuesta y usarse para validar que la respuesta es válida y viene de Azure.


## <a name="example-scenarios-for-usage"></a>Escenarios de ejemplo de uso  

### <a name="tracking-vm-running-on-azure"></a>Seguimiento de una máquina virtual que se ejecuta en Azure

Como proveedor de servicios, es posible que necesite hacer seguimiento de la cantidad de máquinas virtuales que ejecutan su software o que tenga agentes que deban hacer seguimiento de la unicidad de la máquina virtual. Para poder obtener un identificador único para una máquina virtual, use el campo `vmId` del servicio de metadatos de instancia.

**Solicitud**

```bash
curl -H Metadata:true "http://169.254.169.254/metadata/instance/compute/vmId?api-version=2017-08-01&format=text"
```

**Respuesta**

```text
5c08b38e-4d57-4c23-ac45-aca61037f084
```

### <a name="placement-of-containers-data-partitions-based-faultupdate-domain"></a>Ubicación de los contenedores y el dominio de error/actualización basado en particiones de datos 

Para ciertos escenarios, la ubicación de las distintas réplicas de datos es de máxima importancia. Por ejemplo, para [la ubicación de réplicas de HDFS](https://hadoop.apache.org/docs/stable/hadoop-project-dist/hadoop-hdfs/HdfsDesign.html#Replica_Placement:_The_First_Baby_Steps) o la ubicación de contenedores a través de un [orquestador](https://kubernetes.io/docs/user-guide/node-selection/) se debe saber en qué `platformFaultDomain` y `platformUpdateDomain` se ejecuta la máquina virtual.
También puede usar las [zonas de disponibilidad](../../availability-zones/az-overview.md) para las instancias para tomar estas decisiones.
Puede consultar directamente estos datos a través del servicio de metadatos de instancia.

**Solicitud**

```bash
curl -H Metadata:true "http://169.254.169.254/metadata/instance/compute/platformFaultDomain?api-version=2017-08-01&format=text"
```

**Respuesta**

```text
0
```

### <a name="getting-more-information-about-the-vm-during-support-case"></a>Obtención de más información sobre la VM durante el caso de soporte técnico

Como proveedor de servicios, es posible que reciba una llamada de soporte técnico en la que le gustaría tener más información sobre la máquina virtual. Pedirle al cliente que comparta los metadatos del equipo puede proporcionar información básica para que el profesional de soporte técnico conozca la variante de máquina virtual en Azure. 

**Solicitud**

```bash
curl -H Metadata:true "http://169.254.169.254/metadata/instance/compute?api-version=2017-08-01"
```

**Respuesta**

> [!NOTE]
> La respuesta es una cadena JSON. La respuesta de ejemplo siguiente se ha impreso correctamente para mejorar la legibilidad.

```json
{
  "compute": {
    "location": "CentralUS",
    "name": "IMDSCanary",
    "offer": "RHEL",
    "osType": "Linux",
    "platformFaultDomain": "0",
    "platformUpdateDomain": "0",
    "publisher": "RedHat",
    "sku": "7.2",
    "version": "7.2.20161026",
    "vmId": "5c08b38e-4d57-4c23-ac45-aca61037f084",
    "vmSize": "Standard_DS2"
  }
}
```

### <a name="getting-azure-environment-where-the-vm-is-running"></a>Obtención del entorno de Azure donde se está ejecutando la máquina virtual

Azure tiene varias nubes soberanas, como [Azure Government](https://azure.microsoft.com/overview/clouds/government/). En ocasiones, necesitará el entorno de Azure para tomar algunas decisiones acerca del tiempo de ejecución. En el siguiente ejemplo se muestra cómo lograr este comportamiento.

**Solicitud**
```bash
curl -H Metadata:true "http://169.254.169.254/metadata/instance/compute/azEnvironment?api-version=2018-10-01&format=text"
```

**Respuesta**
```bash
AZUREPUBLICCLOUD
```

### <a name="getting-the-tags-for-the-vm"></a>Introducción a las etiquetas de la máquina virtual

Las etiquetas se hayan aplicado a la máquina virtual de Azure para organizarlos de forma lógica en una taxonomía. Las etiquetas asignadas a una máquina virtual se pueden recuperar mediante el uso de la solicitud siguiente.

**Solicitud**

```bash
curl -H Metadata:true "http://169.254.169.254/metadata/instance/compute/tags?api-version=2018-10-01&format=text"
```

**Respuesta**

```text
Department:IT;Environment:Test;Role:WebRole
```

> [!NOTE]
> Las etiquetas están separado de punto y coma. Si se escribe un analizador para extraer mediante programación las etiquetas, los valores y nombres de etiqueta no deben contener puntos y coma en orden para el analizador funcione correctamente.

### <a name="validating-that-the-vm-is-running-in-azure"></a>Validación de que la máquina virtual se ejecuta en Azure

Los proveedores de Marketplace desean asegurarse de que su software tiene licencia para ejecutarse solo en Azure. Si alguien copia el disco duro virtual en un entorno local, debería disponer de un capacidad para detectarlo. Mediante una llamada al servicio de metadatos de instancia, los proveedores pueden obtener datos firmados que garantizan la respuesta únicamente de Azure.

> [!NOTE]
> Requiere jq para instalarse.

**Solicitud**

 ```bash
  # Get the signature
   curl  --silent -H Metadata:True http://169.254.169.254/metadata/attested/document?api-version=2018-10-01 | jq -r '.["signature"]' > signature
  # Decode the signature
  base64 -d signature > decodedsignature
  #Get PKCS7 format
  openssl pkcs7 -in decodedsignature -inform DER -out sign.pk7
  # Get Public key out of pkc7
  openssl pkcs7 -in decodedsignature -inform DER  -print_certs -out signer.pem
  #Get the intermediate certificate
  wget -q -O intermediate.cer "$(openssl x509 -in signer.pem -text -noout | grep " CA Issuers -" | awk -FURI: '{print $2}')"
  openssl x509 -inform der -in intermediate.cer -out intermediate.pem
  #Verify the contents
  openssl smime -verify -in sign.pk7 -inform pem -noverify
 ```

 **Respuesta**

```json
Verification successful
{"nonce":"20181128-001617",
  "plan":
    {
     "name":"",
     "product":"",
     "publisher":""
    },
"timeStamp":
  {
    "createdOn":"11/28/18 00:16:17 -0000",
    "expiresOn":"11/28/18 06:16:17 -0000"
  },
"vmId":"d3e0e374-fda6-4649-bbc9-7f20dc379f34"
}
```

Datos | DESCRIPCIÓN
-----|------------
valor de seguridad | El usuario especificó una cadena opcional con la solicitud. Si no se proporcionó ningún nonce en la solicitud, se devuelve la marca de tiempo UTC actual
plan | [Plan](https://docs.microsoft.com/rest/api/compute/virtualmachines/createorupdate#plan) en una máquina virtual es una imagen de Azure Marketplace que contiene el nombre, producto y editor
timestamp/createdOn | La marca de tiempo en la que se creó el primer documento firmado
timestamp/expiresOn | La marca de tiempo en la que expira el documento firmado
vmId |  [Identificador único](https://azure.microsoft.com/blog/accessing-and-using-azure-vm-unique-id/) para la máquina virtual.

#### <a name="verifying-the-signature"></a>Comprobación de la firma

Una vez que obtenga la firma anterior, podrá comprobar que esta es de Microsoft. También podrá comprobar el certificado intermedio y la cadena de certificados.

> [!NOTE]
> El certificado para la nube pública y la nube soberana serán diferentes.

 Regiones | Certificate
---------|-----------------
[Todas las regiones globales de Azure disponibles con carácter general](https://azure.microsoft.com/regions/)     | metadata.azure.com
[Azure Government](https://azure.microsoft.com/overview/clouds/government/)              | metadata.azure.us
[Azure en China](https://www.azure.cn/)                                                           | metadata.azure.cn
[Azure Alemania](https://azure.microsoft.com/overview/clouds/germany/)                    | metadata.microsoftazure.de

```bash

# Verify the subject name for the main certificate
openssl x509 -noout -subject -in signer.pem
# Verify the issuer for the main certificate
openssl x509 -noout -issuer -in signer.pem
#Validate the subject name for intermediate certificate
openssl x509 -noout -subject -in intermediate.pem
# Verify the issuer for the intermediate certificate
openssl x509 -noout -issuer -in intermediate.pem
# Verify the certificate chain
openssl verify -verbose -CAfile /etc/ssl/certs/Baltimore_CyberTrust_Root.pem -untrusted intermediate.pem signer.pem
```

En casos donde el certificado intermedio no se pueden descargar debido a restricciones de red durante la validación, se puede anclar el certificado intermedio. Sin embargo, Azure se implementará a través de los certificados según el procedimiento PKI estándar. Los certificados anclados deberá actualizarse cuando se produce la sustitución. Cada vez que se ha planeado un cambio para actualizar el certificado intermedio, se actualizará el blog de Azure y le notificará los clientes de Azure. Pueden encontrar los certificados intermedios [aquí](https://www.microsoft.com/pki/mscorp/cps/default.htm). Los certificados intermedios para cada una de las regiones pueden ser diferentes.

### <a name="failover-clustering-in-windows-server"></a>Clústeres de conmutación por error de Windows Server

Para determinados escenarios, al consultar Instance Metadata Service con los clústeres de conmutación por error, es necesario agregar una ruta a la tabla de enrutamiento.

1. Abra un símbolo del sistema con privilegios de administrador.

2. Ejecute el siguiente comando y anote la dirección de la interfaz de red de destino (`0.0.0.0`) en la tabla de enrutamiento IPv4.

```bat
route print
```

> [!NOTE] 
> La siguiente salida de ejemplo de una máquina virtual de Windows Server con un clúster de conmutación por error habilitado contiene solo la tabla de rutas IPv4 por motivos de simplicidad.

```bat
IPv4 Route Table
===========================================================================
Active Routes:
Network Destination        Netmask          Gateway       Interface  Metric
          0.0.0.0          0.0.0.0         10.0.1.1        10.0.1.10    266
         10.0.1.0  255.255.255.192         On-link         10.0.1.10    266
        10.0.1.10  255.255.255.255         On-link         10.0.1.10    266
        10.0.1.15  255.255.255.255         On-link         10.0.1.10    266
        10.0.1.63  255.255.255.255         On-link         10.0.1.10    266
        127.0.0.0        255.0.0.0         On-link         127.0.0.1    331
        127.0.0.1  255.255.255.255         On-link         127.0.0.1    331
  127.255.255.255  255.255.255.255         On-link         127.0.0.1    331
      169.254.0.0      255.255.0.0         On-link     169.254.1.156    271
    169.254.1.156  255.255.255.255         On-link     169.254.1.156    271
  169.254.255.255  255.255.255.255         On-link     169.254.1.156    271
        224.0.0.0        240.0.0.0         On-link         127.0.0.1    331
        224.0.0.0        240.0.0.0         On-link     169.254.1.156    271
        224.0.0.0        240.0.0.0         On-link         10.0.1.10    266
  255.255.255.255  255.255.255.255         On-link         127.0.0.1    331
  255.255.255.255  255.255.255.255         On-link     169.254.1.156    271
  255.255.255.255  255.255.255.255         On-link         10.0.1.10    266
```

1. Ejecute el siguiente comando y use la dirección de la interfaz de red de destino (`0.0.0.0`) que es (`10.0.1.10`) en el ejemplo.

```bat
route add 169.254.169.254/32 10.0.1.10 metric 1 -p
```

### <a name="custom-data"></a>Datos personalizados
Instance Metadata Service proporciona la capacidad de la máquina virtual tener acceso a sus datos personalizados. Los datos binarios deben ser inferior a 64 KB y se proporcionan a la máquina virtual en formato codificado en base64.

Datos personalizados de Azure se pueden insertar en la máquina virtual a través de las API de REST, Cmdlets de PowerShell, interfaz de línea de comandos (CLI) de Azure o una plantilla de ARM.

Para obtener un ejemplo de la interfaz de línea de comandos de Azure, consulte [datos personalizados y Cloud-Init en Microsoft Azure](https://azure.microsoft.com/blog/custom-data-and-cloud-init-on-windows-azure/).

Para obtener un ejemplo de plantilla ARM, consulte [implementar una máquina Virtual con CustomData](https://github.com/Azure/azure-quickstart-templates/tree/master/101-vm-customdata).

Datos personalizados están disponibles para todos los procesos que se ejecutan en la máquina virtual. Se recomienda que los clientes no inserción información secreta en datos personalizados.

Actualmente, se garantiza que los datos personalizados estén disponibles durante el arranque de una máquina virtual. Si las actualizaciones se realizan en la máquina virtual como discos de agregar o cambiar el tamaño de la máquina virtual, Instance Metadata Service no proporcionará datos personalizados. Proporcionar CustomData forma persistente a través de Instance Metadata Service está actualmente en curso.

#### <a name="retrieving-custom-data-in-virtual-machine"></a>Recuperación de datos personalizado en la máquina Virtual
Instance Metadata Service proporciona los datos personalizados a la máquina virtual en formato codificado en base64. En el ejemplo siguiente, se descodifica la cadena codificada en base64.

> [!NOTE]
> Los datos personalizados en este ejemplo se interpretan como una cadena ASCII que dice "Mis datos personalizados.".

**Solicitud**

```bash
curl -H "Metadata:true" "http://169.254.169.254/metadata/instance/compute/customData?api-version=2019-02-01&&format=text" | base64 --decode
```

**Respuesta**

```text
My custom data.
```

### <a name="examples-of-calling-metadata-service-using-different-languages-inside-the-vm"></a>Ejemplos de llamadas al servicio de metadatos mediante lenguajes diferentes dentro de la máquina virtual 

Idioma | Ejemplo
---------|----------------
Ruby     | https://github.com/Microsoft/azureimds/blob/master/IMDSSample.rb
Ir  | https://github.com/Microsoft/azureimds/blob/master/imdssample.go
Python   | https://github.com/Microsoft/azureimds/blob/master/IMDSSample.py
C++      | https://github.com/Microsoft/azureimds/blob/master/IMDSSample-windows.cpp
C#       | https://github.com/Microsoft/azureimds/blob/master/IMDSSample.cs
JavaScript | https://github.com/Microsoft/azureimds/blob/master/IMDSSample.js
PowerShell | https://github.com/Microsoft/azureimds/blob/master/IMDSSample.ps1
Bash       | https://github.com/Microsoft/azureimds/blob/master/IMDSSample.sh
Perl       | https://github.com/Microsoft/azureimds/blob/master/IMDSSample.pl
Java       | https://github.com/Microsoft/azureimds/blob/master/imdssample.java
Visual Basic | https://github.com/Microsoft/azureimds/blob/master/IMDSSample.vb
Puppet | https://github.com/keirans/azuremetadata

## <a name="faq"></a>Preguntas más frecuentes

1. Obtengo el error `400 Bad Request, Required metadata header not specified`. ¿Qué significa?
   * El servicio de metadatos de instancia requiere que el encabezado `Metadata: true` se transmita en la solicitud. Transmitir este encabezado en la llamada de REST permite tener acceso al servicio de metadatos de instancia.
2. ¿Por qué no recibo información de proceso de mi máquina virtual?
   * Actualmente el servicio de metadatos de instancia solo admite instancias creadas con Azure Resource Manager. En el futuro, es posible que se agregue compatibilidad para máquinas virtuales de servicio en la nube.
3. Creé mi máquina virtual mediante Azure Resource Manager hace un tiempo. ¿Por qué no veo la información de metadatos de proceso?
   * En el caso de las máquinas virtuales que se crearon después de septiembre de 2016, agregue una [etiqueta](../../azure-resource-manager/resource-group-using-tags.md) para empezar a ver los metadatos de proceso. En el caso de máquinas virtuales anteriores (creadas antes de septiembre de 2016), agregue o quite extensiones o discos de datos a la máquina virtual para actualizar los metadatos.
4. No veo todos los datos rellenados para la nueva versión
   * En el caso de las máquinas virtuales que se crearon después de septiembre de 2016, agregue una [etiqueta](../../azure-resource-manager/resource-group-using-tags.md) para empezar a ver los metadatos de proceso. En el caso de máquinas virtuales anteriores (creadas antes de septiembre de 2016), agregue o quite extensiones o discos de datos a la máquina virtual para actualizar los metadatos.
5. ¿Por qué recibo el error `500 Internal Server Error`?
   * Vuelva a intentar la solicitud en función del sistema de interrupción exponencial. Si el problema persiste, póngase en contacto con el servicio de soporte técnico de Azure.
6. ¿Dónde puedo compartir preguntas o comentarios adicionales?
   * Envíe sus comentarios en https://feedback.azure.com.
7. ¿Esto funciona para la instancia del conjunto de escala de máquinas virtuales?
   * Sí, el servicio de metadatos está disponible para las instancias del conjunto de escala.
8. ¿Cómo puedo obtener soporte técnico para el servicio?
   * Para obtener soporte técnico para el servicio, cree un problema de compatibilidad en Azure Portal en la máquina virtual en la que no obtiene respuesta de metadatos después de reintentos prolongados.
9. ¿Por qué se agotó el tiempo de espera de solicitud para mi llamada al servicio?
   * Se deben realizar llamadas de metadatos desde la dirección IP principal asignada a la tarjeta de red de la máquina virtual; además, en caso de que haya cambiado las rutas, debe tener una ruta para la dirección 169.254.0.0/16 desde su tarjeta de red.
10. He actualizado mis etiquetas en el conjunto de escalado de máquinas virtuales, pero no aparecen en las instancias, a diferencia de las máquinas virtuales.
    * Actualmente, las etiquetas para ScaleSets solo se muestran a la máquina virtual durante un reinicio, restablecimiento de imagen o cambio de disco en la instancia.

    ![Soporte técnico de metadatos de instancia](./media/instance-metadata-service/InstanceMetadata-support.png)

## <a name="next-steps"></a>Pasos siguientes

- Más información sobre los [eventos programados](scheduled-events.md).
