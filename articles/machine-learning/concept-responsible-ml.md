---
title: Descripción del aprendizaje automático responsable (versión preliminar)
titleSuffix: Azure Machine Learning
description: Conozca en qué consiste el aprendizaje automático y cómo usarlo en Azure Machine Learning
services: machine-learning
ms.service: machine-learning
ms.subservice: core
ms.topic: conceptual
ms.author: luquinta
author: luisquintanilla
ms.date: 08/05/2020
ms.openlocfilehash: 689b90fc1f45faad72640f47e5eebe936d2dc8b7
ms.sourcegitcommit: 2ff0d073607bc746ffc638a84bb026d1705e543e
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/06/2020
ms.locfileid: "87829397"
---
# <a name="what-is-responsible-machine-learning-preview"></a>Descripción del aprendizaje automático responsable (versión preliminar)

En este artículo va a conocer lo que es el aprendizaje automático (ML) responsable y las formas en que se puede poner en práctica con Azure Machine Learning.

## <a name="responsible-machine-learning-principles"></a>Principios del aprendizaje automático responsable

A lo largo del desarrollo y el uso de sistemas de AI, la confianza debe ser el centro de todo. Debe confiar en la plataforma, el proceso y los modelos. En Microsoft, el aprendizaje automático responsable engloba los siguientes valores y principios:

- Descripción de los modelos de Machine Learning
  - Interpretación y explicación del comportamiento del modelo
  - Evaluación y mitigación de la parcialidad del modelo
- Protección de los usuarios y sus datos
  - Prevención de la exposición de datos con privacidad diferencial
  - Trabajo con datos cifrados mediante cifrado homomórfico
- Control del proceso de aprendizaje automático de un extremo a otro
  - Documentación del ciclo de vida del aprendizaje automático con hojas de datos

:::image type="content" source="media/concept-responsible-ml/responsible-ml-pillars.png" alt-text="Pilares del aprendizaje automático responsable: interpretación, privacidad diferencial, cifrado homomórfico, registro de auditoría - Azure Machine Learning":::

La inteligencia artificial y los sistemas autónomos están cada vez más integrados en la sociedad, por lo que es importante hacer un esfuerzo para prever y mitigar las consecuencias no deseadas de estas tecnologías.

## <a name="interpret-and-explain-model-behavior"></a>Interpretación y explicación del comportamiento del modelo

Los sistemas de caja opaca o aquellos que son difíciles de explicar pueden suponer un problema, ya que las partes interesadas como desarrolladores de sistemas, reguladores, usuarios y responsables de negocios, pueden no entender con claridad por qué estos sistemas toman ciertas decisiones. Algunos sistemas de IA son más fáciles de explicar que otros y, a veces, existe cierto equilibrio entre un sistema que cuenta con una precisión mayor y otro más descriptivo.

Para compilar sistemas de AI interpretables, use [InterpretML](https://github.com/interpretml/interpret), que es un paquete de código abierto que ha compilado Microsoft. [InterpretML se puede usar en Azure Machine Learning](how-to-machine-learning-interpretability.md) para [interpretar y explicar los modelos de Machine Learning](how-to-machine-learning-interpretability-aml.md), incluidos los [modelos automatizados de Machine Learning](how-to-machine-learning-interpretability-automl.md).

## <a name="mitigate-fairness-in-machine-learning-models"></a>Mitigación de la equidad en los modelos de Machine Learning

A medida que los sistemas de IA se involucran cada vez más en el día a día de nuestra sociedad, es extremadamente importante que estos sistemas funcionen bien para que puedan proporcionar buenos resultados a todos los usuarios.

La parcialidad de los sistemas de IA puede dar lugar a las siguientes consecuencias no deseadas:

- Pérdida de oportunidades, recursos o información de usuarios.
- Se refuerzan los sesgos y los estereotipos.

Muchos aspectos de la imparcialidad no se pueden capturar ni representar mediante métricas. Existen herramientas y prácticas que pueden mejorar la imparcialidad en el diseño y desarrollo de sistemas de IA.

Dos pasos clave para reducir la parcialidad en los sistemas de IA son la valoración y la mitigación. Le recomendamos que use [FairLearn](https://github.com/fairlearn/fairlearn), un paquete de código abierto que puede valorar y mitigar la posible parcialidad de los sistemas de IA. Para obtener más información sobre la imparcialidad y el paquete FairLearn, consulte [el artículo sobre la imparcialidad en ML](./concept-fairness-ml.md).

## <a name="prevent-data-exposure-with-differential-privacy"></a>Prevención de la exposición de datos con privacidad diferencial

Cuando se usan datos para el análisis, es importante que los estos sean privados y confidenciales a lo largo del proceso. La privacidad diferencial es un conjunto de sistemas y prácticas que ayudan a mantener los datos de los usuarios seguros y privados.

En escenarios tradicionales, los datos sin procesar se almacenan en archivos y bases de datos. Cuando los usuarios analizan datos, suelen usar los datos sin procesar. Esto supone un problema, ya que se podría infringir la privacidad de los usuarios. La privacidad diferencial intenta solucionar este problema agregando "ruido" o aleatoriedad a los datos para que los usuarios no puedan identificar puntos de datos individuales.

La implementación de sistemas privados de forma diferencial es difícil. [WhiteNoise](https://github.com/opendifferentialprivacy/whitenoise-core) es un proyecto de código abierto que contiene distintos componentes para la creación de sistemas privados globales de forma diferencial. Para obtener más información sobre la privacidad diferencial y el proyecto WhiteNoise, consulte el artículo para [mantener la privacidad de los datos mediante la privacidad diferencial y WhiteNoise](./concept-differential-privacy.md).

> [!NOTE]
> Tenga en cuenta que estamos cambiando el nombre del kit de herramientas y que presentaremos el nombre nuevo en las próximas semanas. 

## <a name="work-on-encrypted-data-with-homomorphic-encryption"></a>Trabajo con datos cifrados mediante cifrado homomórfico

En las soluciones tradicionales de cálculo y almacenamiento en la nube, esta necesita acceso sin cifrar a los datos del cliente para realizar cálculos con ellos. Este acceso expone los datos a los operadores de nube. La privacidad de los datos depende de las directivas de control de acceso implementadas por la nube y en las que el cliente confía.

El cifrado homomórfico permite realizar cálculos en datos cifrados sin necesidad de tener acceso a una clave secreta (descifrado). Los resultados de los cálculos están cifrados y solo los puede revelar el propietario de la clave secreta. Con el cifrado homomórfico, los operadores de nube nunca tienen acceso sin cifrar a los datos que almacenan y con los que realizan cálculos. Los cálculos se realizan directamente en los datos cifrados. La privacidad de los datos depende de criptografía de vanguardia y el propietario de los datos controla todas las salidas de información. Para obtener más información sobre el cifrado homomórfico en Microsoft, vea [Microsoft Research](https://www.microsoft.com/research/project/homomorphic-encryption/).

Para empezar a trabajar con el cifrado homomórfico en Azure Machine Learning, use los enlaces de Python de [inferencia cifrada](https://pypi.org/project/encrypted-inference/) para [Microsoft SEAL](https://github.com/microsoft/SEAL). Microsoft SEAL es una biblioteca de cifrado homomórfico de código abierto que permite realizar sumas y multiplicaciones con enteros cifrados o números reales. Para obtener más información sobre Microsoft SEAL, vea el [Centro de arquitectura de Azure](https://docs.microsoft.com/azure/architecture/solution-ideas/articles/homomorphic-encryption-seal) o la [página del proyecto de Microsoft Research](https://www.microsoft.com/research/project/microsoft-seal/).

Vea el ejemplo siguiente para obtener información sobre la [Implementación de un servicio web de inferencia cifrada en Azure Machine Learning](how-to-homomorphic-encryption-seal.md).

## <a name="document-the-machine-learning-lifecycle-with-datasheets"></a>Documentación del ciclo de vida del aprendizaje automático con hojas de datos

Documentar la información adecuada en el proceso de aprendizaje automático es indispensable para poder tomar decisiones responsables en cada fase. Las hojas de datos son una forma de documentar los recursos de aprendizaje automático que se usan y se crean como parte del ciclo de vida del aprendizaje automático.

Los modelos suelen considerarse como "cajas opacas" y, a menudo, hay poca información sobre ellos. Dado que los sistemas de aprendizaje automático están cada vez más generalizados y se usan en la toma de decisiones, el uso de hojas de datos es un paso más hacia el desarrollo de sistemas de aprendizaje automático más responsables.

Aquí tiene información del modelo que tal vez quiera documentar como parte de la hoja de datos:

- Uso previsto
- Arquitectura del modelo
- Datos de aprendizaje usados
- Datos de evaluación usados
- Métricas de rendimiento del modelo de entrenamiento
- Información de imparcialidad

Consulte el ejemplo siguiente para obtener información sobre cómo usar el SDK de Azure Machine Learning para implementar las [hojas de datos para modelos](https://github.com/microsoft/MLOps/blob/master/pytorch_with_datasheet/model_with_datasheet.ipynb).

## <a name="additional-resources"></a>Recursos adicionales

- Para obtener más información, vea [Innovación responsable: Un kit de herramientas de procedimientos recomendados](https://docs.microsoft.com/azure/architecture/guide/responsible-innovation/) a fin de conocer procedimientos recomendados.
- Obtenga más información sobre el conjunto de directrices de [ABOUT ML](https://www.partnershiponai.org/about-ml/) referentes a la documentación del sistema de aprendizaje automático.
