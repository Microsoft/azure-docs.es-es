---
title: 'Autorización: Herramienta de modelado de amenazas de Microsoft (Azure) | Microsoft Docs'
description: Obtenga información sobre la mitigación de la autorización en Threat Modeling Tool. Consulte una lista de posibles amenazas e instrucciones de mitigación.
services: security
documentationcenter: na
author: jegeib
manager: jegeib
editor: jegeib
ms.assetid: na
ms.service: security
ms.subservice: security-develop
ms.workload: na
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: article
ms.date: 02/07/2017
ms.author: jegeib
ms.custom: devx-track-csharp
ms.openlocfilehash: 51d8b740ba1275b23bc17a58284141dce0d48fe0
ms.sourcegitcommit: 58d3b3314df4ba3cabd4d4a6016b22fa5264f05a
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/02/2020
ms.locfileid: "89300007"
---
# <a name="security-frame-authorization--mitigations"></a>Marco de seguridad: Autorización | Mitigaciones 
| Producto o servicio | Artículo |
| --------------- | ------- |
| **Límite de confianza de la máquina** | <ul><li>[Comprobación de que hay configuradas ACL adecuadas para restringir el acceso no autorizado a los datos en el dispositivo](#acl-restricted-access)</li><li>[Comprobación de que el contenido de la aplicación confidencial del usuario se almacena en el directorio del perfil de usuario](#sensitive-directory)</li><li>[Comprobación de que las aplicaciones implementadas se ejecutan con privilegios mínimos](#deployed-privileges)</li></ul> |
| **Aplicación web** | <ul><li>[Aplicación de un orden secuencial al procesar flujos de lógica empresarial](#sequential-logic)</li><li>[Implementación de un mecanismo de limitación de velocidad para impedir la enumeración](#rate-enumeration)</li><li>[Comprobación de que se aplica la autorización adecuada y se cumple el principio de privilegios mínimos](#principle-least-privilege)</li><li>[Las decisiones sobre la autorización para acceder a recursos y lógicas empresariales no deben basarse en parámetros de solicitud entrantes](#logic-request-parameters)</li><li>[Comprobación de que no es posible enumerar ni acceder al contenido ni los recursos mediante navegación forzada](#enumerable-browsing)</li></ul> |
| **Base de datos** | <ul><li>[Comprobación de que se emplean cuentas con privilegios mínimos para conectarse al servidor de base de datos](#privileged-server)</li><li>[Implementación de seguridad de nivel de fila (RLS) para impedir que los inquilinos accedan a los datos de los demás](#rls-tenants)</li><li>[El rol sysadmin solo debe tener a los usuarios válidos necesarios](#sysadmin-users)</li></ul> |
| **Puerta de enlace de nube de IoT** | <ul><li>[Conexión a la puerta de enlace de la nube mediante tokens con privilegios mínimos](#cloud-least-privileged)</li></ul> |
| **Centro de eventos de Azure** | <ul><li>[Uso de una clave SAS de permisos solo de envío para generar tokens de dispositivo](#sendonly-sas)</li><li>[No usar tokens de acceso que proporcionan acceso directo al centro de eventos](#access-tokens-hub)</li><li>[Conexión al centro de eventos mediante claves SAS que tienen los permisos mínimos necesarios](#sas-minimum-permissions)</li></ul> |
| **Azure Document DB** | <ul><li>[Uso de tokens de recursos para conectarse a Azure Cosmos DB siempre que sea posible](#resource-docdb)</li></ul> |
| **Límites de confianza de Azure** | <ul><li>[Habilitación de administración avanzada de acceso a suscripción de Azure mediante RBAC](#grained-rbac)</li></ul> |
| **Límites de confianza de Service Fabric** | <ul><li>[Restricción del acceso de cliente a operaciones de clúster mediante RBAC](#cluster-rbac)</li></ul> |
| **Dynamics CRM** | <ul><li>[Realización de modelos de seguridad y uso de seguridad de nivel de campo cuando sea necesario](#modeling-field)</li></ul> |
| **Portal de Dynamics CRM** | <ul><li>[Realización de modelos de seguridad de cuentas del portal teniendo en cuenta que el modelo de seguridad para el portal es distinto al resto de CRM](#portal-security)</li></ul> |
| **Almacenamiento de Azure** | <ul><li>[Concesión de permiso detallado en una serie de entidades de Azure Table Storage](#permission-entities)</li><li>[Habilitación de control de acceso basado en roles (RBAC) para una cuenta de almacenamiento de Azure mediante Azure Resource Manager](#rbac-azure-manager)</li></ul> |
| **Cliente para dispositivos móviles** | <ul><li>[Implementación de detección implícita de jailbreak o rooting](#rooting-detection)</li></ul> |
| **WCF** | <ul><li>[Referencia débil de clase en WCF](#weak-class-wcf)</li><li>[WCF: implementación de control de autorización](#wcf-authz)</li></ul> |
| **API web** | <ul><li>[Implementación del mecanismo de autorización adecuado en ASP.NET Web API](#authz-aspnet)</li></ul> |
| **Dispositivo IoT** | <ul><li>[Realización de comprobaciones de autorización en el dispositivo si admite varias acciones que requieren distintos niveles de permiso](#device-permission)</li></ul> |
| **Puerta de enlace de campo de IoT** | <ul><li>[Realización de comprobaciones de autorización en la puerta de enlace de campo si admite varias acciones que requieren distintos niveles de permiso](#field-permission)</li></ul> |

## <a name="ensure-that-proper-acls-are-configured-to-restrict-unauthorized-access-to-data-on-the-device"></a><a id="acl-restricted-access"></a>Comprobación de que hay configuradas ACL adecuadas para restringir el acceso no autorizado a los datos en el dispositivo

| Título                   | Detalles      |
| ----------------------- | ------------ |
| **Componente**               | Límite de confianza de la máquina | 
| **Fase de SDL**               | Implementación |  
| **Tecnologías aplicables** | Genérico |
| **Atributos**              | N/D  |
| **Referencias**              | N/D  |
| **Pasos** | Asegúrese de que hay configuradas ACL adecuadas para restringir el acceso no autorizado a los datos en el dispositivo.|

## <a name="ensure-that-sensitive-user-specific-application-content-is-stored-in-user-profile-directory"></a><a id="sensitive-directory"></a>Comprobación de que el contenido de la aplicación confidencial del usuario se almacena en el directorio del perfil de usuario

| Título                   | Detalles      |
| ----------------------- | ------------ |
| **Componente**               | Límite de confianza de la máquina | 
| **Fase de SDL**               | Implementación |  
| **Tecnologías aplicables** | Genérico |
| **Atributos**              | N/D  |
| **Referencias**              | N/D  |
| **Pasos** | Compruebe que el contenido de la aplicación confidencial del usuario se almacena en el directorio del perfil de usuario. De esta manera, se evita que los diversos usuarios de la máquina tengan acceso a los datos de los demás.|

## <a name="ensure-that-the-deployed-applications-are-run-with-least-privileges"></a><a id="deployed-privileges"></a>Comprobación de que las aplicaciones implementadas se ejecutan con privilegios mínimos

| Título                   | Detalles      |
| ----------------------- | ------------ |
| **Componente**               | Límite de confianza de la máquina | 
| **Fase de SDL**               | Implementación |  
| **Tecnologías aplicables** | Genérico |
| **Atributos**              | N/D  |
| **Referencias**              | N/D  |
| **Pasos** | Compruebe que la aplicación implementada se ejecuta con privilegios mínimos. |

## <a name="enforce-sequential-step-order-when-processing-business-logic-flows"></a><a id="sequential-logic"></a>Aplicación de un orden secuencial al procesar flujos de lógica empresarial

| Título                   | Detalles      |
| ----------------------- | ------------ |
| **Componente**               | Aplicación web | 
| **Fase de SDL**               | Build |  
| **Tecnologías aplicables** | Genérico |
| **Atributos**              | N/D  |
| **Referencias**              | N/D  |
| **Pasos** | Si desea comprobar que un usuario auténtico ha completado esta fase, es recomendable que configure la aplicación para que solo procese flujos de lógica empresarial en orden secuencial, en los que se haya empleado en procesar los pasos un tiempo realista para una persona, y que no procese pasos desordenados u omitidos, pasos procesados de otro usuario ni transacciones enviadas demasiado rápido.|

## <a name="implement-rate-limiting-mechanism-to-prevent-enumeration"></a><a id="rate-enumeration"></a>Implementación de un mecanismo de limitación de velocidad para impedir la enumeración

| Título                   | Detalles      |
| ----------------------- | ------------ |
| **Componente**               | Aplicación web | 
| **Fase de SDL**               | Build |  
| **Tecnologías aplicables** | Genérico |
| **Atributos**              | N/D  |
| **Referencias**              | N/D  |
| **Pasos** | Asegúrese de que los identificadores sensibles son aleatorios. Implemente el control CAPTCHA en páginas anónimas. Asegúrese de que no se revelen datos específicos por errores y excepciones.|

## <a name="ensure-that-proper-authorization-is-in-place-and-principle-of-least-privileges-is-followed"></a><a id="principle-least-privilege"></a>Comprobación de que se aplica la autorización adecuada y se cumple el principio de privilegios mínimos

| Título                   | Detalles      |
| ----------------------- | ------------ |
| **Componente**               | Aplicación web | 
| **Fase de SDL**               | Build |  
| **Tecnologías aplicables** | Genérico |
| **Atributos**              | N/D  |
| **Referencias**              | N/D  |
| **Pasos** | <p>De acuerdo con este principio, solo se otorga a una cuenta de usuario aquellos privilegios que son fundamentales para el trabajo de ese usuario. Por ejemplo, un usuario de copia de seguridad no necesita instalar software, por lo que solo tiene privilegios para ejecutar aplicaciones de copia de seguridad y otras relacionadas con copias de seguridad. Los demás privilegios, como la instalación de software nuevo, se bloquean. El principio se aplica también a un usuario de PC que normalmente trabaja con una cuenta de usuario normal y que abre una cuenta con privilegios protegida por contraseña (es decir, un superusuario) solo cuando es imprescindible. </p><p>Este principio también puede aplicarse a las aplicaciones web. En lugar de depender exclusivamente de los métodos de autenticación basados en roles, deseamos asignar privilegios a los usuarios por medio de un sistema de autenticación mediante base de datos. Aún utilizamos sesiones para determinar si el usuario inició sesión correctamente, si bien ahora, en lugar de asignar a ese usuario un rol específico, le asignamos privilegios para comprobar qué acciones puede realizar en el sistema. Un gran punto a favor de este método es también que cada vez que hay que asignar menos privilegios a un usuario los cambios se aplicarán sobre la marcha, ya que la asignación no depende de la sesión, que tenía que expirar primero.</p>|

## <a name="business-logic-and-resource-access-authorization-decisions-should-not-be-based-on-incoming-request-parameters"></a><a id="logic-request-parameters"></a>Las decisiones sobre la autorización para acceder a recursos y lógicas empresariales no deben basarse en parámetros de solicitud entrantes

| Título                   | Detalles      |
| ----------------------- | ------------ |
| **Componente**               | Aplicación web | 
| **Fase de SDL**               | Build |  
| **Tecnologías aplicables** | Genérico |
| **Atributos**              | N/D  |
| **Referencias**              | N/D  |
| **Pasos** | Al comprobar si un usuario tiene restricciones para revisar determinados datos, las restricciones de acceso deben procesarse en el servidor. El identificador de usuario se debe almacenar dentro de una variable de sesión al iniciar sesión y debe usarse para recuperar datos de usuario de la base de datos |

### <a name="example"></a>Ejemplo
```SQL
SELECT data 
FROM personaldata 
WHERE userID=:id < - session var 
```
Ahora un posible atacante no podrá manipular ni cambiar el funcionamiento de la aplicación, ya que el identificador para recuperar los datos se administra desde el servidor.

## <a name="ensure-that-content-and-resources-are-not-enumerable-or-accessible-via-forceful-browsing"></a><a id="enumerable-browsing"></a>Comprobación de que no es posible enumerar ni acceder al contenido ni los recursos mediante navegación forzada

| Título                   | Detalles      |
| ----------------------- | ------------ |
| **Componente**               | Aplicación web | 
| **Fase de SDL**               | Build |  
| **Tecnologías aplicables** | Genérico |
| **Atributos**              | N/D  |
| **Referencias**              | N/D  |
| **Pasos** | <p>Los archivos de configuración y estáticos confidenciales no se deben mantener en la raíz web. En el caso de contenido que no sea necesario hacer público, se deben aplicar controles de acceso adecuados o eliminar el propio contenido.</p><p>Además, la navegación forzada suele emplearse junto con técnicas de fuerza bruta para recopilar información accediendo a tantas direcciones URL como sea posible para enumerar los directorios y archivos de un servidor. Los atacantes pueden buscar todas las variantes de archivos habituales. Por ejemplo, la búsqueda de un archivo de contraseña podría incluir archivos como contraseña.txt, contraseña.htm, contraseña.dat y otras variantes.</p><p>Para reducir este riesgo, deben aplicarse funciones de detección de ataques por fuerza bruta.</p>|

## <a name="ensure-that-least-privileged-accounts-are-used-to-connect-to-database-server"></a><a id="privileged-server"></a>Comprobación de que se emplean cuentas con privilegios mínimos para conectarse al servidor de base de datos

| Título                   | Detalles      |
| ----------------------- | ------------ |
| **Componente**               | Base de datos | 
| **Fase de SDL**               | Build |  
| **Tecnologías aplicables** | Genérico |
| **Atributos**              | N/D  |
| **Referencias**              | [Jerarquía de permisos de SQL](https://docs.microsoft.com/sql/relational-databases/security/permissions-hierarchy-database-engine), [Elementos protegibles de SQL](https://docs.microsoft.com/sql/relational-databases/security/securables) |
| **Pasos** | Para conectarse a la base de datos deben usarse cuentas con privilegios mínimos. El inicio de sesión de la aplicación debe estar restringido en la base de datos y solo debe ejecutar procedimientos almacenados seleccionados. El inicio de sesión de la aplicación no debe tener acceso directo a tablas. |

## <a name="implement-row-level-security-rls-to-prevent-tenants-from-accessing-each-others-data"></a><a id="rls-tenants"></a>Implementación de seguridad de nivel de fila (RLS) para impedir que los inquilinos accedan a los datos de los demás

| Título                   | Detalles      |
| ----------------------- | ------------ |
| **Componente**               | Base de datos | 
| **Fase de SDL**               | Build |  
| **Tecnologías aplicables** | SQL Azure, OnPrem |
| **Atributos**              | Versión de SQL: V12, versión de SQL: MsSQL2016 |
| **Referencias**              | [Seguridad de nivel de fila (RLS) de SQL Server](https://msdn.microsoft.com/library/azure/dn765131.aspx) |
| **Pasos** | <p>La seguridad de nivel de fila permite a los clientes controlar el acceso a las filas de una tabla de base de datos en función de las características del usuario que ejecuta una consulta (por ejemplo, la pertenencia a un grupo o el contexto de ejecución).</p><p>La seguridad de nivel de fila (RLS) simplifica el diseño y la codificación de la seguridad de la aplicación. RLS permite implementar restricciones de acceso a filas de datos. Por ejemplo, garantiza que los empleados únicamente puedan acceder a aquellas filas de datos necesarios para su departamento o que un cliente solo pueda acceder a los datos relevantes para su empresa.</p><p>La lógica de la restricción de acceso está ubicada en el nivel de base de datos en lugar de estar alejado de los datos en otro nivel de aplicación. El sistema de base de datos aplica las restricciones de acceso cada vez que se intenta acceder a los datos desde cualquier nivel. Esto hace que el sistema de seguridad resulte más sólido y fiable al reducir el área expuesta del sistema de seguridad.</p><p>|

Tenga en cuenta que RLS, al ser una característica de base de datos de serie, solo es aplicable a SQL Server a partir de 2016, Azure SQL Database y SQL Managed Instance. Si no está implementada la característica RLS de serie, es necesario garantizar que el acceso a los datos se restrinja mediante vistas y procedimientos

## <a name="sysadmin-role-should-only-have-valid-necessary-users"></a><a id="sysadmin-users"></a>El rol sysadmin solo debe tener a los usuarios válidos necesarios

| Título                   | Detalles      |
| ----------------------- | ------------ |
| **Componente**               | Base de datos | 
| **Fase de SDL**               | Build |  
| **Tecnologías aplicables** | Genérico |
| **Atributos**              | N/D  |
| **Referencias**              | [Jerarquía de permisos de SQL](https://docs.microsoft.com/sql/relational-databases/security/permissions-hierarchy-database-engine), [Elementos protegibles de SQL](https://docs.microsoft.com/sql/relational-databases/security/securables) |
| **Pasos** | Los miembros del rol del servidor fijo SysAdmin deben estar muy limitados y no deben contener cuentas utilizadas por las aplicaciones.  Revise la lista de los usuarios del rol y quite todas las cuentas innecesarias.|

## <a name="connect-to-cloud-gateway-using-least-privileged-tokens"></a><a id="cloud-least-privileged"></a>Conexión a la puerta de enlace de la nube mediante tokens con privilegios mínimos

| Título                   | Detalles      |
| ----------------------- | ------------ |
| **Componente**               | Puerta de enlace de la nube de IoT | 
| **Fase de SDL**               | Implementación |  
| **Tecnologías aplicables** | Genérico |
| **Atributos**              | Elección de puerta de enlace: Azure IoT Hub |
| **Referencias**              | [Control de acceso de IOT Hub](https://azure.microsoft.com/documentation/articles/iot-hub-devguide/#Security) |
| **Pasos** | Proporcione permisos de privilegios mínimos a varios componentes que se conectan a la puerta de enlace de la nube (IoT Hub). Un ejemplo típico es el de un componente de aprovisionamiento o administración de dispositivos que usa registryread/write, mientras que el procesador de eventos (ASA) usa la directiva Service Connect. Los dispositivos individuales se conectan mediante credenciales de dispositivo.|

## <a name="use-a-send-only-permissions-sas-key-for-generating-device-tokens"></a><a id="sendonly-sas"></a>Uso de una clave SAS de permisos solo de envío para generar tokens de dispositivo

| Título                   | Detalles      |
| ----------------------- | ------------ |
| **Componente**               | Centro de eventos de Azure | 
| **Fase de SDL**               | Build |  
| **Tecnologías aplicables** | Genérico |
| **Atributos**              | N/D  |
| **Referencias**              | [Introducción al modelo de autenticación y seguridad de Event Hubs](https://azure.microsoft.com/documentation/articles/event-hubs-authentication-and-security-model-overview/) |
| **Pasos** | Se usa una clave SAS para generar tokens de dispositivo individuales. Utilice una clave SAS de permisos solo de envío al generar el token del dispositivo para un publicador determinado.|

## <a name="do-not-use-access-tokens-that-provide-direct-access-to-the-event-hub"></a><a id="access-tokens-hub"></a>No usar tokens de acceso que proporcionan acceso directo al centro de eventos

| Título                   | Detalles      |
| ----------------------- | ------------ |
| **Componente**               | Centro de eventos de Azure | 
| **Fase de SDL**               | Build |  
| **Tecnologías aplicables** | Genérico |
| **Atributos**              | N/D  |
| **Referencias**              | [Introducción al modelo de autenticación y seguridad de Event Hubs](https://azure.microsoft.com/documentation/articles/event-hubs-authentication-and-security-model-overview/) |
| **Pasos** | No se debe proporcionar un token que concede acceso directo al centro de eventos al dispositivo. El empleo de un token con privilegios mínimos para el dispositivo que proporcione acceso solo a un editor podría ayudar a identificarlo y no permitirlo si resultara ser un dispositivo malintencionado o en peligro.|

## <a name="connect-to-event-hub-using-sas-keys-that-have-the-minimum-permissions-required"></a><a id="sas-minimum-permissions"></a>Conexión al centro de eventos mediante claves SAS que tienen los permisos mínimos necesarios

| Título                   | Detalles      |
| ----------------------- | ------------ |
| **Componente**               | Centro de eventos de Azure | 
| **Fase de SDL**               | Build |  
| **Tecnologías aplicables** | Genérico |
| **Atributos**              | N/D  |
| **Referencias**              | [Introducción al modelo de autenticación y seguridad de Event Hubs](https://azure.microsoft.com/documentation/articles/event-hubs-authentication-and-security-model-overview/) |
| **Pasos** | Proporcione permisos de privilegios mínimos a diversas aplicaciones back-end que se conectan al centro de eventos. Genere claves de SAS independientes para cada aplicación de back-end y otórgueles solo los permisos necesarios (envío, recepción o administración).|

## <a name="use-resource-tokens-to-connect-to-cosmos-db-whenever-possible"></a><a id="resource-docdb"></a>Uso de tokens de recursos para conectarse a Cosmos DB siempre que sea posible

| Título                   | Detalles      |
| ----------------------- | ------------ |
| **Componente**               | Azure DocumentDB | 
| **Fase de SDL**               | Build |  
| **Tecnologías aplicables** | Genérico |
| **Atributos**              | N/D  |
| **Referencias**              | N/D  |
| **Pasos** | Un token de recurso se asocia a un recurso de permiso de Azure Cosmos DB y captura la relación entre el usuario de una base de datos y el permiso que este tiene para un determinado recurso de aplicación de Azure Cosmos DB (por ejemplo, colección o documento). Use siempre un token de recurso para acceder a Azure Cosmos DB si la administración de claves maestras y de solo lectura del cliente no es de confianza, como en el caso de una aplicación de usuario final (por ejemplo, un cliente móvil o de escritorio). Use una clave maestra o claves de solo lectura en aplicaciones back-end que pueden almacenar estas claves de forma segura.|

## <a name="enable-fine-grained-access-management-to-azure-subscription-using-rbac"></a><a id="grained-rbac"></a>Habilitación de administración avanzada de acceso a suscripción de Azure mediante RBAC

| Título                   | Detalles      |
| ----------------------- | ------------ |
| **Componente**               | Límites de confianza de Azure | 
| **Fase de SDL**               | Build |  
| **Tecnologías aplicables** | Genérico |
| **Atributos**              | N/D  |
| **Referencias**              | [Uso de asignaciones de roles para administrar el acceso a los recursos de la suscripción de Azure](https://azure.microsoft.com/documentation/articles/role-based-access-control-configure/)  |
| **Pasos** | El control de acceso basado en rol de Azure (Azure RBAC) permite realizar una administración detallada del acceso para Azure. Con RBAC, puede conceder únicamente el grado de acceso que los usuarios necesiten para realizar sus trabajos.|

## <a name="restrict-clients-access-to-cluster-operations-using-rbac"></a><a id="cluster-rbac"></a>Restricción del acceso de cliente a operaciones de clúster mediante RBAC

| Título                   | Detalles      |
| ----------------------- | ------------ |
| **Componente**               | Límites de confianza de Service Fabric | 
| **Fase de SDL**               | Implementación |  
| **Tecnologías aplicables** | Genérico |
| **Atributos**              | Entorno: Azure |
| **Referencias**              | [Control de acceso basado en roles para clientes de Service Fabric](https://azure.microsoft.com/documentation/articles/service-fabric-cluster-security-roles/) |
| **Pasos** | <p>Azure Service Fabric admite dos tipos distintos de control de acceso para los clientes que están conectados a un clúster de Service Fabric: administrador y usuario. El control de acceso permite al administrador de clústeres limitar el acceso a determinadas operaciones de clúster para distintos grupos de usuarios, lo que aumenta la seguridad del clúster.</p><p>Los administradores tienen acceso total a las capacidades de administración (incluidas las capacidades de lectura y escritura). Los usuarios, de forma predeterminada, tienen acceso de solo lectura a las capacidades de administración (por ejemplo, capacidad de consulta) y a la capacidad para resolver las aplicaciones y los servicios.</p><p>Especifique los dos roles de cliente (administrador y cliente) cuando cree el clúster con certificados independientes para cada uno.</p>|

## <a name="perform-security-modeling-and-use-field-level-security-where-required"></a><a id="modeling-field"></a>Realización de modelos de seguridad y uso de seguridad de nivel de campo cuando sea necesario

| Título                   | Detalles      |
| ----------------------- | ------------ |
| **Componente**               | Dynamics CRM | 
| **Fase de SDL**               | Build |  
| **Tecnologías aplicables** | Genérico |
| **Atributos**              | N/D  |
| **Referencias**              | N/D  |
| **Pasos** | Realice modelos de seguridad y use seguridad de nivel de campo cuando sea necesario.|

## <a name="perform-security-modeling-of-portal-accounts-keeping-in-mind-that-the-security-model-for-the-portal-differs-from-the-rest-of-crm"></a><a id="portal-security"></a>Realización de modelos de seguridad de cuentas del portal teniendo en cuenta que el modelo de seguridad para el portal es distinto al resto de CRM

| Título                   | Detalles      |
| ----------------------- | ------------ |
| **Componente**               | Portal de Dynamics CRM | 
| **Fase de SDL**               | Build |  
| **Tecnologías aplicables** | Genérico |
| **Atributos**              | N/D  |
| **Referencias**              | N/D  |
| **Pasos** | Realice modelos de seguridad de cuentas del portal teniendo en cuenta que el modelo de seguridad para el portal es distinto al resto de CRM.|

## <a name="grant-fine-grained-permission-on-a-range-of-entities-in-azure-table-storage"></a><a id="permission-entities"></a>Concesión de permiso detallado en una serie de entidades de Azure Table Storage

| Título                   | Detalles      |
| ----------------------- | ------------ |
| **Componente**               | Azure Storage | 
| **Fase de SDL**               | Build |  
| **Tecnologías aplicables** | Genérico |
| **Atributos**              | StorageType: tabla |
| **Referencias**              | [Delegación del acceso a objetos en su cuenta de almacenamiento de Azure mediante SAS](https://azure.microsoft.com/documentation/articles/storage-security-guide/#_data-plane-security) |
| **Pasos** | En determinados escenarios empresariales, es posible que sea necesario almacenar en Azure Table Storage datos confidenciales para distintas entidades. Por ejemplo, datos confidenciales pertenecientes a distintos países o regiones. En tales casos, es posible crear firmas SAS especificando los intervalos de clave de fila y partición, de forma que un usuario pueda acceder a datos específicos de un país o región determinados.| 

## <a name="enable-role-based-access-control-rbac-to-azure-storage-account-using-azure-resource-manager"></a><a id="rbac-azure-manager"></a>Habilitación de control de acceso basado en roles (RBAC) para una cuenta de almacenamiento de Azure mediante Azure Resource Manager

| Título                   | Detalles      |
| ----------------------- | ------------ |
| **Componente**               | Azure Storage | 
| **Fase de SDL**               | Build |  
| **Tecnologías aplicables** | Genérico |
| **Atributos**              | N/D  |
| **Referencias**              | [Protección de su cuenta de almacenamiento con el control de acceso basado en roles (RBAC)](https://azure.microsoft.com/documentation/articles/storage-security-guide/#management-plane-security) |
| **Pasos** | <p>Cuando se crea una nueva cuenta de almacenamiento, se selecciona un modelo de implementación clásico o de Azure Resource Manager. El modelo clásico de creación de recursos de Azure únicamente permite acceder a toda la suscripción en su conjunto o bien a nada de ella, y lo mismo ocurre con la cuenta de almacenamiento.</p><p>Con el modelo de Azure Resource Manager, coloque la cuenta de almacenamiento en un grupo de recursos y controle el acceso al plano de administración de dicha cuenta de almacenamiento concreta mediante Azure Active Directory. Por ejemplo, puede proporcionar a usuarios específicos la posibilidad de tener acceso a las claves de cuenta de almacenamiento, mientras que otros usuarios pueden ver información sobre la cuenta de almacenamiento pero no pueden tener acceso a sus claves.</p>|

## <a name="implement-implicit-jailbreak-or-rooting-detection"></a><a id="rooting-detection"></a>Implementación de detección implícita de jailbreak o rooting

| Título                   | Detalles      |
| ----------------------- | ------------ |
| **Componente**               | Cliente móvil | 
| **Fase de SDL**               | Build |  
| **Tecnologías aplicables** | Genérico |
| **Atributos**              | N/D  |
| **Referencias**              | N/D  |
| **Pasos** | <p>La aplicación debe proteger sus propios datos de configuración y de usuario en caso de rooting o jailbreak del teléfono. El rooting o jailbreak implican un acceso no autorizado, que los usuarios normales no realizarían en su propio teléfono. Por lo tanto, la aplicación debe contar con una lógica de detección implícita al iniciarse, a fin de detectar si el teléfono se ha desbloqueado mediante rooting.</p><p>La lógica de detección puede consistir simplemente en acceder a archivos a los que solo el usuario raíz puede acceder en condiciones normales, como los siguientes:</p><ul><li>/system/app/Superuser.apk</li><li>/sbin/su</li><li>/system/bin/su</li><li>/system/xbin/su</li><li>/data/local/xbin/su</li><li>/data/local/bin/su</li><li>/system/sd/xbin/su</li><li>/system/bin/failsafe/su</li><li>/data/local/su</li></ul><p>Si la aplicación puede acceder a cualquiera de estos archivos, esta se está ejecutando como usuario raíz.</p>|

## <a name="weak-class-reference-in-wcf"></a><a id="weak-class-wcf"></a>Referencia débil de clase en WCF

| Título                   | Detalles      |
| ----------------------- | ------------ |
| **Componente**               | WCF | 
| **Fase de SDL**               | Build |  
| **Tecnologías aplicables** | Genérico, .NET Framework 3 |
| **Atributos**              | N/D  |
| **Referencias**              | [MSDN](https://msdn.microsoft.com/library/ff648500.aspx), [Fortify Kingdom](https://vulncat.fortify.com/en/detail?id=desc.config.dotnet.wcf_misconfiguration_weak_class_reference) |
| **Pasos** | <p>El sistema utiliza una referencia débil de clase, que podría permitir que un atacante ejecute código no autorizado. El programa hace referencia a una clase definida por el usuario que no se identifica de forma exclusiva. Cuando .NET carga esta clase débilmente identificada, el cargador de tipo CLR busca la clase en las ubicaciones siguientes en el orden especificado:</p><ol><li>Si se conoce el ensamblado del tipo, el cargador busca en las ubicaciones de redirección del archivo de configuración, la caché global de ensamblados, el ensamblado actual que utiliza información de configuración y el directorio base de la aplicación.</li><li>Si el ensamblado es desconocido, el cargador busca en el ensamblado actual, mscorlib y la ubicación devuelta por el controlador de eventos TypeResolve.</li><li>El orden de la búsqueda de CLR se puede modificar con enlaces, como el mecanismo de reenvío de tipos y el evento AppDomain.TypeResolve.</li></ol><p>Si un atacante aprovecha el orden de la búsqueda de CLR al crear una clase alternativa con el mismo nombre y colocarla en una ubicación alternativa que CLR cargará en primer lugar, CLR ejecutará involuntariamente el código proporcionado por el atacante.</p>|

### <a name="example"></a>Ejemplo
El elemento `<behaviorExtensions/>` del archivo de configuración de WCF siguiente indica a WCF que añada una clase de comportamiento personalizada a una extensión de WCF determinada.
```
<system.serviceModel>
    <extensions>
        <behaviorExtensions>
            <add name=""myBehavior"" type=""MyBehavior"" />
        </behaviorExtensions>
    </extensions>
</system.serviceModel>
```
Al utilizar nombres completos (seguros), se identifica exclusivamente a un tipo, lo que mejora la seguridad del sistema. Utilice nombres de ensamblado completos al registrar tipos en los archivos machine.config y app.config.

### <a name="example"></a>Ejemplo
El elemento `<behaviorExtensions/>` del archivo de configuración de WCF siguiente indica a WCF que añada una clase de comportamiento personalizada con una referencia segura a una extensión de WCF determinada.
```
<system.serviceModel>
    <extensions>
        <behaviorExtensions>
            <add name=""myBehavior"" type=""Microsoft.ServiceModel.Samples.MyBehaviorSection, MyBehavior,
            Version=1.0.0.0, Culture=neutral, PublicKeyToken=null"" />
        </behaviorExtensions>
    </extensions>
</system.serviceModel>
```

## <a name="wcf-implement-authorization-control"></a><a id="wcf-authz"></a>WCF: implementación de control de autorización

| Título                   | Detalles      |
| ----------------------- | ------------ |
| **Componente**               | WCF | 
| **Fase de SDL**               | Build |  
| **Tecnologías aplicables** | Genérico, .NET Framework 3 |
| **Atributos**              | N/D  |
| **Referencias**              | [MSDN](https://msdn.microsoft.com/library/ff648500.aspx), [Fortify Kingdom](https://vulncat.fortify.com/en/detail?id=desc.config.dotnet.wcf_misconfiguration_weak_class_reference) |
| **Pasos** | <p>Este servicio no usa un control de autorización. Cuando un cliente llama a un determinado servicio de WCF, WCF proporciona varios esquemas de autorización que comprueban que el llamador tiene permiso para ejecutar el método de servicio en el servidor. Si los controles de autorización no están habilitados para los servicios de WCF, un usuario autenticado puede lograr una elevación de privilegios.</p>|

### <a name="example"></a>Ejemplo
La siguiente configuración indica a WCF que no compruebe el nivel de autorización del cliente cuando se ejecuta el servicio:
```
<behaviors>
    <serviceBehaviors>
        <behavior>
            ...
            <serviceAuthorization principalPermissionMode=""None"" />
        </behavior>
    </serviceBehaviors>
</behaviors>
```
Utilice un esquema de autorización de servicio para comprobar que el llamador del método de servicio está autorizado para hacerlo. WCF ofrece dos modos y permite la definición de un esquema de autorización personalizado. El modo UseWindowsGroups utiliza roles y usuarios de Windows, mientras que el modo UseAspNetRoles usa un proveedor de roles ASP.NET, como SQL Server, para autenticar.

### <a name="example"></a>Ejemplo
La siguiente configuración indica a WCF que confirme que el cliente forma parte del grupo de administradores antes de ejecutar el servicio de adición:
```
<behaviors>
    <serviceBehaviors>
        <behavior>
            ...
            <serviceAuthorization principalPermissionMode=""UseWindowsGroups"" />
        </behavior>
    </serviceBehaviors>
</behaviors>
```
A continuación, el servicio se declara de la manera siguiente:
```
[PrincipalPermission(SecurityAction.Demand,
Role = ""Builtin\\Administrators"")]
public double Add(double n1, double n2)
{
double result = n1 + n2;
return result;
}
```

## <a name="implement-proper-authorization-mechanism-in-aspnet-web-api"></a><a id="authz-aspnet"></a>Implementación del mecanismo de autorización adecuado en ASP.NET Web API

| Título                   | Detalles      |
| ----------------------- | ------------ |
| **Componente**               | API Web | 
| **Fase de SDL**               | Build |  
| **Tecnologías aplicables** | Genérico, MVC5 |
| **Atributos**              | N/D; Proveedor de identidades; ADFS, Proveedor de identidades; Azure AD |
| **Referencias**              | [Autenticación y autorización en ASP.NET Web API](https://www.asp.net/web-api/overview/security/authentication-and-authorization-in-aspnet-web-api) |
| **Pasos** | <p>Es posible obtener información de roles de los usuarios de la aplicación a partir de notificaciones ADFS o Azure AD si la aplicación los utiliza como proveedores de identidades o si la propia aplicación las proporciona. En cualquiera de estos casos, la implementación de la autorización personalizada debe validar la información de roles de usuario.</p><p>Es posible obtener información de roles de los usuarios de la aplicación a partir de notificaciones ADFS o Azure AD si la aplicación los utiliza como proveedores de identidades o si la propia aplicación las proporciona. En cualquiera de estos casos, la implementación de la autorización personalizada debe validar la información de roles de usuario.</p>

### <a name="example"></a>Ejemplo
```csharp
[AttributeUsage(AttributeTargets.Class | AttributeTargets.Method, Inherited = true, AllowMultiple = true)]
public class ApiAuthorizeAttribute : System.Web.Http.AuthorizeAttribute
{
        public async override Task OnAuthorizationAsync(HttpActionContext actionContext, CancellationToken cancellationToken)
        {
            if (actionContext == null)
            {
                throw new Exception();
            }

            if (!string.IsNullOrEmpty(base.Roles))
            {
                bool isAuthorized = ValidateRoles(actionContext);
                if (!isAuthorized)
                {
                    HandleUnauthorizedRequest(actionContext);
                }
            }

            base.OnAuthorization(actionContext);
        }

public bool ValidateRoles(actionContext)
{
   //Authorization logic here; returns true or false
}

}
```
Todos los controladores y métodos de acción que es necesario proteger deben contener el atributo anterior.
```csharp
[ApiAuthorize]
public class CustomController : ApiController
{
     //Application code goes here
}
```

## <a name="perform-authorization-checks-in-the-device-if-it-supports-various-actions-that-require-different-permission-levels"></a><a id="device-permission"></a>Realización de comprobaciones de autorización en el dispositivo si admite varias acciones que requieren distintos niveles de permiso

| Título                   | Detalles      |
| ----------------------- | ------------ |
| **Componente**               | Dispositivo IoT | 
| **Fase de SDL**               | Build |  
| **Tecnologías aplicables** | Genérico |
| **Atributos**              | N/D  |
| **Referencias**              | N/D  |
| **Pasos** | <p>El dispositivo debe autorizar al llamador para comprobar si el llamador tiene los permisos necesarios para realizar la acción solicitada. Por ejemplo, supongamos que el dispositivo es una cerradura de puerta inteligente que puede supervisarse desde la nube, además de proporcionar funcionalidades como el bloqueo remoto de la puerta.</p><p>La cerradura de puerta inteligente ofrece una funcionalidad de desbloqueo solo cuando alguien se acerca físicamente a la puerta con una tarjeta. En este caso, la implementación del control y el comando remoto debe realizarse de manera que no proporcione ninguna funcionalidad para desbloquear la puerta, ya que la puerta de enlace de la nube no tiene autorización para enviar un comando para desbloquear la puerta.</p>|

## <a name="perform-authorization-checks-in-the-field-gateway-if-it-supports-various-actions-that-require-different-permission-levels"></a><a id="field-permission"></a>Realización de comprobaciones de autorización en la puerta de enlace de campo si admite varias acciones que requieren distintos niveles de permiso

| Título                   | Detalles      |
| ----------------------- | ------------ |
| **Componente**               | Puerta de enlace de campo de IoT | 
| **Fase de SDL**               | Build |  
| **Tecnologías aplicables** | Genérico |
| **Atributos**              | N/D  |
| **Referencias**              | N/D  |
| **Pasos** | La puerta de enlace de campo debe autorizar al llamador para comprobar si el llamador tiene los permisos necesarios para realizar la acción solicitada. Por ejemplo, debe haber diferentes permisos para una interfaz o API de usuario administrador que se usen para configurar una puerta de enlace de campo frente a los dispositivos que se conectan a ella.|
