---
title: Creación y eliminación de puntos de conexión privados en un clúster de Azure Stream Analytics
description: Obtenga información sobre cómo crear y eliminar puntos de conexión privados en un clúster de Azure Stream Analytics
author: sidramadoss
ms.author: sidram
ms.reviewer: mamccrea
ms.service: stream-analytics
ms.topic: overview
ms.custom: mvc
ms.date: 09/22/2020
ms.openlocfilehash: baf80e3d543bee455dd8dfa5bc09bc5bf43c3453
ms.sourcegitcommit: 53acd9895a4a395efa6d7cd41d7f78e392b9cfbe
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/22/2020
ms.locfileid: "90945226"
---
# <a name="create-and-delete-private-endpoints-in-an-azure-stream-analytics-cluster"></a>Creación y eliminación de puntos de conexión privados en un clúster de Azure Stream Analytics

Puede conectar los trabajos de Azure Stream Analytics que se ejecutan en un clúster a los recursos de entrada y salida subyacentes de un firewall o una instancia de Azure Virtual Network (red virtual). En primer lugar, cree un punto de conexión privado para un recurso, como un centro de eventos de Azure o una instancia de Azure SQL Database, en el clúster de Stream Analytics. A continuación, apruebe la conexión del punto de conexión privado de la entrada o la salida.

Una vez aprobada la conexión, cualquier trabajo que se ejecute en el clúster de Stream Analytics tiene acceso al recurso a través del punto de conexión privado. En este artículo se muestra cómo crear y eliminar puntos de conexión privados en un clúster de Stream Analytics.

## <a name="create-private-endpoint-in-stream-analytics-cluster"></a>Creación de un punto de conexión privado del clúster de Stream Analytics

En esta sección, aprenderá a crear un punto de conexión privado en un clúster de Stream Analytics.

1. En Azure Portal, busque y seleccione el clúster de Stream Analytics.

1. En **Configuración**, seleccione **Puntos de conexión privados**.

1. Seleccione **Agregar un punto de conexión privado** y escriba la información siguiente para elegir el recurso al que quiere obtener acceso de forma segura a través de un punto de conexión privado.

   |Parámetro|Valor|
   |---|---|
   |Nombre|Escriba cualquier nombre para su punto de conexión privado. Si el nombre ya existe, cree uno único.|
   |Método de conexión|Seleccione **Conectarse a un recurso de Azure en mi directorio**.<br><br>Puede elegir uno de los recursos para conectarse a él de forma segura mediante el punto de conexión privado o puede conectarse al recurso de otra persona mediante un identificador del recurso o un alias que le comparta dicha persona.|
   |Subscription|Seleccione su suscripción.|
   |Tipo de recurso|Elija el [tipo de recurso que se asigna a su recurso](../private-link/private-endpoint-overview.md#private-link-resource).|
   |Recurso|Seleccione el recurso al que quiere conectarse con el punto de conexión privado.|
   |Subrecurso de destino|Tipo de subrecurso del recurso seleccionado anteriormente al que podrá acceder su punto de conexión privado.|

   ![Experiencia de creación del punto de conexión privado](./media/private-endpoints/create-private-endpoint.png)

1. Apruebe la conexión desde el recurso de destino. Por ejemplo, si creó un punto de conexión privado a una instancia de Azure SQL Database en el paso anterior, debe ir a esta instancia de SQL Database y poder ver una conexión pendiente que se debe aprobar. La solicitud de conexión puede tardar unos minutos en aparecer.

    ![Aprobación de un punto de conexión privado](./media/private-endpoints/approve-private-endpoint.png)

1. Puede volver al clúster de Stream Analytics para ver cómo el estado cambia en dos minutos de **Pendiente de aprobación del cliente** a **Pending DNS Setup** (Pendiente de configuración de DNS) y, luego, a **Set up complete**" (Configuración completada).

## <a name="delete-a-private-endpoint-in-a-stream-analytics-cluster"></a>Eliminación de un punto de conexión privado del clúster de Stream Analytics

1. En Azure Portal, busque y seleccione el clúster de Stream Analytics.

1. En **Configuración**, seleccione **Puntos de conexión privados**.

1. Seleccione el punto de conexión privado que quiere eliminar y seleccione **Eliminar.**

   ![Eliminación del punto de conexión privado](./media/private-endpoints/delete-private-endpoint.png)

## <a name="next-steps"></a>Pasos siguientes

Ahora ha obtenido información general sobre cómo administrar puntos de conexión privados en un clúster de Azure Stream Analytics. A continuación, puede obtener información sobre cómo escalar los clústeres y ejecutar trabajos en el clúster:

* [Escalado de un clúster de Azure Stream Analytics](scale-cluster.md)
* [Administración de trabajos de Stream Analytics en un clúster de Stream Analytics](manage-jobs-cluster.md)