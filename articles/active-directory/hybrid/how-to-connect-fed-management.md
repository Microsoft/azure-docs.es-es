---
title: 'Azure AD Connect: administración y personalización de AD FS | Microsoft Docs'
description: Administración de AD FS con Azure AD Connect y personalización del inicio de sesión de AD FS del usuario con Azure AD Connect y PowerShell.
keywords: AD FS, ADFS, administración de AD FS, AAD Connect, Connect, inicio de sesión, personalización de AD FS, reparación de la confianza, Office 365, federación, usuario de confianza
services: active-directory
documentationcenter: ''
author: billmath
manager: daveba
editor: ''
ms.assetid: 2593b6c6-dc3f-46ef-8e02-a8e2dc4e9fb9
ms.service: active-directory
ms.workload: identity
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: how-to
ms.date: 07/18/2017
ms.subservice: hybrid
ms.author: billmath
ms.custom: seohack1
ms.collection: M365-identity-device-management
ms.openlocfilehash: 72339d9ce492446d05f23b63ac2d0a1c2d62f766
ms.sourcegitcommit: c94a177b11a850ab30f406edb233de6923ca742a
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/01/2020
ms.locfileid: "89276786"
---
# <a name="manage-and-customize-active-directory-federation-services-by-using-azure-ad-connect"></a>Administre y personalice Servicios de federación de Active Directory con Azure AD Connect
En este artículo se describe cómo administrar y personalizar Servicios de federación de Active Directory (AD FS) con Azure Active Directory (Azure AD) Connect. También se incluyen otras tareas comunes de AD FS que podría tener que hacer para configurar completamente una granja de servidores de AD FS.

| Tema | Qué trata |
|:--- |:--- |
| **Administración de AD FS** | |
| [Reparación de la confianza](#repairthetrust) |Cómo reparar la confianza de federación con Office 365. |
| [Federación con Azure AD mediante un identificador de inicio de sesión alternativo](#alternateid) | Configuración de la federación con un identificador de inicio de sesión alternativo  |
| [Adición de un servidor de AD FS](#addadfsserver) |Cómo expandir una granja de servidores de AD FS con un servidor de AD FS adicional. |
| [Incorporación de un servidor proxy de aplicación web de AD FS](#addwapserver) |Cómo expandir una granja de servidores de AD FS con un servidor proxy de aplicación web (WAP) adicional. |
| [Adición de un dominio federado](#addfeddomain) |Cómo agregar un dominio federado. |
| [Actualización del certificado TLS/SSL](how-to-connect-fed-ssl-update.md)| Cómo actualizar el certificado TLS/SSL de una granja de servidores de AD FS. |
| **Personalización de AD FS** | |
| [Adición de un logotipo de la compañía personalizado o una ilustración](#customlogo) |Cómo personalizar una página de inicio de sesión de AD FS con una ilustración y un logotipo de la empresa. |
| [Adición de la descripción de inicio de sesión](#addsignindescription) |Cómo agregar una descripción a la página de inicio de sesión. |
| [Modificación de las reglas de notificaciones de AD FS](#modclaims) |Cómo modificar las notificaciones de AD FS en diversos escenarios de federación. |

## <a name="manage-ad-fs"></a>Administración de AD FS
Puede realizar diversas tareas relacionadas con AD FS en Azure AD Connect con mínima intervención por parte del usuario mediante el Asistente para Azure AD Connect. Una vez finalizada la instalación de Azure AD Connect con el asistente, puede ejecutarlo de nuevo para realizar tareas adicionales.

## <a name="repair-the-trust"></a><a name="repairthetrust"></a>Reparar la relación de confianza 
Puede usar Azure AD Connect para comprobar el estado actual de la confianza de AD FS y Azure AD, y tomar las medidas adecuadas para repararla. Siga estos pasos para reparar la confianza de Azure AD y AD FS.

1. Seleccione **Reparar AAD y confianza de ADFS** en la lista de tareas disponibles.
   ![Reparar AAD y confianza de ADFS](./media/how-to-connect-fed-management/RepairADTrust1.PNG)

2. En la página **Conectar con Azure AD**, proporcione las credenciales de administrador global de Azure AD y haga clic en **Siguiente**.
   ![Conectarse a Azure AD](./media/how-to-connect-fed-management/RepairADTrust2.PNG)

3. En la página **Credenciales de acceso remoto** , proporcione las credenciales del administrador de dominio.

   ![Credenciales de acceso remoto](./media/how-to-connect-fed-management/RepairADTrust3.PNG)

    Cuando hace clic en **Siguiente**, Azure AD Connect comprueba el estado del certificado y muestra los problemas que existen.

    ![Estado de los certificados](./media/how-to-connect-fed-management/RepairADTrust4.PNG)

    La página **Listo para configurar** muestra la lista de acciones que se llevarán a cabo para reparar la confianza.

    ![Listo para configurar](./media/how-to-connect-fed-management/RepairADTrust5.PNG)

4. Haga clic en **Instalar** para reparar la confianza.

> [!NOTE]
> Azure AD Connect solo puede reparar o actuar en los certificados autofirmados. Azure AD Connect no puede reparar certificados de terceros.

## <a name="federate-with-azure-ad-using-alternateid"></a><a name="alternateid"></a>Federación con Azure AD mediante AlternateID 
Se recomienda que el nombre principal de usuario (UPN) local y el nombre principal de usuario en la nube sean el mismo. Si el UPN local usa un dominio no enrutable (p. ej. Contoso.local) o no se puede cambiar debido a dependencias de aplicación locales, se recomienda configurar el identificador de inicio de sesión alternativo. El identificador de inicio de sesión alternativo le permite configurar una experiencia de inicio de sesión donde los usuarios puedan iniciar sesión con un atributo que no sea su UPN, por ejemplo, el correo electrónico. La opción predeterminada de nombre principal de usuario en Azure AD Connect es el atributo userPrincipalName en Active Directory. Si elige cualquier otro atributo como nombre principal de usuario y está realizando una federación mediante AD FS, Azure AD Connect configurará AD FS para el identificador de inicio de sesión alternativo. A continuación, se muestra un ejemplo de elección de un atributo diferente para el nombre principal de usuario:

![Selección de atributo de identificador alternativo](./media/how-to-connect-fed-management/attributeselection.png)

La configuración del identificador de inicio de sesión alternativo para AD FS consta de dos pasos principales:
1. **Configurar el conjunto de notificaciones de emisión correcto**: Las reglas de notificación de emisiones del usuario de confianza de Azure AD se modifican para usar el atributo UserPrincipalName seleccionado como identificador alternativo del usuario.
2. **Habilitar el identificador de inicio de sesión alternativo en la configuración de AD FS**: Se actualiza la configuración de AD FS para que AD FS pueda buscar usuarios en los bosques correspondientes con el identificador alternativo. Esta configuración se admite en AD FS en Windows Server 2012 R2 (con KB2919355) o versiones posteriores. Si los servidores de AD FS son 2012 R2, Azure AD Connect comprueba si está presente la KB necesaria. Si no se detecta la KB, se mostrará una advertencia cuando finalice la configuración, tal y como se muestra a continuación:

    ![Advertencia de que falta la KB en 2012 R2](./media/how-to-connect-fed-management/kbwarning.png)

    Para corregir la configuración en caso de que falte la KB, instale [KB2919355](https://go.microsoft.com/fwlink/?LinkID=396590) y, a continuación, repare la confianza utilizando [Reparar AAD y confianza de ADFS ](#repairthetrust).

> [!NOTE]
> Para más información sobre alternateID y los pasos para configurarlo manualmente, consulte [Configuración del identificador de inicio de sesión alternativo](/windows-server/identity/ad-fs/operations/configuring-alternate-login-id)

## <a name="add-an-ad-fs-server"></a><a name="addadfsserver"></a>Adición de un servidor de AD FS 

> [!NOTE]
> Para agregar un servidor de AD FS, Azure AD Connect necesita el certificado PFX. Por lo tanto, puede realizar esta operación solamente si ha configurado la granja de servidores de AD FS con Azure AD Connect.

1. Seleccione **Implementar un servidor de federación adicional** y haga clic en **Siguiente**.

   ![Servidor de federación adicional](./media/how-to-connect-fed-management/AddNewADFSServer1.PNG)

2. En la página **Conectar con Azure AD**, proporcione las credenciales de administrador global de Azure AD y haga clic en **Siguiente**.

   ![Conectarse a Azure](./media/how-to-connect-fed-management/AddNewADFSServer2.PNG)

3. Proporcione las credenciales del administrador de dominio.

   ![Credenciales del administrador de dominio](./media/how-to-connect-fed-management/AddNewADFSServer3.PNG)

4. Azure AD Connect le pide la contraseña del archivo PFX que proporcionó al configurar la nueva granja de servidores de AD FS con Azure AD Connect. Haga clic en **Escribir contraseña** para proporcionar la contraseña del archivo PFX.

   ![Contraseña del certificado](./media/how-to-connect-fed-management/AddNewADFSServer4.PNG)

    ![Especificar un certificado TLS/SSL](./media/how-to-connect-fed-management/AddNewADFSServer5.PNG)

5. En la página **Servidores de AD FS** , escriba el nombre del servidor o la dirección IP que se agregarán a la granja de servidores de AD FS.

   ![Servidores de AD FS](./media/how-to-connect-fed-management/AddNewADFSServer6.PNG)

6. Haga clic en **Siguiente** y recorra la página **Configurar** final. Una vez que Azure AD Connect haya terminado de agregar los servidores a la granja de servidores de AD FS, se le ofrecerá la opción de comprobar la conectividad.

   ![Listo para configurar](./media/how-to-connect-fed-management/AddNewADFSServer7.PNG)

    ![Instalación completada](./media/how-to-connect-fed-management/AddNewADFSServer8.PNG)

## <a name="add-an-ad-fs-wap-server"></a><a name="addwapserver"></a>Adición de un servidor de AD FS WAP 

> [!NOTE]
> Para agregar un servidor WAP, Azure AD Connecte necesita el certificado PFX. Por lo tanto, puede realizar esta operación solamente si ha configurado la granja de servidores de AD FS con Azure AD Connect.

1. Seleccione **Implementar proxy de aplicación web** en la lista de tareas disponibles.

   ![Implementación de Proxy de aplicación web](./media/how-to-connect-fed-management/WapServer1.PNG)

2. Proporcione las credenciales del administrador global de Azure.

   ![Conectarse a Azure](./media/how-to-connect-fed-management/wapserver2.PNG)

3. En la página **Especificar el certificado SSL**, indique la contraseña para el archivo PFX que proporcionó cuando configuró la granja de servidores de AD FS con Azure AD Connect.
   ![Contraseña del certificado](./media/how-to-connect-fed-management/WapServer3.PNG)

    ![Especificar un certificado TLS/SSL](./media/how-to-connect-fed-management/WapServer4.PNG)

4. Incluya el servidor que se agregará como servidor WAP. Como el servidor WAP puede no estar unido al dominio, se le pedirán credenciales administrativas para el servidor que se va a agregar.

   ![Credenciales administrativas del servidor](./media/how-to-connect-fed-management/WapServer5.PNG)

5. En la página **Credenciales de confianza del proxy**, proporcione credenciales administrativas para configurar la confianza del proxy y tener acceso al servidor principal en la granja de servidores de AD FS.

   ![Credenciales de confianza del proxy](./media/how-to-connect-fed-management/WapServer6.PNG)

6. En la página **Listo para configurar**, el asistente muestra la lista de acciones que se realizarán.

   ![Listo para configurar](./media/how-to-connect-fed-management/WapServer7.PNG)

7. Haga clic en **Instalar** para finalizar la configuración. Una vez completada la configuración, se le ofrece la opción de comprobar la conectividad con los servidores. Haga clic en **Comprobar** para validar la conectividad.

   ![Instalación completada](./media/how-to-connect-fed-management/WapServer8.PNG)

## <a name="add-a-federated-domain"></a><a name="addfeddomain"></a>Adición de un dominio federado 

Es fácil agregar un dominio para la federación con Azure AD mediante Azure AD Connect. Azure AD Connect agrega el dominio para la federación y modifica las reglas de notificaciones para reflejar correctamente el emisor cuando existen varios dominios federados con Azure AD.

1. Para agregar un dominio federado, seleccione la tarea **Agregar un dominio de Azure AD adicional**.

   ![Dominio de Azure AD adicional](./media/how-to-connect-fed-management/AdditionalDomain1.PNG)

2. En la siguiente página del asistente, proporcione las credenciales de administrador global para Azure AD.

   ![Conectarse a Azure](./media/how-to-connect-fed-management/AdditionalDomain2.PNG)

3. En la página **Credenciales de acceso remoto** , proporcione las credenciales del administrador del dominio.

   ![Credenciales de acceso remoto](./media/how-to-connect-fed-management/additionaldomain3.PNG)

4. En la siguiente página del asistente, aparece una lista de dominios de Azure AD con los que puede federar su directorio local. Elija el dominio en la lista.

   ![Dominio de Azure AD](./media/how-to-connect-fed-management/AdditionalDomain4.PNG)

    Después de elegir el dominio, se le proporcionará información adecuada sobre otras acciones que el asistente vaya a llevar a cabo y cómo afectarán a la configuración. En algunos casos, si selecciona un dominio que aún no está comprobado en Azure AD, el asistente le proporcionará información para comprobarlo. Consulte [Incorporación de su nombre de dominio personalizado a Azure Active Directory](../fundamentals/add-custom-domain.md) para más información.

5. Haga clic en **Next**. En la página **Listo para configurar** se muestra la lista de acciones que realizará Azure AD Connect. Haga clic en **Instalar** para finalizar la configuración.

   ![Listo para configurar](./media/how-to-connect-fed-management/AdditionalDomain5.PNG)

> [!NOTE]
> Los usuarios del dominio federado agregado deben sincronizarse para poder iniciar sesión en Azure AD.

## <a name="ad-fs-customization"></a>Personalización de AD FS
En las secciones siguientes, se proporcionan detalles sobre algunas de las tareas comunes que probablemente deba realizar cuando personalice la página de inicio de sesión de AD FS.

## <a name="add-a-custom-company-logo-or-illustration"></a><a name="customlogo"></a>Adición de un logotipo de la compañía personalizado o una ilustración 
Para cambiar el logotipo de la compañía que se muestra en la página de **inicio de sesión**, use el siguiente cmdlet de Windows PowerShell y la sintaxis.

> [!NOTE]
> Las dimensiones recomendadas para el logotipo son 260 x 35 a 96 \@ con un tamaño de archivo no superior a 10 KB.

```azurepowershell-interactive
Set-AdfsWebTheme -TargetName default -Logo @{path="c:\Contoso\logo.PNG"}
```

> [!NOTE]
> Se requiere el parámetro *TargetName* . El tema predeterminado que se incluyó con AD FS se llamada Default.

## <a name="add-a-sign-in-description"></a><a name="addsignindescription"></a>Adición de una descripción de inicio de sesión 
Para agregar una descripción de la **página de inicio de sesión**a dicha página, use el siguiente cmdlet de Windows PowerShell y la sintaxis.

```azurepowershell-interactive
Set-AdfsGlobalWebContent -SignInPageDescriptionText "<p>Sign-in to Contoso requires device registration. Click <A href='http://fs1.contoso.com/deviceregistration/'>here</A> for more information.</p>"
```

## <a name="modify-ad-fs-claim-rules"></a><a name="modclaims"></a>Modificación de las reglas de notificaciones de AD FS 
AD FS admite un amplio lenguaje de notificaciones que sirve para crear reglas de notificaciones personalizadas. Para más información, consulte [El papel de lenguaje de reglas de notificación](/previous-versions/windows/it-pro/windows-server-2012-R2-and-2012/dd807118(v=ws.11)).

En las secciones siguientes, se describe cómo escribir reglas personalizadas para algunos escenarios relacionados con la federación de Azure AD y AD FS.

### <a name="immutable-id-conditional-on-a-value-being-present-in-the-attribute"></a>Identificador inmutable dependiente de si hay un valor presente en el atributo
Azure AD Connect permite especificar un atributo que se usará como delimitador de origen cuando los objetos se sincronicen con Azure AD. Si el valor del atributo personalizado no está vacío, puede emitir una notificación de identificador inmutable.

Por ejemplo, puede seleccionar **ms-ds-consistencyguid** como atributo del delimitador de origen y emitir **ImmutableID** como **ms-ds-consistencyguid** por si el atributo tiene un valor para él. Si no hay ningún valor para el atributo, emita **objectGuid** como identificador inmutable. Puede construir el conjunto de reglas de notificaciones personalizadas, tal como se describe en la sección siguiente.

**Regla 1: Consultar atributos**

```claim-rule-language
c:[Type == "http://schemas.microsoft.com/ws/2008/06/identity/claims/windowsaccountname"]
=> add(store = "Active Directory", types = ("http://contoso.com/ws/2016/02/identity/claims/objectguid", "http://contoso.com/ws/2016/02/identity/claims/msdsconsistencyguid"), query = "; objectGuid,ms-ds-consistencyguid;{0}", param = c.Value);
```

En esta regla simplemente se consultan los valores de **ms-ds-consistencyguid** y **objectGuid** para el usuario desde Active Directory. Cambie el nombre del almacén por un nombre adecuado en la implementación de AD FS. Cambie también el tipo de notificaciones a un tipo correcto para la federación, tal como se define para los atributos **objectGuid** y **ms-ds-consistencyguid**.

Además, al usar **add** y no **issue**, no se tiene que agregar una emisión de salida para la entidad y solo se usan los valores como valores intermedios. Se emitirá la notificación en una regla posterior después de establecerse el valor que se usará como identificador inmutable.

**Regla 2: Comprobar si ms-ds-consistencyguid existe para el usuario**

```claim-rule-language
NOT EXISTS([Type == "http://contoso.com/ws/2016/02/identity/claims/msdsconsistencyguid"])
=> add(Type = "urn:anandmsft:tmp/idflag", Value = "useguid");
```

Esta regla define un marcador temporal **idflag**, que se establece en **useguid** si no hay ningún atributo **ms-ds-consistencyguid** rellenado para el usuario. Esto se debe a que ADFS no admite notificaciones vacías. De este modo, al agregar notificaciones `http://contoso.com/ws/2016/02/identity/claims/objectguid` y `http://contoso.com/ws/2016/02/identity/claims/msdsconsistencyguid` en la primera regla, solo se generará una notificación **msdsconsistencyguid** si se ha rellenado el valor del usuario. Si no se rellena, AD FS ve que tiene un valor vacío y lo coloca inmediatamente. Todos los objetos tendrán el atributo **objectGuid**, así que esa notificación seguirá estando después de que se ejecute la regla 1.

**Regla 3: Emitir el atributo ms-ds-consistencyguid como identificador inmutable si está presente**

```claim-rule-language
c:[Type == "http://contoso.com/ws/2016/02/identity/claims/msdsconsistencyguid"]
=> issue(Type = "http://schemas.microsoft.com/LiveID/Federation/2008/05/ImmutableID", Value = c.Value);
```

Se trata de una comprobación **Exist** implícita. Si el valor de la notificación existe, emítalo como identificador inmutable. En el ejemplo anterior se utiliza la notificación **nameidentifier** . Tendrá que cambiar este valor al tipo de notificación adecuado para un identificador inmutable en su entorno.

**Regla 4: Emitir el atributo objectGuid como identificador inmutable si ms-ds-consistencyguid no está presente**

```claim-rule-language
c1:[Type == "urn:anandmsft:tmp/idflag", Value =~ "useguid"]
&& c2:[Type == "http://contoso.com/ws/2016/02/identity/claims/objectguid"]
=> issue(Type = "http://schemas.microsoft.com/LiveID/Federation/2008/05/ImmutableID", Value = c2.Value);
```

En esta regla, simplemente va a comprobar el atributo **idflag**temporal. Decida si va a emitir la notificación basándose en su valor.

> [!NOTE]
> La secuencia de estas reglas es importante.

### <a name="sso-with-a-subdomain-upn"></a>SSO con un UPN de subdominio

Puede agregar más de un dominio para federarlo mediante Azure AD Connect, tal como se describe en [Incorporación de un nuevo dominio federado](how-to-connect-fed-management.md#addfeddomain). Azure AD Connect versión 1.1.553.0 y versiones posteriores crea automáticamente la regla de notificación correcta para issuerID. Si no se puede utilizar Azure AD Connect versión 1.1.553.0 o versiones más recientes, se recomienda usar la herramienta [Azure AD RPT Claim Rules](https://aka.ms/aadrptclaimrules) para generar y establecer reglas de notificación correctas para la confianza de usuarios autenticados de Azure AD.

## <a name="next-steps"></a>Pasos siguientes
Obtenga más información sobre las [opciones de inicio de sesión del usuario](plan-connect-user-signin.md).