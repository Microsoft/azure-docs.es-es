---
title: 'Tutorial: Creación de un dominio administrado de Azure Active Directory Domain Services | Microsoft Docs'
description: En este tutorial aprenderá a crear y configurar un dominio administrado de Azure Active Directory Domain Services mediante Azure Portal.
author: iainfoulds
manager: daveba
ms.service: active-directory
ms.subservice: domain-services
ms.workload: identity
ms.topic: tutorial
ms.date: 07/06/2020
ms.author: iainfou
ms.openlocfilehash: dc225cef2981bd6307a919d52b75d3e0791e46cf
ms.sourcegitcommit: 11e2521679415f05d3d2c4c49858940677c57900
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/31/2020
ms.locfileid: "87491007"
---
# <a name="tutorial-create-and-configure-an-azure-active-directory-domain-services-managed-domain"></a>Tutorial: Creación y configuración de un dominio administrado de Azure Active Directory Domain Services

Azure Active Directory Domain Services (Azure AD DS) proporciona servicios de dominio administrados como, por ejemplo, unión a un dominio, directiva de grupo, LDAP o autenticación Kerberos/NTLM, que son totalmente compatibles con Windows Server Active Directory. Estos servicios de dominio se usan sin necesidad de implementar, administrar ni aplicar revisiones a los controladores de dominio. Azure AD DS se integra con el inquilino de Azure AD existente. Esta integración permite a los usuarios iniciar sesión con sus credenciales corporativas y, además, le permite usar grupos y cuentas de usuario existentes para proteger el acceso a los recursos.

Puede crear un dominio administrado con las opciones de configuración predeterminadas de sincronización y redes o [definir manualmente estos valores][tutorial-create-instance-advanced]. En este tutorial se muestra cómo usar las opciones predeterminadas para crear y configurar un dominio administrado de Azure AD DS mediante Azure Portal.

En este tutorial, aprenderá a:

> [!div class="checklist"]
> * Requisitos de DNS para un dominio administrado
> * Creación de un dominio administrado
> * Habilitación de la sincronización de hash de contraseñas

Si no tiene una suscripción a Azure, [cree una cuenta](https://azure.microsoft.com/free/?WT.mc_id=A261C142F) antes de empezar.

## <a name="prerequisites"></a>Requisitos previos

Para completar este tutorial, necesitará los siguientes recursos y privilegios:

* Una suscripción de Azure activa.
    * Si no tiene una suscripción a Azure, [cree una cuenta](https://azure.microsoft.com/free/?WT.mc_id=A261C142F).
* Un inquilino de Azure Active Directory asociado a su suscripción, ya sea sincronizado con un directorio en el entorno local o con un directorio solo en la nube.
    * Si es necesario, [cree un inquilino de Azure Active Directory][create-azure-ad-tenant] o [asocie una suscripción a Azure con su cuenta][associate-azure-ad-tenant].
* Necesita privilegios de *administrador global* en el inquilino de Azure AD para habilitar Azure AD DS.
* Necesita privilegios de *colaborador* en la suscripción de Azure para crear los recursos de Azure AD DS necesarios.

Aunque no es necesario para Azure AD DS, se recomienda [configurar el autoservicio de restablecimiento de contraseña (SSPR) ][configure-sspr] para el inquilino de Azure AD. Los usuarios pueden cambiar su contraseña sin SSPR, pero este les ayuda si olvidan la contraseña y necesitan restablecerla.

> [!IMPORTANT]
> Después de crear un dominio administrado, no puede trasladarlo a otro grupo de recursos, red virtual, suscripción, etc. Tenga cuidado a la hora de seleccionar la suscripción, el grupo de recursos, la región y la red virtual más adecuados al implementar el dominio administrado.

## <a name="sign-in-to-the-azure-portal"></a>Inicio de sesión en Azure Portal

En este tutorial creará y configurará el dominio administrado mediante Azure Portal. Para empezar a trabajar, primero inicie sesión en [Azure Portal](https://portal.azure.com).

## <a name="create-a-managed-domain"></a>Creación de un dominio administrado

Para iniciar el Asistente para **habilitar Azure AD Domain Services**, complete los pasos siguientes:

1. En el menú de Azure Portal o en la **página principal**, seleccione **Crear un recurso**.
1. Escriba *Domain Services* en la barra de búsqueda y, a continuación, elija *Azure AD Domain Services* en las sugerencias de búsqueda.
1. En la página Azure AD Domain Services, seleccione **Crear**. Se inicia el Asistente para **habilitar Azure AD Domain Services**.
1. Seleccione la **suscripción** de Azure en la que desea crear el dominio administrado.
1. Seleccione el **grupo de recursos** al que debería pertenecer el dominio administrado. Elija **Crear nuevo** o seleccione un grupo de recursos existente.

Cuando se crea un dominio administrado, se especifica un nombre DNS. Hay algunas consideraciones que se deben tener en cuenta al elegir este nombre DNS:

* **Nombre de dominio integrado:** de forma predeterminada, se usa el nombre de dominio integrado del directorio (un sufijo *.onmicrosoft.com*). Si se quiere habilitar el acceso LDAP seguro al dominio administrado a través de Internet, no se puede crear un certificado digital para proteger la conexión con este dominio predeterminado. Microsoft es el propietario del dominio *.onmicrosoft.com*, por lo que una entidad de certificación no emitirá un certificado.
* **Nombres de dominio personalizados:** el enfoque más común consiste en especificar un nombre de dominio personalizado, normalmente uno que ya se posee y es enrutable. Cuando se usa un dominio personalizado enrutable, el tráfico puede fluir correctamente según sea necesario para admitir las aplicaciones.
* **Sufijos de dominio no enrutables:** por lo general, se recomienda evitar un sufijo de nombre de dominio no enrutable, como *contoso.local*. El sufijo *.local* no es enrutable y puede provocar problemas con la resolución de DNS.

> [!TIP]
> Si crea un nombre de dominio personalizado, tenga cuidado con los espacios de nombres DNS existentes. Se recomienda usar un nombre de dominio independiente de cualquier espacio de nombres de DNS local o de Azure existentes.
>
> Por ejemplo, si tiene un espacio de nombres de DNS existente para *contoso.com*, cree un dominio administrado con el nombre de dominio personalizado *aaddscontoso.com*. Si necesita usar LDAP seguro, debe registrar y poseer este nombre de dominio personalizado para generar los certificados necesarios.
>
> Es posible que tenga que crear algunos registros DNS adicionales para otros servicios del entorno o reenviadores DNS condicionales entre los espacios de nombres de DNS existentes en el entorno. Por ejemplo, si ejecuta un servidor web que hospeda un sitio mediante el nombre DNS raíz, puede haber conflictos de nomenclatura que requieran más entradas DNS.
>
> En estos tutoriales y artículos de procedimientos, el dominio personalizado *aaddscontoso.com* se usa como ejemplo breve. En todos los comandos, especifique su propio nombre de dominio.

También se aplican las siguientes restricciones de nombre DNS:

* **Restricciones de prefijo de dominio:** no se puede crear un dominio administrado con un prefijo de más de 15 caracteres. El prefijo del nombre de dominio especificado (por ejemplo, *aaddscontoso* en el nombre de dominio *aaddscontoso.com*) debe contener 15 caracteres o menos.
* **Conflictos de nombres de red:** el nombre de dominio DNS del dominio administrado no puede existir ya en la red virtual. Busque, en concreto, los siguientes escenarios que provocarían un conflicto de nombres:
    * Ya tiene un dominio de Active Directory con el mismo nombre de dominio DNS en la red virtual de Azure.
    * La red virtual en la que planea habilitar el dominio administrado tiene una conexión VPN con la red local. En este escenario, asegúrese de que no tiene un dominio con el mismo nombre de dominio DNS de la red local.
    * Ya dispone de un servicio en la nube de Azure con ese nombre en la red virtual de Azure.

Complete los campos de la ventana *Datos básicos* de Azure Portal para crear un dominio administrado:

1. Escriba un **nombre de dominio DNS** para el dominio administrado, teniendo en cuenta los puntos anteriores.
1. Elija la **ubicación** de Azure en que se debe crear el dominio administrado. Si elige una región que admite zonas de disponibilidad de Azure, los recursos de Azure AD DS se distribuyen entre las zonas para conseguir redundancia adicional.

    > [!TIP]
    > Las zonas de disponibilidad son ubicaciones físicas exclusivas dentro de una región de Azure. Cada zona de disponibilidad consta de uno o varios centros de datos equipados con alimentación, refrigeración y redes independientes. Para garantizar la resistencia, hay un mínimo de tres zonas independientes en todas las regiones habilitadas.
    >
    > No es necesario realizar ninguna configuración para que Azure AD DS se distribuya entre zonas. La plataforma Azure controla automáticamente la distribución en zonas de los recursos. Para más información y consulta de la disponibilidad en las regiones, consulte [¿Qué son las zonas de disponibilidad en Azure?][availability-zones]

1. La **SKU** determina el rendimiento, la frecuencia de la copia de seguridad y el número máximo de confianzas de bosque que se pueden crear. Puede cambiar la SKU una vez creado el dominio administrado si cambian sus necesidades o requisitos empresariales. Para más información, consulte [Conceptos de las SKU de Azure AD DS][concepts-sku].

    Para este tutorial, seleccione la SKU *Estándar*.
1. Un *bosque* es una construcción lógica que Active Directory Domain Services utiliza para agrupar uno o más dominios. De forma predeterminada, un dominio administrado se crea como un bosque de *usuarios*. Este tipo de bosque sincroniza todos los objetos de Azure AD, incluidas las cuentas de usuario creadas en un entorno de AD DS local.

    Un bosque de *Recursos* solo sincroniza los usuarios y grupos creados directamente en Azure AD. Para más información sobre los bosques de *Recursos*, incluido el motivo por el que puede usar uno y cómo crear confianzas de bosque con dominios de AD DS locales, consulte [Introducción a los bosques de recursos de Azure AD DS][resource-forests].

    En este tutorial, elija crear un bosque de *Usuario*.

    ![Configuración de las opciones básicas de un dominio administrado de Azure AD Domain Services](./media/tutorial-create-instance/basics-window.png)

Para crear rápidamente un dominio administrado, puede seleccionar **Revisión y creación** para aceptar otras opciones de configuración predeterminadas. Los siguientes valores predeterminados se configuran al elegir esta opción de creación:

* Crea una red virtual llamada *aadds-vnet* que usa el intervalo de direcciones IP de *10.0.2.0/24*.
* Crea una subred denominada *aadds-subnet* mediante el intervalo de direcciones IP de *10.0.2.0/24*.
* Sincroniza *Todos* los usuarios de Azure AD con el dominio administrado.

Seleccione **Revisión y creación** para aceptar estas opciones de configuración predeterminadas.

## <a name="deploy-the-managed-domain"></a>Eliminación del dominio administrado

En la página **Resumen** del asistente, revise la configuración del dominio administrado. Puede volver a cualquier paso del asistente para realizar cambios. Para volver a implementar un dominio administrado en un inquilino de Azure AD diferente de una manera coherente con estas opciones de configuración, también puede **descargar una plantilla para automatizar el procedimiento**.

1. Para crear el dominio administrado, seleccione **Crear**. Aparece una nota que indica que determinadas opciones de configuración, como el nombre de DNS o la red virtual, no se pueden cambiar después de que se ha creado la instancia administrada de Azure AD DS. Para continuar, seleccione **Aceptar**.
1. El proceso de aprovisionamiento del dominio administrado puede tardar hasta una hora. En el portal se muestra una notificación que señala el progreso de la implementación de Azure AD DS. Seleccione la notificación para ver el progreso detallado de la implementación.

    ![Notificación en Azure Portal de la implementación en curso](./media/tutorial-create-instance/deployment-in-progress.png)

1. La página se cargará con las actualizaciones del proceso de implementación, incluida la creación de recursos en el directorio.
1. Seleccione el grupo de recursos, como *myResourceGroup* y, a continuación, elija el dominio administrado en la lista de recursos de Azure, como *aaddscontoso.com*. La pestaña **Información general** muestra que el dominio administrado se está *Implementando*. No se puede configurar el dominio administrado hasta que está totalmente aprovisionado.

    ![Estado de Domain Services durante el estado de aprovisionamiento](./media/tutorial-create-instance/provisioning-in-progress.png)

1. Cuando el dominio administrado está aprovisionado por completo, la pestaña **Información general** muestra el estado del dominio como *En ejecución*.

    ![Estado de Domain Services una vez aprovisionado correctamente](./media/tutorial-create-instance/successfully-provisioned.png)

> [!IMPORTANT]
> El dominio administrado se asocia al inquilino de Azure AD. Durante el proceso de aprovisionamiento, Azure AD DS crea dos aplicaciones empresariales, *Domain Controller Services* y *AzureActiveDirectoryDomainControllerServices* en el inquilino de Azure AD. Estas aplicaciones empresariales se necesitan para dar servicio al dominio administrado. No las elimine.

## <a name="update-dns-settings-for-the-azure-virtual-network"></a>Actualización de la configuración DNS para Azure Virtual Network

Ahora que Azure AD DS se ha implementado correctamente, es el momento de configurar la red virtual para permitir que otras máquinas virtuales y aplicaciones conectadas usen el dominio administrado. Para proporcionar esta conectividad, actualice la configuración del servidor DNS de la red virtual de modo que apunte a las dos direcciones IP donde se ha implementado el dominio administrado.

1. En la pestaña **Información general** del dominio administrado se muestran algunos de los **Pasos de configuración necesarios**. El primer paso de configuración es la actualización de la configuración de servidores DNS para la red virtual. Cuando la configuración de DNS esté correctamente establecida, ya no se mostrará este paso.

    Las direcciones que aparecen son los controladores de dominio que se usan en la red virtual. En este ejemplo, estas direcciones son *10.0.2.4* y *10.0.2.5*. Más adelante puede encontrar estas direcciones IP en la pestaña **Propiedades**.

    ![Configuración de los valores de DNS para la red virtual con las direcciones IP de Azure AD Domain Services](./media/tutorial-create-instance/configure-dns.png)

1. Seleccione el botón **Configurar** para actualizar la configuración del servidor DNS de la red virtual. Los valores de DNS se configuran automáticamente para la red virtual.

> [!TIP]
> Si ha seleccionado una red virtual existente en los pasos anteriores, las máquinas virtuales conectadas a la red solo obtendrán la nueva configuración de DNS después de un reinicio. Puede reiniciar las máquinas virtuales mediante Azure Portal, Azure PowerShell o la CLI de Azure.

## <a name="enable-user-accounts-for-azure-ad-ds"></a>Habilitación de cuentas de usuario para Azure AD DS

Para autenticar a los usuarios en el dominio administrado, Azure AD DS necesita los hash de las contraseñas en un formato adecuado para la autenticación de NT LAN Manager (NTLM) y Kerberos. A menos que habilite Azure AD DS para el inquilino, Azure AD no genera ni almacena los hash de las contraseñas en el formato necesario para la autenticación NTLM o Kerberos. Por motivos de seguridad, Azure AD tampoco almacena las credenciales de contraseñas en forma de texto sin cifrar. Por consiguiente, Azure AD no tiene forma de generar automáticamente estos hash de las contraseñas de NTLM o Kerberos basándose en las credenciales existentes de los usuarios.

> [!NOTE]
> Una vez configurados correctamente, los hash de las contraseñas que se pueden usar se almacenan en el dominio administrado. Si elimina el dominio administrado, también se eliminarán los hash de las contraseñas que haya almacenados en ese momento.
>
> La información de credenciales sincronizada en Azure AD no se puede volver a usar si posteriormente crea un dominio administrado: debe volver a configurar la sincronización de los hash de contraseña para almacenarlos de nuevo. Los usuarios o las máquinas virtuales anteriormente unidos a un dominio no podrán autenticarse de inmediato, ya que Azure AD debe generar y almacenar los hash de contraseña en el nuevo dominio administrado.
>
> Para más información, consulte [Proceso de sincronización de hash de contraseña para Azure AD DS y Azure AD Connect][password-hash-sync-process].

Los pasos necesarios para generar y almacenar estos hash de contraseña son diferentes según se trate de cuentas de usuario solo de nube creadas en Azure AD o de las cuentas de usuarios que se sincronizan desde el directorio local mediante Azure AD Connect.

Una cuenta de usuario solo de nube es una cuenta creada en su directorio de Azure AD mediante Azure Portal, o bien mediante cmdlets de PowerShell de Azure AD. Estas cuentas de usuario no se sincronizan desde un directorio local.

> En este tutorial, vamos a trabajar con una cuenta de usuario básica solo de nube. Para más información sobre los pasos adicionales necesarios para usar Azure AD Connect, consulte [Sincronización de los hash de contraseña para las cuentas de usuario sincronizadas desde la instancia local de AD con el dominio administrado][on-prem-sync].

> [!TIP]
> Si el inquilino de Azure AD tiene una combinación entre usuarios solo de nube y usuarios de la instancia local de AD, debe realizar ambos procedimientos.

En el caso de las cuentas de usuario solo de nube, los usuarios deben cambiar sus contraseñas para poder usar Azure AD DS. Este proceso de cambio de contraseña hace que los valores hash de contraseña para la autenticación Kerberos y NTLM se generen y almacenen en Azure AD. La cuenta no se sincroniza desde Azure AD a Azure AD DS hasta que se cambia la contraseña. Puede hacer expirar las contraseñas de todos los usuarios en la nube del inquilino que tenga que usar Azure AD DS, lo que fuerza un cambio de contraseña en el siguiente inicio de sesión, o bien indicarles que cambien sus contraseñas manualmente. En este tutorial vamos a cambiar manualmente una contraseña de usuario.

Para que un usuario pueda restablecer su contraseña, el inquilino de Azure AD debe estar [configurado para el autoservicio de restablecimiento de contraseña][configure-sspr].

Para cambiar la contraseña de un usuario solo de nube, el usuario debe completar los pasos siguientes:

1. Vaya a la página Panel de acceso de Azure AD en [https://myapps.microsoft.com](https://myapps.microsoft.com).
1. En la esquina superior derecha, seleccione su nombre y, a continuación, elija **Perfil** en el menú desplegable.

    ![Seleccionar perfil](./media/tutorial-create-instance/select-profile.png)

1. En la página **Perfil**, seleccione **Cambiar contraseña**.
1. En la página **Cambiar contraseña**, escriba la contraseña existente (anterior) y luego escriba y confirme una nueva contraseña.
1. Seleccione **Submit** (Enviar).

Tras el cambio, la nueva contraseña tarda unos minutos en poder usarse en Azure AD DS y en iniciar sesión en los equipos unidos al dominio administrado.

## <a name="next-steps"></a>Pasos siguientes

En este tutorial, ha aprendido a:

> [!div class="checklist"]
> * Requisitos de DNS para un dominio administrado
> * Creación de un dominio administrado
> * Incorporación de usuarios administrativos a la administración de dominios
> * Habilitación de cuentas de usuario para Azure AD DS y generación de hash de contraseña

Antes de unir a un dominio las máquinas virtuales y de implementar las aplicaciones que usan el dominio administrado, configure una red virtual de Azure para las cargas de trabajo de aplicación.

> [!div class="nextstepaction"]
> [Configuración de la red virtual de Azure para cargas de trabajo de aplicación para usar el dominio administrado](tutorial-configure-networking.md)

<!-- INTERNAL LINKS -->
[tutorial-create-instance-advanced]: tutorial-create-instance-advanced.md
[create-azure-ad-tenant]: ../active-directory/fundamentals/sign-up-organization.md
[associate-azure-ad-tenant]: ../active-directory/fundamentals/active-directory-how-subscriptions-associated-directory.md
[network-considerations]: network-considerations.md
[create-dedicated-subnet]: ../virtual-network/virtual-network-manage-subnet.md#add-a-subnet
[scoped-sync]: scoped-synchronization.md
[on-prem-sync]: tutorial-configure-password-hash-sync.md
[configure-sspr]: ../active-directory/authentication/tutorial-enable-sspr.md
[password-hash-sync-process]: ../active-directory/hybrid/how-to-connect-password-hash-synchronization.md#password-hash-sync-process-for-azure-ad-domain-services
[tutorial-create-instance-advanced]: tutorial-create-instance-advanced.md
[skus]: overview.md
[resource-forests]: concepts-resource-forest.md
[availability-zones]: ../availability-zones/az-overview.md
[concepts-sku]: administration-concepts.md#azure-ad-ds-skus

<!-- EXTERNAL LINKS -->
[naming-prefix]: /windows-server/identity/ad-ds/plan/selecting-the-forest-root-domain#selecting-a-prefix
