---
title: archivo de inclusión
description: archivo de inclusión
services: app-service
author: cephalin
ms.service: app-service
ms.topic: include
ms.date: 02/02/2018
ms.author: cephalin
ms.custom: include file
ms.openlocfilehash: 8539696f4521a1b4a2f56fe7d2936b45dec26ec9
ms.sourcegitcommit: 2ffa5bae1545c660d6f3b62f31c4efa69c1e957f
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/11/2020
ms.locfileid: "88077716"
---
En la ventana del terminal local, agregue una instancia remota de Azure al repositorio de Git local. Reemplace *\<deploymentLocalGitUrl-from-create-step>* por la dirección URL del repositorio Git remoto que guardó en [Creación de una aplicación web](#create-a-web-app).

```bash
git remote add azure <deploymentLocalGitUrl-from-create-step>
```

Realice la insercion en la instancia remota de Azure para implementar la aplicación con el comando siguiente. Cuando el Administrador de credenciales de Git le solicite las credenciales, asegúrese de que especifica las que creó en **Configuración de un usuario de implementación**, no las que se usan para iniciar sesión en Azure Portal.

```bash
git push azure master
```

Este comando puede tardar varios minutos en ejecutarse. Durante la ejecución, muestra información similar a la del ejemplo siguiente:
