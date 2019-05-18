---
title: Uso de Ansible con Azure | Microsoft Docs
description: Introducción al uso de Ansible para automatizar el aprovisionamiento en la nube, la administración de configuración y las implementaciones de aplicaciones.
keywords: ansible, azure, devops, introducción, aprovisionamiento en la nube, administración de configuración, implementación de aplicaciones, módulos de ansible, guiones de procedimientos de ansible
ms.topic: overview
ms.service: ansible
author: tomarchermsft
manager: jeconnoc
ms.author: tarcher
ms.date: 04/30/2019
ms.openlocfilehash: 4d7e9f8a0527587106985d8a6f0a95e19e640245
ms.sourcegitcommit: 2ce4f275bc45ef1fb061932634ac0cf04183f181
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/07/2019
ms.locfileid: "65230126"
---
# <a name="using-ansible-with-azure"></a>Uso de Ansible con Azure

[Ansible](https://www.ansible.com) es un producto de código abierto que automatiza el aprovisionamiento en la nube, la administración de configuración y las implementaciones de aplicaciones. Con Ansible puede aprovisionar maquinas virtuales, contenedores e infraestructuras en la nube y de red. Además, Ansible permite automatizar la implementación y la configuración de los recursos del entorno.

Este artículo proporciona una introducción básica a algunas de las ventajas del uso de Ansible con Azure.

## <a name="ansible-playbooks"></a>guiones de procedimientos de Ansible

Los [cuadernos de estrategias de Ansible](https://docs.ansible.com/ansible/latest/playbooks.html) permiten indicar a Ansible cómo configurar el entorno. Los cuadernos de estrategias se codifican con YAML para que sean legibles. La sección de tutoriales ofrece muchos ejemplos del uso de cuadernos de estrategias para instalar y configurar recursos de Azure. 

## <a name="ansible-modules"></a>Módulos de Ansible

Ansible incluye un conjunto de [módulos de Ansible](https://docs.ansible.com/ansible/latest/modules_by_category.html) que se pueden ejecutar directamente en hosts remotos o mediante [cuadernos de estrategias](https://docs.ansible.com/ansible/latest/playbooks.html). Los usuarios pueden crear sus propios módulos. Los módulos se utilizan para controlar recursos del sistema (como servicios, paquetes o archivos) o para ejecutar comandos del sistema.

Para interactuar con los servicios de Azure, Ansible incluye un conjunto de [módulos de Ansible en la nube](https://docs.ansible.com/ansible/list_of_cloud_modules.html#azure). Estos módulos le permiten crear y organizar su infraestructura de Azure. 

## <a name="migrate-existing-workload-to-azure"></a>Migración de la carga de trabajo existente a Azure

Tras usar Ansible para definir la infraestructura, puede aplicar el cuaderno de estrategias de la aplicación para permitir que Azure escale automáticamente el entorno según sea necesario. 

## <a name="automate-cloud-native-application-in-azure"></a>Automatización de aplicaciones nativas de la nube en Azure

Ansible permite automatizar aplicaciones nativas de la nube en Azure con microservicios de Azure como [Azure Functions](https://azure.microsoft.com//services/functions/) y [Kubernetes en Azure](https://azure.microsoft.com/services/container-service/kubernetes/).  

## <a name="manage-deployments-with-dynamic-inventory"></a>Administración de implementaciones mediante el inventario dinámico

A través de su característica de [inventario dinámico](https://docs.ansible.com/ansible/intro_dynamic_inventory.html), Ansible proporciona la capacidad de extraer el inventario de recursos de Azure. A continuación, puede etiquetar las implementaciones existentes de Azure y administrar esas implementaciones etiquetadas mediante Ansible.

## <a name="additional-azure-marketplace-options"></a>Opciones adicionales de Azure Marketplace

[Ansible Tower](https://azuremarketplace.microsoft.com/marketplace/apps/redhat.ansible-tower) es una imagen de Azure Marketplace creada por Red Hat. 

Ansible Tower es una interfaz de usuario basada en web y un panel para Ansible con las siguientes características:

* Permite definir el control de acceso basado en rol, la programación de trabajos y una administración gráfica del inventario. 
* Incluye una API REST y la CLI para que pueda insertar Tower en las herramientas y procesos existentes. 
* Admite la salida en tiempo real de las ejecuciones del cuaderno de estrategias. 
* Cifra las credenciales (como las claves de Azure y SSH) para que pueda delegar las tareas sin exponer las credenciales.

## <a name="ansible-module-and-version-matrix-for-azure"></a>Matriz de módulo y versión de Ansible para Azure

Ansible incluye un conjunto de módulos para su uso en el aprovisionamiento y configuración de los recursos de Azure. Estos recursos incluyen máquinas virtuales, conjuntos de escalado, servicios de redes y servicios de contenedores. En la [matriz de Ansible](./ansible-matrix.md) se enumeran los módulos de Ansible para Azure y las versiones de Ansible en las que se incluyen.

## <a name="next-steps"></a>Pasos siguientes

- [Inicio rápido: Implementación de la plantilla de solución de Ansible para Azure en CentOS](./ansible-deploy-solution-template.md)
- [Inicio rápido: Configuración de máquinas virtuales Linux en Azure con Ansible](/azure/virtual-machines/linux/ansible-install-configure?toc=%2Fazure%2Fansible%2Ftoc.json&bc=%2Fazure%2Fbread%2Ftoc.json)