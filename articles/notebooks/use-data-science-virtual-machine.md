---
title: Usar máquinas virtuales de ciencia de datos de Azure
description: Conectarse a un Azure Data Science Virtual Machine (DSVM) para ampliar la potencia de proceso disponible para cuadernos de Azure.
services: app-service
documentationcenter: ''
author: getroyer
manager: andneil
ms.assetid: 0ccc2529-e17f-4221-b7c7-9496d6a731cc
ms.service: azure-notebooks
ms.workload: na
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: article
ms.date: 05/08/2019
ms.author: getroyer
ms.openlocfilehash: d4321fe60d47bf942e2c413ba81c2c9f54317fd1
ms.sourcegitcommit: 1fbc75b822d7fe8d766329f443506b830e101a5e
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 05/14/2019
ms.locfileid: "65597872"
---
# <a name="use-azure-data-science-virtual-machines"></a>Usar máquinas virtuales de ciencia de datos de Azure

De forma predeterminada, los proyectos se ejecutan el **gratuitos Compute** nivel, que está limitado a 4 GB de memoria y 1 GB de datos para evitar abusos. Puede omitir estas limitaciones mediante el uso de una máquina virtual diferente que se haya aprovisionado en una suscripción a Azure. Para este propósito, la mejor opción es una máquina Virtual de ciencia de datos (DSVM) de Azure mediante el **Data Science Virtual Machine para Linux (Ubuntu)** imagen. Este tipo de instancia de DSVM viene preconfigurado con todo lo que necesita para Azure Notebooks y aparece automáticamente en el **ejecutar** lista desplegable en blocs de notas de Azure.

> [!Note]
> Azure Notebooks solo se admite en Dsvm que se creó con la imagen de Linux Ubuntu en. No se admiten los blocs de notas en las imágenes de Windows 2012, Windows 2016 o CentOS Linux.

## <a name="create-a-dsvm-instance"></a>Crear una instancia DSVM

Para crear una nueva instancia de DSVM, siga las instrucciones de la sección [Create an Ubuntu Data Science VM](/azure/machine-learning/data-science-virtual-machine/dsvm-ubuntu-intro) (Creación de un entorno de Data Science VM de Ubuntu). Para obtener más información, incluidos los detalles de precios, consulte [Data Science Virtual Machines](https://azure.microsoft.com/services/virtual-machines/data-science-virtual-machines/).

## <a name="connect-to-the-dsvm"></a>Conexión a la DSVM

Una vez que creen la DSVM, seleccione el **ejecutar** lista desplegable en el Azure Notebooks panel del proyecto y seleccione la instancia DSVM adecuada. La lista desplegable muestra las instancias DSVM si las condiciones siguientes son verdaderas:

- Ha iniciado sesión en Azure Notebooks con una cuenta que usa Azure Active Directory (AAD), como una cuenta de empresa.
- La cuenta está conectada a una suscripción a Azure.
- Tener una o varias máquinas virtuales en esa suscripción, con al menos acceso de lectura, que usa la máquina Virtual de ciencia de datos para la imagen de Linux (Ubuntu).)

![Instancias de Data Science Virtual Machine en la lista desplegable del panel del proyecto](media/project-compute-tier-dsvm.png)

Al seleccionar una instancia de DSVM, Azure Notebooks puede solicitar las credenciales de la máquina específica que usó cuando creó la VM.

Si no se cumple alguna de las condiciones, todavía puede conectarse a la instancia de DSVM. En la lista desplegable, seleccione el **proceso directo** opción, que le pide las credenciales de la máquina virtual, dirección IP de la máquina virtual y puerto (normalmente 8000, el puerto predeterminado para el que realiza escuchas de JupyterHub) y un nombre (que se muestra en la lista):

![Solicitud para recopilar información del servidor para la opción Direct Compute (Proceso directo)](media/project-compute-tier-direct.png)

Obtener estos valores desde la página de la DSVM en Azure portal.

## <a name="accessing-azure-notebooks-files-from-the-dsvm"></a>Acceso a archivos de Azure Notebooks desde DSVM

Conservar la paridad de las rutas de acceso de archivo con el **gratuitos Compute** nivel, es posible abrir sólo un proyecto a la vez en una instancia de DSVM. Para abrir un nuevo proyecto, debe cerrar el proyecto abierto en primer lugar.

![Botón de apagado en Azure Notebooks](media/shutdown.png)

Cuando un proyecto se ejecuta en una máquina virtual, los archivos se montan en el directorio raíz del servidor de Jupyter (el directorio que se muestra en JupyterHub), reemplazando los archivos de Azure Notebooks predeterminados. Cuando apaga la máquina virtual mediante la **apagado** botón en el Bloc de notas la interfaz de usuario, Azure Notebooks restaura los archivos de forma predeterminada.

## <a name="next-steps"></a>Pasos siguientes

Más información sobre Dsvm en [Introducción a Azure Data Science Virtual Machines](/machine-learning/data-science-virtual-machine/overview).
