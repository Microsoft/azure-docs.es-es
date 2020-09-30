---
title: 'Tutorial: Filtrado y análisis de datos para la implementación avanzada con el proceso en Azure Stack Edge Pro | Microsoft Docs'
description: Aprenda a configurar el rol de proceso en Azure Stack Edge Pro y a usarlo para transformar los datos para el flujo de implementación avanzada antes de enviarlos a Azure.
services: databox
author: alkohli
ms.service: databox
ms.subservice: edge
ms.topic: tutorial
ms.date: 05/20/2019
ms.author: alkohli
Customer intent: As an IT admin, I need to understand how to configure compute on Azure Stack Edge Pro for advanced deployment flow so I can use it to transform the data before sending it to Azure.
ms.openlocfilehash: f62eec29aebdcc98569134e0c3b75457467bc014
ms.sourcegitcommit: 53acd9895a4a395efa6d7cd41d7f78e392b9cfbe
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/22/2020
ms.locfileid: "90903693"
---
# <a name="tutorial-transform-data-with-azure-stack-edge-pro-for-advanced-deployment-flow"></a>Tutorial: Transformación de datos con Azure Stack Edge Pro para el flujo de implementación avanzada

Este tutorial describe cómo configurar el rol de proceso para un flujo de implementación avanzada en un dispositivo de Azure Stack Edge Pro. Una vez configurado el rol de proceso, Azure Stack Edge Pro puede transformar los datos antes de enviarlos a Azure.

El proceso puede configurarse para un flujo de implementación sencilla o avanzada en el dispositivo.

| Criterios | Implementación sencilla                                | Implementación avanzada                   |
|------------------|--------------------------------------------------|---------------------------------------|
| Destinado a     | Administradores de TI                                | Desarrolladores                            |
| Tipo             | Usar el servicio Azure Stack Edge para implementar módulos      | Usar el servicio IoT Hub para implementar módulos |
| Módulos implementados | Single                                           | Varios módulos o módulos en cadena           |


Este procedimiento tarda aproximadamente entre 20 y 30 minutos en completarse.

En este tutorial, aprenderá a:

> [!div class="checklist"]
> * Configurar el proceso
> * Agregar recursos compartidos
> * Incorporación de un desencadenador
> * Agregar un módulo de proceso
> * Comprobar la transformación y la transferencia de los datos

 
## <a name="prerequisites"></a>Requisitos previos

Antes de configurar un rol de proceso en el dispositivo de Azure Stack Edge Pro, asegúrese de que:

- Ha activado el dispositivo físico de Azure Stack Edge Pro tal como se describe en [Conexión, configuración y activación de Azure Stack Edge Pro](azure-stack-edge-deploy-connect-setup-activate.md).


## <a name="configure-compute"></a>Configurar el proceso

Para configurar el proceso en Azure Stack Edge Pro, creará un recurso de IoT Hub.

1. En la instancia de Azure Portal del recurso de Azure Stack Edge, vaya a **Información general**. En el panel derecho, en el icono **Proceso**, seleccione **Comenzar**.

    ![Introducción al proceso](./media/azure-stack-edge-deploy-configure-compute-advanced/configure-compute-1.png)

2. En el icono **Configurar el proceso de Edge**, seleccione **Configurar proceso**.

    ![Introducción al proceso](./media/azure-stack-edge-deploy-configure-compute-advanced/configure-compute-2.png)

3. En la hoja **Configurar el proceso de Edge**, escriba lo siguiente:

   
    |Campo  |Value  |
    |---------|---------|
    |IoT Hub     | Elija entre **Nuevo** o **Existente**. <br> De forma predeterminada, se usa un nivel estándar (S1) para crear un recurso de IoT. Para usar un recurso de IoT de nivel gratuito, cree uno y, a continuación, seleccione el recurso existente. <br> En cualquier caso, el recurso de IoT Hub usa la misma suscripción y el mismo grupo de recursos que el recurso de Azure Stack Edge.     |
    |Nombre     |Escriba un nombre para el recurso de IoT Hub.         |

    ![Introducción al proceso](./media/azure-stack-edge-deploy-configure-compute-advanced/configure-compute-3.png)

4. Seleccione **Crear**. La creación del recurso de IoT Hub tarda unos minutos. Después de crear el recurso de IoT Hub, el icono **Configurar el proceso perimetral** se actualiza para mostrar la configuración de proceso. Para confirmar que se ha configurado el rol de proceso perimetral, seleccione **Configuración de vista** en el icono **Configurar proceso**.
    
    ![Introducción al proceso](./media/azure-stack-edge-deploy-configure-compute-advanced/configure-compute-4.png)

    Cuando el rol de proceso de Edge está configurado en el dispositivo de Edge, este crea dos dispositivos: uno IoT y el otro IoT Edge. Ambos se pueden ver en el recurso de IoT Hub. Un entorno de ejecución de IoT Edge también se ejecuta en este dispositivo de IoT Edge.

    En este momento, solo está disponible la plataforma Linux para el dispositivo IoT Edge.


## <a name="add-shares"></a>Agregar recursos compartidos

En este tutorial, se realizará una implementación avanzada con dos recursos compartidos: un recurso compartido perimetral y otro recurso compartido local perimetral.

1. Para agregar un recurso compartido perimetral al dispositivo, siga estos pasos:

    1. En el recurso de Azure Stack Edge, vaya a **Proceso perimetral > Comenzar**.
    2. En el icono **Add share(s)** (Agregar recursos compartidos), seleccione **Agregar**.
    3. En la hoja **Agregar recurso compartido**, proporcione el nombre del recurso compartido y seleccione el tipo de recurso compartido.
    4. Para montar el recurso compartido perimetral, active la casilla **Usar el recurso compartido con el proceso perimetral**.
    5. Seleccione la **cuenta de almacenamiento**, el **servicio de almacenamiento**, un usuario existente y, luego, **Crear**.

        ![Agregar un recurso compartido perimetral](./media/azure-stack-edge-deploy-configure-compute-advanced/add-edge-share-1.png)

    <!--If you created a local NFS share, use the following remote sync (rsync) command option to copy files onto the share:

    `rsync <source file path> < destination file path>`

    For more information about the rsync command, go to [Rsync documentation](https://www.computerhope.com/unix/rsync.htm).-->

    Una vez creado el recurso compartido perimetral, recibirá una notificación para indicar que la creación se realizó correctamente. La lista de recursos compartidos se actualiza para reflejar el nuevo recurso compartido.

2. Para agregar un recurso compartido local perimetral al dispositivo perimetral, repita todos los pasos anteriores y active la casilla **Configurar como recurso compartido local de Edge**. Los datos en el recurso compartido local permanecen en el dispositivo.

    ![Agregar un recurso compartido local perimetral](./media/azure-stack-edge-deploy-configure-compute-advanced/add-edge-share-2.png)

3. En la hoja **Recursos compartidos**, consulte la lista actualizada de los recursos compartidos.

    ![Lista de recursos compartidos actualizada](./media/azure-stack-edge-deploy-configure-compute-advanced/add-edge-share-3.png)

4. Para ver las propiedades del recurso compartido local recién creado, selecciónelo en la lista. En el cuadro **Punto de montaje local para módulos de proceso de Edge**, copie el valor correspondiente a este recurso compartido.

    Usará este punto de montaje local al implementar el módulo.

    ![El cuadro "Punto de montaje local para módulos del proceso de Edge"](./media/azure-stack-edge-deploy-configure-compute-advanced/add-edge-share-4.png)
 
5. Para ver las propiedades del recurso compartido perimetral creado, selecciónelo en la lista. En el cuadro **Punto de montaje local para módulos de proceso de Edge**, copie el valor correspondiente a este recurso compartido.

    Usará este punto de montaje local al implementar el módulo.

    ![Agregar un módulo personalizado](./media/azure-stack-edge-deploy-configure-compute-advanced/add-edge-share-5.png)


## <a name="add-a-trigger"></a>Incorporación de un desencadenador

1. Vaya a **Proceso perimetral > Desencadenadores**. Seleccione **+ Agregar desencadenador**.

    ![Incorporación de un desencadenador](./media/azure-stack-edge-deploy-configure-compute-advanced/add-trigger-1.png)

2. En la hoja **Agregar desencadenador**, introduzca los siguientes valores.

    |Campo  |Value  |
    |---------|---------|
    |Nombre del desencadenador     | Un nombre exclusivo para el concentrador.         |
    |Tipo de desencadenador     | Seleccione el desencadenador **Archivo**. Un desencadenador de archivo se activa cada vez que se produce un evento de archivo, por ejemplo, cuando se escribe un archivo en el recurso compartido de entrada. Por otra parte, un desencadenador programado se activa según una programación definida por el usuario. En este ejemplo, necesitamos un desencadenador de archivo.    |
    |Recurso compartido de entrada     | Seleccione un recurso compartido de entrada. En este caso, el recurso compartido local perimetral es el recurso compartido de entrada. El módulo que se usa aquí mueve los archivos desde el recurso compartido local perimetral hasta un recurso compartido perimetral donde se cargan en la nube.        |

    ![Incorporación de un desencadenador](./media/azure-stack-edge-deploy-configure-compute-advanced/add-trigger-2.png)

3. Recibirá una notificación cuando el desencadenador se cree. La lista de desencadenadores se actualiza para mostrar el desencadenador recién creado. Seleccione el desencadenador que acaba de crear.

    ![Incorporación de un desencadenador](./media/azure-stack-edge-deploy-configure-compute-advanced/add-trigger-3.png)

4. Copie y guarde la ruta de ejemplo. Modificará esta ruta de ejemplo y la usará más adelante en IoT Hub.

    `"sampleroute": "FROM /* WHERE topic = 'mydbesmbedgelocalshare1' INTO BrokeredEndpoint(\"/modules/modulename/inputs/input1\")"`

    ![Incorporación de un desencadenador](./media/azure-stack-edge-deploy-configure-compute-advanced/add-trigger-4.png)

## <a name="add-a-module"></a>Agregar un módulo

No hay módulos personalizados en este dispositivo perimetral. Puede agregar un módulo personalizado o uno creado previamente. Para aprender a crear un módulo personalizado, vaya a [Desarrollo de un módulo C# para un dispositivo de Azure Stack Edge Pro](azure-stack-edge-create-iot-edge-module.md).

En esta sección, agregará un módulo personalizado al dispositivo de IoT Edge que creó en [Desarrollo de un módulo C# para Azure Stack Edge Pro](azure-stack-edge-create-iot-edge-module.md). Este módulo personalizado toma los archivos de un recurso compartido local perimetral del dispositivo perimetral y los mueve a un recurso compartido perimetral (nube) del dispositivo. A continuación, el recurso compartido en la nube inserta los archivos en la cuenta de Azure Storage asociada con este.

1. Vaya a **Proceso perimetral > Comenzar**. En el icono **Agregar módulos**, seleccione el tipo de escenario como **avanzado**. Seleccione **Ir a IoT Hub**.

    ![Selección de la implementación avanzada](./media/azure-stack-edge-deploy-configure-compute-advanced/add-module-1.png)

<!--2. In the **Configure and add module** blade, input the following values:  

    |Input share     | Select an input share. The Edge local share is the input share in this case. The module used here moves files from the Edge local share to an Edge share where they are uploaded into the cloud.        |
    |Output share     | Select an output share. The Edge share is the output share in this case.        |
-->

2. En el recurso de IoT Hub, vaya a **Dispositivo IoT Edge**  y, a continuación, seleccione el dispositivo correspondiente.

    ![Ir al dispositivo IoT Edge en IoT Hub](./media/azure-stack-edge-deploy-configure-compute-advanced/add-module-2.png)

3. En **Detalles del dispositivo**, seleccione **Establecer módulos**.

    ![El vínculo Establecer módulos](./media/azure-stack-edge-deploy-configure-compute-advanced/add-module-3.png)

4. En **Agregar módulos**, haga lo siguiente:

    1. Proporcione el nombre, dirección, nombre de usuario y contraseña de la configuración del Registro del contenedor para el módulo personalizado.
    El nombre, la dirección y las credenciales mostradas se utilizan para recuperar aquellos módulos con una dirección URL que coincida. Para implementar este módulo, en **Módulos de implementación**, seleccione **Módulo de IoT Edge**. Este módulo de IoT Edge es un contenedor de Docker que puede implementar en un dispositivo de IoT Edge asociado con el dispositivo de Azure Stack Edge Pro.

        ![La página Establecer módulos](./media/azure-stack-edge-deploy-configure-compute-advanced/add-module-4.png) 
 
    2. Especifique la configuración para el módulo personalizado de IoT Edge. Escriba los siguientes valores.
     
        |Campo  |Value  |
        |---------|---------|
        |Nombre     | Un nombre único para el módulo. Este módulo es un contenedor de Docker que puede implementar en el dispositivo de IoT Edge asociado a Azure Stack Edge Pro.        |
        |URI de imagen     | El URI de la imagen de contenedor correspondiente del módulo.        |
        |Credenciales necesarias     | Si está activada, se usa el nombre de usuario y la contraseña para recuperar los módulos con una dirección URL coincidente.        |
    
        En el cuadro **Opciones de creación del contenedor**, escriba los puntos de montaje local para los módulos de Edge que ha copiado en los pasos anteriores para el recurso compartido perimetral y el recurso compartido local perimetral.

        > [!IMPORTANT]
        > Las rutas de acceso que se usan aquí se montan en el contenedor, por lo que deben coincidir con la funcionalidad esperada en el contenedor. Si sigue [Creación de un módulo personalizado](azure-stack-edge-create-iot-edge-module.md#update-the-module-with-custom-code), el código especificado en dicho módulo espera las rutas de acceso copiadas. No modifique estas rutas de acceso.
    
        En el cuadro **Opciones de creación del contenedor**, puede pegar el ejemplo siguiente:
    
        ```
        {
          "HostConfig": 
          {
           "Binds": 
            [
             "/home/hcsshares/mydbesmbedgelocalshare1:/home/input",
             "/home/hcsshares/mydbesmbedgeshare1:/home/output"
            ]
           }
        }
        ```

        Proporcione todas las variables de entorno utilizadas para el módulo. Las variables de entorno ofrecen información opcional que le ayudará a definir el entorno en que se ejecuta su módulo.

        ![El cuadro Opciones de creación del contenedor](./media/azure-stack-edge-deploy-configure-compute-advanced/add-module-5.png) 
 
    4. Si es necesario, configure las opciones avanzadas del entorno de ejecución de Edge y, a continuación, haga clic en **Siguiente**.

        ![Agregar un módulo personalizado](./media/azure-stack-edge-deploy-configure-compute-advanced/add-module-6.png)
 
5. En **Especificar rutas**, defina las rutas entre módulos.  
   
   ![Especificación de rutas](./media/azure-stack-edge-deploy-configure-compute-advanced/add-module-7.png)

    Puede reemplazar la *ruta* por la cadena de ruta siguiente copiada anteriormente. En este ejemplo, proporcione el nombre del recurso compartido local que insertará los datos en el recurso compartido en la nube. Reemplace `modulename` por el nombre del módulo. Seleccione **Next** (Siguiente).
        
    ```
    "route": "FROM /* WHERE topic = 'mydbesmbedgelocalshare1' INTO BrokeredEndpoint(\"/modules/filemove/inputs/input1\")"
    ```

    ![La sección "Especificar rutas"](./media/azure-stack-edge-deploy-configure-compute-advanced/add-module-8.png)

6. En **Revisar la implementación**, revise todos los valores y, después, seleccione **Enviar** a fin de enviar el módulo para la implementación.

   ![La página Establecer módulos](./media/azure-stack-edge-deploy-configure-compute-advanced/add-module-9.png)
 
    Esta acción inicia la implementación del módulo. Una vez completada la implementación, el **Estado en tiempo de ejecución** del módulo es **En ejecución**.

    ![Agregar un módulo personalizado](./media/azure-stack-edge-deploy-configure-compute-advanced/add-module-10.png)

## <a name="verify-data-transform-transfer"></a>Comprobar la transformación y la transferencia de los datos

El último paso es asegurarse de que el módulo está conectado y funciona según lo previsto. El estado del entorno de ejecución del módulo debe ser "en ejecución" para el dispositivo IoT Edge del recurso de IoT Hub.

Realice los pasos siguientes para verificar la transformación de los datos y la transferencia a Azure.
 
1. En el Explorador de archivos, conéctese a los recursos compartidos local perimetral y perimetral que creó anteriormente.

   ![Comprobar transformación de datos](./media/azure-stack-edge-deploy-configure-compute-advanced/verify-data-2.png)
 
1. Agregue datos al recurso compartido local.

   ![Comprobar transformación de datos](./media/azure-stack-edge-deploy-configure-compute-advanced/verify-data-3.png)
 
    Los datos se mueven al recurso compartido en la nube.

    ![Comprobar transformación de datos](./media/azure-stack-edge-deploy-configure-compute-advanced/verify-data-4.png)  

    A continuación, los datos se trasladan del recurso compartido en la nube a la cuenta de almacenamiento. Para ver los datos, vaya a la cuenta de almacenamiento y, a continuación, seleccione **Explorador de Storage**. Puede ver los datos cargados en la cuenta de almacenamiento.

    ![Comprobar transformación de datos](./media/azure-stack-edge-deploy-configure-compute-advanced/verify-data-5.png)
 
Ha completado el proceso de validación.

## <a name="next-steps"></a>Pasos siguientes

En este tutorial, ha aprendido a:

> [!div class="checklist"]
> * Configurar el proceso
> * Agregar recursos compartidos
> * Incorporación de un desencadenador
> * Agregar un módulo de proceso
> * Comprobar la transformación y la transferencia de los datos

Para aprender a administrar el dispositivo de Azure Stack Edge Pro, consulte:

> [!div class="nextstepaction"]
> [Uso de la interfaz de usuario web local para administrar Azure Stack Edge Pro](azure-stack-edge-manage-access-power-connectivity-mode.md).
