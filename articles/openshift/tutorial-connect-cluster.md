---
title: 'Tutorial: Conexión a un clúster de la versión 4 de Red Hat OpenShift en Azure'
description: Aprenda a conectar un clúster de Red Hat OpenShift en Microsoft Azure.
author: sakthi-vetrivel
ms.author: suvetriv
ms.topic: tutorial
ms.service: container-service
ms.date: 04/24/2020
ms.openlocfilehash: 226cf29b1a94b4508a9d68f02b7400a18eba4bc2
ms.sourcegitcommit: 02ca0f340a44b7e18acca1351c8e81f3cca4a370
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/19/2020
ms.locfileid: "88587861"
---
# <a name="tutorial-connect-to-an-azure-red-hat-openshift-4-cluster"></a>Tutorial: Conexión a un clúster de la versión 4 de Red Hat OpenShift en Azure

En este tutorial, la segunda de tres partes, se conectará a un clúster de Red Hat OpenShift en Azure que ejecuta OpenShift 4 como usuario kubeadmin mediante la consola web de OpenShift. Aprenderá a:
> [!div class="checklist"]
> * Obtener las credenciales de `kubeadmin` para el clúster
> * Instalación de la CLI de OpenShift
> * Conectarse a un clúster de Red Hat OpenShift en Azure mediante la CLI de OpenShift

## <a name="before-you-begin"></a>Antes de empezar

En los tutoriales anteriores, se creó un clúster de Red Hat OpenShift en Azure. Si no ha realizado estos pasos, pero desea continuar, comience con el [Tutorial 1: Creación de un clúster de la versión 4 de Red Hat Openshift en Azure](tutorial-create-cluster.md).

Si decide instalar y usar la CLI de forma local, en este tutorial es preciso que ejecute la CLI de Azure de la versión 2.6.0, u otra posterior. Ejecute `az --version` para encontrar la versión. Si necesita instalarla o actualizarla, vea [Instalación de la CLI de Azure](https://docs.microsoft.com/cli/azure/install-azure-cli?view=azure-cli-latest).

## <a name="connect-to-the-cluster"></a>Conectarse al clúster

Puede iniciar sesión en el clúster con el usuario `kubeadmin`.  Ejecute el siguiente comando para buscar la contraseña del usuario `kubeadmin`.

```azurecli-interactive
az aro list-credentials \
  --name $CLUSTER \
  --resource-group $RESOURCEGROUP
```

En la siguiente salida de ejemplo se muestra que la contraseña estará en `kubeadminPassword`.

```json
{
  "kubeadminPassword": "<generated password>",
  "kubeadminUsername": "kubeadmin"
}
```

Para encontrar la dirección URL de la consola del clúster, ejecute el siguiente comando, que será similar a `https://console-openshift-console.apps.<random>.<region>.aroapp.io/`.

```azurecli-interactive
 az aro show \
    --name $CLUSTER \
    --resource-group $RESOURCEGROUP \
    --query "consoleProfile.url" -o tsv
```

Inicie la dirección URL de la consola en un explorador e inicie sesión con las credenciales de `kubeadmin`.

![Pantalla de inicio de sesión de Red Hat OpenShift en Azure](media/aro4-login.png)

## <a name="install-the-openshift-cli"></a>Instalación de la CLI de OpenShift

Cuando haya iniciado sesión en la consola web de OpenShift, haga clic en el signo **?** en la parte superior derecha y, luego, en **Command Line Tools** (Herramientas de línea de comandos). Descargue la versión adecuada para su máquina.

![Pantalla de inicio de sesión de Red Hat OpenShift en Azure](media/aro4-download-cli.png)

También puede descargar la versión más reciente de la CLI adecuada en su máquina desde <https://mirror.openshift.com/pub/openshift-v4/clients/ocp/latest/>.

Si va a ejecutar los comandos en Azure Cloud Shell, descargue la CLI más reciente de OpenShift 4 para Linux.

```azurecli-interactive
cd ~
wget https://mirror.openshift.com/pub/openshift-v4/clients/ocp/latest/openshift-client-linux.tar.gz

mkdir openshift
tar -zxvf openshift-client-linux.tar.gz -C openshift
echo 'export PATH=$PATH:~/openshift' >> ~/.bashrc && source ~/.bashrc
```

## <a name="connect-using-the-openshift-cli"></a>Conexión mediante la CLI de OpenShift

Recupere la dirección del servidor de la API.

```azurecli-interactive
apiServer=$(az aro show -g $RESOURCEGROUP -n $CLUSTER --query apiserverProfile.url -o tsv)
```

Inicie sesión en el servidor de API del clúster de OpenShift mediante el siguiente comando. Reemplace **\<kubeadmin password>** por la contraseña que acaba de recuperar.

```azurecli-interactive
oc login $apiServer -u kubeadmin -p <kubeadmin password>
```

## <a name="next-steps"></a>Pasos siguientes

En esta parte del tutorial, ha aprendido a:
> [!div class="checklist"]
> * Obtener las credenciales de `kubeadmin` para el clúster
> * Instalación de la CLI de OpenShift
> * Conectarse a un clúster de Red Hat OpenShift en Azure mediante la CLI de OpenShift

Avance hasta el siguiente tutorial:
> [!div class="nextstepaction"]
> [Eliminación de un clúster de Red Hat OpenShift en Azure](tutorial-delete-cluster.md)
