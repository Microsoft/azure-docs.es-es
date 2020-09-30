---
title: Creación de un grupo de servidores Hiperescala de Azure Database for PostgreSQL en Azure Arc
description: Creación de un grupo de servidores Hiperescala de Azure Database for PostgreSQL en Azure Arc
services: azure-arc
ms.service: azure-arc
ms.subservice: azure-arc-data
author: TheJY
ms.author: jeanyd
ms.reviewer: mikeray
ms.date: 09/22/2020
ms.topic: how-to
ms.openlocfilehash: e845136c4fed5a3d2e6863fdab0aa9f70fb30b5d
ms.sourcegitcommit: 53acd9895a4a395efa6d7cd41d7f78e392b9cfbe
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/22/2020
ms.locfileid: "90932376"
---
# <a name="create-an-azure-arc-enabled-postgresql-hyperscale-server-group"></a>Creación de un grupo de servidores Hiperescala de PostgreSQL habilitado para Azure Arc

En este documento se describen los pasos para crear un grupo de servidores Hiperescala de PostgreSQL en Azure Arc.

[!INCLUDE [azure-arc-common-prerequisites](../../../includes/azure-arc-common-prerequisites.md)]

[!INCLUDE [azure-arc-data-preview](../../../includes/azure-arc-data-preview.md)]

## <a name="getting-started"></a>Introducción
Si ya está familiarizado con los temas siguientes, puede omitir este párrafo.
Hay temas importantes que puede que le interese leer antes de continuar con la creación:
- [Información general sobre los servicios de datos habilitados para Azure Arc](overview.md)
- [Modos de conectividad y requisitos](connectivity.md)
- [Conceptos de almacenamiento de Kubernetes y configuración de almacenamiento](storage-configuration.md)
- [Modelo de recursos de Kubernetes](https://github.com/kubernetes/community/blob/master/contributors/design-proposals/scheduling/resources.md#resource-quantities)

Si prefiere realizar pruebas sin aprovisionar un entorno completo, empiece a trabajar rápidamente con [Azure Arc JumpStart](https://github.com/microsoft/azure_arc#azure-arc-enabled-data-services) en Azure Kubernetes Service (AKS), AWS Elastic Kubernetes Service (EKS), Google Cloud Kubernetes Engine (GKE) o en una máquina virtual de Azure.


## <a name="login-to-the-azure-arc-data-controller"></a>Inicio de sesión en el controlador de datos de Azure Arc

Antes de poder crear una instancia, primero debe iniciar sesión en el controlador de datos de Azure Arc. Si ya ha iniciado sesión en el controlador de datos, puede omitir este paso.

```console
azdata login
```

Después, se le pedirá el nombre de usuario, la contraseña y el espacio de nombres del sistema.  

> Si ha usado el script para crear el controlador de datos, el espacio de nombres debe ser **arc**.

```console
Namespace: arc
Username: arcadmin
Password:
Logged in successfully to `https://10.0.0.4:30080` in namespace `arc`. Setting active context to `arc`
```

## <a name="preliminary-and-temporary-step-for-openshift-users-only"></a>Paso preliminar y temporal solo para usuarios de OpenShift

Implemente este paso antes de pasar al siguiente. Para implementar el grupo de servidores Hiperescala de PostgreSQL en Red Hat OpenShift en un proyecto distinto del predeterminado, debe ejecutar los comandos siguientes en el clúster para actualizar las restricciones de seguridad. Este comando concede los privilegios necesarios a las cuentas de servicio que ejecutarán el grupo de servidores Hiperescala de PostgreSQL. La restricción de contexto de seguridad (SCC) **_arc-data-scc_** es la que ha agregado al implementar el controlador de datos de Azure Arc.

```console
oc adm policy add-scc-to-group arc-data-scc -z <server-group-name> -n <namespace name>
```

_**Server-group-name** es el nombre del grupo de servidores que se va a crear en el siguiente paso._
   
Para obtener más detalles sobre las SCC en OpenShift, consulte la [documentación de OpenShift](https://docs.openshift.com/container-platform/4.2/authentication/managing-security-context-constraints.html).
Ahora puede implementar el paso siguiente.

## <a name="create-an-azure-database-for-postgresql-hyperscale-server-group"></a>Creación de un grupo de servidores Hiperescala de Azure Database for PostgreSQL

Para crear un grupo de servidores Hiperescala de Azure Database for PostgreSQL en Azure Arc, use el comando siguiente:

```console
azdata arc postgres server create -n <name> --workers 2 --storage-class-data <storage class name> --storage-class-logs <storage class name> --storage-class-backups <storage class name>

#Example
#azdata arc postgres server create -n postgres01 --workers 2
```

> [!NOTE]
> - **Hay otros parámetros de línea de comandos disponibles.  Para ver la lista completa de opciones, ejecute `azdata arc postgres server create --help`.**
> - En Versión preliminar, debe indicar una clase de almacenamiento para las copias de seguridad ( _--storage-class-backups -scb_) en el momento de crear un grupo de servidores para poder realizar copias de seguridad y restauraciones.
> - La unidad aceptada por los parámetros --volume-size-* es una cantidad de recursos Kubernetes (es suficiente con un entero seguido de uno de estos valores SI (T, G, M, K, m), o bien sus equivalentes elevados a la potencia de dos (Ti, Gi, Mi, Ki)).
> - Los nombres deben tener una longitud de 10 caracteres o menos, y ajustarse a las convenciones de nomenclatura de DNS.
> - Se le pedirá que escriba la contraseña del usuario administrativo estándar _postgres_.  Puede omitir el mensaje interactivo si establece la variable de entorno de sesión `AZDATA_PASSWORD` antes de ejecutar el comando create.
> - Si ha implementado el controlador de datos con AZDATA_USERNAME y AZDATA_PASSWORD en la misma sesión de terminal, los valores de AZDATA_USERNAME y AZDATA_PASSWORD también se usarán para implementar el grupo de servidores Hiperescala de PostgreSQL. El nombre del usuario administrador predeterminado para el motor de base de datos de Hiperescala de PostgreSQL es _postgresql_ y no se puede cambiar en este momento.
> - La creación de un grupo de servidores Hiperescala de PostgreSQL no registrará los recursos en Azure de forma inmediata. Como parte del proceso de carga de [inventario de recursos](upload-metrics-and-logs-to-azure-monitor.md) o [datos de uso](view-billing-data-in-azure.md) a Azure, los recursos se crearán en Azure y podrá verlos en Azure Portal.
> - En este momento no se puede cambiar el parámetro --port.
> - Si no tiene una clase de almacenamiento predeterminada en el clúster de Kubernetes, tendrá que usar el parámetro --metadataStorageClass para especificar una. Si no lo hace, se producirá un error en el comando create. Para comprobar si tiene una clase de almacenamiento predeterminada declarada en el clúster de Kubernetes, ejecute el comando siguiente: 
>
>   ```console
>   kubectl get sc
>   ```
>
> - Si hay una clase de almacenamiento configurada como la clase de almacenamiento predeterminada, verá **(default)** junto a su nombre. Por ejemplo:
>
>   ```output
>   NAME                       PROVISIONER                        AGE
>   local-storage (default)    kubernetes.io/no-provisioner       4d18h
>   ```


## <a name="list-your-azure-database-for-postgresql-server-groups-created-in-your-arc-setup"></a>Enumeración de los grupos de servidores de Azure Database for PostgreSQL creados en la instalación de Arc

Para ver los grupos de servidores Hiperescala de PostgreSQL en Azure Arc, use el comando siguiente:

```console
azdata arc postgres server list
```


```output
Name        State     Workers
----------  --------  ---------
postgres01  Ready     2
```

## <a name="get-the-endpoints-to-connect-to-your-azure-database-for-postgresql-server-groups"></a>Conexión de los puntos de conexión a los grupos de servidores de Azure Database for PostgreSQL

Para ver los puntos de conexión de una instancia de PostgreSQL, ejecute el comando siguiente:

```console
azdata arc postgres endpoint list -n <server group name>
```
Por ejemplo:
```console
[
  {
    "Description": "PostgreSQL Instance",
    "Endpoint": "postgresql://postgres:<replace with password>@12.345.123.456:1234"
  },
  {
    "Description": "Log Search Dashboard",
    "Endpoint": "https://12.345.123.456:12345/kibana/app/kibana#/discover?_a=(query:(language:kuery,query:'custom_resource_name:\"postgres01\"'))"
  },
  {
    "Description": "Metrics Dashboard",
    "Endpoint": "https://12.345.123.456:12345/grafana/d/postgres-metrics?var-Namespace=arc3&var-Name=postgres01"
  }
]
```

Puede usar el punto de conexión de la instancia de PostgreSQL para conectarse al grupo de servidores Hiperescala de PostgreSQL desde la herramienta que prefiera:  [Azure Data Studio](https://aka.ms/getazuredatastudio), [pgcli](https://www.pgcli.com/) psql, pgAdmin, etc.

Si usa una máquina virtual de Azure para realizar pruebas, siga estas instrucciones:

## <a name="special-note-about-azure-virtual-machine-deployments"></a>Nota especial sobre las implementaciones de máquinas virtuales de Azure

Cuando se usa una máquina virtual de Azure, la dirección IP del punto de conexión no mostrará la _IP pública_. Para localizar la IP pública, use el comando siguiente:

```console
az network public-ip list -g azurearcvm-rg --query "[].{PublicIP:ipAddress}" -o table
```

Después, puede combinar la IP pública con el puerto para establecer la conexión.

Es posible que también tenga que exponer el puerto del grupo de servidores Hiperescala de PostgreSQL a través de la puerta de enlace de seguridad de red (NSG). Para permitir el tráfico a través de NSG, tendrá que agregar una regla mediante el comando siguiente:

Para establecer una regla, tendrá que conocer el nombre de su NSG. Puede determinar el NSG con el comando siguiente:

```console
az network nsg list -g azurearcvm-rg --query "[].{NSGName:name}" -o table
```

Una vez que tenga el nombre del NSG, puede agregar una regla de firewall mediante el comando siguiente. En estos valores de ejemplo se crea una regla de NSG para el puerto 30655 y se permite la conexión desde **cualquier** dirección IP de origen.  No es un procedimiento recomendado de seguridad.  Puede mejorar la operación si especifica un valor -source-address-prefixes específico de la dirección IP del cliente, o bien un intervalo de direcciones IP que abarque las direcciones IP del equipo o la organización.

Reemplace el valor del parámetro --destination-port-ranges siguiente por el número de puerto que ha obtenido del comando "azdata arc postgres server list" anterior.

```console
az network nsg rule create -n db_port --destination-port-ranges 30655 --source-address-prefixes '*' --nsg-name azurearcvmNSG --priority 500 -g azurearcvm-rg --access Allow --description 'Allow port through for db access' --destination-address-prefixes '*' --direction Inbound --protocol Tcp --source-port-ranges '*'
```

## <a name="connect-with-azure-data-studio"></a>Conexión con Azure Data Studio

Abra Azure Data Studio y conéctese a la instancia con la dirección IP y el número de puerto del punto de conexión externo anterior, y la contraseña que haya especificado al crear la instancia.  Si PostgreSQL no está disponible en la lista desplegable *Tipo de conexión*, puede instalar la extensión PostgreSQL si busca PostgreSQL en la pestaña Extensiones.

> [!NOTE]
> Tendrá que hacer clic en el botón [Avanzado] del panel de conexión para escribir el número de puerto.

Recuerde que, si usa una máquina virtual de Azure, necesitará la _IP pública_ a la que se pueda acceder mediante el comando siguiente:

```console
az network public-ip list -g azurearcvm-rg --query "[].{PublicIP:ipAddress}" -o table
```

## <a name="connect-with-psql"></a>Conexión con psql

Para acceder al grupo de servidores Hiperescala de PostgreSQL, pase el punto de conexión externo del grupo de servidores Hiperescala de PostgreSQL que ha recuperado antes:

Ahora puede conectarse a psql:

```console
psql postgresql://postgres:<EnterYourPassword>@10.0.0.4:30655
```

## <a name="next-steps"></a>Pasos siguientes

- Lea los conceptos y las guías paso a paso de Hiperescala de Azure Database for PostgreSQL para distribuir los datos entre varios nodos de Hiperescala de PostgreSQL y beneficiarse de toda la eficacia de esta opción. :
    * [Nodos y tablas](../../postgresql/concepts-hyperscale-nodes.md)
    * [Determinar el tipo de aplicación](../../postgresql/concepts-hyperscale-app-type.md)
    * [Elección de una columna de distribución](../../postgresql/concepts-hyperscale-choose-distribution-column.md)
    * [Colocación de tabla](../../postgresql/concepts-hyperscale-colocation.md)
    * [Distribución y modificación de tablas](../../postgresql/howto-hyperscale-modify-distributed-tables.md)
    * [Diseño de una base de datos multiinquilino](../../postgresql/tutorial-design-database-hyperscale-multi-tenant.md)*
    * [Diseño de un panel de análisis en tiempo real](../../postgresql/tutorial-design-database-hyperscale-realtime.md)*

    > \* En los documentos anteriores, omita las secciones **Inicio de sesión en Azure Portal** y **Creación de una instancia de Azure Database for PostgreSQL: Hiperescala (Citus)** . Implemente los pasos restantes en la implementación de Azure Arc. Esas secciones son específicas de Hiperescala (Citus) de Azure Database for PostgreSQL que se ofrece como un servicio PaaS en la nube de Azure, pero las demás partes de los documentos se aplican directamente a Hiperescala de PostgreSQL habilitada para Azure Arc.

- [Escalado del grupo de servidores Hiperescala de Azure Database for PostgreSQL](scale-out-postgresql-hyperscale-server-group.md)
- [Conceptos de almacenamiento de Kubernetes y configuración de almacenamiento](storage-configuration.md)
- [Expansión de notificaciones de volúmenes persistentes](https://kubernetes.io/docs/concepts/storage/persistent-volumes/#expanding-persistent-volumes-claims)
- [Modelo de recursos de Kubernetes](https://github.com/kubernetes/community/blob/master/contributors/design-proposals/scheduling/resources.md#resource-quantities)
