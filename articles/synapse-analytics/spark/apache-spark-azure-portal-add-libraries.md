---
title: Incorporación y administración de bibliotecas para Apache Spark
description: Obtenga información sobre cómo agregar y administrar bibliotecas que usa Apache Spark en Azure Synapse Analytics.
services: synapse-analytics
author: euangMS
ms.service: synapse-analytics
ms.topic: conceptual
ms.date: 04/15/2020
ms.author: euang
ms.reviewer: jrasnick, carlrab
ms.openlocfilehash: c0d34d80df77b5c6fcdefc39b3bc3b1619a93705
ms.sourcegitcommit: 11e2521679415f05d3d2c4c49858940677c57900
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/31/2020
ms.locfileid: "87496260"
---
# <a name="add-and-manage-libraries-for-apache-spark-in-azure-synapse-analytics"></a>Incorporación y administración de bibliotecas para Apache Spark en Azure Synapse Analytics

Apache Spark depende de muchas bibliotecas para ofrecer funcionalidades. Estas bibliotecas se pueden aumentar o reemplazar con bibliotecas adicionales, o con versiones actualizadas de las anteriores.

Los paquetes de Python se pueden agregar en el nivel de grupo de Spark (versión preliminar) y los paquetes basados en .jar se pueden agregar en el nivel de las definiciones de trabajo de Spark.

## <a name="add-or-update-python-libraries"></a>Incorporación o actualización de bibliotecas de Python

Apache Spark en Azure Synapse Analytics tiene una instalación completa de Anaconda y otras bibliotecas adicionales. Puede encontrar la lista de bibliotecas completas en [Compatibilidad con las versiones de Apache Spark](apache-spark-version-support.md).

Cuando se inicia una instancia de Spark, se crea un nuevo entorno virtual con esta instalación como base. Además, se puede usar un archivo *requirements.txt* (salida del comando `pip freeze`) para actualizar el entorno virtual. Los paquetes que se enumeran en este archivo para su instalación o actualización se descargan desde PyPi cuando se inicia el clúster. Este archivo de requisitos se usa cada vez que se crea una instancia de Spark desde ese grupo de Spark.

> [!IMPORTANT]
>
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

### <a name="python-library-user-interface"></a>Interfaz de usuario de la biblioteca de Python

La interfaz de usuario para agregar bibliotecas se encuentra en la pestaña **Configuración adicional** de la página **Create Apache Spark pool** (Crear grupo de Apache Spark) en Azure Portal.

Cargue el archivo de configuración de entorno mediante el selector de archivos en la sección **Paquetes** de la página.

![Adición de bibliotecas de Python](./media/apache-spark-azure-portal-add-libraries/add-python-libraries.png "Adición de bibliotecas de Python")

### <a name="verify-installed-libraries"></a>Comprobación de las bibliotecas instaladas

Para comprobar si se han instalado las versiones correctas de las bibliotecas correctas, ejecute el código siguiente

```python
import pip #needed to use the pip functions
for i in pip.get_installed_distributions(local_only=True):
    print(i)
```

## <a name="next-steps"></a>Pasos siguientes

- [Azure Synapse Analytics](https://docs.microsoft.com/azure/synapse-analytics)
- [Documentación de Apache Spark](https://spark.apache.org/docs/2.4.4/)
