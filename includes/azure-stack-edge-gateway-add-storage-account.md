---
author: alkohli
ms.service: databox
ms.topic: include
ms.date: 08/30/2020
ms.author: alkohli
ms.openlocfilehash: 30ca4d330d9b16214396ac81e5ab5722ca0e7569
ms.sourcegitcommit: bcda98171d6e81795e723e525f81e6235f044e52
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/01/2020
ms.locfileid: "89254308"
---
1. En [Azure Portal](https://portal.azure.com/), seleccione el recurso de Azure Stack Edge y, después, vaya a **Información general**. El dispositivo debe estar en línea.

2. Seleccione **+ Agregar cuenta de almacenamiento** en la barra de comandos del dispositivo. 

   ![Agregar una cuenta de almacenamiento](media/azure-stack-edge-gateway-add-storage-account/add-storage-account-1.png)

3. En el panel **Agregar cuenta de almacenamiento de Edge**, especifique la siguiente configuración:

    a. Un nombre único para la cuenta de almacenamiento de Edge en el dispositivo. Los nombres de las cuentas de almacenamiento solo pueden contener números y letras en minúsculas. No se aceptan caracteres especiales. El nombre de la cuenta de almacenamiento debe ser único en el dispositivo (no en todos los dispositivos).

    b. Una descripción opcional de la información sobre los datos que contiene la cuenta de almacenamiento.  
    
    c. De forma predeterminada, la cuenta de almacenamiento de Edge está asignada a una cuenta de Azure Storage en la nube y los datos de la cuenta de almacenamiento se insertan automáticamente en la nube. Especifique la cuenta de Azure Storage a la que está asignada la cuenta de almacenamiento de Edge.  

    d. A continuación, cree un nuevo contenedor o seleccione un contenedor existente en la cuenta de almacenamiento de Azure. Los datos del dispositivo que se escriben en la cuenta de almacenamiento de Edge se cargan automáticamente en el contenedor de almacenamiento seleccionado en la cuenta de Azure Storage asignada.

    <!--![Add a storage account](media/azure-stack-edge-gateway-add-storage-account/add-storage-account-2.png)-->

    e. Después de especificar todas las opciones de cuenta de almacenamiento, seleccione **Agregar** para crear la cuenta de almacenamiento de Edge. Recibirá una notificación cuando la cuenta de almacenamiento de Edge se cree correctamente. La nueva cuenta de almacenamiento de Edge se muestra en la lista de cuentas de almacenamiento en Azure Portal. 

    
4. Si selecciona esta nueva cuenta de almacenamiento y va a **Claves de acceso**, puede encontrar el punto de conexión de Blob Service y el nombre de la cuenta de almacenamiento correspondiente. Copie esta información, ya que estos valores junto con las claves de acceso le ayudarán a conectarse a la cuenta de almacenamiento de Edge.

    ![Agregar una cuenta de almacenamiento](media/azure-stack-edge-gateway-add-storage-account/add-storage-account-4.png)

    Puede obtener las claves de acceso mediante la [conexión a las API locales del dispositivo con Azure Resource Manager](../articles/databox-online/azure-stack-edge-j-series-connect-resource-manager.md). 
