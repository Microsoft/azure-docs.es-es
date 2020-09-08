---
title: Conceptos de diseño de Azure AD Connect | Microsoft Docs
description: En este tema se detallan algunas áreas de diseño de implementación
services: active-directory
documentationcenter: ''
author: billmath
manager: daveba
editor: ''
ms.assetid: 4114a6c0-f96a-493c-be74-1153666ce6c9
ms.service: active-directory
ms.custom: azure-ad-connect
ms.devlang: na
ms.topic: conceptual
ms.tgt_pltfrm: na
ms.workload: Identity
ms.date: 08/10/2018
ms.subservice: hybrid
ms.author: billmath
ms.collection: M365-identity-device-management
ms.openlocfilehash: baa03499cc11bda24ead986dd64621572484cbb1
ms.sourcegitcommit: c94a177b11a850ab30f406edb233de6923ca742a
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/01/2020
ms.locfileid: "89279659"
---
# <a name="azure-ad-connect-design-concepts"></a>Azure AD Connect: conceptos de diseño
El propósito de este documento es describir las áreas que se deben tener en cuenta durante el diseño de implementación de Azure AD Connect. Este documento trata de una profundización en determinadas áreas y estos conceptos se describen también brevemente en otros documentos.

## <a name="sourceanchor"></a>sourceAnchor
El atributo sourceAnchor se define como *un atributo inmutable durante la vigencia de un objeto*. Identifica de forma única que un objeto es el mismo objeto en el entorno local y en Azure AD. El atributo también se denomina **immutableId** y los dos nombres se usan indistintamente.

La palabra inmutable, es decir, que no se puede cambiar, es importante en este documento. Puesto que el valor de este atributo no se puede cambiar después de que se haya establecido, es importante elegir un diseño que respalde su caso.

El atributo se usa en las situaciones siguientes:

* Cuando un nuevo servidor de motor de sincronización se genera, o regenera, después de una situación de recuperación ante desastres, este atributo vincula los objetos existentes en Azure AD con los objetos locales.
* Si se mueve de una identidad solo de nube a un modelo de identidad sincronizada, este atributo permitirá que los objetos existentes en Azure AD coincidan exactamente con los objetos locales.
* Si usa federación, este atributo junto con el **userPrincipalName** se usan en la notificación para identificar de forma exclusiva un usuario.

En este tema solo se habla de sourceAnchor, puesto que está relacionado con los usuarios. Las mismas reglas se aplican a todos los tipos de objeto, pero es solo este problema el que supone una preocupación para los usuarios.

### <a name="selecting-a-good-sourceanchor-attribute"></a>Selección de un buen atributo sourceAnchor
El valor de atributo debe seguir las reglas siguientes:

* Menos de 60 caracteres.
  * Los caracteres que no estén entre a-z, A-Z o 0-9 se codifican y cuentan como 3 caracteres.
* No debe contener caracteres especiales: &#92; ! # $ % & * + / = ? ^ &#96; { } | ~ < > ( ) ' ; : , [ ] " \@ _
* Debe ser único globalmente.
* Debe ser una cadena, un entero o un número binario.
* No se debe basar en el nombre del usuario porque este puede cambiar.
* No debe distinguir mayúsculas y minúsculas; evite valores que pueden variar según el caso
* Debe asignarse cuando se crea el objeto.

Si el atributo sourceAnchor seleccionado no es de tipo cadena, Azure AD Connect aplica Base64Encode al valor de atributo para asegurarse de que no se muestre ningún carácter especial. Si usa otro servidor de federación distinto de ADFS, asegúrese de que también pueda aplicar Base64Encode al atributo.

El atributo sourceAnchor distingue mayúsculas de minúsculas. El valor "JohnDoe" no es igual que "johndoe". Pero no debería tener dos objetos diferentes con solo una diferencia en mayúsculas o minúsculas.

Si tiene un solo bosque local, el atributo que debe usar es **objectGUID**. También es el atributo que se usa con la configuración rápida en Azure AD Connect y el atributo de sincronización de directorios.

Si tiene varios bosques y no mueve usuarios entre bosques y dominios, **objectGUID** es un buen atributo para usar incluso en este caso.

Si mueve usuarios entre bosques y dominios, debe buscar un atributo que no cambie o se puede mover con los usuarios durante el movimiento. Un enfoque recomendado es introducir un atributo sintético. Un atributo que pueda contener algo parecido a un GUID sería adecuado. Durante la creación del objeto se crea un nuevo GUID y se marca en el usuario. Puede crear una regla de sincronización en el servidor del motor de sincronización para crear este valor según el **objectGUID** y actualizar el atributo seleccionado en ADDS. Al mover el objeto, asegúrese de copiar también el contenido de este valor.

Otra solución consiste en seleccionar un atributo existente que sepa que no va a cambiar. Uno de los atributos de uso más común es **employeeID**. Si está considerando el uso de un atributo que contenga letras, asegúrese de que no haya ninguna posibilidad de que la distinción de mayúsculas y minúsculas pueda cambiar el valor del atributo. Entre los atributos incorrectos que no deben usarse se incluyen aquellos con el nombre del usuario. En un matrimonio o divorcio se espera que cambie el nombre, lo que no está permitido para este atributo. Este también es uno de los motivos de que atributos como **userPrincipalName**, **mail** y **targetAddress** no se puedan seleccionar en el asistente de instalación de Azure AD Connect. Esos atributos también contienen el carácter "\@", que no está permitido en sourceAnchor.

### <a name="changing-the-sourceanchor-attribute"></a>Cambio del atributo sourceAnchor
No se puede cambiar el valor del atributo sourceAnchor después de que el objeto se ha creado en Azure AD y la identidad se ha sincronizado.

Por este motivo, se aplican las restricciones siguientes a Azure AD Connect:

* El atributo sourceAnchor solo puede establecerse durante la instalación inicial. Si vuelve a ejecutar el asistente de instalación, esta opción es de solo lectura. Si necesita cambiar esto, debe desinstalar y reinstalar la aplicación.
* Si instala otro servidor de Azure AD Connect, debe seleccionar el mismo atributo sourceAnchor usado anteriormente. Si anteriormente ha estado usando la sincronización de directorios y se pasa a Azure AD Connect, debe usar **objectGUID** ya que es el atributo de sincronización de directorios.
* Si se cambia el valor de sourceAnchor después de que el objeto se ha exportado a Azure AD, Azure AD Connect Sync produce un error y no permite más cambios en ese objeto antes de que el problema se haya corregido y el atributo sourceAnchor cambie de nuevo en el directorio de origen.

## <a name="using-ms-ds-consistencyguid-as-sourceanchor"></a>Uso de msDS-ConsistencyGuid como sourceAnchor
De forma predeterminada, Azure AD Connect (versión 1.1.486.0 y anteriores) usa objectGUID como atributo sourceAnchor. objectGUID es genera en el sistema. No puede especificar su valor al crear objetos de AD locales. Como se explica en la sección [sourceAnchor](#sourceanchor), en algunos escenarios es necesario especificar el valor de sourceAnchor. Si es su caso, debe usar un atributo de AD configurable (por ejemplo, ms-DS-ConsistencyGuid) como atributo sourceAnchor.

Ahora, Azure AD Connect (versión 1.1.524.0 y posteriores) facilita el uso de ms-DS-ConsistencyGuid como atributo sourceAnchor. Al utilizar esta característica, Azure AD Connect configura automáticamente las reglas de sincronización para:

1. Usar ms-DS-ConsistencyGuid como atributo sourceAnchor para los objetos de usuario. objectGUID se usa para otros tipos de objeto.

2. Para cualquier objeto de usuario de AD local cuyo atributo ms-DS-ConsistencyGuid no esté rellenado, Azure AD Connect escribe su propio valor objectGUID en el atributo ms-DS-ConsistencyGuid de la instancia local de Active Directory. Después de rellenar el atributo ms-DS-ConsistencyGuid, Azure AD Connect exporta el objeto a Azure AD.

>[!NOTE]
> Una vez que se haya importado un objeto de AD local en Azure AD Connect (es decir, que se haya importado en el espacio conector de AD y se haya proyectado en el metaverso), ya no se puede cambiar su valor para sourceAnchor. Para especificar el valor de sourceAnchor para un objeto de AD local determinado, configure el atributo ms-DS-ConsistencyGuid antes de importarlo a Azure AD Connect.

### <a name="permission-required"></a>Permiso necesario
Para que funcione esta característica, se debe conceder a la cuenta de AD DS que se utiliza para sincronizar con la instancia local de Active Directory permiso de escritura en el atributo ms-DS-ConsistencyGuid de la instancia local de Active Directory.

### <a name="how-to-enable-the-consistencyguid-feature---new-installation"></a>Habilitación de la característica ConsistencyGuid: nueva instalación
Puede habilitar el uso de ConsistencyGuid como sourceAnchor durante la nueva instalación. En esta sección se trata la instalación rápida y personalizada con detalle.

  >[!NOTE]
  > Solo las versiones más recientes de Azure AD Connect (1.1.524.0 y posteriores) admiten el uso de ConsistencyGuid como sourceAnchor durante la nueva instalación.

### <a name="how-to-enable-the-consistencyguid-feature"></a>Habilitación de la característica ConsistencyGuid

#### <a name="express-installation"></a>Instalación rápida
Cuando se instala Azure AD Connect con el modo Rápido, el asistente de Azure AD Connect determina automáticamente el atributo de AD más adecuado para usarse como atributo sourceAnchor según la lógica siguiente:

* En primer lugar, el asistente de Azure AD Connect consulta el inquilino de Azure AD para recuperar el atributo de AD usado como atributo sourceAnchor en la anterior instalación de Azure AD Connect (de haberla). Si esta información está disponible, Azure AD Connect usa el mismo atributo de AD.

  >[!NOTE]
  > Las versiones más recientes de Azure AD Connect (1.1.524.0 y posteriores) son las únicas que almacenan información en su inquilino de Azure AD sobre el atributo sourceAnchor usado durante la instalación. Las versiones anteriores de Azure AD Connect no lo hacen.

* Si la información sobre el atributo sourceAnchor usado no está disponible, el asistente comprueba el estado del atributo ms-DS-ConsistencyGuid en su Active Directory local. Si el atributo no está configurado en ningún objeto del directorio, el asistente usa ms-DS-ConsistencyGuid como atributo sourceAnchor. Si el atributo está configurado en uno o varios objetos del directorio, el asistente concluye que el atributo se está usando en otras aplicaciones y no es adecuado como atributo sourceAnchor…

* En ese caso, el asistente recurre de nuevo a usar objectGUID como atributo sourceAnchor.

* Una vez que decide el atributo sourceAnchor, el asistente almacena la información en su inquilino de Azure AD. Se usará la información en una instalación futura de Azure AD Connect.

Una vez completada la instalación rápida, el asistente le informa de qué atributo se ha elegido como atributo sourceAnchor.

![El asistente informa del atributo de AD seleccionado como sourceAnchor](./media/plan-connect-design-concepts/consistencyGuid-01.png)

#### <a name="custom-installation"></a>Instalación personalizada
Cuando se instala Azure AD Connect con el modo Personalizado, el asistente de Azure AD Connect proporciona dos opciones al configurar el atributo sourceAnchor:

![Instalación personalizada: configuración de sourceAnchor](./media/plan-connect-design-concepts/consistencyGuid-02.png)

| Configuración | Descripción |
| --- | --- |
| Let Azure manage the source anchor for me (Dejar que Azure administre automáticamente el delimitador de origen) | Seleccione esta opción si desea que Azure AD elija automáticamente el atributo. Si selecciona esta opción, el asistente de Azure AD Connect aplica la misma [lógica de selección del atributo sourceAnchor usada durante la instalación rápida](#express-installation). De forma parecida a la instalación rápida, el asistente indica qué atributo se ha elegido como atributo sourceAnchor una vez finalizada la instalación personalizada. |
| Un atributo específico | Seleccione esta opción si desea especificar un atributo existente de AD como atributo sourceAnchor. |

### <a name="how-to-enable-the-consistencyguid-feature---existing-deployment"></a>Habilitación de la característica ConsistencyGuid: implementación existente
Si tiene una implementación de Azure AD Connect existente que está usando objectGUID como el atributo sourceAnchor, puede cambiarla para usar ConsistencyGuid en su lugar.

>[!NOTE]
> Solo las versiones más recientes de Azure AD Connect (1.1.552.0 y posteriores) admiten el cambio de ObjectGuid a ConsistencyGuid como el atributo sourceAnchor.

Para cambiar de objectGUID a ConsistencyGuid como el atributo sourceAnchor:

1. Inicie el asistente de Azure AD Connect y haga clic en **Configurar** para ir a la pantalla de tareas.

2. Seleccione la opción de tarea **Configuración del delimitador de origen** y haga clic en **Siguiente**.

   ![Habilitación de ConsistencyGuid para la implementación existente: paso 2](./media/plan-connect-design-concepts/consistencyguidexistingdeployment01.png)

3. Escriba sus credenciales de administrador de Azure AD y haga clic en **Siguiente**.

4. El asistente de Azure AD Connect analiza el estado del atributo ms-DS-ConsistencyGuid en Active Directory local. Si el atributo no está configurado en ningún objeto del directorio, Azure AD Connect concluye que ninguna otra aplicación está usando en estos momentos el atributo y es seguro usarlo como el atributo sourceAnchor. Haga clic en **Siguiente** para continuar.

   ![Habilitación de ConsistencyGuid para la implementación existente: paso 4](./media/plan-connect-design-concepts/consistencyguidexistingdeployment02.png)

5. En la pantalla **Listo para configurar**, haga clic en **Configurar** para que la configuración cambie.

   ![Habilitación de ConsistencyGuid para la implementación existente: paso 5](./media/plan-connect-design-concepts/consistencyguidexistingdeployment03.png)

6. Una vez que se complete la configuración, el asistente indica que ms-DS-ConsistencyGuid ahora se usa como el atributo sourceAnchor.

   ![Habilitación de ConsistencyGuid para la implementación existente: paso 6](./media/plan-connect-design-concepts/consistencyguidexistingdeployment04.png)

Durante el análisis (paso 4), si el atributo está configurado en uno o varios objetos del directorio, el asistente concluye que el atributo se está usando en otra aplicación y devuelve un error, como se ilustra en la imagen siguiente. Este error también puede producirse si se ha habilitado previamente la característica ConsistencyGuid en el servidor de Azure AD Connect principal y se intenta hacer lo mismo en el servidor de almacenamiento provisional.

![Habilitación de ConsistencyGuid para la implementación existente: error](./media/plan-connect-design-concepts/consistencyguidexistingdeploymenterror.png)

 Si está seguro de que otras aplicaciones existentes no usan el atributo, para suprimir el error, reinicie el Asistente de Azure AD Connect con el modificador **/SkipLdapSearch** especificado. Para ello, ejecute el comando siguiente en el símbolo del sistema:

```
"c:\Program Files\Microsoft Azure Active Directory Connect\AzureADConnect.exe" /SkipLdapSearch
```

### <a name="impact-on-ad-fs-or-third-party-federation-configuration"></a>Repercusiones en AD FS o la configuración de la federación de terceros
Si usa Azure AD Connect para administrar una implementación de AD FS local, Azure AD Connect actualiza automáticamente las reglas de notificación para que usen el mismo atributo de AD como sourceAnchor. Esto garantiza que la notificación de ImmutableID generada por AD FS sea coherente con los valores de sourceAnchor que se exportan a Azure AD.

Si administra AD FS fuera de Azure AD Connect o usa servidores de federación de terceros para la autenticación, debe actualizar manualmente las reglas de notificación para que la notificación de ImmutableID sea coherente con los valores de sourceAnchor exportados a Azure AD, como se describe en la sección del artículo [Modificación de las reglas de notificaciones de AD FS](./how-to-connect-fed-management.md#modclaims). El asistente devuelve la siguiente advertencia al completarse la instalación:

![Configuración de federación de terceros](./media/plan-connect-design-concepts/consistencyGuid-03.png)

### <a name="adding-new-directories-to-existing-deployment"></a>Incorporación de nuevos directorios a la implementación existente
Suponga que ha implementado Azure AD Connect con la característica ConsistencyGuid habilitada y ahora desea agregar otro directorio a la implementación. Cuando intenta agregarlo, el asistente de Azure AD Connect comprueba el estado del atributo ms-DS-ConsistencyGuid en el directorio. Si el atributo está configurado en uno o varios objetos del directorio, el asistente concluye que el atributo se está usando en otras aplicaciones y devuelve un error, como se ilustra en la imagen siguiente. Si está seguro de que las aplicaciones existentes no usan el atributo, para suprimir el error, reinicie el Asistente de Azure AD Connect con el modificador **/SkipLdapSearch** especificado tal como se describe anteriormente o debe ponerse en contacto con el servicio de soporte técnico para más información.

![Incorporación de nuevos directorios a la implementación existente](./media/plan-connect-design-concepts/consistencyGuid-04.png)

## <a name="azure-ad-sign-in"></a>Inicio de sesión de Azure AD
Al integrar su directorio local con Azure AD, es importante entender cómo la configuración de sincronización puede afectar a la forma en la que los usuarios se autentican. Azure AD usa userPrincipalName (UPN) para autenticar al usuario. Sin embargo, al sincronizar los usuarios tiene que elegir el atributo que se utilizará para el valor userPrincipalName cuidadosamente.

### <a name="choosing-the-attribute-for-userprincipalname"></a>Selección del atributo para userPrincipalName
Cuando se selecciona el atributo para proporcionar el valor de UPN que se va a usar en Azure es necesario asegurarse de lo siguiente:

* Los valores de atributo se ajustan a la sintaxis UPN (RFC 822), es decir, debe tener el formato nombreUsuario\@dominio
* El sufijo en los valores coincide con uno de los dominios personalizados comprobados en Azure AD

En la configuración rápida, la opción que se presupone para el atributo es userPrincipalName. Si el atributo userPrincipalName no contiene el valor que quiera que los usuarios inicien sesión en Azure, debe elegir la **instalación personalizada**.

### <a name="custom-domain-state-and-upn"></a>Estado de dominio personalizado y UPN
Es importante asegurarse de que hay un dominio comprobado para el sufijo UPN.

John es un usuario de "contoso.com". Desea que John use el UPN local john\@contoso.com para iniciar sesión en Azure después haber sincronizado a los usuarios con el directorio de Azure AD azurecontoso.onmicrosoft.com. Para ello, tiene que agregar y comprobar "contoso.com" como un dominio personalizado en Azure AD antes de sincronizar a los usuarios. Si el sufijo UPN de John, por ejemplo, contoso.com, no coincide con un dominio comprobado en Azure AD, Azure AD reemplaza el sufijo UPN con contoso.onmicrosoft.com.

### <a name="non-routable-on-premises-domains-and-upn-for-azure-ad"></a>Dominios locales no enrutables y UPN para Azure AD
Algunas organizaciones tienen dominios no enrutables, como "contoso.local" o dominios de etiqueta única simple como "contoso". No podrá comprobar un dominio no enrutable en Azure AD. Azure AD Connect puede sincronizarse solo con un dominio comprobado en Azure AD. Cuando se crea un directorio de Azure AD, se crea un dominio enrutable que se convierte en el dominio predeterminado para Azure SD, por ejemplo, "contoso.onmicrosoft.com". Por lo tanto, es necesario comprobar cualquier otro dominio enrutable en este escenario, en caso de que no desee sincronizar con el valor predeterminado ".onmicrosoft.com".

Consulte [Incorporación de su nombre de dominio personalizado a Azure Active Directory](../fundamentals/add-custom-domain.md) para más información sobre cómo agregar y comprobar dominios.

Azure AD Connect detecta si está ejecutando en un entorno de dominio no enrutable y le advertirá adecuadamente antes de que prosiga con la configuración rápida. Si está trabajando en un dominio no enrutable, es probable que el UPN de los usuarios tenga también un sufijo no enrutable. Por ejemplo, si se ejecuta bajo "contoso.local", Azure AD Connect le sugiere que utilice la configuración personalizada en lugar de la configuración rápida. Al usar una configuración personalizada, podrá especificar el atributo que debe usarse como UPN para iniciar sesión en Azure después de que los usuarios se sincronicen con Azure AD.

## <a name="next-steps"></a>Pasos siguientes
Obtenga más información sobre la [Integración de las identidades locales con Azure Active Directory](whatis-hybrid-identity.md).