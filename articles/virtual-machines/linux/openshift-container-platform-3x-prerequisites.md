---
title: Requisitos previos de OpenShift Container Platform 3.11 en Azure
description: Requisitos previos para implementar OpenShift Container Platform 3.11 en Azure.
author: haroldwongms
manager: mdotson
ms.service: virtual-machines-linux
ms.subservice: workloads
ms.topic: how-to
ms.workload: infrastructure
ms.date: 10/23/2019
ms.author: haroldw
ms.openlocfilehash: 68bd748e890659e4b79d76e4ccab038f251a937a
ms.sourcegitcommit: f353fe5acd9698aa31631f38dd32790d889b4dbb
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/29/2020
ms.locfileid: "87368190"
---
# <a name="common-prerequisites-for-deploying-openshift-container-platform-311-in-azure"></a>Requisitos previos para implementar OpenShift Container Platform 3.11 en Azure

En este artículo se describen los requisitos previos comunes para implementar OpenShift Container Platform u OKD en Azure.

La instalación de OpenShift usa cuadernos de estrategias de Ansible. Ansible usa Secure Shell (SSH) para conectarse a todos los hosts del clúster y completar los pasos de instalación.

Cuando Ansible realiza la conexión SSH a los hosts remotos, no puede escribir una contraseña. Por este motivo, la clave privada no puede tener una contraseña (frase de contraseña) asociada o se producirá un error en la implementación.

Dado que todas las máquinas virtuales (VM) se implementan a través de plantillas de Azure Resource Manager, se usa la misma clave pública para acceder a todas ellas. La clave privada correspondiente debe estar en la máquina virtual que ejecuta todos los cuadernos de estrategias. Para llevar a cabo esta acción de forma segura, se usa Azure Key Vault para pasar la clave privada a la máquina virtual.

Si es necesario el almacenamiento persistente de contenedores, también se necesitan volúmenes persistentes. OpenShift es compatible con discos duros virtuales (VHD) de Azure para volúmenes persistentes, pero Azure debe configurarse primero como proveedor de nube.

En este modelo, OpenShift:

- Crea un objeto VHD en una cuenta de Azure Storage o un disco administrado.
- Monta el VHD en una máquina virtual y da formato al volumen.
- Monta el volumen en el pod.

Para que esta configuración funcione, OpenShift necesita permisos para realizar estas tareas en Azure. Para esta finalidad se usa una entidad de servicio. La entidad de servicio es una cuenta de seguridad de Azure Active Directory con permisos para los recursos.

La entidad de servicio debe tener acceso a las cuentas de almacenamiento y a las máquinas virtuales que componen el clúster. Si todos los recursos de clúster de OpenShift se implementan en un único grupo de recursos, pueden concederse permisos a la entidad de servicio para ese grupo de recursos.

En esta guía se describe cómo crear los artefactos asociados con los requisitos previos.

> [!div class="checklist"]
> * Crear un almacén de claves para administrar las claves SSH para el clúster de OpenShift.
> * Cree una entidad de servicio para que la use el proveedor en la nube de Azure.

Si no tiene una suscripción a Azure, cree una [cuenta gratuita](https://azure.microsoft.com/free/?WT.mc_id=A261C142F) antes de empezar.

## <a name="sign-in-to-azure"></a>Inicio de sesión en Azure 
Inicie sesión en la suscripción de Azure con el comando [az login](/cli/azure/reference-index) y siga las instrucciones de la pantalla, o bien haga clic en **Pruébelo** para usar Cloud Shell.

```azurecli
az login
```

## <a name="create-a-resource-group"></a>Crear un grupo de recursos

Para crear un grupo de recursos, use el comando [az group create](/cli/azure/group). Un grupo de recursos de Azure es un contenedor lógico en el que se implementan y se administran los recursos de Azure. Debe usar un grupo de recursos dedicado para hospedar el almacén de claves. Este grupo es independiente del grupo de recursos en el que se implementan los recursos del clúster de OpenShift.

En el ejemplo siguiente se crea un grupo de recursos denominado *keyvaultrg* en la ubicación *eastus*:

```azurecli
az group create --name keyvaultrg --location eastus
```

## <a name="create-a-key-vault"></a>Creación de un Almacén de claves
Cree un almacén de claves para almacenar las claves SSH para el clúster con el comando [az keyvault create](/cli/azure/keyvault). El nombre del almacén de claves debe ser único globalmente y debe estar habilitado para la implementación de plantillas o se producirá un error de implementación "KeyVaultParameterReferenceSecretRetrieveFailed".

En el ejemplo siguiente se crea un almacén de claves denominado *keyvault* en el grupo de recursos *keyvaultrg*:

```azurecli 
az keyvault create --resource-group keyvaultrg --name keyvault \
       --enabled-for-template-deployment true \
       --location eastus
```

## <a name="create-an-ssh-key"></a>Creación de una clave SSH 
Se necesita una clave SSH para proteger el acceso al clúster de OpenShift. Cree un par de claves SSH con el comando `ssh-keygen` (en Linux o macOS):
 
```bash
ssh-keygen -f ~/.ssh/openshift_rsa -t rsa -N ''
```

> [!NOTE]
> El par de claves SSH no puede tener una contraseña ni frase de contraseña.

Para obtener más información sobre las claves SSH en Windows, consulte el artículo sobre la [creación de claves SSH en Windows](./ssh-from-windows.md). Asegúrese de exportar la clave privada en formato OpenSSH.

## <a name="store-the-ssh-private-key-in-azure-key-vault"></a>Almacenamiento de la clave privada SSH en Azure Key Vault
La implementación de OpenShift utiliza la clave SSH que creó para proteger el acceso al maestro de OpenShift. Para permitir que la implementación recupere de forma segura la clave SSH, almacénela en Key Vault con el comando siguiente:

```azurecli
az keyvault secret set --vault-name keyvault --name keysecret --file ~/.ssh/openshift_rsa
```

## <a name="create-a-service-principal"></a>Creación de una entidad de servicio 
OpenShift se comunica con Azure mediante un nombre de usuario y una contraseña, o bien a través de una entidad de servicio. Las entidades de servicio de Azure son identidades de seguridad que pueden usarse con aplicaciones, servicios y herramientas de automatización como OpenShift. El usuario controla los permisos y los define con respecto a cuáles son las operaciones que la entidad de servicio puede realizar en Azure. Lo mejor es definir el ámbito de los permisos de la entidad de servicio a grupos de recursos específicos en lugar de la suscripción completa.

Cree una entidad de servicio con [az ad sp create-for-rbac](/cli/azure/ad/sp) y genere las credenciales necesarias para OpenShift.

En el ejemplo siguiente se crea una entidad de servicio y se le asignan permisos de colaborador para un grupo de recursos llamado *openshiftrg*.

En primer lugar, cree el grupo de recursos llamado *openshiftrg*:

```azurecli
az group create -l eastus -n openshiftrg
```

Cree una entidad de servicio:

```azurecli
az group show --name openshiftrg --query id
```

Guarde la salida del comando y úsela en lugar de $scope en el siguiente comando

```azurecli
az ad sp create-for-rbac --name openshiftsp \
      --role Contributor --scopes $scope \
```

Tome nota de la propiedad appId y la contraseña que devuelve el comando:

```json
{
  "appId": "11111111-abcd-1234-efgh-111111111111",
  "displayName": "openshiftsp",
  "name": "http://openshiftsp",
  "password": {Strong Password},
  "tenant": "XXXXXXXX-XXXX-XXXX-XXXX-XXXXXXXXXXXX"
}
```

 > [!WARNING] 
 > Asegúrese de anotar la contraseña segura, ya que no se podrá volver a recuperar.

Para más información sobre las entidades de servicio, consulte [Creación de una entidad de servicio de Azure con la CLI de Azure](/cli/azure/create-an-azure-service-principal-azure-cli?view=azure-cli-latest).

## <a name="prerequisites-applicable-only-to-resource-manager-template"></a>Requisitos previos aplicables únicamente a la plantilla de Resource Manager

Los secretos deben crearse para la clave privada SSH (**sshPrivateKey**), el secreto del cliente de Azure AD (**aadClientSecret**), la contraseña del administrador de OpenShift (**openshiftPassword**) y la contraseña o clave de activación de Red Hat Subscription Manager (**rhsmPasswordOrActivationKey**).  Además, si se usan certificados TLS/SSL personalizados, deberán crearse seis secretos adicionales: **routingcafile**, **routingcertfile**, **routingkeyfile**, **mastercafile**, **mastercertfile** y **masterkeyfile**.  Estos parámetros se explicarán con más detalle.

La plantilla hace referencia a nombres de secretos específicos por lo que **deben** usarse los nombres en negrita mencionados anteriormente (con distinción de mayúsculas y minúsculas).

### <a name="custom-certificates"></a>Certificados personalizados

De forma predeterminada, la plantilla implementará un clúster de OpenShift mediante certificados autofirmados para la consola web de OpenShift y el dominio de enrutamiento. Si quiere usar certificados TLS/SSL personalizados, establezca "routingCertType" en "custom" y "masterCertType" en "custom".  Necesitará los archivos de CA, certificado y clave en formato .pem para los certificados.  Es posible utilizar certificados personalizados para uno de los casos, pero no para los otros.

Deberá almacenar estos archivos en secretos de Key Vault.  Use el mismo Key Vault que ha utilizado para la clave privada.  En lugar de requerir seis entradas adicionales para los nombres de secreto, la plantilla está codificada de forma rígida para utilizar nombres de secreto específicos para cada uno de los archivos de certificado TLS/SSL.  Almacene los datos de certificado con la información de la tabla siguiente.

| Nombre del secreto      | Archivo de certificado   |
|------------------|--------------------|
| mastercafile     | Archivo de CA maestro     |
| mastercertfile   | Archivo CERT principal   |
| masterkeyfile    | Archivo de clave maestro    |
| routingcafile    | Archivo de CA de enrutamiento    |
| routingcertfile  | Archivo CERT de enrutamiento  |
| routingkeyfile   | Archivo de clave de enrutamiento   |

Cree los secretos con la CLI de Azure. A continuación se muestra un ejemplo.

```azurecli
az keyvault secret set --vault-name KeyVaultName -n mastercafile --file ~/certificates/masterca.pem
```

## <a name="next-steps"></a>Pasos siguientes

En este artículo se trataron los temas siguientes:
> [!div class="checklist"]
> * Crear un almacén de claves para administrar las claves SSH para el clúster de OpenShift.
> * Crear una entidad de servicio para que la use el Proveedor de soluciones en la nube de Azure.

A continuación, implemente un clúster de OpenShift:

- [Deploy OpenShift Container Platform](./openshift-container-platform-3x.md) (Implementar OpenShift Container Platform)
- [Implementación con la oferta de Marketplace de OpenShift Container Platform autoadministrada](./openshift-container-platform-3x-marketplace-self-managed.md)
