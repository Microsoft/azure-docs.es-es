---
title: Integración de Azure App Configuration con la implementación de Kubernetes mediante Helm
description: Aprenda a usar configuraciones dinámicas en la implementación de Kubernetes con Helm.
services: azure-app-configuration
author: shenmuxiaosen
manager: zhenlan
ms.service: azure-app-configuration
ms.topic: tutorial
ms.date: 04/14/2020
ms.author: shuawan
ms.openlocfilehash: aac42e6f782ac1e939ff955c5811238f99e703eb
ms.sourcegitcommit: 62717591c3ab871365a783b7221851758f4ec9a4
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/22/2020
ms.locfileid: "83725676"
---
# <a name="integrate-with-kubernetes-deployment-using-helm"></a>Integración con la implementación de Kubernetes mediante Helm

Helm proporciona una manera de definir, instalar y actualizar las aplicaciones que se ejecutan en Kubernetes. Un gráfico de Helm contiene la información necesaria para crear una instancia de una aplicación de Kubernetes. La configuración se almacena fuera del propio gráfico, en un archivo denominado *values.yaml*. 

Durante el proceso de lanzamiento, Helm combina el gráfico con la configuración adecuada para ejecutar la aplicación. Por ejemplo, se puede hacer referencia a las variables definidas en *values.yaml* como variables de entorno dentro de los contenedores en ejecución. Helm también admite la creación de secretos de Kubernetes, que se pueden montar como volúmenes de datos o exponerse como variables de entorno.

Puede invalidar los valores almacenados en *values.yaml* si proporciona archivos de configuración basados en YAML adicionales en la línea de comandos al ejecutar Helm. Azure App Configuration admite la exportación de valores de configuración a archivos YAML. La integración de esta funcionalidad de exportación en la implementación permite a las aplicaciones de Kubernetes aprovechar los valores de configuración almacenados en App Configuration.

En este tutorial, aprenderá a:
> [!div class="checklist"]
> * Usar valores de App Configuration al implementar una aplicación en Kubernetes con Helm.
> * Crear un secreto de Kubernetes basado en una referencia de Key Vault en App Configuration.

En este tutorial se da por supuesto que tiene conocimientos básicos sobre la administración de Kubernetes con Helm. Consulte más información sobre la instalación de aplicaciones con Helm en [Azure Kubernetes Service (AKS)](https://docs.microsoft.com/azure/aks/kubernetes-helm).

## <a name="prerequisites"></a>Prerrequisitos

- [!INCLUDE [quickstarts-free-trial-note](../../includes/quickstarts-free-trial-note.md)]
- Instalar la [CLI de Azure](https://docs.microsoft.com/cli/azure/install-azure-cli?view=azure-cli-latest) (versión 2.4.0 o posterior)
- Instalar [Helm](https://helm.sh/docs/intro/install/) (versión 2.14.0 o posterior)
- Un clúster de Kubernetes.

## <a name="create-an-app-configuration-store"></a>Creación de un almacén de App Configuration

[!INCLUDE [azure-app-configuration-create](../../includes/azure-app-configuration-create.md)]

6. Seleccione **Explorador de configuración** > **Crear** para agregar los siguientes pares clave-valor:

    | Clave | Value |
    |---|---|
    | settings.color | Blanco |
    | settings.message | Datos de Azure App Configuration |

    Deje **Etiqueta** y **Tipo de contenido** en blanco, por ahora.

## <a name="add-a-key-vault-reference-to-app-configuration"></a>Incorporación de una referencia de Key Vault a App Configuration
1. Inicie sesión en [Azure Portal ](https://portal.azure.com) y agregue un secreto a [Key Vault](https://docs.microsoft.com/azure/key-vault/secrets/quick-create-portal#add-a-secret-to-key-vault) con el nombre **Password** y el valor **miContraseña**. 
2. Seleccione la instancia del almacén de App Configuration que creó en la sección anterior.

3. Seleccione **Explorador de configuración**.

4. Seleccione **+ Crear** > **Referencia del almacén de claves** y, a continuación, especifique los valores siguientes:
    - **Clave**: Seleccione **secrets.password**.
    - **Etiqueta**: deje este valor en blanco.
    - **Suscripción**, **Grupo de recursos** y **Key Vault**: escriba los valores correspondientes a los del almacén de claves que creó en el paso anterior.
    - **Secreto**: seleccione el secreto llamado **Password** que creó en la sección anterior.

## <a name="create-helm-chart"></a>Creación de un gráfico de Helm ##
En primer lugar, cree un gráfico de Helm de ejemplo con el siguiente comando:
```console
helm create mychart
```

Helm crea un directorio nuevo denominado mychart con la estructura que se muestra a continuación. 

> [!TIP]
> Siga esta [guía de gráficos](https://helm.sh/docs/chart_template_guide/getting_started/) para más información.

```
mychart
|-- Chart.yaml
|-- charts
|-- templates
|   |-- NOTES.txt
|   |-- _helpers.tpl
|   |-- deployment.yaml
|   |-- ingress.yaml
|   `-- service.yaml
`-- values.yaml
```

A continuación, actualice la sección de **spec:template:spec:containers** del archivo *deployment.yaml*. En el fragmento de código siguiente se agregan dos variables de entorno al contenedor. Establecerá sus valores dinámicamente en el momento de la implementación.

```yaml
env:
- name: Color
    value: {{ .Values.settings.color }}
- name: Message
    value: {{ .Values.settings.message }}
``` 

El archivo *deployment.yaml* completo después de la actualización debe ser similar al siguiente.

```yaml
apiVersion: apps/v1beta2
kind: Deployment
metadata:
  name: {{ include "mychart.fullname" . }}
  labels:
    app.kubernetes.io/name: {{ include "mychart.name" . }}
    helm.sh/chart: {{ include "mychart.chart" . }}
    app.kubernetes.io/instance: {{ .Release.Name }}
    app.kubernetes.io/managed-by: {{ .Release.Service }}
spec:
  replicas: {{ .Values.replicaCount }}
  selector:
    matchLabels:
      app.kubernetes.io/name: {{ include "mychart.name" . }}
      app.kubernetes.io/instance: {{ .Release.Name }}
  template:
    metadata:
      labels:
        app.kubernetes.io/name: {{ include "mychart.name" . }}
        app.kubernetes.io/instance: {{ .Release.Name }}
    spec:
      containers:
        - name: {{ .Chart.Name }}
          image: "{{ .Values.image.repository }}:{{ .Values.image.tag }}"
          imagePullPolicy: {{ .Values.image.pullPolicy }}
          env:
            - name: Color
              value: {{ .Values.settings.color }}
            - name: Message
              value: {{ .Values.settings.message }}
          ports:
            - name: http
              containerPort: 80
              protocol: TCP
          livenessProbe:
            httpGet:
              path: /
              port: http
          readinessProbe:
            httpGet:
              path: /
              port: http
          resources:
{{ toYaml .Values.resources | indent 12 }}
    {{- with .Values.nodeSelector }}
      nodeSelector:
{{ toYaml . | indent 8 }}
    {{- end }}
    {{- with .Values.affinity }}
      affinity:
{{ toYaml . | indent 8 }}
    {{- end }}
    {{- with .Values.tolerations }}
      tolerations:
{{ toYaml . | indent 8 }}
    {{- end }}
```

Para almacenar datos confidenciales como secretos de Kubernetes, agregue un archivo *secrets.yaml* en la carpeta de plantillas.

> [!TIP]
> Obtenga más información sobre cómo usar [secretos de Kubernetes](https://kubernetes.io/docs/concepts/configuration/secret/#using-secrets).

```yaml
apiVersion: v1
kind: Secret
metadata:
  name: mysecret
type: Opaque
data:
  password: {{ .Values.secrets.password | b64enc }}
```

Por último, actualice el archivo *values.yaml* con el siguiente contenido para proporcionar de forma opcional los valores predeterminados de las opciones de configuración y los secretos a los que se hace referencia en los archivos *deployment.yaml* y *secrets.yaml*. Los valores reales se sobrescribirán con la configuración extraída de App Configuration.

```yaml
# settings will be overwritten by App Configuration
settings:
    color: red
    message: myMessage
```

## <a name="pass-configuration-from-app-configuration-in-helm-install"></a>Paso de la configuración desde App Configuration en la instalación de Helm ##
En primer lugar, descargue la configuración de App Configuration en un archivo *myConfig.yaml*. Use un filtro de clave para descargar solo las claves que comiencen por **settings.** . Si en su caso el filtro de clave no es suficiente para excluir las claves de las referencias de Key Vault, puede usar el argumento **--skip-keyvault** para excluirlas. 

> [!TIP]
> Obtenga más información sobre el [comando export](https://docs.microsoft.com/cli/azure/appconfig/kv?view=azure-cli-latest#az-appconfig-kv-export). 

```azurecli-interactive
az appconfig kv export -n myAppConfiguration -d file --path myConfig.yaml --key "settings.*"  --separator "." --format yaml
```

A continuación, descargue los secretos en un archivo denominado *mySecrets.yaml*. El argumento de línea de comandos **--resolve-keyvault** resuelve las referencias de Key Vault mediante la recuperación de los valores reales de Key Vault. Deberá ejecutar este comando con credenciales que tengan permisos de acceso a la instancia de Key Vault correspondiente.

> [!WARNING]
> Como este archivo contiene información confidencial, conserve el archivo con cuidado y elimínelo cuando ya no lo necesite.

```azurecli-interactive
az appconfig kv export -n myAppConfiguration -d file --path mySecrets.yaml --key "secrets.*" --separator "." --resolve-keyvault --format yaml
```

Use el argumento **-f** de la actualización de Helm para pasar los dos archivos de configuración que ha creado. Invalidarán los valores de configuración definidos en *values.yaml* con los valores exportados de App Configuration.

```console
helm upgrade --install -f myConfig.yaml -f mySecrets.yaml "example" ./mychart 
```

También puede usar el argumento **--set** para que la actualización de Helm pase valores de clave literal. El uso del argumento **--set** es una buena manera de evitar la persistencia de datos confidenciales en el disco. 

```powershell
$secrets = az appconfig kv list -n myAppConfiguration --key "secrets.*" --resolve-keyvault --query "[*].{name:key, value:value}" | ConvertFrom-Json

foreach ($secret in $secrets) {
  $keyvalues += $secret.name + "=" + $secret.value + ","
}

if ($keyvalues){
  $keyvalues = $keyvalues.TrimEnd(',')
  helm upgrade --install --set $keyvalues "example" ./mychart 
}
else{
  helm upgrade --install "example" ./mychart 
}

```

Compruebe que las configuraciones y los secretos se establecieron correctamente; para ello, acceda al [panel de Kubernetes](https://docs.microsoft.com/azure/aks/kubernetes-dashboard). Verá que los valores **color** y **message** de App Configuration se rellenaron en las variables de entorno del contenedor.

![Inicio de la aplicación del artículo de inicio rápido en un entorno local](./media/kubernetes-dashboard-env-variables.png)

Un secreto, **password**, se almacena como referencia de Key Vault en App Configuration y también se agregó a los secretos de Kubernetes. 

![Inicio de la aplicación del artículo de inicio rápido en un entorno local](./media/kubernetes-dashboard-secrets.png)

## <a name="clean-up-resources"></a>Limpieza de recursos

[!INCLUDE [azure-app-configuration-cleanup](../../includes/azure-app-configuration-cleanup.md)]

## <a name="next-steps"></a>Pasos siguientes

En este tutorial, ha exportado los datos de Azure App Configuration que se van a usar en una implementación de Kubernetes con Helm. Para más información sobre App Configuration, continúe con los ejemplos de la CLI de Azure.

> [!div class="nextstepaction"]
> [CLI de Azure](https://docs.microsoft.com/cli/azure/appconfig?view=azure-cli-latest)
