---
title: 'Tutorial: Aprovisionamiento de una instancia de Azure Spring Cloud con Terraform'
description: Aprovisione una instancia de Azure Spring Cloud con Terraform.
author: MikeDodaro
ms.author: brendm
ms.service: spring-cloud
ms.topic: tutorial
ms.date: 06/26/2020
ms.custom: devx-track-java
ms.openlocfilehash: 70a9d9f8b137f02e647d1ea0edddd409ec3cdaf3
ms.sourcegitcommit: 2ffa5bae1545c660d6f3b62f31c4efa69c1e957f
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/11/2020
ms.locfileid: "88079199"
---
# <a name="tutorial-provision-an-azure-spring-cloud-instance-with-terraform"></a>Tutorial: Aprovisionamiento de una instancia de Azure Spring Cloud con Terraform

En este tutorial se crea una instancia de Azure Spring Cloud mediante Terraform. Los procedimientos le guiarán en la creación de los siguientes recursos:

> [!div class="checklist"]
> * Grupo de recursos
> * Instancia de Azure Spring Cloud
> * Azure Storage para Log Analytics

> [!NOTE]
> Para obtener soporte técnico específico de Terraform, use uno de los canales de soporte técnico de la comunidad de HashiCorp para Terraform:
>
> * Preguntas, casos de uso y patrones útiles: [sección Terraform del portal de la comunidad de HashiCorp](https://discuss.hashicorp.com/c/terraform-core).
> * Preguntas relacionadas con proveedores: [sección Proveedores de Terraform del portal de la comunidad de HashiCorp](https://discuss.hashicorp.com/c/terraform-providers).

## <a name="prerequisites"></a>Requisitos previos

- **Suscripción de Azure**: Si no tiene una suscripción a Azure, cree una [cuenta gratuita](https://azure.microsoft.com/free/?ref=microsoft.com&utm_source=microsoft.com&utm_medium=docs&utm_campaign=visualstudio) antes de empezar.

## <a name="create-configuration-file"></a>Creación de un archivo de configuración

1. Inicie sesión en [Azure Portal](https://go.microsoft.com/fwlink/p/?LinkID=525040).

1. Abra [Azure Cloud Shell](https://docs.microsoft.com/azure/app-service/quickstart-java#use-azure-cloud-shell).

1. Inicie el editor de Cloud Shell:

    ```bash
    code main.tf
    ```

1. La configuración de este paso modela los recursos de Azure, incluido un grupo de recursos de Azure y una instancia de Azure Spring Cloud.

    ```hcl
    provider "azurerm" {
        features {}
    }

    resource "azurerm_resource_group" "example" {
      name     = "username"
      location = "eastus"
    }

    resource "azurerm_spring_cloud_service" "example" {
      name                = "usernamesp"
      resource_group_name = azurerm_resource_group.example.name
      location            = azurerm_resource_group.example.location

      config_server_git_setting {
        uri          = "https://github.com/Azure-Samples/piggymetrics-config"
        label        = "master"
        search_paths = ["."]
      }
    }
    ```

1. Guarde el archivo ( **&lt;Ctrl > S**) y salga del editor ( **&lt;Ctrl > Q**).

## <a name="apply-the-configuration"></a>Aplicación de la configuración

En esta sección, se usan varios comandos de Terraform para ejecutar la configuración.

1. El comando [terraform init](https://www.terraform.io/docs/commands/init.html) inicializa el directorio de trabajo. En Cloud Shell, ejecute el comando siguiente:

    ```bash
    terraform init
    ```

1. El comando [terraform plan](https://www.terraform.io/docs/commands/plan.html) se usa para comprobar la sintaxis de configuración. El parámetro `-out` dirige los resultados a un archivo. El archivo de salida se puede usar más adelante para aplicar la configuración. En Cloud Shell, ejecute el comando siguiente:

    ```bash
    terraform plan --out plan.out
    ```

1. El comando [terraform apply](https://www.terraform.io/docs/commands/apply.html) se usa para aplicar la configuración. El comando especifica el archivo de salida del paso anterior. Crea los recursos de Azure. En Cloud Shell, ejecute el comando siguiente:

    ```bash
    terraform apply plan.out
    ```

1. Para comprobar los resultados en Azure Portal, vaya al nuevo grupo de recursos. La nueva instancia de **Azure Cosmos DB** muestra el nuevo grupo de recursos.

## <a name="update-configuration-to-config-logs-and-metrics"></a>Actualización de la configuración de los registros y métricas

En esta sección se muestra cómo actualizar la configuración para habilitar el registro y las métricas de Azure Spring Cloud con una cuenta de Azure Storage.

1. Inicie el editor de Cloud Shell:

    ```bash
    code main.tf
    ```

1. Agregue la siguiente configuración al final del archivo:

    ```hcl
    resource "azurerm_storage_account" "example" {
      name                     = "usernamest"
      resource_group_name      = azurerm_resource_group.example.name
      location                 = azurerm_resource_group.example.location
      account_tier             = "Standard"
      account_replication_type = "GRS"
    }

    resource "azurerm_monitor_diagnostic_setting" "example" {
      name               = "example"
      target_resource_id = "${azurerm_spring_cloud_service.example.id}"
      storage_account_id = "${azurerm_storage_account.example.id}"

      log {
        category = "SystemLogs"
        enabled  = true

        retention_policy {
          enabled = false
        }
      }

      metric {
        category = "AllMetrics"

        retention_policy {
          enabled = false
        }
      }
    }
    ```

1. Guarde el archivo ( **&lt;Ctrl>S**) y salga del editor ( **&lt;Ctrl>Q**).

1. Como en la sección anterior, ejecute el siguiente comando para realizar los cambios:

    ```bash
    terraform plan --out plan.out
    ```

1. Por último, ejecute el comando `terraform apply` para aplicar la configuración.

    ```bash
    terraform apply plan.out
    ```

## <a name="clean-up-resources"></a>Limpieza de recursos

Cuando ya no los necesite, elimine los recursos creados en este artículo.

Ejecute el comando [terraform destroy](https://www.terraform.io/docs/commands/destroy.html) para quitar los recursos de Azure creados en este tutorial:

```bash
terraform destroy -auto-approve
```

## <a name="next-steps"></a>Pasos siguientes

> [!div class="nextstepaction"]
> [Instalación y configuración de Terraform para aprovisionar recursos de Azure](https://docs.microsoft.com/azure/developer/terraform/getting-started-cloud-shell).