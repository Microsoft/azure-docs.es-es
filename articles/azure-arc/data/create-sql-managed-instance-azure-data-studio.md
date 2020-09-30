---
title: Creación de instancia administrada de Azure SQL con Azure Data Studio
description: Creación de instancia administrada de Azure SQL con Azure Data Studio
services: azure-arc
ms.service: azure-arc
ms.subservice: azure-arc-data
author: twright-msft
ms.author: twright
ms.reviewer: mikeray
ms.date: 09/22/2020
ms.topic: how-to
ms.openlocfilehash: 989496885445a8a0a8d3bbc1a789975a2875c6e0
ms.sourcegitcommit: 53acd9895a4a395efa6d7cd41d7f78e392b9cfbe
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/22/2020
ms.locfileid: "90932375"
---
# <a name="create-sql-managed-instance---azure-arc-using-azure-data-studio"></a>Creación de instancia administrada de SQL: Azure Arc con Azure Data Studio

En este documento se le guía por los pasos para la instalación de instancia administrada de Azure SQL: Azure Arc con Azure Data Studio

[!INCLUDE [azure-arc-common-prerequisites](../../../includes/azure-arc-common-prerequisites.md)]

[!INCLUDE [azure-arc-data-preview](../../../includes/azure-arc-data-preview.md)]

## <a name="log-in-to-the-azure-arc-data-controller"></a>Inicio de sesión en el controlador de datos de Azure Arc

Antes de poder crear una instancia, inicie sesión en el controlador de datos de Azure Arc si todavía no lo ha hecho.

```console
azdata login
```

Después, se le solicitará el espacio de nombres en el que se ha creado el controlador de datos, el nombre de usuario y la contraseña para iniciar sesión en el controlador.  

> Si tiene que validar el espacio de nombres, puede ejecutar ```kubectl get pods -A``` para obtener una lista de todos los espacios de nombres del clúster.

```console
Username: arcadmin
Password:
Namespace: arc
Logged in successfully to `https://10.0.0.4:30080` in namespace `arc`. Setting active context to `arc`
```

## <a name="create-azure-sql-managed-instance-on-azure-arc"></a>Creación de una instancia administrada de Azure SQL en Azure Arc

- Inicio de Azure Data Studio
- En la pestaña Conexiones, haga clic en los tres puntos de la parte superior izquierda y elija "Nueva implementación".
- En las opciones de implementación, seleccione **Instancia administrada de Azure SQL: Azure Arc**. 
  > **Nota:** Es posible que aquí se le pida que instale la CLI de azdata si no lo está actualmente.
- Acepte los términos de privacidad y de licencia, y haga clic en **Seleccionar** en la parte inferior.



- En la hoja Implementar instancia administrada de Azure SQL: Azure Arc, escriba la información siguiente:
  - Escriba un nombre para la instancia de SQL Server
  - Escriba y confirme una contraseña para la instancia de SQL Server
  - Seleccione la clase de almacenamiento adecuada para los datos
  - Seleccione la clase de almacenamiento adecuada para los registros

- Haga clic en el botón **Implementar**

- Esto debería iniciar la creación de la instancia administrada de Azure SQL: Azure Arc en el controlador de datos.

- Tras unos minutos, la creación se completará de forma correcta.

## <a name="connect-to-azure-sql-managed-instance---azure-arc-from-azure-data-studio"></a>Conexión a la instancia administrada de Azure SQL: Azure Arc desde Azure Data Studio

- Inicie sesión en el controlador de datos de Azure Arc; para ello, proporcione el espacio de nombres, el nombre de usuario y la contraseña para el controlador de datos: 
```console
azdata login
```

- Para ver todas las instancias administradas de Azure SQL aprovisionadas, use los comandos siguientes:

```console
azdata arc sql mi list
```

La salida debería ser similar a la siguiente; copie el valor ServerEndpoint (incluido el número de puerto) desde aquí.

```console

Name          Replicas    ServerEndpoint     State
------------  ----------  -----------------  -------
sqlinstance1  1/1         25.51.65.109:1433  Ready
```

- En Azure Data Studio, en la pestaña **Conexiones**, haga clic en **Nueva conexión** en la vista **Servidores**
- En la hoja **Conexión**, pegue el valor ServerEndpoint en el cuadro de texto Servidor
- Seleccione **Inicio de sesión SQL** como tipo de autenticación.
- Escriba *sa* como nombre de usuario.
- Especifique la contraseña de la cuenta `sa`.
- Opcionalmente, escriba el nombre de la base de datos específica a la que quiera conectarse
- Opcionalmente, seleccione o agregue un nuevo grupo de servidores según corresponda
- Seleccione **Conectar** para conectarse a la instancia administrada de Azure SQL: Azure Arc




## <a name="next-steps"></a>Pasos a seguir

Ahora intente [supervisar la instancia de SQL](monitor-grafana-kibana.md)
