---
title: Guía de seguridad del Explorador de Azure Storage | Microsoft Docs
description: Guía de seguridad del Explorador de Azure Storage
services: storage
author: cralvord
ms.service: storage
ms.topic: best-practice
ms.date: 07/30/2020
ms.author: cralvord
ms.openlocfilehash: d9dea7cd0cc22cc8a1e0aa5c93ece76d689de0e0
ms.sourcegitcommit: 2ff0d073607bc746ffc638a84bb026d1705e543e
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/06/2020
ms.locfileid: "87835449"
---
# <a name="azure-storage-explorer-security-guide"></a>Guía de seguridad del Explorador de Azure Storage

El Explorador de Microsoft Azure Storage permite trabajar de manera sencilla y segura con los datos de Azure Storage en Windows, macOS y Linux. Si sigue las instrucciones de abajo, puede asegurarse de que los datos se mantengan protegidos.

## <a name="general"></a>General

- **Use siempre la versión más reciente del Explorador de Storage.** Las versiones del Explorador de Storage pueden contener actualizaciones de seguridad. Por tanto, mantenerlo actualizado ayuda a garantizar la seguridad general.
- **Conéctese solo a recursos en los que confíe.** Los datos que se descargan de orígenes que no son de confianza podrían ser malintencionados, y cargar datos en un origen que no es de confianza puede provocar la pérdida o el robo de los datos.
- **Use HTTPS siempre que sea posible.** De forma predeterminada, el Explorador de Storage usa HTTPS. Algunos escenarios permiten usar HTTP, pero este protocolo solo debe usarse como último recurso.
- **Asegúrese de que solo se conceden los permisos necesarios a las personas que los necesiten.** Evite ser demasiado permisivo al conceder a cualquiera acceso a los recursos.
- **Tenga cuidado al ejecutar operaciones críticas.** Ciertas operaciones, como eliminar y sobrescribir, son irreversibles y pueden provocar la pérdida de datos. Asegúrese de que está trabajando con los recursos correctos antes de ejecutar estas operaciones.

## <a name="choosing-the-right-authentication-method"></a>Selección del método de autenticación adecuado

El Explorador de Storage ofrece varias formas de acceder a los recursos de Azure Storage. Sea cual sea el método que elija, estas son nuestras recomendaciones.

### <a name="azure-ad-authentication"></a>Autenticación de Azure AD

La forma más sencilla y segura de acceder a los recursos de Azure Storage es iniciar sesión con una cuenta de Azure. Al hacerlo, se usa la autenticación de Azure AD, que permite lo siguiente:

- Conceder acceso a usuarios y grupos específicos
- Revocar el acceso a usuarios y grupos específicos en cualquier momento
- Aplicar condiciones de acceso, como requerir la autenticación multifactor

Se recomienda usar la autenticación de Azure AD siempre que sea posible.

En esta sección se describen las dos tecnologías basadas en Azure AD que se pueden usar para proteger los recursos de almacenamiento.

#### <a name="role-based-access-control-rbac"></a>Control de acceso basado en roles (RBAC)

El [control de acceso basado en roles de Azure (RBAC de Azure)](https://docs.microsoft.com/azure/role-based-access-control/overview) permite controlar de forma detallada los recursos de Azure. Los roles y permisos de RBAC se pueden administrar desde Azure Portal.

El Explorador de Storage admite el acceso RBAC a cuentas de almacenamiento, blobs y colas. Si necesita acceder a recursos compartidos de archivos o tablas, deberá asignar roles de RBAC que concedan permiso para enumerar claves de cuenta de almacenamiento.

#### <a name="access-control-lists-acls"></a>Listas de control de acceso (ACL)

Las [listas de control de acceso (ACL)](https://docs.microsoft.com/azure/storage/blobs/data-lake-storage-access-control) permiten controlar el acceso a los archivos y carpetas en contenedores de blobs de ADLS Gen2. Puede administrar las ACL con el Explorador de Storage.

### <a name="shared-access-signatures-sas"></a>Firmas de acceso compartido (SAS)

Si no puede usar la autenticación de Azure AD, se recomienda usar firmas de acceso compartido. Con las firmas de acceso compartido, puede hacer lo siguiente:

- Proporcionar acceso limitado anónimo a recursos seguros.
- Revocar una SAS inmediatamente si se generó a partir de una directiva de acceso compartido (SAP).

Sin embargo, con las firmas de acceso compartido, no puede realizar ninguna de las siguientes operaciones:

- Restringir quién puede usar una SAS. Cualquier usuario que tenga una SAS válida puede usarla.
- Revocar una SAS si no se generó a partir de una SAP.

Al usar SAS en el Explorador de Storage, se recomiendan las siguientes directrices:

- **Limite la distribución de tokens y URI de SAS.** Distribuya solo los tokens y URI de SAS a usuarios de confianza. Al limitar la distribución de SAS, se reduce la posibilidad de que se pueda usar incorrectamente una SAS.
- **Use solo tokens y URI de SAS de entidades en las que confíe.**
- **Use las SAP al generar tokens y URI de SAS si es posible.** Una SAS basada en una directiva de acceso compartido es más segura que una SAS básica, ya que una SAS se puede revocar eliminando una SAP.
- **Genere tokens con permisos y acceso mínimos a recursos.** Los permisos mínimos limitan los posibles daños que podrían producirse si se usa incorrectamente una SAS.
- **Genere tokens que solo sean válidos durante el tiempo que sea necesario.** Que tengan una duración corta es especialmente importante en el caso de las SAS básicas, ya que no hay manera de revocarlas una vez que se han generado.

> [!IMPORTANT]
> Al compartir tokens y URI de SAS para solucionar problemas, como en solicitudes de servicio o informes de errores, debe ocultar siempre al menos la parte de la firma de la SAS.

### <a name="storage-account-keys"></a>Claves de cuenta de almacenamiento

Las claves de cuenta de almacenamiento conceden acceso no restringido a los servicios y recursos de una cuenta de almacenamiento. Por esta razón, se recomienda limitar el uso de las claves para acceder a los recursos del Explorador de Storage. En su lugar, use las características de RBAC o SAS para proporcionar acceso.

Algunos roles de RBAC conceden permiso para recuperar claves de cuenta de almacenamiento. Los usuarios con estos roles pueden eludir de manera eficaz los permisos concedidos o denegados mediante RBAC. Se recomienda no conceder este permiso, a menos que sea necesario.

El Explorador de Storage intentará usar claves de cuenta de almacenamiento, si están disponibles para autenticar solicitudes. Puede deshabilitar esta característica en Configuración (**Servicios > Cuentas de almacenamiento > Usage of Keys [Deshabilitar el uso de claves]** ). Algunas características no son compatibles con RBAC, como trabajar con cuentas de almacenamiento clásicas. Estas características siguen requiriendo claves y no se ven afectadas por esta configuración.

Si debe usar claves para tener acceso a los recursos de almacenamiento, se recomiendan las siguientes directrices:

- **No comparta sus claves de cuenta con nadie.**
- **Trate las claves de cuenta de almacenamiento como si fueran contraseñas.** Si sus claves deben estar accesibles, use soluciones de almacenamiento seguro como [Azure Key Vault](https://azure.microsoft.com/services/key-vault/).

> [!NOTE]
> Si cree que una clave de cuenta de almacenamiento se ha compartido o distribuido por error, puede generar nuevas claves para la cuenta de almacenamiento desde Azure Portal.

### <a name="public-access-to-blob-containers"></a>Acceso público a contenedores de blobs

El Explorador de Storage permite modificar el nivel de acceso de los contenedores de Azure Blob Storage. Los contenedores de blobs no privados permiten el acceso de lectura anónimo a los datos de esos contenedores.

Al habilitar el acceso público a un contenedor de blobs, se recomiendan las siguientes directrices:

- **No habilite el acceso público a un contenedor de blobs que pueda contener datos potencialmente confidenciales.** Asegúrese de que el contenedor de blobs no tenga datos privados.
- **No cargue datos potencialmente confidenciales en un contenedor de blobs con acceso a blobs o contenedores.** 

## <a name="next-steps"></a>Pasos siguientes

- [Recomendaciones de seguridad](https://docs.microsoft.com/azure/storage/blobs/security-recommendations)
