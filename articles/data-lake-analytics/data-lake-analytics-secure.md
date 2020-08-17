---
title: Protección de Azure Data Lake Analytics para varios usuarios
description: Obtenga información sobre cómo configurar varios usuarios para que ejecuten trabajos en Azure Data Lake Analytics.
ms.service: data-lake-analytics
services: data-lake-analytics
ms.reviewer: jasonh
ms.topic: how-to
ms.date: 05/30/2018
ms.openlocfilehash: 9006a22c588a7f1456585d40da0b4345145c6d05
ms.sourcegitcommit: 0e8a4671aa3f5a9a54231fea48bcfb432a1e528c
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/24/2020
ms.locfileid: "87132490"
---
# <a name="configure-user-access-to-job-information-to-job-information-in-azure-data-lake-analytics"></a>Configuración del acceso de usuario en la información del trabajo desde Azure Data Lake Analytics 

En Azure Data Lake Analytics puede usar varias cuentas de usuario o entidades de servicio para ejecutar trabajos. 

Para que esos usuarios puedan ver la información detallada del trabajo, deben poder leer el contenido de las carpetas de trabajo. Dichas carpetas están en el directorio `/system/`. 

Si no se configuran los permisos necesarios, es posible que se muestre el error siguiente al usuario: `Graph data not available - You don't have permissions to access the graph data.` 

## <a name="configure-user-access-to-job-information"></a>Configuración del acceso de usuario en la información del trabajo

Puede usar el **Asistente para agregar usuario** para configurar las ACL en las carpetas. Para obtener más información, consulte la sección [Agregar un nuevo usuario](data-lake-analytics-manage-use-portal.md#add-a-new-user).

Si necesita un control más detallado o tiene que crear scripts para los permisos, proteja las carpetas de esta forma:

1. Conceda permisos de **ejecución** (mediante una ACL de acceso) en la carpeta raíz:
   - /
   
2. Conceda permisos de **ejecución** y **lectura** (mediante una ACL de acceso y una ACL predeterminada) en las carpetas que contienen las carpetas de trabajo. Por ejemplo, para un trabajo específico que se ejecutó el 25 de mayo de 2018, es necesario acceder a estas carpetas:
   - /system
   - /system/jobservice
   - /system/jobservice/jobs
   - /system/jobservice/jobs/Usql
   - /system/jobservice/jobs/Usql/2018
   - /system/jobservice/jobs/Usql/2018/05
   - /system/jobservice/jobs/Usql/2018/05/25
   - /system/jobservice/jobs/Usql/2018/05/25/11
   - /system/jobservice/jobs/Usql/2018/05/25/11/01
   - /system/jobservice/jobs/Usql/2018/05/25/11/01/b074bd7a-1448-d879-9d75-f562b101bd3d

## <a name="next-steps"></a>Pasos siguientes
[Agregar un nuevo usuario](data-lake-analytics-manage-use-portal.md#add-a-new-user)
