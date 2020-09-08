---
title: 'Azure AD Connect: Cuando ya tiene una cuenta de Azure AD | Microsoft Docs'
description: En este tema se describe cómo utilizar Connect si ya tiene un inquilino de Azure AD.
services: active-directory
documentationcenter: ''
author: billmath
manager: daveba
editor: ''
ms.assetid: ''
ms.service: active-directory
ms.workload: identity
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: how-to
ms.date: 04/25/2019
ms.subservice: hybrid
ms.author: billmath
ms.collection: M365-identity-device-management
ms.openlocfilehash: 582ec01a7a843358bef749aec693a59f88a1d655
ms.sourcegitcommit: 628be49d29421a638c8a479452d78ba1c9f7c8e4
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/20/2020
ms.locfileid: "88640657"
---
# <a name="azure-ad-connect-when-you-have-an-existing-tenant"></a>Azure AD Connect: Si tiene un inquilino existente
En la mayoría de los temas sobre cómo usar Azure AD Connect se da por supuesto que empieza con un nuevo inquilino de Azure AD sin objetos ni usuarios. Sin embargo, si ha empezado con un inquilino de Azure AD, rellenado con usuarios y otros objetos, y ahora desea utilizar Connect, eche un vistazo a este tema.

## <a name="the-basics"></a>Conceptos básicos
Un objeto de Azure AD se controla en la nube (Azure AD) o en un entorno local. En el caso de un solo objeto, no puede administrar algunos atributos en un entorno local y otros atributos en Azure AD. Cada objeto tiene una marca que indica que el objeto se ha administrado.

Puede administrar algunos usuarios locales y otros en la nube. Un escenario común de esta configuración es una organización que tiene una combinación de trabajadores de contabilidad y trabajadores de ventas. Los trabajadores de contabilidad tienen una cuenta de AD local; sin embargo los trabajadores de ventas no, ellos tienen una cuenta de Azure AD. Tendría que administrar algunos usuarios en el entorno local y otros en Azure AD.

Si ya comenzó a administrar usuarios en Azure AD que también se encuentran en AD local y, posteriormente, desea volver a utilizar Connect, debe tener en cuenta más escenarios.

## <a name="sync-with-existing-users-in-azure-ad"></a>Sincronización con los usuarios existentes de Azure AD
Al instalar Azure AD Connect y empezar la sincronización, el servicio de sincronización de Azure AD (en Azure AD) realiza una comprobación de cada nuevo objeto y trata de buscar un objeto coincidente que ya existe. Hay tres atributos que se utilizan para este proceso: **userPrincipalName**, **proxyAddresses** y **sourceAnchor**/**immutableID**. Una coincidencia de **userPrincipalName** y **proxyAddresses** se conoce como **coincidencia parcial**. Una coincidencia de **sourceAnchor** se conoce como **coincidencia exacta**. En el caso del atributo **proxyAddresses**, solo se usa para la evaluación el valor con el atributo **SMTP:** , que es la dirección de correo electrónico principal.

La coincidencia solo se evalúa para los nuevos objetos procedentes de Connect. Si cambia uno que ya exista para que coincida con alguno de estos atributos, verá un error en su lugar.

Si Azure AD encuentra un objeto cuyos valores de atributo son los mismos que los de uno procedente de Connect y que ya se encuentra en Azure AD, el objeto de Azure AD pasa a ser propiedad de Connect. El objeto administrado previamente en la nube se marca como administrado en local. Todos los atributos de Azure AD con un valor en AD local se sobrescriben con el valor local. La excepción es cuando un atributo tiene un valor **NULL** local. En este caso, el valor sigue estando en Azure AD, pero solo podrá cambiarlo en local.

> [!WARNING]
> Puesto que todos los atributos de Azure AD se van a sobrescribir por el valor local, asegúrese de que dispone de datos en buen estado en un entorno local. Por ejemplo, si solo tiene una dirección de correo electrónico administrada en Office 365 y no se mantiene actualizada en AD DS local, se pierden todos los valores de Azure AD u Office 365 que no se encuentren en AD DS.

> [!IMPORTANT]
> Si usa la sincronización de contraseñas, que siempre se utiliza en la configuración rápida, la contraseña de Azure AD se sobrescribe por la de AD local. Si los usuarios se usan para administrar contraseñas diferentes, tendrá que informarles de que deben usar la contraseña local tras instalar Connect.

Hay que tener en cuenta en la planeación la sección anterior y la advertencia. Si ha realizado muchos cambios en Azure AD que no se reflejan en AD DS local, debe planear cómo rellenar AD DS con los valores actualizados antes de sincronizar los objetos con Azure AD Connect.

Si hay una coincidencia parcial de objetos, el atributo **sourceAnchor** se agrega al objeto en Azure AD para que más tarde se pueda usar una coincidencia exacta.

>[!IMPORTANT]
> Microsoft recomienda encarecidamente no sincronizar las cuentas locales con cuentas administrativas ya existentes en Azure Active Directory.

### <a name="hard-match-vs-soft-match"></a>Diferencias entre la coincidencia parcial y la exacta
En una instalación nueva de Connect, apenas las hay. La diferencia reside en los escenarios de recuperación ante desastres. Si su servidor ha perdido la conexión con Azure AD Connect, puede volver a instalar una nueva instancia sin perder datos. Un objeto con un atributo sourceAnchor se envía a Connect durante la instalación inicial. Después, el cliente (Azure AD Connect) puede evaluar la coincidencia, con lo que el proceso es más mucho rápido que se si hace en Azure AD. Las coincidencias exactas las evalúan Connect y Azure AD, y las parciales, Azure AD.

### <a name="other-objects-than-users"></a>Otros objetos distintos a los usuarios
Para grupos y contactos habilitados para correo electrónico, puede hacer una coincidencia parcial en función de las direcciones de proxy. La coincidencia exacta no es aplicable porque solo puede actualizar el sourceAnchor o inmutableID (mediante PowerShell) en los usuarios. Para grupos que no están habilitados para correo, no se admiten actualmente la coincidencia parcial ni la coincidencia exacta.

### <a name="admin-role-considerations"></a>Consideraciones sobre el rol de administrador
Para evitar que los usuarios de confianza locales coincidan con un usuario en la nube que tenga cualquier rol de administrador, Azure AD Connect no hará coincidir objetos de usuario locales con objetos que tengan un rol de administrador. Esto se aplica de manera predeterminada. Para resolver este comportamiento, puede hacer lo siguiente:

1.  Quitar los roles de directorio del objeto de usuario solo de nube.
2.  Eliminar de forma permanente el objeto en cuarentena en la nube si se produjo un error en el intento de sincronización de usuario.
3.  Desencadenar una sincronización.
4.  Opcionalmente, agregar los roles de directorio al objeto de usuario en la nube una vez que la coincidencia se haya producido.



## <a name="create-a-new-on-premises-active-directory-from-data-in-azure-ad"></a>Creación de una instancia de Active Directory local a partir de los datos de Azure AD
Algunos clientes comienzan con una solución solo en la nube con Azure AD y no tienen una implementación de AD local. Más adelante, desean consumir recursos locales y crear una implementación de AD local basada en los datos de Azure AD. Azure AD Connect no puede ayudarlo en este escenario, ya que no crea los usuarios locales y no tiene ninguna funcionalidad para hacer que la contraseña local sea la misma que la de Azure AD.

Si la única razón por la que piensa agregar AD local es admitir LOB (aplicaciones de línea de negocio), quizá debería plantearse usar los [servicios de dominio de Azure AD](../../active-directory-domain-services/index.yml) en su lugar.

## <a name="next-steps"></a>Pasos siguientes
Obtenga más información sobre la [Integración de las identidades locales con Azure Active Directory](whatis-hybrid-identity.md).
