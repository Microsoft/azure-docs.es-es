---
title: archivo de inclusión
description: archivo de inclusión
services: container-instances
author: dlepow
ms.service: container-instances
ms.topic: include
ms.date: 08/13/2020
ms.author: danlep
ms.custom: include file
ms.openlocfilehash: 3221c4f3e196cf1573bd7c0424fa3b4530c0b2ca
ms.sourcegitcommit: 7374b41bb1469f2e3ef119ffaf735f03f5fad484
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/16/2020
ms.locfileid: "90708297"
---
## <a name="create-azure-context"></a>Creación de un contexto de Azure

Para usar los comandos de Docker para ejecutar contenedores en Azure Container Instances, primero inicie sesión en Azure:

```bash
docker login azure
```

Cuando se le solicite, escriba o seleccione sus credenciales de Azure.


Cree un contexto ACI; para ello, ejecute `docker context create aci`. Este contexto asocia Docker con una suscripción de Azure y un grupo de recursos para que pueda crear y administrar las instancias de contenedor. Por ejemplo, para crear un contexto llamado *myacicontext*:

```
docker context create aci myacicontext
```

Cuando se le solicite, seleccione el id. de suscripción de Azure y después seleccione un grupo de recursos existente o **cree un nuevo grupo de recursos**. Si elige un nuevo grupo de recursos, se creará con un nombre generado por el sistema. Las instancias de contenedores de Azure, al igual que otros recursos de Azure, se deben implementar en un grupo de recursos. Los grupos de recursos le permiten organizar y administrar los recursos relacionados de Azure.


Ejecute `docker context ls` para confirmar que agregó el contexto de ACI a los contextos de Docker:

```
docker context ls
```