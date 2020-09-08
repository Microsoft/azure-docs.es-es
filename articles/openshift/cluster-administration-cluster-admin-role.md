---
title: Rol de administrador de clústeres de Red Hat OpenShift en Azure | Microsoft Docs
description: Asignación y uso del rol de administrador de clústeres de Azure Red Hat OpenShift
services: container-service
author: mjudeikis
ms.author: jzim
ms.service: container-service
ms.topic: article
ms.date: 09/25/2019
ms.openlocfilehash: 38686ba35285159d7a27724b5402a6b6e2f3a696
ms.sourcegitcommit: d39f2cd3e0b917b351046112ef1b8dc240a47a4f
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/25/2020
ms.locfileid: "88815528"
---
# <a name="azure-red-hat-openshift-customer-administrator-role"></a>Rol de administrador de clientes de Red Hat OpenShift en Azure
 
Usted es el administrador de un clúster de Azure Red Hat OpenShift. Su cuenta tiene más permisos y dispone de acceso a todos los proyectos creados por usuarios.

Cuando su cuenta lleva enlazado el rol de autorización customer-admin-cluster, puede administrar automáticamente un proyecto.

> [!Note] 
> El rol clúster customer-admin-cluster no lo mismo que el rol cluster-admin.

Por ejemplo, puede ejecutar acciones asociadas a un conjunto de verbos (`create`) para operar en un conjunto de nombres de recursos (`templates`). Para ver los detalles de estos roles y sus conjuntos de verbos y recursos, ejecute el siguiente comando:

`$ oc get clusterroles customer-admin-cluster -o yaml`

Los nombres de verbo no tienen por qué estar asignados directamente a los comandos `oc`. Suelen equivaler a los tipos de operaciones de la CLI que puede realizar. 

Por ejemplo, tener el verbo `list` significa que puede mostrar una lista de todos los objetos de un nombre de recurso (`oc get`). El verbo `get` significa que puede mostrar los detalles de un objeto concreto si conoce su nombre (`oc describe`).

## <a name="configure-the-customer-administrator-role"></a>Configuración del rol de administrador de clientes

Solo puede configurar el rol customer-admin-cluster durante la creación del clúster cuando se proporciona la marca `--customer-admin-group-id`. Este campo no se puede configurar actualmente en Azure Portal. Para más información sobre cómo configurar Azure Active Directory y el grupo de administradores, consulte [Integración de Azure Active Directory para Red Hat OpenShift en Azure](howto-aad-app-configuration.md).

## <a name="confirm-membership-in-the-customer-administrator-role"></a>Confirmación de la pertenencia al rol Administrador de clientes

Para confirmar su pertenencia al grupo de administradores de clientes, pruebe los comandos de la CLI de OpenShift `oc get nodes` o `oc projects`. `oc get nodes` mostrará una lista de nodos si tiene el rol customer-admin-cluster y un error de permiso si solo tiene el rol customer-admin-project. `oc projects` mostrará todos los proyectos del clúster, no solo aquellos en los que esté trabajando.

Para explorar más roles y permisos del clúster, puede usar el comando [`oc policy who-can <verb> <resource>`](https://docs.openshift.com/container-platform/3.11/admin_guide/manage_rbac.html#managing-role-bindings).

## <a name="next-steps"></a>Pasos siguientes

Configure el rol de clúster customer-admin-cluster:
> [!div class="nextstepaction"]
> [Integración de Azure Active Directory con Red Hat OpenShift en Azure](howto-aad-app-configuration.md)
