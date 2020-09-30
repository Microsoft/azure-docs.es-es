---
title: Estrategias de acceso a datos
description: Azure Data Factory admite ahora intervalos de direcciones IP estáticas.
services: data-factory
ms.author: abnarain
author: nabhishek
ms.service: data-factory
ms.workload: data-services
ms.topic: conceptual
ms.date: 05/28/2020
ms.openlocfilehash: 76181f089511a6645a51707f9a8537c1589d82bf
ms.sourcegitcommit: de2750163a601aae0c28506ba32be067e0068c0c
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/04/2020
ms.locfileid: "89484959"
---
# <a name="data-access-strategies"></a>Estrategias de acceso a datos

[!INCLUDE[appliesto-adf-asa-md](includes/appliesto-adf-asa-md.md)]

Un objetivo de seguridad vital de una organización es proteger sus almacenes de datos del acceso aleatorio a través de Internet, puede ser un almacén de datos local o de nube o SaaS. 

Normalmente, un almacén de datos en la nube controla el acceso mediante los mecanismos siguientes:
* Vínculo privado de una red virtual a los orígenes de datos habilitados para el punto de conexión privado
* Reglas de firewall que limitan la conectividad mediante una dirección IP
* Mecanismos de autenticación que solicitan a los usuarios sus identidades
* Mecanismos de autorización que limitan los usuarios a acciones y datos específicos

> [!TIP]
> Con la [introducción del intervalo de direcciones IP estáticas](https://docs.microsoft.com/azure/data-factory/azure-integration-runtime-ip-addresses), ahora puede permitir la lista de intervalos IP para la región del entorno de ejecución de integración de Azure determinada para asegurarse de que no tiene que permitir todas las direcciones IP de Azure en los almacenes de datos en la nube. De este modo, puede restringir las direcciones IP a las que se les permite el acceso a los almacenes de datos.

> [!NOTE] 
> Los intervalos de direcciones IP están bloqueados para Azure Integration Runtime y actualmente solo se usan para movimiento de datos, canalización y actividades externas. Azure Integration Runtime y los flujos de datos que habilitan la red virtual administrada ahora no usan estos intervalos IP. 

Esto debería funcionar en muchos escenarios, y sabemos que es deseable una dirección IP estática única por cada tiempo de ejecución de integración, pero actualmente esto no sería posible mediante el uso de Azure Integration Runtime, que está sin servidor. Si es necesario, siempre puede configurar un entorno de ejecución de integración autohospedado y usar su dirección IP estática. 

## <a name="data-access-strategies-through-azure-data-factory"></a>Estrategias de acceso a datos a través de Azure Data Factory

* **[Private Link](https://docs.microsoft.com/azure/private-link/private-link-overview)** : puede crear una instancia de Azure Integration Runtime en la red virtual administrada de Azure Data Factory y aprovechará los puntos de conexión privados para conectarse de forma segura a los almacenes de datos compatibles. El tráfico entre la red virtual administrada y los orígenes de datos viaja por la red troncal de Microsoft y no se exponen a la red pública.
* **[Servicio de confianza](https://docs.microsoft.com/azure/storage/common/storage-network-security#exceptions)** : Azure Storage (BLOB, ADLS Gen2) admite la configuración del firewall que permite seleccionar servicios de la plataforma de Azure de confianza para acceder a la cuenta de almacenamiento de forma segura. Los servicios de confianza aplican la autenticación de identidad administrada, lo que garantiza que ninguna otra factoría de datos pueda conectarse a este almacenamiento, a menos que se incluya en la lista de permitidos mediante su identidad administrada. Puede encontrar más información en **[este blog](https://techcommunity.microsoft.com/t5/azure-data-factory/data-factory-is-now-a-trusted-service-in-azure-storage-and-azure/ba-p/964993)** . Por lo tanto, es muy seguro y recomendado. 
* **IP estática única**: deberá configurar una instancia de entorno de ejecución de integración autohospedado para obtener una dirección IP estática para los conectores de Data Factory. Este mecanismo garantiza que puede bloquear el acceso desde todas las demás direcciones IP. 
* **[Intervalo de direcciones IP estáticas](https://docs.microsoft.com/azure/data-factory/azure-integration-runtime-ip-addresses)** : puede usar las direcciones IP de Azure Integration Runtime para permitir que se muestren en el almacenamiento (por ejemplo, S3, Salesforce, etc.). Ciertamente, restringe las direcciones IP que se pueden conectar a los almacenes de datos, pero también se basa en las reglas de autenticación y autorización.
* **[Etiqueta de servicio](https://docs.microsoft.com/azure/virtual-network/service-tags-overview)** : una etiqueta de servicio representa un grupo de prefijos de direcciones IP de un servicio de Azure determinado (como Azure Data Factory). Microsoft administra los prefijos de direcciones que la etiqueta de servicio incluye y actualiza automáticamente dicha etiqueta a medida que las direcciones cambian, lo que minimiza la complejidad de las actualizaciones frecuentes en las reglas de seguridad de red. Resulta útil cuando se incluye en la lista de permitidos el acceso a los datos de los almacenes de datos hospedados en IaaS de Virtual Network.
* **Permitir servicios de Azure**: algunos servicios admiten permitir que todos los servicios de Azure se conecten a él en caso de que elija esta opción. 

Para obtener más información sobre los mecanismos de seguridad de red admitidos en los almacenes de datos de Azure Integration Runtime y Integration Runtime autohospedado, consulte las dos tablas siguientes.  
* **Azure Integration Runtime**

    | Almacenes de datos                  | Mecanismo de seguridad de red compatible con almacenes de datos | Private Link     | Servicio de confianza     | Rango de direcciones IP estáticas | Etiquetas de servicio | Permitir servicios de Azure |
    |------------------------------|-------------------------------------------------------------|---------------------|-----------------|--------------|----------------------|-----------------|
    | Almacenes de datos PaaS de Azure       | Azure Cosmos DB                                     | Sí              | -                   | Sí             | -            | Sí                  |
    |                              | Explorador de datos de Azure                                 | -                | -                   | Sí*            | Sí*         | -                    |
    |                              | Azure Data Lake Gen1                                | -                | -                   | Sí             | -            | Sí                  |
    |                              | Azure Database for MariaDB, MySQL y PostgreSQL       | -                | -                   | Sí             | -            | Sí                  |
    |                              | Azure File Storage                                  | Sí              | -                   | Sí             | -            | .                    |
    |                              | Azure Storage (Blob, ADLS Gen2)                     | Sí              | Sí (solo autenticación MSI) | Sí             | -            | .                    |
    |                              | Azure SQL DB, Azure Synapse Analytics), SQL Ml  | Sí (solo Azure SQL DB/DW)        | -                   | Sí             | -            | Sí                  |
    |                              | Azure Key Vault (para capturar secretos/cadena de conexión) | sí      | Sí                 | Sí             | -            | -                    |
    | Otros almacenes de datos de PaaS/SaaS | AWS S3, SalesForce, Google Cloud Storage, etc.    | -                | -                   | Sí             | -            | -                    |
    | Azure laaS                   | SQL Server, Oracle, etc.                          | -                | -                   | Sí             | Sí          | -                    |
    | LaaS local              | SQL Server, Oracle, etc.                          | -                | -                   | Sí             | -            | -                    |
    
    **Solo se aplica cuando Azure Data Explorer es la red virtual inyectada y se puede aplicar el intervalo IP en NSG/Firewall.* 

* **Entorno de ejecución de integración autohospedado (en Vnet/local)**
    
    | Almacenes de datos                  | Mecanismo de seguridad de red compatible con almacenes de datos         | IP estática | Servicios de confianza  |
    |--------------------------------|---------------------------------------------------------------|-----------|---------------------|
    | Almacenes de datos PaaS de Azure       | Azure Cosmos DB                                               | Sí       | -                   |
    |                                | Explorador de datos de Azure                                           | -         | -                   |
    |                                | Azure Data Lake Gen1                                          | Sí       | -                   |
    |                                | Azure Database for MariaDB, MySQL y PostgreSQL               | Sí       | -                   |
    |                                | Azure File Storage                                            | Sí       | -                   |
    |                                | Azure Storage (blog, ADLS Gen2)                             | Sí       | Sí (solo autenticación MSI) |
    |                                | Azure SQL DB, Azure Synapse Analytics), SQL Ml          | Sí       | -                   |
    |                                | Azure Key Vault (para capturar secretos/cadena de conexión) | Sí       | Sí                 |
    | Otros almacenes de datos de PaaS/SaaS | AWS S3, SalesForce, Google Cloud Storage, etc.              | Sí       | -                   |
    | Azure laaS                     | SQL Server, Oracle, etc.                                  | Sí       | -                   |
    | LaaS local              | SQL Server, Oracle, etc.                                  | Sí       | -                   |    

## <a name="next-steps"></a>Pasos siguientes

Para más información, consulte los siguientes artículos:
* [Almacenes de datos compatibles](https://docs.microsoft.com/azure/data-factory/copy-activity-overview#supported-data-stores-and-formats)
* [Servicios de confianza de Azure Key Vault](https://docs.microsoft.com/azure/key-vault/key-vault-overview-vnet-service-endpoints#trusted-services)
* [Servicios de confianza de Microsoft Azure Storage](https://docs.microsoft.com/azure/storage/common/storage-network-security#trusted-microsoft-services)
* [Identidad administrada de Data Factory](https://docs.microsoft.com/azure/data-factory/data-factory-service-identity)
