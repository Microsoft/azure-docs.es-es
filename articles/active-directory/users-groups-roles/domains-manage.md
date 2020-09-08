---
title: Incorporación y verificación de nombres de dominio personalizados en Azure Active Directory | Microsoft Docs
description: Conceptos y procedimientos de administración de un nombre de dominio en Azure Active Directory
services: active-directory
documentationcenter: ''
author: curtand
manager: mtillman
ms.service: active-directory
ms.workload: identity
ms.subservice: users-groups-roles
ms.topic: how-to
ms.date: 01/31/2019
ms.author: curtand
ms.reviewer: elkuzmen
ms.custom: it-pro
ms.collection: M365-identity-device-management
ms.openlocfilehash: 12193b469b6e1f014e6c948da7b2adb370e78d4b
ms.sourcegitcommit: c5021f2095e25750eb34fd0b866adf5d81d56c3a
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/25/2020
ms.locfileid: "88795341"
---
# <a name="managing-custom-domain-names-in-your-azure-active-directory"></a>Administración de los nombres de dominio personalizados en Azure Active Directory

Un nombre de dominio es una parte importante del identificador para muchos recursos de directorio: forma parte de un nombre de usuario o una dirección de correo electrónico de un usuario, forma parte de la dirección para un grupo y puede formar parte del URI del identificador de una aplicación. Un recurso en Azure Active Directory (Azure AD) puede incluir un nombre de dominio que pertenezca al directorio que contiene el recurso. Solo los administradores globales pueden administrar dominios en Azure AD.

## <a name="set-the-primary-domain-name-for-your-azure-ad-directory"></a>Establecimiento del nombre de dominio principal para su directorio de Azure AD

Cuando se crea el directorio, el nombre de dominio inicial, por ejemplo, contoso.onmicrosoft.com, también será el nombre de dominio principal. El dominio principal será el nombre de dominio predeterminado de un nuevo usuario cuando este se cree. Establecer un nombre de dominio primario simplifica el proceso para que un administrador cree usuarios nuevos en el portal. Para cambiar el nombre de dominio principal, siga estos pasos:

1. Inicie sesión en [Azure Portal](https://portal.azure.com) con una cuenta que tenga el rol de administrador global para el directorio.
2. Seleccione **Azure Active Directory**.
3. Seleccione **Nombres de dominio personalizados**.
  
   ![Abrir la página de administración de usuarios](./media/domains-manage/add-custom-domain.png)
4. Seleccione el nombre del dominio que quiere que sea el dominio principal.
5. Seleccione el comando **Convertir en principal**. Confirme la elección cuando se le pregunte.
  
   ![Convertir un nombre de dominio en principal](./media/domains-manage/make-primary-domain.png)

Puede cambiar el nombre de dominio principal para el directorio de modo que sea cualquier dominio personalizado verificado que no esté federado. El hecho de cambiar el dominio principal para el directorio no cambiará los nombres de usuario existentes.

## <a name="add-custom-domain-names-to-your-azure-ad-organization"></a>Adición de nombres de dominio personalizados a la organización de Azure AD

Puede agregar un máximo de 900 nombres de dominio administrados. Si quiere configurar todos los dominios para la federación con una instancia local de Active Directory, puede agregar un máximo de 450 nombres de dominio en cada directorio.

## <a name="add-subdomains-of-a-custom-domain"></a>Adición de subdominios de un dominio personalizado

Si desea agregar un nombre de dominio de tercer nivel, como "europe.contoso.com", a su directorio, primero debe agregar y comprobar el dominio de segundo nivel, como contoso.com. Azure AD verificará automáticamente el subdominio. Para ver que el subdominio que ha agregado se ha verificado, actualice la lista de dominios en el explorador.

Nota:

Si ya ha agregado un dominio de contoso.com a un inquilino de Azure AD, también puede Agregar el subdominio europe.contoso.com a un segundo inquilino de Azure AD. Al agregar el subdominio, se le pedirá que agregue un registro TXT en el proveedor de hospedaje DNS.

## <a name="what-to-do-if-you-change-the-dns-registrar-for-your-custom-domain-name"></a>Qué hacer si se cambia el registrador DNS del nombre de dominio personalizado

Si cambia a los registradores DNS más habituales, no hay ninguna tarea de configuración adicional en Azure AD. Puede continuar usando el nombre de dominio con Azure AD sin interrupciones. Si usa el nombre de dominio personalizado con Office 365, Intune u otros servicios que dependan de los nombres de dominio personalizados en Azure AD, consulte la documentación de dichos servicios.

## <a name="delete-a-custom-domain-name"></a>Eliminación de un nombre de dominio personalizado

Puede eliminar un nombre de dominio personalizado de Azure AD si su organización ya no utiliza ese nombre de dominio o si necesita utilizar ese nombre de dominio con otro Azure AD.

Para eliminar un nombre de dominio personalizado, primero debe asegurarse de que no haya recursos en el directorio que se basen en el nombre de dominio. No puede eliminar un nombre de dominio de su directorio si:

* Algún usuario tiene un nombre de usuario, una dirección de correo electrónico o una dirección de proxy que incluyan el nombre de dominio.
* Algún grupo tiene una dirección de correo electrónico o una dirección de proxy que incluya el nombre de dominio.
* Alguna aplicación en Azure AD tiene un URI de identificador de aplicación que incluya el nombre de dominio.

Debe cambiar o eliminar dichos recursos en su directorio Azure AD para poder eliminar el nombre de dominio personalizado.

### <a name="forcedelete-option"></a>Opción ForceDelete

Puede usar la operación **ForceDelete** en un nombre de dominio en el [Centro de administración de Azure AD](https://aad.portal.azure.com) o mediante [Microsoft Graph API](/graph/api/domain-forcedelete?view=graph-rest-beta). Estas opciones usan una operación asincrónica y actualizan todas las referencias del nombre de dominio personalizado, como "user@contoso.com", al nombre de dominio predeterminado inicial, como "user@contoso.onmicrosoft.com". 

Para llamar a **ForceDelete** en Azure Portal, debe asegurarse de que hay menos de 1000 referencias en el nombre de dominio, y todas las referencias en las que Exchange sea el servicio de aprovisionamiento deben actualizarse o quitarse del [Centro de administración de Exchange](https://outlook.office365.com/ecp/). Esto incluye los grupos de seguridad habilitados para el correo electrónico de Exchange y las listas distribuidas. Para obtener más información, consulte [Quitar grupos de seguridad habilitados para correo](/Exchange/recipients/mail-enabled-security-groups?view=exchserver-2019#Remove%20mail-enabled%20security%20groups). Además, la operación **ForceDelete** no se realizará correctamente si se cumple alguna de las siguientes condiciones:

* Adquirió un dominio a través de servicios de suscripción del dominio de Office 365.
* Es un asociado de administración en nombre de otra organización del cliente.

Las siguientes acciones se realizan como parte de la operación **ForceDelete**:

* Cambia el nombre de UPN, la dirección de correo electrónico y la dirección proxy de los usuarios con referencias al nombre de dominio personalizado al nombre de dominio predeterminado inicial.
* Cambia el nombre de la dirección de correo electrónico de los usuarios con referencias al nombre de dominio personalizado al nombre de dominio predeterminado inicial.
* Cambia el nombre de los URI de identificador de los usuarios con referencias al nombre de dominio personalizado al nombre de dominio predeterminado inicial.

Se devuelve el error cuando:

* El número de objetos cuyo nombre se va a cambiar es mayor que 1000.
* Una de las aplicaciones cuyo nombre se va a cambiar es una aplicación de varios inquilinos.

### <a name="frequently-asked-questions"></a>Preguntas más frecuentes

**P: ¿Por qué la eliminación del dominio produce un error que indica que tengo grupos controlados de Exchange en este nombre de dominio?** <br>
**R:** Actualmente, Exchange aprovisiona ciertos grupos, como los grupos de seguridad habilitados para correo y las listas distribuidas, que se deben limpiar manualmente en el [Centro de administración de Exchange (EAC)](https://outlook.office365.com/ecp/). Es posible que haya direcciones de proxy que confíen en el nombre de dominio personalizado y deberán actualizarse manualmente a otro nombre de dominio. 

**P: Tengo la sesión iniciada como admin\@contoso.com, pero no puedo eliminar el nombre de dominio "contoso.com".**<br>
**R:** No puede hacer referencia al nombre de dominio personalizado que intenta eliminar en su nombre de cuenta de usuario. Asegúrese de que la cuenta de administrador global use el nombre de dominio predeterminado inicial (. onmicrosoft.com), como admin@contoso.onmicrosoft.com. Inicie sesión con otra cuenta de administrador global, como admin@contoso.onmicrosoft.com, u otro nombre de dominio personalizado, como "fabrikam.com" donde la cuenta es admin@fabrikam.com.

**P: He hecho clic en el botón Eliminar dominio y veo el estado `In Progress` de la operación de eliminación. ¿Cuánto tiempo tarda? ¿Qué pasa si se produce un error?**<br>
**R:** La operación de eliminación de un dominio es una tarea asincrónica que se realiza en segundo plano y cambia el nombre de todas las referencias al nombre de dominio. Debería completarse en un par de minutos. Si se produce un error en la eliminación del dominio, asegúrese de no tener:

* Aplicaciones configuradas en el nombre de dominio con appIdentifierURI
* Ningún grupo habilitado para correo que haga referencia el nombre de dominio personalizado
* Más de 1000 referencias al nombre de dominio

Si encuentra que alguna de las condiciones no se cumplen, limpie manualmente las referencias y vuelva a intentar eliminar el dominio.

## <a name="use-powershell-or-the-microsoft-graph-api-to-manage-domain-names"></a>Utilización de PowerShell o Microsoft Graph API para administrar nombres de dominio

También se pueden completar la mayoría de las tareas de administración para los nombres de dominio de Azure Active Directory mediante Microsoft PowerShell, o mediante programación utilizando Microsoft Graph API.

* [Utilización de PowerShell para administrar los nombres de dominio en Azure AD](/powershell/module/azuread/?view=azureadps-2.0#domains)
* [Tipo de recurso de dominio](/graph/api/resources/domain?view=graph-rest-1.0)

## <a name="next-steps"></a>Pasos siguientes

* [Agregar nombres de dominio personalizados](../fundamentals/add-custom-domain.md?context=azure%2factive-directory%2fusers-groups-roles%2fcontext%2fugr-context)
* [Quitar grupos de seguridad habilitados para correo de Exchange en el Centro de administración de Exchange en un nombre de dominio personalizado en Azure AD](/Exchange/recipients/mail-enabled-security-groups?view=exchserver-2019#Remove%20mail-enabled%20security%20groups)
* [ForceDelete a custom domain name with Microsoft Graph API](/graph/api/domain-forcedelete?view=graph-rest-beta) (Uso de la operación ForceDelete en un nombre de dominio personalizado con Microsoft Graph API)