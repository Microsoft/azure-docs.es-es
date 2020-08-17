---
title: Implementación con Terraform desde Azure Cloud Shell | Microsoft Docs
description: Implementación con Terraform desde Azure Cloud Shell
services: Azure
documentationcenter: ''
author: tomarchermsft
manager: routlaw
tags: azure-cloud-shell
ms.service: azure
ms.workload: infrastructure-services
ms.tgt_pltfrm: vm-linux
ms.devlang: na
ms.topic: article
ms.date: 11/15/2017
ms.author: tarcher
ms.custom: devx-track-terraform
ms.openlocfilehash: e13e6d9ac2f4600e41f221efd23997f712dffc54
ms.sourcegitcommit: 3d79f737ff34708b48dd2ae45100e2516af9ed78
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/23/2020
ms.locfileid: "87032092"
---
# <a name="deploy-with-terraform-from-bash-in-azure-cloud-shell"></a>Implementación con Terraform de Bash en Azure Cloud Shell
Este artículo le guía en la creación de un grupo de recursos con el [proveedor de AzureRM de Terraform](https://www.terraform.io/docs/providers/azurerm/index.html).

[Terraform de Hashicorp](https://www.terraform.io/) es una herramienta de código abierto que codifica las API en archivos de configuración declarativos que se pueden compartir entre los miembros del equipo para editarlos, revisarlos y controlar sus versiones. El proveedor de Microsoft AzureRM se usa para interactuar con los recursos admitidos por Azure Resource Manager mediante las API de AzureRM.

## <a name="automatic-authentication"></a>Autenticación automática
Terraform se instala en Bash en Cloud Shell de forma predeterminada. Además, Cloud Shell autentica automáticamente la suscripción predeterminada de la CLI de Azure para implementar los recursos mediante los módulos de Azure de Terraform.

Terraform usa la suscripción predeterminada de la CLI de Azure que se especifica. Para actualizar las suscripciones predeterminadas, ejecute:

```azurecli-interactive
az account set --subscription mySubscriptionName
```

## <a name="walkthrough"></a>Tutorial
### <a name="launch-bash-in-cloud-shell"></a>Inicio de Bash en Cloud Shell
1. Inicie Cloud Shell desde su ubicación preferida.
2. Compruebe que su suscripción preferida está establecida.

```azurecli-interactive
az account show
```

### <a name="create-a-terraform-template"></a>Creación de una plantilla de Terraform
Cree una nueva plantilla de Terraform llamada main.tf con el editor de texto que prefiera.

```
vim main.tf
```

Copie y pegue el siguiente código en Cloud Shell.

```
resource "azurerm_resource_group" "myterraformgroup" {
    name = "myRgName"
    location = "West US"
}
```

Guarde el archivo y salga de su editor de texto.

### <a name="terraform-init"></a>Terraform init
Para comenzar, ejecute `terraform init`.

```
justin@Azure:~$ terraform init

Initializing provider plugins...

The following providers do not have any version constraints in configuration,
so the latest version was installed.

To prevent automatic upgrades to new major versions that may contain breaking
changes, it is recommended to add version = "..." constraints to the
corresponding provider blocks in configuration, with the constraint strings
suggested below.

* provider.azurerm: version = "~> 0.2"

Terraform has been successfully initialized!

You may now begin working with Terraform. Try running "terraform plan" to see
any changes that are required for your infrastructure. All Terraform commands
should now work.

If you ever set or change modules or backend configuration for Terraform,
rerun this command to reinitialize your working directory. If you forget, other
commands will detect it and remind you to do so if necessary.
```

El [comando terraform init](https://www.terraform.io/docs/commands/init.html) se usa para inicializar un directorio de trabajo que contiene los archivos de configuración de Terraform. El comando `terraform init` es el primer comando que se debe ejecutar después de escribir una nueva configuración de Terraform o de clonar una ya existente desde el control de versiones. Es seguro ejecutar este comando varias veces.

### <a name="terraform-plan"></a>Plan de Terraform
Obtenga una vista previa de los recursos que se crearán mediante la plantilla de Terraform con `terraform plan`.

```
justin@Azure:~$ terraform plan
Refreshing Terraform state in-memory prior to plan...
The refreshed state will be used to calculate this plan, but will not be
persisted to local or remote state storage.


------------------------------------------------------------------------

An execution plan has been generated and is shown below.
Resource actions are indicated with the following symbols:
  + create

Terraform will perform the following actions:

  + azurerm_resource_group.demo
      id:       <computed>
      location: "westus"
      name:     "myRGName"
      tags.%:   <computed>


Plan: 1 to add, 0 to change, 0 to destroy.

------------------------------------------------------------------------

Note: You didn't specify an "-out" parameter to save this plan, so Terraform
can't guarantee that exactly these actions will be performed if
"terraform apply" is subsequently run.
```

El [comando terraform plan](https://www.terraform.io/docs/commands/plan.html) se usa para crear un plan de ejecución. Terraform realiza una actualización, a menos que se deshabilite explícitamente y, a continuación, determina qué acciones son necesarias para alcanzar el estado deseado especificado en los archivos de configuración. El plan se puede guardar mediante -out y luego proporcionarse a terraform apply para garantizar que solo se ejecutan las acciones ya planeadas.

### <a name="terraform-apply"></a>Terraform apply
Aprovisione los recursos de Azure con `terraform apply`.

```
justin@Azure:~$ terraform apply
azurerm_resource_group.demo: Creating...
  location: "" => "westus"
  name:     "" => "myRGName"
  tags.%:   "" => "<computed>"
azurerm_resource_group.demo: Creation complete after 0s (ID: /subscriptions/mySubIDmysub/resourceGroups/myRGName)

Apply complete! Resources: 1 added, 0 changed, 0 destroyed.
```

El [comando terraform apply](https://www.terraform.io/docs/commands/apply.html) se usa para aplicar los cambios necesarios y así alcanzar el estado deseado de la configuración.

### <a name="verify-deployment-with-azure-cli"></a>Comprobación de la implementación con la CLI de Azure
Ejecute `az group show -n myRgName` para comprobar que el recurso se ha aprovisionado correctamente.

```azurecli-interactive
az group show -n myRgName
```

### <a name="clean-up-with-terraform-destroy"></a>Limpieza con terraform destroy
Limpie el grupo de recursos creado con el [comando Terraform destroy](https://www.terraform.io/docs/commands/destroy.html) para limpiar la infraestructura creada por Terraform.

```
justin@Azure:~$ terraform destroy
azurerm_resource_group.demo: Refreshing state... (ID: /subscriptions/mySubID/resourceGroups/myRGName)

An execution plan has been generated and is shown below.
Resource actions are indicated with the following symbols:
  - destroy

Terraform will perform the following actions:

  - azurerm_resource_group.demo


Plan: 0 to add, 0 to change, 1 to destroy.

Do you really want to destroy?
  Terraform will destroy all your managed infrastructure, as shown above.
  There is no undo. Only 'yes' will be accepted to confirm.

  Enter a value: yes

azurerm_resource_group.demo: Destroying... (ID: /subscriptions/mySubID/resourceGroups/myRGName)
azurerm_resource_group.demo: Still destroying... (ID: /subscriptions/mySubID/resourceGroups/myRGName, 10s elapsed)
azurerm_resource_group.demo: Still destroying... (ID: /subscriptions/mySubID/resourceGroups/myRGName, 20s elapsed)
azurerm_resource_group.demo: Still destroying... (ID: /subscriptions/mySubID/resourceGroups/myRGName, 30s elapsed)
azurerm_resource_group.demo: Still destroying... (ID: /subscriptions/mySubID/resourceGroups/myRGName, 40s elapsed)
azurerm_resource_group.demo: Destruction complete after 45s

Destroy complete! Resources: 1 destroyed.
```

Ha creado correctamente un recurso de Azure a través de Terraform. Consulte los pasos siguientes para seguir aprendiendo acerca de Cloud Shell.

## <a name="next-steps"></a>Pasos siguientes
[Más información sobre el proveedor de Terraform de Azure](https://www.terraform.io/docs/providers/azurerm/#)<br>
[Guía de inicio rápido de Bash en Cloud Shell](quickstart.md)
