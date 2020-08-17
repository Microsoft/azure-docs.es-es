---
title: Creación de una valoración de AVS con la herramienta de evaluación de servidores de Azure Migrate | Microsoft Docs
description: En este artículo se describe cómo crear una valoración de AVS con la herramienta de evaluación de servidores de Azure Migrate.
author: rashi-ms
ms.service: azure-migrate
ms.topic: article
ms.date: 06/26/2020
ms.author: rajosh
ms.openlocfilehash: b2917c10e13f110d7ac9784da16a10fc61eb9298
ms.sourcegitcommit: dccb85aed33d9251048024faf7ef23c94d695145
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/28/2020
ms.locfileid: "87288867"
---
# <a name="create-an-azure-vmware-solution-avs-assessment"></a>Creación de una valoración de Azure VMware Solution (AVS)

En este artículo se describe cómo crear una valoración de Azure VMware Solution (AVS) para máquinas virtuales locales de VMware con Azure Migrate: Server Assessment.

[Azure Migrate](migrate-services-overview.md) le ayuda a migrar a Azure. Azure Migrate proporciona un centro principal para realizar el seguimiento de la detección, valoración y migración de infraestructuras, aplicaciones y datos locales a Azure. El centro proporciona herramientas de Azure para la valoración y migración, así como ofertas de proveedores de software independientes (ISV) externos.

## <a name="before-you-start"></a>Antes de comenzar

- Asegúrese de que ha [creado](how-to-add-tool-first-time.md) un proyecto de Azure Migrate.
- Si ya ha creado un proyecto, asegúrese de que ha [agregado](how-to-assess.md) la herramienta Azure Migrate: Server Assessment.
- Para crear una evaluación, debe configurar un dispositivo con Azure Migrate para [VMware](how-to-set-up-appliance-vmware.md), que detecta las máquinas locales y envía metadatos y datos de rendimiento a Azure Migrate: Server Assessment. [Más información](migrate-appliance.md).
- También podría [importar los metadatos de servidor](tutorial-assess-import.md) en formato de valores separados por comas (CSV).


## <a name="azure-vmware-solution-avs-assessment-overview"></a>Información general sobre la valoración de Azure VMware Solution (AVS)

Con Azure Migrate: Server Assessment se pueden crear dos Server Assessment.

**Tipo de evaluación** | **Detalles**
--- | --- 
**MV de Azure** | Evaluaciones para la migración de los servidores locales a máquinas virtuales de Azure. <br/><br/> Puede evaluar las [máquinas virtuales VMware](how-to-set-up-appliance-vmware.md), las [máquinas virtuales Hyper-V](how-to-set-up-appliance-hyper-v.md) y los [servidores físicos](how-to-set-up-appliance-physical.md) locales para la migración a Azure con este tipo de evaluación. [Más información](concepts-assessment-calculation.md)
**Azure VMware Solution (AVS)** | Evaluaciones para la migración de los servidores locales a [Azure VMware Solution (AVS)](../azure-vmware/introduction.md). <br/><br/> Puede evaluar las [máquinas virtuales VMware](how-to-set-up-appliance-vmware.md) locales para la migración a Azure VMware Solution (AVS) con este tipo de evaluación. [Más información](concepts-azure-vmware-solution-assessment-calculation.md)

> [!NOTE]
> La valoración de Azure VMware Solution (AVS) se encuentra actualmente en versión preliminar y se puede crear solo para máquinas virtuales VMware.


Existen dos tipos de criterios de tamaño que puede usar para crear valoraciones de Azure VMware Solution (AVS):

**Valoración** | **Detalles** | **Data**
--- | --- | ---
**Basada en el rendimiento** | Evaluaciones basadas en los datos de rendimiento recopilados de las máquinas virtuales locales. | **Tamaño del nodo recomendado**: se basa en los datos de uso de la CPU y de la memoria junto con el tipo de nodo, el tipo de almacenamiento y la configuración de FTT seleccionados para la evaluación.
**Como local** | Evaluaciones que se basan en el tamaño local. | **Tamaño del nodo recomendado**: se basa en el tamaño de la máquina virtual local junto con el tipo de nodo, el tipo de almacenamiento y la configuración de FTT seleccionados para la evaluación.


## <a name="run-an-azure-vmware-solution-avs-assessment"></a>Ejecución de una valoración de Azure VMware Solution (AVS)

Ejecute una valoración de Azure VMware Solution (AVS) como se indica a continuación:

1. Consulte los [procedimientos recomendados](best-practices-assessment.md) para crear evaluaciones.

2. En la pestaña **Servidores**, en el icono **Azure Migrate: Server Assessment**, haga clic en **Evaluar**.

    ![Evaluar](./media/how-to-create-assessment/assess.png)

3. En **Evaluar los servidores**, seleccione el tipo de evaluación como "Azure VMware Solution (AVS)", seleccione el origen de detección y especifique el nombre de la evaluación.

    ![Aspectos básicos de la evaluación](./media/how-to-create-avs-assessment/assess-servers-avs.png)

4. Haga clic en **View all** (Ver todo) para revisar la configuración de la valoración.

    ![Propiedades de la valoración de AVS](./media/how-to-create-avs-assessment/avs-view-all.png)

5. Haga clic en **Siguiente** para **seleccionar las máquinas que se van a evaluar**. En **Seleccionar o crear un grupo**, seleccione **Crear nuevo** y especifique un nombre de grupo. Un grupo recopila una o varias máquinas virtuales para su evaluación.

6. En **Agregar máquinas al grupo**, seleccione las máquinas virtuales que se van a agregar al grupo.

7. Haga clic en **Siguiente** para **Revisar y crear evaluación** para revisar los detalles de la evaluación.

8. Haga clic en **Crear evaluación** para crear el grupo y realizar la evaluación.

    ![Creación de una valoración de AVS](./media/how-to-create-avs-assessment/avs-assessment-create.png)

9. Una vez creada la evaluación, se puede ver en **Servidores** > **Azure Migrate: Server Assessment** > **Evaluaciones**.

10. Haga clic en **Exportar la evaluación** para descargarla como un archivo de Excel.


## <a name="review-an-azure-vmware-solution-avs-assessment"></a>Revise una valoración de Azure VMware Solution (AVS)

Una valoración de Azure VMware Solution (AVS) describe:

- **Preparación de Azure VMware Solution (AVS)** : si las máquinas virtuales locales son adecuadas para la migración a Azure VMware Solution (AVS).
- **Número de nodos de AVS**: número estimado de nodos de AVS necesarios para ejecutar las máquinas virtuales.
- **Uso en los nodos de AVS**: uso previsto de CPU, memoria y almacenamiento en todos los nodos.
- **Estimación del costo mensual**: los costos mensuales estimados de todos los nodos de Azure VMware Solution (AVS) que ejecutan las máquinas virtuales locales.


### <a name="view-an-assessment"></a>Visualización de una evaluación

1. En **Objetivos de migración** >  **Servidores**, haga clic en **Evaluaciones** en **Azure Migrate: Server Assessment**.

2. En **Evaluaciones**, haga clic en una evaluación para abrirla.

    ![Resumen de la valoración de AVS](./media/how-to-create-avs-assessment/avs-assessment-summary.png)

### <a name="review-azure-vmware-solution-avs-readiness"></a>Revisión de la preparación de Azure VMware Solution (AVS)

1. En **Preparación para Azure**, compruebe si las máquinas virtuales están listas para su migración a AVS.

2. Revise el estado de la máquina virtual:
    - **Listo para AVS**: la máquina se puede migrar como está a Azure (AVS) sin realizar ningún cambio. Se iniciará en AVS con soporte técnico de AVS completo.
    - **Preparado con condiciones**: Es posible que la máquina virtual tenga problemas de compatibilidad con la versión actual de vSphere y que, además, requiera herramientas de VMware y otras opciones de configuración antes de que pueda conseguirse la funcionalidad completa de la máquina virtual en AVS.
    - **No está listo para AVS**: la máquina virtual no se iniciará en AVS. Por ejemplo, si la máquina virtual VMware local tiene un dispositivo externo conectado, como un CD-ROM, se producirá un error en la operación de VMotion (si se usa VMware VMotion).
    - **Preparación desconocida**: Azure Migrate no ha podido encontrar la preparación de la máquina debido a una recopilación insuficiente de metadatos en el entorno local.

3. Revise la herramienta sugerida:
    - **VMware HCX o Enterprise** En el caso de las máquinas de VMware, la solución Hybrid Cloud Extension (HCX) de VMWare es la herramienta de migración sugerida para migrar su carga de trabajo en el entorno local a la nube privada de Azure VMware Solution (AVS). [Más información](../azure-vmware/hybrid-cloud-extension-installation.md).
    - **Desconocido**: En el caso de las máquinas importadas mediante un archivo CSV, se desconoce la herramienta de migración predeterminada. Sin embargo, para las máquinas de VMware, se recomienda usar la solución Hybrid Cloud Extension (HCX) de VMware. 

4. Haga clic en un estado de **Preparación para AVS**. Puede ver los detalles de la preparación de la máquina virtual y explorar en profundidad los detalles de esta, entre los que se incluye la configuración de proceso, almacenamiento y red.



### <a name="review-cost-details"></a>Revisión de los datos de costo

Esta vista muestra el costo estimado de ejecución de máquinas virtuales en Azure VMware Solution (AVS).

1. Revise los costos totales mensuales. Los costos se agregan para todas las máquinas virtuales del grupo evaluado. 

    - Las estimaciones de los costos se basan en el número de nodos de AVS necesarios teniendo en cuenta los requisitos de recursos de todas las máquinas virtuales en total.
    - Como los precios de Azure VMware Solution (AVS) son por nodo, el costo total no tiene costo de proceso ni distribución de costos de almacenamiento.
    - La estimación de costos es para ejecutar las máquinas virtuales locales en AVS. Azure Migrate Server Assessment no tiene en cuenta los costos de PaaS o SaaS.
    
2. Puede revisar las estimaciones del costo de almacenamiento mensual. Esta vista muestra los costos de almacenamiento agregados del grupo evaluado, divididos por los diferentes tipos de discos de almacenamiento.

3. Puede explorar en profundidad para ver los detalles de cada una de las máquinas virtuales.


### <a name="review-confidence-rating"></a>Examen de la clasificación de confianza

Cuando se realizan valoraciones basadas en el rendimiento, se asigna una clasificación de confianza a la evaluación.

![Clasificación de confianza](./media/how-to-create-assessment/confidence-rating.png)

- Se concede una clasificación que oscila entre 1 estrella (la más baja) y 5 estrellas (la más alta).
- La clasificación de confianza sirve de ayuda para calcular la confiabilidad de las recomendaciones de tamaño que proporciona la evaluación.
- La clasificación de confianza se basa en la disponibilidad de los puntos de datos necesarios para calcular tal evaluación.
- Para el ajuste de tamaño según el rendimiento, las valoraciones de AVS en Server Assessment necesitan los datos de uso de la CPU y la memoria de la máquina virtual. Los siguientes datos de rendimiento se recopilan, pero no se usan en las recomendaciones de ajuste de tamaño de las valoraciones de AVS:
  - Los datos de IOPS y rendimiento de cada disco conectado a la máquina virtual.
  - La entrada o la salida de red de cada adaptador de red conectado a una máquina virtual.

Estas son las posibles clasificaciones de confianza de una evaluación.

**Disponibilidad del punto de datos** | **Clasificación de confianza**
--- | ---
0 % - 20 % | 1 estrella
21 % - 40 % | 2 estrellas
41 % - 60 % | 3 estrellas
61 % - 80 % | 4 estrellas
81 % - 100 % | 5 estrellas

[Más información](concepts-azure-vmware-solution-assessment-calculation.md) sobre los datos de rendimiento 


## <a name="next-steps"></a>Pasos siguientes

- Aprenda a usar la [asignación de dependencias de máquina](how-to-create-group-machine-dependencies.md) para crear grupos de confianza alta.
- [Obtenga más información](concepts-azure-vmware-solution-assessment-calculation.md) sobre cómo se calculan las valoraciones de AVS.
