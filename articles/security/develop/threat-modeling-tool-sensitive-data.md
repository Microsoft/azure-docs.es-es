---
title: 'Información confidencial: Microsoft Threat Modeling Tool (Azure) | Microsoft Docs'
description: Más información sobre la mitigación de datos confidenciales en Threat Modeling Tool. Consulte la información de mitigación y los ejemplos de código.
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
ms.custom: devx-track-javascript
ms.openlocfilehash: 8f8b18d36453ac65300a5dd19fa7e07b1449bc28
ms.sourcegitcommit: 8def3249f2c216d7b9d96b154eb096640221b6b9
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/03/2020
ms.locfileid: "87538955"
---
# <a name="security-frame-sensitive-data--mitigations"></a>Marco de seguridad: información confidencial | Mitigaciones 
| Producto o servicio | Artículo |
| --------------- | ------- |
| **Límite de confianza de la máquina** | <ul><li>[Asegúrese de que los archivos binarios estén ofuscados si contienen información confidencial](#binaries-info)</li><li>[Considere la posibilidad de usar el sistema de cifrado de archivos (EFS) para proteger información confidencial específica del usuario](#efs-user)</li><li>[Asegúrese de que se cifre la información confidencial almacenada por la aplicación en el sistema de archivos](#filesystem)</li></ul> | 
| **Aplicación web** | <ul><li>[Asegúrese de que no se almacene contenido confidencial en la memoria caché del explorador](#cache-browser)</li><li>[Cifre las secciones de los archivos de configuración de la aplicación web que contengan información confidencial](#encrypt-data)</li><li>[Deshabilite explícitamente el atributo HTML autocomplete en formularios y entradas confidenciales](#autocomplete-input)</li><li>[Asegúrese de que la información confidencial que se muestra en la pantalla del usuario esté enmascarada](#data-mask)</li></ul> | 
| **Base de datos** | <ul><li>[Implemente el enmascaramiento dinámico de datos para limitar la exposición de información confidencial a usuarios sin privilegios](#dynamic-users)</li><li>[Asegúrese de que las contraseñas se almacenen en formato de hash con sal](#salted-hash)</li><li>[Asegúrese de que la información confidencial en las columnas de la base de datos esté cifrada](#db-encrypted)</li><li>[Asegúrese de que el cifrado de base de datos (TDE) esté habilitado](#tde-enabled)</li><li>[Asegúrese de que las copias de seguridad de la base de datos estén cifradas](#backup)</li></ul> | 
| **API web** | <ul><li>[Asegúrese de que la información confidencial relevante para la API web no se guarde en el almacenamiento del explorador](#api-browser)</li></ul> | 
| Azure DocumentDB | <ul><li>[Cifre la información confidencial almacenada en Azure Cosmos DB](#encrypt-docdb)</li></ul> | 
| **Límites de confianza de VM de IaaS de Azure** | <ul><li>[Use Azure Disk Encryption para cifrar discos usados por máquinas virtuales](#disk-vm)</li></ul> | 
| **Límites de confianza de Service Fabric** | <ul><li>[Cifre los secretos en aplicaciones de Service Fabric](#fabric-apps)</li></ul> | 
| **Dynamics CRM** | <ul><li>[Realice el modelado de seguridad y use unidades de negocio y equipos cuando sea necesario](#modeling-teams)</li><li>[Minimice el acceso a la característica para compartir en entidades críticas](#entities)</li><li>[Entrene a los usuarios sobre los riesgos asociados con la característica Compartir de Dynamics CRM y los procedimientos recomendados de seguridad](#good-practices)</li><li>[Incluya una regla de estándares de desarrollo que prohíba mostrar detalles de configuración en la administración de excepciones](#exception-mgmt)</li></ul> | 
| **Almacenamiento de Azure** | <ul><li>[Use el cifrado del servicio Azure Storage (SSE) para datos en reposo (versión preliminar)](#sse-preview)</li><li>[Use el cifrado en el cliente para almacenar información confidencial en Azure Storage](#client-storage)</li></ul> | 
| **Cliente para dispositivos móviles** | <ul><li>[Cifre la información confidencial o de identificación personal guardada en el almacenamiento local de teléfonos](#pii-phones)</li><li>[Ofusque los archivos binarios generados antes de distribuirlos a los usuarios finales](#binaries-end)</li></ul> | 
| **WCF** | <ul><li>[Establezca clientCredentialType en Certificate o Windows](#cert)</li><li>[El modo de seguridad de WCF no está habilitado](#security)</li></ul> | 

## <a name="ensure-that-binaries-are-obfuscated-if-they-contain-sensitive-information"></a><a id="binaries-info"></a>Asegúrese de que los archivos binarios estén ofuscados si contienen información confidencial

| Título                   | Detalles      |
| ----------------------- | ------------ |
| **Componente**               | Límite de confianza de la máquina | 
| **Fase de SDL**               | Implementación |  
| **Tecnologías aplicables** | Genérico |
| **Atributos**              | N/D  |
| **Referencias**              | N/D  |
| **Pasos** | Asegúrese de que los archivos binarios estén ofuscados si contienen información confidencial como secretos comerciales o lógica de negocios confidencial que no se deba revelar. Esto sirve para detener la ingeniería inversa de ensamblados. Para este fin se pueden usar herramientas como `CryptoObfuscator`. |

## <a name="consider-using-encrypted-file-system-efs-is-used-to-protect-confidential-user-specific-data"></a><a id="efs-user"></a>Considere la posibilidad de usar el sistema de cifrado de archivos (EFS) para proteger información confidencial específica del usuario

| Título                   | Detalles      |
| ----------------------- | ------------ |
| **Componente**               | Límite de confianza de la máquina | 
| **Fase de SDL**               | Build |  
| **Tecnologías aplicables** | Genérico |
| **Atributos**              | N/D  |
| **Referencias**              | N/D  |
| **Pasos** | Considere la posibilidad de usar el sistema de cifrado de archivos (EFS) para proteger información confidencial específica del usuario frente a adversarios con acceso físico al equipo. |

## <a name="ensure-that-sensitive-data-stored-by-the-application-on-the-file-system-is-encrypted"></a><a id="filesystem"></a>Asegúrese de que se cifre la información confidencial almacenada por la aplicación en el sistema de archivos

| Título                   | Detalles      |
| ----------------------- | ------------ |
| **Componente**               | Límite de confianza de la máquina | 
| **Fase de SDL**               | Implementación |  
| **Tecnologías aplicables** | Genérico |
| **Atributos**              | N/D  |
| **Referencias**              | N/D  |
| **Pasos** | Asegúrese de que la información confidencial almacenada por la aplicación en el sistema de archivos esté cifrada (por ejemplo, mediante DPAPI) si no se puede aplicar EFS. |

## <a name="ensure-that-sensitive-content-is-not-cached-on-the-browser"></a><a id="cache-browser"></a>Asegúrese de que no se almacene contenido confidencial en la memoria caché del explorador

| Título                   | Detalles      |
| ----------------------- | ------------ |
| **Componente**               | Aplicación web | 
| **Fase de SDL**               | Build |  
| **Tecnologías aplicables** | Genérico, formularios Web Forms, MVC5, MVC6 |
| **Atributos**              | N/D  |
| **Referencias**              | N/D  |
| **Pasos** | Los exploradores pueden almacenar información para la memoria caché y el historial. Estos archivos almacenados en caché se guardan en una carpeta, como la carpeta Archivos temporales de Internet en el caso de Internet Explorer. Si se vuelve a hacer referencia a estas páginas, el explorador las muestra desde su memoria caché. Si se muestra información confidencial al usuario (por ejemplo, su dirección, datos de la tarjeta de crédito, número del seguro social o nombre de usuario), podría quedar almacenada en la memoria caché del explorador y, por tanto, ser recuperable mediante un examen de la memoria caché del explorador o simplemente al presionar el botón "Atrás" del explorador. Establezca el valor del encabezado de respuesta cache-control en "no-store" para todas las páginas. |

### <a name="example"></a>Ejemplo
```XML
<configuration>
  <system.webServer>
   <httpProtocol>
    <customHeaders>
        <add name="Cache-Control" value="no-cache" />
        <add name="Pragma" value="no-cache" />
        <add name="Expires" value="-1" />
    </customHeaders>
  </httpProtocol>
 </system.webServer>
</configuration>
```

### <a name="example"></a>Ejemplo
Esto se puede implementar por medio de un filtro. Puede usar el siguiente ejemplo: 
```csharp
public override void OnActionExecuting(ActionExecutingContext filterContext)
        {
            if (filterContext == null || (filterContext.HttpContext != null && filterContext.HttpContext.Response != null && filterContext.HttpContext.Response.IsRequestBeingRedirected))
            {
                //// Since this is MVC pipeline, this should never be null.
                return;
            }

            var attributes = filterContext.ActionDescriptor.GetCustomAttributes(typeof(System.Web.Mvc.OutputCacheAttribute), false);
            if (attributes == null || **Attributes**.Count() == 0)
            {
                filterContext.HttpContext.Response.Cache.SetNoStore();
                filterContext.HttpContext.Response.Cache.SetCacheability(HttpCacheability.NoCache);
                filterContext.HttpContext.Response.Cache.SetExpires(DateTime.UtcNow.AddHours(-1));
                if (!filterContext.IsChildAction)
                {
                    filterContext.HttpContext.Response.AppendHeader("Pragma", "no-cache");
                }
            }

            base.OnActionExecuting(filterContext);
        }
``` 

## <a name="encrypt-sections-of-web-apps-configuration-files-that-contain-sensitive-data"></a><a id="encrypt-data"></a>Cifre las secciones de los archivos de configuración de la aplicación web que contengan información confidencial

| Título                   | Detalles      |
| ----------------------- | ------------ |
| **Componente**               | Aplicación web | 
| **Fase de SDL**               | Build |  
| **Tecnologías aplicables** | Genérico |
| **Atributos**              | N/D  |
| **Referencias**              | [Cómo: Encrypt Configuration Sections in ASP.NET 2.0 Using DPAPI](https://msdn.microsoft.com/library/ff647398.aspx) (Procedimiento para cifrar secciones de configuración en ASP.NET 2.0 mediante DPAPI), [Especificar un proveedor de configuración protegida](https://msdn.microsoft.com/library/68ze1hb2.aspx), [Uso de Azure Key Vault para proteger los secretos de la aplicación](https://azure.microsoft.com/documentation/articles/guidance-multitenant-identity-keyvault/) |
| **Pasos** | Los archivos de configuración tales como web.config y appsettings.json se suelen usar para almacenar información confidencial, como nombres de usuario, contraseñas, cadenas de conexión a la base de datos y claves de cifrado. Si no protege esta información, la aplicación es vulnerable a atacantes o usuarios malintencionados que obtienen información confidencial, como nombres de usuario y contraseñas de cuentas, nombres de bases de datos y nombres de servidores. Según el tipo de implementación (Azure o local), cifre las secciones confidenciales de los archivos de configuración mediante DPAPI o servicios como Azure Key Vault. |

## <a name="explicitly-disable-the-autocomplete-html-attribute-in-sensitive-forms-and-inputs"></a><a id="autocomplete-input"></a>Deshabilite explícitamente el atributo HTML autocomplete en formularios y entradas confidenciales

| Título                   | Detalles      |
| ----------------------- | ------------ |
| **Componente**               | Aplicación web | 
| **Fase de SDL**               | Build |  
| **Tecnologías aplicables** | Genérico |
| **Atributos**              | N/D  |
| **Referencias**              | [MSDN: autocomplete attribute](https://msdn.microsoft.com/library/ms533486(VS.85).aspx) (MSDN: atributo autocomplete), [Using Autocomplete en HTML forms](https://msdn.microsoft.com/library/ms533032.aspx) (Uso de Autocompletar en formularios HTML), [Vulnerabilidad en la comprobación del estado de HTML](https://technet.microsoft.com/security/bulletin/MS10-071), [Autocomplete.,again?!](https://blog.mindedsecurity.com/2011/10/autocompleteagain.html) (Vuelta a Autocompletar) |
| **Pasos** | El atributo autocomplete especifica si un formulario debe tener Autocompletar activado o desactivado. Cuando está activado, el explorador completa automáticamente los valores en función de los que el usuario haya escrito antes. Por ejemplo, cuando se escribe un nombre y una contraseña nuevos en un formulario y se envía este, el explorador le pregunta si se debe guardar la contraseña. A partir de entonces, cuando se muestre el formulario, el nombre y la contraseña se rellenan automáticamente o se completan cuando se escribe el nombre. Un atacante con acceso local podría obtener la contraseña no cifrada de la memoria caché del explorador. De forma predeterminada, la función Autocompletar está habilitada y se debe deshabilitar explícitamente. |

### <a name="example"></a>Ejemplo
```csharp
<form action="Login.aspx" method="post " autocomplete="off" >
      Social Security Number: <input type="text" name="ssn" />
      <input type="submit" value="Submit" />    
</form>
```

## <a name="ensure-that-sensitive-data-displayed-on-the-user-screen-is-masked"></a><a id="data-mask"></a>Asegúrese de que la información confidencial que se muestra en la pantalla del usuario esté enmascarada

| Título                   | Detalles      |
| ----------------------- | ------------ |
| **Componente**               | Aplicación web | 
| **Fase de SDL**               | Build |  
| **Tecnologías aplicables** | Genérico |
| **Atributos**              | N/D  |
| **Referencias**              | N/D  |
| **Pasos** | La información confidencial, como contraseñas, números de tarjeta de crédito, números del seguro social, etc., debe estar enmascarada cuando se muestre en pantalla. El objetivo es impedir que el personal no autorizado acceda a los datos (por ejemplo, contraseñas vistas por encima del hombro, personal de soporte que ve los números del seguro social de los usuarios). Asegúrese de que estos elementos de datos no sean visibles sin cifrar y que estén enmascarados correctamente. Esto se debe llevar a cabo al mismo tiempo que se aceptan como entrada (por ejemplo, input type="password"), así como cuando se muestran en pantalla (por ejemplo, mostrar solo los cuatro últimos dígitos del número de la tarjeta de crédito). |

## <a name="implement-dynamic-data-masking-to-limit-sensitive-data-exposure-non-privileged-users"></a><a id="dynamic-users"></a>Implemente el enmascaramiento dinámico de datos para limitar la exposición de información confidencial a usuarios sin privilegios

| Título                   | Detalles      |
| ----------------------- | ------------ |
| **Componente**               | Base de datos | 
| **Fase de SDL**               | Build |  
| **Tecnologías aplicables** | SQL Azure, OnPrem |
| **Atributos**              | Versión de SQL: V12, versión de SQL: MsSQL2016 |
| **Referencias**              | [Enmascaramiento de datos dinámicos](https://msdn.microsoft.com/library/mt130841) |
| **Pasos** | La finalidad del enmascaramiento dinámico de datos consiste en limitar la exposición de la información confidencial, con lo que se impide que los usuarios vean datos a los que no deberían poder acceder. El enmascaramiento dinámico de datos no pretende evitar que los usuarios de la base de datos se conecten directamente a ella y ejecuten consultas exhaustivas que expongan información confidencial. El enmascaramiento dinámico de datos complementa a otras características de seguridad de SQL Server (auditoría, cifrado, seguridad de nivel de fila…) y se recomienda encarecidamente que se utilice junto con ellas para proteger mejor la información confidencial en la base de datos. Tenga en cuenta que esta característica solo es compatible con SQL Server a partir de 2016 y Azure SQL Database. |

## <a name="ensure-that-passwords-are-stored-in-salted-hash-format"></a><a id="salted-hash"></a>Asegúrese de que las contraseñas se almacenen en formato de hash con sal

| Título                   | Detalles      |
| ----------------------- | ------------ |
| **Componente**               | Base de datos | 
| **Fase de SDL**               | Build |  
| **Tecnologías aplicables** | Genérico |
| **Atributos**              | N/D  |
| **Referencias**              | [Hash de contraseña mediante las API de criptografía de .NET](https://docs.asp.net/en/latest/security/data-protection/consumer-apis/password-hashing.html) |
| **Pasos** | Las contraseñas no se deberían almacenar en bases de datos de almacén de usuarios personalizadas. En su lugar, los hash de contraseña se deben almacenar con valores sal. Asegúrese de que el valor de sal del usuario es siempre único y de que aplica b-crypt, s-crypt o PBKDF2 antes de almacenar la contraseña, con un número de iteraciones de factor de trabajo mínimo de 150 000 bucles para eliminar la posibilidad de ataques de fuerza bruta.| 

## <a name="ensure-that-sensitive-data-in-database-columns-is-encrypted"></a><a id="db-encrypted"></a>Asegúrese de que la información confidencial en las columnas de la base de datos esté cifrada

| Título                   | Detalles      |
| ----------------------- | ------------ |
| **Componente**               | Base de datos | 
| **Fase de SDL**               | Build |  
| **Tecnologías aplicables** | Genérico |
| **Atributos**              | Versión de SQL: todas |
| **Referencias**              | [Cifrar datos confidenciales en SQL server](https://technet.microsoft.com/library/ff848751(v=sql.105).aspx), [Cifrar una columna de datos](https://msdn.microsoft.com/library/ms179331), [Cifrar por certificado](https://msdn.microsoft.com/library/ms188061) |
| **Pasos** | La información confidencial, como los números de tarjeta de crédito, debe estar cifrada en la base de datos. Los datos se pueden cifrar con cifrado de nivel de columna o con una función de aplicación mediante las funciones de cifrado. |

## <a name="ensure-that-database-level-encryption-tde-is-enabled"></a><a id="tde-enabled"></a>Asegúrese de que el cifrado de base de datos (TDE) esté habilitado

| Título                   | Detalles      |
| ----------------------- | ------------ |
| **Componente**               | Base de datos | 
| **Fase de SDL**               | Build |  
| **Tecnologías aplicables** | Genérico |
| **Atributos**              | N/D  |
| **Referencias**              | [Descripción del Cifrado de datos transparente (TDE) en SQL Server](https://technet.microsoft.com/library/bb934049(v=sql.105).aspx) |
| **Pasos** | La característica Cifrado de datos transparente (TDE) de SQL Server ayuda a cifrar la información confidencial en una base de datos y a proteger las claves que se usan para cifrar los datos con un certificado. Esto impide que alguien que carezca de las claves use los datos. TDE protege los datos "en reposo", es decir, los archivos de datos y de registro. Ofrece la posibilidad de cumplir muchas leyes, normativas y directrices establecidas en diversos sectores. |

## <a name="ensure-that-database-backups-are-encrypted"></a><a id="backup"></a>Asegúrese de que las copias de seguridad de la base de datos estén cifradas

| Título                   | Detalles      |
| ----------------------- | ------------ |
| **Componente**               | Base de datos | 
| **Fase de SDL**               | Build |  
| **Tecnologías aplicables** | SQL Azure, OnPrem |
| **Atributos**              | Versión de SQL: V12, versión de SQL: MsSQL2014 |
| **Referencias**              | [Cifrado de copia de seguridad de bases de datos SQL](https://msdn.microsoft.com/library/dn449489) |
| **Pasos** | SQL Server tiene la capacidad de cifrar los datos mientras se crea una copia de seguridad. Al especificar el algoritmo de cifrado y el sistema de cifrado (un certificado o una clave asimétrica) cuando se crea una copia de seguridad, se puede crear un archivo de copia de seguridad cifrado. |

## <a name="ensure-that-sensitive-data-relevant-to-web-api-is-not-stored-in-browsers-storage"></a><a id="api-browser"></a>Asegúrese de que la información confidencial relevante para la API web no se guarde en el almacenamiento del explorador

| Título                   | Detalles      |
| ----------------------- | ------------ |
| **Componente**               | API Web | 
| **Fase de SDL**               | Build |  
| **Tecnologías aplicables** | MVC 5, MVC 6 |
| **Atributos**              | Proveedor de identidades; ADFS, Proveedor de identidades; Azure AD |
| **Referencias**              | N/D  |
| **Pasos** | <p>En algunas implementaciones, se guardan artefactos confidenciales pertinentes para la autenticación de la API web en el almacenamiento local del explorador. Por ejemplo, artefactos de autenticación de Azure AD como adal.idtoken, adal.nonce.idtoken, adal.access.token.key, adal.token.keys, adal.state.login, adal.session.state, adal.expiration.key, etc.</p><p>Todos estos artefactos están disponibles aun después de que se cierren la sesión o el explorador. Si un adversario obtiene acceso a estos artefactos, puede volver a usarlos para acceder a los recursos protegidos (las API). Asegúrese de que todos los artefactos confidenciales relacionados con la API web no se guarden en el almacenamiento del explorador. En aquellos casos en que el almacenamiento del lado cliente sea inevitable (por ejemplo, las aplicaciones con una sola página (SPA) que aprovechan los flujos de OAuth/OpenIdConnect implícito necesitan almacenar localmente los tokens de acceso), use opciones de almacenamiento sin persistencia. Por ejemplo, es preferible SessionStorage a LocalStorage.</p>| 

### <a name="example"></a>Ejemplo
El siguiente fragmento de código JavaScript es de una biblioteca de autenticación personalizada que guarda artefactos de autenticación en almacenamiento local. Se deben evitar estas implementaciones. 
```javascript
ns.AuthHelper.Authenticate = function () {
window.config = {
instance: 'https://login.microsoftonline.com/',
tenant: ns.Configurations.Tenant,
clientId: ns.Configurations.AADApplicationClientID,
postLogoutRedirectUri: window.location.origin,
cacheLocation: 'localStorage', // enable this for IE, as sessionStorage does not work for localhost.
};
```

## <a name="encrypt-sensitive-data-stored-in-cosmos-db"></a><a id="encrypt-docdb"></a>Cifre la información confidencial almacenada en Cosmos DB

| Título                   | Detalles      |
| ----------------------- | ------------ |
| **Componente**               | Azure DocumentDB | 
| **Fase de SDL**               | Build |  
| **Tecnologías aplicables** | Genérico |
| **Atributos**              | N/D  |
| **Referencias**              | N/D  |
| **Pasos** | Cifre la información confidencial en el nivel de aplicación antes de almacenarla en DocumentDB o almacene cualquier información confidencial en otras soluciones de almacenamiento como Azure Storage o SQL de Azure.| 

## <a name="use-azure-disk-encryption-to-encrypt-disks-used-by-virtual-machines"></a><a id="disk-vm"></a>Use Azure Disk Encryption para cifrar discos usados por máquinas virtuales

| Título                   | Detalles      |
| ----------------------- | ------------ |
| **Componente**               | Límites de confianza de máquinas virtuales de IaaS de Azure | 
| **Fase de SDL**               | Implementación |  
| **Tecnologías aplicables** | Genérico |
| **Atributos**              | N/D  |
| **Referencias**              | [Uso de Azure Disk Encryption para cifrar discos usados por las máquinas virtuales](https://azure.microsoft.com/documentation/articles/storage-security-guide/#_using-azure-disk-encryption-to-encrypt-disks-used-by-your-virtual-machines) |
| **Pasos** | <p>El Azure Disk Encryption es una nueva característica que está actualmente en su versión preliminar. Esta característica le permite cifrar los discos de datos y del sistema operativo usados por una máquina virtual de IaaS. Para Windows, las unidades se cifran mediante la tecnología de cifrado de BitLocker estándar del sector. Para Linux, los discos se cifran mediante la tecnología DM-Crypt. Se integra con Azure Key Vault para permitirle controlar y administrar las claves de cifrado del disco. La solución Azure Disk Encryption admite los tres siguientes escenarios de cifrado de cliente:</p><ul><li>Habilitar el cifrado en nuevas máquinas virtuales de IaaS creadas a partir de archivos VHD cifrados por el cliente y claves de cifrado proporcionadas por el cliente, que se almacenan en Azure Key Vault.</li><li>Habilitar el cifrado en nuevas máquinas virtuales de IaaS creadas en Azure Marketplace.</li><li>Habilitar el cifrado en máquinas virtuales de IaaS existentes que ya se ejecutan en Azure.</li></ul>| 

## <a name="encrypt-secrets-in-service-fabric-applications"></a><a id="fabric-apps"></a>Cifre los secretos en aplicaciones de Service Fabric

| Título                   | Detalles      |
| ----------------------- | ------------ |
| **Componente**               | Límites de confianza de Service Fabric | 
| **Fase de SDL**               | Build |  
| **Tecnologías aplicables** | Genérico |
| **Atributos**              | Entorno: Azure |
| **Referencias**              | [Administración de secretos en aplicaciones de Service Fabric](https://azure.microsoft.com/documentation/articles/service-fabric-application-secret-management/) |
| **Pasos** | Los secretos pueden ser cualquier información confidencial, como cadenas de conexión de almacenamiento, contraseñas u otros valores que no se deben administrar en texto sin formato. Use Azure Key Vault para administrar claves y secretos en aplicaciones de Service Fabric. |

## <a name="perform-security-modeling-and-use-business-unitsteams-where-required"></a><a id="modeling-teams"></a>Realice el modelado de seguridad y use unidades de negocio y equipos cuando sea necesario

| Título                   | Detalles      |
| ----------------------- | ------------ |
| **Componente**               | Dynamics CRM | 
| **Fase de SDL**               | Build |  
| **Tecnologías aplicables** | Genérico |
| **Atributos**              | N/D  |
| **Referencias**              | N/D  |
| **Pasos** | Realice el modelado de seguridad y use unidades de negocio y equipos cuando sea necesario. |

## <a name="minimize-access-to-share-feature-on-critical-entities"></a><a id="entities"></a>Minimice el acceso a la característica para compartir en entidades críticas

| Título                   | Detalles      |
| ----------------------- | ------------ |
| **Componente**               | Dynamics CRM | 
| **Fase de SDL**               | Implementación |  
| **Tecnologías aplicables** | Genérico |
| **Atributos**              | N/D  |
| **Referencias**              | N/D  |
| **Pasos** | Minimice el acceso a la característica para compartir en entidades críticas. |

## <a name="train-users-on-the-risks-associated-with-the-dynamics-crm-share-feature-and-good-security-practices"></a><a id="good-practices"></a>Entrene a los usuarios sobre los riesgos asociados con la característica Compartir de Dynamics CRM y los procedimientos recomendados de seguridad

| Título                   | Detalles      |
| ----------------------- | ------------ |
| **Componente**               | Dynamics CRM | 
| **Fase de SDL**               | Implementación |  
| **Tecnologías aplicables** | Genérico |
| **Atributos**              | N/D  |
| **Referencias**              | N/D  |
| **Pasos** | Entrene a los usuarios sobre los riesgos asociados con la característica Compartir de Dynamics CRM y los procedimientos recomendados de seguridad. |

## <a name="include-a-development-standards-rule-proscribing-showing-config-details-in-exception-management"></a><a id="exception-mgmt"></a>Incluya una regla de estándares de desarrollo que prohíba mostrar detalles de configuración en la administración de excepciones

| Título                   | Detalles      |
| ----------------------- | ------------ |
| **Componente**               | Dynamics CRM | 
| **Fase de SDL**               | Implementación |  
| **Tecnologías aplicables** | Genérico |
| **Atributos**              | N/D  |
| **Referencias**              | N/D  |
| **Pasos** | Incluya una regla de estándares de desarrollo que prohíba mostrar detalles de configuración en la administración de excepciones fuera del desarrollo. Pruebe esto como parte de las revisiones del código o de la inspección periódica.|

## <a name="use-azure-storage-service-encryption-sse-for-data-at-rest-preview"></a><a id="sse-preview"></a>Use el cifrado del servicio Azure Storage (SSE) para datos en reposo (versión preliminar)

| Título                   | Detalles      |
| ----------------------- | ------------ |
| **Componente**               | Azure Storage | 
| **Fase de SDL**               | Build |  
| **Tecnologías aplicables** | Genérico |
| **Atributos**              | StorageType: blob |
| **Referencias**              | [Cifrado del servicio Azure Storage para datos en reposo (versión preliminar)](https://azure.microsoft.com/documentation/articles/storage-service-encryption/) |
| **Pasos** | <p>Cifrado del servicio Azure Storage (SSE) para datos en reposo le ayuda a asegurar y proteger sus datos con el fin de cumplir con los compromisos de cumplimiento y seguridad de su organización. Con esta característica, Azure Storage cifra automáticamente sus datos antes de continuar al almacenamiento y los descifra después de la recuperación. La administración de claves, cifrado y descifrado es completamente transparente para los usuarios. SSE se aplica solo a blobs en bloques, blobs en páginas y blobs en anexos. Los otros tipos de datos, como tablas, colas y archivos, no se cifrarán.</p><p>Flujo de trabajo de cifrado y descifrado:</p><ul><li>El cliente habilita el cifrado en la cuenta de almacenamiento.</li><li>Cuando el cliente escribe datos nuevos (PUT Blob, PUT Block, PUT Page, etc.) en Blob Storage, cada escritura se cifra mediante un cifrado AES de 256 bits, uno de los cifrados de bloques más sólidos disponibles.</li><li>Cuando el cliente necesita acceder a datos (GET Blob, etc.), estos se descifran automáticamente antes de que se devuelvan al usuario.</li><li>Si el cifrado está deshabilitado, las escrituras nuevas se dejan de cifrar y los datos cifrados existentes siguen cifrados hasta que el usuario los reescriba. Mientras el cifrado esté habilitado, se cifrarán las escrituras en el Almacenamiento de blobs. El estado de los datos no cambia cuando el usuario habilita y deshabilita el cifrado de la cuenta de almacenamiento.</li><li>Microsoft almacena, cifra y administra todas las claves de cifrado.</li></ul><p>Tenga en cuenta que, en este momento, Microsoft administra las claves utilizadas para el cifrado. Microsoft genera las claves originalmente y administra su almacenamiento seguro, así como la rotación periódica de acuerdo con la política interna de Microsoft. En el futuro, los clientes tendrán la posibilidad de administrar sus propias claves de cifrado, y se proporcionará una ruta de migración de las claves administradas por Microsoft a las administradas por el cliente.</p>| 

## <a name="use-client-side-encryption-to-store-sensitive-data-in-azure-storage"></a><a id="client-storage"></a>Use el cifrado en el cliente para almacenar información confidencial en Azure Storage

| Título                   | Detalles      |
| ----------------------- | ------------ |
| **Componente**               | Azure Storage | 
| **Fase de SDL**               | Build |  
| **Tecnologías aplicables** | Genérico |
| **Atributos**              | N/D  |
| **Referencias**              | [Cifrado del lado de cliente y Azure Key Vault para Microsoft Azure Storage](https://azure.microsoft.com/documentation/articles/storage-client-side-encryption/), [Tutorial: Cifrado y descifrado de blobs en Microsoft Azure Storage con Azure Key Vault](https://azure.microsoft.com/documentation/articles/storage-encrypt-decrypt-blobs-key-vault/), [Storing Data Securely in Azure Blob Storage with Azure Encryption Extensions](https://blogs.msdn.microsoft.com/partnercatalystteam/2015/06/17/storing-data-securely-in-azure-blob-storage-with-azure-encryption-extensions/) (Almacenamiento seguro de datos en Azure Blob Storage con Azure Encryption Extensions) |
| **Pasos** | <p>La biblioteca de cliente de Azure Storage para el paquete NuGet de .NET admite el cifrado de datos dentro de las aplicaciones cliente antes de cargarlos en Azure Storage y el descifrado de los datos mientras se descargan al cliente. La biblioteca también admite la integración con Azure Key Vault para la administración de las claves de la cuenta de almacenamiento. Esta es una breve descripción de cómo funciona el cifrado del lado cliente:</p><ul><li>El SDK de cliente de Azure Storage genera una clave de cifrado de contenido (CEK), que es una clave simétrica de un solo uso.</li><li>Los datos de usuario se cifran mediante esta CEK.</li><li>Se encapsula la CEK (cifrada) con la clave de cifrado de clave (KEK). La KEK se identifica mediante un identificador de clave y puede ser un par de clave asimétrico o una clave simétrica que puede administrarse de forma local o guardarse en Azure Key Vault. El propio cliente de almacenamiento no tiene acceso a KEK. Simplemente invoca el algoritmo de ajuste de clave proporcionado por Key Vault. Los clientes pueden elegir usar proveedores personalizados para encapsular y desencapsular la clave si así lo desean.</li><li>A continuación, se cargan los datos cifrados en el servicio Azure Storage. Consulte los vínculos de la sección Referencias para ver detalles de implementación de bajo nivel.</li></ul>|

## <a name="encrypt-sensitive-or-pii-data-written-to-phones-local-storage"></a><a id="pii-phones"></a>Cifre la información confidencial o de identificación personal guardada en el almacenamiento local de teléfonos

| Título                   | Detalles      |
| ----------------------- | ------------ |
| **Componente**               | Cliente móvil | 
| **Fase de SDL**               | Build |  
| **Tecnologías aplicables** | Genérico, Xamarin  |
| **Atributos**              | N/D  |
| **Referencias**              | [Administrar la configuración y las características de los dispositivos con directivas de Microsoft Intune](https://docs.microsoft.com/mem/intune/configuration/), [Keychain Valet](https://components.xamarin.com/view/square.valet) |
| **Pasos** | <p>Si la aplicación escribe información confidencial, como información de identificación personal del usuario (correo electrónico, número de teléfono, nombre, apellidos, preferencias, etc.), en el sistema de archivos del dispositivo móvil, se debería cifrar antes de escribirla en el sistema de archivos local. Si la aplicación es empresarial, considere la posibilidad de publicar la aplicación mediante Windows Intune.</p>|

### <a name="example"></a>Ejemplo
Intune se puede configurar con las siguientes directivas de seguridad para proteger datos confidenciales: 
```csharp
Require encryption on mobile device    
Require encryption on storage cards
Allow screen capture
```

### <a name="example"></a>Ejemplo
Si la aplicación no es empresarial, use el almacén de claves proporcionado por la plataforma, llaveros para almacenar claves de cifrado, con cualquier operación criptográfica que se pueda realizar en el sistema de archivos. En el siguiente fragmento de código, se muestra cómo acceder a la clave del llavero con xamarin: 
```csharp
        protected static string EncryptionKey
        {
            get
            {
                if (String.IsNullOrEmpty(_Key))
                {
                    var query = new SecRecord(SecKind.GenericPassword);
                    query.Service = NSBundle.MainBundle.BundleIdentifier;
                    query.Account = "UniqueID";

                    NSData uniqueId = SecKeyChain.QueryAsData(query);
                    if (uniqueId == null)
                    {
                        query.ValueData = NSData.FromString(System.Guid.NewGuid().ToString());
                        var err = SecKeyChain.Add(query);
                        _Key = query.ValueData.ToString();
                    }
                    else
                    {
                        _Key = uniqueId.ToString();
                    }
                }

                return _Key;
            }
        }
```

## <a name="obfuscate-generated-binaries-before-distributing-to-end-users"></a><a id="binaries-end"></a>Ofusque los archivos binarios generados antes de distribuirlos a los usuarios finales

| Título                   | Detalles      |
| ----------------------- | ------------ |
| **Componente**               | Cliente móvil | 
| **Fase de SDL**               | Build |  
| **Tecnologías aplicables** | Genérico |
| **Atributos**              | N/D  |
| **Referencias**              | [Crypto Obfuscator For .Net](https://www.ssware.com/cryptoobfuscator/obfuscator-net.htm) |
| **Pasos** | Los archivos binarios generados (ensamblados en apk) se deben ofuscar para detener la ingeniería inversa de los ensamblados. Para tal propósito se pueden usar herramientas como `CryptoObfuscator`. |

## <a name="set-clientcredentialtype-to-certificate-or-windows"></a><a id="cert"></a>Establezca clientCredentialType en Certificate o Windows

| Título                   | Detalles      |
| ----------------------- | ------------ |
| **Componente**               | WCF | 
| **Fase de SDL**               | Build |  
| **Tecnologías aplicables** | .NET Framework 3 |
| **Atributos**              | N/D  |
| **Referencias**              | [Fortify](https://vulncat.fortify.com/en/detail?id=desc.config.dotnet.wcf_misconfiguration_weak_token) |
| **Pasos** | Si se usa un token UsernameToken con una contraseña sin cifrar en un canal no cifrado, se expone la contraseña a atacantes que puedan rastrear los mensajes SOAP. Es posible que los proveedores de servicios que usan UsernameToken acepten las contraseñas enviadas sin cifrar. Si se envían contraseñas sin cifrar por un canal no cifrado, se puede exponer la credencial a atacantes que puedan rastrear el mensaje SOAP. | 

### <a name="example"></a>Ejemplo
La siguiente configuración del proveedor de servicios WCF usa UsernameToken: 
```
<security mode="Message"> 
<message clientCredentialType="UserName" />
``` 
Establezca clientCredentialType en Certificate o Windows. 

## <a name="wcf-security-mode-is-not-enabled"></a><a id="security"></a>El modo de seguridad de WCF no está habilitado

| Título                   | Detalles      |
| ----------------------- | ------------ |
| **Componente**               | WCF | 
| **Fase de SDL**               | Build |  
| **Tecnologías aplicables** | Genérico, .NET Framework 3 |
| **Atributos**              | Modo de seguridad: Transport, modo de seguridad: Message |
| **Referencias**              | [MSDN](https://msdn.microsoft.com/library/ff648500.aspx), [Fortify Kingdom](https://vulncat.fortify.com/en/detail?id=desc.config.dotnet.wcf_misconfiguration_weak_class_reference), [Fundamentals of WCF Security CoDe Magazine](https://www.codemag.com/article/0611051) (Fundamentos de la seguridad de WCF en CoDe Magazine) |
| **Pasos** | No se ha definido la seguridad de transporte ni de mensajes. Las aplicaciones que transmiten mensajes sin seguridad de transporte ni de mensajes no pueden garantizar la integridad ni la confidencialidad de los mensajes. Cuando un enlace de seguridad de WCF se establece en None, se deshabilitan la seguridad de mensajes y la de transporte. |

### <a name="example"></a>Ejemplo
La siguiente configuración establece el modo de seguridad en None. 
```
<system.serviceModel> 
  <bindings> 
    <wsHttpBinding> 
      <binding name=""MyBinding""> 
        <security mode=""None""/> 
      </binding> 
  </bindings> 
</system.serviceModel> 
```

### <a name="example"></a>Ejemplo
Modo de seguridad En todos los enlaces de servicio, existen cinco modos de seguridad posibles: 
* Ninguno. Desactiva la seguridad. 
* Transport. Usa la seguridad de transporte para la autenticación mutua y la protección de mensajes. 
* Message. Usa la seguridad de mensajes para la autenticación mutua y la protección de mensajes. 
* Both. Permite proporcionar la configuración para la seguridad de transporte y de mensajes (solo es compatible con MSMQ). 
* TransportWithMessageCredential. Las credenciales se pasan con el mensaje, y la capa de transporte proporciona la protección de mensajes y la autenticación del servidor. 
* TransportCredentialOnly. Las credenciales del cliente se pasan con la capa de transporte y no se aplica la protección de mensajes. Use la seguridad de mensajes y de transporte para proteger la integridad y confidencialidad de los mensajes. La configuración siguiente indica al servicio que use la seguridad de transporte con las credenciales del mensaje.
  ```
  <system.serviceModel>
  <bindings>
    <wsHttpBinding>
    <binding name=""MyBinding""> 
    <security mode=""TransportWithMessageCredential""/> 
    <message clientCredentialType=""Windows""/> 
    </binding> 
  </bindings> 
  </system.serviceModel> 
  ```
