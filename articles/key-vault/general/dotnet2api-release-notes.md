---
title: Notas de la versión de la API de .NET 2.x de Key Vault | Microsoft Docs
description: Aprenda a actualizar aplicaciones escritas para versiones anteriores de Azure Key Vault para que funcionen con la versión 2.0 de la biblioteca de Azure Key Vault para C# y .NET.
services: key-vault
author: msmbaldwin
manager: rkarlin
editor: bryanla
ms.service: key-vault
ms.subservice: general
ms.topic: conceptual
ms.date: 05/02/2017
ms.author: mbaldwin
ms.openlocfilehash: 018570019b306dced76760fefa4441ee7d86ad2a
ms.sourcegitcommit: 9ce0350a74a3d32f4a9459b414616ca1401b415a
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/13/2020
ms.locfileid: "88189828"
---
# <a name="azure-key-vault-net-20---release-notes-and-migration-guide"></a>.NET 2.0 de Azure Key Vault: notas de la versión y guía de migración
La información siguiente ayuda a migrar a la versión 2.0 de la biblioteca de Azure Key Vault para C# y .NET.  Las aplicaciones escritas para las versiones anteriores se deben actualizar para ser compatibles con la versión más reciente.  Estos cambios son necesarios para admitir completamente las características nuevas y mejoradas, como los **certificados de Key Vault**.

## <a name="key-vault-certificates"></a>Certificados de Key Vault

Los certificados de Key Vault administran los certificados x509 y admiten los comportamientos siguientes:  

* Creación de certificados a través de un proceso de creación de Key Vault o importe el certificado existente. Esto incluye tanto certificados autofirmados como certificados generados por una entidad de certificación (CA).
* Almacenamiento y administración de manera segura del almacenamiento de certificados x509 sin ninguna interacción mediante el material de clave privada.  
* Definición de las directivas que indican a Key Vault que administre el ciclo de vida de los certificados.  
* Suministro de información de contacto para los eventos de ciclo de vida, como las advertencias de expiración y las notificaciones de renovación.  
* Renovación automática de los certificados con los emisores seleccionados (entidades de certificación y proveedores de certificados x509 de asociados de Key Vault).* Compatibilidad con certificado de proveedores alternativos (no asociados) y entidades de certificación (no se admite la renovación automática).  

## <a name="net-support"></a>Compatibilidad con .NET

* **.NET 4.0** no es compatible con la versión 2.0 de la biblioteca .NET de Azure Key Vault.
* **.NET Framework 4.5.2** es compatible con la versión 2.0 de la biblioteca .NET de Azure Key Vault.
* **.NET Standard 1.4** es compatible con la versión 2.0 de la biblioteca .NET de Azure Key Vault.

## <a name="namespaces"></a>Espacios de nombres

* El espacio de nombres para **modelos** se cambia de **Microsoft.Azure.KeyVault** a **Microsoft.Azure.KeyVault.Models**.
* El espacio de nombres **Microsoft.Azure.KeyVault.Internal** se quita.
* Los espacios de nombres de las dependencias del SDK de Azure han cambiado 

    - **Hyak.Common** ahora es **Microsoft.Rest**.
    - **Hyak.Common.Internals** ahora es **Microsoft.Rest.Serialization**.

## <a name="type-changes"></a>Cambios de tipo

* *Secret* se cambia a *SecretBundle*
* *Dictionary* se cambia a *IDictionary*
* *List\<T>, string []* se cambia a *IList\<T>*
* *NextList* se cambia a  *NextPageLink*

## <a name="return-types"></a>Tipos de valores devueltos

* **KeyList** y **SecretList** ahora devuelve *IPage\<T>* en lugar de *ListKeysResponseMessage*
* El valor **BackupKeyAsync** generado ahora devuelve *BackupKeyResult*, que contiene *Value* (blob de copia de seguridad). Anteriormente, el método se encapsuló y solo se devolvió el valor.

## <a name="exceptions"></a>Excepciones

* *KeyVaultClientException* se cambia a *KeyVaultErrorException*
* El error de servicio cambió de *exception.Error* a *exception.Body.Error.Message*.
* Información adicional quitada del mensaje de error para **[JsonExtensionData]** .

## <a name="constructors"></a>Constructores

* En lugar de aceptar *HttpClient* como argumento de constructor, el constructor solo acepta *HttpClientHandler* o *DelegatingHandler[]* .

## <a name="downloaded-packages"></a>Paquetes descargados

Cuando un cliente procesa una dependencia de Key Vault, se descargan los paquetes siguientes:

### <a name="previous-package-list"></a>Lista de paquetes anterior

* `package id="Hyak.Common" version="1.0.2" targetFramework="net45"`
* `package id="Microsoft.Azure.Common" version="2.0.4" targetFramework="net45"`
* `package id="Microsoft.Azure.Common.Dependencies" version="1.0.0" targetFramework="net45"`
* `package id="Microsoft.Azure.KeyVault" version="1.0.0" targetFramework="net45"`
* `package id="Microsoft.Bcl" version="1.1.9" targetFramework="net45"`
* `package id="Microsoft.Bcl.Async" version="1.0.168" targetFramework="net45"`
* `package id="Microsoft.Bcl.Build" version="1.0.14" targetFramework="net45"`
* `package id="Microsoft.Net.Http" version="2.2.22" targetFramework="net45"`

### <a name="current-package-list"></a>Lista de paquetes actual

* `package id="Microsoft.Azure.KeyVault" version="2.0.0-preview" targetFramework="net45"`
* `package id="Microsoft.Rest.ClientRuntime" version="2.2.0" targetFramework="net45"`
* `package id="Microsoft.Rest.ClientRuntime.Azure" version="3.2.0" targetFramework="net45"`

## <a name="class-changes"></a>Cambios de clase

* Se quitó la clase **UnixEpoch**.
* El nombre de la clase **Base64UrlConverter** se cambió a **Base64UrlJsonConverter**.

## <a name="other-changes"></a>Otros cambios

* La compatibilidad con la configuración de la directiva de reintentos de operaciones de KV sobre errores transitorios se ha agregado a esta versión de la API.

## <a name="microsoftazuremanagementkeyvault-nuget"></a>NuGet Microsoft.Azure.Management.KeyVault

* Para las operaciones que devolvieron un *almacén*, el tipo de valor devuelto era una clase que contenía una propiedad **Vault**. Ahora el tipo de valor devuelto es *Vault*.
* Ahora *PermissionsToKeys* y *PermissionsToSecrets* son *Permissions.Keys* y *Permissions.Secrets*
* Ciertos cambios de tipos de valor devuelto se aplican también al plano de control.

## <a name="microsoftazurekeyvaultextensions-nuget"></a>NuGet Microsoft.Azure.KeyVault.Extensions

* El paquete se divide en **Microsoft.Azure.KeyVault.Extensions** y **Microsoft.Azure.KeyVault.Cryptography** para las operaciones de criptografía.

