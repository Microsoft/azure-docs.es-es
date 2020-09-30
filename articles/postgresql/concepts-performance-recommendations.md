---
title: 'Recomendaciones de rendimiento de Azure Database for PostgreSQL: servidor único'
description: En este artículo se describe la característica Recomendaciones de rendimiento de Azure Database for PostgreSQL con un único servidor.
author: rachel-msft
ms.author: raagyema
ms.service: postgresql
ms.topic: conceptual
ms.date: 08/21/2019
ms.openlocfilehash: f0ce3843752ebd6ed56281f6699783181b52fdc6
ms.sourcegitcommit: 53acd9895a4a395efa6d7cd41d7f78e392b9cfbe
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/22/2020
ms.locfileid: "90903878"
---
# <a name="performance-recommendations-in-azure-database-for-postgresql---single-server"></a>Recomendaciones de rendimiento en Azure Database for PostgreSQL con un único servidor

**Se aplica a:** Azure Database for PostgreSQL: Versiones de servidor único 9.6, 10, 11

La característica Recomendaciones de rendimiento analiza las bases de datos para crear sugerencias personalizadas para mejorar el rendimiento. Para generar las recomendaciones, el análisis examina diversas características de bases de datos, incluido el esquema. Habilite [Almacén de consultas](concepts-query-store.md) en el servidor para poder usar por completo la característica Recomendaciones de rendimiento. Después de implementar una recomendación de rendimiento, debe probar el rendimiento para evaluar el efecto de estos cambios. 

## <a name="permissions"></a>Permisos
Los permisos **Propietario** o **Colaborador** necesarios para ejecutar el análisis con la característica Recomendaciones de rendimiento.

## <a name="performance-recommendations"></a>Recomendaciones de rendimiento
La característica [Recomendaciones de rendimiento](concepts-performance-recommendations.md) analiza las cargas de trabajo en el servidor para identificar los índices con el potencial de mejorar el rendimiento.

Abra **Recomendaciones de rendimiento** desde la sección **Rendimiento inteligente** de la barra de menús en la página de Azure Portal para el servidor PostgreSQL.

:::image type="content" source="./media/concepts-performance-recommendations/performance-recommendations-page.png" alt-text="Página de inicio de Recomendaciones de rendimiento":::

Seleccione **Analizar** y elija una base de datos, lo que iniciará el análisis. Según la carga de trabajo, el análisis puede tardar varios minutos. Una vez que se realice el análisis, habrá una notificación en el portal. El análisis realiza un examen en profundidad de la base de datos. Se recomienda realizar el análisis durante los períodos de poca actividad. 

La ventana **Recomendaciones** mostrará una lista de recomendaciones, si se ha detectado alguna.

:::image type="content" source="./media/concepts-performance-recommendations/performance-recommendations-result.png" alt-text="Nueva página de Recomendaciones de rendimiento":::

Las recomendaciones no se aplican automáticamente. Para aplicar la recomendación, copie el texto de la consulta y ejecútelo desde el cliente que prefiera. No olvide probar y supervisar para evaluar la recomendación. 

## <a name="recommendation-types"></a>Tipos de recomendaciones

Actualmente, se admiten dos tipos de recomendaciones: *Crear índice* y *Quitar índice*.

### <a name="create-index-recommendations"></a>Recomendaciones para crear índice
Las recomendaciones de *Crear índice* sugieren nuevos índices para acelerar las consultas que se ejecutan con mayor frecuencia o que consumen mucho tiempo en la carga de trabajo. Este tipo de recomendación requiere que [Almacén de consultas](concepts-query-store.md) esté habilitado. Almacén de consultas recopila información de consultas y proporciona estadísticas detalladas de tiempo de ejecución y frecuencia de las consultas, que el análisis usa para hacer la recomendación.

### <a name="drop-index-recommendations"></a>Recomendaciones para quitar índice
Además de detectar índices que faltan, Azure Database for PostgreSQL analiza el rendimiento de los índices existentes. Si un índice se usa muy poco o es redundante, el analizador recomienda quitarlo.

## <a name="considerations"></a>Consideraciones
* Las recomendaciones de rendimiento no están disponibles para las [réplicas de lectura](concepts-read-replicas.md).
## <a name="next-steps"></a>Pasos siguientes
- Más información acerca de la [supervisión y ajuste](concepts-monitoring.md) en Azure Database for PostgreSQL.

