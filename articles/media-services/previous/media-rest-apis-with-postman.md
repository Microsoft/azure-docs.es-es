---
title: Configuración de Postman para llamadas API de REST de Azure Media Services
description: En este artículo se describe cómo configurar Postman para llamadas API REST de Media Services.
services: media-services
documentationcenter: ''
author: Juliako
manager: femila
editor: ''
ms.service: media-services
ms.workload: media
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: article
ms.date: 04/01/2019
ms.author: juliako
ms.openlocfilehash: 21eb3d9862c8fa2b840117d5f0e9fbd0ad2214c9
ms.sourcegitcommit: 3d79f737ff34708b48dd2ae45100e2516af9ed78
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/23/2020
ms.locfileid: "87091749"
---
# <a name="configure-postman-for-media-services-v2-rest-api-calls"></a>Configuración de Postman para llamadas a API REST de Media Services v2  

> [!NOTE]
> No hay características o funcionalidades nuevas para agregar a Media Services, versión 2. <br/>Finalice la compra de la versión más reciente, [Media Services v3](../latest/index.yml). Consulte también la [guía de migración de la versión v2 a v3](../latest/migrate-from-v2-to-v3.md).

En este tutorial se muestra cómo configurar **Postman** para que se pueda usar para llamar a las API de REST de Azure Media Services (AMS). El tutorial muestra cómo importar archivos de entorno y colección a **Postman**. La colección contiene definiciones agrupadas de solicitudes HTTP que llaman a API de REST de Azure Media Services. El archivo de entorno contiene variables que la colección usa.

Este entorno y esta colección se utilizan en los artículos que muestran cómo realizar diversas tareas con las API de REST de Azure Media Services.

## <a name="prerequisites"></a>Prerequisites

- Instale el cliente de REST de [Postman](https://www.getpostman.com/) para ejecutar las API de REST mostradas en algunos de los tutoriales de REST de AMS. 

    Usamos **Postman** pero cualquier herramienta de REST sería adecuada. Otras alternativas son: **Visual Studio Code** con el complemento de REST o **Telerik Fiddler**. 

## <a name="configure-the-environment"></a>Configuración del entorno 

1. Cree un archivo .json que contenga las variables de entorno utilizadas en los tutoriales de AMS. Asigne el nombre (por ejemplo, **AzureMediaServices.postman_environment.json**) al archivo. Abra el archivo y pegue el código que define el entorno de Postman desde [esta lista de códigos](postman-environment.md). 
2. Abra **Postman**.
3. A la derecha de la pantalla, seleccione la opción **Manage Environments** (Administrar entornos).

    ![Cargar un archivo](./media/media-services-rest-upload-files/postman-create-env.png)
4. En el cuadro de diálogo **Manage Environments** (Administrar entornos), haga clic en **Import** (Importar).
5. Busque y seleccione el archivo **AzureMediaServices.postman_environment.json**.
6. Se agregará el entorno **AzureMedia**.
7. Cierre el cuadro de diálogo.
8. Seleccione el entorno **AzureMedia**.

    ![Cargar un archivo](./media/media-services-rest-upload-files/postman-choose-env.png)

## <a name="configure-the-collection"></a>Configuración de la colección

1. Cree un archivo .json que contenga la colección de **Postman** con todas las operaciones que son necesarios para cargar un archivo a Media Services. Asigne el nombre (por ejemplo, **AzureMediaServicesOperations.postman_collection.json**) al archivo. Abra el archivo y pegue el código que define la colección de **Postman** desde [esta lista de códigos](postman-collection.md).
2. Haga clic en **Import** (Importar) para importar el archivo de la colección.
3. Elija el archivo **AzureMediaServicesOperations.postman_collection.json**.

    ![Cargar un archivo](./media/media-services-rest-upload-files/postman-import-collection.png)

## <a name="next-steps"></a>Pasos siguientes

Consulte el artículo sobre [carga de activos](media-services-rest-upload-files.md).  
