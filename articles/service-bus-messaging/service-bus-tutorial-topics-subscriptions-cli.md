---
title: Uso de la CLI de Azure para crear temas y suscripciones de Service Bus
description: En este inicio rápido, aprenderá a crear un tema de Service Bus y suscripciones a dicho tema mediante la CLI de Azure.
ms.date: 06/23/2020
ms.topic: quickstart
author: spelluru
ms.author: spelluru
ms.openlocfilehash: 3a6535a13ab00c4e22ac4cd8c2de5a5bbb02d0a8
ms.sourcegitcommit: 9ce0350a74a3d32f4a9459b414616ca1401b415a
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/13/2020
ms.locfileid: "88189810"
---
# <a name="use-azure-cli-to-create-a-service-bus-topic-and-subscriptions-to-the-topic"></a>Uso de la CLI de Azure para crear un tema de Service Bus y suscripciones a dicho tema
En este inicio rápido, usará la CLI de Azure para crear un tema de Service Bus y, después, crear suscripciones a dicho tema. 

## <a name="what-are-service-bus-topics-and-subscriptions"></a>Qué son los temas y las suscripciones de Service Bus
Las suscripciones y los temas de Service Bus son compatibles con el modelo de comunicación de mensajería de *publicación/suscripción* . Cuando se usan temas y suscripciones, los componentes de una aplicación distribuida no se comunican directamente entre sí, sino que intercambian mensajes a través de un tema, que actúa como un intermediario.

![TopicConcepts](./media/service-bus-java-how-to-use-topics-subscriptions/sb-topics-01.png)

A diferencia de las colas de Service Bus, en las que un solo destinatario procesa cada mensaje, los temas y las suscripciones proporcionan una forma de comunicación de uno a varios mediante un patrón de publicación/suscripción. Es posible registrar varias suscripciones en un tema. Cuando un mensaje se envía a un tema, pasa a estar disponible para cada suscripción para la administración o el procesamiento de manera independiente. Una suscripción a un tema se asemeja a una cola virtual que recibe copias de los mensajes que se enviaron al tema. Opcionalmente, puede registrar reglas de filtros para un tema por suscripción, lo que le permite filtrar o restringir qué mensajes para un tema reciben las suscripciones a un tema.

Las suscripciones y temas de Service Bus le permiten escalar para realizar el procesamiento de un número elevado de mensajes en una serie amplia de usuarios y aplicaciones.

## <a name="prerequisites"></a>Requisitos previos
Si no tiene una suscripción a Azure, puede crear una [cuenta gratuita][free account] antes de empezar.

En este inicio rápido, usará Azure Cloud Shell, que puede iniciar después de iniciar sesión en Azure Portal. Para más información sobre Azure Cloud Shell, consulte [Introducción a Azure Cloud Shell](../cloud-shell/overview.md). También puede [instalar](/cli/azure/install-azure-cli) y usar Azure PowerShell en la máquina. 

## <a name="create-a-service-bus-topic-and-subscriptions"></a>Creación de un tema y suscripciones de Service Bus
Cada [suscripción a un tema](service-bus-messaging-overview.md#topics) puede recibir una copia de cada mensaje. Los temas son completamente compatibles desde el punto de vista semántico y del protocolo con las colas de Service Bus. Los temas de Service Bus admiten una amplia variedad de reglas de selección con condiciones de filtro, con acciones opcionales que establecerán o modificarán las propiedades del mensaje. Cada vez que coincide una regla, esta genera un mensaje. Para más información acerca de reglas, filtros y acciones, siga este [vínculo](topic-filters.md).

1. Inicie sesión en el [Portal de Azure](https://portal.azure.com).
2. Para iniciar Azure Cloud Shell, seleccione el icono que se muestra en la siguiente imagen. Cambie al modo **Bash** si Cloud Shell está en modo **PowerShell**. 

    :::image type="content" source="./media/service-bus-quickstart-powershell/launch-cloud-shell.png" alt-text="Iniciar Cloud Shell":::
3. Ejecute el siguiente comando para crear un grupo de recursos de Azure. Actualice el nombre del grupo de recursos y la ubicación si lo desea. 

    ```azurecli-interactive
    az group create --name MyResourceGroup --location eastus
    ```
4. Ejecute el comando siguiente para crear un espacio de nombres de mensajería de Service Bus. Actualice el nombre del espacio de nombres para que sea único. 

    ```azurecli-interactive
    namespaceName=MyNameSpace$RANDOM
    az servicebus namespace create --resource-group MyResourceGroup --name $namespaceName --location eastus
    ```
5. Ejecute el siguiente comando para crear un tema en el espacio de nombres. 

    ```azurecli-interactive
    az servicebus topic create --resource-group MyResourceGroup   --namespace-name $namespaceName --name MyTopic
    ```
6. Cree la primera suscripción al tema.
    
    ```azurecli-interactive
    az servicebus topic subscription create --resource-group MyResourceGroup --namespace-name $namespaceName --topic-name MyTopic --name S1    
    ```
6. Cree la segunda suscripción al tema.
    
    ```azurecli-interactive
    az servicebus topic subscription create --resource-group MyResourceGroup --namespace-name $namespaceName --topic-name MyTopic --name S2    
    ```
6. Cree la tercera suscripción al tema.
    
    ```azurecli-interactive
    az servicebus topic subscription create --resource-group MyResourceGroup --namespace-name $namespaceName --topic-name MyTopic --name S3    
    ```
7. Filtre la primera suscripción usando propiedades personalizadas (`StoreId` es `Store1`, `Store2` o `Store3`).

    ```azurecli-interactive
    az servicebus topic subscription rule create --resource-group MyResourceGroup --namespace-name $namespaceName --topic-name MyTopic --subscription-name S1 --name MyFilter --filter-sql-expression "StoreId IN ('Store1','Store2','Store3')"    
    ```
8. Filtre la segunda suscripción usando propiedades del cliente (`StoreId = Store4`).

    ```azurecli-interactive
    az servicebus topic subscription rule create --resource-group MyResourceGroup --namespace-name $namespaceName --topic-name myTopic --subscription-name S2 --name MySecondFilter --filter-sql-expression "StoreId = 'Store4'"    
    ```
9. Filtre la tercera suscripción usando propiedades del cliente (`StoreId` no está en `Store1`, `Store2`, `Store3` o `Store4`).

    ```azurecli-interactive
    az servicebus topic subscription rule create --resource-group MyResourceGroup --namespace-name $namespaceName --topic-name MyTopic --subscription-name S3 --name MyThirdFilter --filter-sql-expression "StoreId IN ('Store1','Store2','Store3', 'Store4')"     
    ```
10. Ejecute el siguiente comando para obtener la cadena de conexión principal del espacio de nombres. Esta cadena de conexión se usará para conectarse a la cola y enviar y recibir mensajes. 

    ```azurecli-interactive
    az servicebus namespace authorization-rule keys list --resource-group MyResourceGroup --namespace-name $namespaceName --name RootManageSharedAccessKey --query primaryConnectionString --output tsv    
    ```

    Anote la cadena de conexión y el nombre del tema. Se usan para enviar y recibir mensajes. 
    

## <a name="next-steps"></a>Pasos siguientes
Para obtener información acerca de cómo enviar mensajes a un tema y recibirlos a través de una suscripción, consulte el artículo siguiente: seleccione el lenguaje de programación en la tabla de contenido. 

> [!div class="nextstepaction"]
> [Publicación y suscripción de mensajes](service-bus-dotnet-how-to-use-topics-subscriptions.md)


[free account]: https://azure.microsoft.com/free/?ref=microsoft.com&utm_source=microsoft.com&utm_medium=docs&utm_campaign=visualstudio
[fully qualified domain name]: https://wikipedia.org/wiki/Fully_qualified_domain_name
[Install the Azure CLI]: /cli/azure/install-azure-cli
[az group create]: /cli/azure/group#az_group_create