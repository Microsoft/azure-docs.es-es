---
title: Uso de OpenFaaS con Azure Kubernetes Service (AKS)
description: Aprenda a implementar y usar OpenFaaS en un clúster de Azure Kubernetes Service (AKS) para crear funciones sin servidor con contenedores.
author: justindavies
ms.topic: conceptual
ms.date: 03/05/2018
ms.author: juda
ms.custom: mvc
ms.openlocfilehash: 98b1842f81703041f419850be17c0c05a24b7c6b
ms.sourcegitcommit: bf1340bb706cf31bb002128e272b8322f37d53dd
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/03/2020
ms.locfileid: "89440905"
---
# <a name="using-openfaas-on-aks"></a>Uso de OpenFaaS en AKS

[OpenFaaS][open-faas] es una plataforma para la creación de funciones sin servidor mediante contenedores. Como proyecto de código abierto, se ha adoptado a gran escala dentro de la comunidad. En este documento se detalla la instalación y uso de OpenFaas en un clúster de Azure Kubernetes Service (AKS).

## <a name="prerequisites"></a>Prerrequisitos

Para completar los pasos de este artículo, necesita lo siguiente.

* Conocimientos básicos sobre Kubernetes.
* Un clúster de Azure Kubernetes Service (AKS) y las credenciales de AKS configuradas en el sistema de desarrollo.
* CLI de Azure instalada en el sistema de desarrollo.
* Herramientas de línea de comandos GIT instaladas en el sistema.

## <a name="add-the-openfaas-helm-chart-repo"></a>Incorporación del repositorio de gráficos de Helm de OpenFaaS

Vaya a [https://shell.azure.com](https://shell.azure.com) para abrir Azure Cloud Shell en el explorador.

OpenFaaS mantiene sus propios gráficos de Helm para estar al día con los últimos cambios.

```console
helm repo add openfaas https://openfaas.github.io/faas-netes/
helm repo update
```

## <a name="deploy-openfaas"></a>Implementación de OpenFaaS

Como recomendación, OpenFaaS y sus funciones deberían almacenarse en su propio espacio de nombres de Kubernetes.

Cree un espacio de nombres para el sistema y las funciones de OpenFaaS:

```console
kubectl apply -f https://raw.githubusercontent.com/openfaas/faas-netes/master/namespaces.yml
```

Genere una contraseña para el portal de la interfaz de usuario de OpenFaaS y la API REST:

```console
# generate a random password
PASSWORD=$(head -c 12 /dev/urandom | shasum| cut -d' ' -f1)

kubectl -n openfaas create secret generic basic-auth \
--from-literal=basic-auth-user=admin \
--from-literal=basic-auth-password="$PASSWORD"
```

Puede obtener el valor del secreto con `echo $PASSWORD`.

La contraseña que creamos aquí se usará en el gráfico de Helm para habilitar la autenticación básica en la puerta de enlace de OpenFaaS, que se expone en Internet mediante un equilibrador de carga de nube.

En el repositorio clonado se incluye un gráfico de Helm para OpenFaaS. Utilice este gráfico para implementar OpenFaaS en el clúster de AKS.

```console
helm upgrade openfaas --install openfaas/openfaas \
    --namespace openfaas  \
    --set basic_auth=true \
    --set functionNamespace=openfaas-fn \
    --set serviceType=LoadBalancer
```

Salida:

```output
NAME:   openfaas
LAST DEPLOYED: Wed Feb 28 08:26:11 2018
NAMESPACE: openfaas
STATUS: DEPLOYED

RESOURCES:
==> v1/ConfigMap
NAME                 DATA  AGE
prometheus-config    2     20s
alertmanager-config  1     20s

{snip}

NOTES:
To verify that openfaas has started, run:

```console
kubectl --namespace=openfaas get deployments -l "release=openfaas, app=openfaas"
```

Se crea una dirección IP pública para tener acceso a la puerta de enlace de OpenFaaS. Para recuperar esta dirección IP, use el comando [kubectl get service][kubectl-get]. La asignación de la dirección IP al servicio puede tardar hasta un minuto.

```console
kubectl get service -l component=gateway --namespace openfaas
```

Salida.

```output
NAME               TYPE           CLUSTER-IP     EXTERNAL-IP    PORT(S)          AGE
gateway            ClusterIP      10.0.156.194   <none>         8080/TCP         7m
gateway-external   LoadBalancer   10.0.28.18     52.186.64.52   8080:30800/TCP   7m
```

Para probar el sistema OpenFaaS, vaya a la dirección IP externa en el puerto 8080, `http://52.186.64.52:8080` en este ejemplo. Se le pedirá que inicie sesión. Para capturar la contraseña, escriba `echo $PASSWORD`.

![UI de OpenFaaS](media/container-service-serverless/openfaas.png)

Por último, instale la CLI de OpenFaaS. En este ejemplo se usaba brew. Consulte la [documentación de la CLI de OpenFaaS][open-faas-cli] para ver más opciones.

```console
brew install faas-cli
```

Establezca `$OPENFAAS_URL` en la dirección IP pública que se encuentra encima.

Inicie sesión con la CLI de Azure:

```console
export OPENFAAS_URL=http://52.186.64.52:8080
echo -n $PASSWORD | ./faas-cli login -g $OPENFAAS_URL -u admin --password-stdin
```

## <a name="create-first-function"></a>Creación de la primera función

Ahora que OpenFaaS está en funcionamiento, cree una función mediante el portal de OpenFaas.

Haga clic en **Deploy New Function** (Implementar nueva función) y busque **Figlet**. Seleccione la función Figlet y haga clic en **Deploy** (Implementar).

![Figlet](media/container-service-serverless/figlet.png)

Use curl para invocar la función. Reemplace la dirección IP en el ejemplo siguiente con la de la puerta de enlace de OpenFaas.

```console
curl -X POST http://52.186.64.52:8080/function/figlet -d "Hello Azure"
```

Salida:

```output
 _   _      _ _            _
| | | | ___| | | ___      / \    _____   _ _ __ ___
| |_| |/ _ \ | |/ _ \    / _ \  |_  / | | | '__/ _ \
|  _  |  __/ | | (_) |  / ___ \  / /| |_| | | |  __/
|_| |_|\___|_|_|\___/  /_/   \_\/___|\__,_|_|  \___|

```

## <a name="create-second-function"></a>Creación de la segunda función

Ahora cree la segunda función. Este ejemplo se implementará con la CLI de OpenFaaS e incluye una imagen de contenedor personalizada y la recuperación de datos de una instancia de Cosmos DB. Antes de crear la función, hay que configurar varios elementos.

En primer lugar, hay que crear un grupo de recursos para Cosmos DB.

```azurecli-interactive
az group create --name serverless-backing --location eastus
```

Implemente una instancia de CosmosDB de tipo `MongoDB`. La instancia necesita un nombre único, actualice `openfaas-cosmos` con un nombre único para su entorno.

```azurecli-interactive
az cosmosdb create --resource-group serverless-backing --name openfaas-cosmos --kind MongoDB
```

Obtenga la cadena de conexión de Cosmos DB y almacénela en una variable.

Actualice el valor del argumento `--resource-group` con el nombre de su grupo de recursos y el argumento `--name` con el de su instancia de Cosmos DB.

```azurecli-interactive
COSMOS=$(az cosmosdb list-connection-strings \
  --resource-group serverless-backing \
  --name openfaas-cosmos \
  --query connectionStrings[0].connectionString \
  --output tsv)
```

Ahora rellene la instancia de Cosmos DB con datos de prueba. Cree un archivo denominado `plans.json` y cópielo en el siguiente código json.

```json
{
    "name" : "two_person",
    "friendlyName" : "Two Person Plan",
    "portionSize" : "1-2 Person",
    "mealsPerWeek" : "3 Unique meals per week",
    "price" : 72,
    "description" : "Our basic plan, delivering 3 meals per week, which will feed 1-2 people.",
    "__v" : 0
}
```

Use la herramienta *mongoimport* para cargar la instancia de CosmosDB con datos.

Si es necesario, instale las herramientas de MongoDB. En el ejemplo siguiente se instalan estas herramientas con brew. Consulte la [documentación de MongoDB][install-mongo] para ver otras opciones.

```console
brew install mongodb
```

Cargue los datos en la base de datos.

```console
mongoimport --uri=$COSMOS -c plans < plans.json
```

Salida:

```output
2018-02-19T14:42:14.313+0000    connected to: localhost
2018-02-19T14:42:14.918+0000    imported 1 document
```

Ejecute el siguiente comando para crear la función. Actualice el valor del argumento `-g` con la dirección de la puerta de enlace de OpenFaaS.

```console
faas-cli deploy -g http://52.186.64.52:8080 --image=shanepeckham/openfaascosmos --name=cosmos-query --env=NODE_ENV=$COSMOS
```

Una vez implementado, debería ver el punto de conexión de OpenFaaS recién creado para la función.

```output
Deployed. 202 Accepted.
URL: http://52.186.64.52:8080/function/cosmos-query
```

Pruebe la función con curl. Actualice la dirección IP con la dirección de puerta de enlace de OpenFaaS.

```console
curl -s http://52.186.64.52:8080/function/cosmos-query
```

Salida:

```json
[{"ID":"","Name":"two_person","FriendlyName":"","PortionSize":"","MealsPerWeek":"","Price":72,"Description":"Our basic plan, delivering 3 meals per week, which will feed 1-2 people."}]
```

También puede probar la función dentro de la UI de OpenFaaS.

![texto alternativo](media/container-service-serverless/OpenFaaSUI.png)

## <a name="next-steps"></a>Pasos siguientes

Para seguir aprendiendo, puede realizar el taller sobre OpenFaaS, un conjunto de laboratorios prácticos que abarcan temas como la creación de su propio bot de GitHub, el consumo de secretos, la visualización de métricas y el escalado automático.

<!-- LINKS - external -->
[install-mongo]: https://docs.mongodb.com/manual/installation/
[kubectl-get]: https://kubernetes.io/docs/reference/generated/kubectl/kubectl-commands#get
[open-faas]: https://www.openfaas.com/
[open-faas-cli]: https://github.com/openfaas/faas-cli
[openfaas-workshop]: https://github.com/openfaas/workshop
