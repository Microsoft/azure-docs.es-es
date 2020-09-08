---
title: 'Tutorial: creación de confianza de bosque en Azure AD Domain Services | Microsoft Docs'
description: Obtenga información acerca de cómo crear un bosque de salida unidireccional a un dominio de AD DS local en Azure Portal para Azure AD Domain Services
services: active-directory-ds
author: iainfoulds
manager: daveba
ms.service: active-directory
ms.subservice: domain-services
ms.workload: identity
ms.topic: tutorial
ms.date: 07/06/2020
ms.author: iainfou
ms.openlocfilehash: 24928ec4117b321cfec7177fdad40f2a3ab7a1f4
ms.sourcegitcommit: 5b6acff3d1d0603904929cc529ecbcfcde90d88b
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/21/2020
ms.locfileid: "88722728"
---
# <a name="tutorial-create-an-outbound-forest-trust-to-an-on-premises-domain-in-azure-active-directory-domain-services"></a>Tutorial: Creación de una confianza de bosque de salida en un dominio local de Azure Active Directory Domain Services

En entornos en los que no se pueden sincronizar los hash de contraseña, o en los que los usuarios inician sesión de forma exclusiva con tarjetas inteligentes y no saben su contraseña, puede usar un bosque de recursos en Azure Active Directory Domain Services (Azure AD DS). Un bosque de recursos usa una confianza de salida unidireccional desde Azure AD DS a uno o varios entornos locales de AD DS. Esta relación de confianza permite que los usuarios, las aplicaciones y los equipos se autentiquen en un dominio local desde el dominio administrado de Azure AD DS. En un bosque de recursos, los hashes de contraseña locales nunca se sincronizan.

![Diagrama de la confianza de bosque desde Azure AD DS a un entorno local de AD DS](./media/concepts-resource-forest/resource-forest-trust-relationship.png)

En este tutorial, aprenderá a:

> [!div class="checklist"]
> * Configurar DNS en un entorno de AD DS local para admitir la conectividad de Azure AD DS.
> * Crear una confianza de bosque de entrada unidireccional en un entorno local de AD DS
> * Crear una confianza de bosque de salida unidireccional en Azure AD DS.
> * Probar y validar la relación de confianza para la autenticación y el acceso a los recursos.

Si no tiene una suscripción a Azure, [cree una cuenta](https://azure.microsoft.com/free/?WT.mc_id=A261C142F) antes de empezar.

## <a name="prerequisites"></a>Requisitos previos

Para completar este tutorial, necesitará los siguientes recursos y privilegios:

* Una suscripción de Azure activa.
    * Si no tiene una suscripción a Azure, [cree una cuenta](https://azure.microsoft.com/free/?WT.mc_id=A261C142F).
* Un inquilino de Azure Active Directory asociado a su suscripción, ya sea sincronizado con un directorio en el entorno local o con un directorio solo en la nube.
    * Si es necesario, [cree un inquilino de Azure Active Directory][create-azure-ad-tenant] o [asocie una suscripción a Azure con su cuenta][associate-azure-ad-tenant].
* Un dominio administrado de Azure Active Directory Domain Services creado mediante un bosque de recursos y configurado en su inquilino de Azure AD.
    * Si es necesario, [cree y configure un dominio administrado de Azure Active Directory Domain Services][create-azure-ad-ds-instance-advanced].
    
    > [!IMPORTANT]
    > Asegúrese de crear un dominio administrado con un bosque de *recursos*. La opción predeterminada crea un bosque de *usuarios*. Solo los bosques de recursos pueden crear confianzas con entornos de AD DS locales.
    >
    > También debe usar como mínimo la SKU *Enterprise* para el dominio administrado. Si es necesario, [cambie la SKU de un dominio administrado][howto-change-sku].

## <a name="sign-in-to-the-azure-portal"></a>Inicio de sesión en Azure Portal

En este tutorial creará y configurará la confianza de bosque de salida de Azure AD DS mediante Azure Portal. Para empezar a trabajar, primero inicie sesión en [Azure Portal](https://portal.azure.com).

## <a name="networking-considerations"></a>Consideraciones sobre redes

La red virtual que hospeda el bosque de recursos de Azure AD DS necesita conectividad de red con su entorno local de Active Directory. Las aplicaciones y los servicios también necesitan conectividad de red con la red virtual que hospeda el bosque de recursos de Azure AD DS. La conectividad de red con el bosque de recursos de Azure AD DS debe estar siempre activa y estable. De lo contrario, es posible que los usuarios no puedan autenticarse ni acceder a los recursos.

Antes de configurar una confianza de bosque en Azure AD DS, asegúrese de que las redes entre Azure y el entorno local cumplan los requisitos siguientes:

* Use direcciones IP privadas. No use DHCP con asignación de direcciones IP dinámicas.
* Evite los espacios de direcciones IP superpuestos para permitir el emparejamiento y el enrutamiento de redes virtuales con el fin de que Azure y el entorno local se comuniquen correctamente.
* Una red virtual de Azure necesita una subred de puerta de enlace para configurar una conexión de [VPN de sitio a sitio (S2S) de Azure][vpn-gateway] o [ExpressRoute][expressroute].
* Cree subredes con suficientes direcciones IP para admitir su escenario.
* Asegúrese de que Azure AD DS tenga su propia subred; no comparta esta subred de red virtual con los servicios y las máquinas virtuales de la aplicación.
* Las redes virtuales emparejadas no son transitivas.
    * Los emparejamientos de redes virtuales de Azure deben crearse entre todas las redes virtuales que quiera usar la confianza del bosque de recursos de Azure AD en el entorno local de AD DS.
* Proporcione conectividad de red continua al bosque local de Active Directory. No use conexiones a petición.
* Asegúrese de que haya una resolución de nombres (DNS) continua entre el nombre del bosque de recursos de Azure AD DS y el nombre del bosque de Active Directory local.

## <a name="configure-dns-in-the-on-premises-domain"></a>Configuración de DNS en el dominio local

Para resolver correctamente el dominio administrado desde el entorno local, quizá tenga que agregar reenviadores a los servidores DNS existentes. Si no ha configurado el entorno local para comunicarse con el dominio administrado, complete los pasos siguientes desde una estación de trabajo de administración para el dominio de AD DS local:

1. Seleccione **Inicio | Herramientas administrativas | DNS**.
1. Haga clic con el botón derecho en el servidor DNS, como *myAD01*, y seleccione **Propiedades**.
1. Elija **Reenviadores** y, después, **Editar** para agregar reenviadores adicionales.
1. Agregue las direcciones IP del dominio administrado, como *10.0.2.4* y *10.0.2.5*.

## <a name="create-inbound-forest-trust-in-the-on-premises-domain"></a>Creación de una confianza de bosque de entrada en el dominio local

El dominio de AD DS local necesita una confianza de bosque de entrada para el dominio administrado. Esta confianza se debe crear manualmente en el dominio de AD DS local, no se puede crear a partir de Azure Portal.

Para configurar la confianza de entrada en el dominio de AD DS local, complete los pasos siguientes desde una estación de trabajo de administración para el dominio de AD DS local:

1. Seleccione **Inicio | Herramientas administrativas | Dominios y confianzas de Active Directory**.
1. Haga clic con el botón derecho en el dominio, como *onprem.contoso.com*, y seleccione **Propiedades**.
1. Elija la pestaña **Confianzas** y, a continuación, **Nueva confianza**.
1. Escriba el nombre de dominio de Azure AD DS, por ejemplo, *aaddscontoso.com* y, después, seleccione **Siguiente**.
1. Seleccione la opción para crear una **Confianza de bosque** y, a continuación, para crear una confianza **Unidireccional: de entrada**.
1. Elija la opción para crear la confianza **Solo para este dominio**. En el paso siguiente, creará la confianza en Azure Portal para el dominio administrado.
1. Elija usar **Autenticación en todo el bosque** y después escriba y confirme una contraseña de confianza. Esta misma contraseña también se escribe en Azure Portal en la sección siguiente.
1. Deje las opciones predeterminadas de las siguientes ventanas y después elija la opción **No, no confirmar la confianza saliente**.
1. Seleccione **Finish** (Finalizar).

## <a name="create-outbound-forest-trust-in-azure-ad-ds"></a>Creación de una confianza de bosque de salida en Azure AD DS

Con el dominio local de AD DS configurado para resolver el dominio administrado y una confianza de bosque de entrada creada, cree ahora la confianza de bosque de salida. Esta confianza de bosque de salida completa la relación de confianza entre el dominio local de AD DS y el dominio administrado.

Para crear la confianza de salida para el dominio administrado en Azure Portal, realice los pasos siguientes:

1. En Azure Portal, busque y seleccione **Azure AD Domain Services** y después seleccione el dominio administrado, como *aaddscontoso.com*.
1. En el menú del lado izquierdo del dominio administrado, seleccione **Confianzas** y, a continuación, elija **+ Agregar** una confianza.

   > [!NOTE]
   > Si no ve la opción de menú **Confianzas**, compruebe bajo **Propiedades** el *Tipo de bosque*. Solo los bosques de *recursos* pueden crear confianzas. Si el bosque es de tipo *Usuario*, no se pueden crear confianzas. Actualmente no hay ninguna manera de cambiar el tipo de bosque de un dominio administrado. Debe eliminar y volver a crear el dominio administrado como un bosque de recursos.

1. Escriba un nombre para mostrar que identifique la confianza y, después, el nombre DNS del bosque de confianza local, como *onprem.contoso.com*.
1. Proporcione la misma contraseña de confianza que usó al configurar la confianza de bosque de entrada para el dominio local de AD DS en la sección anterior.
1. Proporcione al menos dos servidores DNS para el dominio local de AD DS, como *10.1.1.4* y *10.1.1.5*.
1. Cuando esté listo, **Guarde** la confianza de bosque de salida.

    ![Creación de una confianza de bosque de salida en Azure Portal](./media/tutorial-create-forest-trust/portal-create-outbound-trust.png)

## <a name="validate-resource-authentication"></a>Validación de autenticación de recursos

Los siguientes escenarios habituales permiten validar que la confianza de bosque autentica correctamente a los usuarios y al acceso a los recursos:

* [Autenticación de usuarios locales desde el bosque de recursos de Azure AD DS](#on-premises-user-authentication-from-the-azure-ad-ds-resource-forest)
* [Acceso a los recursos del bosque de recursos de Azure AD DS mediante el usuario local](#access-resources-in-the-azure-ad-ds-resource-forest-using-on-premises-user)
    * [Habilitación del uso compartido de archivos e impresoras](#enable-file-and-printer-sharing)
    * [Creación de un grupo de seguridad e incorporación de miembros](#create-a-security-group-and-add-members)
    * [Creación de un recurso compartido de archivos para el acceso entre bosques](#create-a-file-share-for-cross-forest-access)
    * [Validación de la autenticación entre bosques en un recurso](#validate-cross-forest-authentication-to-a-resource)

### <a name="on-premises-user-authentication-from-the-azure-ad-ds-resource-forest"></a>Autenticación de usuarios locales desde el bosque de recursos de Azure AD DS

Debe tener una máquina virtual de Windows Server unida al dominio administrado. Use esta máquina virtual para probar si el usuario local puede autenticarse en una máquina virtual. Si es necesario, [cree una máquina virtual de Windows y únala al dominio administrado][join-windows-vm].

1. Conéctese a la máquina virtual de Windows Server unida al bosque de recursos de Azure AD DS mediante [Azure Bastion](../bastion/bastion-overview.md) y sus credenciales de administrador de Azure AD DS.
1. Abra un símbolo del sistema y use el comando `whoami` para mostrar el nombre distintivo del usuario autenticado actualmente:

    ```console
    whoami /fqdn
    ```

1. Use el comando `runas` para autenticarse como un usuario del dominio local. En el siguiente comando, reemplace `userUpn@trusteddomain.com` por el UPN de un usuario del dominio local de confianza. El comando le solicita la contraseña de usuario:

    ```console
    Runas /u:userUpn@trusteddomain.com cmd.exe
    ```

1. Si la autenticación se realiza correctamente, se abre un nuevo símbolo del sistema. El título del nuevo símbolo del sistema incluye `running as userUpn@trusteddomain.com`.
1. Utilice `whoami /fqdn` en el nuevo símbolo del sistema para ver el nombre distintivo del usuario autenticado del entorno local de Active Directory.

### <a name="access-resources-in-the-azure-ad-ds-resource-forest-using-on-premises-user"></a>Acceso a los recursos del bosque de recursos de Azure AD DS mediante el usuario local

Con la máquina virtual de Windows Server unida al bosque de recursos de Azure AD DS, puede probar el escenario en el que los usuarios pueden acceder a los recursos hospedados en el bosque de recursos cuando se autentican desde los equipos del dominio local con los usuarios del dominio local. En los siguientes ejemplos se muestra cómo crear y probar varios escenarios habituales.

#### <a name="enable-file-and-printer-sharing"></a>Habilitación del uso compartido de archivos e impresoras

1. Conéctese a la máquina virtual de Windows Server unida al bosque de recursos de Azure AD DS mediante [Azure Bastion](../bastion/bastion-overview.md) y sus credenciales de administrador de Azure AD DS.

1. Abra **Configuración de Windows** y busque y seleccione **Centro de redes y recursos compartidos**.
1. Elija la opción **Cambiar configuración de uso compartido avanzado**.
1. En **Perfil de dominio**, seleccione **Activar el uso compartido de archivos e impresoras** y, a continuación, **Guardar cambios**.
1. Cierre el **Centro de redes y recursos compartidos**.

#### <a name="create-a-security-group-and-add-members"></a>Creación de un grupo de seguridad e incorporación de miembros

1. Abra **Usuarios y equipos de Active Directory**.
1. Haga clic derecho en el nombre de dominio, elija **Nuevo** y después seleccione **Unidad organizativa**.
1. En el cuadro de nombre, escriba *LocalObjects* y seleccione **Aceptar**.
1. Seleccione y haga clic con el botón derecho en **LocalObjects** en el panel de navegación. Seleccione **Nuevo** y, a continuación, **Grupo**.
1. Escriba *FileServerAccess* en el cuadro **Nombre de grupo**. En **Ámbito de grupo**, seleccione **Dominio local** y, a continuación, elija **Aceptar**.
1. En el panel de contenido, haga doble clic en **FileServerAccess**. Seleccione **Miembros**, elija **Agregar** y, a continuación, seleccione **Ubicaciones**.
1. Seleccione el entorno local de Active Directory en la vista **Ubicación** y, a continuación, elija **Aceptar**.
1. Escriba *Usuarios del dominio* en el cuadro **Escriba los nombres de objeto que desea seleccionar**. Seleccione **Comprobar nombres**, proporcione las credenciales del entorno local de Active Directory y, a continuación, seleccione **Aceptar**.

    > [!NOTE]
    > Debe especificar las credenciales porque la relación de confianza solo es unidireccional. Esto implica que los usuarios del dominio administrado de Azure AD DS no pueden acceder a los recursos ni buscar usuarios o grupos en el dominio de confianza (local).

1. El grupo **Usuarios del dominio** del entorno local de Active Directory debe ser miembro del grupo **FileServerAccess**. Seleccione **Aceptar** para guardar el grupo y cerrar la ventana.

#### <a name="create-a-file-share-for-cross-forest-access"></a>Creación de un recurso compartido de archivos para el acceso entre bosques

1. En la máquina virtual de Windows Server unida al bosque de recursos de Azure AD DS, cree una carpeta y proporcione un nombre como *CrossForestShare*.
1. Haga clic con el botón derecho en la carpeta y elija **Propiedades**.
1. Seleccione la pestaña **Seguridad** y después **Editar**.
1. En el cuadro de diálogo *Permisos para CrossForestShare*, seleccione **Agregar**.
1. Escriba *FileServerAccess* en **Escriba los nombres de objeto que desea seleccionar** y, a continuación, seleccione **Aceptar**.
1. Seleccione *FileServerAccess* de la lista **Nombres de grupos o usuarios**. En la lista **Permisos para FileServerAccess**, elija *Permitir* para los permisos **Modificar** y **Escribir** y después seleccione **Aceptar**.
1. Seleccione la pestaña **Compartir** y después elija **Uso compartido avanzado...**
1. Elija **Compartir esta carpeta** y escriba un nombre fácil de recordar para el recurso compartido de archivos en **Nombre del recurso compartido**, como *CrossForestShare*.
1. Seleccione **Permisos**. En la lista **Permisos para todos**, elija **Permitir** para el permiso **Cambiar**.
1. Seleccione **Aceptar** dos veces y, a continuación, **Cerrar**.

#### <a name="validate-cross-forest-authentication-to-a-resource"></a>Validación de la autenticación entre bosques en un recurso

1. Inicie sesión en un equipo Windows unido a su entorno local de Active Directory mediante una cuenta de usuario del entorno local de Active Directory.
1. Con el **Explorador de Windows**, conéctese al recurso compartido que creó. Para ello, use el nombre de host completo y el recurso compartido, como en `\\fs1.aaddscontoso.com\CrossforestShare`.
1. Para validar el permiso de escritura, haga clic con el botón derecho en la carpeta, elija **Nuevo** y después seleccione **Documento de texto**. Use el nombre predeterminado **Nuevo documento de texto**.

    Si los permisos de escritura están configurados correctamente, se crea un nuevo documento de texto. Los pasos siguientes abrirán, editarán y eliminarán el archivo según corresponda.
1. Para validar el permiso de lectura, abra el **Nuevo documento de texto**.
1. Para validar el permiso de modificación, agregue texto al archivo y cierre el **Bloc de notas**. Cuando se le pregunte si quiere guardar los cambios, elija **Guardar**.
1. Para validar el permiso de eliminación, seleccione el **Nuevo documento de texto** y elija **Eliminar**. Elija **Sí** para confirmar la eliminación del archivo.

## <a name="next-steps"></a>Pasos siguientes

En este tutorial, ha aprendido a:

> [!div class="checklist"]
> * Configurar DNS en un entorno de AD DS local para admitir la conectividad de Azure AD DS.
> * Crear una confianza de bosque de entrada unidireccional en un entorno local de AD DS
> * Crear una confianza de bosque de salida unidireccional en Azure AD DS.
> * Probar y validar la relación de confianza para la autenticación y el acceso a los recursos.

Para obtener más información conceptual sobre los tipos de bosque de Azure AD DS, consulte [¿Qué son los bosques de recursos?][concepts-forest] y [¿Cómo funcionan las confianzas de bosque en Azure AD DS?][concepts-trust]

<!-- INTERNAL LINKS -->
[concepts-forest]: concepts-resource-forest.md
[concepts-trust]: concepts-forest-trust.md
[create-azure-ad-tenant]: ../active-directory/fundamentals/sign-up-organization.md
[associate-azure-ad-tenant]: ../active-directory/fundamentals/active-directory-how-subscriptions-associated-directory.md
[create-azure-ad-ds-instance-advanced]: tutorial-create-instance-advanced.md
[howto-change-sku]: change-sku.md
[vpn-gateway]: ../vpn-gateway/vpn-gateway-about-vpngateways.md
[expressroute]: ../expressroute/expressroute-introduction.md
[join-windows-vm]: join-windows-vm.md