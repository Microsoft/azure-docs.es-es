---
title: Creación de una restauración de aplicaciones del clúster de Red Hat OpenShift en Azure 4
description: Aprenda a crear una restauración de las aplicaciones del clúster de Red Hat OpenShift en Azure con Velero
ms.service: container-service
ms.topic: article
ms.date: 06/22/2020
author: troy0820
ms.author: b-trconn
keywords: aro, openshift, az aro, red hat, cli
ms.custom: mvc
ms.openlocfilehash: 0cd6797bcdfadca807e25f8b3decf34bd553fc56
ms.sourcegitcommit: 4feb198becb7a6ff9e6b42be9185e07539022f17
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/04/2020
ms.locfileid: "89470058"
---
# <a name="create-an-azure-red-hat-openshift-4-cluster-application-restore"></a>Creación de una restauración de aplicaciones del clúster de Red Hat OpenShift en Azure 4

En este artículo, prepará el entorno para crear una restauración de aplicaciones del clúster de Red Hat OpenShift en Azure 4. Aprenderá a:

> [!div class="checklist"]
> * Configurar los requisitos previos e instalar las herramientas necesarias
> * Crear una restauraciones de aplicaciones de Red Hat OpenShift en Azure 4

Si decide instalar y usar la CLI de forma local, en este tutorial es preciso que ejecute la CLI de Azure de la versión 2.6.0, u otra posterior. Ejecute `az --version` para encontrar la versión. Si necesita instalarla o actualizarla, vea [Instalación de la CLI de Azure](/cli/azure/install-azure-cli?view=azure-cli-latest).

## <a name="before-you-begin"></a>Antes de empezar

### <a name="create-an-azure-red-hat-openshift-4-application-backup"></a>Creación de una copia de seguridad de aplicaciones de Red Hat OpenShift en Azure 4

Para crear una copia de seguridad de aplicaciones de Red Hat OpenShift en Azure 4, consulte [Creación de una copia de seguridad de Red Hat OpenShift en Azure 4](./howto-create-a-backup.md).

## <a name="restore-an-azure-red-hat-openshift-4-application"></a>Restauración de una aplicación de Red Hat OpenShift en Azure 4

Estos pasos le permitirán restaurar una aplicación de la que se hizo una copia de seguridad con Velero.
Puede comprobar la lista de copias de seguridad que el clúster reconoce actualmente para ver qué copias de seguridad están disponibles para su restauración.  Para llevar a cabo este paso, deberá ejecutar el comando siguiente:

_(En este paso se supone que tiene instalado Velero en un proyecto denominado "velero")_

```bash
oc get backups -n velero
```

Una vez que tenga la copia de seguridad que desea restaurar, deberá realizar la restauración con el comando siguiente:

```bash
velero restore create --from-backup <name of backup from above output list>
```

En este paso se crearán los objetos de Kubernetes de los que se hizo una copia de seguridad en el paso anterior al crear una copia de seguridad.

Para ver el estado de la restauración, ejecute el paso siguiente:

```bash
oc get restore -n velero <name of restore created previously> -o yaml
```
Cuando la fase indica `Completed`, la aplicación Red Hat en Azure 4 se debería restaurar.

## <a name="next-steps"></a>Pasos siguientes

En este artículo se restauró la aplicación del clúster de Red Hat OpenShift en Azure 4. Ha aprendido a:

> [!div class="checklist"]
> * Crear una restauración de aplicaciones del clúster de OpenShift v4 con Velero


Vaya al artículo siguiente para aprender sobre los recursos compatibles con Red Hat OpenShift en Azure 4.

* [Recursos compatibles con Red Hat OpenShift en Azure 4](supported-resources.md)