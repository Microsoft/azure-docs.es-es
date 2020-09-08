---
title: Red virtual administrada y puntos de conexión privados administrados
description: Obtenga información sobre la red virtual administrada y los puntos de conexión privados administrados en Azure Data Factory.
services: data-factory
ms.author: abnarain
author: nabhishek
manager: shwang
ms.reviewer: douglasl
ms.service: data-factory
ms.workload: data-services
ms.topic: conceptual
ms.custom: seo-lt-2019
ms.date: 07/15/2020
ms.openlocfilehash: 14a3a76ef4fefb7a33b272b846e1f1cb66644c3e
ms.sourcegitcommit: 152c522bb5ad64e5c020b466b239cdac040b9377
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/14/2020
ms.locfileid: "88225689"
---
# <a name="azure-data-factory-managed-virtual-network-preview"></a>Red virtual administrada de Azure Data Factory (versión preliminar)

[!INCLUDE[appliesto-adf-asa-md](includes/appliesto-adf-asa-md.md)]

En este artículo se explica la red virtual administrada y los puntos de conexión privados administrados en Azure Data Factory.


## <a name="managed-virtual-network"></a>Red virtual administrada

Al crear una instancia de Azure Integration Runtime (IR) en la red virtual (VNET) administrada de Azure Data Factory, el entorno de ejecución de integración se aprovisionará con la red virtual administrada y aprovechará los puntos de conexión privados para conectarse de forma segura a almacenes de datos compatibles. 

La creación de una instancia de Azure IR dentro de una red virtual administrada garantiza que el proceso de integración de datos esté aislado y protegido. 

Ventajas del uso de una red virtual administrada:

- Con una red virtual administrada puede dejar que Azure Data Factory se ocupe de la pesada tarea de administrar la red virtual. No es necesario crear una subred para Azure Integration Runtime que podría usar muchas direcciones IP privadas de la red virtual y requeriría una planeación previa de la infraestructura de la red. 
- No se necesita un conocimiento profundo de las redes de Azure para realizar integraciones de datos de forma segura. En su lugar, empezar a utilizar ETL seguro es mucho más sencillo para los ingenieros de datos. 
- La red virtual administrada, junto con los puntos de conexión privados administrados, sirven de protección contra la filtración de datos. 

> [!IMPORTANT]
>Actualmente, la red virtual administrada solo se admite en la misma región que la región de Azure Data Factory.
 

![Arquitectura de la red virtual de Azure Data Factory](./media/managed-vnet/managed-vnet-architecture-diagram.png)

## <a name="managed-private-endpoints"></a>Puntos de conexión privados administrados

Los puntos de conexión privados administrados se crean en la red virtual administrada de Azure Data Factory y establecen un vínculo privado a recursos de Azure. Azure Data Factory administra estos puntos de conexión privados en su nombre. 

![Nuevo punto de conexión privado administrado](./media/tutorial-copy-data-portal-private/new-managed-private-endpoint.png)

Azure Data Factory admite vínculos privados. El vínculo privado le permite acceder a servicios de Azure (PaaS) [como Azure Storage, Azure Cosmos DB o Azure Synapse Analytics (anteriormente Azure SQL Data Warehouse)].

Cuando se usa un vínculo privado, el tráfico entre los almacenes de datos y la red virtual administrada atraviesa completamente la red troncal de Microsoft. Private Link protege frente a los riesgos de la filtración de datos. El vínculo privado a un recurso se establece mediante la creación de un punto de conexión privado.

El punto de conexión privado usa una dirección IP privada en la red virtual administrada para incorporar el servicio de manera eficaz a la red virtual. Los puntos de conexión privados se asignan a un recurso específico de Azure, no a todo el servicio. Los clientes pueden limitar la conectividad a un recurso específico aprobado por su organización. Más información sobre [vínculos privados y puntos de conexión privados](https://docs.microsoft.com/azure/private-link/).

> [!NOTE]
> Se recomienda crear puntos de conexión privados administrados para conectarse a todos los orígenes de datos de Azure. 
 
> [!WARNING]
> Si un almacén de datos de PaaS (Blob, ADLS Gen2, SQL DW) tiene un punto de conexión privado ya creado en él, incluso si permite el acceso desde todas las redes, ADF solo podría acceder a dicho almacén mediante un punto de conexión privado administrado. Asegúrese de crear un punto de conexión privado en estos escenarios. 

Una conexión de punto de conexión privado se crea en un estado "pendiente" cuando se crea un punto de conexión privado administrado en Azure Data Factory. Se inicia un flujo de trabajo de aprobación. El propietario del recurso de vínculo privado es responsable de aprobar o rechazar la conexión.

![Administración de un punto de conexión privado](./media/tutorial-copy-data-portal-private/manage-private-endpoint.png)

Si el propietario aprueba la conexión, se establece el vínculo privado. De lo contrario, no se establece. En cualquier caso, el punto de conexión privado administrado se actualizará con el estado de la conexión.

![Punto de conexión privado administrado aprobado](./media/tutorial-copy-data-portal-private/approve-private-endpoint.png)

Solo un punto de conexión privado administrado en un estado aprobado puede enviar tráfico a un recurso de vínculo privado determinado.

## <a name="limitations-and-known-issues"></a>Limitaciones y problemas conocidos
### <a name="supported-data-sources"></a>Orígenes de datos compatibles
Los orígenes de datos siguientes pueden establecer una conexión a través de un vínculo privado desde la red virtual administrada de ADF.
- Azure Blob Storage
- Azure Table Storage
- Azure Files
- Azure Data Lake Gen2
- Azure SQL Database (sin incluir Azure SQL Managed Instance)
- Azure Synapse Analytics (anteriormente Azure SQL Data Warehouse)
- Azure CosmosDB SQL
- Azure Key Vault
- Servicio Azure Private Link

### <a name="outbound-communications-through-public-endpoint-from-adf-managed-virtual-network"></a>Comunicaciones salientes a través del punto de conexión público desde la red virtual administrada de ADF
- Solo se abre el puerto 443 para las comunicaciones salientes.
- Azure Storage y Azure Data Lake Gen2 no pueden establecer una conexión a través del punto de conexión público desde la red virtual administrada de ADF.

## <a name="next-steps"></a>Pasos siguientes

- Tutorial: [Creación de una canalización de copia mediante la red virtual administrada y los puntos de conexión privados](tutorial-copy-data-portal-private.md) 
- Tutorial: [Creación de una canalización de flujo de datos de asignación mediante la red virtual administrada y los puntos de conexión privados](tutorial-data-flow-private.md)
