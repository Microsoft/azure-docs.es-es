---
title: Migrar aplicaciones desde BizTalk Services a Azure Logic Apps
description: Cómo migrar aplicaciones y soluciones de Microsoft Azure BizTalk Services (MABS) a Azure Logic Apps
services: logic-apps
ms.suite: integration
author: jonfancey
ms.author: jonfan
ms.reviewer: estfan, logicappspm
ms.topic: article
ms.date: 05/30/2017
ms.openlocfilehash: 6c07ab4b18c017bd29723d2640129b8e67374e3c
ms.sourcegitcommit: 2ff0d073607bc746ffc638a84bb026d1705e543e
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/06/2020
ms.locfileid: "87837387"
---
# <a name="migrate-your-apps-and-solutions-from-biztalk-services-to-azure-logic-apps"></a>Migre sus aplicaciones y soluciones de BizTalk Services a Azure Logic Apps

Microsoft Azure BizTalk Services (MABS) está en retirada. Para migrar las soluciones de integración de MABS a [Azure Logic Apps](../logic-apps/logic-apps-overview.md), siga las orientaciones de este artículo. 

## <a name="introduction"></a>Introducción

BizTalk Services consta de dos subservicios:

* Conexiones híbridas de Microsoft BizTalk Services
* Integración basada en puente de EAI y EDI

[Conexiones híbridas de Azure App Service](../app-service/app-service-hybrid-connections.md) reemplaza a Conexiones híbridas de BizTalk Services. Conexiones híbridas de Azure está disponible en Azure App Service a través de Azure Portal. Este servicio proporciona un nuevo Administrador de conexiones híbridas para administrar las conexiones híbridas existentes de BizTalk Services y también las nuevas conexiones híbridas que cree en el portal. 

[Logic Apps](../logic-apps/logic-apps-overview.md) reemplaza la integración basada en puentes de EAI y EDI por todas las mismas funciones de BizTalk Services y alguna más. Este servicio proporciona características de orquestación y flujo de trabajo basado en consumo a escala de nube que permiten compilar rápida y fácilmente soluciones complejas de integración con un explorador o con Visual Studio.

Esta tabla asigna funciones de BizTalk Services a Logic Apps.

| BizTalk Services   | Logic Apps            | Propósito                      |
| ------------------ | --------------------- | ---------------------------- |
| Conector          | Conector             | Envío y recepción de datos   |
| Puente             | Aplicación lógica             | Procesador de canalización           |
| Fase de validación     | Acción de validación XML | Validación de un documento XML con respecto a un esquema | 
| Fase de enriquecimiento       | Tokens de datos           | Promoción de propiedades en mensajes o para enrutar decisiones |
| Fase de transformación    | Acción de transformación      | Conversión de mensajes XML de un formato a otro |
| Fase de descodificación       | Acción de descodificación de archivo plano | Conversión de archivo plano a XML |
| Fase de codificación       | Acción de codificación de archivo plano | Conversión de XML a archivo plano |
| Inspector de mensajes  | Azure Functions o API Apps | Ejecución de código personalizado en las integraciones |
| Acción de enrutamiento       | Condición o modificador | Enrutamiento de mensajes a uno de los conectores especificados |
|||| 

## <a name="biztalk-services-artifacts"></a>Artefactos de BizTalk Services

BizTalk Services tiene varios tipos de artefactos. 

## <a name="connectors"></a>Conectores

Los conectores de BizTalk Services ayudan a los puentes a enviar y recibir datos, incluidos los puentes bidireccionales que habilitan interacciones de solicitud/respuesta basadas en HTTP. Logic Apps usa la misma terminología y tiene cientos de conectores que sirven al mismo objetivo conectándose a una amplia gama de tecnologías y servicios. Por ejemplo, los conectores están disponibles para servicios SaaS y PaaS en la nube, como OneDrive, Office365, Dynamics CRM y otros, además de sistemas locales a través de la Puerta de enlace de datos local, que reemplaza al Servicio de adaptador de BizTalk de BizTalk Services. Los orígenes de BizTalk Services están limitados a FTP, SFTP y suscripción a Tema o Cola de Service Bus.

![Diagrama que muestra el flujo de BizTalk Services.](media/logic-apps-move-from-mabs/sources.png)

De manera predeterminada, cada puente tiene un punto de conexión HTTP, el que se configura con las propiedades Dirección en tiempo de ejecución y Dirección relativa del puente. Para lograr los mismos resultados con Logic Apps, use las acciones [Solicitud y respuesta](../connectors/connectors-native-reqres.md).

## <a name="xml-processing-and-bridges"></a>Puentes y procesamiento de XML

En BizTalk Services, un puente es análogo a una canalización de procesamiento. Un puente puede tomar los datos recibidos desde un conector, trabajar con esos datos y luego enviarlos a otro sistema. Logic Apps hace exactamente lo mismo, ya que es compatible con los mismos patrones de interacción basada en canalización que BizTalk Services y, además, proporciona otros patrones de integración. El [puente de solicitud-respuesta XML](/previous-versions/azure/hh689781(v=azure.100)) en BizTalk Services es conocido como una canalización VETER, que consta de fases en que se realizan estas tareas:

* (V) Validación
* (E) Enriquecimiento
* (T) Transformación
* (E) Enriquecimiento
* (R) EnRutamiento

En esta imagen se muestra cómo se divide el procesamiento entre solicitud y respuesta, lo cual ofrece control sobre las rutas de solicitud y respuesta de forma separada, por ejemplo, utilizando distintas asignaciones para cada una:

![Captura de pantalla que muestra cómo se divide el procesamiento entre la solicitud y la respuesta.](media/logic-apps-move-from-mabs/xml-request-reply.png)

Además, un puente unidireccional XML agrega las fases de descodificación y codificación al inicio y el fin del procesamiento. El puente de paso a través contiene una sola fase de enriquecimiento.

### <a name="message-processing-decoding-and-encoding"></a>Procesamiento, descodificación y codificación de mensajes

En BizTalk Services, puede recibir mensajes XML de distintos tipos y determinar el esquema coincidente para el mensaje recibido. Este trabajo se realiza en la fase *Tipos de mensaje* de la canalización de procesamiento de recepción. Luego, la fase de descodificación usa el tipo de mensaje detectado para descodificarlo con el esquema proporcionado. Si el esquema es un esquema de archivo plano, esta fase convierte el archivo plano entrante en XML. 

Logic Apps proporciona funcionalidades similares. Recibe un archivo plano a través de diferentes protocolos con los distintos desencadenadores de conector (sistema de archivos, FTP, HTTP, etc.) y usa la acción [Descodificación de archivo plano](../logic-apps/logic-apps-enterprise-integration-flatfile.md) para convertir los datos entrantes en XML. Puede mover los esquemas de archivo plano existentes directamente a Logic Apps sin que sea necesario hacer ningún cambio y, luego, cargar los esquemas en la cuenta de integración.

### <a name="validation"></a>Validación

Una vez que los datos entrantes se convierten en XML (o si el formato de mensaje que se recibió era XML), la validación se ejecuta para determinar si el mensaje cumple con el esquema XSD. Para hacer esta tarea en Logic Apps, use la acción [Validación XML](../logic-apps/logic-apps-enterprise-integration-xml-validation.md). Puede usar los mismos esquemas de BizTalk Services sin hacer ningún cambio.

### <a name="transform-messages"></a>Transformación de mensajes

En BizTalk Services, la fase de transformación convierte un formato de mensaje XML en otro. Este trabajo se hace aplicando un mapa, mediante el mapeador basado en TRFM. El proceso es similar en Logic Apps. La acción de transformación ejecuta un mapa desde la cuenta de integración. La principal diferencia es que los mapas de Logic Apps están en el formato XSLT. XSLT incluye la capacidad de volver a usar el XSLT existente que ya tiene, incluidos los mapas creados para BizTalk Server que contienen functioids. 

### <a name="routing-rules"></a>Reglas de enrutamiento

BizTalk Services toma una decisión de enrutamiento sobre qué punto de conexión o conector envía los mensajes o datos entrantes. La capacidad de seleccionar a partir de los puntos de conexión preconfigurados es posible con la opción de filtro de enrutamiento:

![Captura de pantalla que muestra la opción de filtro de enrutamiento.](media/logic-apps-move-from-mabs/route-filter.png)

En BizTalk Services, si hay solo dos opciones, el uso de una *condición* es la mejor manera de convertir filtros de enrutamiento en BizTalk Services. Si hay más de dos, use un **modificador**.

Logic Apps ofrece funcionalidades lógicas más sofisticadas, además de flujo de control y enrutamiento avanzandos, con [declaraciones condicionales](../logic-apps/logic-apps-control-flow-conditional-statement.md) y [declaraciones de conmutación](../logic-apps/logic-apps-control-flow-switch-statement.md).

### <a name="enrich"></a>Enriquecimiento

En el procesamiento de BizTalk Services, la fase de enriquecimiento agrega propiedades al contexto de mensaje asociado con los datos recibidos. Por ejemplo, promover una propiedad para usarla para el enrutamiento desde una búsqueda en base de datos o mediante la extracción de un valor con una expresión XPath. Logic Apps proporciona acceso a todas las salidas de datos contextuales de acciones precedentes, lo que simplifica replicar el mismo comportamiento. Por ejemplo, mediante la acción de conexión SQL `Get Row`, devuelve datos desde una base de datos de SQL Server y usa esos datos en una acción de decisión para enrutamiento. Del mismo modo, las propiedades en mensajes en cola entrantes de Service Bus por un desencadenador son direccionables, al igual que XPath con la expresión de lenguaje de definición de flujo de trabajo XPath.

### <a name="run-custom-code"></a>Ejecución de código personalizado

BizTalk Services le permite [ejecutar código personalizado](/previous-versions/azure/dn232389(v=azure.100)) que está cargado en sus propios ensamblados. Esta función se implementa mediante la interfaz [IMessageInspector](). Cada fase del puente incluye dos propiedades (On Enter Inspector y On Exit Inspector) que proporciona el tipo de .NET que se creó que implementa esta interfaz. El código personalizado le permite realizar procesamientos más complejos en los datos, así como volver a usar código existente en ensamblados que realizan una lógica de negocios común. 

Logic Apps proporciona dos maneras principales para ejecutar el código personalizado: Azure Functions y API Apps. Es posible crear y llamar a Azure Functions desde aplicaciones lógicas. Consulte [Adición y ejecución de código personalizado para aplicaciones lógicas con Azure Functions](../logic-apps/logic-apps-azure-functions.md). Use API Apps, parte de Azure App Service, para crear sus propios desencadenadores y acciones. Obtenga más información sobre cómo [crear una API personalizada para usarla con Logic Apps](../logic-apps/logic-apps-create-api-app.md). 

Si tiene código personalizado en ensamblados a los que llama desde BizTalk Services, puede mover este código a Azure Functions o crear API personalizadas con API Apps, en función de lo que está implementando. Por ejemplo, si tiene código que encapsula otro servicio para el que Logic Apps no tiene conector, cree una instancia de API Apps y use las acciones que esta instancia proporciona dentro de la aplicación lógica. Si tiene funciones o bibliotecas auxiliares, Azure Functions es probablemente la mejor opción.

### <a name="edi-processing-and-trading-partner-management"></a>Administración de entidades y procesamiento de EDI

BizTalk Services y Logic Apps incluyen procesamiento de EDI y B2B con compatibilidad con AS2 (Applicability Statement 2), X12 y EDIFACT. En BizTalk Services, se crean puentes de EDI y se crean o administran entidades y contratos en el portal de seguimiento y administración dedicado.
En Logic Apps, esta funcionalidad se obtiene mediante [Enterprise Integration Pack (EIP)](../logic-apps/logic-apps-enterprise-integration-overview.md). EIP consta de la [cuenta de integración](../logic-apps/logic-apps-enterprise-integration-create-integration-account.md) y acciones de B2B para procesamiento de EDI y B2B. La cuenta de integración se usa también para crear y administrar [entidades](../logic-apps/logic-apps-enterprise-integration-partners.md) y [contratos](../logic-apps/logic-apps-enterprise-integration-agreements.md). Una vez que crea una cuenta de integración, puede vincular una o varias aplicaciones lógicas a la cuenta. A continuación puede usar las acciones de B2B para obtener acceso a la información de entidades dentro de la aplicación lógica. Se proporcionan las siguientes acciones:

* Codificación AS2
* Descodificación AS2
* Codificación X12
* Descodificación X12
* Codificación EDIFACT
* Descodificación EDIFACT

A diferencia de BizTalk Services, estas acciones se separan de los protocolos de transporte. Por lo tanto, cuando crea las aplicaciones lógicas, tiene más flexibilidad sobre qué conectores usar para enviar y recibir datos. Por ejemplo, puede recibir archivos X12 como archivos adjuntos de correo electrónico y luego procesar esos archivos en una aplicación lógica. 

## <a name="manage-and-monitor"></a>Administración y supervisión

En BizTalk Services, un portal dedicado ofrecía funciones de seguimiento para supervisar y solucionar problemas. Logic Apps ofrece funcionalidades más completas de seguimiento y supervisión para [supervisar las aplicaciones lógicas en Azure Portal](../logic-apps/monitor-logic-apps.md), e incluye una aplicación móvil para que pueda vigilar las cosas cuando esté en movimiento.

## <a name="high-availability"></a>Alta disponibilidad

Para la alta disponibilidad (HA) en BizTalk Services, es posible compartir la carga de procesamiento usando más de una instancia de una región específica. Logic Apps proporciona alta disponibilidad en la región sin costos adicionales. 

En BizTalk Services, la recuperación ante desastres fuera de la región para el procesamiento B2B requiere un proceso de copia de seguridad y restauración. Para la continuidad del negocio, Logic Apps proporciona una [funcionalidad de recuperación ante desastres](../logic-apps/logic-apps-enterprise-integration-b2b-business-continuity.md) activa/pasiva entre regiones que le permite sincronizar datos B2B entre cuentas de integración en distintas regiones.

## <a name="next-steps"></a>Pasos siguientes

* [¿Qué es Azure Logic Apps?](../logic-apps/logic-apps-overview.md)
* [Cree la primera aplicación lógica](../logic-apps/quickstart-create-first-logic-app-workflow.md) o empiece a trabajar rápidamente mediante una [plantilla precompilada](../logic-apps/logic-apps-create-logic-apps-from-templates.md)  
* [Vea todos los conectores disponibles](../connectors/apis-list.md) que se pueden utilizar en aplicaciones lógicas.