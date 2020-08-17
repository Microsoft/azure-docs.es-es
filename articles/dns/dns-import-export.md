---
title: Importar y exportar un archivo de zona de dominio de la CLI de Azure
titleSuffix: Azure DNS
description: Más información sobre cómo importar y exportar un archivo de zona DNS en DNS de Azure mediante la CLI de Azure
services: dns
author: rohinkoul
ms.service: dns
ms.date: 7/30/2020
ms.author: rohink
ms.topic: how-to
ms.openlocfilehash: 8f17d7f32d774cac283ec335c372e8f68d50931f
ms.sourcegitcommit: e71da24cc108efc2c194007f976f74dd596ab013
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/29/2020
ms.locfileid: "87424229"
---
# <a name="import-and-export-a-dns-zone-file-using-the-azure-cli"></a>Importación y exportación de un archivo de zona DNS mediante la CLI de Azure

Este artículo le guiará a través de la importación y exportación de archivos de zona DNS para Azure DNS con la CLI de Azure.

## <a name="introduction-to-dns-zone-migration"></a>Introducción a la migración de zona DNS

Un archivo de zona DNS es un archivo de texto que contiene los detalles de cada registro DNS (Sistema de nombres de dominio) de la zona. Sigue un formato estándar, por lo que es adecuado para transferir registros DNS entre distintos sistemas DNS. Usar un archivo de zona es una manera rápida, confiable y cómoda de transferir una zona DNS a DNS de Azure o desde él.

DNS de Azure admite la importación y la exportación de archivos de zona mediante la interfaz de la línea de comandos (CLI) de Azure. La importación de archivos de zona **no** se permite actualmente mediante Azure PowerShell o el portal de Azure.

La CLI de Azure es una herramienta de línea de comandos multiplataforma que se usa para administrar servicios de Azure. Está disponible para las plataformas Windows, Mac y Linux en la [página de descargas de Azure](https://azure.microsoft.com/downloads/). La compatibilidad multiplataforma es importante para la importación y la exportación de archivos de zona, porque el software de servidor de nombres más común, [BIND](https://www.isc.org/downloads/bind/), se suele ejecutar en Linux.

## <a name="obtain-your-existing-dns-zone-file"></a>Obtención del archivo de zona DNS existente

Antes de importar un archivo de zona DNS a DNS de Azure, debe obtener una copia del archivo de zona. El origen de este archivo varía en función de dónde se hospede la zona DNS.

* Si la zona DNS se hospeda en un servicio de colaboradores (como un registrador de dominios, un proveedor de hospedaje DNS dedicado o un proveedor de nube alternativo), ese servicio debería ofrecer la posibilidad de descargar el archivo de zona DNS.
* Si la zona DNS se hospeda en DNS de Windows, la carpeta predeterminada para los archivos de zona es **%systemroot%\system32\dns**. También se muestra la ruta de acceso completa de cada archivo de zona en la pestaña **General** de la consola de DNS.
* Si la zona DNS se hospeda con BIND, la ubicación del archivo de zona para cada zona se especifica en el archivo de configuración de BIND **named.conf**.

## <a name="import-a-dns-zone-file-into-azure-dns"></a>Importación de un archivo de zona DNS a DNS de Azure

Si aún no existe, al importar un archivo de zona se crea una nueva zona DNS de Azure. Si la zona ya existe, los conjuntos de registros de la zona deben combinarse con los conjuntos de registros existentes.

### <a name="merge-behavior"></a>Comportamiento de combinación

* De forma predeterminada, se combinan los conjuntos de registros nuevos y existentes. Los registros idénticos dentro de un conjunto de registros combinado se desduplican.
* Cuando se combinan conjuntos de registros, se usa el período de vida (TTL) de los conjuntos de registros existentes.
* Los parámetros de inicio de autoridad (SOA), a excepción de `host`, siempre se toman del archivo de zona importado. De forma similar, para el conjunto de registros de servidor de nombres en el ápice de zona, el TTL siempre se toma del archivo de zona importado.
* Un registro CNAME importado no reemplaza a un registro CNAME existente con el mismo nombre.  
* Cuando surge un conflicto entre un registro CNAME y otro registro del mismo nombre pero de distinto tipo (sin importar cuál sea el existente y cuál el nuevo), se conserva el registro existente. 

### <a name="additional-information-about-importing"></a>Información adicional sobre la importación

En las notas siguientes, se ofrecen detalles técnicos adicionales sobre el proceso de importación de zona.

* La directiva `$TTL` es opcional y se admite. Cuando no se indica ninguna directiva `$TTL`, los registros sin TTL explícito se importan con un TTL predeterminado de 3600 segundos. Cuando dos registros del mismo conjunto de registros especifican diferentes TTL, se usa el valor más bajo.
* La directiva `$ORIGIN` es opcional y se admite. Cuando no se establece ninguna directiva `$ORIGIN` , el valor predeterminado que se usa es el nombre de la zona según lo especificado en la línea de comandos (terminado en ".").
* No se admiten las directivas `$INCLUDE` ni `$GENERATE`.
* Se admiten estos tipos de registros: A, AAAA, CAA, CNAME, MX, NS, SOA, SRV y TXT.
* DNS de Azure crea automáticamente el registro SOA cuando se crea una zona. Cuando se importa un archivo de zona, todos los parámetros SOA se toman del archivo de zona *excepto* el parámetro `host`. Este parámetro usa el valor proporcionado por DNS de Azure. Esto se debe a que este parámetro debe hacer referencia al servidor de nombres principal proporcionado por DNS de Azure.
* DNS de Azure también crea automáticamente el conjunto de registros de servidor de nombres en el ápice de zona al crear la zona. Solo se importa el TTL de este conjunto de registros. Estos registros contienen los nombres de servidores de nombres proporcionados por DNS de Azure. No se sobrescriben los datos del registro con los valores contenidos en el archivo de zona importado.
* Durante la versión preliminar pública, DNS de Azure admite solamente registros TXT de cadena única. Los registros TXT multicadena se concatenan y se truncan tras 255 caracteres.

### <a name="cli-format-and-values"></a>Valores y formato de la CLI

El formato del comando de CLI de Azure para importar una zona DNS es:

```azurecli
az network dns zone import -g <resource group> -n <zone name> -f <zone file name>
```

Valores:

* `<resource group>` es el nombre del grupo de recursos para la zona en DNS de Azure.
* `<zone name>` es el nombre de la zona.
* `<zone file name>` es la ruta de acceso y el nombre del archivo de zona que se va a importar.

Si no existe una zona con este nombre en el grupo de recursos, se crea automáticamente. Si la zona ya existe, los conjuntos de registros importados se combinan con conjuntos de registros existentes. 

### <a name="step-1-import-a-zone-file"></a>Paso 1. Importación de un archivo de zona

Para importar un archivo de zona para la zona **contoso.com**.

1. Si todavía no tiene uno, debe crear un grupo de recursos de Resource Manager.

    ```azurecli
    az group create --resource-group myresourcegroup -l westeurope
    ```

2. Para importar la zona **contoso.com** del archivo **contoso.com.txt** a una nueva zona DNS en el grupo de recursos **myresourcegroup**, ejecute el comando `az network dns zone import`.<BR>Con este comando se carga el archivo de zona y se analiza. El comando ejecuta una serie de comandos en el servicio DNS de Azure para crear la zona y todos los conjuntos de registros de la zona. El comando notifica el progreso en la ventana de la consola, junto con los errores o las advertencias. Puesto que los conjuntos de registros se crean en serie, puede tardar unos minutos en importar un archivo de zona de gran tamaño.

    ```azurecli
    az network dns zone import -g myresourcegroup -n contoso.com -f contoso.com.txt
    ```

### <a name="step-2-verify-the-zone"></a>Paso 2. Comprobación de la zona

Para comprobar la zona DNS después de importar el archivo, puede utilizar cualquiera de los métodos siguientes:

* Puede mostrar una lista de los registros mediante el siguiente comando de CLI de Azure:

    ```azurecli
    az network dns record-set list -g myresourcegroup -z contoso.com
    ```

* Puede mostrar una lista de los registros mediante el siguiente comando de la CLI de Azure: `az network dns record-set ns list`.
* O puede utilizar `nslookup` para comprobar la resolución de nombres para los registros. Como la zona aún no está delegada, debe especificar explícitamente los servidores de nombres DNS de Azure correctos. En el ejemplo siguiente se muestra cómo recuperar los nombres de servidores de nombres asignados a la zona. También muestra cómo consultar el registro "www" mediante `nslookup`.

    ```azurecli
    az network dns record-set ns list -g myresourcegroup -z contoso.com  --output json 
    ```

    ```json
    [
      {
       .......
       "name": "@",
        "nsRecords": [
          {
            "additionalProperties": {},
            "nsdname": "ns1-03.azure-dns.com."
          },
          {
            "additionalProperties": {},
            "nsdname": "ns2-03.azure-dns.net."
          },
          {
            "additionalProperties": {},
            "nsdname": "ns3-03.azure-dns.org."
          },
          {
            "additionalProperties": {},
            "nsdname": "ns4-03.azure-dns.info."
          }
        ],
        "resourceGroup": "myresourcegroup",
        "ttl": 86400,
        "type": "Microsoft.Network/dnszones/NS"
      }
    ]
    ```

    ```cmd
    nslookup www.contoso.com ns1-03.azure-dns.com

        Server: ns1-01.azure-dns.com
        Address:  40.90.4.1

        Name:www.contoso.com
        Addresses:  134.170.185.46
        134.170.188.221
    ```

### <a name="step-3-update-dns-delegation"></a>Paso 3. Actualización de la delegación de DNS

Una vez que haya comprobado que la zona se importó correctamente, debe actualizar la delegación de DNS para que apunte a los servidores de nombres DNS de Azure. Para más información, consulte el artículo [Delegación de un dominio en DNS de Azure](dns-domain-delegation.md).

## <a name="export-a-dns-zone-file-from-azure-dns"></a>Exportación de un archivo de zona DNS de DNS de Azure

El formato del comando de la CLI de Azure para exportar una zona DNS es:

```azurecli
az network dns zone export -g <resource group> -n <zone name> -f <zone file name>
```

Valores:

* `<resource group>` es el nombre del grupo de recursos para la zona en DNS de Azure.
* `<zone name>` es el nombre de la zona.
* `<zone file name>` es la ruta de acceso y el nombre del archivo de zona que se va a exportar.

Al igual que con la importación de zona, en primer lugar necesita iniciar sesión, elegir su suscripción y configurar la CLI de Azure para que use el modo de Resource Manager.

### <a name="to-export-a-zone-file"></a>Para exportar un archivo de zona

Para exportar la zona DNS de Azure existente **contoso.com** en el grupo de recursos **myresourcegroup** al archivo **contoso.com.txt** (en la carpeta actual), ejecute `azure network dns zone export`. Este comando llama al servicio DNS de Azure para enumerar los conjuntos de registros de la zona y exportar los resultados a un archivo de zona compatible con BIND.

```azurecli
az network dns zone export -g myresourcegroup -n contoso.com -f contoso.com.txt
```

## <a name="next-steps"></a>Pasos siguientes

* En esta guía se explica cómo [administrar conjuntos de registros y registros](dns-getstarted-create-recordset-cli.md) en la zona DNS.

* Aprenda a cómo [delegar el dominio a Azure DNS](dns-domain-delegation.md).
