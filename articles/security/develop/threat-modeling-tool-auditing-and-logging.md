---
title: 'Auditoría y registro: Microsoft Threat Modeling Tool: Azure | Microsoft Docs'
description: Más información sobre la mitigación de auditoría y registro en Threat Modeling Tool. Consulte la información de mitigación y los ejemplos de código.
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
ms.openlocfilehash: bac17073650736df9ec48e951290852e082e9417
ms.sourcegitcommit: 8def3249f2c216d7b9d96b154eb096640221b6b9
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/03/2020
ms.locfileid: "87543000"
---
# <a name="security-frame-auditing-and-logging--mitigations"></a>Marco de seguridad: Auditoría y registro | Mitigaciones 

| Producto o servicio | Artículo |
| --------------- | ------- |
| **Dynamics CRM**    | <ul><li>[Identificación de las entidades confidenciales de la solución e implementación de la auditoría de cambios](#sensitive-entities)</li></ul> |
| **Aplicación web** | <ul><li>[Comprobación de que la auditoría y el registro se aplican en la aplicación](#auditing)</li><li>[Comprobación de que la rotación de registros y la separación están en funcionamiento](#log-rotation)</li><li>[Comprobación de que la aplicación no registra datos confidenciales del usuario](#log-sensitive-data)</li><li>[Comprobación de que los archivos de registro y auditoría tienen acceso restringido](#log-restricted-access)</li><li>[Comprobación de que se registran los eventos de administración de usuario](#user-management)</li><li>[Comprobación de que el sistema tiene defensas integradas contra usos indebidos](#inbuilt-defenses)</li><li>[Habilitación del registro de diagnóstico para aplicaciones web en Azure App Service](#diagnostics-logging)</li></ul> |
| **Base de datos** | <ul><li>[Comprobación de que está habilitada la auditoría de inicio de sesión en SQL Server](#identify-sensitive-entities)</li><li>[Habilitamiento de la detección de amenazas en Azure SQL](#threat-detection)</li></ul> |
| **Almacenamiento de Azure** | <ul><li>Uso de [Análisis de Azure Storage para auditar el acceso de Azure Storage](#analytics)</li></ul> |
| **WCF** | <ul><li>[Implementación de suficientes registros](#sufficient-logging)</li><li>[Implementación de un control suficiente de errores de auditoría](#audit-failure-handling)</li></ul> |
| **API web** | <ul><li>[Comprobación de que la auditoría y el registro se aplican en Web API](#logging-web-api)</li></ul> |
| **Puerta de enlace de campo de IoT** | <ul><li>[Comprobación de que se aplican la auditoría y registro adecuados en la puerta de enlace de campo](#logging-field-gateway)</li></ul> |
| **Puerta de enlace de nube de IoT** | <ul><li>[Comprobación de que se aplican la auditoría y registro adecuados en la puerta de enlace de nube](#logging-cloud-gateway)</li></ul> |

## <a name="identify-sensitive-entities-in-your-solution-and-implement-change-auditing"></a><a id="sensitive-entities"></a>Identificación de las entidades confidenciales de la solución e implementación de la auditoría de cambios

| Título                   | Detalles      |
| ----------------------- | ------------ |
| **Componente**               | Dynamics CRM | 
| **Fase de SDL**               | Build |  
| **Tecnologías aplicables** | Genérico |
| **Atributos**              | N/D  |
| **Referencias**              | N/D  |
| **Pasos**                   | Identificación de las entidades de la solución que contienen datos confidenciales e implementación de la auditoría de cambios en esas entidades y campos |

## <a name="ensure-that-auditing-and-logging-is-enforced-on-the-application"></a><a id="auditing"></a>Comprobación de que la auditoría y el registro se aplican en la aplicación

| Título                   | Detalles      |
| ----------------------- | ------------ |
| **Componente**               | Aplicación web | 
| **Fase de SDL**               | Build |  
| **Tecnologías aplicables** | Genérico |
| **Atributos**              | N/D  |
| **Referencias**              | N/D  |
| **Pasos**                   | Habilite la auditoría y el registro en todos los componentes. Los registros de auditoría deben reflejar el contexto de usuario. Identifique todos los eventos importantes y regístrelos. Implemente el registro centralizado. |

## <a name="ensure-that-log-rotation-and-separation-are-in-place"></a><a id="log-rotation"></a>Comprobación de que la rotación de registros y la separación están en funcionamiento

| Título                   | Detalles      |
| ----------------------- | ------------ |
| **Componente**               | Aplicación web | 
| **Fase de SDL**               | Build |  
| **Tecnologías aplicables** | Genérico |
| **Atributos**              | N/D  |
| **Referencias**              | N/D  |
| **Pasos**                   | <p>La rotación de registros es un proceso automatizado que se usa en la administración del sistema en el que se archivan los archivos de registro con fecha. Los servidores que ejecutan aplicaciones grandes a menudo registran todas las solicitudes: en el caso de registros voluminosos, la rotación de registros es una forma de limitar el tamaño total de estos al tiempo que se permite el análisis de eventos recientes. </p><p>La separación de registros significa básicamente que se deben almacenar los archivos de registro en una partición diferente a aquella en la que se ejecuta la aplicación o el SO con el fin de evitar un ataque de denegación de servicio o la disminución del rendimiento de la aplicación</p>|

## <a name="ensure-that-the-application-does-not-log-sensitive-user-data"></a><a id="log-sensitive-data"></a>Comprobación de que la aplicación no registra datos confidenciales del usuario

| Título                   | Detalles      |
| ----------------------- | ------------ |
| **Componente**               | Aplicación web | 
| **Fase de SDL**               | Build |  
| **Tecnologías aplicables** | Genérico |
| **Atributos**              | N/D  |
| **Referencias**              | N/D  |
| **Pasos**                   | <p>Compruebe que no se registra ningún dato confidencial que un usuario envía a su sitio. Compruebe para ver si hay algún registro intencionado, o efectos secundarios causados por problemas de diseño. Ejemplos de datos confidenciales incluyen:</p><ul><li>Credenciales de usuario</li><li>Número del seguro social u otra información de identificación</li><li>Números de tarjeta de crédito u otra información financiera</li><li>Información relacionada con la salud</li><li>Claves privadas u otros datos que se pudieron usar para descifrar información cifrada</li><li>Información del sistema o de la aplicación que se puede usar para atacar la aplicación de forma más eficaz</li></ul>|

## <a name="ensure-that-audit-and-log-files-have-restricted-access"></a><a id="log-restricted-access"></a>Comprobación de que los archivos de registro y auditoría tienen acceso restringido

| Título                   | Detalles      |
| ----------------------- | ------------ |
| **Componente**               | Aplicación web | 
| **Fase de SDL**               | Build |  
| **Tecnologías aplicables** | Genérico |
| **Atributos**              | N/D  |
| **Referencias**              | N/D  |
| **Pasos**                   | <p>Compruebe que los derechos de acceso a los archivos de registro se han establecido correctamente. Las cuentas de la aplicación deben tener operadores y acceso de solo escritura y el personal de soporte técnico debe tener acceso de solo lectura según sea necesario.</p><p>Las cuentas de los administradores son las únicas cuentas que deben tener un acceso total. Compruebe la ACL de Windows en los archivos de registro para garantizar que estén correctamente restringidos:</p><ul><li>Las cuentas de la aplicación deben tener acceso de solo escritura</li><li>Los operadores y el personal de soporte técnico deben tener acceso de solo lectura según sea necesario</li><li>Los administradores tienen las únicas cuentas que deben tener acceso total</li></ul>|

## <a name="ensure-that-user-management-events-are-logged"></a><a id="user-management"></a>Comprobación de que se registran los eventos de administración de usuario

| Título                   | Detalles      |
| ----------------------- | ------------ |
| **Componente**               | Aplicación web | 
| **Fase de SDL**               | Build |  
| **Tecnologías aplicables** | Genérico |
| **Atributos**              | N/D  |
| **Referencias**              | N/D  |
| **Pasos**                   | <p>Compruebe que la aplicación supervisa los eventos de administración de usuario como los inicios de sesión de usuario correctos e incorrectos, el restablecimiento de contraseñas, los cambios de contraseña, el bloqueo de cuenta y el registro de usuario. Si hace esto ayuda a detectar y a reaccionar ante comportamientos potencialmente sospechosos. También permite recopilar datos de las operaciones. Por ejemplo, para realizar un seguimiento de quién tiene acceso a la aplicación</p>|

## <a name="ensure-that-the-system-has-inbuilt-defenses-against-misuse"></a><a id="inbuilt-defenses"></a>Comprobación de que el sistema tiene defensas integradas contra usos indebidos

| Título                   | Detalles      |
| ----------------------- | ------------ |
| **Componente**               | Aplicación web | 
| **Fase de SDL**               | Build |  
| **Tecnologías aplicables** | Genérico |
| **Atributos**              | N/D  |
| **Referencias**              | N/D  |
| **Pasos**                   | <p>Los controles deben funcionar de forma que generen excepciones de seguridad en caso de uso incorrecto de una aplicación. Por ejemplo, si la validación de entrada está en funcionamiento y un atacante intenta insertar código malicioso que no coincide con la expresión regular, se generará una excepción de seguridad que puede ser un indicativo de uso indebido del sistema</p><p>Por ejemplo, se recomienda registrar las excepciones de seguridad y realizar acciones para los siguientes problemas:</p><ul><li>Validación de entradas</li><li>Infracciones de CSRF</li><li>Fuerza bruta (límite superior en relación con el número de solicitudes por usuario y recurso)</li><li>Infracciones de carga de archivos</li><ul>|

## <a name="enable-diagnostics-logging-for-web-apps-in-azure-app-service"></a><a id="diagnostics-logging"></a>Habilitación del registro de diagnóstico para aplicaciones web en Azure App Service

| Título                   | Detalles      |
| ----------------------- | ------------ |
| **Componente**               | Aplicación web | 
| **Fase de SDL**               | Build |  
| **Tecnologías aplicables** | Genérico |
| **Atributos**              | EnvironmentType: Azure |
| **Referencias**              | N/D  |
| **Pasos** | <p>Azure integra diagnósticos para ayudar a depurar Aplicaciones web de App Service. También se aplica a aplicaciones móviles y aplicaciones de API. Aplicaciones web de App Service ofrece la funcionalidad de diagnóstico para registrar información del servidor web y de la aplicación web.</p><p>De forma lógica, estos diagnósticos se dividen en diagnósticos del servidor web y diagnósticos de aplicaciones.</p>|

## <a name="ensure-that-login-auditing-is-enabled-on-sql-server"></a><a id="identify-sensitive-entities"></a>Comprobación de que está habilitada la auditoría de inicio de sesión en SQL Server

| Título                   | Detalles      |
| ----------------------- | ------------ |
| **Componente**               | Base de datos | 
| **Fase de SDL**               | Build |  
| **Tecnologías aplicables** | Genérico |
| **Atributos**              | N/D  |
| **Referencias**              | [Configuración de la auditoría de inicio de sesión](https://msdn.microsoft.com/library/ms175850.aspx) |
| **Pasos** | <p>La auditoría de inicio de sesión del servidor de base de datos debe estar habilitada para detectar y confirmar ataques para averiguar la contraseña. Es importante capturar los intentos de inicio de sesión incorrectos. Capturar los intentos de inicio de sesión correctos e incorrectos proporciona una ventaja adicional durante las investigaciones</p>|

## <a name="enable-threat-detection-on-azure-sql"></a><a id="threat-detection"></a>Habilitamiento de la detección de amenazas en Azure SQL

| Título                   | Detalles      |
| ----------------------- | ------------ |
| **Componente**               | Base de datos | 
| **Fase de SDL**               | Build |  
| **Tecnologías aplicables** | SQL Azure |
| **Atributos**              | Versión de SQL: V12 |
| **Referencias**              | [Introducción a Detección de amenazas de SQL Database](https://azure.microsoft.com/documentation/articles/sql-database-threat-detection-get-started/)|
| **Pasos** |<p>Detección de amenazas detecta actividades anómalas en la base de datos que indican posibles amenazas de seguridad a la base de datos. La detección de amenazas ofrece un nuevo nivel de seguridad, que permite a los clientes detectar amenazas potenciales y responder a ellas a medida que se producen, gracias a las alertas de seguridad sobre actividades anómalas que se proporcionan.</p><p>Los usuarios pueden explorar los eventos sospechosos mediante la auditoría de Azure SQL Database para determinar si proceden de un intento de acceder a los datos en la base de datos, infringir su seguridad o aprovecharlos.</p><p>Detección de amenazas facilita la solución de las posibles amenazas a la base de datos sin necesidad de ser un experto en seguridad ni administrar sistemas de supervisión de seguridad avanzada.</p>|

## <a name="use-azure-storage-analytics-to-audit-access-of-azure-storage"></a><a id="analytics"></a>Uso de Análisis de Azure Storage para auditar el acceso de Azure Storage

| Título                   | Detalles      |
| ----------------------- | ------------ |
| **Componente**               | Azure Storage | 
| **Fase de SDL**               | Implementación |  
| **Tecnologías aplicables** | Genérico |
| **Atributos**              | N/D |
| **Referencias**              | [Uso de Análisis de Storage para supervisar el tipo de autorización](https://azure.microsoft.com/documentation/articles/storage-security-guide/#storage-analytics) |
| **Pasos** | <p>Para cada cuenta de almacenamiento, se puede habilitar el Análisis de Azure Storage para realizar el registro y almacenar datos de métricas. Los registros de análisis de almacenamiento proporcionan información importante como, por ejemplo, el método de autenticación utilizado por un usuario al acceder al almacenamiento.</p><p>Esto puede resultar muy útil si se está esforzando por proteger el acceso al almacenamiento. Por ejemplo, en Blob Storage puede configurar todos los contenedores para que sean privados e implementar el uso de un servicio de Firma de acceso compartido en sus aplicaciones. Luego puede comprobar los registros periódicamente para ver si se ha accedido a los blobs mediante las claves de la cuenta de almacenamiento, lo que podría indicar una infracción de seguridad, o si los blobs son públicos pero no deberían serlo.</p>|

## <a name="implement-sufficient-logging"></a><a id="sufficient-logging"></a>Implementación de suficientes registros

| Título                   | Detalles      |
| ----------------------- | ------------ |
| **Componente**               | WCF | 
| **Fase de SDL**               | Build |  
| **Tecnologías aplicables** | .NET Framework |
| **Atributos**              | N/D  |
| **Referencias**              | [MSDN](https://msdn.microsoft.com/library/ff648500.aspx), [Fortify Kingdom](https://vulncat.fortify.com/en/detail?id=desc.config.dotnet.wcf_misconfiguration_insufficient_logging) |
| **Pasos** | <p>La falta de una traza de auditoría después de un incidente de seguridad, puede dificultar los esfuerzos de investigación. Windows Communication Foundation (WCF) ofrece la capacidad de registrar los intentos de autenticación correctos e incorrectos.</p><p>Si registra los intentos de autenticación incorrectos puede advertir a los administradores de posibles ataques de fuerza bruta. Del mismo modo, el registro de eventos de autenticación correctos puede proporcionar una traza de auditoría útil cuando está en peligro una cuenta legítima. Habilite la característica de auditoría de seguridad del servicio de WCF. |

### <a name="example"></a>Ejemplo
El siguiente es un ejemplo de configuración con la auditoría habilitada
```
<system.serviceModel>
    <behaviors>
        <serviceBehaviors>
            <behavior name=""NewBehavior"">
                <serviceSecurityAudit auditLogLocation=""Default""
                suppressAuditFailure=""false"" 
                serviceAuthorizationAuditLevel=""SuccessAndFailure""
                messageAuthenticationAuditLevel=""SuccessAndFailure"" />
                ...
            </behavior>
        </servicebehaviors>
    </behaviors>
</system.serviceModel>
```

## <a name="implement-sufficient-audit-failure-handling"></a><a id="audit-failure-handling"></a>Implementación de un control suficiente de errores de auditoría

| Título                   | Detalles      |
| ----------------------- | ------------ |
| **Componente**               | WCF | 
| **Fase de SDL**               | Build |  
| **Tecnologías aplicables** | .NET Framework |
| **Atributos**              | N/D  |
| **Referencias**              | [MSDN](https://msdn.microsoft.com/library/ff648500.aspx), [Fortify Kingdom](https://vulncat.fortify.com/en/detail?id=desc.config.dotnet.wcf_misconfiguration_insufficient_audit_failure_handling) |
| **Pasos** | <p>La solución desarrollada se ha configurado para que no genere una excepción cuando se produce un error al escribir en un registro de auditoría. Si WCF está configurado para que no genere una excepción cuando no puede escribir en un registro de auditoría, el programa no recibirá la notificación del error y puede que no se produzca la auditoría de eventos de seguridad críticos.</p>|

### <a name="example"></a>Ejemplo
El elemento `<behavior/>` del archivo de configuración de WCF que aparece a continuación, indica a WCF que no notifique a la aplicación cuando WCF no pueda escribir en un registro de auditoría.
```
<behaviors>
    <serviceBehaviors>
        <behavior name="NewBehavior">
            <serviceSecurityAudit auditLogLocation="Application"
            suppressAuditFailure="true"
            serviceAuthorizationAuditLevel="Success"
            messageAuthenticationAuditLevel="Success" />
        </behavior>
    </serviceBehaviors>
</behaviors>
```
Configure WCF para que informe al programa siempre que no pueda escribir en un registro de auditoría. El programa debe tener un esquema de notificación alternativo en funcionamiento para alertar a la organización de que no se están conservando las trazas de auditoría. 

## <a name="ensure-that-auditing-and-logging-is-enforced-on-web-api"></a><a id="logging-web-api"></a>Comprobación de que la auditoría y el registro se aplican en Web API

| Título                   | Detalles      |
| ----------------------- | ------------ |
| **Componente**               | API Web | 
| **Fase de SDL**               | Build |  
| **Tecnologías aplicables** | Genérico |
| **Atributos**              | N/D  |
| **Referencias**              | N/D  |
| **Pasos** | Habilite la auditoría y el registro en las Web API. Los registros de auditoría deben reflejar el contexto de usuario. Identifique todos los eventos importantes y regístrelos. Implemente el registro centralizado. |

## <a name="ensure-that-appropriate-auditing-and-logging-is-enforced-on-field-gateway"></a><a id="logging-field-gateway"></a>Comprobación de que se aplican la auditoría y registro adecuados en la puerta de enlace de campo

| Título                   | Detalles      |
| ----------------------- | ------------ |
| **Componente**               | Puerta de enlace de campo de IoT | 
| **Fase de SDL**               | Build |  
| **Tecnologías aplicables** | Genérico |
| **Atributos**              | N/D  |
| **Referencias**              | N/D  |
| **Pasos** | <p>Cuando varios dispositivos se conectan a una puerta de enlace de campo, asegúrese de que los intentos de conexión y el estado de autenticación (correcto o incorrecto) de los dispositivos individuales se registra y conserva en la puerta de enlace de campo.</p><p>Además, en los casos en los que la puerta de enlace de campo conserva las credenciales de IoT Hub de los dispositivos individuales, asegúrese de que la auditoría se realiza cuando se recuperan estas credenciales. Desarrolle un proceso para cargar periódicamente los registros en Azure IoT Hub o Storage para su retención a largo plazo.</p> |

## <a name="ensure-that-appropriate-auditing-and-logging-is-enforced-on-cloud-gateway"></a><a id="logging-cloud-gateway"></a>Comprobación de que se aplican la auditoría y registro adecuados en la puerta de enlace de nube

| Título                   | Detalles      |
| ----------------------- | ------------ |
| **Componente**               | Puerta de enlace de la nube de IoT | 
| **Fase de SDL**               | Build |  
| **Tecnologías aplicables** | Genérico |
| **Atributos**              | N/D  |
| **Referencias**              | [Introducción a la supervisión de operaciones de IoT Hub](https://azure.microsoft.com/documentation/articles/iot-hub-operations-monitoring/) |
| **Pasos** | <p>Diseño para recopilar y almacenar datos de auditoría recopilados mediante la supervisión de operaciones de IoT Hub. Habilite las siguientes categorías de supervisión:</p><ul><li>Operaciones de identidad de dispositivos</li><li>Comunicaciones de dispositivo a nube</li><li>Comunicaciones de nube a dispositivo</li><li>Conexiones</li><li>Cargas de archivos</li></ul>|