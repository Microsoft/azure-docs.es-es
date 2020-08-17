---
title: Protección de servicios vinculados
description: Aprenda a aprovisionar y proteger un servicio vinculado con una VNET administrada.
services: synapse-analytics
author: acomet
ms.service: synapse-analytics
ms.topic: how-to
ms.subservice: ''
ms.date: 04/15/2020
ms.author: acomet
ms.reviewer: jrasnick
ms.openlocfilehash: 1ce127dbfd9984b3fb18e518701cbbd3a87f5988
ms.sourcegitcommit: 5b8fb60a5ded05c5b7281094d18cf8ae15cb1d55
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/29/2020
ms.locfileid: "87387255"
---
# <a name="secure-a-linked-service-with-private-links"></a>Protección de un servicio vinculado con vínculos privados 

En este artículo, obtendrá información sobre cómo proteger un servicio vinculado en Synapse con un punto de conexión privado.

## <a name="prerequisites"></a>Prerrequisitos

* **Suscripción de Azure**: Si no tiene una suscripción a Azure, cree una [cuenta gratuita de Azure](https://azure.microsoft.com/free/) antes de empezar.
* **Cuenta de Azure Storage**: Puede usar Azure Data Lake Gen2 como almacén de datos de *origen*. Si no tiene una cuenta de almacenamiento, consulte [Crear una cuenta de almacenamiento](../../storage/blobs/data-lake-storage-quickstart-create-account.md) para crear una. Asegúrese de que la cuenta de almacenamiento tenga el filtrado de IP de Synapse Studio para acceder a ella y de que solo permita **Redes seleccionadas** para acceder a la cuenta de almacenamiento. La configuración de la hoja **Firewalls y redes virtuales** debe parecerse a la imagen siguiente.

![Cuenta de almacenamiento protegida](./media/secure-storage-account.png)

## <a name="create-a-linked-service-with-private-links"></a>Creación de un servicio vinculado con vínculos privados

En Azure Synapse Analytics, un servicio vinculado es donde se define la información de conexión a otros servicios. En esta sección, agregará Azure Synapse Analytics y Azure Data Lake Gen2 como servicios vinculados.

1. Abra Azure Synapse Studio y vaya a la pestaña **Administrar**.
1. En **Conexiones externas**, seleccione **Servicios vinculados**.
1. Para agregar un servicio vinculado, haga clic en **Nuevo**.
1. Seleccione el icono de Azure Data Lake Storage Gen2 de la lista y haga clic en **Continuar**.
1. Asegúrese de habilitar **Interactive Authoring** (Creación interactiva). Puede tardar aproximadamente 1 minuto en habilitarse. 
1. Especifique las credenciales de autenticación. Actualmente, la clave de cuenta, la entidad de servicio y la identidad administrada son los tipos de autenticación admitidos. Haga clic en probar conexión para verificar que las credenciales son correctas.
1. Seleccione **Probar conexión**, se debería producir un error, ya que la cuenta de almacenamiento no permite el acceso a ella sin la creación y aprobación de un punto de conexión privado. En el mensaje de error, debería ver un vínculo para crear un **punto de conexión privado** que puede seguir para ir al siguiente elemento. Si sigue ese vínculo, omita la parte siguiente.
1. Cuando haya terminado, seleccione **Crear**.

## <a name="create-a-managed-private-endpoint"></a>Creación de un punto de conexión privado administrado

En caso de no haber hecho clic en el hipervínculo al probar la conexión anterior, siga la ruta de acceso siguiente. Ahora debe crear un punto de conexión privado administrado que conectará al servicio vinculado creado anteriormente.

1. Vaya a la pestaña **Administrar**.
1. Vaya a la sección **Managed Virtual Networks** (Redes virtuales administradas).
1. Seleccione **+ Nuevo** en punto de conexión privado administrado.
1. Seleccione el icono de Azure Data Lake Storage Gen2 de la lista y seleccione **Continuar**.
1. Escriba el nombre de la cuenta de almacenamiento que creó anteriormente.
1. Seleccione **Crear**
1. Después de esperar algunos segundos, debería ver que el vínculo privado creado necesita una aprobación.

## <a name="private-link-approval"></a>Aprobación del vínculo privado
1. Seleccione el punto de conexión privado que creó anteriormente. Puede ver un hipervínculo que le permitirá aprobar el punto de conexión privado a nivel de la cuenta de almacenamiento. *Una alternativa es ir directamente a la cuenta de almacenamiento en Azure Portal e ir a la hoja **Conexiones de punto de conexión privado**.*
1. Marque el punto de conexión privado que creó en Studio y seleccione **Aprobar**.
1. Agregue una descripción y haga clic en **Sí**.
1. Vuelva a Synapse Studio en la sección **Managed Virtual Networks** (Redes virtuales administradas) de la pestaña **Manage** (Administrar).
1. La aprobación tardará aproximadamente 1 minuto en verse reflejada para el punto de conexión privado.

## <a name="check-the-connection-works"></a>Comprobación de que la conexión funciona
1. Vaya a la pestaña **Administrar** y seleccione el servicio vinculado que ha creado.
1. Asegúrese de que la opción **Interactive authoring** (Creación interactiva) está activa.
1. Seleccione **Test connection** (Probar conexión). Debería ver que la conexión se ha establecido correctamente.

Ahora ha establecido una conexión segura y privada entre Synapse y su servicio vinculado.

## <a name="next-steps"></a>Pasos siguientes

Para comprender mejor los puntos de conexión privados administrados en Synapse Analytics, consulte el artículo [Concepto de punto de conexión privado administrado en Synapse](data-integration-data-lake.md).

Para obtener más información sobre la integración de datos para Synapse Analytics, consulte el artículo [Ingesta de datos en Data Lake](data-integration-data-lake.md).
