---
title: 'Inicio rápido: Uso de la CLI de Azure para crear una cola de Service Bus | Microsoft Docs'
description: En este inicio rápido aprenderá, en primer lugar, a utilizar la CLI de Azure para crear un espacio de nombres de Service Bus y, posteriormente, una cola en ese mismo espacio.
author: spelluru
ms.topic: quickstart
ms.date: 08/12/2020
ms.author: spelluru
ms.openlocfilehash: 201ea38c6feabbda2576d8480a9983f00d62d175
ms.sourcegitcommit: 9ce0350a74a3d32f4a9459b414616ca1401b415a
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/13/2020
ms.locfileid: "88191272"
---
# <a name="use-the-azure-cli-to-create-a-service-bus-namespace-and-a-queue"></a>Uso de la CLI de Azure para crear un espacio de nombres de Service Bus y una cola
En este inicio rápido se muestra cómo crear un espacio de nombres de Service Bus y una cola mediante la CLI de Azure. También se muestra cómo obtener credenciales de autorización que una aplicación cliente puede usar para enviar mensajes a la cola, o recibirlos de ella. 

[!INCLUDE [howto-service-bus-queues](../../includes/howto-service-bus-queues.md)]

## <a name="prerequisites"></a>Requisitos previos
Si no tiene una suscripción a Azure, puede crear una [cuenta gratuita][free account] antes de empezar.

En este inicio rápido, usará Azure Cloud Shell, que puede iniciar después de iniciar sesión en Azure Portal. Para más información sobre Azure Cloud Shell, consulte [Introducción a Azure Cloud Shell](../cloud-shell/overview.md). También puede [instalar](/cli/azure/install-azure-cli) y usar Azure PowerShell en la máquina. 

## <a name="provision-resources"></a>Aprovisionamiento de recursos
1. Inicie sesión en el [Portal de Azure](https://portal.azure.com).
2. Para iniciar Azure Cloud Shell, seleccione el icono que se muestra en la siguiente imagen. Cambie al modo **Bash** si Cloud Shell está en modo **PowerShell**. 

    :::image type="content" source="./media/service-bus-quickstart-powershell/launch-cloud-shell.png" alt-text="Iniciar Cloud Shell":::
3. Ejecute el siguiente comando para crear un grupo de recursos de Azure. Si lo desea, actualice el nombre del grupo de recursos y la ubicación. 

    ```azurecli-interactive
    az group create --name ContosoRG --location eastus
    ```
4. Ejecute el comando siguiente para crear un espacio de nombres de mensajería de Service Bus.

    ```azurecli-interactive
    az servicebus namespace create --resource-group ContosoRG --name ContosoSBusNS --location eastus
    ```
5. Ejecute el siguiente comando para crear una cola en el espacio de nombres que creó en el paso anterior. En este ejemplo, `ContosoRG` es el grupo de recursos que creó en el paso anterior. `ContosoSBusNS` es el nombre del espacio de nombres de Service Bus creado en ese grupo de recursos. 

    ```azurecli-interactive
    az servicebus queue create --resource-group ContosoRG --namespace-name ContosoSBusNS --name ContosoOrdersQueue
    ```
6. Ejecute el siguiente comando para obtener la cadena de conexión principal del espacio de nombres. Esta cadena de conexión se usará para conectarse a la cola y enviar y recibir mensajes. 

    ```azurecli-interactive
    az servicebus namespace authorization-rule keys list --resource-group ContosoRG --namespace-name ContosoSBusNS --name RootManageSharedAccessKey --query primaryConnectionString --output tsv    
    ```

    Anote la cadena de conexión y el nombre de la cola. Se usan para enviar y recibir mensajes. 


## <a name="next-steps"></a>Pasos siguientes
En este artículo, ha creado un espacio de nombres de Service Bus y una cola en dicho espacio de nombres. Para aprender a enviar mensajes a la cola y recibirlos de ella, consulte uno de los siguientes inicios rápido en la sección **Envío y recepción de mensajes**. 

- [.NET](service-bus-dotnet-get-started-with-queues.md)
- [Java](service-bus-java-how-to-use-queues.md)
- [JavaScript](service-bus-nodejs-how-to-use-queues-new-package.md)
- [Python](service-bus-python-how-to-use-queues.md)
- [PHP](service-bus-php-how-to-use-queues.md)
- [Ruby](service-bus-ruby-how-to-use-queues.md)

[free account]: https://azure.microsoft.com/free/?ref=microsoft.com&utm_source=microsoft.com&utm_medium=docs&utm_campaign=visualstudio

