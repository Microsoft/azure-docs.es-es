---
title: 'ML Studio (clásico): Creación de puntos de conexión de servicio web: Azure'
description: Cree puntos de conexión de servicio web en Azure Machine Learning Studio (clásico). Cada punto de conexión del servicio web se administra, limita y dirige de forma independiente.
services: machine-learning
ms.service: machine-learning
ms.subservice: studio
ms.topic: how-to
author: likebupt
ms.author: keli19
ms.custom: seodec18
ms.date: 02/15/2019
ms.openlocfilehash: 31c426ffa85d4eb78a8420d56cce93132b4b7539
ms.sourcegitcommit: 0b8320ae0d3455344ec8855b5c2d0ab3faa974a3
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/30/2020
ms.locfileid: "87432859"
---
# <a name="create-endpoints-for-deployed-azure-machine-learning-studio-classic-web-services"></a>Creación de puntos de conexión para los servicios web de Azure Machine Learning Studio (clásico) implementados

**SE APLICA A:**  ![sí](../../../includes/media/aml-applies-to-skus/yes.png)Machine Learning Studio (clásico) ![no](../../../includes/media/aml-applies-to-skus/no.png)[Azure Machine Learning](../compare-azure-ml-to-studio-classic.md)


> [!NOTE]
> En este tema se describen técnicas que se aplican a un servicio web de Machine Learning **Clásico**.

Cuando se implementa un servicio web, se crea un punto de conexión predeterminado para ese servicio. El punto de conexión predeterminado puede llamarse mediante su clave de API. Puede agregar más puntos de conexión con sus propias claves desde el portal de servicios web.
Cada punto de conexión del servicio web se administra, limita y dirige de forma independiente. Cada punto de conexión es una dirección URL única con una clave de autorización que puede distribuir a sus clientes.

## <a name="add-endpoints-to-a-web-service"></a>Incorporación de puntos de conexión a un servicio web

Puede agregar un punto de conexión a un servicio web mediante el portal de servicios web de Azure Machine Learning. Una vez creado el extremo, puede consumirlo a través de API sincrónicas, API de lotes y hojas de cálculo de Excel.

> [!NOTE]
> Si ha agregado más puntos de conexión al servicio web, no podrá eliminar el predeterminado.

1. En Machine Learning Studio (clásico), en la columna de navegación izquierda, haga clic en Servicios web.
2. En la parte inferior del panel de servicios web, haga clic en **Manage endpoints**(Administrar puntos de conexión). El portal de servicios web de Azure Machine Learning se abre en la página de puntos de conexión del servicio web.
3. Haga clic en **Nueva**.
4. Escriba un nombre y una descripción para el nuevo punto de conexión. Los nombres de los puntos de conexión deben tener 24 caracteres o menos y deben estar formados por letras en minúsculas o números. Seleccione el nivel de registro y si los datos de ejemplo están habilitados. Para más información sobre los registros, vea [Habilitar el registro para los servicios web de Machine Learning](web-services-logging.md).

## <a name="scale-a-web-service-by-adding-additional-endpoints"></a><a id="scaling"></a> Escalado de un servicio web mediante la incorporación de puntos de conexión adicionales

De manera predeterminada, cada servicio web publicado está configurado para admitir 20 solicitudes simultáneas y 200 solicitudes simultáneas como máximo. Azure Machine Learning Studio (clásico) optimiza automáticamente este valor con el fin de brindar el mejor rendimiento al servicio web, por lo que se omite el valor del portal.

Si tiene previsto llamar a la API con una carga mayor que un máximo de 200 llamadas simultáneas, debe crear varios puntos de conexión en el mismo servicio web. Acto seguido, podrá distribuir aleatoriamente la carga entre todos ellos.

El escalado de un servicio web es una tarea común. Algunos de los motivos para ello son admitir más de 200 solicitudes simultáneas, aumentar la disponibilidad a través de varios puntos de conexión u ofrecer puntos de conexión independientes para el servicio web. Para aumentar la escala, puede agregar más puntos de conexión para el mismo servicio web mediante el portal de [servicios web de Azure Machine Learning](https://services.azureml.net/).

Tenga en cuenta que usar un recuento de simultaneidad alto puede ser perjudicial si no se llama a la API con una tasa elevada en consecuencia. Puede que vea tiempos de espera esporádicos o picos de latencia si pone una carga relativamente baja en una API configurada para una carga elevada.

Las API sincrónicas se usan normalmente en situaciones en las que se desea una latencia baja. Latencia aquí implica el tiempo que tarda la API en completar una solicitud, sin contar los retrasos de red. Supongamos que tiene una API con una latencia de 50 ms. Para utilizar totalmente la capacidad disponible con nivel de limitación alto y un número máximo de llamadas simultáneas de 20, debe llamar a esta API 20 * 1000/50 = 400 veces por segundo. Al ampliar esto aún más, un número máximo de llamadas simultáneas de 200 le permitirá llamar a la API 4000 veces por segundo, si presuponemos que la latencia es de 50 ms.

## <a name="next-steps"></a>Pasos siguientes

[Cómo consumir un servicio web de Azure Machine Learning](consume-web-services.md).
