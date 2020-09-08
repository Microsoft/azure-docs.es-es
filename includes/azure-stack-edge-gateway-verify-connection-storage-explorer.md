---
author: alkohli
ms.service: databox
ms.topic: include
ms.date: 12/02/2019
ms.author: alkohli
ms.openlocfilehash: 3474a97f4b05fd1f944e18abebcef927232453e4
ms.sourcegitcommit: 656c0c38cf550327a9ee10cc936029378bc7b5a2
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/28/2020
ms.locfileid: "89081056"
---
Si es la primera vez que usa el Explorador de Storage, deberá realizar los pasos siguientes.

1. En la barra de comandos superior, vaya a **Editar > API de Azure Stack de destino**.

    ![Configuración del Explorador de Storage](media/azure-stack-edge-gateway-verify-connection-storage-explorer/connect-with-storage-explorer-1.png)

2. Reinicie el Explorador de Storage para que los cambios surtan efecto.


Siga estos pasos para conectarse a la cuenta de almacenamiento y comprobar la conexión.

1. En el Explorador de Azure Storage, seleccione Cuentas de almacenamiento. Haga clic con el botón derecho y seleccione la opción **Conectar a Azure Storage**. 

    ![Configuración del Explorador de Storage](media/azure-stack-edge-gateway-verify-connection-storage-explorer/connect-with-storage-explorer-2.png)

2. En el cuadro de diálogo **Connect to Azure Storage** (Conectar con Azure Storage), seleccione **Use a storage account name and key** (Usar un nombre y una clave de cuenta de almacenamiento).

    ![Configuración del Explorador de Storage](media/azure-stack-edge-gateway-verify-connection-storage-explorer/connect-with-storage-explorer-3.png)

2. En el cuadro de diálogo **Conectarse con el nombre y la clave**, realice los pasos siguientes:

    1. Escriba un nombre para mostrar para la cuenta de almacenamiento de Edge. 
    2. Proporcione el nombre de la cuenta de almacenamiento de Edge.
    3. Pegue la clave de acceso que recibió de las API locales del dispositivo mediante Azure Resource Manager.
    4. Seleccione el dominio de Storage como **Other (enter below)** [Otros (indicar a continuación)] y, después, proporcione el sufijo del punto de conexión de Blob service en este formato: `<appliance name>.<DNSdomain>`. 
    5. Active la opción **Usar HTTP** cuando la transferencia se realice a través de *http*. 
    6. Seleccione **Next** (Siguiente).

    ![Configuración del Explorador de Storage](media/azure-stack-edge-gateway-verify-connection-storage-explorer/connect-with-storage-explorer-4.png)    

3. En el cuadro de diálogo **Connection Summary** (Resumen de conexiones), compruebe la información proporcionada. Seleccione **Conectar**.

    ![Configuración del Explorador de Storage](media/azure-stack-edge-gateway-verify-connection-storage-explorer/connect-with-storage-explorer-5.png)

4. La cuenta que ha agregado correctamente se muestra en el panel izquierdo del Explorador de Storage, con (External, Other) anexado al nombre. Seleccione **Contenedores de blobs** para ver el contenedor.

    ![Visualización de contenedores de blobs](media/azure-stack-edge-gateway-verify-connection-storage-explorer/connect-with-storage-explorer-6.png)

El siguiente paso consiste en comprobar que la transferencia de datos realmente funciona correctamente a través de esta conexión.

Siga los pasos que se indican a continuación para cargar datos en la cuenta de almacenamiento de Edge del dispositivo y, automáticamente, estos se deben poner en capas en la cuenta de Azure Storage asignada.

1. Seleccione el contenedor en el que quiere cargar los datos de la cuenta de almacenamiento de Edge. Seleccione **Cargar** y, después, seleccione **Cargar archivos**.

    ![Comprobación de la transferencia de datos](media/azure-stack-edge-gateway-verify-connection-storage-explorer/verify-data-transfer-1.png)

2. En el cuadro de diálogo **Cargar archivos**, vaya a los archivos que quiere cargar y selecciónelos. Seleccione **Next** (Siguiente).

    ![Comprobación de la transferencia de datos](media/azure-stack-edge-gateway-verify-connection-storage-explorer/verify-data-transfer-2.png)

3. Compruebe que se han cargado los archivos. Los archivos cargados se muestran en el contenedor.

    ![Comprobación de la transferencia de datos](media/azure-stack-edge-gateway-verify-connection-storage-explorer/verify-data-transfer-3.png)

4. A continuación, se conectará a la cuenta de Azure Storage que se asignó a esta cuenta de almacenamiento de Edge. Los datos que se cargan en la cuenta de almacenamiento de Edge deben organizarse por niveles automáticamente en la cuenta de Azure Storage. 
    
    Para obtener la cadena de conexión de la cuenta de Azure Storage, vaya a **Cuenta de Azure Storage > Claves de acceso** y copie la cadena de conexión.

    ![Comprobación de la transferencia de datos](media/azure-stack-edge-gateway-verify-connection-storage-explorer/verify-data-transfer-5.png)

    Use la cadena de conexión para conectarse a la cuenta de Azure Storage.  

    ![Comprobación de la transferencia de datos](media/azure-stack-edge-gateway-verify-connection-storage-explorer/verify-data-transfer-4.png)


5. En el cuadro de diálogo **Connection Summary** (Resumen de conexiones), compruebe la información proporcionada. Seleccione **Conectar**.

    ![Comprobación de la transferencia de datos](media/azure-stack-edge-gateway-verify-connection-storage-explorer/verify-data-transfer-6.png)

6. Verá que los archivos que ha cargado en la cuenta de almacenamiento de Edge se han transferido a la cuenta de Azure Storage.

    ![Comprobación de la transferencia de datos](media/azure-stack-edge-gateway-verify-connection-storage-explorer/verify-data-transfer-7.png)