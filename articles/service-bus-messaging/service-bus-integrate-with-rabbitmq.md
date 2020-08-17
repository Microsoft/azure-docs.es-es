---
title: Cómo integrar RabbitMQ con Azure Service Bus
description: Guía paso a paso sobre cómo integrar RabbitMQ con Azure Service Bus
services: Azure Service Bus
author: videlalvaro
ms.topic: how-to
ms.service: service-bus
ms.date: 07/02/2020
ms.author: alvidela
ms.openlocfilehash: 373629c86f2d842ad2e02dd2b66739f3963bf7ed
ms.sourcegitcommit: d8b8768d62672e9c287a04f2578383d0eb857950
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/11/2020
ms.locfileid: "88064560"
---
# <a name="how-to-integrate-rabbitmq-with-azure-service-bus"></a>Cómo integrar RabbitMQ con Azure Service Bus

En esta guía, vamos aprenderemos a enviar mensajes desde RabbitMQ hasta Azure Service Bus.

Estos son algunos escenarios en los que podemos hacer uso de estas funcionalidades:

- **Configuración perimetral**: Tenemos una configuración perimetral en la que enviamos mensajes a RabbitMQ, pero queremos reenviar esos mensajes a [Azure Service Bus](./service-bus-messaging-overview.md) para un procesamiento posterior, de modo que podamos usar muchas de las [funciones de macrodatos de Azure ](/azure/architecture/guide/architecture-styles/big-data).
- **Nube híbrida**: Su empresa acaba de adquirir un tercero que usa RabbitMQ para sus necesidades de mensajería. Se encuentran en una nube diferente. Mientras realizan la transición a Azure, ya puede empezar a compartir datos mediante el puente de RabbitMQ con Azure Service Bus.
- **Integración de terceros**: Un tercero usa RabbitMQ como agente y desea enviarnos sus datos, pero están fuera de la organización. Podemos proporcionarles una clave SAS que les permita acceder a un conjunto limitado de colas de Azure Service Bus a las que pueden reenviar sus mensajes.

La lista continúa, pero podemos resolver la mayoría de estos casos de uso mediante el puente de RabbitMQ a Azure.

En primer lugar, debe crear una cuenta gratuita de Azure al registrarse [aquí](https://azure.microsoft.com/free/)

Una vez que haya iniciado sesión en su cuenta, vaya a [Azure Portal](https://portal.azure.com/) y cree un nuevo Azure Service Bus [espacio de nombres](./service-bus-create-namespace-portal.md). Los espacios de nombres son los contenedores de alcance donde se activarán nuestros componentes de mensajería, como las colas y los temas.

## <a name="adding-a-new-azure-service-bus-namespace"></a>Agregue un nuevo espacio de nombres de Azure Service Bus

En Azure Portal, haga clic en el botón más grande para agregar un nuevo recurso

:::image type="content" source="./media/service-bus-integrate-with-rabbitmq/create-resource.png" alt-text="Crear recurso":::

Después, seleccione Integración y haga clic en Azure Service Bus para crear un espacio de nombres de mensajería:

:::image type="content" source="./media/service-bus-integrate-with-rabbitmq/integration.png" alt-text="Seleccione Azure Service Bus":::

Se le pedirá que ingrese la información del espacio de nombres. Seleccione la suscripción de Azure que quiere usar. Si no tiene un [grupo de recursos](../azure-resource-manager/management/manage-resource-groups-portal.md) existente, puede crear uno.

:::image type="content" source="./media/service-bus-integrate-with-rabbitmq/create-namespace.png" alt-text="Crear un espacio de nombres":::

Use `rabbitmq` para `Namespace name`, pero podría ser todo lo que desee. A continuación, establezca `East US` para la ubicación. Elija `Basic` como el plan de tarifa.

Si todo ha salido bien, debería ver la siguiente pantalla de confirmación:

:::image type="content" source="./media/service-bus-integrate-with-rabbitmq/create-namespace-confirm.png" alt-text="Confirmación de crear espacio de nombres":::

Al volver a Azure Portal, verá el nuevo espacio de nombres `rabbitmq` en la lista. Haga clic en él para tener acceso al recurso, de modo que pueda agregarle una cola.

:::image type="content" source="./media/service-bus-integrate-with-rabbitmq/resource-view-with-namespace.png" alt-text="Lista de recursos con nuevo espacio de nombres":::

## <a name="creating-our-azure-service-bus-queue"></a>Crear nuestra cola de Azure Service Bus

Ahora que tiene su espacio de nombres de Azure Service Bus, haga clic en el botón `Queues` de la izquierda, debajo de `Entities`, para agregar una nueva cola:

:::image type="content" source="./media/service-bus-integrate-with-rabbitmq/create-queue.png" alt-text="Crear cola":::

El nombre de la cola será `from-rabbitmq` como un recordatorio de dónde proceden los mensajes. Puede dejar las demás opciones como valores predeterminados, pero puede cambiarlas para adaptarlas a las necesidades de su aplicación.

## <a name="enabling-the-rabbitmq-shovel-plugin"></a>Habilitar el complemento Shovel de RabbitMQ

Para enviar mensajes desde RabbitMQ a Azure Service Bus, utilizaremos el [complemento Shovel](https://www.rabbitmq.com/shovel.html) que viene empaquetado con RabbitMQ. Puede habilitar el complemento y su interfaz visual con este comando:

```bash
rabbitmq-plugins enable rabbitmq_shovel_management
```

>Es posible que tenga que ejecutar ese comando como raíz.

Ahora es el momento de obtener las credenciales necesarias para conectar RabbitMQ con Azure.

## <a name="connecting-rabbitmq-to-azure-service-bus"></a>Conexión de RabbitMQ con Azure Service Bus

Tendrá que crear una [política de acceso compartido](../storage/common/storage-sas-overview.md) (SAS) para la cola, de modo que RabbitMQ pueda publicar mensajes en ella. Una directiva SAS le permite especificar qué entidad externa se puede hacer con el recurso. La idea es que RabbitMQ puede enviar mensajes, pero no escuchar ni administrar la cola.

:::image type="content" source="./media/service-bus-integrate-with-rabbitmq/create-sas-policy.png" alt-text="Agregar directiva SAS":::

Marque el cuadro `Send` y haga clic en `Create` para tener la directiva de SAS establecida.

Una vez creada la Directiva, haga clic en ella para ver la **cadena de conexión principal**. Lo usaremos para permitir que RabbitMQ se comunique con Azure Service Bus:

:::image type="content" source="./media/service-bus-integrate-with-rabbitmq/sas-policy-key.png" alt-text="Obtener directiva SAS":::

Antes de poder utilizar esa cadena de conexión, tendrá que convertirla al formato de conexión AMQP de RabbitMQ. Así que, vaya a la [herramienta de conversión de cadena de conexión](https://red-mushroom-0f7446a0f.azurestaticapps.net/) y pegue la cadena de conexión en el formulario. Haga clic en Convertir. Obtendrá una cadena de conexión RabbitMQ lista. (Ese sitio web ejecuta todo lo que sea local en el explorador, por lo que los datos no se envían a través del cable). Puede acceder a su código fuente en [GitHub](https://github.com/videlalvaro/connstring_to_amqp).

:::image type="content" source="./media/service-bus-integrate-with-rabbitmq/converter.png" alt-text="Convertir cadena de conexión":::

Ahora, abra el complemento de administración de RabbitMQ en nuestros exploradores `http://localhost:15672/#/dynamic-shovels` y vaya a `Admin -> Shovel Management`, donde puede agregar el nuevo Shovel que se encargará de enviar mensajes desde una cola de RabbitMQ hasta la cola de Azure Service Bus.

:::image type="content" source="./media/service-bus-integrate-with-rabbitmq/add-shovel.png" alt-text="Agregar RabbitMQ Shovel":::

Aquí, llame a Shovel `azure` y elija `AMQP 0.9.1` como el protocolo de origen. En la captura de pantalla, tenemos `amqp://`, que es el URI predeterminado que nos conecta con un servidor de RabbitMQ local. Asegúrese de adaptarse a la implementación actual.

En el lado de la cola, puede usar `azure` como el nombre de la cola. Si la cola no existe, RabbitMQ la creará automáticamente. También puede elegir el nombre de una cola que ya existe. Puede dejar las otras opciones con el valor predeterminado.

A continuación, en el lado `destination`, elija `AMQP 1.0` como el protocolo. En el campo `URI`, escriba la cadena de conexión que recibió en el paso anterior, donde convirtió su cadena de conexión de Azure al formato RabbitMQ. Debería ser parecido a este:

```
amqps://rabbitmq-shovel:StringOfRandomChars@rabbitmq.servicebus.windows.net:5671/?sasl=plain
```

En el campo `Address` escribiremos el nombre de la **cola de Azure Service Bus**. En este caso, se llamaba `from-rabbitmq`. Haga clic en `Add Shovel` y el programa de instalación debería estar listo para comenzar a recibir mensajes.

## <a name="publishing-messages-from-rabbitmq-to-azure-service-bus"></a>Publicación de mensajes de RabbitMQ en Azure Service Bus

En la interfaz de administración de RabbitMQ, podemos ir a `Queues`, seleccionar la cola de `azure` y buscar el panel de `Publish message`. Aparecerá un formulario que le permitirá publicar mensajes directamente en la cola. En nuestro ejemplo, agregaremos `fist message` como `Payload` y seleccionaremos `Publish Message`:

:::image type="content" source="./media/service-bus-integrate-with-rabbitmq/first-message.png" alt-text="Publicar el primer mensaje":::

Vuelva a Azure e inspeccione la cola. Haga clic en `Service Bus Explorer`, en el panel izquierdo. Si todo ha ido bien, verá que la cola tiene ahora un mensaje. ¡Enhorabuena!

:::image type="content" source="./media/service-bus-integrate-with-rabbitmq/service-bus-queue.png" alt-text="Cola de Azure Service Bus":::

Pero vamos a asegurarnos de que el mensaje es el que envió desde RabbitMQ. Seleccione la pestaña `Peek` y haga clic en el botón `Peek` para recuperar los últimos mensajes de la cola. Haga clic en el mensaje para inspeccionar su contenido. Debería ver algo parecido a la imagen siguiente, donde aparece el `first message`.

:::image type="content" source="./media/service-bus-integrate-with-rabbitmq/peek.png" alt-text="Peek de cola":::

## <a name="lets-recap"></a>Recapitulemos

¡Enhorabuena! ¡Ha conseguido mucho! Logró enviar sus mensajes de RabbitMQ a Azure Service Bus, recapitulemos los pasos:

1. Crear un espacio de nombres de Azure Service Bus
2. Agregar una cola al espacio de nombres
3. Agregar una directiva SAS a la cola
4. Obtención de la cadena de conexión de la cola
5. Habilitar el complemento Shovel de RabbitMQ y la interfaz de administración
6. Convertir la cadena de conexión de Azure Service Bus al formato AMQP de RabbitMQ
7. Agregar una nuevo complemento Shovel a RabbitMQ y conectarlo a Azure Service Bus
8. Publicar mensajes

Al seguir los pasos anteriores, integró áreas de su organización que estaban fuera de Azure. El complemento Shovel le permitió enviar mensajes desde RabbitMQ a Azure Service Bus. Esto tiene enormes ventajas, ya que ahora puede permitir que terceros de confianza conecten sus aplicaciones con su implementación de Azure.

Al final, la mensajería consiste en habilitar las conexiones y, con esta técnica, se acaba de abrir una nueva.