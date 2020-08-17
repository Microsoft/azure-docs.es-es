---
title: 'Patrones de integración de contratos inteligentes: Azure Blockchain Workbench'
description: Introducción a los patrones de integración de contratos inteligentes en la versión preliminar de Azure Blockchain Workbench.
ms.date: 11/20/2019
ms.topic: conceptual
ms.reviewer: mmercuri
ms.openlocfilehash: dae63e16356e825d3be31380df1648749e59d8bd
ms.sourcegitcommit: 3543d3b4f6c6f496d22ea5f97d8cd2700ac9a481
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/20/2020
ms.locfileid: "86538384"
---
# <a name="smart-contract-integration-patterns"></a>Patrones de integración de contratos inteligentes

Los contratos inteligentes representan a menudo un flujo de trabajo empresarial que necesita integrarse con sistemas y dispositivos externos.

Los requisitos de estos flujos de trabajo incluyen la necesidad de iniciar transacciones en un libro de contabilidad distribuida que incluye datos de un sistema, servicio o dispositivo externo. También necesitan que los sistemas externos reaccionen a eventos que se originan en los contratos inteligentes de un libro de contabilidad distribuida.

La API REST y la integración de mensajería envían las transacciones de sistemas externos a contratos inteligentes incluidos en una aplicación de Azure Blockchain Workbench. También envían notificaciones de eventos a sistemas externos según los cambios que tienen lugar dentro de una aplicación.

Para los escenarios de integración de datos, Azure Blockchain Workbench incluye un conjunto de vistas de base de datos que permiten combinar los datos transaccionales de la cadena de bloques y los metadatos sobre las aplicaciones y los contratos inteligentes.

Además, algunos escenarios, como los relacionados con la cadena o los medios de suministro, también pueden requerir la integración de documentos. Aunque Azure Blockchain Workbench no proporciona llamadas API para administrar los documentos directamente, estos se pueden incorporar a una aplicación de cadena de bloques. Esta sección también incluye ese patrón.

Esta sección incluye los patrones que se identificaron para la implementación de cada uno de estos tipos de integraciones en sus soluciones globales.

## <a name="rest-api-based-integration"></a>Integración basada en API REST

Las funcionalidades de la aplicación web que generó Azure Blockchain Workbench se exponen a través de la API REST. Las funcionalidades incluyen la carga, configuración y administración de aplicaciones mediante Azure Blockchain Workbench, el envío de transacciones a un libro de contabilidad distribuida y la consulta de los metadatos de la aplicación y los datos del libro de contabilidad.

La API REST se usa principalmente con clientes interactivos como aplicaciones web, móviles y bot.

En esta sección se describen patrones que se centran en los aspectos de la API de REST que envían transacciones a un libro de contabilidad distribuida, además de patrones que consultan datos sobre transacciones de la base de datos *fuera de la cadena* de Azure Blockchain Workbench.

### <a name="sending-transactions-to-a-distributed-ledger-from-an-external-system"></a>Envío de transacciones a un libro de contabilidad distribuida desde un sistema externo

La API REST de Azure Blockchain Workbench envía solicitudes autenticadas para ejecutar transacciones en un libro de contabilidad distribuida.

![Envío de transacciones a un libro de contabilidad distribuida](./media/integration-patterns/send-transactions-ledger.png)

La ejecución de transacciones se produce mediante el proceso descrito anteriormente, en el que:

-   La aplicación externa se autentica en la instancia de Azure Active Directory que se aprovisionó como parte de la implementación de Azure Blockchain Workbench.
-   Los usuarios autorizados reciben un token de portador que se puede enviar con solicitudes a la API.
-   Las aplicaciones externas realizan llamadas a la API REST mediante el token de portador.
-   La API REST empaqueta la solicitud como un mensaje y la envía a Service Bus. Desde aquí se recupera, firma y envía al libro de contabilidad distribuida adecuado.
-   La API REST realiza una solicitud a la base de datos SQL de Azure Blockchain Workbench para registrar la solicitud y establecer el estado de aprovisionamiento actual.
-   La base de datos SQL devuelve el estado de aprovisionamiento y la llamada API devuelve el identificador a la aplicación externa que lo llamó.

### <a name="querying-blockchain-workbench-metadata-and-distributed-ledger-transactions"></a>Consulta de metadatos y transacciones de libro de contabilidad distribuida de Blockchain Workbench

La API REST de Azure Blockchain Workbench envía solicitudes autenticadas para consultar detalles relacionados con la ejecución de contratos inteligentes en un libro de contabilidad distribuida.

![Consulta de metadatos](./media/integration-patterns/querying-metadata.png)

Las consultas se producen mediante el proceso descrito anteriormente, en el que:

1. La aplicación externa se autentica en la instancia de Azure Active Directory que se aprovisionó como parte de la implementación de Azure Blockchain Workbench.
2. Los usuarios autorizados reciben un token de portador que se puede enviar con solicitudes a la API.
3. Las aplicaciones externas realizan llamadas a la API REST mediante el token de portador.
4. La API REST consulta los datos de la solicitud desde la base de datos SQL y los devuelve al cliente.

## <a name="messaging-integration"></a>Integración de mensajería

La integración de mensajería facilita la interacción con sistemas, servicios y dispositivos en los que un inicio de sesión interactivo no es posible o conveniente. La integración de mensajería se centra en dos tipos de mensajes: los mensajes que solicitan que las transacciones se ejecuten en un libro de contabilidad distribuida y los eventos que ese libro de contabilidad expone cuando las transacciones han tenido lugar.

La integración de mensajería se centra en la ejecución y supervisión de transacciones relacionadas con la creación de usuarios, la creación de contratos y la ejecución de transacciones de contratos y la usan principalmente los sistemas back-end *sin periféricos*.

En esta sección se describen los patrones que se centran en los aspectos de la API basada en mensajes que envían transacciones a un libro de contabilidad distribuida, además de en los patrones que representan los mensajes de eventos que expone el libro de contabilidad distribuida subyacente.

### <a name="one-way-event-delivery-from-a-smart-contract-to-an-event-consumer"></a>Entrega de eventos unidireccional desde un contrato inteligente a un consumidor de eventos 

En este escenario se produce un evento en un contrato inteligente como, por ejemplo, un cambio de estado o la ejecución de un tipo concreto de transacción. Este evento se difunde a través de una instancia de Event Grid a los consumidores de nivel inferior y, a continuación, estos toman las medidas oportunas.

Un ejemplo de este escenario se da cuando se produce una transacción, un consumidor recibe una alerta y puede realizar acciones como, por ejemplo, el registro de la información en una base de datos SQL o en Common Data Service. Este escenario es el mismo patrón que sigue Workbench para rellenar la base de datos SQL *fuera de la cadena*.

Otro escenario se da cuando un contrato inteligente efectúa una transición a un estado determinado como, por ejemplo, si un contrato cambia al estado *OutOfCompliance*. Cuando se produce este cambio de estado, se puede desencadenar una alerta que se envíe al teléfono móvil de un administrador.

![Entrega de eventos unidireccional](./media/integration-patterns/one-way-event-delivery.png)

Este escenario se produce mediante el proceso descrito anteriormente, en el que:

-   El contrato inteligente efectúa la transición a un estado nuevo y envía un evento al libro de contabilidad.
-   El libro de contabilidad recibe y entrega el evento a Azure Blockchain Workbench.
-   Azure Blockchain Workbench está suscrito a los eventos del libro de contabilidad y recibe el evento.
-   Azure Blockchain Workbench publica el evento en los suscriptores de Event Grid.
-   Los sistemas externos están suscritos a Event Grid, consumen el mensaje y toman las acciones adecuadas.

## <a name="one-way-event-delivery-of-a-message-from-an-external-system-to-a-smart-contract"></a>Entrega de eventos unidireccional de un mensaje desde un sistema externo a un contrato inteligente

También existe un escenario que fluye desde la dirección opuesta. En este caso, un sensor o sistema externo genera un evento y se debe enviar este evento a un contrato inteligente.

Un ejemplo común es la entrega de datos obtenidos de mercados financieros como, por ejemplo, precios de mercancías, productos en stock, o bonos, a un contrato inteligente.

### <a name="direct-delivery-of-an-azure-blockchain-workbench-in-the-expected-format"></a>Entrega directa de una instancia de Azure Blockchain Workbench en el formato esperado

Algunas aplicaciones se compilan para integrarse con Azure Blockchain Workbench y generar y enviar mensajes en los formatos esperados.

![Entrega directa](./media/integration-patterns/direct-delivery.png)

Esta entrega se produce mediante el proceso descrito anteriormente, en el que:

-   Se produce un evento en un sistema externo que desencadena la creación de un mensaje para Azure Blockchain Workbench.
-   El sistema externo tiene código escrito para crear este mensaje en un formato conocido y enviarlo directamente a Service Bus.
-   Azure Blockchain Workbench está suscrito a los eventos de Service Bus y recupera el mensaje.
-   Azure Blockchain Workbench inicia una llamada al libro de contabilidad, enviando los datos del sistema externo a un contrato determinado.
-   Tras la recepción del mensaje, el contrato efectúa una transición a un estado nuevo.

### <a name="delivery-of-a-message-in-a-format-unknown-to-azure-blockchain-workbench"></a>Entrega de un mensaje en un formato desconocido a Azure Blockchain Workbench

Algunos sistemas no se pueden modificar para entregar mensajes en los formatos estándar que usa Azure Blockchain Workbench. En tales casos, a menudo se pueden emplear los mecanismos y formatos de mensajes existentes en estos sistemas. Concretamente, los tipos de mensaje nativos de estos sistemas se pueden transformar mediante Logic Apps, Azure Functions u otro código personalizado para que se asignen a uno de los formatos de mensajería estándar esperados.

![Formato de mensaje desconocido](./media/integration-patterns/unknown-message-format.png)

Esto se produce mediante el proceso descrito anteriormente, en el que:

-   Se produce un evento en un sistema externo que desencadena la creación de un mensaje.
-   Se emplea una instancia de Logic Apps o código personalizado para recibir ese mensaje y transformarlo en un mensaje con un formato estándar de Azure Blockchain Workbench.
-   La instancia de Logic Apps envía el mensaje transformado directamente a Service Bus.
-   Azure Blockchain Workbench está suscrito a los eventos de Service Bus y recupera el mensaje.
-   Azure Blockchain Workbench inicia una llamada al libro de contabilidad, enviando los datos del sistema externo a una función específica del contrato.
-   La función se ejecuta y normalmente se modifica el estado. El cambio de estado hace avanzar el flujo de trabajo empresarial reflejado en el contrato inteligente, lo que permite que ahora se puedan ejecutar otras funciones según sea adecuado.

### <a name="transitioning-control-to-an-external-process-and-await-completion"></a>Efectuar la transición del control a un proceso externo y esperar la finalización

Hay escenarios en los que un contrato inteligente debe detener la ejecución interna y entregar a un proceso externo. Ese proceso externo, a continuación, completaría y enviaría un mensaje al contrato inteligente, y la ejecución continuaría posteriormente dentro de este.

#### <a name="transition-to-the-external-process"></a>Transición al proceso externo

Este patrón normalmente se implementa mediante el siguiente método:

-   El contrato inteligente efectúa una transición a un estado específico. En este estado, no se puede ejecutar ninguna función, o solo un número limitado de ellas, hasta que un sistema externo realice una acción deseada.
-   El cambio de estado aparece como un evento para un consumidor de nivel inferior.
-   El consumidor de nivel inferior recibe el evento y desencadena la ejecución externa del código.

![El diagrama muestra un cambio de estado en el contrato que provoca que un evento vaya al libro de contabilidad distribuida. Luego, Blockchain Workbench toma el evento y lo publica.](./media/integration-patterns/transition-external-process.png)

#### <a name="return-of-control-from-the-smart-contract"></a>Devolución del control desde el contrato inteligente

Según la capacidad de personalizar el sistema externo, es posible que este pueda enviar mensajes, o no, en alguno de los formatos estándar que espera Azure Blockchain Workbench. Según la capacidad de los sistemas externos para generar uno de estos mensajes se determina cuál de las dos rutas de devolución descritas a continuación se usará.

##### <a name="direct-delivery-of-an-azure-blockchain-workbench-in-the-expected-format"></a>Entrega directa de una instancia de Azure Blockchain Workbench en el formato esperado

![El diagrama muestra el mensaje A P I del sistema externo que Blockchain Workbench ha seleccionado a través de Service Bus. Luego, Blockchain Workbench envía un mensaje, en forma de transacción, al libro de contabilidad distribuida en nombre del agente. Se pasa al contrato, donde provoca un cambio de estado.](./media/integration-patterns/direct-delivery.png)

En este modelo, se produce la comunicación con el contrato y el cambio de estado subsiguiente siguiendo los pasos del proceso descrito anteriormente, en el que:

-   Al finalizar o llegar a un hito determinado en la ejecución externa del código, se envía un evento a la instancia de Service Bus conectada a Azure Blockchain Workbench.

-   En aquellos sistemas que no se pueden adaptar directamente para escribir un mensaje que cumpla las expectativas de la API, este se transforma.

-   El contenido del mensaje se empaqueta y se envía a una función específica del contrato inteligente. Esta entrega se realiza en nombre del usuario asociado con el sistema externo.

-   La función se ejecuta y normalmente se modifica el estado. El cambio de estado hace avanzar el flujo de trabajo empresarial reflejado en el contrato inteligente, lo que permite que ahora se puedan ejecutar otras funciones según sea adecuado.

### 

### <a name="delivery-of-a-message-in-a-format-unknown-to-azure-blockchain-workbench"></a>Entrega de un mensaje en un formato desconocido a Azure Blockchain Workbench

![Formato de mensaje desconocido](./media/integration-patterns/unknown-message-format.png)

En este modelo, en el que no se puede enviar un mensaje directamente en un formato estándar, se produce la comunicación con el contrato y el cambio de estado subsiguiente siguiendo los pasos del proceso descrito anteriormente, en el que:

1.  Al finalizar o llegar a un hito determinado en la ejecución externa del código, se envía un evento a la instancia de Service Bus conectada a Azure Blockchain Workbench.
2.  Se emplea una instancia de Logic Apps o código personalizado para recibir ese mensaje y transformarlo en un mensaje con un formato estándar de Azure Blockchain Workbench.
3.  La instancia de Logic Apps envía el mensaje transformado directamente a Service Bus.
4.  Azure Blockchain Workbench está suscrito a los eventos de Service Bus y recupera el mensaje.
5.  Azure Blockchain Workbench inicia una llamada al libro de contabilidad, enviando los datos del sistema externo a un contrato determinado.
6. El contenido del mensaje se empaqueta y se envía a una función específica del contrato inteligente. Esta entrega se realiza en nombre del usuario asociado con el sistema externo.
7.  La función se ejecuta y normalmente se modifica el estado. El cambio de estado hace avanzar el flujo de trabajo empresarial reflejado en el contrato inteligente, lo que permite que ahora se puedan ejecutar otras funciones según sea adecuado.

## <a name="iot-integration"></a>Integración de IoT

Un escenario de integración habitual es la inclusión de datos de telemetría que se recuperan a partir de los sensores de un contrato inteligente. Según los datos entregados por los sensores, los contratos inteligentes pueden realizar acciones basadas en la información y alterar el estado del contrato.

Por ejemplo, si un camión de reparto de medicamentos experimenta una subida de temperatura de 110 grados, esta puede afectar a la eficacia del medicamento y provocar un problema de seguridad pública si no se detecta y se retira de la cadena de suministro. O si un conductor acelera el coche a más de 160 kilómetros por hora, la información resultante del sensor puede desencadenar la cancelación del seguro por parte de la empresa aseguradora. Si el coche era uno de alquiler, los datos del GPS podrían indicar si el conductor se salió del área geográfica cubierta por el contrato de alquiler y, en consecuencia, aplicar una penalización.

El desafío es que estos sensores pueden entregar datos de forma constante pero no es adecuado enviar todos los datos a un contrato inteligente. Un enfoque habitual consiste en limitar el número de mensajes que se envían a la cadena de bloques, aunque se siguen entregando todos los mensajes a un almacenamiento secundario. Por ejemplo, entregar los mensajes que se reciban solamente en un intervalo específico como, por ejemplo, una vez cada hora, o si uno de los valores que contiene está fuera de un intervalo previamente acordado para un contrato inteligente. Comprobar los valores que se encuentran fuera de las tolerancias, garantiza que los datos relevantes para la lógica de negocios de los contratos se reciben y ejecutan. Comprobar el valor a intervalos permite confirmar que el sensor sigue informando. Todos los datos se envían a un almacén de informes secundario que permite más informes, análisis y aprendizaje automático. Por ejemplo, puede que no sea necesario obtener las lecturas del sensor GPS cada minuto para un contrato inteligente, pero estas lecturas podrían proporcionar datos interesantes para su uso en informes o rutas de asignación.

En la plataforma Azure, la integración con dispositivos se suele realizar con IoT Hub. IoT Hub proporciona enrutamiento de los mensajes según el contenido y permite el tipo de funcionalidad descrito anteriormente.

![Mensajes de IoT](./media/integration-patterns/iot.png)

El proceso describe un patrón:

-   Un dispositivo se comunica directamente o a través de una puerta de enlace de campo con IoT Hub.
-   IoT Hub recibe los mensajes y los evalúa comparándolos con las rutas establecidas que comprueban el contenido del mensaje, por ejemplo. *¿El sensor notifica una temperatura superior a 50 grados?*
-   IoT Hub envía los mensajes que cumplen los criterios a una instancia de Service Bus definida para la ruta.
-   Una aplicación lógica u otro código escucha a la instancia de Service Bus que IoT Hub ha establecido para la ruta.
-   La aplicación lógica o el otro código recupera y transforma el mensaje a un formato conocido.
-   El mensaje transformado, ya en un formato estándar, se envía a la instancia de Service Bus para Azure Blockchain Workbench.
-   Azure Blockchain Workbench está suscrito a los eventos de Service Bus y recupera el mensaje.
-   Azure Blockchain Workbench inicia una llamada al libro de contabilidad, enviando los datos del sistema externo a un contrato determinado.
-   Tras la recepción del mensaje, el contrato evalúa los datos y puede cambiar el estado en función del resultado de esa evaluación, por ejemplo, en caso de una temperatura demasiado alta, cambia el estado a *No conforme*.

## <a name="data-integration"></a>Integración de datos

Además de REST y la API basada en mensajes, Azure Blockchain Workbench también proporciona acceso a una base de datos SQL rellena con metadatos de aplicación y de contrato, así como con los datos transaccionales de los libros de contabilidad distribuida.

![Integración de datos](./media/integration-patterns/data-integration.png)

La integración de datos es bien conocida:

-   Azure Blockchain Workbench almacena metadatos acerca de aplicaciones, flujos de trabajo, contratos y transacciones como parte de su funcionamiento normal.
-   Los sistemas o herramientas externos proporcionan uno o más cuadros de diálogo para facilitar la recopilación de información acerca de la base de datos. Información como, por ejemplo, el nombre del servidor de bases de datos, el nombre de la base de datos, el tipo de autenticación, las credenciales de inicio de sesión y qué vistas de la base de datos se pueden utilizar.
-   Las consultas se escriben en las vistas de la base de datos para facilitar el consumo en niveles inferiores por parte de sistemas externos, servicios, informes, herramientas para desarrolladores y herramientas de productividad empresarial.

## <a name="storage-integration"></a>Integración del almacenamiento

Muchos escenarios pueden requerir la necesidad de incorporar archivos verificables. Por varios motivos, no resulta apropiado poner los archivos en una cadena de bloques. En vez de eso, un enfoque habitual es realizar un hash criptográfico (por ejemplo, SHA-256) en un archivo y compartir ese hash en un libro de contabilidad distribuida. Si se realiza el hash de nuevo en el futuro, este debería devolver el mismo resultado. Si se modifica el archivo, incluso aunque solo se modifique un píxel de una imagen, el hash devuelve un valor diferente.

![Integración del almacenamiento](./media/integration-patterns/storage-integration.png)

Se puede implementar un patrón, en el que:

-   Un sistema externo conserva un archivo en un mecanismo de almacenamiento como Azure Storage.
-   Se genera un hash con el archivo, o con el archivo y las metadatos asociados: un identificador del propietario, la dirección URL donde está ubicado el archivo, etc.
-   El hash y todos los metadatos se envían a una función de un contrato inteligente como, por ejemplo, *FileAdded*
-   En el futuro, se puede realizar un hash de nuevo al archivo y a los metadatos, y comparar los valores con los que hay almacenados en el libro de contabilidad.

## <a name="prerequisites-for-implementing-integration-patterns-using-the-rest-and-message-apis"></a>Requisitos previos para la implementación de patrones de integración mediante REST y la API de mensajes

Para mejorar la capacidad de un dispositivo o sistema externo de interactuar con el contrato inteligente mediante REST o la API de mensajes, se debe dar lo siguiente:

1. En la instancia de Azure Active Directory del consorcio se debe crear una cuenta que represente al sistema o dispositivo externo.
2. Al menos uno de los contratos inteligentes adecuados de la aplicación de Azure Blockchain Workbench tienen funciones definidas para aceptar los eventos de su dispositivo o sistema externo.
3. El archivo de configuración de la aplicación del contrato inteligente contiene el rol al que está asignado el sistema o dispositivo.
4. El archivo de configuración de la aplicación del contrato inteligente debe identificar en qué estados se llama a esta función mediante el rol definido.
5. El archivo de configuración de la aplicación y sus contratos inteligentes deben estar cargados en Azure Blockchain Workbench.

Una vez que se carga la aplicación, se debe asignar la cuenta de Azure Active Directory del sistema externo al contrato y al rol asociado.

## <a name="testing-external-system-integration-flows-prior-to-writing-integration-code"></a>Comprobación de los flujos de integración del sistema externo antes de escribir código de integración 

Integrarse con sistemas externos es un requisito principal en muchos escenarios. Es conveniente poder validar el diseño del contrato inteligente antes de desarrollar el código para la integración con sistemas externos o hacerlo de forma simultánea.

El uso de Azure Active Directory (Azure AD) puede acelerar en gran medida la productividad del desarrollador y el tiempo de amortización. En concreto, la integración de código con un sistema externo puede tardar una cantidad de tiempo que se debe tener en cuenta. Con el uso de Azure AD y la generación automática de experiencia del usuario mediante Azure Blockchain Workbench, puede permitir a los desarrolladores iniciar sesión en Blockchain Workbench como sistema externo y rellenar los valores de ese sistema mediante la experiencia de usuario. Puede desarrollar y validar rápidamente las ideas en un entorno de prueba de concepto antes de que se escriba el código de integración para los sistemas externos.
