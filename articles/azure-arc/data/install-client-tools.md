---
title: Instalar las herramientas de cliente
description: Instale azdata, kubectl, la CLI de Azure, psql, Azure Data Studio (Insider) y la extensión ARC para Azure Data Studio
services: azure-arc
ms.service: azure-arc
ms.subservice: azure-arc-data
author: twright-msft
ms.author: twright
ms.reviewer: mikeray
ms.date: 09/22/2020
ms.topic: how-to
ms.openlocfilehash: 3eb41b868aba7e7350f54594fa43f01eeeabebc4
ms.sourcegitcommit: 53acd9895a4a395efa6d7cd41d7f78e392b9cfbe
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/22/2020
ms.locfileid: "90932568"
---
# <a name="install-client-tools-for-deploying-and-managing-azure-arc-enabled-data-services"></a>Instalación de las herramientas de cliente para implementar y administrar los servicios de datos habilitados para Azure Arc

> [!IMPORTANT]
> Si va a actualizar la versión a una nueva mensual, actualice también la versión a la más reciente de Azure Data Studio, la herramienta de CLI de Azure Data (azdata) y las extensiones de la CLI de Azure Data y Azure Arc para Azure Data Studio.

Este documento le guía por los pasos necesarios para instalar la CLI de Azure Data (azdata), Azure Data Studio, CLI de Azure (az) y la herramienta de la CLI de Kubernetes (kubectl) en el equipo cliente.

[!INCLUDE [azure-arc-data-preview](../../../includes/azure-arc-data-preview.md)]

## <a name="tools-for-creating-and-managing-azure-arc-enabled-data-services"></a>Herramientas para crear y administrar los servicios de datos habilitados para Azure Arc 

En la siguiente tabla se enumeran las herramientas comunes necesarias para crear y administrar los servicios de datos habilitados para Azure Arc y cómo instalar esas herramientas:

| Herramienta | Obligatorio | Descripción | Instalación |
|---|---|---|---|
| CLI de Azure Data (azdata) | Sí | Herramienta de línea de comandos para instalar y administrar un clúster de macrodatos. La CLI de Azure Data también incluye una utilidad de línea de comandos para conectarse y consultar instancias de Azure SQL y SQL Server y servidores Postgres mediante los comandos `azdata sql query` (ejecute una sola consulta desde la línea de comandos), `azdata sql shell` (un shell interactivo), `azdata postgres query` y `azdata postgres shell`. | [Instalación](/sql/azdata/install/deploy-install-azdata?toc=/azure/azure-arc/data/toc.json&bc=/azure/azure-arc/data/breadcrumb/toc.json) |
| Azure Data Studio | Sí | Herramienta de experiencia enriquecida para conectarse a una serie de bases de datos y consultarlas, como Azure SQL, SQL Server, PostrgreSQL y MySQL. Las extensiones para Azure Data Studio proporcionan una experiencia de administración para los servicios de datos habilitados para Azure Arc. | [Instalación](https://aka.ms/getazuredatastudio) |
| Extensión de la CLI de Azure Data para Azure Data Studio | Sí | Extensión para Azure Data Studio que instalará la CLI de Azure Data si aún no la tiene.| Instalación desde la galería de extensiones en Azure Data Studio.|
| Extensión de Azure Arc para Azure Data Studio | Sí | Extensión para Azure Data Studio que proporciona una experiencia de administración para los servicios de datos habilitados para Azure Arc. Existe una dependencia de la extensión de la CLI de Azure Data para Azure Data Studio. | Instalación desde la galería de extensiones en Azure Data Studio.|
| Extensión de PostgreSQL en Azure Data Studio | No | Extensión de PostgreSQL para Azure Data Studio que proporciona capacidades de administración para PostgreSQL. | <!--{need link} [Install](../azure-data-studio/data-virtualization-extension.md) --> Instalación desde la galería de extensiones en Azure Data Studio.|
| CLI de Azure (az)<sup>1</sup> | Sí | Interfaz de línea de comandos moderna para administrar servicios de Azure. Se usa con las implementaciones de AKS y para cargar datos de facturación y de inventario de los servicios de datos habilitados para Azure Arc en Azure. ([Más información](/cli/azure/?view=azure-cli-latest&preserve-view=true)). | [Instalación](/cli/azure/install-azure-cli?view=azure-cli-latest&preserve-view=true) |
| CLI de Kubernetes (kubectl)<sup>2</sup> | Sí | Herramienta de línea de comandos para supervisar el clúster de Kubernetes ([más información](https://kubernetes.io/docs/tasks/tools/install-kubectl/)). | [Windows](https://kubernetes.io/docs/tasks/tools/install-kubectl/#install-with-powershell-from-psgallery) \| [Linux](https://kubernetes.io/docs/tasks/tools/install-kubectl/#install-using-native-package-management) |
| curl <sup>3</sup> | Se necesita para algunos scripts de ejemplo. | Herramienta de línea de comandos para transferir datos con direcciones URL. | [Windows](https://curl.haxx.se/windows/) \| Linux: instalación de paquete curl |
| oc | Obligatoria para las implementaciones de Red Hat OpenShift y Red Hat OpenShift en Azure. |`oc` es la interfaz de la línea de comandos (CLI) de Open Shift. | [Instalación de la CLI](https://docs.openshift.com/container-platform/4.4/cli_reference/openshift_cli/getting-started-cli.html#installing-the-cli)



<sup>1</sup> Debe usar la CLI de Azure versión 2.0.4 o posterior. Ejecute `az --version` para encontrar la versión si fuera necesario.

<sup>2</sup> Debe usar la versión 1.13 de `kubectl` o una posterior. Además, la versión de `kubectl` debe ser más o menos una versión secundaria del clúster de Kubernetes. Si quiere instalar una versión concreta en el cliente de `kubectl`, vea [Instalación de `kubectl` binario mediante curl](https://kubernetes.io/docs/tasks/tools/install-kubectl/#install-kubectl-binary-using-curl) (en Windows 10, use cmd.exe y no Windows PowerShell para ejecutar curl).

<sup>3</sup> Si usa PowerShell, curl es un alias del cmdlet Invoke-WebRequest.

## <a name="next-steps"></a>Pasos siguientes

[Creación del controlador de datos de Azure Arc](create-data-controller.md)
