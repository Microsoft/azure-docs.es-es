---
title: Introducción a la versión preliminar de Azure Blockchain Workbench
description: Introducción a la versión preliminar de Azure Blockchain Workbench y sus funcionalidades.
ms.date: 05/22/2020
ms.topic: overview
ms.reviewer: brendal
ms.openlocfilehash: fbd6be3907dbd10b003d065dfb14031a0e378478
ms.sourcegitcommit: 3d79f737ff34708b48dd2ae45100e2516af9ed78
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/23/2020
ms.locfileid: "87003161"
---
# <a name="what-is-azure-blockchain-workbench"></a>¿Qué es Azure Blockchain Workbench?

La versión preliminar de Azure Blockchain Workbench es una colección de servicios y funcionalidades de Azure diseñada para ayudarle a crear e implementar aplicaciones de cadena de bloques para compartir datos y procesos empresariales con otras organizaciones. Azure Blockchain Workbench proporciona el scaffolding de infraestructura para compilar aplicaciones de cadena de bloques, lo que permite a los desarrolladores centrarse en crear lógica de negocios y contratos inteligentes. También facilita la creación de aplicaciones de cadena de bloques al integrar varios servicios y funcionalidades de Azure para ayudar a automatizar tareas comunes de desarrollo.

[!INCLUDE [Preview note](./includes/preview.md)]

## <a name="create-blockchain-applications"></a>Creación de aplicaciones de cadena de bloques

Con Blockchain Workbench, puede definir aplicaciones de cadena de bloques con la configuración y con la escritura de código de contrato inteligente. También puede iniciar rápidamente el desarrollo de aplicaciones de cadena de bloques y centrarse en definir el contrato y escribir lógica de negocios en lugar de compilar scaffolding y configurar servicios auxiliares.

## <a name="manage-applications-and-users"></a>Administrar aplicaciones y usuarios

Azure Blockchain Workbench proporciona una aplicación web y API de REST para administrar usuarios y aplicaciones de cadena de bloques. Los administradores de Blockchain Workbench pueden administrar el acceso a la aplicación y asignar usuarios a roles de aplicación. Los usuarios de Azure AD se asignan automáticamente a los miembros de la aplicación.

## <a name="integrate-blockchain-with-applications"></a>Integrar cadenas de bloques con aplicaciones

Puede usar las API de REST de Blockchain Workbench y API basadas en mensajes para realizar la integración con sistemas existentes. Las API proporcionan una interfaz para permitir el reemplazo o uso de varias ofertas de tecnologías de libro de contabilidad distribuida, almacenamiento y base de datos.

Blockchain Workbench puede transformar mensajes enviados a la API basada en mensajes para compilar transacciones en un formato compatible con la API nativa de esa cadena de bloques.  Workbench puede firmar y enrutar las transacciones a la cadena de bloques adecuada. 

Workbench entrega eventos automáticamente a Service Bus y Event Grid para enviar mensajes a consumidores de bajada. Los desarrolladores pueden integrar con cualquiera de estos sistemas de mensajería para dirigir transacciones y observar los resultados.

## <a name="deploy-a-blockchain-network"></a>Implementar una red de cadena de bloques

Azure Blockchain Workbench simplifica la configuración de red de cadena de bloques del consorcio como una solución preconfigurada con una plantilla de solución de Azure Resource Manager. La plantilla proporciona implementación simplificada que implementa todos los componentes necesarios para ejecutar un consorcio. En la actualidad, Blockchain Workbench admite Ethereum.

## <a name="use-active-directory"></a>Usar Active Directory

Con los protocolos existentes de cadena de bloques, las identidades de cadena de bloques se representan como una dirección en la red. Azure Blockchain Workbench abstrae la identidad de cadena de bloques mediante la asociación con una identidad de Active Directory, lo que simplifica la compilación de aplicaciones empresariales con identidades de Active Directory.

## <a name="synchronize-on-chain-data-with-off-chain-storage"></a>Sincronizar datos en cadena con almacenamiento fuera de la cadena

Azure Blockchain Workbench facilita el análisis de datos y eventos de cadena de bloques mediante la sincronización automática de datos de la cadena de bloques con el almacenamiento fuera de la cadena. En lugar de extraer datos directamente desde la cadena de bloques, puede consultar sistemas de bases de datos fuera de la cadena, como SQL Server. Los usuarios finales que realizan tareas de análisis de datos no necesitan conocimientos sobre cadenas de bloques.

## <a name="support-and-feedback"></a>Soporte y comentarios

Para leer noticias acerca de Azure Blockchain, visite el [blog de Azure Blockchain](https://azure.microsoft.com/blog/topics/blockchain/), que le permitirá mantenerse al día sobre las ofertas de servicio de Blockchain y le proporcionará información del equipo de ingeniería de Azure Blockchain.

Para proporcionar comentarios sobre el producto o solicitar nuevas características, publique o vote una idea a través del [Foro de comentarios de Azure para Blockchain](https://aka.ms/blockchainuservoice).

### <a name="community-support"></a>Soporte técnico de la comunidad

Interactúe con los ingenieros de Microsoft y con expertos de la comunidad de Azure Blockchain.

* [Página de preguntas y respuestas de Microsoft sobre Azure Blockchain Workbench](/answers/topics/azure-blockchain-workbench.html)
* [Comunidad tecnológica de Microsoft](https://techcommunity.microsoft.com/t5/Blockchain/bd-p/AzureBlockchain)
* [Stack Overflow](https://stackoverflow.com/questions/tagged/azure-blockchain-workbench)

## <a name="next-steps"></a>Pasos siguientes

> [!div class="nextstepaction"]
> [Arquitectura de Azure Blockchain Workbench](architecture.md)
