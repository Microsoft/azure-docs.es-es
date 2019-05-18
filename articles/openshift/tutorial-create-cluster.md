---
title: 'Tutorial: Creación de un clúster de Red Hat OpenShift en Azure | Microsoft Docs'
description: Aprenda a crear un clúster de Microsoft Azure Red Hat OpenShift con la CLI de Azure
services: container-service
author: TylerMSFT
ms.author: twhitney
manager: jeconnoc
ms.topic: tutorial
ms.service: openshift
ms.date: 05/08/2019
ms.openlocfilehash: baada8a5238725456ca4a2ec7e8257c229066115
ms.sourcegitcommit: e6d53649bfb37d01335b6bcfb9de88ac50af23bd
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/09/2019
ms.locfileid: "65466176"
---
# <a name="tutorial-create-an-azure-red-hat-openshift-cluster"></a>Tutorial: Creación de un clúster de Red Hat OpenShift en Azure

Este tutorial es la primera parte de una serie. Aprenderá a crear un clúster de Microsoft Azure Red Hat OpenShift con la CLI de Azure, escalarlo y eliminarlo para limpiar los recursos.

En la primera parte de la serie, se aprende a:

> [!div class="checklist"]
> * Creación de un clúster de Red Hat OpenShift en Azure

En esta serie de tutoriales, se aprende a:
> [!div class="checklist"]
> * Creación de un clúster de Red Hat OpenShift en Azure
> * [Escalado de un clúster de Red Hat OpenShift en Azure](tutorial-scale-cluster.md)
> * [Eliminación de un clúster de Red Hat OpenShift en Azure](tutorial-delete-cluster.md)

## <a name="prerequisites"></a>Requisitos previos

Antes de empezar este tutorial:

Asegúrese de haber [configurado el entorno de desarrollo](howto-setup-environment.md), lo que incluye lo siguiente:
- Instale la CLI más reciente (versión 2.0.64 o superior)
- Creación de un inquilino
- Creación de un objeto de aplicación de Azure
- Creación de un usuario de Active Directory usado para iniciar sesión en aplicaciones que se ejecutan en el clúster

## <a name="step-1-sign-in-to-azure"></a>Paso 1: Inicio de sesión en Azure

Si ejecuta la CLI de Azure de manera local, abra un shell de comandos de Bash y ejecute `az login` para iniciar sesión en Azure.

```bash
az login
```

 Si tiene acceso a varias suscripciones, ejecute `az account set -s {subscription ID}` y reemplace `{subscription ID}` por la suscripción que quiere usar.

## <a name="step-2-create-an-azure-red-hat-openshift-cluster"></a>Paso 2: Creación de un clúster de Red Hat OpenShift en Azure

En la ventana de comandos de Bash, establezca las siguientes variables:

> [!IMPORTANT]
> El nombre del clúster debe estar en minúscula o se producirá un error en la creación del clúster.

```bash
CLUSTER_NAME=<cluster name in lowercase>
```

 Use el mismo nombre para el clúster que eligió en el paso 6 de [Creación de un registro de aplicaciones](howto-aad-app-configuration.md#create-a-new-app-registration).

```bash
LOCATION=<location>
```

Elija una ubicación para crear el clúster. Para obtener una lista de regiones de Azure que admitan OpenShift en Azure, consulte [Regiones admitidas](supported-resources.md#azure-regions). Por ejemplo: `LOCATION=eastus`.

Establezca `FQDN` en el nombre completo del clúster. Este nombre se compone del nombre del clúster, la ubicación y `.cloudapp.azure.com` anexado al final. Es igual que la dirección URL de inicio de sesión que creó en el paso 6 de [Creación de un registro de aplicaciones](howto-aad-app-configuration.md#create-a-new-app-registration). Por ejemplo:   

```bash
FQDN=$CLUSTER_NAME.$LOCATION.cloudapp.azure.com
```

Establezca `APPID` en el valor que guardó en el paso 9 de [Creación de un registro de aplicaciones](howto-aad-app-configuration.md#create-a-new-app-registration).  

```bash
APPID=<app ID value>
```

Establezca `SECRET` en el valor que guardó en el paso 6 de [Creación de un secreto de cliente](howto-aad-app-configuration.md#create-a-client-secret).  

```bash
SECRET=<secret value>
```

Establezca `TENANT` en el valor de identificador de inquilino que guardó en el paso 7 de [Creación de un nuevo inquilino](howto-create-tenant.md#create-a-new-azure-ad-tenant).  

```bash
TENANT=<tenant ID>
```

Cree el grupo de recursos para el clúster. Ejecute el siguiente comando desde el shell de Bash que usó para definir las variables anteriores:

```bash
az group create --name $CLUSTER_NAME --location $LOCATION
```

### <a name="optional-connect-the-clusters-virtual-network-to-an-existing-virtual-network"></a>Opcional: Conexión de la red virtual del clúster a una red virtual existente

Si no necesita conectar la red virtual (VNET) del clúster que crea a una red virtual existente mediante el emparejamiento, omita este paso.

En primer lugar, obtenga el identificador de la red virtual existente. El identificador tendrá el siguiente formato: `/subscriptions/{subscription id}/resourceGroups/{resource group of VNET}/providers/Microsoft.Network/virtualNetworks/{VNET name}`.

Si no conoce el nombre de red o el grupo de recursos al que pertenece la red virtual existente, vaya a la [hoja Redes virtuales](https://ms.portal.azure.com/#blade/HubsExtension/BrowseResourceBlade/resourceType/Microsoft.Network%2FvirtualNetworks) y haga clic en la red virtual. Se mostrará la página Red virtual, con el nombre de la red y el grupo de recursos al que pertenece.

Defina una variable VNET_ID con el siguiente comando de la CLI en un shell de BASH:

```bash
VNET_ID=$(az network vnet show -n {VNET name} -g {VNET resource group} --query id -o tsv)
```

Por ejemplo: `VNET_ID=$(az network vnet show -n MyVirtualNetwork -g MyResourceGroup --query id -o tsv`

### <a name="create-the-cluster"></a>Creación de clústeres

Ya está listo para crear un clúster.

 Si no va a conectar la red virtual del clúster a una red virtual existente, omita el parámetro `--vnet-peer-id $VNET_ID` final en el ejemplo siguiente.

```bash
az openshift create --resource-group $CLUSTER_NAME --name $CLUSTER_NAME -l $LOCATION --fqdn $FQDN --aad-client-app-id $APPID --aad-client-app-secret $SECRET --aad-tenant-id $TENANT --vnet-peer-id $VNET_ID
```

Después de unos minutos, `az openshift create` se completará correctamente y devolverá una respuesta JSON con los detalles del clúster.

> [!NOTE]
> Si obtiene un error indicando que el nombre de host no está disponible, es posible que el nombre del clúster no sea único. Pruebe a eliminar el registro de aplicación original y repita los pasos descritos en [Creación de un registro de aplicaciones] (howto-aad-app-configuration.md#create-a-new-app-registration) (omitiendo el paso final de crear un nuevo usuario, dado que ya ha creado uno) con un nombre de clúster diferente.

## <a name="step-3-sign-in-to-the-openshift-console"></a>Paso 3: Inicio de sesión en la consola de OpenShift

Ahora está listo para iniciar sesión en la consola de OpenShift para el nuevo clúster. La [consola web de OpenShift](https://docs.openshift.com/aro/architecture/infrastructure_components/web_console.html) le permite visualizar, explorar y administrar el contenido de los proyectos de OpenShift.

Ahora iniciará sesión como el [nuevo usuario de Azure AD](howto-aad-app-configuration.md#create-a-new-active-directory-user) que creó para las pruebas. Para ello, necesitará una instancia nueva del explorador que no haya almacenado en caché la identidad que normalmente usa para iniciar sesión en Azure Portal.

1. Abra una ventana de *incógnito* (Chrome) o *InPrivate* (Microsoft Edge).
2. Vaya a la dirección URL de inicio de sesión que creó en el paso 6 de [Creación de un registro de aplicaciones](howto-aad-app-configuration.md#create-a-new-app-registration). Por ejemplo: https://constoso.eastus.cloudapp.azure.com

> [!NOTE]
> La consola de OpenShift utiliza un certificado autofirmado.
> Cuando se le solicite en el explorador, omita la advertencia y acepte el certificado que "no es de confianza".

Inicie sesión con el usuario y la contraseña que creó en [Crear un nuevo usuario de Active Directory](howto-aad-app-configuration.md#create-a-new-active-directory-user). Cuando aparezca el cuadro de diálogo **Permisos solicitados**, seleccione **Consentimiento en nombre de la organización** y, a continuación, **Aceptar**.

Al hacerlo, iniciará sesión en la consola del clúster.

![Captura de pantalla de la consola del clúster de OpenShift](./media/aro-console.png)

 Obtenga más información acerca del [uso de la consola de OpenShift](https://docs.openshift.com/aro/getting_started/developers_console.html) para crear imágenes en la documentación de [Red Hat OpenShift](https://docs.openshift.com/aro/welcome/index.html).

## <a name="step-4-install-the-openshift-cli"></a>Paso 4: Instalación de la CLI de OpenShift

La [CLI de OpenShift](https://docs.openshift.com/aro/cli_reference/get_started_cli.html) (u *OC Tools*) proporciona comandos para administrar las aplicaciones y utilidades de nivel inferior para interactuar con los distintos componentes del clúster de OpenShift.

En la consola de OpenShift, haga clic en el signo de interrogación de la esquina superior derecha junto a su nombre de inicio de sesión y seleccione **Command Line Tools** (Herramientas de línea de comandos).  Siga el vínculo **Latest Release** (Versión más reciente) para descargar e instalar la CLI de OC compatible con Linux, macOS o Windows.

> [!NOTE]
> Si no ve el icono de signo de interrogación en la esquina superior derecha, seleccione *Service Catalog* (Catálogo de servicios) o *Consola de aplicación* (Application Console) en la lista desplegable de la parte superior izquierda.
>
> Como alternativa, puede [descargar la CLI de OC](https://www.okd.io/download.html) directamente.

La página **Command Line Tools** (Herramientas de línea de comandos) proporciona un comando con el formato `oc login https://<your cluster name>.<azure region>.cloudapp.azure.com --token=<token value>`.  Haga clic en el botón *Copy to clipboard* (Copiar al Portapapeles) para copiar este comando.  En una ventana de terminal, [establezca la ruta de acceso](https://docs.okd.io/latest/cli_reference/get_started_cli.html#installing-the-cli) para incluir la instalación local de las herramientas de OC. A continuación, inicie sesión en el clúster mediante el comando de la CLI de OC que copió.

Si no se pudo obtener el valor del token siguiendo los pasos anteriores, obténgalo de: `https://<your cluster name>.<azure region>.cloudapp.azure.com/oauth/token/request`.

## <a name="next-steps"></a>Pasos siguientes

En esta parte del tutorial, ha aprendido a:

> [!div class="checklist"]
> * Creación de un clúster de Red Hat OpenShift en Azure

Avance hasta el siguiente tutorial:
> [!div class="nextstepaction"]
> [Escalado de un clúster de Red Hat OpenShift en Azure](tutorial-scale-cluster.md)