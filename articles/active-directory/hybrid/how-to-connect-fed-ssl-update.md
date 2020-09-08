---
title: 'Azure AD Connect: actualización del certificado TLS/SSL de una granja de AD FS | Microsoft Docs'
description: En este documento se detallan los pasos para actualizar el certificado TLS/SSL de una granja de AD FS con Azure AD Connect.
services: active-directory
manager: daveba
editor: billmath
ms.assetid: 7c781f61-848a-48ad-9863-eb29da78f53c
ms.service: active-directory
ms.workload: identity
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: how-to
ms.date: 07/09/2018
ms.subservice: hybrid
author: billmath
ms.custom: seohack1
ms.author: billmath
ms.collection: M365-identity-device-management
ms.openlocfilehash: 451b50e70b98849dfc4654566d09a5a961abe451
ms.sourcegitcommit: c94a177b11a850ab30f406edb233de6923ca742a
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/01/2020
ms.locfileid: "89279914"
---
# <a name="update-the-tlsssl-certificate-for-an-active-directory-federation-services-ad-fs-farm"></a>Actualización del certificado TLS/SSL de una granja de Servicios de federación de Active Directory (AD FS)

## <a name="overview"></a>Información general
En este artículo se describe cómo se puede usar Azure AD Connect para actualizar el certificado TLS/SSL de la granja de Servicios de federación de Active Directory (AD FS). Puede usar la herramienta Azure AD Connect para actualizar fácilmente el certificado TLS/SSL de la granja de AD FS, aun cuando el método de inicio de sesión del usuario seleccionado no sea AD FS.

Puede realizar toda la operación de actualización del certificado TLS/SSL de la granja de AD FS en todos los servidores de federación y de proxy de aplicación web (WAP) en tres sencillos pasos:

![Tres pasos](./media/how-to-connect-fed-ssl-update/threesteps.png)


>[!NOTE]
>Para obtener más información sobre los certificados usados por AD FS, consulte el artículo [Descripción de los certificados usados por AD FS](/previous-versions/windows/it-pro/windows-server-2008-R2-and-2008/cc730660(v=ws.11)).

## <a name="prerequisites"></a>Prerrequisitos

* **Granja de AD FS**: asegúrese de que la granja de AD FS se base en Windows Server 2012 R2 o una versión posterior.
* **Azure AD Connect**: asegúrese de que la versión de Azure AD Connect sea 1.1.553.0 o posterior. Usará la tarea **Actualización del certificado SSL de AD FS**.

![Tarea de actualización de TLS](./media/how-to-connect-fed-ssl-update/updatessltask.png)

## <a name="step-1-provide-ad-fs-farm-information"></a>Paso 1: Proporcionar información de la granja de AD FS

Azure AD Connect trata de obtener automáticamente la información sobre la granja de AD FS. Para ello, realiza las tareas siguientes:
1. Consultar la información de la granja de AD FS (Windows Server 2016 o posterior).
2. Hacer referencia a la información de las ejecuciones anteriores (que se almacena localmente con Azure AD Connect).

Puede modificar la lista de servidores que aparecen mediante la adición o eliminación de los servidores a fin de reflejar la configuración actual de la granja de AD FS. Tan pronto como se proporciona la información del servidor, Azure AD Connect muestra la conectividad y el estado actual del certificado TLS/SSL.

![Información del servidor de AD FS](./media/how-to-connect-fed-ssl-update/adfsserverinfo.png)

Si la lista contiene un servidor que ya no forma parte de la granja de AD FS, haga clic en **Quitar** para eliminar el servidor de la lista de servidores de la granja de AD FS.

![Servidor sin conexión en la lista](./media/how-to-connect-fed-ssl-update/offlineserverlist.png)

>[!NOTE]
> La eliminación de un servidor de la lista de servidores de la granja de AD FS en Azure AD Connect es una operación local y actualiza la información de la granja de AD FS que Azure AD Connect mantiene localmente. Azure AD Connect no modifica la configuración en AD FS para reflejar el cambio.    

## <a name="step-2-provide-a-new-tlsssl-certificate"></a>Paso 2: Proporcionar el nuevo certificado TLS/SSL

Después de confirmar la información sobre los servidores de la granja de AD FS, Azure AD Connect solicita el nuevo certificado TLS/SSL. Proporcione un certificado PFX protegido con contraseña para continuar con la instalación.

![Certificado TLS/SSL](./media/how-to-connect-fed-ssl-update/certificate.png)

Después de proporcionar el certificado, Azure AD Connect pasa por una serie de requisitos previos. Compruebe el certificado para asegurarse de que es correcto para la granja de AD FS:

-   El nombre de sujeto o el nombre alternativo de sujeto para el certificado es igual que el nombre del servicio de federación o es un certificado comodín.
-   El certificado es válido durante más de 30 días.
-   La cadena de confianza del certificado es válida.
-   El certificado está protegido con contraseña.

## <a name="step-3-select-servers-for-the-update"></a>Paso 3: Seleccionar los servidores para la actualización

En el paso siguiente, seleccione los servidores que deben tener el certificado TLS/SSL actualizado. Los servidores que están sin conexión no se pueden seleccionar para la actualización.

![Seleccionar los servidores para actualizar](./media/how-to-connect-fed-ssl-update/selectservers.png)

Una vez completada la configuración, Azure AD Connect muestra el mensaje que indica el estado de la actualización y proporciona una opción para comprobar el inicio de sesión de AD FS.

![Configuración completada](./media/how-to-connect-fed-ssl-update/configurecomplete.png)   

## <a name="faqs"></a>Preguntas más frecuentes

* **¿Cuál debe ser el nombre de sujeto del certificado del nuevo certificado TLS/SSL de AD FS?**

    Azure AD Connect comprueba si el nombre de sujeto o el nombre alternativo de sujeto del certificado contiene el nombre del servicio de federación. Por ejemplo, si el nombre del servicio de federación es fs.contoso.com, el nombre de sujeto o el nombre de sujeto alternativo debe ser fs.contoso.com.  También se aceptan certificados comodín.

* **¿Por qué me piden de nuevo las credenciales en la página del servidor de WAP?**

    Si las credenciales proporcionadas para conectarse a servidores de AD FS no tienen el privilegio para administrar también los servidores de WAP, Azure AD Connect pide las credenciales que tienen privilegios administrativos en los servidores de WAP.

* **El servidor se muestra como sin conexión. ¿qué debo hacer?**

    Azure AD Connect no puede realizar ninguna operación si el servidor está sin conexión. Si el servidor forma parte de la granja de AD FS, compruebe la conectividad con el servidor. Después de resolver el problema, pulse el icono de actualización para actualizar el estado en el asistente. Si el servidor formó parte de la granja anteriormente pero ahora ya no existe, haga clic en **Quitar** para eliminarlo de la lista de servidores que mantiene Azure AD Connect. El hecho de quitar el servidor de la lista en Azure AD Connect no modifica la propia configuración de AD FS. Si usa AD FS en Windows Server 2016 o posterior, el servidor permanece en las opciones de configuración y se mostrará la próxima vez que se ejecute la tarea.

* **¿Puedo actualizar un subconjunto de los servidores de la granja con el nuevo certificado TLS/SSL?**

    Sí. Siempre puede ejecutar la tarea **Actualizar certificado SSL** de nuevo para actualizar el resto de servidores. En la página **Seleccionar servidores para la actualización del certificado SSL**, puede ordenar la lista de servidores por **Fecha de expiración de SSL** para tener acceso fácilmente a los servidores que todavía no se han actualizado.

* **Quité el servidor en la ejecución anterior, pero todavía se muestra como sin conexión y aparece en la página de servidores de AD FS. ¿Por qué sigue estando sin conexión el servidor incluso después de la eliminación?**

    Quitar el servidor de la lista en Azure AD Connect no lo quita de la configuración de AD FS. Azure AD Connect hace referencia a AD FS (Windows Server 2016 o posterior) para cualquier información sobre la granja. Si el servidor sigue estando presente en la configuración de AD FS, aparecerá de nuevo en la lista.  

## <a name="next-steps"></a>Pasos siguientes

- [Azure AD Connect y la federación](how-to-connect-fed-whatis.md)
- [Servicios de federación de Active Directory y personalización con Azure AD Connect](how-to-connect-fed-management.md)