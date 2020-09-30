---
title: Azure Private Link para Azure Data Factory
description: Más información sobre el funcionamiento de Azure Private Link en Azure Data Factory.
services: data-factory
ms.author: abnarain
author: nabhishek
manager: shwang
ms.reviewer: douglasl
ms.service: data-factory
ms.workload: data-services
ms.topic: conceptual
ms.custom: seo-lt-2019
ms.date: 09/01/2020
ms.openlocfilehash: 1d560bd4a6b826e2bf0b3e23dde563cc39863537
ms.sourcegitcommit: 5ed504a9ddfbd69d4f2d256ec431e634eb38813e
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/02/2020
ms.locfileid: "89322848"
---
# <a name="azure-private-link-for-azure-data-factory"></a>Azure Private Link para Azure Data Factory

[!INCLUDE[appliesto-adf-asa-md](includes/appliesto-adf-xxx-md.md)]

Private Link permite conectarse a varios servicios PaaS en Azure mediante un punto de conexión privado. Para obtener una lista de los servicios PaaS que admiten la funcionalidad Private Link, vaya a la [página de la documentación de Private Link](https://docs.microsoft.com/azure/private-link/). Un punto de conexión privado es una dirección IP privada dentro de una red virtual y una subred específicas.

## <a name="secure-communication-between-customer-network-and-azure-data-factory-service"></a>Comunicación segura entre la red del cliente y el servicio Azure Data Factory
Para proteger los recursos de Azure de los ataques en la red pública o permitirles comunicarse de forma segura entre sí, puede configurar una instancia de Azure Virtual Network como una representación lógica de la red en la nube. Puede también conectar una red local a la red virtual mediante la configuración de una conexión VPN de IPSec (de sitio a sitio) o ExpressRoute (Emparejamiento privado) Un entorno de ejecución de integración autohospedado se puede instalar en una máquina local o en una máquina virtual para ejecutar actividades de copia entre un almacén de datos en la nube y un almacén de datos de una red privada, o distribuir actividades de transformación a los recursos de proceso de una red local o una red virtual de Azure. 

Se necesitan varios canales de comunicación entre Data Factory y la red virtual del cliente.


| **Dominio** | **Puerto** | **Descripción** |
| ---------- | -------- | --------------- |
| `pe-adf.azure.com` | 443 | Plano de control. Necesario para la creación y supervisión en Data Factory. |
| `*.{region}.datafactory.azure.net` | 443 | El entorno de ejecución de integración autohospedado lo necesita para conectarse al servicio Data Factory. |
| `*.servicebus.windows.net` | 443 | Lo necesita el entorno de ejecución de integración autohospedado para la creación interactiva. |
| `download.microsoft.com` | 443 | Lo necesita el entorno de ejecución de integración autohospedado para descargar las actualizaciones. |


Con la ayuda de Azure Private Link para Azure Data Factory, puede crear un punto de conexión privado (PE) en la red virtual y habilitar la conexión privada a la instancia específica de Azure Data Factory. Las comunicaciones con el servicio Azure Data Factory pasan a través de Azure Private Link, el cual proporciona conectividad privada segura. Y no es necesario configurar el dominio y el puerto anteriores en la red virtual ni el firewall corporativo que proporciona una manera más segura de proteger los recursos.  

![Arquitectura de Private Link para Azure Data Factory](./media/data-factory-private-link/private-link-architecture.png)

Estas son las ventajas de habilitar el servicio Private Link para cada uno de los canales de comunicación descritos anteriormente:
- Admitido: puede realizar la creación y supervisión de Azure Data Factory en la red virtual incluso si bloquea todas las comunicaciones salientes.
- Admitido: las comunicaciones de comandos entre el entorno de ejecución de integración autohospedado y el servicio Azure Data Factory se pueden realizar de forma segura en un entorno de red privado. El tráfico entre el entorno de ejecución de integración autohospedado y el servicio Azure Data Factory pasa a través Private Link. 
- No admitido actualmente: la creación interactiva mediante el uso del entorno de ejecución de integración autohospedado pasa a través de Private Link como, por ejemplo, en el caso de las pruebas de conexiones, el examen de la lista de carpetas y de tablas, la obtención de esquemas y la vista previa de los datos.
- No admitido actualmente: la nueva versión del entorno de ejecución de integración autohospedado se puede descargar automáticamente desde el centro de descargas si habilita la actualización automática.

> [!NOTE]
> Para las funcionalidades que no se admiten actualmente deberá configurar el dominio y el puerto anteriores en la red virtual o el firewall corporativo. 

> [!WARNING]
> Cuando cree un servicio vinculado asegúrese de que la credencial se almacena en Azure Key Vault. De lo contrario, no funcionará cuando habilite el servicio Private Link en Azure Data Factory.

## <a name="how-to-set-up-private-link-for-azure-data-factory"></a>Configuración de Private Link para Azure Data Factory
Los puntos de conexión privados se pueden crear mediante Azure Portal, PowerShell o la CLI de Azure:

[Portal](https://docs.microsoft.com/azure/private-link/create-private-endpoint-portal)


También puede ir a Azure Data Factory en Azure Portal y crear un punto de conexión privado (PE):

![Creación de un punto de conexión privado](./media/data-factory-private-link/create-private-endpoint.png)


Si desea bloquear el acceso público a esta instancia de Azure Data Factory y solo permitir el acceso a través de Private Link, puede deshabilitar el acceso a la red de Azure Data Factory en Azure Portal:

![Creación de un punto de conexión privado](./media/data-factory-private-link/disable-network-access.png)

> [!NOTE]
> Deshabilitar el acceso a la red pública solo es aplicable al entorno de ejecución de integración autohospedado, no a Azure Integration Runtime ni a Azure-SSIS Integration Runtime.

> [!NOTE]
> Los usuarios pueden seguir accediendo al portal de Azure Data Factory a través de la red pública después de deshabilitar el acceso a la red pública.

## <a name="next-steps"></a>Pasos siguientes

- [Creación de una factoría de datos con la interfaz de usuario de Azure Data Factory](quickstart-create-data-factory-portal.md)

- [Introducción al servicio Azure Data Factory](introduction.md)

- [Creación visual de Azure Data Factory](author-visually.md)

