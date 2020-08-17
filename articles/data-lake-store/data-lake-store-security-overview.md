---
title: Introducción a la seguridad de Azure Data Lake Storage Gen1 | Microsoft Docs
description: Descripción de cómo Azure Data Lake Storage Gen1 es un almacén de macrodatos más seguro
services: data-lake-store
author: twooley
ms.service: data-lake-store
ms.topic: conceptual
ms.date: 03/11/2020
ms.author: twooley
ms.openlocfilehash: a6a6b0b15b8d2dc08f1581cb2ea0ea4c7e8036ca
ms.sourcegitcommit: 3d56d25d9cf9d3d42600db3e9364a5730e80fa4a
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/03/2020
ms.locfileid: "87529983"
---
# <a name="security-in-azure-data-lake-storage-gen1"></a>Seguridad de Azure Data Lake Storage Gen1

Muchas empresas están aprovechando el análisis de macrodatos para obtener conocimientos empresariales que les ayuden a tomar decisiones inteligentes. Es posible que una organización tenga un entorno regulado y complejo, con un número cada vez mayor de distintos tipos de usuarios. Es esencial para una empresa asegurarse de que los datos empresariales más importantes se almacenaron de la forma más segura, y que se concedió a los usuarios individuales el nivel adecuado de acceso. Azure Data Lake Storage Gen1 está diseñado para ayudar a cumplir estos requisitos de seguridad. En este artículo aprenderá las funcionalidades de seguridad de Data Lake Storage Gen1, como:

* Authentication
* Authorization
* Aislamiento de red
* Protección de los datos
* Auditoría

## <a name="authentication-and-identity-management"></a>Autenticación y administración de identidades

La autenticación es el proceso por el que se verifica una identidad de usuario cuando el usuario interactúa con Data Lake Storage Gen1 o con cualquier servicio que se conecte al mismo. Para la autenticación y administración de identidades, Data Lake Storage Gen1 usa [Azure Active Directory](../active-directory/fundamentals/active-directory-whatis.md), una completa solución en la nube de administración de identidades y acceso que simplifica la administración de usuarios y grupos.

Cada una de las suscripciones de Azure puede asociarse a una instancia de Azure Active Directory. Solo los usuarios y las identidades de servicio definidas en el servicio de Azure Active Directory pueden tener acceso a la cuenta de Data Lake Storage Gen1 mediante Azure Portal, las herramientas de línea de comandos o las aplicaciones cliente que la organización compila con el SDK de Data Lake Storage Gen1. Las principales ventajas del uso de Azure Active Directory como un mecanismo de control de acceso centralizado son:

* Administración simplificada del ciclo de vida de las identidades. La identidad de un usuario o un servicio (una identidad principal del servicio) se puede crear y revocar rápidamente; basta con eliminar o deshabilitar la cuenta en el directorio.
* Multi-factor authentication. [Multi-factor authentication](../active-directory/authentication/multi-factor-authentication.md) proporciona un nivel de seguridad adicional para los inicios de sesión y las transacciones del usuario.
* Autenticación desde cualquier cliente mediante un protocolo estándar abierto, como OAuth u OpenID.
* Federación con los servicios de directorio de empresa y los proveedores de identidades en la nube.

## <a name="authorization-and-access-control"></a>Control de autorización y acceso

Una vez que un usuario se autentica mediante Azure Active Directory para acceder a Data Lake Storage Gen1, la autorización controla los permisos de acceso a Data Lake Storage Gen1. Data Lake Storage Gen1 separa la autorización para las actividades relacionadas con cuentas y datos de la siguiente manera:

* [Control de acceso basado en rol](../role-based-access-control/overview.md) (RBAC) proporcionado por Azure para la administración de cuentas
* ACL de POSIX para el acceso a datos en el almacén

### <a name="rbac-for-account-management"></a>RBAC para la administración de cuentas

Se definen cuatro roles básicos para Data Lake Storage Gen1 de forma predeterminada. Estos roles permiten diferentes operaciones en una cuenta de Data Lake Storage Gen1 mediante Azure Portal, los cmdlets de PowerShell y las API REST. Los roles Propietario y Colaborador pueden realizar diversas funciones de administración en la cuenta. Puede asignar el rol Lector a los usuarios que solo ven los datos de administración de cuentas.

![Roles de Azure](./media/data-lake-store-security-overview/rbac-roles.png "Roles de Azure")

Tenga en cuenta que aunque se asignen roles para la administración de cuentas, algunos roles afectarán al acceso a los datos. Para las operaciones que el usuario puede realizar en el sistema de archivos, deberá utilizar listas de control de acceso (ACL). La tabla siguiente muestra un resumen de los derechos de administración y los derechos de acceso a datos para los roles predeterminados.

| Roles | Derechos de administración | Derechos de acceso a datos | Explicación |
| --- | --- | --- | --- |
| Ningún rol asignado |None |Regido por ACL |El usuario no puede utilizar Azure Portal ni los cmdlets de Azure PowerShell para explorar Data Lake Storage Gen1. El usuario solo puede utilizar las herramientas de línea de comandos. |
| Propietario |All |All |El rol de Propietario es un superusuario. Este rol puede administrar todo y tiene acceso completo a los datos. |
| Lector |Solo lectura |Regido por ACL |El rol Lector puede ver todo lo relacionado con la administración de cuentas, como qué usuario está asignado a qué rol. Pero no puede realizar ningún cambio. |
| Colaborador |Todos, excepto agregar y quitar roles |Regido por ACL |El rol Colaborador puede administrar algunos aspectos de una cuenta, como las implementaciones y la creación y administración de alertas. El rol Colaborador no puede agregar ni quitar roles. |
| Administrador de acceso de usuario |Agregar y quitar roles |Regido por ACL |El rol Administrador de acceso de usuarios puede administrar el acceso de los usuarios a las cuentas. |

Para ver instrucciones, consulte [Assign users or security groups to Data Lake Storage Gen1 accounts](data-lake-store-secure-data.md#assign-users-or-security-groups-to-data-lake-storage-gen1-accounts) (Asignación de grupos de seguridad o usuarios a cuentas de Data Lake Storage Gen1).

### <a name="using-acls-for-operations-on-file-systems"></a>Uso de ACL para operaciones en sistemas de archivos

Data Lake Storage Gen1 es un sistema de archivos jerárquico como el sistema de archivos distribuido de Hadoop (HDFS) que admite [POSIX ACL](https://hadoop.apache.org/docs/current/hadoop-project-dist/hadoop-hdfs/HdfsPermissionsGuide.html#ACLs_Access_Control_Lists). Controla los permisos de lectura (r), escritura (w) y ejecución (x) para los recursos del rol Propietario, para el grupo de propietarios y para otros usuarios y grupos. En Data Lake Storage Gen1, las ACL se pueden habilitar en la carpeta raíz, en las subcarpetas y en los archivos individuales. Para obtener más información sobre cómo funcionan las ACL en el contexto de Data Lake Storage Gen1, consulte [Control de acceso en Data Lake Storage Gen1](data-lake-store-access-control.md).

Se recomienda definir las ACL para varios usuarios mediante [grupos de seguridad](../active-directory/fundamentals/active-directory-groups-create-azure-portal.md). Agregue usuarios a un grupo de seguridad y asigne las ACL de los archivos o carpetas a ese grupo de seguridad. Esto es útil cuando desea proporcionar permisos asignados, porque está limitado a un máximo de 28 entradas de permisos asignados. Para obtener más información acerca de cómo proteger los datos almacenados en Data Lake Storage Gen1 mediante grupos de seguridad de Azure Active Directory, consulte [Assign users or security group as ACLs to the Data Lake Storage Gen1 file system](data-lake-store-secure-data.md#filepermissions) (Asignación de usuarios o grupos de seguridad como ACL al sistema de archivos de Data Lake Storage Gen1).

![Lista de permisos de acceso](./media/data-lake-store-security-overview/adl.acl.2.png "Lista de permisos de acceso")

## <a name="network-isolation"></a>Aislamiento de red

Utilice Data Lake Storage Gen1 para ayudar a controlar el acceso a su almacén de datos a nivel de la red. Puede establecer firewalls y definir un intervalo de direcciones IP para los clientes de confianza. Con un intervalo de direcciones IP, solo los clientes que tengan una dirección IP que esté dentro del intervalo definido podrán conectarse a Data Lake Storage Gen1.

![Configuración del firewall y el acceso a IP](./media/data-lake-store-security-overview/firewall-ip-access.png "Configuración del firewall y dirección IP")

Las redes virtuales de Azure admiten etiquetas de servicio para Data Lake Gen 1. Una etiqueta de servicio representa un grupo de prefijos de direcciones IP de un servicio de Azure determinado. Microsoft administra los prefijos de direcciones que la etiqueta de servicio incluye y actualiza automáticamente dicha etiqueta a medida que las direcciones cambian. Para más información, consulte [Introducción a las etiquetas de servicio de Azure](../virtual-network/service-tags-overview.md).

## <a name="data-protection"></a>Protección de los datos

Data Lake Storage Gen1 protege los datos durante todo su ciclo de vida. Para los datos en tránsito, Data Lake Storage Gen1 utiliza el protocolo Seguridad de la capa de transporte (TLS 1.2) estándar del sector para proteger los datos de la red.

![Cifrado en Data Lake Storage Gen1](./media/data-lake-store-security-overview/adls-encryption.png "Cifrado en Data Lake Storage Gen1")

Data Lake Storage Gen1 también proporciona el cifrado de los datos que se almacenan en la cuenta. Puede elegir si cifrar o no los datos. Si se decide por el cifrado, los datos almacenados en Data Lake Storage Gen1 se cifrarán antes de almacenarlos en medios persistentes. En tal caso, Data Lake Storage Gen1 cifrará automáticamente los datos antes de la persistencia y los descifrará antes de la recuperación, por lo que resulta un proceso completamente transparente para el cliente que accede a los datos. No se requiere ningún cambio de código en el lado del cliente para cifrar o descifrar datos.

Para la administración de claves, Data Lake Storage Gen1 ofrece dos modos de administrar las claves de cifrado maestras (MEK) que son necesarias para descifrar los datos que se almacenan en Data Lake Storage Gen1. Puede permitir que Data Lake Storage Gen1 administre las MEK en su nombre o decidir conservar la propiedad de estas mediante su cuenta de Azure Key Vault. Puede especificar el modo de administración de claves durante la creación de una cuenta de Data Lake Storage Gen1. Para obtener más información sobre cómo proporcionar una configuración relacionada con el cifrado, consulte [Introducción a Azure Data Lake Storage Gen1 con Azure Portal](data-lake-store-get-started-portal.md).

## <a name="activity-and-diagnostic-logs"></a>Registros de actividad y diagnóstico

Puede usar los registros de actividad o diagnóstico en función de si desea consultar registros de actividades relacionadas con la administración de cuentas o con los datos.

* Las actividades relacionadas con la administración de cuentas utilizan las API de Azure Resource Manager y se exponen en Azure Portal mediante los registros de actividad.
* Las actividades relacionadas con los datos usan las API de REST de WebHDFS y se exponen en el Portal de Azure mediante los registros de diagnóstico.

### <a name="activity-log"></a>Registro de actividades

Para cumplir las normativas, las organizaciones pueden requerir pistas de auditoría adecuadas de las actividades de administración de cuentas si necesitan profundizar en un incidente concreto. Data Lake Storage Gen1 cuenta con supervisión integrada y registra todas las actividades de administración de cuentas.

Para las trazas de auditoría de administración de cuentas, vea y elija las columnas que desea registrar. También puede exportar los registros de actividad a Azure Storage.

![Registro de actividad](./media/data-lake-store-security-overview/activity-logs.png "Registro de actividades")

Para obtener más información sobre cómo trabajar con los registros de actividad, consulte [Visualización de registros de actividad para auditar las acciones sobre los recursos](../azure-resource-manager/management/view-activity-logs.md).

### <a name="diagnostics-logs"></a>Registros de diagnóstico

Puede habilitar el registro de auditoría y diagnóstico de acceso a los datos en Azure Portal y enviar los registros a una cuenta de Azure Blob Storage, un centro de eventos o registros de Azure Monitor.

![Registros de diagnóstico](./media/data-lake-store-security-overview/diagnostic-logs.png "Registros de diagnóstico")

Para obtener más información sobre cómo trabajar con registros de diagnóstico con Data Lake Storage Gen1, consulte [Accessing diagnostic logs for Data Lake Storage Gen1](data-lake-store-diagnostic-logs.md) (Acceso a los registros de diagnóstico de Data Lake Storage Gen1).

## <a name="summary"></a>Resumen

Los clientes empresariales demandan una plataforma en la nube de análisis de datos segura y fácil de usar. Data Lake Storage Gen1 se ha diseñado para satisfacer estos requisitos con la administración de identidades y la autenticación mediante la integración de Azure Active Directory, la autorización basada en ALC, el aislamiento de red, el cifrado de datos en tránsito y en reposo, así como la auditoría.

Si quiere ver las nuevas características de Data Lake Storage Gen1, envíenos sus comentarios en el [foro de UserVoice de Data Lake Storage Gen1](https://feedback.azure.com/forums/327234-data-lake).

## <a name="see-also"></a>Consulte también

* [Introducción a Azure Data Lake Storage Gen1](data-lake-store-overview.md)
* [Introducción a Data Lake Storage Gen1](data-lake-store-get-started-portal.md)
* [Protección de datos en Data Lake Storage Gen1](data-lake-store-secure-data.md)
