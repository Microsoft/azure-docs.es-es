---
title: Evaluación de grandes cantidades de máquinas virtuales de VMware para la migración a Azure con Azure Migrate
description: Describe cómo evaluar grandes cantidades de máquinas virtuales de VMware para la migración a Azure mediante el servicio Azure Migrate.
ms.topic: how-to
ms.date: 03/23/2020
ms.openlocfilehash: 6490a5448bb68dcccd61784d149e9765107400c2
ms.sourcegitcommit: d7bd8f23ff51244636e31240dc7e689f138c31f0
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/24/2020
ms.locfileid: "87171907"
---
# <a name="assess-large-numbers-of-vmware-vms-for-migration-to-azure"></a>Evaluación de grandes cantidades de VM de VMware para la migración a Azure


En este artículo se describe cómo evaluar grandes cantidades (> 1000-35 000) de máquinas virtuales de VMware locales para su migración a Azure mediante la herramienta de evaluación del servidor de Azure Migrate.

[Azure Migrate](migrate-services-overview.md) proporciona un centro de herramientas que le ayuda a detecta las aplicaciones, la infraestructura y las cargas de trabajo, a evaluarlas y a migrarlas a Microsoft Azure. Este centro incluye herramientas de Azure Migrate y ofertas de fabricantes de software independientes (ISV) de terceros. 

En este artículo aprenderá a:
> [!div class="checklist"]
> * Planificar la evaluación a escala.
> * Configurar los permisos de Azure y preparar VMware para la evaluación.
> * Crear un proyecto de Azure Migrate y crear una evaluación.
> * Revisar la evaluación mientras planea la migración.


> [!NOTE]
> Si quiere probar una prueba de concepto para evaluar un par de VM antes de realizar la evaluación a escala, siga nuestra [serie de tutoriales](tutorial-prepare-vmware.md).

## <a name="plan-for-assessment"></a>Planeación de la evaluación

Al planear la evaluación de un gran número de VM de VMware, hay un par de cosas que hay que tener en cuenta:

- **Planeación de proyectos de Azure Migrate**: Averigüe cómo implementar proyectos de Azure Migrate. Por ejemplo, si los centros de datos están en zonas geográficas diferentes, o si necesita almacenar metadatos relacionados con la detección, la evaluación o la migración en una zona geográfica diferente, es posible que necesite varios proyectos. 
- **Planear dispositivos**: Azure Migrate usa un dispositivo de Azure Migrate local, implementado como VM de VMware, para detectar continuamente VM. El dispositivo supervisa los cambios de entorno, como la incorporación de VM, discos o adaptadores de red. También envía metadatos y datos de rendimiento acerca de ellos a Azure. Debe averiguar el número de dispositivos que necesitará implementar.
- **Planear cuentas para la detección**: El dispositivo Azure Migrate usa una cuenta con acceso a vCenter Server para detectar VM con fines de evaluación y migración. Si detecta más de 10 000 máquinas virtuales, configure varias cuentas, ya que no debe existir ninguna superposición entre las máquinas virtuales detectadas en dos dispositivos cualquiera de un proyecto. 

> [!NOTE]
> Si va a configurar varios dispositivos, asegúrese de que no exista ninguna superposición entre las máquinas virtuales en las cuentas de vCenter proporcionadas. No se admiten los escenarios con detecciones con este tipo de superposición. Si más de un dispositivo detecta la misma máquina virtual, habrá duplicados en la detección y se presentarán problemas al habilitar la replicación para la máquina virtual mediante Azure Portal durante la migración del servidor.

## <a name="planning-limits"></a>Límites de planeación
 
Use los límites resumidos en esta tabla para la planeación.

**Planeamiento** | **Límites**
--- | --- 
**Proyectos de Azure Migrate** | Evalúe hasta 35 000 VM en un proyecto.
**Dispositivo con Azure Migrate** | Un dispositivo puede detectar hasta 10 000 VM en una instancia de vCenter Server.<br/> Un dispositivo solo puede conectarse a una instancia de vCenter Server.<br/> Un dispositivo solo se puede asociar con un único proyecto de Azure Migrate.<br/>  Se puede asociar cualquier número de dispositivos a un solo proyecto de Azure Migrate. <br/><br/> 
**Grupo** | Puede agregar hasta 35 000 máquinas virtuales en un solo grupo.
**Evaluación de Azure Migrate** | Puede acceder hasta a 35 000 máquinas virtuales en una única evaluación.

Teniendo en cuenta estos límites, estas son algunas implementaciones de ejemplo:


**Servidor vCenter** | **VM en el servidor** | **Recomendación** | **Acción**
---|---|---|---
Uno | < 10 000 | Un proyecto de Azure Migrate.<br/> Un dispositivo.<br/> Una cuenta de vCenter para la detección. | Configure el dispositivo y conéctese a vCenter Server con una cuenta.
Uno | > 10 000 | Un proyecto de Azure Migrate.<br/> Varios dispositivos.<br/> Varias cuentas de vCenter. | Configure el dispositivo para cada 10 000 VM.<br/><br/> Configure cuentas de vCenter y divida el inventario para limitar el acceso de una cuenta a menos de 10 000 VM.<br/> Conecte cada dispositivo a vCenter Server con una cuenta.<br/> Puede analizar las dependencias entre las máquinas que se detectan con diferentes dispositivos. <br/> <br/> Asegúrese de que no exista ninguna superposición entre las máquinas virtuales en las cuentas de vCenter proporcionadas. No se admiten los escenarios con detecciones con este tipo de superposición. Si más de un dispositivo detecta la misma máquina virtual, habrá duplicados en la detección y se presentarán problemas al habilitar la replicación para la máquina virtual mediante Azure Portal durante la migración del servidor.
Múltiple | < 10 000 |  Un proyecto de Azure Migrate.<br/> Varios dispositivos.<br/> Una cuenta de vCenter para la detección. | Configure los dispositivos y conéctese a vCenter Server con una cuenta.<br/> Puede analizar las dependencias entre las máquinas que se detectan con diferentes dispositivos.
Múltiple | > 10 000 | Un proyecto de Azure Migrate.<br/> Varios dispositivos.<br/> Varias cuentas de vCenter. | Si vCenter Server detectó menos de 10 000 VM, configure un dispositivo para cada instancia de vCenter Server.<br/><br/> Si vCenter Server detectó más de 10 000 VM, configure un dispositivo para cada 10 000 VM.<br/> Configure cuentas de vCenter y divida el inventario para limitar el acceso de una cuenta a menos de 10 000 VM.<br/> Conecte cada dispositivo a vCenter Server con una cuenta.<br/> Puede analizar las dependencias entre las máquinas que se detectan con diferentes dispositivos. <br/><br/> Asegúrese de que no exista ninguna superposición entre las máquinas virtuales en las cuentas de vCenter proporcionadas. No se admiten los escenarios con detecciones con este tipo de superposición. Si más de un dispositivo detecta la misma máquina virtual, habrá duplicados en la detección y se presentarán problemas al habilitar la replicación para la máquina virtual mediante Azure Portal durante la migración del servidor.



## <a name="plan-discovery-in-a-multi-tenant-environment"></a>Planeación de la detección en un entorno de varios inquilinos

Si está planeando un entorno de varios inquilinos, puede limitar la detección en vCenter Server.

- Puede establecer el ámbito de detección del dispositivo en un centro de datos de vCenter Server, varios clústeres o una carpeta de clústeres, varios hosts o una carpeta de hosts, o VM individuales.
- Si el entorno se comparte entre inquilinos y quiere detectar cada inquilino por separado, puede limitar el acceso a la cuenta de vCenter que el dispositivo usa para la detección. 
    - Puede que quiera limitarse a las carpetas de VM si los inquilinos comparten hosts. Azure Migrate no puede detectar VM si a la cuenta de vCenter se le ha concedido acceso en el nivel de carpeta de VM de vCenter. Si tiene intención de limitar el ámbito de detección por carpetas de VM, asegúrese de que la cuenta de vCenter tiene asignado acceso de solo lectura asignado a nivel de VM. [Más información](set-discovery-scope.md).

## <a name="prepare-for-assessment"></a>Preparación para la evaluación

Preparar Azure y VMware para la evaluación del servidor. 

1. Consulte [los requisitos y las limitaciones de compatibilidad de VMware](migrate-support-matrix-vmware.md).
2. Configure los permisos de la cuenta de Azure para interactuar con Azure Migrate.
3. Preparar VMware para la evaluación.

Siga las instrucciones de [este tutorial](tutorial-prepare-vmware.md) para definir estas configuraciones.


## <a name="create-a-project"></a>Crear un proyecto

Según los requisitos de planeación, haga lo siguiente:

1. Cree un proyecto de Azure Migrate.
2. Agregue la herramienta de evaluación del servidor de Azure Migrate a los proyectos.

[Más información](how-to-add-tool-first-time.md)

## <a name="create-and-review-an-assessment"></a>Creación y revisión de una evaluación

1. Cree evaluaciones para VM de VMware.
1. Revise las evaluaciones en preparación para la planeación de la migración.


Siga las instrucciones de [este tutorial](tutorial-assess-vmware.md) para definir estas configuraciones.
    

## <a name="next-steps"></a>Pasos siguientes

En este artículo:
 
> [!div class="checklist"] 
> * Planeó escalar evaluaciones de Azure Migrate para VM de VMware
> * Preparó Azure y VMware para la evaluación
> * Creó un proyecto de Azure Migrate y ejecutó las evaluaciones
> * Revisó las evaluaciones como preparación para la migración.

Ahora, [obtenga información](concepts-assessment-calculation.md) sobre cómo se calculan las evaluaciones y cómo [modificarlas](how-to-modify-assessment.md).
