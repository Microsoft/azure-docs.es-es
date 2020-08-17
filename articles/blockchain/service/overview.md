---
title: Introducción a Azure Blockchain Service
description: Introducción a Azure Blockchain Service
ms.date: 05/22/2020
ms.topic: overview
ms.reviewer: ravastra
ms.openlocfilehash: 2a28baeb6de3c8ca1e8ad89e11d45b492f33588c
ms.sourcegitcommit: 3d79f737ff34708b48dd2ae45100e2516af9ed78
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/23/2020
ms.locfileid: "87076957"
---
# <a name="what-is-azure-blockchain-service"></a>¿Qué es Azure Blockchain Service?

Azure Blockchain Service es un servicio de libro de contabilidad totalmente administrado que permite a los usuarios crear y operar redes de cadena de bloques a escala en Azure. Al proporcionar un control unificado tanto para la administración de la infraestructura como para el gobernanza de la red de cadena de bloques, Azure Blockchain Service proporciona:

* Implementación y operaciones de red sencillas
* Administración de consorcios integrada
* Desarrollo de contratos inteligentes con herramientas de desarrollo conocidas

Azure Blockchain Service está diseñado para admitir varios protocolos de libro de contabilidad. Actualmente, ofrece compatibilidad con el libro de contabilidad [Quorum](https://www.goquorum.com/) de Ethereum mediante el mecanismo de consenso [Istanbul Byzantine Fault Tolerance (IBFT)](https://github.com/jpmorganchase/quorum/wiki/Quorum-Consensus).

Estas funcionalidades no requieren casi ninguna tarea de administración y todas se proporcionan sin ningún costo adicional. Puede centrarse en el desarrollo de aplicaciones y la lógica empresarial en lugar de asignar tiempo y recursos a la administración de máquinas virtuales e infraestructura. Además, puede seguir desarrollando la aplicación con las herramientas de código abierto y en la plataforma que prefiera para ofrecer sus soluciones sin necesidad de adquirir nuevas aptitudes.

## <a name="network-deployment-and-operations"></a>Implementación y operaciones de red

La implementación de Azure Blockchain Service se realiza a través de Azure Portal, la CLI de Azure, o bien a través del código de Visual Studio mediante la extensión de Azure Blockchain. La implementación es sencilla, e incluye el aprovisionamiento de nodos tanto de transacción como de validación, las redes de Azure Virtual Network para el aislamiento de seguridad y el almacenamiento administrado por el servicio.  Además, al implementar a un nuevo miembro de la cadena de bloques, los usuarios también crean un consorcio o se unen a él.  Los consorcios permiten que varias partes de suscripciones de Azure diferentes puedan comunicarse entre sí de forma segura en una cadena de bloques compartida.  Esta implementación simplificada reduce la implementación de redes de cadena de bloques de días a minutos.

### <a name="performance-and-service-tiers"></a>Niveles de servicio y rendimiento

Azure Blockchain Service ofrece dos niveles de servicio: *Básico* y *Estándar*. Cada nivel ofrece un rendimiento diferente y diferentes capacidades para admitir el desarrollo ligero y probar las cargas de trabajo hasta implementaciones de cadena de bloques de producción a escala masiva. Use el nivel *Básico* para desarrollo, pruebas y prueba de concepto. Use el nivel *Estándar* para las implementaciones de nivel de producción. Ambos niveles incluyen al menos un nodo de transacción y un nodo de validación (Básico) o dos nodos de validación (Estándar). 

![Planes de tarifa](./media/overview/pricing-tiers.png)

Además de ofrecer dos nodos de validación, el nivel *Estándar* proporciona dos *núcleos virtuales* para cada nodo de transacción y validación, mientras que el nivel *Básico* ofrece una configuración con un núcleo virtual.  Al ofrecer dos núcleos virtuales para los nodos de transacción y validación, un núcleo virtual puede dedicarse al libro de contabilidad Quorum, mientras que el otro puede usarse para otros servicios relacionados con la infraestructura, lo que garantiza un rendimiento óptimo para las cargas de trabajo de cadenas de bloques de producción. Para obtener más información sobre los detalles de precios, consulte [Precios de Azure Blockchain Service](https://azure.microsoft.com/pricing/details/blockchain-service).

### <a name="security-and-maintenance"></a>Seguridad y mantenimiento

Tras aprovisionar al primer miembro de la cadena de bloques, tiene la capacidad de agregar nodos de transacción adicionales al miembro.  De forma predeterminada, los nodos de transacción se protegen mediante reglas de firewall y requieren una configuración para el acceso.  Además, todos los nodos de transacción cifran los datos en tránsito mediante TLS.  Existen varias opciones para proteger el acceso a los nodos de transacción, entre las que se incluyen reglas de firewall, autenticación básica, claves de acceso e integración de Azure Active Directory. Para obtener más información, consulte cómo [configurar los nodos de transacción](configure-transaction-nodes.md) y cómo [configurar el acceso a Azure Active Directory](configure-aad.md).

Como servicio administrado, Azure Blockchain Service garantiza que los nodos de su miembro de la cadena de bloques se revisan con las actualizaciones más recientes del sistema operativo y la pila de software de libro de contabilidad, configuradas para la alta disponibilidad (solo en el nivel Estándar), lo que elimina gran parte de los DevOps necesarios para los nodos de cadena de bloques de IaaS tradicionales.  Para obtener más información sobre la aplicación de revisiones y actualizaciones, consulte las [versiones de libro de contabilidad de Azure Blockchain Service admitidas](ledger-versions.md).

### <a name="monitoring-and-logging"></a>Supervisión y registro

Además, Azure Blockchain Service proporciona métricas enriquecidas a través del servicio Azure Monitor que proporciona información sobre el uso de la CPU, la memoria y el almacenamiento de los nodos.  Azure Monitor también proporciona información útil sobre la actividad de la red de cadena de bloques, como las transacciones y los bloques extraídos, la profundidad de cola de transacciones y las conexiones activas.  Las métricas se pueden personalizar para proporcionar vistas de la información relevantes para la aplicación de la cadena de bloques.  Además, los umbrales se pueden definir a través de alertas, lo que permite a los usuarios desencadenar acciones como, por ejemplo, enviar un mensaje de texto o un correo, ejecutar una aplicación lógica o una función de Azure o bien enviar a un webhook personalizado.

![Métricas](./media/overview/metrics.png)

A través de los análisis de registros de Azure, los usuarios pueden ver los registros relacionados con el libro de contabilidad Quorum u otra información importante, como, por ejemplo, los intentos de conexión a los nodos de transacción.

## <a name="built-in-consortium-management"></a>Administración de consorcios integrada

Al implementar el primer miembro de la cadena de bloques, se une a un consorcio o crea uno nuevo.  Un consorcio es un grupo lógico que se usa para administrar la gobernanza y la conectividad entre miembros de la cadena de bloques que operan en un proceso de varias partes.  Azure Blockchain Service proporciona controles de gobernanza integrados a través de contratos inteligentes predefinidos, que determinan qué acciones pueden llevar a cabo los miembros del consorcio.  El administrador del consorcio puede personalizar estos controles de gobernanza según sea necesario. Al crear un nuevo consorcio, el miembro de la cadena de bloques es el administrador predeterminado del consorcio y tiene capacidad para invitar a otras partes a unirse al consorcio.  Solo puede unirse a un consorcio si le han invitado con anterioridad.  Al unirse a un consorcio, el miembro de la cadena de bloques está sujeto a los controles de gobernanza que haya impuesto el administrador del consorcio.

![Administración de consorcios](./media/overview/consortium.png)

Las acciones de administración de consorcios, como, por ejemplo, agregar y quitar a miembros de un consorcio, están disponibles a través de PowerShell y una API de REST. Puede administrar un consorcio mediante programación utilizando interfaces habituales en lugar de modificar y enviar contratos inteligentes basados en la solidez. Para obtener más información, consulte el artículo sobre la [administración de consorcios](consortium.md).

## <a name="develop-using-familiar-development-tools"></a>Desarrollar con herramientas de desarrollo conocidas

Según el libro de contabilidad de Quorum Ethereum de código abierto, puede desarrollar aplicaciones para Azure Blockchain Service de la misma manera en que lo hace para las aplicaciones de Ethereum existentes. La extensión Azure Blockchain Development Kit de Visual Studio Code trabaja con partners líderes del sector y permite a los desarrolladores aprovechar herramientas conocidas, como, por ejemplo, Truffle Suite, para generar contratos inteligentes. Con la extensión Azure Blockchain Visual Studio Code, puede crear un consorcio o conectarse a uno para generar e implementar sus contratos inteligentes desde un solo entorno de desarrollo integrado. Para más información, consulte [Azure Blockchain Development Kit en el Marketplace de VS Code](https://aka.ms/vscodebcextension) y la [Guía de usuario de Azure Blockchain Development Kit](https://aka.ms/vscodebcextensionwiki).

## <a name="publish-blockchain-data"></a>Publicación de datos de una cadena de bloques

Blockchain Data Manager para Azure Blockchain Service captura, transforma y entrega datos de transacciones de Azure Blockchain Service a temas de Azure Event Grid, lo que proporcionan una integración confiable y escalable del libro de contabilidad de la cadena de bloques con los servicios de Azure. Blockchain Data Manager se puede usar para integrar aplicaciones y almacenes de datos que estén fuera de la cadena. Para más información, consulte [Blockchain Data Manager para Azure Blockchain Service](data-manager.md).

## <a name="support-and-feedback"></a>Soporte y comentarios

Para leer noticias acerca de Azure Blockchain, visite el [blog de Azure Blockchain](https://azure.microsoft.com/blog/topics/blockchain/), que le permitirá mantenerse al día sobre las ofertas de servicio de Blockchain y le proporcionará información del equipo de ingeniería de Azure Blockchain.

Para proporcionar comentarios sobre el producto o solicitar nuevas características, publique o vote una idea a través del [Foro de comentarios de Azure para Blockchain](https://aka.ms/blockchainuservoice).

### <a name="community-support"></a>Soporte técnico de la comunidad

Interactúe con los ingenieros de Microsoft y con expertos de la comunidad de Azure Blockchain.

* [Página de preguntas y respuestas de Microsoft sobre Azure Blockchain Service](/answers/topics/azure-blockchain-service.html)
* [Comunidad tecnológica de Microsoft](https://techcommunity.microsoft.com/t5/Blockchain/bd-p/AzureBlockchain)
* [Stack Overflow](https://stackoverflow.com/questions/tagged/AzureBlockchainService)

## <a name="next-steps"></a>Pasos siguientes

Para empezar, pruebe un inicio rápido u obtenga más detalles de estos recursos.
* [Crear un miembro de cadena de bloques con Azure Portal](create-member.md) o [Crear un miembro de cadena de bloques mediante la CLI de Azure](create-member-cli.md)
* Visite la [página de precios](https://azure.microsoft.com/pricing/details/blockchain-service) para consultar comparaciones y calculadoras de costos.
* Compilar la primera aplicación mediante [Azure Blockchain Development Kit](https://github.com/Azure-Samples/blockchain-devkit)
* [Guía del usuario](https://github.com/Microsoft/vscode-azure-blockchain-ethereum/wiki) de la extensión VSCode de Azure Blockchain
