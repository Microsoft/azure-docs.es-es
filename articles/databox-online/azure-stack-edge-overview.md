---
title: Introducción a Microsoft Azure Stack Edge Pro | Microsoft Docs
description: Se describe Azure Stack Edge Pro, una solución de almacenamiento que usa un dispositivo físico para las transferencias basadas en red a Azure.
services: databox
author: alkohli
ms.service: databox
ms.subservice: edge
ms.topic: overview
ms.date: 09/09/2020
ms.author: alkohli
ms.openlocfilehash: 7030030699668b3d316743955dabfb2cc175f6e1
ms.sourcegitcommit: 53acd9895a4a395efa6d7cd41d7f78e392b9cfbe
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/22/2020
ms.locfileid: "90893883"
---
# <a name="what-is-azure-stack-edge-pro-with-fpga"></a>¿Qué es Azure Stack Edge Pro con FPGA?

[!INCLUDE [data-box-edge-gateway-rename-note](../../includes/data-box-edge-gateway-rename-note.md)]

Azure Stack Edge Pro con FPGA es un dispositivo informático perimetral habilitado para la inteligencia artificial que cuenta con funcionalidades de transferencia de datos de red. En este artículo se proporciona información general acerca de la solución de Azure Stack Edge Pro con FPGA, sus ventajas, sus funcionalidades clave y los escenarios en los que puede implementar este dispositivo.

Azure Stack Edge Pro con FPGA es una solución de hardware como servicio. Microsoft le envía un dispositivo administrado en la nube con una matriz de puertas programables (FPGA) que permite la inferencia acelerada de IA y tiene todas las funcionalidades de una puerta de enlace de almacenamiento de red. 

## <a name="use-cases"></a>Casos de uso

Estos son los distintos escenarios en los que se puede usar Azure Stack Edge Pro para la inferencia rápida de Machine Learning (ML) perimetral y el preprocesamiento de datos antes de enviarlos a Azure.

- **Inferencia con Azure Machine Learning**: con Azure Stack Edge Pro, puede ejecutar modelos de ML para obtener resultados rápidos sobre los que se puede actuar antes de que los datos se envíen a la nube. El conjunto de datos completo se puede transferir opcionalmente para continuar entrenando y mejorando los modelos de Machine Learning. Para más información sobre cómo usar los modelos acelerados por hardware de Azure ML en el dispositivo de Azure Stack Edge Pro, consulte [Implementación de modelos acelerados por hardware de Azure ML en Azure Stack Edge Pro](https://docs.microsoft.com/azure/machine-learning/how-to-deploy-fpga-web-service#deploy-to-a-local-edge-server).

- **Preprocesamiento de datos**: transforme datos antes de enviarlos a Azure para crear un conjunto de datos más accionable. El preprocesamiento se puede usar para: 

    - Agregar datos.
    - Modificar datos, por ejemplo, para quitar los datos personales.
    - Subconjuntos de datos para optimizar el almacenamiento y el ancho de banda, o para un análisis más exhaustivo.
    - Analizar y reaccionar a los eventos de IoT. 

- **Transferir datos a través de la red a Azure**: use Azure Stack Edge Pro para transferir datos a Azure de forma rápida y sencilla para poder realizar más procesos y análisis, o para fines de archivado. 

## <a name="key-capabilities"></a>Principales capacidades

Azure Stack Edge Pro tiene las siguientes funcionalidades:

|Capacidad |Descripción  |
|---------|---------|
|Inferencia de IA acelerada| Habilitado por la FPGA integrada.|
|Procesamiento       |Permite el análisis, procesamiento y filtrado de datos.|
|Alto rendimiento | Transferencias de datos y proceso de alto rendimiento.|
|Acceso a datos     | Ofrece acceso directo a los datos de Azure Storage Blob y Azure Files mediante las API en la nube para un procesamiento de datos adicional en la nube. La memoria caché local del dispositivo se usa para agilizar el acceso a los archivos usados más recientemente.|
|Administración en la nube     |El dispositivo y el servicio se administran con Azure Portal.  |
|Carga sin conexión     | El modo sin conexión es compatible con escenarios de carga sin conexión.|
|Protocolos admitidos     | Compatibilidad con los protocolos estándar SMB y NFS para la ingesta de datos. <br> Para más información sobre las versiones compatibles, consulte [Requisitos del sistema de Azure Stack Edge Pro](azure-stack-edge-system-requirements.md).|
|Actualización de datos     | Capacidad para actualizar los archivos locales con la versión más reciente de la nube.|
|Cifrado    | Compatibilidad con BitLocker para, localmente, cifrar los datos y proteger las transferencias de datos hacia la nube a través de *https*.|
|Limitación de ancho de banda| Limite el uso de ancho de banda durante las horas punta.|
|ExpressRoute | Mayor seguridad a través de ExpressRoute. Use la configuración de emparejamiento en la que el tráfico de los dispositivos locales a los puntos de conexión de almacenamiento en la nube viaja a través de ExpressRoute. Para más información, consulte [Información general sobre ExpressRoute](../expressroute/expressroute-introduction.md).

## <a name="components"></a>Componentes

La solución Azure Stack Edge Pro consta de un recurso Azure Stack Edge, un dispositivo físico de Azure Stack Edge Pro y una interfaz de usuario web local.

* **Dispositivo físico de Azure Stack Edge Pro**: un servidor de montaje en bastidor 1U proporcionado por Microsoft que se puede configurar para enviar datos a Azure.
    
* **Recurso Azure Stack Edge**: es un recurso de Azure Portal que le permite administrar un dispositivo de Azure Stack Edge Pro desde una interfaz web a la cual puede acceder desde diferentes ubicaciones geográficas. Use el recurso Azure Stack Edge para crear y administrar recursos, ver y administrar dispositivos y alertas y administrar recursos compartidos.  

    <!--![The Azure Stack Edge service in Azure portal](media/data-box-overview/data-box-Edge-service1.png)-->

    Para más información, vaya a [Creación de un pedido para el dispositivo de Azure Stack Edge Pro](azure-stack-edge-deploy-prep.md#create-a-new-resource).

* **Interfaz de usuario web local de Azure Stack Edge Pro**: use la interfaz de usuario web local para ejecutar diagnósticos, apagar y reiniciar el dispositivo de Azure Stack Edge Pro, ver registros de copia y ponerse en contacto con el Soporte técnico de Microsoft para realizar una solicitud de servicio.

    <!--![The Azure Stack Edge Pro local web UI](media/data-box-Edge-overview/data-box-Edge-local-web-ui.png)-->

    Para más información acerca de cómo usar la interfaz de usuario basada en web, consulte [Uso de la interfaz de usuario web para administrar Azure Stack Edge Pro](azure-stack-edge-manage-access-power-connectivity-mode.md).

## <a name="region-availability"></a>Disponibilidad en regiones

El dispositivo físico de Azure Stack Edge Pro, el recurso de Azure y la cuenta de almacenamiento de destino a la que transferirá los datos no tienen que estar en la misma región.

- **Disponibilidad de recursos**: para ver una lista de todas las regiones en las que está disponible el recurso de Azure Stack Edge, consulte [Productos de Azure disponibles por región](https://azure.microsoft.com/global-infrastructure/services/?products=databox&regions=all). Azure Stack Edge Pro también se puede implementar en la nube de Azure Government. Para más información, consulte [¿Qué es Azure Government?](https://docs.microsoft.com/azure/azure-government/documentation-government-welcome).
    
- **Cuentas de almacenamiento de destino**: las cuentas de almacenamiento que almacenan los datos están disponibles en todas las regiones de Azure. Las regiones en las que las cuentas de almacenamiento almacenan los datos de Azure Stack Edge Pro deben estar ubicadas cerca de la ubicación del dispositivo para un rendimiento óptimo. Una cuenta de almacenamiento que se encuentra lejos del dispositivo causa latencias largas y un rendimiento más lento.

## <a name="next-steps"></a>Pasos siguientes

- Revise [los requisitos del sistema de Azure Stack Edge Pro](azure-stack-edge-system-requirements.md).
- Consulte los [límites de Azure Stack Edge Pro](azure-stack-edge-limits.md).
- Implemente [Azure Stack Edge Pro](azure-stack-edge-deploy-prep.md) en Azure Portal.
