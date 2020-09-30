---
title: Paneles de Azure Data Studio
description: Paneles de Azure Data Studio
services: azure-arc
ms.service: azure-arc
ms.subservice: azure-arc-data
author: twright-msft
ms.author: twright
ms.reviewer: mikeray
ms.date: 09/22/2020
ms.topic: how-to
ms.openlocfilehash: c2dd0ce3c3e9a0f1f699b58ed2657394ad9acbe1
ms.sourcegitcommit: 53acd9895a4a395efa6d7cd41d7f78e392b9cfbe
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/22/2020
ms.locfileid: "90932355"
---
# <a name="azure-data-studio-dashboards"></a>Paneles de Azure Data Studio

[Azure Data Studio](https://aka.ms/azuredatastudio) proporciona una experiencia similar a la de Azure Portal para ver información sobre los recursos de Azure Arc.  Estas vistas se denominan **paneles** y tienen un diseño y unas opciones similares a lo que se puede ver sobre un recurso determinado en el Azure Portal, pero le ofrecen la flexibilidad de ver esa información localmente en su entorno en los casos en los que no tiene una conexión disponible para Azure.

[!INCLUDE [azure-arc-data-preview](../../../includes/azure-arc-data-preview.md)]

## <a name="connecting-to-a-data-controller"></a>Conexión a un controlador de datos

### <a name="prerequisites"></a>Requisitos previos

- Descargar [Azure Data Studio](https://aka.ms/getazuredatastudio)
- Extensión de Azure Arc instalada

### <a name="determine-the-data-controller-server-api-endpoint-url"></a>Determinación de la dirección URL del punto de conexión de la API del servidor del controlador de datos

En primer lugar, necesitará conectar Azure Data Studio a la dirección URL del punto de conexión de la API del servicio del controlador de datos.

Para obtener este punto de conexión, puede ejecutar el siguiente comando:

```console
kubectl get svc/controller-svc-external -n <namespace name>

#Example:
kubectl get svc/controller-svc-external -n arc
```

Verá una salida similar a la siguiente:

```console
NAME                      TYPE           CLUSTER-IP     EXTERNAL-IP      PORT(S)                                       AGE
controller-svc-external   LoadBalancer   10.0.175.137   52.154.152.24    30080:32192/TCP                               22h
```

Si usa un tipo de LoadBalancer, deberá copiar la dirección IP externa y el número de puerto 300080.  Si usa NodePort, deberá usar la dirección IP del servidor de la API de Kubernetes y el número de puerto que aparece en la columna PUERTOS.

Ahora, deberá crear una dirección URL para el punto de conexión combinando esta información de la siguiente manera:

```console
https://<ip address>:<port>

Example:
https://52.;154.152.24:30080
```

Tenga en cuenta la dirección IP, ya que la usará en el siguiente paso.

### <a name="connect"></a>Conectar

1. Procedimiento para abrir Azure Data Studio

1. Seleccione la pestaña **Conexiones** a la izquierda.

Hacia la parte inferior, expanda el panel llamado **Controladores de Azure Arc**.

Haga clic en el icono + para agregar una nueva conexión de controlador de datos.

En la parte superior de la pantalla, en la paleta de comandos, escriba la dirección URL que creó en el paso 1 y haga clic en Entrar.
Escriba el nombre de usuario del controlador de datos.  Este era el valor del nombre de usuario que pasó durante la implementación del controlador de datos.  Haga clic en Entrar.
Escriba la contraseña del controlador de datos.  Este era el valor de la contraseña que pasó durante la implementación del controlador de datos. Haga clic en Entrar.

Ahora que está conectado a un controlador de datos, puede ver los paneles del controlador de datos y las instancias administradas de SQL o los recursos del grupo de servidores de hiperescala de PostgreSQL que tenga.

## <a name="view-the-data-controller-dashboard"></a>Vista del panel del controlador de datos

Haga clic con el botón derecho en el controlador de datos del panel de conexiones del panel extensible **Controladores de Arc** y seleccione **Administrar**.

Aquí puede ver los detalles sobre el recurso del controlador de datos como el nombre, la región, el modo de conexión, el grupo de recursos, la suscripción, el punto de conexión del controlador y el espacio de nombres.  También puede ver una lista de todos los recursos de bases de datos administrados por el controlador de datos.

Observará que el diseño es similar al que puede ver en Azure Portal.

Puede iniciar la creación de una instancia administrada de SQL o un grupo de servidores de hiperescala de PostgreSQL fácilmente si hace clic en el botón + Nueva instancia.

También puede abrir el Azure Portal en el contexto de este controlador de datos si hace clic en el botón Abrir en Azure Portal.

## <a name="view-the-sql-managed-instance-dashboards"></a>Vista de los paneles de una instancia administrada de SQL

Si ha creado algunas instancias administradas de SQL, puede verlas en el panel Conexiones, en el panel extensible Controladores de datos de Azure, en el controlador de datos que las administra.

Para ver el panel de instancias administradas de SQL de una instancia determinada, haga clic con el botón derecho en la instancia y elija Administrar.

El panel Conexión aparecerá a la derecha y le pedirá el nombre de inicio de sesión y la contraseña para conectarse a esa instancia de SQL. Si conoce la información de conexión, puede escribirla y hacer clic en Conectar.  Si no la sabe, puede hacer clic en Cancelar.  En cualquier caso, se le dirigirá al panel cuando se cierre el panel Conexión.

En la pestaña Información general, puede ver detalles sobre la instancia administrada de SQL, como grupo de recursos, controlador de datos, identificador de suscripción, estado, región, etc.  También puede ver un vínculo en el que puede hacer clic para ir a los paneles Grafana o Kibana en el contexto de esa instancia administrada de SQL.

Si puede conectarse a la instancia administrada de SQL, puede ver más información aquí.

Puede eliminar la instancia administrada de SQL desde aquí o abrir Azure Portal para verla ahí.

Si hace clic en la pestaña Cadenas de conexión de la izquierda, puede ver una lista de cadenas de conexión construidas previamente para la instancia administrada de SQL, lo que facilita las acciones de copiar y pegar en otras aplicaciones o códigos.

## <a name="view-the-postgresql-hyperscale-server-group-dashboards"></a>Vista de los paneles del grupo de servidores de hiperescala de PostgreSQL

Si ha creado algunos grupos de servidores Hiperescala de PostgreSQL, puede verlos en el panel Conexiones, en el panel extensible Controladores de datos de Azure, en el controlador de datos que los administra.

Para ver el panel del grupo de servidores de hiperescala de PostgreSQL de un grupo de servidores determinado, haga clic con el botón derecho en el grupo de servidores y elija Administrar.

En la pestaña Información general, puede ver detalles sobre el grupo de servidores, como el grupo de recursos, el controlador de datos, el identificador de suscripción, el estado, la región, etc.  También puede ver un vínculo en el que puede hacer clic para ir a los paneles Grafana o Kibana en el contexto de ese grupo de servidores.

Puede eliminar el grupo de servidores desde aquí o abrir el Azure Portal para ver el grupo de servidores en el Azure Portal.

Si hace clic en la pestaña Cadenas de conexión de la izquierda, puede ver una lista de cadenas de conexión construidas previamente para ese grupo de servidores, lo que facilita las acciones de copiar y pegar en otras aplicaciones o códigos.

Si hace clic en la pestaña Propiedades de la izquierda, puede ver más detalles.

Si hace clic en la pestaña Estado de los recursos de la izquierda, puede ver el estado actual de alto nivel de ese grupo de servidores.

Si hace clic en la pestaña Diagnosticar y solucionar problemas a la izquierda, puede iniciar el cuaderno de solución de problemas de PostgreSQL.

Si hace clic en la pestaña Nueva solicitud de soporte técnico de la izquierda, puede iniciar el Azure Portal en el contexto del grupo de servidores y crear una solicitud de soporte técnico de Azure desde allí.
