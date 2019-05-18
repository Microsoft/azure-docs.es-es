---
title: Cómo administrar una cuenta de Marketplace comercial en el centro de partners
description: Obtenga información sobre cómo administrar una cuenta de Marketplace comercial en el centro de partners.
author: mattwojo
manager: evansma
ms.author: parthp
ms.service: marketplace
ms.topic: how-to
ms.date: 05/30/2019
ms.openlocfilehash: 935d2e1c96705506636c2883113a64bb70c39336
ms.sourcegitcommit: 36c50860e75d86f0d0e2be9e3213ffa9a06f4150
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 05/16/2019
ms.locfileid: "65806197"
---
# <a name="how-to-manage-your-commercial-marketplace-account-in-partner-center"></a>Cómo administrar su cuenta de Marketplace comercial en el centro de partners 

Una vez que haya [creado una cuenta de centro de partners](./create-account.md), puede administrar su cuenta y las ofertas con la [panel Marketplace comercial](https://partner.microsoft.com/dashboard/commercial-marketplace/overview).

En este artículo, analizaremos en profundidad cómo administrar su cuenta del centro de partners, incluido cómo: 

- [Obtener acceso a la configuración de su cuenta de centro de partners](#access-your-account-settings)
- [Busque el Id. de publicador, Id. de vendedor, Id. de usuario y los inquilinos de Azure AD](#account-details)
- [Actualizar información de contacto](#contact-info)
- [Administrar detalles financieros (cuenta de pago, el perfil de impuestos, estado de suspensión de pago)](#financial-details)
- [Configuración de los GUID de seguimiento para supervisar el uso del cliente](#tracking-guids)
- [Usuarios del administrador](#manage-users)
- [Grupos de administrador](#manage-groups)
- [Aplicaciones de administrador Azure AD](#manage-azure-ad-applications)
- [Definir los permisos y roles de usuario](#define-user-roles-and-permissions)
- [Administración de inquilinos de Azure AD (cuentas profesionales)](#manage-tenants)
- [Acuerdos de centro de partners de administrador](#agreements)


## <a name="access-your-account-settings"></a>Acceda a la configuración de cuenta

Si aún no lo ha hecho, usted (o el Administrador de su organización) debe acceder a la [configuración de la cuenta](https://partner.microsoft.com/dashboard/account/management) para la cuenta del centro de partners para:
- comprobar el estado de comprobación de cuenta de su empresa
- Confirme su Id. de vendedor, el Id. de MPN, el Id. de publicador y la información de contacto, incluyendo el contacto de vendedor y aprobador de la empresa
- configurar detalles financieros de su compañía, incluyendo exenciones fiscales si es necesario
- crear cuentas de usuario para cualquier persona que utilizará su cuenta de empresa en el centro de partners

### <a name="open-developer-settings"></a>Abrir configuración de desarrollador

Configuración de la cuenta se encuentra en la esquina superior derecha de su [panel Marketplace comercial](https://partner.microsoft.com/dashboard/commercial-marketplace) en Centro de partners. Seleccione el icono de engranaje (cerca de la esquina superior derecha del panel) y, a continuación, seleccione **opciones del desarrollador**. 

![Menú de configuración de cuenta en el centro de partners](./media/dashboard-developer-settings.png)

Dentro de **configuración de la cuenta**, podrá ver su:
- **Detalles de cuenta**: Tipo de cuenta y el estado de la cuenta
- **Id. de publicador**: Id. de vendedor, Id. de usuario, Id. de publicador, inquilinos de Azure AD, etcetera.
- **Información de contacto**: Nombre para mostrar del publicador, nombre de contacto del vendedor, correo electrónico, teléfono y dirección
- **Detalles financieros**: Estado de suspensión de cuenta de pago, el perfil de impuestos y pagos
- **Dispositivos**: Los dispositivos de pruebas asociados con su cuenta
- **GUID de seguimiento**: Cualquier seguimiento GUID asociado a su cuenta

### <a name="account-details"></a>Detalles de la cuenta

En la sección de detalles de la cuenta, puede ver información básica, como su **tipo de cuenta** (persona o empresa) y el **estado de comprobación** de su cuenta. Durante el proceso de comprobación de la cuenta, esta configuración mostrará cada paso necesario, incluida la comprobación de correos electrónicos, comprobación de empleo y comprobación de negocio. También puede actualizar el correo electrónico aquí y reenviar la comprobación si es necesario. 

### <a name="publisher-ids"></a>Id. de publicador

En la sección de identificadores de editor, puede ver su **Id. de vendedor**, **Id. de MPN**, y **Id. de publicador**. Estos valores son asignados por Microsoft para identificar la cuenta de desarrollador y no se puede editar.

### <a name="contact-info"></a>Información de contacto

En la sección de información de contacto, puede ver su **nombre para mostrar del publicador**, **información de contacto del vendedor** (el nombre de contacto, correo electrónico, número de teléfono y dirección para el vendedor de la empresa) y el **empresa aprobador** (el nombre, correo electrónico y número de teléfono de la persona con la autoridad para aprobar las decisiones de la empresa). 

### <a name="financial-details"></a>Detalles financieros

En la sección de detalles financieros, puede proporcionar o actualizar la información financiera si publicar aplicaciones de pago, complementos o servicios. 

Si solo va a publicar ofertas gratis, no es necesario configurar una cuenta de pago o rellene los formularios de impuestos. Si cambia de opinión más adelante y decide que desea vender a través de Microsoft, puede configurar la cuenta de pago y rellene los formularios de impuestos en ese momento. 

#### <a name="payout-account"></a>Cuenta de pago

Una cuenta de pago es la cuenta bancaria a la que se envía los lleva a cabo desde las ventas. Esta cuenta bancaria debe estar en el mismo país que se ha registrado su cuenta de centro de partners.

Para configurar la cuenta de pago, deberá **asociar tu Account Microsoft**:
1. En **configuración de la cuenta**, en el **detalles financieros** sección, seleccione **asociar tu Account Microsoft**. 
2. Cuando se le solicite, inicie sesión con su cuenta de Microsoft (MSA). Esta cuenta ya no se puede asociar con otra cuenta de centro de partners. 
3. Para completar la configuración de la cuenta de pago, registro completamente fuera del centro de partners, a continuación, vuelva a iniciarla con su Account de Microsoft (en lugar de su cuenta profesional). 

Ahora que está asociada la Account de Microsoft, para agregar una cuenta de pago, deberá:
- **Elija un método de pago**: Cuenta bancaria o PayPal
- **Agregar información de pago**: Esto puede incluir la elección de un tipo de cuenta (cheques o ahorro), escriba el nombre del titular de la cuenta, el número de cuenta y enrutamiento de número, dirección de facturación, número de teléfono o dirección de correo electrónico de PayPal. * Para obtener más información sobre el uso de PayPal como el método de pago de su cuenta y para averiguar si se admite en la región del mercado, consulte [información de PayPal](https://docs.microsoft.com/windows/uwp/publish/setting-up-your-payout-account-and-tax-forms#paypal-info).

> [!IMPORTANT]
> El hecho de cambiar la cuenta de pago puede dar lugar a retrasos en los pagos de hasta un ciclo completo. Este retraso se produce porque es necesario comprobar el cambio de cuenta, tal como lo hacemos cuando la primera vez que configura la cuenta de pago. Aunque se le seguirá pagando el importe completo después de que se haya comprobado la cuenta, los pagos pendientes del ciclo actual se agregarán al siguiente.  

#### <a name="tax-profile"></a>Perfil de impuestos

Revisar su estado actual del perfil de impuestos, confirmar el correcto **tipo de entidad** y **su información fiscal certificado** se muestra. Seleccione **editar** para actualizar o realizar alguna necesarias formularios.

Para establecer el estado de impuestos, debe especificar el país de residencia y ciudadanía y complete los formularios de impuestos adecuada asociados a su país o región.

Independientemente de su país de residencia o ciudadanía, rellene los formularios de impuestos de los Estados Unidos para vender las ofertas a través de Microsoft. Los socios que cumplen determinados requisitos de residencia de los Estados Unidos deben rellenar un formulario de IRS W-9. Otros socios fuera de Estados Unidos deben rellenar un formulario de IRS W-8. Puede rellenar estos formularios en línea a medida que complete su perfil de impuestos.

Un número de identificación fiscal individuales de Estados Unidos (o NIF) no es necesario para recibir pagos de Microsoft o a impuestos Tratado ventajas de notificación.

Puede completar y enviar los formularios de impuestos electrónicamente en el centro de partners; en la mayoría de los casos, no es necesario imprimir y enviar todos los formularios.

Regiones y países diferentes tienen requisitos diferentes fiscales. La cantidad exacta que debe pagar los impuestos depende de los países y regiones donde de vender sus ofertas. Microsoft remite de ventas y un impuesto de uso en su nombre en algunos países. Estos países se identificará en el proceso de anunciar la oferta. En otros países, dependiendo de dónde se registran, debe remitir sales y uso de impuestos para sus ventas directamente a la autoridad fiscal competente local. Además, recibirá la recaudación de ventas puede ser pasivo como ingresos. Le recomendamos que ponerse en contacto con la autoridad para su país o región que mejor le permitirá determinar la información de impuestos adecuada para las transacciones de ventas de Microsoft.

##### <a name="withholding-rates"></a>Las tasas de retención
La información que envíe en los formularios de impuestos determina la tasa de retención de impuestos adecuado. La tasa de retención se aplica solo a las ventas que se realicen en Estados Unidos. ventas realizadas en las ubicaciones de Estados Unidos no están sujeto a retención. Las tasas de retención varían, pero para la mayoría de los desarrolladores registrar fuera de Estados Unidos, la velocidad predeterminada es 30%. Tiene la opción que permite reducir esta velocidad si su país ha tratado de un impuesto acordado con los Estados Unidos.

##### <a name="tax-treaty-benefits"></a>Ventajas de Tratado de impuestos
Si está fuera de Estados Unidos, puede aprovechar las ventajas de impuestos sobre las ventajas de Tratado. Estas ventajas varían de un país a otro y es posible que le permiten reducir la cantidad de los impuestos que retiene de Microsoft. Puede reclamar impuestos Tratado ventajas completando parte II del formulario W-8BEN. Se recomienda que se comunica con los recursos adecuados de su país o región para determinar si estas ventajas se aplican a usted.

[Más información sobre los detalles de impuestos para los desarrolladores de aplicaciones y de juegos de Windows y los editores de Azure Marketplace](https://docs.microsoft.com/windows/uwp/publish/tax-details-for-paid-apps).

#### <a name="payout-hold-status"></a>Estado de suspensión de pago

De forma predeterminada, Microsoft envía los pagos mensualmente. Sin embargo, tiene la opción de poner sus pagos en espera, lo que impediría enviar los pagos a su cuenta. Si decide colocar sus pagos en espera, vamos a seguir registrar los ingresos que ganan y proporcionar los detalles en su **resumen de pagos**. Sin embargo, no enviamos los pagos a tu cuenta hasta que quite la suspensión. 

Para colocar los pagos en espera, vaya a **configuración de la cuenta**. En **detalles financieros**, en el **estado de suspensión del pago** sección, activar o desactivar el control deslizante para **en**. Puede cambiar el estado de suspensión de pago en cualquier momento, pero tenga en cuenta que su decisión afectará a su próximo pago mensual. Por ejemplo, si desea mantener los pagos de abril, asegúrese de establecer su estado de suspensión de pagos en **en** antes del final de marzo.

Una vez que haya establecido su pago mantenga el estado como **en**, todos los pagos estará en suspensión hasta que activar o desactivar el control deslizante a **desactivar**. Al hacerlo, se incluyen durante el siguiente ciclo de pago mensual (siempre que se hayan cumplido los umbrales de pago es aplicable). Por ejemplo, si ha tenido sus pagos en espera, pero ¿les gusta tener un pago generado en junio, asegúrese de activar o desactivar el pago del estado de suspensión para **desactivar** antes de finales de mayo.

> [!NOTE]
> Su **estado de suspensión del pago** selección es válida para **todas** fuentes de ingresos que se pagan a través de Microsoft Partner Center, incluidos Azure Marketplace, AppSource, Microsoft Store, publicidad, etcetera.). No puede seleccionar los Estados de suspensión diferente para cada origen de los ingresos.

### <a name="devices"></a>Dispositivos

La configuración de administración de dispositivos se aplica solo a la publicación de UWP. [Más información](https://docs.microsoft.com/windows/uwp/publish/manage-account-settings-and-profile#additional-settings-and-info).

### <a name="tracking-guids"></a>GUID de seguimiento

Globalmente identificadores únicos (globales GUID) son los números de referencia exclusivo (con 32 dígitos hexadecimales) que pueden usarse para realizar un seguimiento del uso de Azure. 

Para crear identificadores únicos globales para el seguimiento, debe usar un generador de GUID. El equipo de Azure Storage ha creado un [formulario de generación de GUID](https://aka.ms/StoragePartners) que enviará por correo electrónico un GUID en el formato correcto. Este GUID se puede reutilizar en los distintos sistemas de seguimiento.

Se recomienda crear un GUID único para todos los canales de distribución y oferta de cada producto. Puede optar por usar un solo GUID para los múltiples canales de distribución del producto si no quiere que los informes se dividan.

Si implementa un producto mediante una plantilla y está disponible tanto en Azure Marketplace como en GitHub, puede crear y registrar 2 GUID distintos:

*   Producto A en Azure Marketplace
*   Producto A en GitHub

La creación de informes se realiza en función del valor de asociado comercial (id. de Partner de Microsoft) y de los GUID. También puede realizar un seguimiento de los GUID en un nivel más granular, alineación de cada plan dentro de su oferta.

Para obtener más información, consulte el [uso del cliente de Azure de seguimiento con GUID de preguntas más frecuentes sobre](https://docs.microsoft.com/azure/marketplace/azure-partner-customer-usage-attribution#faq).



## <a name="multi-user-account-management"></a>Administración de cuentas de varios usuarios

Centro de partners aprovecha [Azure Active Directory](https://docs.microsoft.com/azure/active-directory/fundamentals/active-directory-whatis) (Azure AD) para la administración y acceso a la cuenta de varios usuarios. Su organización Azure AD se asocia automáticamente con su cuenta de centro de partners como parte del proceso de inscripción. 

## <a name="manage-users"></a>Administrar usuarios

El **usuarios** sección del centro de partners (bajo **configuración de la cuenta**) vamos a usar Azure AD para administrar los usuarios, grupos y aplicaciones de Azure AD que tienen acceso a la cuenta del centro de partners. Tenga en cuenta que con el fin de administrar usuarios, debe haber iniciado sesión con su [cuenta profesional](./company-work-accounts.md) (el asociado de Azure AD del inquilino). Para administrar usuarios dentro de una cuenta de trabajo diferente o inquilino, necesitará cerrar sesión y vuelva a iniciarla como un usuario con **Manager** permisos en esa cuenta profesional o de inquilino. 

Una vez haya iniciado sesión con su cuenta profesional (inquilino de Azure AD), hacer lo siguiente:
- [Agregar o quitar usuarios](#add-or-remove-users)
- [Cambiar una contraseña de usuario](#change-a-user-password)
- [Agregar o quitar grupos](#add-or-remove-users)
- [Agregar o quitar aplicaciones de Azure AD](#add-new-azure-ad-applications)
- [Administrar las claves de una aplicación de Azure AD](#manage-keys-for-an-azure-ad-application)
- [Definir los permisos y roles de usuario](#define-user-roles-and-permissions)


Tenga en cuenta que todos los usuarios de centro de partners (incluidos los grupos y aplicaciones de Azure AD) deben tener una cuenta de trabajo activo en un [inquilino de Azure AD](#manage-tenants) que está asociado con la cuenta del centro de partners. 

### <a name="add-or-remove-users"></a>Incorporación o eliminación de usuarios

La cuenta debe tener [ **nivel administrador** ](#define-user-roles-and-permissions) permisos para el [(inquilino de Azure AD) de la cuenta profesional](./company-work-accounts.md) en que desee agregar o editar los usuarios.

#### <a name="add-existing-users"></a>Agregue usuarios existentes

Para agregar usuarios a la cuenta del centro de partners que ya existen en su empresa [(inquilino de Azure AD) de la cuenta profesional](./company-work-accounts.md):

1. Vaya a **usuarios** (bajo **configuración de la cuenta**) y seleccione **agregar usuarios**.
2. Seleccione uno o varios usuarios de la lista que aparece. Puede usar el cuadro de búsqueda para buscar usuarios específicos.
* Si selecciona más de un usuario para agregar a la cuenta del centro de partners, debe asignar el mismo rol o conjunto de permisos personalizados. Para agregar varios usuarios con roles y permisos diferentes, repita estos pasos para cada rol o conjunto de permisos personalizados.
3.  Cuando haya terminado de elegir los usuarios, haga clic en **agregar seleccionado**.
5.  En el **Roles** , especifique los roles o permisos personalizados para los usuarios seleccionados.
6.  Seleccione **Guardar**.

#### <a name="create-new-users"></a>Crear nuevos usuarios

Para crear cuentas de usuario nueva, debe tener una cuenta con [ **administrador Global** ](https://docs.microsoft.com/azure/active-directory/users-groups-roles/directory-assign-admin-roles) permisos. 

1. Vaya a **usuarios** (bajo **configuración de la cuenta**), seleccione **agregar usuarios**, a continuación, elija **crear nuevos usuarios**.
1. Escriba un nombre, apellidos y nombre de usuario para cada usuario nuevo. 
1. Si desea que el nuevo usuario para tener una cuenta de administrador global en el directorio de su organización, active la casilla denominada **convertir a este usuario administrador Global en Azure AD, con control total sobre todos los recursos de directorio**. Esto le dará al usuario acceso completo a todas las características administrativas de la empresa de Azure AD. Podrán agregar y administrar los usuarios de la cuenta de trabajo de su organización (inquilino de Azure AD), aunque no en Partner Center, a menos que se concede el rol o los permisos adecuados a la cuenta. 
1. Si ha seleccionado la casilla para **convertir este usuario a un administrador Global**, deberá proporcionar un **correo electrónico de recuperación de contraseña** para el usuario recuperar su contraseña si es necesario.
1. En el **la pertenencia a grupo** , seleccione cualquier grupo al que desea que pertenezca el usuario nuevo.
1. En el **Roles** , especifique los roles o permisos personalizados para el usuario.
1. Seleccione **Guardar**.

Crear un nuevo usuario en el centro de partners, también se creará una cuenta para ese usuario en la cuenta profesional (inquilino de Azure AD) a la que ha iniciado sesión. Realizar cambios en un nombre de usuario en el centro de partners hará que los mismos cambios en la cuenta de trabajo de su organización (inquilino de Azure AD).

#### <a name="invite-new-users-by-email"></a>Invitar a nuevos usuarios por correo electrónico

Para invitar a los usuarios que actualmente no forman parte de su cuenta de trabajo de empresa (inquilino de Azure AD) mediante correo electrónico, debe tener una cuenta con [ **administrador Global** ](https://docs.microsoft.com/azure/active-directory/users-groups-roles/directory-assign-admin-roles) permisos. 

1. Vaya a **usuarios** (bajo **configuración de la cuenta**), seleccione **agregar usuarios**, a continuación, elija **invitar a usuarios por correo electrónico**.
2. Escriba el correo electrónico de uno o más direcciones (hasta diez), separados por comas o punto y coma.
3. En el **Roles** , especifique los roles o permisos personalizados para el usuario.
4. Seleccione **Guardar**.

Los usuarios que invitó obtendrá una invitación por correo electrónico para unirse a la cuenta del centro de partners. Se creará una nueva cuenta de usuario de invitado en su cuenta profesional (inquilino de Azure AD). Cada usuario deberá aceptar la invitación antes de que puede acceder a su cuenta.

Si necesita volver a enviar una invitación, visite la **usuarios** , busque la invitación en la lista de usuarios, seleccione su dirección de correo electrónico (o el texto que dice *invitación pendiente*). A continuación, en la parte inferior de la página, seleccione **Enviar invitación**.
 

> [!NOTE]
> Si su organización usa [integración de directorios](https://go.microsoft.com/fwlink/p/?LinkID=724033) para sincronizar el servicio de directorio locales con Azure AD, no podrá crear nuevos usuarios, grupos o aplicaciones de Azure AD en el centro de partners. Usted (u otro administrador en el directorio local) deberá crearlas directamente en el directorio local antes de, podrá ver y agregarlos en el centro de partners.

#### <a name="remove-a-user"></a>Eliminación de un usuario

Para quitar un usuario de su cuenta profesional (inquilino de Azure AD), vaya a **usuarios** (bajo **configuración de la cuenta**), seleccione el usuario que le gustaría quitar con la casilla de verificación de la columna más a la derecha, a continuación, elija  **Quitar** de las acciones disponibles. Se mostrará una ventana emergente para que confirmar que desea quitar los usuarios seleccionados.

#### <a name="change-a-user-password"></a>Cambiar una contraseña de usuario

Si uno de los usuarios debe cambiar su contraseña, puede hacerlo ellos mismos si proporcionó una **correo electrónico de recuperación de contraseña** al crear la cuenta de usuario. También puede actualizar una contraseña de usuario siguiendo los pasos siguientes. Para cambiar la contraseña de un usuario en su cuenta de trabajo de la empresa (inquilino de Azure AD), debe iniciado sesión en una cuenta con [ **administrador Global** ](https://docs.microsoft.com/azure/active-directory/users-groups-roles/directory-assign-admin-roles) permisos. Tenga en cuenta que esta operación cambiará la contraseña del usuario en su inquilino de Azure AD, junto con la contraseña usen para tener acceso a centro de partners.

1.  Desde el **usuarios** página (en **configuración de la cuenta**), seleccione el nombre de la cuenta de usuario que desea editar.
2.  Seleccione el **restablecer contraseña** situado en la parte inferior de la página.
3.  Aparecerá una página de confirmación que muestra la información de inicio de sesión del usuario, incluida una contraseña temporal. No olvide imprimir o copiar esta información y proporcionársela al usuario, ya no podrá tener acceso a la contraseña temporal después de salir de esta página.


## <a name="manage-groups"></a>Administrar grupos

Los grupos permiten controlar varios roles de usuario y los permisos todos juntos.

#### <a name="add-an-existing-group"></a>Agregar un grupo existente

Para agregar que un grupo que ya existe en la cuenta de trabajo de su organización (inquilino de Azure AD) a la cuenta del centro de partners: 

1.  Desde el **usuarios** página (en **configuración de la cuenta**), seleccione **agregar grupos**.
2.  Seleccione uno o varios grupos de la lista que aparece. Puede usar el cuadro de búsqueda para buscar grupos específicos.
Si selecciona más de un grupo para agregar a la cuenta del centro de partners, debe asignar el mismo rol o conjunto de permisos personalizados. Para agregar varios grupos de roles o permisos diferentes, repita estos pasos para cada rol o conjunto de permisos personalizados.
3.  Cuando haya terminado de elegir los grupos, haga clic en **agregar seleccionado**.
4.  En el **Roles** , especifique los roles o permisos personalizados para los grupos seleccionados. Todos los miembros del grupo podrá acceder a su cuenta de centro de partners con los permisos que se aplica al grupo, independientemente de los roles y permisos asociados con su cuenta individual.
5.  Seleccione **Guardar**.

Cuando se agrega un grupo existente, todos los usuarios que es un miembro de ese grupo podrá acceder a su cuenta de centro de partners, con los permisos asociados con la función del grupo asignado.

#### <a name="add-a-new-group"></a>Agregar un nuevo grupo

Para agregar un nuevo grupo a la cuenta del centro de partners: 

1.  Desde el **usuarios** página (en **configuración de la cuenta**), seleccione **agregar grupos**.
2.  En la siguiente página, seleccione **nuevo grupo**.
3.  Escriba el nombre para mostrar para el nuevo grupo.
4.  Especifique los roles o permisos personalizados para el grupo. Todos los miembros del grupo podrá acceder a su cuenta de centro de partners con los permisos que se aplica aquí, independientemente de los roles y permisos asociados con su cuenta individual.
5.  Seleccione los usuarios para el nuevo grupo de la lista que aparece. Puede usar el cuadro de búsqueda para buscar usuarios específicos.
6.  Cuando haya terminado de seleccionar usuarios, haga clic en **agregar seleccionado** para agregarlos al grupo nuevo.
7.  Seleccione **Guardar**.

Tenga en cuenta que este nuevo grupo se creará en la cuenta de trabajo de su organización (inquilino de Azure AD), no solo en la cuenta del centro de partners.

#### <a name="remove-a-group"></a>Quitar un grupo

Para quitar un grupo de su cuenta profesional (inquilino de Azure AD), vaya a **usuarios** (bajo **configuración de la cuenta**), seleccione el grupo que desea quitar con la casilla de verificación de la columna más a la derecha, a continuación, elija  **Quitar** de las acciones disponibles. Se mostrará una ventana emergente para que confirmar que desea quitar los grupos seleccionados.

## <a name="manage-azure-ad-applications"></a>Administrar aplicaciones de Azure AD

Puede permitir que las aplicaciones o servicios que forman parte de Azure su compañía AD para acceder a su cuenta de centro de partners. 

#### <a name="add-existing-azure-ad-applications"></a>Agregar aplicaciones de Azure AD existentes 

Para agregar las aplicaciones que ya existen en Azure Active Directory de su empresa: 

1.  Desde el **usuarios** página (en **configuración de la cuenta**), seleccione **agregar aplicaciones de Azure AD**.
2.  En la lista que aparece, seleccione una o varias aplicaciones de Azure AD. Puede usar el cuadro de búsqueda para buscar específicos de aplicaciones de Azure AD. Si selecciona más de una aplicación de Azure AD para agregar a la cuenta del centro de partners, debe asignar el mismo rol o conjunto de permisos personalizados. Para agregar varias aplicaciones de Azure AD con distintos roles y permisos, repita estos pasos para cada rol o conjunto de permisos personalizados.
3.  Cuando termine de seleccionar las aplicaciones de Azure AD, haga clic en **agregar seleccionado**.
5.  En el **Roles** , especifique los roles o permisos personalizados para el seleccionado las aplicaciones de Azure AD.
6.  Seleccione **Guardar**.

#### <a name="add-new-azure-ad-applications"></a>Agregar Azure nuevas aplicaciones de AD 

Si desea conceder acceso de centro de partners a un Azure completamente nuevo cuenta de la aplicación de AD, puede crear uno en el **usuarios** sección. Tenga en cuenta que esto creará una nueva cuenta en su cuenta de trabajo de la empresa (inquilino de Azure AD), no solo en la cuenta del centro de partners. Si se utiliza principalmente esta aplicación de Azure AD para la autenticación del centro de partners y no es necesario a los usuarios para acceder a él directamente, puede escribir cualquier dirección válida para el **dirección URL de respuesta** y **App ID URI**, siempre no se utilizan como los valores por cualquier otra aplicación de Azure AD en el directorio.

1.  Desde el **usuarios** página (en **configuración de la cuenta**), seleccione **agregar aplicaciones de Azure AD**.
2.  En la siguiente página, seleccione **New Azure AD application**.
3.  Escriba el **dirección URL de respuesta** para la nueva aplicación de Azure AD. Se trata de la dirección URL donde los usuarios pueden iniciar sesión y usar la aplicación de Azure AD (a veces, también conocido como dirección URL de aplicación o dirección URL de inicio de sesión). El **dirección URL de respuesta** no puede tener más de 256 caracteres y debe ser único dentro de su directorio.
4.  Escriba el **App ID URI** para la nueva aplicación de Azure AD. Se trata de un identificador lógico para la aplicación de Azure AD que se presenta cuando se envía una solicitud de inicio de sesión única con Azure AD. Tenga en cuenta que el **App ID URI** debe ser único para cada aplicación de Azure AD en el directorio. Este identificador no puede ser superior a 256 caracteres. Para obtener más información sobre el URI de Id. de aplicación, consulte [integración de aplicaciones con Azure Active Directory](https://docs.microsoft.com/azure/active-directory/develop/active-directory-integrating-applications#changing-the-application-registration-to-support-multi-tenant).
5.  En el **Roles** , especifique los roles o permisos personalizados para la aplicación de Azure AD.
6.  Seleccione **Guardar**.

Después de agregar o crear una aplicación de Azure AD, puede volver a la **usuarios** sección y seleccione el nombre de la aplicación para revisar la configuración de la aplicación, incluido el Id. de inquilino, Id. de cliente, dirección URL de respuesta y URI de Id. de aplicación.

#### <a name="remove-an-application"></a>Eliminación de una aplicación

Para quitar una aplicación de su cuenta profesional (inquilino de Azure AD), vaya a **usuarios** (bajo **configuración de la cuenta**), seleccione la aplicación que le gustaría quitar con la casilla de verificación de la columna más a la derecha, a continuación, elija **quitar** de las acciones disponibles. Se mostrará una ventana emergente para que confirmar que desea quitar las aplicaciones seleccionadas.

#### <a name="manage-keys-for-an-azure-ad-application"></a>Administrar las claves de una aplicación de Azure AD

Si la aplicación de Azure AD lee y escribe datos en Microsoft Azure AD, necesitará una clave. Puede crear claves para una aplicación de Azure AD mediante la edición de su información en el centro de partners. También puede quitar las claves que ya no son necesarios.

1.  Desde el **usuarios** página (en **configuración de la cuenta**), seleccione el nombre de la aplicación de Azure AD. Verá todas las claves de la aplicación de Azure AD, incluida la fecha en que se creó la clave y cuándo expirará activas. 
2. Para quitar una clave que ya no es necesario, seleccione **quitar**.
3.  Para agregar una nueva clave, seleccione **agregar nueva clave**.
4.  Verá una pantalla que muestra la **Id. de cliente** y **valores de clave**. Asegúrese de imprimir ni copiar esta información, ya no podrá acceder a ella nuevamente después de salir de esta página.
4.  Seleccione si desea crear más claves, **agregar otra clave**.


### <a name="define-user-roles-and-permissions"></a>Definir los permisos y roles de usuario

Los usuarios de su empresa se pueden asignar los siguientes roles y permisos en el programa de Marketplace comercial en el centro de partners. 

Tenga en cuenta los roles de inquilino de Azure Active Directory (AAD) incluyen el administrador global, Administrador de usuarios y roles CSP. Roles de AAD de no son los roles que no administran al inquilino e incluyen administrador MPN, admin de perfil de empresa, Administrador de referencias, incentivo administrador y usuario incentivo.


|**Rol**|**Permisos**|
|----------------------------------|:---------------------------------|
|Administrador global|• Puede tener acceso a todos los cuenta de Microsoft y servicios con privilegios completos
|      |• Crear incidencias de soporte técnico para el centro de partners
||• Vista acuerdos, listas de precios y ofertas
||• Ver, crear y administrar los usuarios del asociado|
|Administrador|• Puede tener acceso a todas las características de la cuenta de Microsoft, excepto las opciones de pago y fiscal
|      |• Puede administrar usuarios, roles, cuentas y trabajo (inquilinos)|
|Desarrollador|• Puede cargar paquetes, enviar las aplicaciones y los complementos y ver el informe de uso para obtener más información de telemetría
|      |• No se puede obtener acceso a la configuración de cuenta o información financiera|
|Colaborador de la empresa|• Puede tener acceso a información financiera y establecer los detalles de precios
|      |• No se puede crear o enviar nuevas aplicaciones y complementos|
|Colaborador financiero|• Puede ver los informes de pago
|      |• No se puede realizar cambios en las aplicaciones o configuración|
|Comerciante|• Puede responder a las revisiones de cliente y los informes no financieros
|      |• No se puede realizar cambios en las aplicaciones o configuración|

Para obtener más información acerca de cómo administrar roles y permisos en otras áreas del centro de partners, como Azure Active Directory (AD), el proveedor de soluciones en la nube (CSP), Panel de Control proveedor (CPV), los usuarios invitados o Microsoft Partner Network (MPN), consulte [asignar roles de los usuarios y permisos en el centro de partners](https://docs.microsoft.com/partner-center/permissions-overview).


## <a name="manage-tenants"></a>Administración de inquilinos

Un inquilino de Azure Active Directory (AD), que también se denomina la "cuenta profesional" en esta documentación es una representación de su organización en Azure portal y le ayuda a administrar una instancia específica de cloud services internos de Microsoft y los usuarios externos. Si su organización se suscribe a un servicio de nube de Microsoft, como Azure, Microsoft Intune u Office 365, se estableció un inquilino de Azure AD para usted. 

Puede configurar varios inquilinos para usar con el centro de partners. Cualquier usuario con el **Manager** rol en la cuenta del centro de partners tendrá la opción de agregar y quitar los inquilinos de Azure AD de la cuenta.  

### <a name="add-an-existing-tenant"></a>Agregar a un inquilino existente

Para asociar a otro inquilino de Azure AD con su cuenta de centro de partners:

1.  Desde el **inquilinos** página (en **configuración de la cuenta**), seleccione **asociar otro inquilino de Azure AD**.
2. Escriba sus credenciales de Azure AD para el inquilino que desea asociar.
3. Revise el nombre de organización y dominio para el inquilino de Azure AD. Para completar la asociación, seleccione **confirmar**.

Si la asociación se realiza correctamente, a continuación, estará listo para agregar y administrar los usuarios de cuentas en el **usuarios** sección en el centro de partners.

### <a name="create-a-new-tenant"></a>Crear un nuevo inquilino

Para crear a un inquilino de Azure AD nueva marca con la cuenta del centro de partners:

1.  Desde el **inquilinos** página (en **configuración de la cuenta**), seleccione **crear un nuevo inquilino de Azure AD**.
2. Escriba la información del directorio para el nuevo Azure AD:
    - **Nombre de dominio**: El nombre único que vamos a usar para el dominio de Azure AD, junto con ". onmicrosoft.com". Por ejemplo, si escribió "ejemplo", el dominio de AD Azure sería "example.onmicrosoft.com".
    - **Póngase en contacto con el correo electrónico**: Una dirección de correo electrónico donde podemos enviarte sobre su cuenta si es necesario.
    - **Información de cuenta de usuario de administrador global**: El nombre, último nombre, nombre de usuario y contraseña que desea usar para la nueva cuenta de administrador global.
3. Seleccione **crear** para confirmar la nueva información de dominio y la cuenta.
4. Inicie sesión con su Azure nuevo nombre de usuario de administrador global de AD y una contraseña para empezar a [agregar y administrar usuarios](#manage-users).

Para obtener más información sobre la creación de nuevos inquilinos en Azure portal, en lugar de a través del portal del centro de partners, consulte el artículo [crear un nuevo inquilino en Azure Active Directory](https://docs.microsoft.com/azure/active-directory/fundamentals/active-directory-access-create-new-tenant).

### <a name="remove-a-tenant"></a>Quitar un inquilino

Para quitar un inquilino de su cuenta de centro de partners, busque su nombre en el **inquilinos** página (en **configuración de la cuenta**), a continuación, seleccione **quitar**. Se le pedirá que confirme que desea quitar al inquilino. Una vez hecho esto, ningún usuario en ese inquilino podrán iniciar sesión en la cuenta del centro de partners, y se quitarán los permisos que ha configurado para esos usuarios.

Cuando se quita un inquilino, todos los usuarios que se han agregado a la cuenta del centro de partners de ese inquilino ya no podrá iniciar sesión en la cuenta.

> [!TIP]
> No se puede quitar a un inquilino si has iniciado sesión en el centro de partners mediante una cuenta en el mismo inquilino. Para quitar un inquilino, debe iniciar sesión en el centro de partners como un **Manager** para otro inquilino que está asociado con la cuenta. Si hay un solo inquilino asociado con la cuenta, solo se puede quitar ese inquilino después de iniciar sesión con la cuenta de Microsoft que abrió la cuenta.


## <a name="agreements"></a>Acuerdos

El **acuerdos** sección del centro de partners (bajo **configuración de la cuenta**) vamos a ver una lista de los contratos de publicación que hayan autorizado. Estos contratos se enumeran según el nombre y número de versión, incluida la fecha en que se aceptó y el nombre del usuario que ha aceptado el contrato. 

**Las acciones necesarias** puede aparecer en la parte superior de esta página si hay actualizaciones del contrato que requieren su atención. Para que acepte un contrato actualizado, leer la versión de contrato vinculado y seleccione **acepte el contrato**. 

Para obtener información sobre los contratos de proveedor de soluciones en la nube (CSP) en el centro de partners, visite [acuerdos en la nube de Microsoft por región e idioma](https://docs.microsoft.com/partner-center/agreements).

## <a name="next-steps"></a>Pasos siguientes

- [Cree una nueva oferta de SaaS](./create-new-saas-offer.md)
