---
title: Implementación de configuraciones mediante GitOps en clústeres de Kubernetes habilitados para Arc (versión preliminar)
services: azure-arc
ms.service: azure-arc
ms.date: 05/19/2020
ms.topic: article
author: mlearned
ms.author: mlearned
description: Uso de GitOps para una configuración de clúster habilitada para Azure Arc (versión preliminar)
keywords: GitOps, Kubernetes, K8s, Azure, Arc, Azure Kubernetes Service, contenedores
ms.openlocfilehash: e25fdf3a51b3e9264c85707df31d3a4d107b25ea
ms.sourcegitcommit: 3d79f737ff34708b48dd2ae45100e2516af9ed78
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/23/2020
ms.locfileid: "87049966"
---
# <a name="deploy-configurations-using-gitops-on-arc-enabled-kubernetes-cluster-preview"></a>Implementación de configuraciones mediante GitOps en clústeres de Kubernetes habilitados para Arc (versión preliminar)

GitOps es la práctica de declarar el estado deseado de configuración de Kubernetes (implementaciones, espacios de nombres, etc.) en un repositorio de Git seguida de una implementación basada en sondeo e incorporación de cambios de estas configuraciones en el clúster mediante un operador. En este documento se habla de la configuración de estos flujos de trabajo en clústeres de Kubernetes habilitados para Azure Arc.

Se realiza el seguimiento de la conexión entre el clúster y uno o varios repositorios de Git en Azure Resource Manager como un recurso de extensión `sourceControlConfiguration`. Las propiedades del recurso `sourceControlConfiguration` representan dónde y cómo deben fluir los recursos de Kubernetes desde Git al clúster. Los datos de `sourceControlConfiguration` se almacenan cifrados en reposo en una base de datos de Azure Cosmos DB para garantizar su confidencialidad.

El elemento `config-agent` que se ejecuta en el clúster es responsable de inspeccionar los recursos de extensión de `sourceControlConfiguration` nuevos o actualizados en el recurso de Kubernetes habilitado para Azure Arc, de implementar un operador de Flux para inspeccionar el repositorio de Git y de propagar las actualizaciones realizadas en `sourceControlConfiguration`. Incluso es posible crear varios recursos de `sourceControlConfiguration` con ámbito de `namespace` en el mismo clúster de Kubernetes habilitado para Azure Arc para lograr el multiinquilinato. En ese caso, cada operador solo puede implementar configuraciones en su espacio de nombres respectivo.

El repositorio de Git puede contener cualquier recurso de Kubernetes válido, incluidos espacios de nombres, ConfigMaps, implementaciones, DaemonSets, etc.  También puede contener gráficos de Helm para implementar aplicaciones. Un conjunto común de escenarios incluye la definición de una configuración de línea de base para la organización, que podría incluir roles y enlaces de RBAC comunes, agentes de supervisión o registro, o servicios para todo el clúster.

Se puede usar el mismo patrón para administrar una colección mayor de clústeres, que se puede implementar en entornos heterogéneos. Por ejemplo, puede tener un repositorio que defina la configuración de línea de base para la organización y aplicarla a decenas de clústeres de Kubernetes a la vez. [Azure Policy puede automatizar](use-azure-policy.md) la creación de un elemento `sourceControlConfiguration` con un conjunto específico de parámetros en todos los recursos de Kubernetes habilitados para Azure Arc en un ámbito (suscripción o grupo de recursos).

En esta guía de introducción se describirá la aplicación de un conjunto de configuraciones con ámbito de administración de clústeres.

## <a name="create-a-configuration"></a>Creación de una configuración

- Repositorio de ejemplo: <https://github.com/Azure/arc-k8s-demo>

El repositorio de ejemplo está estructurado en torno al rol de un operador de clúster que le gustaría aprovisionar algunos espacios de nombres, implementar una carga de trabajo común y proporcionar una configuración específica del equipo. Mediante este repositorio se crean los siguientes recursos en el clúster:

**Espacios de nombres:** `cluster-config`, `team-a`, `team-b`
**Implementación:** `cluster-config/azure-vote`
**ConfigMap:** `team-a/endpoints`

`config-agent` sondea Azure en busca de elementos `sourceControlConfiguration` nuevos o actualizados cada 30 segundos, que es el tiempo máximo que tarda `config-agent` en seleccionar una configuración nueva o actualizada.
Si va a asociar un repositorio privado a `sourceControlConfiguration`, asegúrese de realizar además los pasos de [Aplicación de la configuración desde un repositorio de Git privado](#apply-configuration-from-a-private-git-repository).

### <a name="using-azure-cli"></a>Uso de la CLI de Azure

Con la extensión de la CLI de Azure para `k8sconfiguration`, se va a vincular el clúster conectado a un [repositorio de Git de ejemplo](https://github.com/Azure/arc-k8s-demo). A esta configuración se le asignará el nombre `cluster-config`, se le indicará al agente que implemente el operador en el espacio de nombres `cluster-config` y se concederán permisos `cluster-admin` al operador.

```console
az k8sconfiguration create \
    --name cluster-config \
    --cluster-name AzureArcTest1 --resource-group AzureArcTest \
    --operator-instance-name cluster-config --operator-namespace cluster-config \
    --repository-url https://github.com/Azure/arc-k8s-demo \
    --scope cluster --cluster-type connectedClusters
```

**Salida:**

```console
Command group 'k8sconfiguration' is in preview. It may be changed/removed in a future release.
{
  "complianceStatus": {
    "ComplianceStatus": 1,
    "clientAppliedTime": null,
    "level": 3,
    "message": "{\"OperatorMessage\":null,\"ClusterState\":null}"
  },
  "configKind": "Git",
  "configName": "cluster-config",
  "configOperator": {
    "operatorInstanceName": "cluster-config",
    "operatorNamespace": "cluster-config",
    "operatorParams": "--git-readonly",
    "operatorScope": "cluster",
    "operatorType": "flux"
  },
  "configType": "",
  "id": null,
  "name": null,
  "operatorInstanceName": null,
  "operatorNamespace": null,
  "operatorParams": null,
  "operatorScope": null,
  "operatorType": null,
  "providerName": "ConnectedClusters",
  "provisioningState": "Succeeded",
  "repositoryPublicKey": null,
  "repositoryUrl": null,
  "sourceControlConfiguration": {
    "repositoryPublicKey": "",
    "repositoryUrl": "git://github.com/Azure/arc-k8s-demo.git"
  },
  "type": null
}
```

#### <a name="repository-url-parameter"></a>Parámetro repository-url

Estos son los escenarios admitidos para el valor del parámetro --repository-url.

| Escenario | Formato | Descripción |
| ------------- | ------------- | ------------- |
| Repositorio privado de GitHub: SSH | git@github.com:username/repo | Par de claves SSH generado por Flux.  El usuario debe agregar la clave pública a la cuenta de GitHub como clave de implementación. |
| Repositorio público de GitHub | `http://github.com/username/repo` o git://github.com/username/repo   | Repositorio público de Git  |

Estos escenarios son compatibles con Flux pero todavía no lo son con sourceControlConfiguration. 

| Escenario | Formato | Descripción |
| ------------- | ------------- | ------------- |
| Repositorio privado de GitHub: HTTPS | `https://github.com/username/repo` | Flux no genera el par de claves SSH.  [Instrucciones](https://docs.fluxcd.io/en/1.17.0/guides/use-git-https.html) |
| Host privado de Git | user@githost:path/to/repo | [Instrucciones](https://docs.fluxcd.io/en/1.18.0/guides/use-private-git-host.html) |
| Repositorio privado de GitHub: SSH (Bring Your Own Key) | git@github.com:username/repo | [Uso de un par de claves SSH propio](https://docs.fluxcd.io/en/1.17.0/guides/provide-own-ssh-key.html) |


#### <a name="additional-parameters"></a>Parámetros adicionales

Para personalizar la creación de la configuración, estos son algunos parámetros adicionales:

`--enable-helm-operator`: *Opcional* modificador para habilitar la compatibilidad con las implementaciones de gráficos de Helm.

`--helm-operator-chart-values`: *Opcional* valores de gráfico para el operador de Helm (si está habilitado).  Por ejemplo, "--set helm.versions=v3".

`--helm-operator-chart-version`: *Opcional* versión del gráfico para el operador de Helm (si está habilitado). Valor predeterminado: "0.6.0".

`--operator-namespace`: *Opcional* nombre para el espacio de nombres del operador. Valor predeterminado: "default"

`--operator-params`: *Opcional* parámetros para el operador. Se debe proporcionar entre comillas simples. Por ejemplo: ```--operator-params='--git-readonly --git-path=releases' ```

Opciones admitidas en --operator-params

| Opción | Descripción |
| ------------- | ------------- |
| --git-branch  | Rama del repositorio de Git que se va a usar para los manifiestos de Kubernetes. El valor predeterminado es "master". |
| --git-path  | Ruta de acceso relativa dentro del repositorio de Git para que Flux localice manifiestos de Kubernetes. |
| --git-readonly | El repositorio de Git se considerará de solo lectura; Flux no intentará escribir en él. |
| --manifest-generation  | Si está habilitado, Flux buscará .flux.yaml y ejecutará Kustomize u otros generadores de manifiestos. |
| --git-poll-interval  | Período en el que se sondea el repositorio de Git en busca de confirmaciones nuevas. El valor predeterminado es "5m" (5 minutos). |
| --sync-garbage-collection  | Si está habilitado, Flux eliminará los recursos que ha creado, pero que ya no están presentes en Git. |
| --git-label  | Etiqueta para realizar el seguimiento del progreso de la sincronización, que se usa para etiquetar la rama de Git.  El valor predeterminado es "flux-sync". |
| --git-user  | Nombre de usuario para la confirmación de Git. |
| --git-email  | Correo electrónico que se va a usar para la confirmación de Git. |

* Si no se establece "--git-user" o "--git-email" (lo que significa que no quiere que Flux escriba en el repositorio), --git-readonly se establecerá de forma automática (si todavía no lo ha establecido).

* Si enableHelmOperator es true, las cadenas operatorInstanceName + operatorNamespace no pueden superar los 47 caracteres combinados.  Si no cumple este límite, aparece el siguiente error:

   ```console
   {"OperatorMessage":"Error: {failed to install chart from path [helm-operator] for release [<operatorInstanceName>-helm-<operatorNamespace>]: err [release name \"<operatorInstanceName>-helm-<operatorNamespace>\" exceeds max length of 53]} occurred while doing the operation : {Installing the operator} on the config","ClusterState":"Installing the operator"}
   ```

Para obtener más información, vea la [documentación de Flux](https://aka.ms/FluxcdReadme).

> [!TIP]
> Es posible crear un elemento sourceControlConfiguration en Azure Portal y en la pestaña **Configuraciones** de la hoja del recurso de Kubernetes habilitado para Azure Arc.

## <a name="validate-the-sourcecontrolconfiguration"></a>Validación de sourceControlConfiguration

Con la CLI de Azure, valide que `sourceControlConfiguration` se ha creado correctamente.

```console
az k8sconfiguration show --resource-group AzureArcTest --name cluster-config --cluster-name AzureArcTest1 --cluster-type connectedClusters
```

Tenga en cuenta que el recurso `sourceControlConfiguration` se actualiza con el estado de cumplimiento, los mensajes y la información de depuración.

**Salida:**

```console
Command group 'k8sconfiguration' is in preview. It may be changed/removed in a future release.
{
  "complianceStatus": {
    "complianceState": "Installed",
    "lastConfigApplied": "2019-12-05T05:34:41.481000",
    "message": "...",
    "messageLevel": "3"
  },
  "id": "/subscriptions/57ac26cf-a9f0-4908-b300-9a4e9a0fb205/resourceGroups/AzureArcTest/providers/Microsoft.Kubernetes/connectedClusters/AzureArcTest1/providers/Microsoft.KubernetesConfiguration/sourceControlConfigurations/cluster-config",
  "name": "cluster-config",
  "operatorInstanceName": "cluster-config",
  "operatorNamespace": "cluster-config",
  "operatorParams": "--git-readonly",
  "operatorScope": "cluster",
  "operatorType": "Flux",
  "provisioningState": "Succeeded",
  "repositoryPublicKey": "...",
  "repositoryUrl": "git://github.com/Azure/arc-k8s-demo.git",
  "resourceGroup": "AzureArcTest",
  "type": "Microsoft.KubernetesConfiguration/sourceControlConfigurations"
}
```

Cuando se crea `sourceControlConfiguration`, se realizan varias operaciones en segundo plano:

1. Azure Arc `config-agent` supervisa Azure Resource Manager para configuraciones nuevas o actualizadas (`Microsoft.KubernetesConfiguration/sourceControlConfiguration`)
1. `config-agent` percibe la nueva configuración de `Pending`
1. `config-agent` lee las propiedades de configuración y se prepara para implementar una instancia administrada de `flux`
    * `config-agent` crea el espacio de nombres de destino.
    * `config-agent` prepara una cuenta de servicio de Kubernetes con el permiso adecuado (con ámbito `cluster` o `namespace`)
    * `config-agent` implementa una instancia de `flux`
    * `flux` genera una clave SSH y registra la clave pública
1. `config-agent` devuelve el estado a `sourceControlConfiguration`

Mientras se produce el proceso de aprovisionamiento, `sourceControlConfiguration` se desplazará a través de algunos cambios de estado. Supervise el progreso con el comando `az k8sconfiguration show ...` anterior:

1. `complianceStatus` -> `Pending`: representa los estados inicial y en curso.
1. `complianceStatus` -> `Installed`: `config-agent` ha podido configurar correctamente el clúster e implementar `flux` sin errores
1. `complianceStatus` -> `Failed`: `config-agent` ha detectado un error al implementar `flux`, los detalles deben estar disponibles en el cuerpo de respuesta de `complianceStatus.message`

## <a name="apply-configuration-from-a-private-git-repository"></a>Aplicación de la configuración desde un repositorio de Git privado

Si va a usar un repositorio de Git privado, debe realizar una tarea más para cerrar el bucle: agregar la clave pública generada por `flux` como **clave de implementación** en el repositorio.

**Obtención de la clave pública mediante la CLI de Azure**

```console
$ az k8sconfiguration show --resource-group <resource group name> --cluster-name <connected cluster name> --name <configuration name> --query 'repositoryPublicKey'
Command group 'k8sconfiguration' is in preview. It may be changed/removed in a future release.
"ssh-rsa AAAAB3NzaC1yc2EAAAADAQABAREDACTED"
```

**Obtención de la clave pública desde Azure Portal**

1. En Azure Portal, vaya al recurso de clúster conectado.
2. En la página del recurso, seleccione "Configuraciones" y vea la lista de configuraciones de este clúster.
3. Seleccione la configuración que usa el repositorio de Git privado.
4. En la parte inferior de la ventana contextual que se abre, copie la **clave pública del repositorio**.

**Adición de la clave pública como clave de implementación al repositorio de Git**

1. Abra GitHub, navegue hasta la bifurcación, después a **Settings** (Configuración) y luego a **Deploy keys** (Implementar claves).
2. Haga clic en **Add deploy key** (Agregar clave de implementación).
3. Proporcione un título.
4. Active **Allow write access** (Permitir acceso de escritura).
5. Pegue la clave pública (menos las comillas).
6. Haga clic en **Add key** (Agregar clave).

Vea la documentación de GitHub para obtener más información sobre cómo administrar estas claves.

**Si usa un repositorio de Azure DevOps, agregue la clave a las claves SSH**

1. En **User Settings** (Configuración de usuario) en la parte superior derecha (junto a la imagen de perfil), haga clic en **SSH public keys** (Claves públicas de SSH).
1. Seleccione **+ New Key**(+ Nueva clave).
1. Proporcione un nombre.
1. Pegue la clave pública sin las comillas.
1. Haga clic en **Agregar**.

## <a name="validate-the-kubernetes-configuration"></a>Validación de la configuración de Kubernetes

Una vez que `config-agent` ha instalado la instancia de `flux`, los recursos que se mantienen en el repositorio de Git deben empezar a fluir al clúster. Compruebe que se hayan creado los espacios de nombres, las implementaciones y los recursos:

```console
kubectl get ns --show-labels
```

**Salida:**

```console
NAME              STATUS   AGE    LABELS
azure-arc         Active   24h    <none>
cluster-config    Active   177m   <none>
default           Active   29h    <none>
itops             Active   177m   fluxcd.io/sync-gc-mark=sha256.9oYk8yEsRwWkR09n8eJCRNafckASgghAsUWgXWEQ9es,name=itops
kube-node-lease   Active   29h    <none>
kube-public       Active   29h    <none>
kube-system       Active   29h    <none>
team-a            Active   177m   fluxcd.io/sync-gc-mark=sha256.CS5boSi8kg_vyxfAeu7Das5harSy1i0gc2fodD7YDqA,name=team-a
team-b            Active   177m   fluxcd.io/sync-gc-mark=sha256.vF36thDIFnDDI2VEttBp5jgdxvEuaLmm7yT_cuA2UEw,name=team-b
```

Se puede ver que se han creado los espacios de nombres `team-a`, `team-b`, `itops` y `cluster-config`.

El operador `flux` se ha implementado en el espacio de nombres `cluster-config`, como indica `sourceControlConfig`:

```console
kubectl -n cluster-config get deploy  -o wide
```

**Salida:**

```console
NAME             READY   UP-TO-DATE   AVAILABLE   AGE   CONTAINERS   IMAGES                         SELECTOR
cluster-config   1/1     1            1           3h    flux         docker.io/fluxcd/flux:1.16.0   instanceName=cluster-config,name=flux
memcached        1/1     1            1           3h    memcached    memcached:1.5.15               name=memcached
```

## <a name="further-exploration"></a>Exploración adicional

Puede explorar los demás recursos implementados como parte del repositorio de configuración:

```console
kubectl -n team-a get cm -o yaml
kubectl -n itops get all
```

## <a name="delete-a-configuration"></a>Eliminación de una configuración

Elimine `sourceControlConfiguration` mediante la CLI de Azure o Azure Portal.  Después de iniciar el comando de eliminación, el recurso `sourceControlConfiguration` se elimina inmediatamente en Azure, pero la eliminación completa de los objetos asociados del clúster puede tardar hasta una hora (hay un elemento de trabajo pendiente para acortar este lapso de tiempo).

> [!NOTE]
> Después de crear un elemento sourceControlConfiguration con ámbito de espacio de nombres, los usuarios con el enlace de rol `edit` en el espacio de nombres pueden implementar cargas de trabajo en este espacio de nombres. Cuando este `sourceControlConfiguration` con ámbito de espacio de nombres se elimina, el espacio de nombres se deja intacto y no se elimina para evitar la interrupción de estas otras cargas de trabajo.
> Los cambios en el clúster que son el resultado de implementaciones desde el repositorio de Git sometido a seguimiento no se eliminan cuando se elimina `sourceControlConfiguration`.

```console
az k8sconfiguration delete --name '<config name>' -g '<resource group name>' --cluster-name '<cluster name>' --cluster-type connectedClusters
```

**Salida:**

```console
Command group 'k8sconfiguration' is in preview. It may be changed/removed in a future release.
```

## <a name="next-steps"></a>Pasos siguientes

- [Uso de Helm con configuración de control de código fuente](./use-gitops-with-helm.md)
- [Uso de Azure Policy para controlar la configuración del clúster](./use-azure-policy.md)
