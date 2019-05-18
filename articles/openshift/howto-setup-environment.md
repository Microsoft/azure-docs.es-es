---
title: Configurar el entorno de desarrollo de Azure Red Hat OpenShift | Microsoft Docs
description: Estos son los requisitos previos para trabajar con Microsoft Azure Red Hat OpenShift.
services: openshift
keywords: configurar el programa de instalación de red hat openshift
author: TylerMSFT
ms.author: twhitney
ms.date: 05/10/2019
ms.topic: conceptual
ms.service: openshift
manager: jeconnoc
ms.openlocfilehash: 6e859f57f9b5f24ea2f0172f5aa35a60d9769f19
ms.sourcegitcommit: f013c433b18de2788bf09b98926c7136b15d36f1
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 05/13/2019
ms.locfileid: "65551692"
---
# <a name="set-up-your-azure-red-hat-openshift-dev-environment"></a>Configurar el entorno de desarrollo de Azure Red Hat OpenShift

Para compilar y ejecutar aplicaciones de Microsoft Azure Red Hat OpenShift, deberá:

* Comprar instancias reservadas de máquina virtual Azure.
* Instalar versión 2.0.65 (o superior) de la CLI de Azure (o use Azure Cloud Shell).
* Registrarse para el `openshiftmanagedcluster` característica y los proveedores de recursos asociados.
* Crear a un inquilino de Azure Active Directory (Azure AD).
* Cree un objeto de aplicación de Azure AD.
* Creación de un usuario de Azure AD.

Las instrucciones siguientes le guiará a través de todos estos requisitos previos.

## <a name="purchase-azure-virtual-machine-reserved-instances"></a>Comprar instancias reservadas de máquina virtual Azure

Para poder usar Azure Red Hat OpenShift, deberá comprar instancias reservadas de máquina virtual Azure.

Si es cliente de Azure, aquí cómo [instancias reservadas de máquina virtual de Azure de la compra](https://aka.ms/openshift/buy). Una reserva reduce el gasto previamente pagando por servicios de Azure totalmente administrados. Consulte [ *¿cuáles son las reservas de Azure* ](https://docs.microsoft.com/azure/billing/billing-save-compute-costs-reservations) para obtener más información sobre las reservas y cómo pueden ahorrar dinero.

Si no es un cliente de Azure, [póngase en contacto con ventas](https://aka.ms/openshift/contact-sales) y rellene el formulario de venta en la parte inferior de la página para iniciar el proceso.

## <a name="install-the-azure-cli"></a>Instalación de la CLI de Azure

Azure Red Hat OpenShift requiere la versión 2.0.65 o posterior de la CLI de Azure. Si ya ha instalado la CLI de Azure, puede comprobar qué versión tiene, ejecute:

```bash
az --version
```

La primera línea de salida tendrá la versión de la CLI, por ejemplo `azure-cli (2.0.65)`.

Aquí encontrará instrucciones para [instalar la CLI de Azure](https://docs.microsoft.com/cli/azure/install-azure-cli?view=azure-cli-latest) si necesita una nueva instalación o una actualización.

Como alternativa, puede usar el [Azure Cloud Shell](https://docs.microsoft.com/azure/cloud-shell/overview). Cuando se usa Azure Cloud Shell, no olvide seleccionar la **Bash** entorno si tiene previsto seguir el [crear y administrar un clúster de Azure Red Hat OpenShift](tutorial-create-cluster.md) serie de tutoriales.

## <a name="register-providers-and-features"></a>Registrar proveedores y características

El `Microsoft.ContainerService openshiftmanagedcluster` característica, `Microsoft.Solutions`, y `Microsoft.Network` proveedores deben estar registrados en su suscripción manualmente antes de implementar su primer clúster de Azure Red Hat OpenShift.

Para registrar manualmente estos proveedores y características, utilice las instrucciones siguientes desde un shell de Bash si ha instalado la CLI, o desde la sesión de Azure Cloud Shell (Bash) en el portal de Azure:

1. Si tiene varias suscripciones de Azure, especifique el identificador de suscripción correspondiente:

    ```bash
    az account set --subscription <SUBSCRIPTION ID>
    ```

2. Registre la característica de openshiftmanagedcluster Microsoft.ContainerService:

    ```bash
    az feature register --namespace Microsoft.ContainerService -n openshiftmanagedcluster
    ```

3. Registre el proveedor Microsoft.Solutions:

    ```bash
    az provider register -n Microsoft.Solutions --wait
    ```

4. Registre el proveedor de Microsoft.Network:

    ```bash
    az provider register -n Microsoft.Network --wait
    ```

5. Registre el proveedor Microsoft.KeyVault:

    ```bash
    az provider register -n Microsoft.KeyVault --wait
    ```

6. Actualizar el registro del proveedor de recursos Microsoft.ContainerService:

    ```bash
    az provider register -n Microsoft.ContainerService --wait
    ```

## <a name="create-an-azure-active-directory-azure-ad-tenant"></a>Crear a un inquilino de Azure Active Directory (Azure AD)

El servicio de Azure Red Hat OpenShift requiere a un inquilino de Azure Active Directory (Azure AD) asociado que representa su organización y su relación a Microsoft. Inquilino de Azure AD le permite registrar, compilar y administrar aplicaciones, así como usar otros servicios de Azure.

Si no tiene Azure AD para usarla como el inquilino de su clúster de Azure Red Hat OpenShift, o que desea crear un inquilino de prueba, siga las instrucciones de [crear un inquilino de Azure AD para el clúster de Azure Red Hat OpenShift](howto-create-tenant.md) antes continuar con esta guía.

## <a name="create-an-azure-ad-user-security-group-and-application-object"></a>Crear un usuario, grupo de seguridad y aplicación de Azure AD objeto

Azure Red Hat OpenShift necesita permisos para realizar tareas en el clúster, como la configuración de almacenamiento. Estos permisos se representan mediante un [serviceprincipal](https://docs.microsoft.com/azure/active-directory/develop/app-objects-and-service-principals#service-principal-object). También conviene crear un nuevo usuario de Active Directory para realizar pruebas de aplicaciones que se ejecutan en el clúster de Azure Red Hat OpenShift.

Siga las instrucciones de [crear un objeto de aplicación de Azure AD y un usuario](howto-aad-app-configuration.md) para crear una entidad de servicio, generar una URL de devolución de llamada de secreto y autenticación de cliente para la aplicación y crear un nuevo grupo de seguridad de Azure AD y el usuario para tener acceso a la clúster.

## <a name="next-steps"></a>Pasos siguientes

Ya está listo para usar Azure Red Hat OpenShift.

Pruebe el tutorial:
> [!div class="nextstepaction"]
> [Creación de un clúster de Red Hat OpenShift en Azure](tutorial-create-cluster.md)

[azure-cli-install]: https://docs.microsoft.com/cli/azure/install-azure-cli