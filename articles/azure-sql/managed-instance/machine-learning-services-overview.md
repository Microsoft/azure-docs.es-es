---
title: Machine Learning Services de Instancia administrada de Azure SQL (versión preliminar)
description: En este artículo se proporciona información general acerca de Machine Learning Services en Instancia administrada de Azure SQL.
services: sql-database
ms.service: sql-managed-instance
ms.subservice: machine-learning
ms.custom: ''
ms.devlang: ''
ms.topic: conceptual
author: garyericson
ms.author: garye
ms.reviewer: carlrab, davidph
manager: cgronlun
ms.date: 06/03/2020
ms.openlocfilehash: ad25a9a62757e1e031ce8b93a44e6f2ada4d9964
ms.sourcegitcommit: 56cbd6d97cb52e61ceb6d3894abe1977713354d9
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/20/2020
ms.locfileid: "88689514"
---
# <a name="machine-learning-services-in-azure-sql-managed-instance-preview"></a>Machine Learning Services de Instancia administrada de Azure SQL (versión preliminar)

Machine Learning Services es una característica de Instancia administrada de Azure SQL (versión preliminar) que proporciona aprendizaje automático en la base de datos, con compatibilidad con scripts de Python y R. Incluye paquetes de R y de Python de Microsoft para el análisis predictivos de alto rendimiento y el aprendizaje automático. Los datos relacionales pueden usarse en scripts mediante procedimientos almacenados, scripts de T-SQL que contienen instrucciones en R o Python o código R o Python que contiene T-SQL.

> [!IMPORTANT]
> Machine Learning Services es una característica de Instancia administrada de Azure SQL que se encuentra actualmente en versión preliminar pública.
> Esta funcionalidad en versión preliminar está inicialmente disponible en un número limitado de regiones de EE. UU., Asia, Europa y Australia, y más adelante se agregarán nuevas regiones.
>
> Esta versión preliminar se ofrece sin Acuerdo de Nivel de Servicio y no se recomienda para cargas de trabajo de producción. Es posible que algunas características no sean compatibles o que tengan sus funcionalidades limitadas.
> Para más información, consulte [Términos de uso complementarios de las Versiones Preliminares de Microsoft Azure](https://azure.microsoft.com/support/legal/preview-supplemental-terms/).
>
> [Suscríbase a la versión preliminar](#signup) a continuación.

## <a name="what-is-machine-learning-services"></a>¿Qué es Machine Learning Services?

Machine Learning Services en Instancia administrada de Azure SQL permite ejecutar scripts de Python y R en la base de datos. Se puede usar para preparar y limpiar los datos, realizar ingeniería de características, y entrenar, evaluar e implementar modelos de aprendizaje automático en una base de datos. La característica ejecuta los scripts donde residen los datos y elimina la transferencia de los datos a otro servidor a través de la red.

Use Machine Learning Services con compatibilidad con R/Python en Instancia administrada de Azure SQL para:

- **Ejecutar scripts de R y Python para la preparación de datos y el procesamiento de datos de uso general**: ahora puede traer scripts de R/Python a Instancia administrada de Azure SQL donde residen los datos, en lugar de tener que trasladar los datos a otro servidor para ejecutar scripts de R y Python. Ya no será necesario el movimiento de datos, con lo que se evitarán problemas relacionados con la latencia, la seguridad y el cumplimiento.

- **Entrenar modelos de aprendizaje automático en la base de datos**: puede entrenar modelos con cualquier algoritmo de código abierto. Puede escalar fácilmente el entrenamiento en todo el conjunto de datos, en lugar de recurrir a conjuntos de datos de ejemplo extraídos de la base de datos.

- **Implementar los modelos y los scripts en producción en procedimientos almacenados**: los scripts y los modelos entrenados se pueden operacionalizar simplemente mediante su inserción en procedimientos almacenados de T-SQL. Las aplicaciones que se conectan a Instancia administrada de Azure SQL pueden aprovechar las predicciones y la inteligencia de estos modelos mediante una llamada a un procedimiento almacenado. También puede usar la función de PREDICT nativa de T-SQL para operacionalizar modelos con el fin de obtener una puntuación rápida en escenarios de puntuación en tiempo real sumamente simultáneos.

Machine Learning Services incluye las distribuciones base de Python y R. Se pueden instalar y usar marcos y paquetes de código abierto, como PyTorch, TensorFlow y scikit-learn, además de los paquetes de Microsoft [revoscalepy](https://docs.microsoft.com/sql/advanced-analytics/python/ref-py-revoscalepy) y [microsoftml](https://docs.microsoft.com/sql/advanced-analytics/python/ref-py-microsoftml) para Python, y [RevoScaleR](https://docs.microsoft.com/sql/advanced-analytics/r/ref-r-revoscaler), [MicrosoftML](https://docs.microsoft.com/sql/advanced-analytics/r/ref-r-microsoftml), [olapr](https://docs.microsoft.com/sql/advanced-analytics/r/ref-r-olapr) y [sqlrutils](https://docs.microsoft.com/sql/advanced-analytics/r/ref-r-sqlrutils) para R.

<a name="signup"></a>

## <a name="sign-up-for-the-preview"></a>Suscríbase a la vista previa

Esta versión preliminar pública limitada está sujeta a las [Condiciones de la versión preliminar de Azure](https://azure.microsoft.com/support/legal/preview-supplemental-terms/). 

Si le interesa unirse al programa de versión preliminar y acepta estas condiciones, puede solicitar la inscripción mediante la creación de una incidencia de soporte técnico de Azure en [ **https://azure.microsoft.com/support/create-ticket/** ](https://azure.microsoft.com/support/create-ticket/). 

1. Seleccione las opciones siguientes:
   - Tipo de problema: **Técnico**
   - Suscripción: *seleccione su suscripción*.
   - Servicio: **Instancia administrada de SQL Database**
   - Resumen: *proporcione una breve descripción de la solicitud*.
   - Tipo de problema: **Machine Learning Services para SQL Managed Instance (versión preliminar)**
   - Subtipo de problema: **Other issue or "How To" questions** (Otra incidencia o preguntas de procedimientos)

1. Haga clic en **Siguiente: Soluciones**.

1. Lea la información sobre la versión preliminar y, luego, haga clic en **Detalles**.

1. En **Descripción**, escriba los detalles de la solicitud, como el nombre del servidor lógico, la región y el identificador de la suscripción que le gustaría inscribir en la versión preliminar. Escriba otros detalles según corresponda.

1. Cuando haya terminado, haga clic en **Siguiente: Revisar y crear** y, luego, en **Crear**.

Cuando ya esté inscrito en el programa, Microsoft le incorporará a la versión preliminar pública y habilitará Machine Learning Services para su base de datos existente o para una nueva.

No se recomienda el uso de Machine Learning Services con Instancia administrada de SQL para cargas de trabajo de producción durante la versión preliminar pública.

## <a name="next-steps"></a>Pasos siguientes

- Consulte las [principales diferencias con Machine Learning Services en SQL Server](machine-learning-services-differences.md).
- Para obtener información sobre cómo usar Python en Machine Learning Services, vea [Ejecución de scripts de Python](https://docs.microsoft.com/sql/machine-learning/tutorials/quickstart-python-create-script?context=/azure/azure-sql/managed-instance/context/ml-context&view=sql-server-ver15).
- Para obtener información sobre cómo usar R en Machine Learning Services, vea [Ejecución de scripts de R](https://docs.microsoft.com/sql/machine-learning/tutorials/quickstart-r-create-script?context=/azure/azure-sql/managed-instance/context/ml-context&view=sql-server-ver15).
- Para obtener más información sobre el aprendizaje automático en otras plataformas de SQL, consulte la [documentación del aprendizaje automático de SQL](https://docs.microsoft.com/sql/machine-learning/).
