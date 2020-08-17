---
title: 'Codificación colaborativa con Git: proceso de ciencia de datos en equipo'
description: Cómo realizar el desarrollo de código de colaboración para los proyectos de ciencia de datos con GIT con una planeación ágil.
author: marktab
manager: marktab
editor: marktab
ms.service: machine-learning
ms.subservice: team-data-science-process
ms.topic: article
ms.date: 01/10/2020
ms.author: tdsp
ms.custom: seodec18, previous-author=deguhath, previous-ms.author=deguhath
ms.openlocfilehash: 739be373992fcd994f085f8571675779e450bfee
ms.sourcegitcommit: 3d79f737ff34708b48dd2ae45100e2516af9ed78
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/23/2020
ms.locfileid: "87090219"
---
# <a name="collaborative-coding-with-git"></a>Codificación colaborativa con GIT

En este artículo se describe cómo usar Git como el marco de desarrollo de código de colaboración para los proyectos de ciencia de datos. En el artículo se describe cómo vincular código de Azure Repos a elementos de trabajo de [Agile Development](agile-development.md) en Azure Boards, cómo realizar revisiones de código y cómo crear y combinar solicitudes de incorporación de cambios.

## <a name="link-a-work-item-to-an-azure-repos-branch"></a><a name='Linkaworkitemwithagitbranch-1'></a>Vinculación de un elemento de trabajo a una rama de Azure Repos 

Azure DevOps proporciona una manera cómoda de conectar un caso de usuario o un elemento de trabajo de tareas de Azure Boards con una rama del repositorio Git de Azure Repos. Puede vincular el caso de usuario o la tarea directamente al código asociado. 

Para conectar un elemento de trabajo a una nueva rama, seleccione el botón de puntos suspensivos ( **...** ) de **Acciones** junto al elemento de trabajo y, en el menú contextual, desplácese a y seleccione **Nueva rama**.  

![1](./media/collaborative-coding-with-git/1-sprint-board-view.png)

En el cuadro de diálogo **Crear una rama**, proporcione el nombre de la nueva rama, el repositorio Git de Azure Repos de base y la rama. El repositorio de base debe estar en el mismo proyecto de Azure DevOps que el elemento de trabajo. La rama base puede ser cualquier rama existente. Seleccione **Crear rama**. 

![2](./media/collaborative-coding-with-git/2-create-a-branch.png)

También puede crear una nueva rama con el siguiente comando de bash de Git en Windows o Linux:

```bash
git checkout -b <new branch name> <base branch name>

```
Si no especifica un valor para \<base branch name>, la nueva rama se basa en `master`. 

Para cambiar a la rama de trabajo, ejecute el siguiente comando: 

```bash
git checkout <working branch name>
```

Después de cambiar a la rama de trabajo, puede empezar a desarrollar código o artefactos de documentación para completar el elemento de trabajo. La ejecución de `git checkout master` le devuelve a la rama `master`.

Un procedimiento recomendado es crear una rama de Git para cada elemento de trabajo del caso de usuario. A continuación, para cada elemento de trabajo de la tarea, puede crear una rama basada en la rama del caso de usuario. Organice las ramas en una jerarquía que se corresponda con la relación entre el caso de usuario y la tarea cuando tenga varias personas trabajando en casos de usuario diferentes para el mismo proyecto o en tareas diferentes para el mismo caso de usuario. Puede minimizar los conflictos si cada miembro del equipo trabaja en una rama diferente o en código u otros artefactos diferentes al compartir una rama. 

En la siguiente imagen se muestra la estrategia de bifurcación recomendada para el proceso de ciencia de datos en equipo. Es posible que no necesite tantas ramas como se muestra aquí, especialmente cuando solo una o dos personas trabajan en un proyecto o cuando solo una persona trabaja en todas las tareas de un caso de usuario. No obstante, separar la rama de desarrollo de la rama principal siempre es una buena práctica y puede ayudar a evitar que las actividades de desarrollo interrumpan la rama de versión. Encontrará una descripción completa del modelo de ramas de Git en [Un modelo correcto de ramas de Git](https://nvie.com/posts/a-successful-git-branching-model/).

![3](./media/collaborative-coding-with-git/3-git-branches.png)

También puede vincular un elemento de trabajo a una rama existente. En la página **Detalles** de un elemento de trabajo, seleccione **Agregar vínculo**. Después, seleccione una rama existente a la que vincular el elemento de trabajo y seleccione **Aceptar**. 

![4](./media/collaborative-coding-with-git/4-link-to-an-existing-branch.png)

## <a name="work-on-the-branch-and-commit-changes"></a><a name='WorkonaBranchandCommittheChanges-2'></a>Trabajo en la rama y confirmación de los cambios 

Después de realizar un cambio en el elemento de trabajo, como agregar un archivo de script de R a la rama `script` de la máquina local, puede confirmar el cambio desde la rama local a la rama de trabajo ascendente mediante los siguientes comandos de bash de Git:

```bash
git status
git add .
git commit -m "added an R script file"
git push origin script
```

![5](./media/collaborative-coding-with-git/5-sprint-push-to-branch.png)

## <a name="create-a-pull-request"></a><a name='CreateapullrequestonVSTS-3'></a>Creación de una solicitud de incorporación de cambios

Después de una o varias confirmaciones y envíos de cambios, cuando esté listo para combinar la rama de trabajo actual en su rama de base, puede crear y enviar una *solicitud de incorporación de cambios* en Azure Repos. 

En la página principal del proyecto de Azure DevOps, seleccione **Repos** > **Solicitudes de incorporación de cambios** en el panel de navegación izquierdo. A continuación, seleccione uno de los botones **Nueva solicitud de incorporación de cambios** o el vínculo **Crear una solicitud de incorporación de cambios**.

![6](./media/collaborative-coding-with-git/6-spring-create-pull-request.png)

En la pantalla **Nueva solicitud de incorporación de cambios**, si es necesario, vaya al repositorio de Git y a la rama en la que desea combinar los cambios. Agregue o cambie cualquier otra información que desee. En **Revisores**, agregue los nombres de los revisores y, a continuación, seleccione **Crear**. 

![7](./media/collaborative-coding-with-git/7-spring-send-pull-request.png)

## <a name="review-and-merge"></a><a name='ReviewandMerge-4'></a>Revisión y combinación

Una vez que se crea la solicitud de incorporación de cambios, los revisores reciben una notificación por correo electrónico para revisar dicha solicitud. Los revisores prueban si los cambios funcionan y comprueban los cambios con el solicitante si es posible. Los revisores pueden realizar comentarios, solicitar cambios y aprobar o rechazar la solicitud de incorporación de cambios en función de su evaluación. 

![8](./media/collaborative-coding-with-git/8-add_comments.png)

Una vez que los revisores aprueban los cambios, usted u otra persona con permisos de combinación pueden combinar la rama de trabajo en su rama de base. Seleccione **Completado** y, a continuación, seleccione **Completar combinación** en el cuadro de diálogo **Completar solicitud de incorporación de cambios**. Puede optar por eliminar la rama de trabajo una vez combinada. 

![10](./media/collaborative-coding-with-git/10-spring-complete-pullrequest.png)

Confirme que la solicitud está marcada como **COMPLETADO**. 

![11](./media/collaborative-coding-with-git/11-spring-merge-pullrequest.png)

Cuando vuelva a **Repos** en el panel de navegación izquierdo, puede ver que ha cambiado a la rama principal puesto que se eliminó la rama `script`.

![12](./media/collaborative-coding-with-git/12-spring-branch-deleted.png)

También puede utilizar los comandos de bash de Git siguientes para combinar la rama de trabajo `script` con la rama de base y eliminar la rama de trabajo después de la combinación:

```bash
git checkout master
git merge script
git branch -d script
```

![13](./media/collaborative-coding-with-git/13-spring-branch-deleted-commandline.png)

## <a name="next-steps"></a>Pasos siguientes

En el artículo [Ejecución de las tareas de ciencia de datos](execute-data-science-tasks.md) se muestra cómo usar las utilidades para completar varias tareas comunes de ciencia de datos, como la exploración interactiva de los datos, el análisis de los datos, la creación de informes y la creación de modelos.

El artículo [Tutoriales de ejemplo](walkthroughs.md) proporciona una lista de tutoriales sobre escenarios específicos con vínculos y descripciones en miniatura. Los escenarios vinculados muestran cómo combinar servicios y herramientas en la nube y locales en flujos de trabajo o canalizaciones para crear aplicaciones inteligentes. 

