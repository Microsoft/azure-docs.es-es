---
title: Uso de Azure Data Studio para administrar la instancia de PostgreSQL
description: Uso de Azure Data Studio para administrar la instancia de PostgreSQL
services: azure-arc
ms.service: azure-arc
ms.subservice: azure-arc-data
author: TheJY
ms.author: jeanyd
ms.reviewer: mikeray
ms.date: 09/22/2020
ms.topic: how-to
ms.openlocfilehash: fc0ad45f575f9190f15b61acdf476c716b7f1638
ms.sourcegitcommit: 53acd9895a4a395efa6d7cd41d7f78e392b9cfbe
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/22/2020
ms.locfileid: "90932604"
---
# <a name="use-azure-data-studio-to-manage-your-azure-arc-enabled-postgresql-hyperscale-server-group"></a>Uso de Azure Data Studio para administrar el grupo de servidores de Hiperescala de PostgreSQL habilitada para Azure Arc


En este artículo se describe cómo:
- Administrar las instancias de PostgreSQL con vistas de panel para información general, cadenas de conexión, propiedades o Resource Health.
- Trabajar con los datos y el esquema.

[!INCLUDE [azure-arc-data-preview](../../../includes/azure-arc-data-preview.md)]

## <a name="prerequisites"></a>Requisitos previos

- [Instalación de azdata, Azure Data Studio y la CLI de Azure](install-client-tools.md)
- Instalación en Azure Data Studio de las extensiones de la **CLI de datos de Azure**, **Azure Arc** y **PostgreSQL**
- Creación del [controlador de datos de Azure Arc](create-data-controller-using-azdata.md)
- Inicio de Azure Data Studio

## <a name="connect-to-the-azure-arc-data-controller"></a>Conexión al controlador de datos de Azure Arc

En Azure Data Studio, expanda el nodo **Azure Arc Controllers** (Controladores de Azure Arc) y seleccione el botón **Connect Controller** (Conectar controlador):

Escriba la información de conexión al controlador de datos de Azure:

- **Controller URL (Dirección URL del controlador):**

    dirección URL para conectarse al controlador en Kubernetes. Escrita con el formato `https://<IP_address_of_the_controller>:<Kubernetes_port.` Por ejemplo:

    ```console
    https://12.345.67.890:30080
    ```
- **Nombre de usuario:**

    nombre de la cuenta de usuario que usa para conectarse al controlador. Utilice el nombre que usa normalmente al ejecutar `azdata login`. No es el nombre del usuario de PostgreSQL que usa para conectarse al motor de base de datos de PostgreSQL, normalmente desde psql.
- **Contraseña:** contraseña de la cuenta de usuario que usa para conectarse al controlador.


Azure Data Studio muestra el controlador de datos de Arc. Expándalo y mostrará la lista de instancias de PostgreSQL que administra.

## <a name="manage-your-azure-arc-enabled-postgresql-hyperscale-server-groups"></a>Administración del grupo de servidores de Hiperescala de PostgreSQL habilitada para Azure Arc

Haga clic con el botón derecho en la instancia de PostgreSQL que quiere administrar y seleccione [Manage] (Administrar).

Vista del panel de PostgreSQL:

Incluye varios paneles que aparecen en el lado izquierdo de ese panel:

- **Información general:** muestra información de resumen sobre la instancia, como el nombre, el identificador de suscripción de Azure, la configuración, la versión del motor de base de datos, los puntos de conexión de Grafana y Kibana...
- **Connection Strings (Cadenas de conexión):** muestra varias cadenas de conexión que puede que necesite para conectarse a la instancia de PostgreSQL, como psql, Node.js, PHP, Ruby...
- **Propiedades:** muestra varias propiedades, como el nombre del usuario administrador de PostgreSQL, el grupo de recursos asociado para el recurso de instantáneas...
- **Diagnose and solve problems (Diagnóstico y solución de problemas)** : página de aterrizaje en la que encontrará varios recursos que le ayudarán a solucionar problemas de la instancia a medida que se expanden los cuadernos de solución de problemas.
- **New support request (Nueva solicitud de soporte técnico):** página de aterrizaje desde la que podrá solicitar asistencia de nuestros servicios de soporte técnico a partir del anuncio de la versión preliminar pública.

## <a name="work-with-your-data-and-schema"></a>Trabajo con los datos y el esquema

En el lado izquierdo de la ventana de Azure Data Studio, expanda el nodo **Servers (Servidores)** :

Seleccione [Add Connection] (Agregar conexión) y rellene los detalles de la conexión a la instancia de PostgreSQL:
- **Tipo de conexión:** PostgreSQL
- **Server name (Nombre del servidor):** escriba el nombre de la instancia de PostgreSQL. Por ejemplo: postgres01
- **Authentication type** (Tipo de autenticación): Contraseña
- **User name (Nombre de usuario):** puede usar el nombre del usuario administrador de PostgreSQL predeterminado estándar, por ejemplo. Tenga en cuenta que este campo distingue mayúsculas de minúsculas.
- **Password (Contraseña):** encontrará la contraseña del nombre de usuario de PostgreSQL en la cadena de conexión de psql en la salida del comando `azdata postgres server endpoint -n postgres01`.
- **Database name (Nombre de la base de datos):** establezca el nombre de la base de datos a la que desea conectarse. Puede dejar el campo establecido en __Default__ (Predeterminada).
- **Server group (Grupo de servidores):** puede dejar el campo establecido en __Default__ (Predeterminado).
- **Name (Nombre) (opcional):** puede dejarlo en blanco.
- **Advanced (Avanzado):**
    - **Host IP Address (Dirección IP del host):** es la dirección IP pública del clúster de Kubernetes.
    - **Port (Puerto):** puerto en el que escucha la instancia de PostgreSQL. Puede encontrar este puerto al final de la cadena de conexión de psql en la salida del comando `azdata postgres server endpoint -n postgres01`. No es el puerto 30080 en el que escucha Kubernetes y que escribió al conectarse al controlador de datos de Azure en Azure Data Studio.
    - **Other parameters (Otros parámetros):** deberían ser autoexplícitos; puede dejar los valores predeterminados o en blanco que aparecen.

Seleccione **[OK] (Aceptar) y [Connect] Conectar** para conectarse al servidor.

Una vez conectado, hay disponibles varias experiencias:
- **Nueva consulta**
- **Nuevo cuaderno**
- **Expanda la pantalla del servidor y examine o trabaje con los objetos de la base de datos.**
- **...**

## <a name="next-step"></a>Paso siguiente
[Supervisión del grupo de servidores](monitor-grafana-kibana.md)
