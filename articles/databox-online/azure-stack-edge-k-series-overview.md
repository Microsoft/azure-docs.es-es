---
title: Introducción a Azure Stack Edge Mini R | Microsoft Docs
description: Se describen Azure Stack Edge Mini R, una solución de almacenamiento para aplicaciones militares que usa un dispositivo físico portátil con una batería para la transferencia a través de Wi-Fi a Azure.
services: databox
author: alkohli
ms.service: databox
ms.subservice: edge
ms.topic: overview
ms.date: 09/22/2020
ms.author: alkohli
ms.openlocfilehash: 20efae411ae4d2fae9bf3b5e69dbfdd98da1603a
ms.sourcegitcommit: bdd5c76457b0f0504f4f679a316b959dcfabf1ef
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/22/2020
ms.locfileid: "90985647"
---
# <a name="what-is-the-azure-stack-edge-mini-r"></a>¿Qué es Azure Stack Edge Mini R?

Azure Stack Edge Mini R es una solución de hardware como servicio. Microsoft le envía un dispositivo administrado en la nube, resistente y ultraportátil con una unidad de procesamiento de visión (VPU) que permite la inferencia acelerada de IA y cuenta con todas las funcionalidades de una puerta de enlace de almacenamiento de red. Estos dispositivos son adecuados para usarse en los entornos más exigentes y están optimizados para la inteligencia artificial, el análisis y la informática sin servidor.

En este artículo le proporcionamos información general acerca de la solución Azure Stack Edge Mini R, sus funcionalidades clave y los escenarios en los que puede implementar este dispositivo.


## <a name="key-capabilities"></a>Principales capacidades

Azure Stack Edge Mini R presenta las siguientes funcionalidades:

|Capacidad |Descripción  |
|---------|---------|
|Hardware resistente| Hardware resistente diseñado para los entornos más exigentes.|
|Ultraportátil| Factor de forma ultra portátil y con batería.|
|Administración en la nube|El dispositivo y el servicio se administran con Azure Portal.|
|Cargas de trabajo de proceso perimetral|Permite el análisis, procesamiento y filtrado de datos.<br>Admite máquinas virtuales y cargas de trabajo en contenedores. |
|Inferencia de IA acelerada| Habilitado por la VPU Intel Movidius Myriad X.|
|Cableado e inalámbrico | Permite las transferencias de datos por cable e inalámbricas.|
|Acceso a datos     | Ofrece acceso directo a los datos de Azure Storage Blob y Azure Files mediante las API en la nube para un procesamiento de datos adicional en la nube. La memoria caché local del dispositivo se usa para agilizar el acceso a los archivos usados más recientemente.|
|Modo desconectado|  El dispositivo y el servicio se pueden administrar a través de Azure Stack Hub. Implemente, ejecute y administre aplicaciones en modo sin conexión. <br> El modo sin conexión es compatible con escenarios de carga sin conexión.|
|Protocolos admitidos     |Compatibilidad con los protocolos estándar SMB, NFS y REST para la ingesta de datos. <br> Para obtener más información sobre las versiones compatibles, vaya a [Requisitos del sistema de Azure Stack Edge Mini R](azure-stack-edge-gpu-system-requirements.md).|
|Actualización de datos     | Capacidad para actualizar los archivos locales con la versión más reciente de la nube.|
|Cifrado doble    | El uso de la unidad de cifrado automático proporciona la primera capa de cifrado. VPN proporciona la segunda capa de cifrado. Compatibilidad con BitLocker para cifrar los datos localmente y proteger las transferencias de datos en la nube mediante *HTTPS*.|
|Límite de ancho de banda| Limite el uso de ancho de banda durante las horas punta.|

## <a name="use-cases"></a>Casos de uso

Estos son los distintos escenarios en los que se puede usar Azure Stack Edge Mini R para la inferencia rápida de Machine Learning (ML) perimetral y el preprocesamiento de datos antes de enviarlos a Azure.

- **Inferencia con Azure Machine Learning**: con Azure Stack Edge Mini R, puede ejecutar modelos de ML para obtener resultados rápidos sobre los que se puede actuar antes de que los datos se envíen a la nube. El conjunto de datos completo se puede transferir opcionalmente para continuar entrenando y mejorando los modelos de Machine Learning. Para más información sobre cómo usar los modelos acelerados por hardware de Azure ML en el dispositivo Azure Stack Edge Mini R, consulte [Implementación de modelos acelerados por hardware de Azure ML en Azure Stack Edge Mini R](https://docs.microsoft.com/azure/machine-learning/service/how-to-deploy-fpga-web-service#deploy-to-a-local-edge-server).

- **Preprocesamiento de datos**: transforme los datos mediante opciones de proceso, como contenedores o máquinas virtuales, antes de enviarlos a Azure para crear un conjunto de datos más útil. El preprocesamiento se puede usar para:

    - Agregar datos.
    - Modificar datos, por ejemplo, para quitar los datos personales.
    - Subconjuntos de datos para optimizar el almacenamiento y el ancho de banda, o para un análisis más exhaustivo.
    - Analizar y reaccionar a los eventos de IoT.

- **Transferir datos a través de la red a Azure**: use Azure Stack Edge Mini R para transferir datos a Azure de forma rápida y sencilla a fin de poder realizar más procesos y análisis o para fines de archivado.

## <a name="components"></a>Componentes

La solución Azure Stack Edge Mini R consta de un recurso Azure Stack Edge, un dispositivo físico Azure Stack Edge Mini R resistente y ultraportátil y una interfaz web local.

* **Dispositivo físico Azure Stack Edge Mini R**: dispositivo ultraportátil con una batería incorporada y una carcasa resistente proporcionada por Microsoft que se puede configurar para enviar datos a Azure. El dispositivo con la batería pesa menos de 3,17 kg.

    ![Dispositivo Azure Stack Edge Mini R](media/azure-stack-edge-k-series-overview/perspective-view-1.png)

* **Recurso Azure Stack Edge**: recurso de Azure Portal que le permite administrar un dispositivo Azure Stack Edge Mini R resistente desde una interfaz web a la cual puede acceder desde diferentes ubicaciones geográficas. Use el recurso Azure Stack Edge para crear y administrar recursos, ver y administrar dispositivos y alertas y administrar recursos compartidos.  

* **Interfaz de usuario web local de Azure Stack Edge Mini R**: use la interfaz de usuario web local para realizar la configuración inicial del dispositivo, ejecutar diagnósticos, apagar y reiniciar el dispositivo Azure Stack Edge Mini R, ver registros de copia y ponerse en contacto con el Soporte técnico de Microsoft a fin de realizar una solicitud de servicio.


## <a name="region-availability"></a>Disponibilidad en regiones

El dispositivo físico Azure Stack Edge Mini R, el recurso de Azure y la cuenta de almacenamiento de destino a la que transferirá los datos no tienen que estar en la misma región.

- **Disponibilidad de recursos**: para obtener una lista de todas las regiones en que está disponible el recurso de Azure Stack Edge, vaya a [Productos de Azure disponibles por región](https://azure.microsoft.com/global-infrastructure/services/?products=databox&regions=all). 

- **Cuentas de almacenamiento de destino**: las cuentas de almacenamiento que almacenan los datos están disponibles en todas las regiones de Azure. Las regiones en las que las cuentas de almacenamiento almacenan los datos de Azure Stack Edge Mini R deben estar cerca de dónde se encuentra el dispositivo a fin de obtener un rendimiento óptimo. Una cuenta de almacenamiento que se encuentra lejos del dispositivo causa latencias largas y un rendimiento más lento.


## <a name="next-steps"></a>Pasos siguientes

- Revise los [requisitos del sistema de Azure Stack Edge Mini R](azure-stack-edge-gpu-system-requirements.md).


