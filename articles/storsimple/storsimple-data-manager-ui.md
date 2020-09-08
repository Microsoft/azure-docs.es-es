---
title: Interfaz de usuario de Microsoft Azure StorSimple Data Manager
description: Aprenda a usar la interfaz de usuario de StorSimple Data Manager para transformar los datos que residen en los dispositivos de la serie 8000 de StorSimple.
author: alkohli
ms.service: storsimple
ms.topic: conceptual
ms.date: 01/16/2018
ms.author: alkohli
ms.openlocfilehash: 5d6273cbef1664984d466fb15c7f02ca505b18e2
ms.sourcegitcommit: faeabfc2fffc33be7de6e1e93271ae214099517f
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/13/2020
ms.locfileid: "88184554"
---
# <a name="manage-the-storsimple-data-manager-service-in-azure-portal"></a>Administración del servicio StorSimple Data Manager en Azure Portal

En este artículo se explica cómo se puede usar la interfaz de usuario de StorSimple Data Manager para transformar los datos que residen en los dispositivos de la serie StorSimple 8000. Posteriormente, los datos transformados pueden consumirlos otros servicios de Azure, como Azure Media Services, Azure HDInsight, Azure Machine Learning y Azure Cognitive Search.


## <a name="use-storsimple-data-transformation"></a>Uso de transformación de datos de StorSimple

StorSimple Data Manager es el recurso en el que se crean instancias de transformación de datos. El servicio Transformación de datos permite transformar los datos desde el formato de StorSimple al formato nativo de blobs de Azure o Azure Files. Para transformar los datos del formato nativo de StorSimple, debe especificar los detalles del dispositivo de la serie StorSimple 8000 y los datos de interés que desee transformar.

### <a name="create-a-storsimple-data-manager-service"></a>Creación de un servicio StorSimple Data Manager

Siga estos pasos para crear un servicio StorSimple Data Manager.

1. Use las credenciales de su cuenta Microsoft para iniciar sesión en [Azure Portal](https://portal.azure.com/).

2. Haga clic en **+ Crear un recurso** y busque StorSimple Data Manager.

    ![Creación de un servicio StorSimple Data Manager 1](./media/storsimple-data-manager-ui/create-service-1.png)

3. Haga clic en StorSimple Data Manager y, después, en **Crear**.
    
    ![Creación de un servicio StorSimple Data Manager 2](./media/storsimple-data-manager-ui/create-service-3.png)

3. Para el nuevo servicio, especifique lo siguiente:

   1. Proporcione un **Service name** (Nombre de servicio) único para StorSimple Data Manager. Se trata de un nombre descriptivo que sirve para identificar el servicio. El nombre puede tener entre 3 y 24 caracteres que pueden ser letras, números y guiones. El nombre debe empezar y terminar con una letra o un número.

   2. Elija una **Subscription** (Suscripción) en la lista desplegable. La suscripción está vinculada a la cuenta de facturación. Este campo se rellena automáticamente (y no se puede seleccionar) si tiene una sola suscripción.

   3. Seleccione un grupo de recursos existente o cree uno nuevo. Para más información, consulte [Grupos de recursos en Azure](/azure/azure-resource-manager/management/manage-resource-groups-portal).

   4. Especifique el valor de **Location** (Ubicación) para el servicio que aloje las cuentas de almacenamiento y el servicio StorSimple Data Manager. El servicio StorSimple Device Manager, el servicio Data Manager y la cuenta de almacenamiento asociada deben estar en las regiones admitidas.
    
   5. Para obtener un vínculo a este servicio en el panel, active **Pin to dashboard** (Anclar al panel).
    
   6. Haga clic en **Crear**.

      ![Creación de un servicio StorSimple Data Manager 3](./media/storsimple-data-manager-ui/create-service-4.png)

Se tarda unos minutos en crear el servicio. Verá una notificación cuando el servicio se haya creado correctamente y aparecerá el nuevo servicio.

### <a name="create-a-data-transformation-job-definition"></a>Creación de una definición del trabajo de transformación de datos

En un servicio StorSimple Data Manager, es preciso crear una definición del trabajo de transformación de datos. Una definición del trabajo especifica los detalles de los datos de StorSimple que esté interesado en mover a una cuenta de almacenamiento en el formato nativo. Una vez que cree una definición de trabajo, puede ejecutar este trabajo de nuevo con parámetros de tiempo de ejecución diferentes.

Realice los siguientes pasos para crear una definición de trabajo.

1. Navegue hasta el servicio que ha creado. Vaya a **Management > Job definitions** (Administración > Definiciones de trabajos).

2. Haga clic en **+ Job definition** (+ Definición de trabajo).

    ![Haga clic en + Job Definition (+ Definición de trabajo)](./media/storsimple-data-manager-ui/create-job-definition-1.png)

3. Proporcione un nombre para la definición de trabajo. El nombre puede tener entre 3 y 63 caracteres. El nombre puede contener letras mayúsculas y minúsculas, números y guiones.

4. Especifique una ubicación para que se ejecute el trabajo. Esta ubicación puede ser diferente de la ubicación donde se implementa el servicio.

5. Haga clic en **Source** (Origen) para especificar el repositorio de datos de origen.

    ![Configuración del repositorio de datos de origen](./media/storsimple-data-manager-ui/create-job-definition-2.png)

6. Al ser un servicio Data Manager nuevo, no se configuran repositorios de datos. En **Configure data source** (Configurar origen de datos), especifique los detalles del dispositivo de la serie StorSimple 8000 y los datos de interés.

   Para agregar StorSimple Device Manager como repositorio de datos, haga clic en **Add new** (Agregar nuevo) en la lista desplegable de repositorios de datos y, después, en **Add Data Repository** (Agregar repositorio de datos).

    ![Agregar un nuevo repositorio de datos](./media/storsimple-data-manager-ui/create-job-definition-3.png)
  
   1. Elija **StorSimple 8000 series Manager** como tipo de repositorio de datos.
    
   2. Escriba un nombre descriptivo para el repositorio de datos de origen.
    
   3. En la lista desplegable, elija una suscripción asociada con el servicio StorSimple Device Manager.
    
   4. Proporcione el nombre de StorSimple Device Manager para **Resource** (Recurso).

   5. Escriba la clave de **cifrado de datos del servicio** para el servicio StorSimple Device Manager. 

      ![Configuración del repositorio de datos de origen 1](./media/storsimple-data-manager-ui/create-job-definition-4.png)

      Pulse **OK** (Aceptar) cuando haya terminado. Esto guarda el repositorio de datos. Vuelva a usar esta instancia de StorSimple Device Manager en otras definiciones de trabajo sin escribir de nuevo estos parámetros. Unos segundos después de hacer clic en **OK** (Aceptar), el repositorio de datos de origen recién creado aparece en el desplegable.

7. En la lista desplegable **Data repository** (Repositorio de datos), seleccione el repositorio de datos que ha creado. 

   1. Escriba el nombre del dispositivo de la serie StorSimple 8000 que contiene los datos de interés.

   2. Especifique el nombre del volumen que reside en el dispositivo de StorSimple que tiene los datos de interés.

   3. En la subsección **Filter** (Filtro), escriba el directorio raíz que contiene los datos de interés en formato _\MyRootDirectory\Data_. Las letras de unidad como _\C:\Data_ no se admiten. Aquí también se pueden agregar los filtros de archivo.

   4. El servicio de transformación de datos solo funciona en la instantánea más reciente de los datos que se insertan en Azure.

   5. Haga clic en **OK**.

      ![Configuración del repositorio de datos de origen 2](./media/storsimple-data-manager-ui/create-job-definition-8.png)

8. A continuación, se debe configurar el repositorio de datos de destino. Elija el primero si desea colocar los archivos en blobs de esa cuenta. En la lista desplegable, seleccione **Agregar nuevo** y, después, **Parámetros de configuración**.

9. Seleccione el tipo de repositorio de destino que desea agregar y los restantes parámetros asociados con el repositorio.

    Si selecciona un destino del tipo Cuenta de almacenamiento, puede especificar un nombre descriptivo, una suscripción (elija la misma que la del servicio u otra) y una cuenta de almacenamiento.
        ![Configuración del repositorio de datos de destino 1](./media/storsimple-data-manager-ui/create-job-definition-10.png)

    Cuando se ejecuta el trabajo se crea una cola de almacenamiento. Esta cola se rellena con mensajes de los blobs transformados a medida que están listos. El nombre de esta cola es el mismo que el de la definición del trabajo.
    
10. Después de agregar el repositorio de datos, espere un par de minutos.
    
    1. Seleccione el repositorio que creó como destino en la lista desplegable **Target account name** (Nombre de la cuenta de destino).

    2. Elija el tipo de almacenamiento como blobs o archivos. Especifique el nombre del contenedor de almacenamiento donde residen los datos transformados. Haga clic en **OK**.

        ![Configuración de la cuenta de almacenamiento del repositorio de datos de destino](./media/storsimple-data-manager-ui/create-job-definition-16.png)

11. También puede marcar la opción de presentar una estimación de la duración del trabajo antes de ejecutar el trabajo. Haga clic en **OK** (Aceptar) para crear la definición del trabajo. La definición de trabajo ya ha finalizado. Esta definición de trabajo se puede usar varias veces a través de la interfaz de usuario con parámetros de tiempo de ejecución diferentes.

    ![Definición de trabajo finalizada](./media/storsimple-data-manager-ui/create-job-definition-13.png)

    La definición del trabajo recién creada se agrega a la lista de definiciones de trabajos para este servicio.

### <a name="run-the-job-definition"></a>Ejecución de la definición de trabajo

Siempre que tenga que mover los datos de StorSimple a la cuenta de almacenamiento que ha especificado en la definición del trabajo, será preciso que la ejecute. En el tiempo de ejecución, algunos parámetros se pueden especificar de manera diferente. Los pasos son los siguientes:

1. Seleccione el servicio StorSimple Data Manager y vaya a **Management > Job definitions** (Administración > Definiciones de trabajos). Seleccione y haga clic en la definición de trabajo que desea ejecutar.
     
     ![Iniciar ejecución de trabajo 1](./media/storsimple-data-manager-ui/start-job-run1.png)

2. Haga clic en **Ejecutar ahora**.
     
     ![Iniciar ejecución de trabajo 2](./media/storsimple-data-manager-ui/start-job-run2.png)

3. Haga clic en **Ejecutar Configuración** para modificar los parámetros que desea cambiar para la ejecución de este trabajo. Haga clic en **Aceptar** y, luego, haga clic en **Ejecutar** para iniciar el trabajo.

    ![Iniciar ejecución de trabajo 3](./media/storsimple-data-manager-ui/start-job-run3.png)

4. Para supervisar este trabajo, vaya a la página **Jobs** (Trabajos) de StorSimple Data Manager. Además de la supervisión de la hoja **Trabajos**, también puede escuchar la cola de almacenamiento a la que se agrega un mensaje cada vez que se mueve un archivo desde StorSimple a la cuenta de almacenamiento.

    ![Iniciar ejecución de trabajo 4](./media/storsimple-data-manager-ui/start-job-run4.png)

### <a name="view-logs-after-job-completion"></a>Visualización de registros después de la finalización de trabajos

Una vez finalizado un trabajo, se puede ver su estado. El estado del trabajo puede ser **Realizado correctamente**, **Realizado parcialmente** y **Con errores**. Puede ver la lista de archivos que se han copiado correctamente y los que no se pudieron copiar. Estas listas están disponibles en un contenedor denominado **"storsimple-data-manager-joblogs"** dentro de la cuenta de almacenamiento de destino. En este contenedor, puede buscar una carpeta que tenga el mismo nombre que la definición del trabajo. Dentro de ella se creará una carpeta para cada ejecución de trabajo que contendrá las listas. El nombre de esta carpeta será el GUID del trabajo, que se puede obtener en la página de detalles del trabajo. Como alternativa, en la mayoría de los casos verá un vínculo para los registros de copia dentro de la propia página de trabajos.
En esta carpeta verá dos conjuntos de archivos csv. Todos los archivos que comiencen por **copiedfilelist...** contendrán la lista de archivos copiados correctamente. Todos los archivos que comiencen por **failedfilelist...** contienen archivos que no se pudieron copiar, junto con un mensaje de error.


## <a name="next-steps"></a>Pasos siguientes

[Use the .Net SDK to initiate data transformation](storsimple-data-manager-dotnet-jobs.md) (Uso del SDK de .NET para iniciar la transformación de datos).
