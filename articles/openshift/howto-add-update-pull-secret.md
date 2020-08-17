---
title: Adición o actualización del secreto de extracción de Red Hat en un clúster de la versión 4 de Red Hat OpenShift en Azure
description: Agregue o actualice el secreto de extracción de Red Hat en clústeres de ARO 4.x existentes.
author: sakthi-vetrivel
ms.author: suvetriv
ms.service: container-service
ms.topic: conceptual
ms.date: 05/21/2020
keywords: secreto de extracción, aro, openshift, red hat
ms.openlocfilehash: 3351052db63f095bfca5f0b91f26e1013319c582
ms.sourcegitcommit: 3d79f737ff34708b48dd2ae45100e2516af9ed78
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/23/2020
ms.locfileid: "87095333"
---
# <a name="add-or-update-your-red-hat-pull-secret-on-an-azure-red-hat-openshift-4-cluster"></a>Adición o actualización del secreto de extracción de Red Hat en un clúster de la versión 4 de Red Hat OpenShift en Azure

En esta guía se describe cómo agregar o actualizar un secreto de extracción de Red Hat para un clúster existente de la versión 4.x de Red Hat OpenShift en Azure.

Si va a crear un clúster por primera vez, puede agregar el secreto de extracción al crear el clúster. Para obtener más información sobre la creación de un clúster de ARO con un secreto de extracción de Red Hat, consulte [Creación de un clúster de la versión 4 de Red Hat OpenShift en Azure](tutorial-create-cluster.md#get-a-red-hat-pull-secret-optional).

## <a name="before-you-begin"></a>Antes de empezar

En esta guía se da por supuesto que tiene un clúster existente de la versión 4 de Red Hat OpenShift en Azure. Asegúrese de tener acceso de administrador al clúster.

## <a name="prepare-your-pull-secret"></a>Preparación del secreto de extracción
Cuando se crea un clúster de ARO sin agregar un secreto de extracción de Red Hat, aún se crea un secreto de extracción automáticamente en el clúster. Sin embargo, este secreto de extracción no se rellena por completo.

Esta sección le guía por los pasos de actualización de ese secreto de extracción con valores adicionales del secreto de extracción de Red Hat.

1. Capture el secreto denominado `pull-secret` en el espacio de nombres openshift-config y guárdelo en un archivo independiente a través del comando siguiente: 

    ```console
    oc get secrets pull-secret -n openshift-config -o template='{{index .data ".dockerconfigjson"}}' | base64 -d > pull-secret.json
    ```

    La salida debería ser similar a la siguiente (tenga en cuenta que se ha quitado el valor del secreto real):

    ```json
    {
        "auths": {
            "arosvc.azurecr.io": {
                "auth": "<my-aroscv.azurecr.io-secret>"
            }
        }
    }
    ```

2. Vaya al [portal del administrador de clústeres de Red Hat OpenShift](https://cloud.redhat.com/openshift/install/azure/aro-provisioned) y haga clic en **Download pull secret** (Descargar secreto de extracción). El secreto de extracción de Red Hat tendrá un aspecto similar al siguiente (tenga en cuenta que se han quitado los valores de secreto reales):

    ```json
    {
        "auths": {
            "cloud.openshift.com": {
                "auth": "<my-crc-secret>",
                "email": "klamenzo@redhat.com"
            },
            "quay.io": {
                "auth": "<my-quayio-secret>",
                "email": "klamenzo@redhat.com"
            },
            "registry.connect.redhat.com": {
                "auth": "<my-registry.connect.redhat.com-secret>",
                "email": "klamenzo@redhat.com"
            },
            "registry.redhat.io": {
                "auth": "<my-registry.redhat.io-secret>",
                "email": "klamenzo@redhat.com"
            }
        }
    }
    ```

3. Edite el archivo del secreto de extracción que recibió del clúster agregando las entradas que se encuentran en el secreto de extracción de Red Hat. 

    > [!IMPORTANT]
    > La inclusión de la entrada `cloud.openshift.com` del secreto de extracción de Red Hat hará que el clúster empiece a enviar datos de telemetría a Red Hat. Incluya solo esta sección si desea enviar datos de telemetría. De lo contrario, no incluya la siguiente sección.
    > ```json
    > {
    >         "cloud.openshift.com": {
    >             "auth": "<my-crc-secret>",
    >             "email": "klamenzo@redhat.com"
    >         }
    > ```

    > [!CAUTION]
    > No quite ni modifique la entrada `arosvc.azurecr.io` del secreto de extracción. Esta sección es necesaria para que el clúster funcione correctamente.
    ```json
    "arosvc.azurecr.io": {
                "auth": "<my-aroscv.azurecr.io-secret>"
            }
    ```

    El archivo final debería tener un aspecto similar al siguiente (tenga en cuenta que se han quitado los valores de secreto reales):

    ```json
    {
        "auths": {
            "cloud.openshift.com": {
                "auth": "<my-crc-secret>",
                "email": "klamenzo@redhat.com"
            },
            "quay.io": {
                "auth": "<my-quayio-secret>",
                "email": "klamenzo@redhat.com"
            },
            "registry.connect.redhat.com": {
                "auth": "<my-registry.connect.redhat.com-secret>",
                "email": "klamenzo@redhat.com"
            },
            "registry.redhat.io": {
                "auth": "<my-registry.redhat.io-secret>",
                "email": "klamenzo@redhat.com"
            },
            "arosvc.azurecr.io": {
                "auth": "<my-aroscv.azurecr.io-secret>"
            }
        }
    }
    ```

4. Asegúrese de que el archivo sea un JSON válido. Hay muchas maneras de validar el archivo JSON. En el ejemplo siguiente se usa jq:
    ```json
    cat pull-secret.json | jq
    ```

    > [!NOTE]
    > Si hay un error en el archivo, puede verse `parse error`.

## <a name="add-your-pull-secret-to-your-cluster"></a>Adición del secreto de extracción al clúster

Ejecute el comando siguiente para actualizar el secreto de extracción:

> [!NOTE]
> Al ejecutar este comando, los nodos del clúster se reiniciarán de uno en uno a medida que se actualicen. 

```console
oc set data secret/pull-secret -n openshift-config --from-file=.dockerconfigjson=./pull-secret.json
```

Una vez que se haya establecido el secreto, está listo para habilitar los operadores certificados en Red Hat.

### <a name="modify-the-configuration-files"></a>Modificación de los archivos de configuración

Modifique los siguientes objetos para habilitar los operadores de Red Hat.

En primer lugar, modifique el archivo de configuración del operador de ejemplos. Luego, puede ejecutar el comando siguiente para editar el archivo de configuración:

```
oc edit configs.samples.operator.openshift.io/cluster -o yaml
```

Cambie los valores de `spec.architectures.managementState` y `status.architecture.managementState` de `Removed` a `Managed`. 

El siguiente fragmento de código de YAML muestra solo las secciones pertinentes del archivo YAML editado.

```yaml
apiVersion: samples.operator.openshift.io/v1
kind: Config
metadata:
  
  ...
  
spec:
  architectures:
  - x86_64
  managementState: Managed
status:
  architectures:

  ...

  managementState: Managed
  version: 4.3.27
```

En segundo lugar, ejecute el comando siguiente para editar el archivo de configuración del centro de operadores:  

```console
oc edit operatorhub cluster -o yaml
```

Cambie los valores de `Spec.Sources.Disabled` y `Status.Sources.Disabled` de `true` a `false` para cualquier origen que desee habilitar.

El siguiente fragmento de código de YAML muestra solo las secciones pertinentes del archivo YAML editado.

```yaml
Name:         cluster

...
                 dd3310b9-e520-4a85-98e5-8b4779ee0f61
Spec:
  Sources:
    Disabled:  false
    Name:      certified-operators
    Disabled:  false
    Name:      redhat-operators
Status:
  Sources:
    Disabled:  false
    Name:      certified-operators
    Status:    Success
    Disabled:  false
    Name:      community-operators
    Status:    Success
    Disabled:  false
    Name:      redhat-operators
    Status:    Success
Events:        <none>
```

Guarde el archivo para aplicar las modificaciones.

## <a name="validate-that-your-secret-is-working"></a>Validación de que el secreto funciona

Después de agregar el secreto de extracción y modificar los archivos de configuración correctos, el clúster puede tardar varios minutos en actualizarse. Para comprobar que el clúster se ha actualizado, ejecute el siguiente comando para mostrar los operadores certificados y los orígenes de operadores de Red Hat disponibles:

```console
$ oc get catalogsource -A
NAMESPACE               NAME                  DISPLAY               TYPE   PUBLISHER   AGE
openshift-marketplace   certified-operators   Certified Operators   grpc   Red Hat     10s
openshift-marketplace   community-operators   Community Operators   grpc   Red Hat     18h
openshift-marketplace   redhat-operators      Red Hat Operators     grpc   Red Hat     11s
```

Si no ve los operadores certificados y los operadores de Red Hat, espere unos minutos y vuelva a intentarlo.

Para asegurarse de que el secreto de extracción se ha actualizado y funciona correctamente, abra OperatorHub y compruebe si hay algún operador de Red Hat verificado. Por ejemplo, compruebe si el operador OpenShift Container Storage está disponible y vea si tiene permisos para instalar.

## <a name="next-steps"></a>Pasos siguientes
Para obtener más información sobre los secretos de extracción de Red Hat, consulte [Using image pull secrets](https://docs.openshift.com/container-platform/4.5/openshift_images/managing_images/using-image-pull-secrets.html).

Para obtener más información acerca de Red Hat OpenShift 4, consulte [Azure Red Hat OpenShift 4](https://docs.openshift.com/aro/4/welcome/index.html).