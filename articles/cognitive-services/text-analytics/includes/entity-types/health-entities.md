---
title: Entidades con nombre para la asistencia sanitaria
titleSuffix: Azure Cognitive Services
services: cognitive-services
author: aahill
manager: nitinme
ms.service: cognitive-services
ms.topic: include
ms.date: 07/28/2020
ms.author: aahi
ms.openlocfilehash: 6880391fb54791fe5f597de2305d24f8c0e47ec6
ms.sourcegitcommit: b8702065338fc1ed81bfed082650b5b58234a702
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/11/2020
ms.locfileid: "88122473"
---
## <a name="health-entity-categories"></a>Categorías de la entidad de salud:

[Text Analytics for Health](../../how-tos/text-analytics-for-health.md) devuelve las categorías de entidad siguientes.  Tenga en cuenta que solo se admite texto en inglés en versión preliminar de contenedor y solo se proporciona una versión de modelo en cada imagen de contenedor.

### <a name="named-entity-recognition"></a>Reconocimiento de entidades con nombre

|Category  |Descripción   |
|----------|--------------|
| Age | Edades. Por ejemplo, *30 años*. |
| AdministrativeEvent | Evento administrativo. |
| BodyStructure | Partes del cuerpo humano, incluidos los órganos y otras estructuras. Por ejemplo, *brazo*o *corazón*. | 
| CareEnvironment | El entorno donde se administra la atención o el tratamiento. Por ejemplo, *sala de emergencias*. | 
| ConditionQualifier | Niveles de condición. Por ejemplo, *leve*, *extendido* o *difuso*. | 
| Diagnóstico | Enfermedades médicas. Por ejemplo, *hipertensión*. | 
| Dirección | Direcciones. Por ejemplo, *izquierda* o *anterior*. | 
| Dosage | Tamaño o cantidad de un medicamento. Por ejemplo, *25 mg*.  | 
| ExaminationName | Método o procedimiento de examen. Por ejemplo, *radiografía*. | 
| RelationalOperator | Operador que define una relación entre dos entidades. Por ejemplo, *menor que* o `>=`.  | 
| MeasurementUnit | Unidad de medida (como un porcentaje). | 
| MeasurementValue | Valor numérico de una unidad de medida. | 
| FamilyRelation | Parentesco. Por ejemplo, *hermana*.  | 
| Frecuencia | Frecuencias.   | 
| Sexo | Géneros. | 
| Gene | Una entidad génica, como *TP53*.   | 
| HealthcareProfession | Método de administración de medicamentos. Por ejemplo *administración por vía oral*. | 
| MedicationClass | Clases de medicación. Por ejemplo, *antibióticos*.  | 
| MedicationForm | Forma del medicamento. Por ejemplo, *cápsula*. | 
| MedicationName  | Medicamentos genéricos y de marca. Por ejemplo, *ibuprofeno*. | 
| MedicationRoute | Método de administración de medicamentos. Por ejemplo *administración por vía oral*. | 
| SymptomOrSign  | Síntomas de la enfermedad. Por ejemplo, *dolor de garganta*. | 
| Time | Tiempo. Por ejemplo, *8 años* o *2:30 esta madrugada*. |
| TreatmentName  | Nombres del tratamiento. Por ejemplo, *terapia*. | 
| Variant | Variante genética de la entidad del gen. | 

### <a name="relation-extraction"></a>Extracción de relaciones

La extracción de relaciones identifica conexiones significativas entre los conceptos mencionados en el texto. Por ejemplo, una relación de "tiempo de enfermedad" se encuentra asociando un nombre de enfermedad con un tiempo. Text Analytics for Health puede identificar las siguientes relaciones:

|Category  |Descripción   |
|----------|--------------|
| DirectionOfBodyStructure | Dirección de una estructura corporal. |
| DirectionOfCondition | Dirección de una enfermedad. |
| DirectionOfExamination | Dirección de un examen. |
| DirectionOfTreatment | Dirección de un tratamiento. |
| TimeOfCondition | Hora asociada a la aparición de una enfermedad. |
| QualifierOfCondition | Calificador asociado para una enfermedad. |
| DosageOfMedication | Dosis de medicamento. |
| FormOfMedication | Forma de medicamento. |
| RouteOfMedication | Ruta o modo de consumo de un medicamento. Por ejemplo, *oral*. |
| FrequencyOfMedication | Frecuencia a la que se consume un medicamento. | 
| ValueOfCondition | Valor numérico asociado a una enfermedad. |
| UnitOfCondition | Unidad (como tiempo) asociada a una enfermedad. |
| TimeOfMedication | Hora a la que se consume un medicamento. |
| TimeOfTreatment | Hora a la que se administra el tratamiento. | 
| FrequencyOfTreatment | Frecuencia a la que se administra el tratamiento. |
| ValueOfExamination | Valor numérico asociado a un examen. | 
| UnitOfExamination | Unidad (como porcentaje) asociada a un examen. |
| RelationOfExamination | Relación entre una entidad y un examen. | 
| TimeOfExamination | Tiempo asociado a un examen. |
| Abreviatura | Abreviatura.  | 
