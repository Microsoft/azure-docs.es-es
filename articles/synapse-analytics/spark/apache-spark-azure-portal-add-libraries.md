---
title: Administración de bibliotecas para Apache Spark en Azure Synapse Analytics
description: Obtenga información sobre cómo agregar y administrar bibliotecas que usa Apache Spark en Azure Synapse Analytics.
services: synapse-analytics
author: euangMS
ms.service: synapse-analytics
ms.topic: conceptual
ms.date: 07/22/2020
ms.author: euang
ms.reviewer: jrasnick, carlrab
ms.subservice: spark
ms.openlocfilehash: b7aea6565e8301e2aeb96263b8e7b1d2ea64995d
ms.sourcegitcommit: 02ca0f340a44b7e18acca1351c8e81f3cca4a370
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/19/2020
ms.locfileid: "88589568"
---
# <a name="manage-libraries-for-apache-spark-in-azure-synapse-analytics"></a>Administración de bibliotecas para Apache Spark en Azure Synapse Analytics

Las bibliotecas proporcionan código reutilizable que se puede incluir en los programas o proyectos. A fin de que el código de terceros o de compilación local esté disponible para las aplicaciones, puede instalar una biblioteca en uno de los grupos de Spark (versión preliminar). Una vez instalada una biblioteca para un grupo de Spark, está disponible para todas las sesiones que usan el mismo grupo. 

## <a name="default-installation"></a>Instalación predeterminada
Apache Spark en Azure Synapse Analytics tiene una instalación completa de Anaconda y otras bibliotecas adicionales. Puede encontrar la lista de bibliotecas completas en [Compatibilidad con las versiones de Apache Spark](apache-spark-version-support.md). 

Cuando se inicie una instancia de Spark, estas bibliotecas se incluirán automáticamente. Se pueden agregar paquetes compilados personalizados y de Python adicionales en el nivel de grupo de Spark (versión preliminar).


## <a name="manage-python-packages"></a>Administración de paquetes de Python
Una vez que haya identificado las bibliotecas que le gustaría usar para la aplicación Spark, puede instalarlas en un grupo de Spark (versión preliminar). 

 Se puede usar un archivo *requirements.txt* (salida del comando `pip freeze`) para actualizar el entorno virtual. Los paquetes que se enumeran en este archivo para su instalación o actualización se descargan desde PyPi cuando se inicia el grupo. Este archivo de requisitos se usa cada vez que se crea una instancia de Spark desde ese grupo de Spark.

> [!IMPORTANT]
> - Si el paquete que va a instalar es de gran tamaño o tarda mucho tiempo en instalarse, afectará al tiempo de inicio de la instancia de Spark.
> - No se admiten los paquetes que requieren compatibilidad con el compilador en el momento de la instalación, como GCC.
> - No es posible cambiar a una versión anterior de los paquetes; solo pueden agregarse o actualizarse.

### <a name="requirements-format"></a>Requisitos de formato

En el fragmento de código siguiente se muestra el formato del archivo de requisitos. Se enumera el nombre del paquete PyPi, junto con una versión exacta. Este archivo sigue el formato descrito en la documentación de referencia de [pip freeze](https://pip.pypa.io/en/stable/reference/pip_freeze/). Este ejemplo fija una versión específica. 

```
absl-py==0.7.0
adal==1.2.1
alabaster==0.7.10
```

### <a name="install-python-packages"></a>Instalación de paquetes de Python
Al desarrollar la aplicación Spark, puede que tenga que actualizar las bibliotecas existentes o instalar nuevas. Las bibliotecas se pueden actualizar durante o después de la creación del grupo.

#### <a name="install-packages-during-pool-creation"></a>Instalación de paquetes durante la creación del grupo
Para instalar las bibliotecas en un grupo de Spark (versión preliminar) durante la creación del grupo:
   
1. Navegue hasta el área de trabajo de Azure Synapse Analytics desde Azure Portal.
   
2. Seleccione **Create Apache Spark pool** (Crear grupo de Apache Spark) y, a continuación, seleccione la pestaña **Configuración adicional**. 
   
3. Cargue el archivo de configuración de entorno mediante el selector de archivos en la sección **Paquetes** de la página. 
   
![Adición de bibliotecas de Python durante la creación de un grupo](./media/apache-spark-azure-portal-add-libraries/apache-spark-azure-portal-add-library-python.png "Adición de bibliotecas de Python")
 

#### <a name="install-packages-from-the-synapse-workspace"></a>Instalación de paquetes desde el área de trabajo de Synapse
Para actualizar o agregar bibliotecas adicionales a un grupo de Spark (versión preliminar) desde el portal de Azure Synapse Analytics:

1.  Navegue hasta el área de trabajo de Azure Synapse Analytics desde Azure Portal.
   
2.  Inicie el área de trabajo de Azure Synapse Analytics desde Azure Portal.

3.  Seleccione **Administrar** en el panel de navegación principal y, a continuación, seleccione **Grupos de Apache Spark**.
   
4. Seleccione un único grupo de Spark y cargue el archivo de configuración de entorno mediante el selector de archivos de la sección **Paquetes** de la página.

![Adición de bibliotecas de Python en Synapse](./media/apache-spark-azure-portal-add-libraries/apache-spark-azure-portal-update.png "Adición de bibliotecas de Python")
   
#### <a name="install-packages-from-the-azure-portal"></a>Instalación de paquetes desde Azure Portal
Para instalar una biblioteca en un grupo de Spark (versión preliminar) directamente desde Azure Portal:
   
 1. Navegue hasta el área de trabajo de Azure Synapse Analytics desde Azure Portal.
   
 2. En la sección **Synapse resources** (Recursos de Synapse), seleccione la pestaña **Apache Spark pools** y seleccione un grupo de Spark en la lista.
   
 3. Seleccione **Paquetes** en la sección **Configuración** del grupo de Spark. 

 4. Cargue el archivo de configuración de entorno mediante el selector de archivos.

![Adición de bibliotecas de Python](./media/apache-spark-azure-portal-add-libraries/apache-spark-add-library-azure.png "Adición de bibliotecas de Python")

### <a name="verify-installed-libraries"></a>Comprobación de las bibliotecas instaladas

Para comprobar si se han instalado las versiones correctas de las bibliotecas correctas, ejecute el código siguiente

```python
import pip #needed to use the pip functions
for i in pip.get_installed_distributions(local_only=True):
    print(i)
```
### <a name="update-python-packages"></a>Actualización de paquetes de Python
Los paquetes se pueden agregar o modificar en cualquier momento entre sesiones. Al cargar un nuevo archivo de configuración de paquetes, se sobrescribirán los paquetes y las versiones existentes.  

Para actualizar o desinstalar una biblioteca:
1. Navegue hasta el área de trabajo de Azure Synapse Analytics. 

2. Con Azure Portal o el área de trabajo Azure Synapse, seleccione el **grupo de Apache Spark** que desea actualizar.

3. Navegue hasta la sección **Paquetes** y cargue un nuevo archivo de configuración de entorno.
   
4. Una vez que guarde los cambios, deberá finalizar las sesiones activas y dejar que el grupo se reinicie. Opcionalmente, puede forzar la finalización de las sesiones activas seleccionando la casilla para **forzar configuración nueva**.

![Adición de bibliotecas de Python](./media/apache-spark-azure-portal-add-libraries/update-libraries.png "Adición de bibliotecas de Python")
   

> [!IMPORTANT]
> Al seleccionar la opción para **forzar configuración nueva**, finalizarán todas las sesiones actuales del grupo de Spark seleccionado. Una vez finalizadas las sesiones, tendrá que esperar a que el grupo se reinicie. 
>
> Si esta opción no está seleccionada, tendrá que esperar a que la sesión de Spark actual finalice o detenerla manualmente. Una vez finalizada la sesión, deberá dejar que el grupo se reinicie. 


## <a name="manage-a-python-wheel"></a>Administración de un archivo wheel de Python

### <a name="install-a-custom-wheel-file"></a>Instalación de un archivo wheel personalizado
Los paquetes wheel de compilación personalizada se pueden instalar en el grupo de Apache Spark mediante la carga de todos los archivos wheel en la cuenta de Azure Data Lake Storage (Gen2) que está vinculada al área de trabajo de Synapse. 

Los archivos se deben cargar en la siguiente ruta de acceso en el contenedor predeterminado de la cuenta de almacenamiento: 

```
abfss://<file_system>@<account_name>.dfs.core.windows.net/synapse/workspaces/<workspace_name>sparkpools/<pool_name>libraries/python/
```

>[!IMPORTANT]
>Los paquetes personalizados se pueden agregar o modificar entre sesiones. Sin embargo, deberá esperar a que el grupo y la sesión se reinicien para ver el paquete actualizado.

## <a name="next-steps"></a>Pasos siguientes
- Visualización de las bibliotecas predeterminadas: [Compatibilidad de las versiones de Azure Spark](apache-spark-version-support.md)
