---
title: Tareas posteriores a la implementación de OpenShift Container Platform 3.11 en Azure
description: Tareas adicionales posteriores a la implementación de un clúster de OpenShift Container Platform 3.11.
author: haroldwongms
manager: mdotson
ms.service: virtual-machines-linux
ms.subservice: workloads
ms.topic: how-to
ms.workload: infrastructure
ms.date: 10/14/2019
ms.author: haroldw
ms.custom: devx-track-ansible
ms.openlocfilehash: 8e34f73f1c403e3a7d21c6c30844f8b9073b3113
ms.sourcegitcommit: f353fe5acd9698aa31631f38dd32790d889b4dbb
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/29/2020
ms.locfileid: "87373579"
---
# <a name="post-deployment-tasks"></a>Tareas posteriores a la implementación

Después de implementar un clúster de OpenShift, puede configurar elementos adicionales. En este artículo se describe:

- Cómo configurar el inicio de sesión único con Azure Active Directory (Azure AD)
- Configuración de registros de Azure Monitor para supervisar OpenShift
- Cómo configurar las métricas y el registro
- Cómo instalar Open Service Broker para Azure (OSBA)

## <a name="configure-single-sign-on-by-using-azure-active-directory"></a>Configurar el inicio de sesión único con Azure Active Directory

Para usar Azure Active Directory para la autenticación, primero debe crear un registro de la aplicación de Azure AD. Este proceso implica dos pasos: crear el registro de la aplicación y configurar los permisos.

### <a name="create-an-app-registration"></a>Crear un registro de aplicación

Estos pasos usan la CLI de Azure para crear el registro de aplicación y la GUI (portal) para definir los permisos. Para crear el registro de aplicación, se necesitan los cinco datos siguientes:

- Nombre para mostrar: nombre del registro de aplicación (por ejemplo, OCPAzureAD)
- Página principal: URL de la consola de OpenShift (por ejemplo, `https://masterdns343khhde.westus.cloudapp.azure.com/console`)
- URI del identificador: URL de la consola de OpenShift (por ejemplo, `https://masterdns343khhde.westus.cloudapp.azure.com/console`)
- URL de respuesta: URL pública maestra y nombre de registro de la aplicación (por ejemplo, `https://masterdns343khhde.westus.cloudapp.azure.com/oauth2callback/OCPAzureAD`)
- Contraseña: proteja la contraseña (utilice una contraseña segura)

En el ejemplo siguiente, se creará un registro de aplicación con la información anterior:

```azurecli
az ad app create --display-name OCPAzureAD --homepage https://masterdns343khhde.westus.cloudapp.azure.com/console --reply-urls https://masterdns343khhde.westus.cloudapp.azure.com/oauth2callback/hwocpadint --identifier-uris https://masterdns343khhde.westus.cloudapp.azure.com/console --password {Strong Password}
```

Si el comando se ejecuta correctamente, recibirá una salida JSON similar a la siguiente:

```json
{
  "appId": "12345678-ca3c-427b-9a04-ab12345cd678",
  "appPermissions": null,
  "availableToOtherTenants": false,
  "displayName": "OCPAzureAD",
  "homepage": "https://masterdns343khhde.westus.cloudapp.azure.com/console",
  "identifierUris": [
    "https://masterdns343khhde.westus.cloudapp.azure.com/console"
  ],
  "objectId": "62cd74c9-42bb-4b9f-b2b5-b6ee88991c80",
  "objectType": "Application",
  "replyUrls": [
    "https://masterdns343khhde.westus.cloudapp.azure.com/oauth2callback/OCPAzureAD"
  ]
}
```

Tome nota de la propiedad appId que ha devuelto el comando para otro paso.

En Azure Portal:

1. Seleccione **Azure Active Directory** > **Registro de aplicaciones**.
2. Busque el registro de su aplicación (p. ej.: OCPAzureAD).
3. En los resultados, haga clic en el registro de aplicación.
4. En **Configuración**, seleccione **Permisos necesarios**.
5. En **Permisos necesarios**, seleccione **Agregar**.

   ![Registro de aplicaciones](media/openshift-post-deployment/app-registration.png)

6. Haga clic en el paso 1: seleccione API y, después, haga clic en **Windows Azure Active Directory (Microsoft.Azure.ActiveDirectory)** . Haga clic en **Seleccionar** en la parte inferior.

   ![Selección de API del registro de aplicaciones](media/openshift-post-deployment/app-registration-select-api.png)

7. En el paso 2: seleccione permisos, seleccione **Iniciar sesión y leer el perfil del usuario**, en **Permisos delegados**, y haga clic en **Seleccionar**.

   ![Acceso al registro de aplicaciones](media/openshift-post-deployment/app-registration-access.png)

8. Seleccione **Listo**.

### <a name="configure-openshift-for-azure-ad-authentication"></a>Configuración de OpenShift para la autenticación de Azure AD

Para configurar OpenShift para usar Azure AD como proveedor de autenticación, el archivo /etc/origin/master/master-config.yaml se debe editar en todos los nodos maestros.

Busque el identificador de inquilino mediante el siguiente comando de CLI:

```azurecli
az account show
```

En el archivo yaml, busque las siguientes líneas:

```yaml
oauthConfig:
  assetPublicURL: https://masterdns343khhde.westus.cloudapp.azure.com/console/
  grantConfig:
    method: auto
  identityProviders:
  - challenge: true
    login: true
    mappingMethod: claim
    name: htpasswd_auth
    provider:
      apiVersion: v1
      file: /etc/origin/master/htpasswd
      kind: HTPasswdPasswordIdentityProvider
```

Inserte las líneas siguientes inmediatamente después de las líneas anteriores:

```yaml
  - name: <App Registration Name>
    challenge: false
    login: true
    mappingMethod: claim
    provider:
      apiVersion: v1
      kind: OpenIDIdentityProvider
      clientID: <appId>
      clientSecret: <Strong Password>
      claims:
        id:
        - sub
        preferredUsername:
        - unique_name
        name:
        - name
        email:
        - email
      urls:
        authorize: https://login.microsoftonline.com/<tenant Id>/oauth2/authorize
        token: https://login.microsoftonline.com/<tenant Id>/oauth2/token
```

Asegúrese de que el texto se alinea correctamente en identityProviders. Busque el identificador de inquilino mediante el siguiente comando de CLI: ```az account show```.

Reinicie los servicios maestros de OpenShift en todos los nodos maestros:

```bash
sudo /usr/local/bin/master-restart api
sudo /usr/local/bin/master-restart controllers
```

En la consola de OpenShift, ahora verá dos opciones para la autenticación: htpasswd_auth y [App Registration].

## <a name="monitor-openshift-with-azure-monitor-logs"></a>Supervisión de OpenShift con registros de Azure Monitor

Hay tres maneras de agregar al agente de Log Analytics a OpenShift.
- Instalar el agente de Log Analytics para Linux directamente en cada nodo de OpenShift
- Habilitar la extensión de máquina virtual de Azure Monitor en cada nodo de OpenShift
- Instalar el agente de Log Analytics como daemon-set de OpenShift

Lea las [instrucciones](../../azure-monitor/insights/containers.md#configure-a-log-analytics-agent-for-red-hat-openshift) completas para más información.

## <a name="configure-metrics-and-logging"></a>Configurar las métricas y el registro

En función de la rama, las plantillas de Azure Resource Manager para OpenShift Container Platform y OKD pueden proporcionar parámetros de entrada para habilitar las métricas y el registro como parte de la instalación.

La oferta de Marketplace de OpenShift Container Platform también proporciona una opción para habilitar las métricas y el registro durante la instalación del clúster.

Si las métricas o el registro no se han habilitado durante la instalación del clúster, se pueden habilitar fácilmente con posterioridad.

### <a name="azure-cloud-provider-in-use"></a>Proveedor de soluciones en la nube de Azure en uso

Conéctese mediante SSH al nodo de tipo bastión o al primer nodo maestro (en función de la plantilla y la rama en uso) con las credenciales proporcionadas durante la implementación. Emita el comando siguiente:

```bash
ansible-playbook /usr/share/ansible/openshift-ansible/playbooks/openshift-metrics/config.yml \
-e openshift_metrics_install_metrics=True \
-e openshift_metrics_cassandra_storage_type=dynamic

ansible-playbook /usr/share/ansible/openshift-ansible/playbooks/openshift-logging/config.yml \
-e openshift_logging_install_logging=True \
-e openshift_logging_es_pvc_dynamic=true
```

### <a name="azure-cloud-provider-not-in-use"></a>Proveedor de soluciones en la nube de Azure que no está en uso

```bash
ansible-playbook /usr/share/ansible/openshift-ansible/playbooks/openshift-metrics/config.yml \
-e openshift_metrics_install_metrics=True

ansible-playbook /usr/share/ansible/openshift-ansible/playbooks/openshift-logging/config.yml \
-e openshift_logging_install_logging=True
```

## <a name="install-open-service-broker-for-azure-osba"></a>Instalación de Open Service Broker para Azure (OSBA)

Open Service Broker para Azure (OSBA) le permite aprovisionar Azure Cloud Services directamente desde OpenShift. OSBA es una implementación de Open Service Broker API para Azure. Open Service Broker API es una especificación que define un lenguaje común para proveedores en la nube que las aplicaciones nativas pueden usar para administrar servicios en la nube sin dependencia del proveedor.

Para instalar OSBA en OpenShift, siga las instrucciones que encontrará aquí: https://github.com/Azure/open-service-broker-azure#openshift-project-template. 
> [!NOTE]
> Complete solo los pasos descritos en la sección de la plantilla de proyecto de OpenShift y no toda la sección de instalación.

## <a name="next-steps"></a>Pasos siguientes

- [Introducción a OpenShift Container Platform](https://docs.openshift.com)
