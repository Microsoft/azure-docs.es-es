---
title: Arquitectura de Azure Blockchain Workbench
description: Introducción a la arquitectura de Azure Blockchain Workbench (versión preliminar) y sus componentes.
ms.date: 09/05/2019
ms.topic: conceptual
ms.reviewer: brendal
ms.openlocfilehash: 1fff70ef2eeb1dc27d33769fd48fe5923f56717b
ms.sourcegitcommit: 3d79f737ff34708b48dd2ae45100e2516af9ed78
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/23/2020
ms.locfileid: "87049172"
---
# <a name="azure-blockchain-workbench-architecture"></a>Arquitectura de Azure Blockchain Workbench

Azure Blockchain Workbench (versión preliminar) simplifica el desarrollo de aplicaciones de cadena de bloques al proporcionar una solución con varios componentes de Azure. Blockchain Workbench se puede implementar mediante una plantilla de solución de Azure Marketplace. La plantilla permite seleccionar los módulos y componentes para implementar, por ejemplo, la pila de cadena de bloques, el tipo de aplicación cliente o la compatibilidad con la integración de IoT. Una vez implementado, Blockchain Workbench proporciona acceso a una aplicación web, una aplicación iOS y una aplicación Android.

![Arquitectura de Blockchain Workbench](./media/architecture/architecture.png)

## <a name="identity-and-authentication"></a>Identidad y autenticación

Mediante el uso de Blockchain Workbench, un consorcio puede federar sus identidades de empresa con Azure Active Directory (Azure AD). Workbench genera nuevas cuentas de usuario para las identidades en la cadena con las identidades de empresa almacenadas en Azure AD. La asignación de identidades facilita el inicio de sesión autenticado a las API de cliente y las aplicaciones y utiliza las directivas de autenticación de las organizaciones. Workbench también proporciona la posibilidad de asociar las identidades de empresa a roles específicos dentro de un contrato inteligente determinado. Además, Workbench también proporciona un mecanismo para identificar las acciones que pueden llevar a cabo esos roles y en qué momento.

Después de implementar Blockchain Workbench, los usuarios interactúan con Blockchain Workbench a través de las aplicaciones cliente, la API REST de cliente o la API de mensajería. En todos los casos, las interacciones deben estar autenticadas, bien a través de Azure Active Directory (Azure AD) o mediante credenciales específicas del dispositivo.

Los usuarios federan sus identidades con un consorcio de Azure AD mediante el envío de una invitación por correo electrónico a los participantes a su dirección de correo electrónico. Al iniciar sesión, estos usuarios se autentican con el nombre, la contraseña y las directivas. Por ejemplo, la autenticación en dos factores de su organización.

Azure AD se usa para administrar todos los usuarios que tienen acceso a Blockchain Workbench. Cada dispositivo que se conecta a un contrato inteligente también está asociado con Azure AD.

Azure AD también se usa para asignar usuarios a un grupo de administradores especial. A los usuarios asociados con el grupo de administradores se les concede acceso a derechos y acciones dentro de Blockchain Workbench, como la implementación de contratos o dar permisos a un usuario para tener acceso a un contrato. Los usuarios ajenos a este grupo no tienen acceso a las acciones de administrador.

## <a name="client-applications"></a>Aplicaciones cliente

Workbench proporciona aplicaciones cliente generadas automáticamente para web y móviles (iOS, Android) que se pueden usar para validar, probar y visualizar aplicaciones de cadena de bloques. La interfaz de la aplicación se genera dinámicamente en función de los metadatos del contrato inteligente y se ajusta a cualquier caso de uso. Las aplicaciones cliente proporcionan un front-end de cara al usuario para todas las aplicaciones de cadena de bloques generadas por Blockchain Workbench. Las aplicaciones cliente autentican a los usuarios a través de Azure Active Directory (Azure AD) y, a continuación, presentan una experiencia de usuario personalizada para el contexto de negocio del contrato inteligente. La experiencia de usuario permite la creación de nuevas instancias de contratos inteligentes por las personas autorizadas y, a continuación, presenta la posibilidad de ejecutar determinados tipos de transacciones en los puntos adecuados del proceso empresarial que representa el contrato inteligente.

En la aplicación web, los usuarios autorizados pueden tener acceso a la consola de administrador. La consola está disponible para los usuarios del grupo de administradores de Azure AD y proporciona acceso a las funcionalidades siguientes:

* Implementar contratos inteligentes proporcionados por Microsoft para escenarios populares. Por ejemplo, un escenario de transferencia de recursos.
* Cargar e implementar sus propios contratos inteligentes.
* Asignar a un usuario acceso al contrato inteligente en el contexto de un rol específico.

Para obtener más información, consulte las[Aplicaciones cliente de ejemplo de Azure Blockchain Workbench en GitHub](https://github.com/Azure-Samples/blockchain-devkit/tree/master/connect/mobile).

## <a name="gateway-service-api"></a>API del servicio de puerta de enlace

Blockchain Workbench incluye una API REST del servicio de puerta de enlace. Al escribir en una cadena de bloques, la API genera y envía mensajes a un agente de eventos. Cuando se solicitan datos mediante la API, las consultas se envían a la base de datos fuera de la cadena. La base de datos contiene una réplica de los datos de la cadena y metadatos que proporcionan contexto e información de configuración para los contratos inteligentes admitidos. Las consultas devuelven los datos solicitados desde la réplica fuera de la cadena en un formato informado por los metadatos del contrato.

Los desarrolladores pueden acceder a la API del servicio de puerta de enlace para generar o integrar soluciones de cadena de bloques sin tener que depender de las aplicaciones cliente de Blockchain Workbench.

> [!NOTE]
> Para habilitar el acceso autenticado a la API, se registran dos aplicaciones cliente en Azure Active Directory. Azure Active Directory requiere registros de aplicación distintos para cada tipo de aplicación (nativo y web).

## <a name="message-broker-for-incoming-messages"></a>Agente de mensajes para mensajes entrantes

Los desarrolladores que desean enviar mensajes directamente a Blockchain Workbench pueden enviar mensajes directamente a Service Bus. Por ejemplo, la API de mensajes podría usarse para la integración de sistema a sistema o de dispositivos de IoT.

## <a name="message-broker-for-downstream-consumers"></a>Agente de mensajes para consumidores de nivel final

Durante el ciclo de vida de la aplicación se producen eventos. Los eventos se pueden desencadenar mediante la API de puerta de enlace o en el libro de contabilidad. Las notificaciones de eventos pueden iniciar código de nivel final según el evento.

Blockchain Workbench implementa automáticamente dos tipos de consumidores de eventos. Un consumidor se desencadena por eventos de la cadena de bloques para alimentar el almacén SQL fuera de la cadena. El otro consumidor es para capturar los metadatos de los eventos generados por la API relacionados con la carga y el almacenamiento de documentos.

## <a name="message-consumers"></a>Consumidores de mensajes

 Los consumidores de mensajes toman mensajes de Service Bus. El modelo de eventos subyacente para los consumidores de mensajes permite extensiones de sistemas y servicios adicionales. Por ejemplo, podría agregar compatibilidad para alimentar CosmosDB o evaluar los mensajes mediante Azure Streaming Analytics. Las secciones siguientes describen los consumidores de mensajes incluidos en Blockchain Workbench.

### <a name="distributed-ledger-consumer"></a>Consumidor de libro de contabilidad distribuida

Los mensajes de la tecnología de libro de contabilidad distribuida (DLT) contienen los metadatos de las transacciones que se escriben en la cadena de bloques. El consumidor recupera los mensajes e inserta datos en un creador, firmante y enrutador de la transacción.

### <a name="database-consumer"></a>Consumidor de base de datos

El consumidor de base de datos toma mensajes de Service Bus e inserta datos en una base de datos asociada; por ejemplo, una base de datos de Azure SQL Database.

### <a name="storage-consumer"></a>Consumidor de almacenamiento

El consumidor de almacenamiento toma mensajes de Service Bus e inserta datos en un almacenamiento conectado. Por ejemplo, almacenar documentos con hash en Azure Storage.

## <a name="transaction-builder-and-signer"></a>Creador y firmante de la transacción

Si un mensaje del agente de mensajes entrantes debe ser escrito en la cadena de bloques, lo procesará el consumidor de DLT. El consumidor de DLT es un servicio que recupera el mensaje que contiene los metadatos para la ejecución de una transacción deseada y, a continuación, envía la información al *creador y firmante de la transacción*. El *creador y firmante de la transacción* ensambla una transacción de la cadena de bloques en función de los datos y el destino de la cadena de bloques deseado. Una vez ensamblada, se firma la transacción. Las claves privadas se almacenan en Azure Key Vault.

 Blockchain Workbench recupera la clave privada correspondiente del almacén de claves y firma la transacción fuera del almacén de claves. Una vez firmada, la transacción se envía a los libros de contabilidad y enrutadores de la transacción.

## <a name="transaction-routers-and-ledgers"></a>Libros de contabilidad y enrutadores de transacciones

Los libros de contabilidad y enrutadores de transacciones toman transacciones firmadas y las enrutan a la cadena de bloques adecuada. Actualmente, Blockchain Workbench es compatible con Ethereum como su cadena de bloques de destino.

## <a name="dlt-watcher"></a>Inspector DLT

Un inspector de tecnología de libro de contabilidad distribuida (DLT) supervisa los eventos que ocurren en las cadenas de bloques conectadas a Blockchain Workbench.
Los eventos reflejan información pertinente para las personas y los sistemas. Por ejemplo, la creación de nuevas instancias de contrato, la ejecución de transacciones y los cambios de estado. Los eventos se capturan y se envían al agente de mensajes salientes, por lo que pueden ser utilizados por los consumidores de nivel final.

Por ejemplo, el consumidor de SQL supervisa los eventos, los consume y alimenta la base de datos con los valores incluidos. La copia permite la creación de una réplica de los datos de la cadena en un almacén fuera de la cadena.

## <a name="azure-sql-database"></a>Azure SQL Database

La base de datos asociada a Blockchain Workbench almacena definiciones de contratos, metadatos de configuración y una réplica accesible mediante SQL de los datos almacenados en la cadena de bloques. Estos datos se pueden consultar, visualizar o analizar fácilmente accediendo directamente a la base de datos. Los desarrolladores y otros usuarios pueden usar la base de datos para informes, análisis u otras integraciones centradas en datos. Por ejemplo, los usuarios pueden visualizar los datos de las transacciones con Power BI.

Este almacenamiento fuera de la cadena proporciona a las organizaciones empresariales la posibilidad de consultar los datos desde SQL en lugar de en un libro de contabilidad de la cadena de bloques. Además, gracias a la estandarización en un esquema estándar que es independiente de las pilas de tecnologías de cadena de bloques, el almacenamiento fuera de la cadena permite la reutilización de los informes y otros artefactos en varios proyectos, escenarios y organizaciones.

## <a name="azure-storage"></a>Azure Storage

Azure Storage se usa para almacenar los contratos y los metadatos asociados con ellos.

Desde pedidos de compra y facturas de embarque o imágenes utilizadas en noticias e imágenes médicas, hasta vídeo procedente de cámaras de cuerpos de policía y películas comerciales, los documentos desempeñan un papel importante en muchos escenarios centrados en la cadena de bloques. No es adecuado situar los documentos directamente en la cadena de bloques.

Blockchain Workbench permite la posibilidad de agregar documentos u otro contenido multimedia con la lógica de negocios de la cadena de bloques. En la cadena de bloques se almacena un hash del documento o contenido multimedia y el contenido o documento real se almacena en Azure Storage. La información de la transacción asociada se entrega al agente de mensajes entrantes, se empaqueta, se firma y se enruta a la cadena de bloques. Este proceso desencadena eventos que se comparten a través del agente de mensajes salientes. SQL Database consume esta información y la envía a la base de datos para su consulta posterior. Los sistemas de nivel final también pueden consumir estos eventos para actuar según corresponda.

## <a name="monitoring"></a>Supervisión

Workbench proporciona el registro de aplicación con Application Insights y Azure Monitor. Application Insights se utiliza para almacenar toda la información registrada de Blockchain Workbench e incluye errores, advertencias y operaciones correctas. Los desarrolladores pueden usar Application Insights para depurar problemas con Blockchain Workbench. 

Azure Monitor proporciona información sobre el mantenimiento de la red de la cadena de bloques. 

## <a name="next-steps"></a>Pasos siguientes

> [!div class="nextstepaction"]
> [Implementación de Azure Blockchain Workbench](./deploy.md)
