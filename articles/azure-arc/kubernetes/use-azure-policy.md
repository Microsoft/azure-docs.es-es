---
title: Uso de Azure Policy para aplicar configuraciones de clúster a escala (versión preliminar)
services: azure-arc
ms.service: azure-arc
ms.date: 05/19/2020
ms.topic: article
author: mlearned
ms.author: mlearned
description: Uso de Azure Policy para aplicar configuraciones de clúster a escala
keywords: Kubernetes, Arc, Azure, K8s, contenedores
ms.openlocfilehash: e4279f3d89376320116067bf191e3196271918ce
ms.sourcegitcommit: 3d79f737ff34708b48dd2ae45100e2516af9ed78
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/23/2020
ms.locfileid: "87050044"
---
# <a name="use-azure-policy-to-apply-cluster-configurations-at-scale-preview"></a>Uso de Azure Policy para aplicar configuraciones de clúster a escala (versión preliminar)

## <a name="overview"></a>Información general

Use Azure Policy para exigir que cada recurso de `Microsoft.Kubernetes/connectedclusters` o `Microsoft.ContainerService/managedClusters` habilitado para operaciones de Git tenga aplicado un elemento `Microsoft.KubernetesConfiguration/sourceControlConfigurations` específico. Para usar Azure Policy, debe seleccionar una definición de directiva existente y crear una asignación de directiva. Al crear la asignación de directiva, establezca el ámbito de la asignación: será un grupo de recursos o una suscripción de Azure. También puede establecer los parámetros para el elemento `sourceControlConfiguration` que se creará. Una vez que se ha creado la asignación, el motor de directivas identificará todos los recursos `connectedCluster` o `managedCluster` que se encuentren dentro del ámbito y aplicará `sourceControlConfiguration` a cada uno de ellos.

Si usa varios repositorios de Git como orígenes de verdad para cada clúster (por ejemplo, un repositorio para el operador de clúster o TI central, y otros para los equipos de la aplicación), puede habilitarlo mediante varias asignaciones de directiva, cada una configurada para utilizar un repositorio de Git diferente.

## <a name="prerequisite"></a>Requisito previo

Asegúrese de que tiene permisos `Microsoft.Authorization/policyAssignments/write` en el ámbito (suscripción o grupo de recursos) en el que quiere crear esta asignación de directiva.

## <a name="create-a-policy-assignment"></a>Creación de una asignación de directiva

1. En Azure Portal, vaya a Directiva y, en la sección **Creación** de la barra lateral, seleccione **Definiciones**.
2. Elija la directiva integrada "implementar GitOps en el clúster de Kubernetes" en la categoría de "Kubernetes" y haga clic en **Asignar**.
3. Establezca el **Ámbito** en el grupo de administración, la suscripción o el grupo de recursos al que se aplicará la asignación de directiva.
4. Si quiere excluir recursos del ámbito de la directiva, establezca **Exclusiones**.
5. Asigne un **Nombre** y una **Descripción** a la asignación de directiva que pueda usar para identificarla con facilidad.
6. Asegúrese de que **Cumplimiento de directivas** esté establecido en *Habilitado*.
7. Seleccione **Next** (Siguiente).
8. Establezca los valores de parámetro que se usarán durante la creación de `sourceControlConfiguration`.
9. Seleccione **Next** (Siguiente).
10. Habilite **Crear una tarea de corrección**.
11. Asegúrese de que **Crear una identidad administrada** está activada y de que la identidad tendrá permisos de **Colaborador**. Vea [esta documentación](../../governance/policy/assign-policy-portal.md) y [el comentario de esta documentación](../../governance/policy/how-to/remediate-resources.md) para obtener más información sobre los permisos que necesita.
12. Seleccione **Revisar + crear**.

Después de crear la asignación de directiva, para cualquier recurso `connectedCluster` nuevo (o recurso `managedCluster` con los agentes de GitOps instalados) que se encuentra dentro del ámbito de la asignación, se aplicará `sourceControlConfiguration`. Para los clústeres existentes, tendrá que ejecutar manualmente una tarea de corrección. Normalmente, se tarda entre 10 y 20 minutos para que la asignación de la directiva surta efecto.

## <a name="verify-a-policy-assignment"></a>Comprobación de una asignación de directiva

1. En Azure Portal, vaya a uno de los recursos `connectedCluster` y, en la sección **Configuración** de la barra lateral, seleccione **Directivas**. (La experiencia de usuario del clúster de AKS todavía no se ha implementado, pero se hará en el futuro).
2. En la lista, debería ver la asignación de directiva que ha creado antes y el valor de **Estado de cumplimiento** debe ser *Conforme*.
3. En la sección **Configuración** de la barra lateral, seleccione **Configuraciones**.
4. En la lista, debería ver el elemento `sourceControlConfiguration` creado por la asignación de directiva.
5. Use **kubectl** para interrogar al clúster: debería ver el espacio de nombres y los artefactos creados por `sourceControlConfiguration`.
6. En cinco minutos, debería ver en el clúster los artefactos que se describen en los manifiestos del repositorio de Git configurado.

## <a name="next-steps"></a>Pasos siguientes

* [Configuración de Azure Monitor para contenedores con clústeres de Kubernetes habilitados para Arc](../../azure-monitor/insights/container-insights-enable-arc-enabled-clusters.md)
