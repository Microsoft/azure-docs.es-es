---
title: Gobernanza del estado de copia de seguridad mediante el Centro de copias de seguridad
description: Aprenda a gobernar el entorno de Azure para asegurarse de que, desde la perspectiva de la copia de seguridad, todos los recursos son compatibles con el Centro de copias de seguridad.
ms.topic: conceptual
ms.date: 09/01/2020
ms.openlocfilehash: 009ee461b0372a3fb73ffb3b0ee5151b77bd8ef8
ms.sourcegitcommit: bdd5c76457b0f0504f4f679a316b959dcfabf1ef
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/22/2020
ms.locfileid: "90993821"
---
# <a name="govern-your-backup-estate-using-backup-center"></a>Gobernanza del estado de copia de seguridad mediante el Centro de copias de seguridad

El Centro de copias de seguridad le ayuda a regular el entorno de Azure para garantizar que todos los recursos son compatibles desde la perspectiva de la copia de seguridad. A continuación, se muestran algunas de las funcionalidades de gobernanza del Centro de copias de seguridad:

* Visualización y asignación de directivas de Azure para copia de seguridad

* Vea todos los orígenes de información que no se han configurado para la copia de seguridad.

## <a name="supported-scenarios"></a>Escenarios admitidos

* Consulte la [matriz de compatibilidad](backup-center-support-matrix.md) para obtener una lista detallada de escenarios admitidos y no admitidos.

## <a name="azure-policies-for-backup"></a>Directivas de Azure para copia de seguridad

Para ver todas las [directivas de Azure](https://docs.microsoft.com/azure/governance/policy/overview) que están disponibles para la copia de seguridad, seleccione el elemento de menú **Azure Policies for Backup** (Directivas de Azure para copia de seguridad). Se mostrarán todas las [definiciones de directivas de Azure para copia de seguridad](policy-reference.md) integradas y personalizadas que están disponibles para su asignación a sus suscripciones y grupos de recursos.

La selección de cualquiera de las definiciones le permite [asignar la directiva](https://docs.microsoft.com/azure/governance/policy/tutorials/create-and-manage#assign-a-policy) a un ámbito.

![Selección de definiciones de Azure Policy](./media/backup-center-govern-environment/azure-policy-definitions.png)

## <a name="protectable-datasources"></a>Orígenes de datos que se pueden proteger

Al seleccionar el elemento de menú **Protectable Datasources** (Orígenes de datos que se pueden proteger), podrá ver todos los orígenes de datos que no se han configurado para la copia de seguridad. Puede filtrar la lista por suscripción, grupo de recursos, ubicación, tipo y etiquetas del origen de datos. Cuando haya identificado un origen de datos del que necesite hacer una copia de seguridad, puede hacer clic con el botón derecho en el elemento de cuadrícula correspondiente y seleccionar **Copia de seguridad** para configurar la copia de seguridad del recurso.

![Menú de orígenes de datos que se pueden proteger](./media/backup-center-govern-environment/protectable-datasources.png)

## <a name="next-steps"></a>Pasos siguientes

* [Supervisión y funcionamiento de las copias de seguridad](backup-center-monitor-operate.md)
* [Acciones con el Centro de copias de seguridad](backup-center-actions.md)
* [Obtención de conclusiones sobre las copias de seguridad](backup-center-obtain-insights.md)
