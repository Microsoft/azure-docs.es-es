---
title: Roles y tareas del proceso de ciencia de datos en equipo
description: Una descripción de los componentes clave, los roles del personal y las tareas asociadas para un grupo de ciencia de datos.
author: marktab
manager: marktab
editor: marktab
services: machine-learning
ms.service: machine-learning
ms.subservice: team-data-science-process
ms.topic: article
ms.date: 01/10/2020
ms.author: tdsp
ms.custom: seodec18, previous-author=deguhath, previous-ms.author=deguhath
ms.openlocfilehash: 4bcbbd9378b6ea861c926eed16d2ceed8131b913
ms.sourcegitcommit: 3d79f737ff34708b48dd2ae45100e2516af9ed78
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/23/2020
ms.locfileid: "87082756"
---
# <a name="team-data-science-process-roles-and-tasks"></a>Roles y tareas del proceso de ciencia de datos en equipo

El proceso de ciencia de datos en equipo (TDSP) es una plataforma desarrollada por Microsoft que proporciona una metodología estructurada para crear soluciones de análisis predictivo y aplicaciones inteligentes de un modo eficaz. En este artículo se describen los roles clave del personal y sus tareas asociadas para un equipo de ciencia de datos que estandariza este proceso.

Este artículo introductorio contiene vínculos a tutoriales sobre cómo configurar el entorno de TDSP. En los tutoriales se proporcionan instrucciones detalladas para usar Azure DevOps Projects, repositorios de Azure Repos y Azure Boards.  El objetivo de la motivación es pasar del concepto al modelado y a la implementación.

Los tutoriales usan Azure DevOps porque así es cómo se implementa TDSP en Microsoft. Azure DevOps facilita la colaboración mediante la integración de la seguridad basada en roles, el seguimiento y la administración de elementos de trabajo, así como el hospedaje de código, el uso compartido y el control de código fuente. En los tutoriales también se utiliza [Data Science Virtual Machine (DSVM)](https://aka.ms/dsvm) de Azure como escritorio de análisis, que tiene varias herramientas de ciencia de datos populares configuradas previamente e integradas con software de Microsoft y servicios de Azure. 

Puede usar los tutoriales para implementar TDSP con otras herramientas y entornos de hospedaje de código, planeamiento de Agile y desarrollo, pero es posible que algunas características no estén disponibles.

## <a name="structure-of-data-science-groups-and-teams"></a>Estructura de los grupos y equipos de ciencia de datos

Las funciones de ciencia de datos en las empresas a menudo se organizan en la siguiente jerarquía:

- Grupo de ciencia de datos
  - Equipos de ciencia de datos dentro del grupo

En este tipo de estructura habrá responsables de grupo y de equipo. Normalmente, un proyecto de ciencia de datos lo realiza un equipo de ciencia de datos. Los equipos de ciencia de datos cuentan con responsables de proyecto (para tareas de administración y gobernanza del proyecto) y científicos de datos e ingenieros individuales que ejecutarán las partes de ciencia de datos e ingeniería de datos del proyecto. Los responsables de grupo, equipo o proyecto realizan la configuración y gobernanza del proyecto inicial.

## <a name="definition-and-tasks-for-the-four-tdsp-roles"></a>Definición y tareas de los cuatro roles de TDSP
Suponiendo que la unidad de ciencia de datos conste de equipos dentro de un grupo, existen cuatro roles distintos para el personal de TDSP:

1. **Administrador de grupo**: administra la unidad de ciencia de datos completa en una empresa. Una unidad de ciencia de datos podría tener varios equipos, cada uno de ellos trabajando en varios proyectos de ciencia de datos en segmentos verticales de negocio distintos. Un Administrador de grupo puede delegar sus tareas en un suplente, pero no cambian las tareas asociadas al rol.
   
2. **Responsable de equipo**: administra un equipo de la unidad de ciencia de datos de una empresa. Un equipo está formada por varios científicos de datos. En una unidad de ciencia de datos reducida, el administrador de grupo y el responsable de equipo podrían ser la misma persona.
   
3. **Responsable de proyecto**: administra las actividades diarias de los científicos de datos en un proyecto de ciencia de datos específico.
   
4. **Colaboradores individuales del proyecto**: científicos de datos, analistas de negocios, ingenieros de datos, arquitectos y otros colaboradores que ejecutan un proyecto de ciencia de datos.

> [!NOTE]
> En función de la estructura y el tamaño de una empresa, una sola persona puede desempeñar más de un rol o un grupo de personas podría ocupar un solo rol.

### <a name="tasks-to-be-completed-by-the-four-roles"></a>Tareas que deben realizar los cuatro roles

En el diagrama siguiente se muestran las tareas de nivel superior para cada rol del proceso de ciencia de datos en equipo. Este esquema y el siguiente, donde las tareas de cada rol de TDSP se describen más detalladamente, pueden ayudarle a elegir el tutorial que necesita en función de sus responsabilidades.

![Información general de roles y tareas](./media/roles-tasks/overview-tdsp-top-level.png)

## <a name="group-manager-tasks"></a>Tareas del administrador de grupo

Las siguientes tareas las realiza el administrador de grupo (o un administrador del sistema TDSP designado) para adoptar TDSP:

- Crea una **organización** de Azure DevOps y un proyecto de grupo dentro de la organización. 
- Crea un **repositorio de plantillas de proyecto** en el proyecto de grupo de Azure DevOps y lo inicializa desde el repositorio de plantillas de proyecto desarrollado por el equipo TDSP de Microsoft. El repositorio de plantillas de proyecto TDSP de Microsoft proporciona:
  - Una **estructura de directorios estandarizada** que incluye directorios de datos, código y documentos.
  - Un conjunto de **plantillas de documentos estandarizadas** como guía para un proceso de ciencia de datos eficaz.
- Crea un **repositorio de utilidades** y lo inicializa desde el repositorio de utilidades desarrollado por el equipo de TDSP de Microsoft. El repositorio de utilidades de TDSP de Microsoft proporciona un conjunto de utilidades útiles para que el trabajo de un científico de datos resulte más eficaz. El repositorio de utilidades de Microsoft incluye utilidades para exploración interactiva de datos, análisis e informes y para informes y modelos de base de referencia.
- Configura la **directiva de control de seguridad** para la cuenta de la organización.

Para obtener instrucciones detalladas, consulte [Tareas del administrador de grupo en un equipo de ciencia de datos](group-manager-tasks.md).

## <a name="team-lead-tasks"></a>Tareas del responsable de equipo

Las siguientes tareas las realiza el responsable de equipo (o un administrador del proyecto designado) para adoptar TDSP:

- Crea un **proyecto** de equipo en la organización de Azure DevOps del grupo.
- Crea el **repositorio de plantillas de proyecto** en el proyecto y lo inicializa desde el repositorio de plantillas de proyecto de grupo configurado por el administrador del grupo o el delegado.
- Crea el **repositorio de utilidades del equipo**, lo inicializa desde el repositorio de utilidades del grupo y agrega utilidades específicas del equipo al repositorio.
- Opcionalmente, crea una instancia de [Azure File Storage](https://azure.microsoft.com/services/storage/files/) para almacenar recursos de datos útiles para el equipo. Otros miembros del equipo pueden montar este almacén de archivos compartidos en la nube en sus escritorios de análisis.
- Opcionalmente, monta la instancia de Azure File Storage en la instancia de **DSVM** del equipo y le agrega recursos de datos de equipo.
- Configura el **control de seguridad** mediante la adición de miembros del equipo y la configuración de sus permisos.

Para obtener instrucciones detalladas, consulte [Tareas del responsable de equipo en un equipo de ciencia de datos](team-lead-tasks.md).


## <a name="project-lead-tasks"></a>Tareas del responsable de proyecto

Las siguientes tareas las realiza el responsable de proyecto para adoptar TDSP:

- Crea un **repositorio del proyecto** en el proyecto de equipo y lo inicializa desde el repositorio de plantillas del proyecto.
- Opcionalmente, crea una instancia de **Azure File Storage** para almacenar recursos de datos del proyecto.
- Opcionalmente, monta la instancia de Azure File Storage en la instancia de **DSVM** y le agrega recursos de datos del proyecto.
- Configura el **control de seguridad** mediante la adición de miembros del proyecto y la configuración de sus permisos.

Para obtener instrucciones detalladas, consulte [Tareas del responsable de proyecto en un equipo de ciencia de datos](project-lead-tasks.md).

## <a name="project-individual-contributor-tasks"></a>Tareas del colaborador individual del proyecto

El colaborador individual del proyecto, normalmente un científico de datos, realiza las siguientes tareas mediante TDSP:

- Clona el **repositorio del proyecto** configurado por el responsable de proyecto.
- Opcionalmente, monta el recurso compartido y la instancia de **Azure File Storage** del proyecto en la instancia de **Data Science Virtual Machine** (DSVM).
- Ejecuta el proyecto.

Para obtener instrucciones detalladas sobre la incorporación a un proyecto, consulte [Colaboradores individuales del proyecto en un equipo de ciencia de datos](project-ic-tasks.md).

## <a name="data-science-project-execution-workflow"></a>Flujo de trabajo de ejecución del proyecto de ciencia de datos

Siguiendo los tutoriales pertinentes, los científicos de datos, los responsables de proyecto y los responsables de equipo pueden crear elementos de trabajo para realizar el seguimiento de todas las tareas y las fases de un proyecto de principio a fin. El uso de Azure Repos también fomenta la colaboración entre los científicos de datos y garantiza que los artefactos que se generan durante la ejecución del proyecto se controlan mediante versión y son compartidos por todos los miembros del proyecto. Azure DevOps permite vincular los elementos de trabajo de Azure Boards con las ramas del repositorio de Azure Repos y realizar un seguimiento sencillo de lo que se ha hecho para un elemento de trabajo.

La siguiente ilustración muestra el flujo de trabajo de TDSP para la ejecución del proyecto:

![Flujo de trabajo de proyecto de ciencia de datos típico](./media/roles-tasks/overview-project-execute.png)

Los pasos del flujo de trabajo se pueden agrupar en tres actividades:

- Los responsables de proyecto realizan el planeamiento de sprints.
- Los científicos de datos desarrollan artefactos en ramas de `git` para tratar los elementos de trabajo.
- Los responsables de proyecto u otros miembros del equipo realizan revisiones de código y combinan ramas de trabajo en la rama primaria.

Para obtener instrucciones detalladas del flujo de trabajo de ejecución del proyecto, consulte [Desarrollo de Agile de proyectos de ciencia de datos](agile-development.md).

## <a name="tdsp-project-template-repository"></a>Repositorio de plantillas de proyecto de TDSP

Utilice el [repositorio de plantillas de proyecto](https://github.com/Azure/Azure-TDSP-ProjectTemplate) del equipo de TDSP de Microsoft para permitir la ejecución y colaboración eficientes del proyecto. El repositorio le ofrece una estructura de directorios estandarizada y plantillas de documentos que puede utilizar para sus propios proyectos de TDSP.

## <a name="next-steps"></a>Pasos siguientes

Explore descripciones más detalladas de los roles y tareas que el Proceso de ciencia de datos en equipo define:

- [Tareas del administrador de grupo en un equipo de ciencia de datos](group-manager-tasks.md)
- [Tareas del responsable de equipo en un equipo de ciencia de datos](team-lead-tasks.md)
- [Tareas del responsable de proyecto en un equipo de ciencia de datos](project-lead-tasks.md)
- [Tareas de colaboradores individuales del proyecto en un equipo de ciencia de datos](project-ic-tasks.md)
