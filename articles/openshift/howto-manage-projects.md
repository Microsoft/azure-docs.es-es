---
title: Administración de recursos Red Hat OpenShift en Azure | Microsoft Docs
description: Administración de proyectos, plantillas y secuencias de imágenes en un clúster de Red Hat OpenShift en Azure
services: openshift
keywords: red hat openshift projects requests self-provisioner
author: mjudeikis
ms.author: gwallace
ms.date: 07/19/2019
ms.topic: conceptual
ms.service: container-service
ms.openlocfilehash: 35474df96254b158a39b6d913171c553bf30aea4
ms.sourcegitcommit: d39f2cd3e0b917b351046112ef1b8dc240a47a4f
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/25/2020
ms.locfileid: "88816446"
---
# <a name="manage-projects-templates-image-streams-in-an-azure-red-hat-openshift-cluster"></a>Administración de proyectos, plantillas y secuencias de imágenes en un clúster de Red Hat OpenShift en Azure

En una plataforma de contenedores de OpenShift, se usan proyectos para agrupar y aislar los objetos relacionados. Como administrador, puede proporcionar a los desarrolladores acceso a proyectos específicos, permitirles crear sus propios proyectos y concederles derechos administrativos a proyectos individuales.

## <a name="self-provisioning-projects"></a>Proyectos de aprovisionamiento automático

Puede permitir que los desarrolladores creen sus propios proyectos. Un punto de conexión de API es responsable de aprovisionar un proyecto de acuerdo con una plantilla denominada proyecto-solicitud. La consola web y el comando `oc new-project` usan este punto de conexión cuando un desarrollador crea un nuevo proyecto.

Cuando se envía una solicitud de proyecto, la API sustituye los parámetros siguientes de la plantilla:

| Parámetro               | Descripción                                    |
| ----------------------- | ---------------------------------------------- |
| PROJECT_NAME            | Nombre del proyecto. Necesario.             |
| PROJECT_DISPLAYNAME     | Nombre para mostrar del proyecto. Puede estar vacío. |
| PROJECT_DESCRIPTION     | Descripción del proyecto. Puede estar vacío.  |
| PROJECT_ADMIN_USER      | Nombre del usuario que se va a administrar.       |
| PROJECT_REQUESTING_USER | Nombre del usuario solicitante.           |

Se concede acceso a la API a los desarrolladores con el enlace del rol de clúster de aprovisionador automático. Esta característica está disponible de forma predeterminada para todos los desarrolladores autenticados.

## <a name="modify-the-template-for-a-new-project"></a>Modificación de la plantilla de un nuevo proyecto 

1. Inicie sesión como usuario con privilegios `customer-admin`.

2. Edite la plantilla de proyecto-solicitud predeterminada.

   ```
   oc edit template project-request -n openshift
   ```

3. Para quitar la plantilla de proyecto predeterminada del proceso de actualización de Red Hat OpenShift en Azure (ARO), agregue la siguiente anotación: `openshift.io/reconcile-protect: "true"`

   ```
   ...
   metadata:
     annotations:
       openshift.io/reconcile-protect: "true"
   ...
   ```

   La plantilla de proyecto-solicitud no se actualizará con el proceso de actualización de ARO. Esto permite a los clientes personalizar la plantilla y conservar estas personalizaciones cuando se actualiza el clúster.

## <a name="disable-the-self-provisioning-role"></a>Deshabilitación del rol de aprovisionamiento automático

Puede impedir que un grupo de usuarios autenticados realice el aprovisionamiento automático de proyectos nuevos.

1. Inicie sesión como usuario con privilegios `customer-admin`.

2. Edite el enlace del rol de clúster de aprovisionadores automáticos.

   ```
   oc edit clusterrolebinding.rbac.authorization.k8s.io self-provisioners
   ```

3. Para quitar el rol del proceso de actualización de ARO, agregue la siguiente anotación: `openshift.io/reconcile-protect: "true"`.

   ```
   ...
   metadata:
     annotations:
       openshift.io/reconcile-protect: "true"
   ...
   ```

4. Cambie el enlace del rol de clúster para impedir que `system:authenticated:oauth` cree proyectos:

   ```
   apiVersion: rbac.authorization.k8s.io/v1
   groupNames:
   - osa-customer-admins
   kind: ClusterRoleBinding
   metadata:
     annotations:
       openshift.io/reconcile-protect: "true"
     labels:
       azure.openshift.io/owned-by-sync-pod: "true"
     name: self-provisioners
   roleRef:
     name: self-provisioner
   subjects:
   - kind: SystemGroup
     name: osa-customer-admins
   ```

## <a name="manage-default-templates-and-imagestreams"></a>Administración de plantillas y secuencias de imágenes predeterminadas

En Red Hat OpenShift en Azure, puede deshabilitar las actualizaciones de las plantillas y las secuencias de imágenes predeterminadas dentro del espacio de nombres `openshift`.
Para deshabilitar las actualizaciones de todos los valores de `Templates` y `ImageStreams` en el espacio de nombres `openshift`, siga estos pasos:

1. Inicie sesión como usuario con privilegios `customer-admin`.

2. Edite el espacio de nombres `openshift`:

   ```
   oc edit namespace openshift
   ```

3. Quite el espacio de nombres `openshift` del proceso de actualización de ARO; para ello, agregue la siguiente anotación: `openshift.io/reconcile-protect: "true"`

   ```
   ...
   metadata:
     annotations:
       openshift.io/reconcile-protect: "true"
   ...
   ```

   Cualquier objeto del espacio de nombres `openshift` se puede quitar del proceso de actualización si se le agrega la anotación `openshift.io/reconcile-protect: "true"`.

## <a name="next-steps"></a>Pasos siguientes

Pruebe el tutorial:
> [!div class="nextstepaction"]
> [Creación de un clúster de Red Hat OpenShift en Azure](tutorial-create-cluster.md)
