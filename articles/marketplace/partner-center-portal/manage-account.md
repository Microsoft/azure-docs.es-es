---
title: Cómo administrar una cuenta de marketplace comercial en el Centro de partners del Microsoft
description: Aprenda a administrar una cuenta de marketplace comercial en el Centro de partners del Microsoft.
ms.service: marketplace
ms.subservice: partnercenter-marketplace-publisher
ms.topic: conceptual
author: parthpandyaMSFT
ms.author: parthp
ms.date: 04/27/2020
ms.openlocfilehash: 8362019365aab002929cc3414e6ca742952fc8ae
ms.sourcegitcommit: a76ff927bd57d2fcc122fa36f7cb21eb22154cfa
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/28/2020
ms.locfileid: "87320024"
---
# <a name="manage-your-commercial-marketplace-account-in-partner-center"></a>Administración de la cuenta de marketplace comercial en el Centro de partners

Después de [crear una cuenta del Centro de partners](./create-account.md), puede usar el [panel de marketplace comercial](https://partner.microsoft.com/dashboard/commercial-marketplace/overview) para administrar su cuenta y las ofertas.

En este artículo, se analizará en profundidad cómo administrar su cuenta del Centro de partners, incluyendo cómo:

- [Acceder a la configuración de la cuenta del Centro de partners](#access-your-account-settings)
- [Buscar los identificadores de anunciante, Symantec, vendedor, usuario y MPN, y los inquilinos de Azure AD](#account-details)
- [Actualizar la información de contacto](#contact-info)
- [Configurar GUID de seguimiento para supervisar el uso del cliente](#tracking-guids)
- [Administrar usuarios](#manage-users)
- [Administración de grupos](#manage-groups)
- [Administración de aplicaciones de Azure AD](#manage-azure-ad-applications)
- [Definir roles y permisos de usuario](#define-user-roles-and-permissions)
- [Administrar inquilinos de Azure AD (cuentas profesionales)](#manage-tenants)
- [Administrar acuerdos del Centro de partners](#agreements)

## <a name="access-your-account-settings"></a>Acceso a la configuración de cuenta

Si aún no lo ha hecho, usted (o el administrador de la organización) deben acceder a la [configuración de la cuenta](https://partner.microsoft.com/dashboard/account/management) del Centro de partners para:
- Confirmar el estado de comprobación de la cuenta de su empresa.
- Confirmar los id. de Symantec, vendedor, Microsoft Partner Network (MPN) y anunciante, junto con la información de contacto, como el contacto del aprobador y el vendedor de la empresa.
- Crear las cuentas de usuario de todos los que vayan a usar su cuenta empresarial en el Centro de partners.

### <a name="open-developer-settings"></a>Abrir configuración de desarrollador

La **Configuración de la cuenta** se encuentra en la esquina superior derecha del [panel de marketplace comercial](https://partner.microsoft.com/dashboard/commercial-marketplace) en el Centro de partners. Seleccione el icono de engranaje (cerca de la esquina superior derecha del panel) y, a continuación, seleccione **Configuración de desarrollador**.

![Menú Configuración de cuenta del Centro de partners](./media/dashboard-developer-settings.png)

En **Configuración de cuenta**, podrá ver:
- **Detalles de la cuenta**: tipo de cuenta y estado de la misma
- **Identificadores del editor**: identificador del vendedor, del usuario, del editor, inquilinos de Azure AD, etc.
- **Información de contacto**: nombre para mostrar del anunciante y nombre de contacto, correo electrónico, teléfono y dirección del vendedor.
- **GUID de seguimiento**: todos los GUID de seguimiento asociados a su cuenta

### <a name="account-details"></a>Detalles de la cuenta

En la sección Detalles de la cuenta, puede ver información básica, como el **tipo de cuenta** (personal o empresarial) y el **estado de comprobación** de la cuenta. Durante el proceso de comprobación de la cuenta, esta configuración mostrará todos los pasos que sean necesarios, incluidas la comprobación de correo electrónico, la comprobación de empleo y la comprobación de negocio. También puede actualizar el correo electrónico aquí y reenviar la comprobación si es necesario.

### <a name="publisher-ids"></a>Identificadores del editor

En la sección de identificadores del anunciante, puede ver los valores de **Id. de Symantec**, **Id. de vendedor**, **Id. de usuario** e **Id. de MPN**, así como los **inquilinos de Azure AD**. Microsoft asigna estos valores para identificar de manera exclusiva su cuenta de desarrollador y no se pueden modificar.

### <a name="contact-info"></a>Información de contacto

En la sección de información de contacto, puede ver el **nombre para mostrar del publicador**, **información de contacto del vendedor** (el nombre de contacto, correo electrónico, número de teléfono y dirección del vendedor de la empresa) y el **aprobador de la empresa** (el nombre, correo electrónico y número de teléfono de la persona con autoridad para aprobar las decisiones de la empresa).

#### <a name="payout-account"></a>Cuenta de pago

Una cuenta de pago es la cuenta bancaria a la que se envían los ingresos de sus ventas. Esta cuenta bancaria debe estar en el mismo país o región en el que se ha registrado su cuenta del Centro de partners.

Para configurar la cuenta de pago:

1. Vaya a la [página de información general de marketplace comercial](https://partner.microsoft.com/dashboard/commercial-marketplace/overview) en el Centro de partners.
2. En la sección Perfil, junto a **Payout Profile** (Perfil de pago), seleccione **Actualizar**.
3. **Elegir una forma de pago**: Cuenta bancaria o PayPal.
4. **Agregar la información de pago**: Puede que esto incluya la elección de un tipo de cuenta (cheques o ahorro), y especificar el nombre, número de cuenta y de identificación bancaria, dirección de facturación, número de teléfono o dirección de correo electrónico de PayPal, del titular de la cuenta. *Para más información sobre el uso de PayPal como forma de pago de la cuenta y para averiguar si esto es compatible con su región, consulte [información de PayPal](https://docs.microsoft.com/windows/uwp/publish/setting-up-your-payout-account-and-tax-forms#paypal-info).

> [!IMPORTANT]
> El hecho de cambiar la cuenta de pago puede dar lugar a retrasos en los pagos de hasta un ciclo completo. Este retraso se produce porque es necesario comprobar el cambio de la cuenta, al igual que hicimos cuando configuró por primera vez la cuenta de pago. Aunque se le seguirá pagando el importe completo después de que se haya comprobado la cuenta, los pagos pendientes del ciclo actual se agregarán al siguiente.  

#### <a name="tax-profile"></a>Perfil fiscal

Revise el estado actual de su perfil fiscal y confirme que aparece el **Tipo de entidad** y la **Información de certificado fiscal** correctos. Seleccione **Editar** para actualizar o rellenar los formularios necesarios.

Para establecer el estado fiscal, debe especificar el país o región de residencia y la ciudadanía, y rellenar los formularios fiscales adecuados asociados a su país o región.

Independientemente de su país o región de residencia o ciudadanía, debe rellenar los formularios fiscales de los Estados Unidos para vender ofertas mediante Microsoft. Los asociados que cumplen determinados requisitos de residencia de los Estados Unidos deben rellenar un formulario IRS W-9. El resto de asociados de fuera de Estados Unidos deben rellenar un formulario de IRS W-8. Puede rellenar estos formularios en línea a medida que completa su perfil fiscal.

No es necesario tener el número de identificación fiscal individual (ITIN) de Estados Unidos para recibir pagos de Microsoft o para reclamar beneficios del tratado fiscal.

Puede completar y enviar los formularios fiscales electrónicamente desde el Centro de partners. En la mayoría de los casos, no es necesario imprimir y enviar por correo ningún formulario.

Los distintos países y regiones tienen requisitos fiscales diferentes. El importe exacto que debe pagar en impuestos dependerá de los países y regiones en los que desea vender sus ofertas. Microsoft remite las ventas y el impuesto sobre el consumo en su nombre en algunos países o regiones. Estos países o regiones se identificarán durante el proceso de publicación de la oferta. En otros países o regiones, según donde esté registrado, puede que tenga que remitir las ventas y los impuestos sobre el consumo directamente ante la autoridad fiscal local. Además, los ingresos por ventas que recibe pueden estar sujetos a impuestos como ingresos. Le recomendamos encarecidamente que se ponga en contacto con la autoridad pertinente del país o región que mejor le pueda ayudar a determinar la información fiscal adecuada para sus transacciones de ventas de Microsoft.

##### <a name="withholding-rates"></a>Tasas de retención
La información que envíe en las declaraciones de impuestos determinará la tasa de retención fiscal adecuada. La tasa de retención se aplica solo a las ventas que se realizan en Estados Unidos. Las ventas realizadas en ubicaciones fuera de Estados Unidos no están sujetas a retenciones. Las tasas de retención varían, pero para la mayoría de los desarrolladores que están registrados fuera de Estados Unidos, la tasa predeterminada es del 30 %. Podrá reducir esta tasa si su país o región ha firmado con Estados Unidos un tratado tributario por ingresos.

##### <a name="tax-treaty-benefits"></a>Beneficios por tratado tributario
Si reside fuera de Estados Unidos, puede aprovechar los beneficios por tratados tributarios. Estos beneficios varían según el país o la región, y puede que le permitan reducir el importe de los impuestos que retiene Microsoft. Puede reclamar los beneficios por tratado tributario rellenando la Parte II del formulario W-8BEN. Es recomendable que se ponga en contacto con los recursos adecuados de su país o región para determinar si estos beneficios se le pueden aplicar.

[Más información sobre detalles fiscales para desarrolladores de aplicaciones y juegos de Windows y editores de Azure Marketplace](https://docs.microsoft.com/windows/uwp/publish/tax-details-for-paid-apps).

#### <a name="payout-hold-status"></a>Estado de retención de pagos

De forma predeterminada, Microsoft envía los pagos mensualmente. No obstante, tiene la opción de poner sus pagos en espera, lo que impediría que se envíen pagos a su cuenta. Si decide retener los pagos, se siguen registrando los ingresos que genera y se le proporcionan los detalles en el **Resumen de pagos**. Pero no se le envía ningún pago a la cuenta hasta que quite la retención. 

Para poner los pagos en espera, vaya a **Configuración de la cuenta**. En **Detalles financieros**, en la sección **Estado de retención de pagos**, seleccione **Activado** en el control deslizante. Puede cambiar el estado de retención de pagos en cualquier momento, pero tenga en cuenta que su decisión afectará a su próximo pago mensual. Por ejemplo, si quiere retener los pagos de abril, asegúrese de establecer el estado de retención de pagos en **Activado** antes del final de marzo.

Una vez que haya establecido el estado de la retención de pagos en **Activado**, todos los pagos se pondrán en espera hasta que desplace el control deslizante de nuevo a **Desactivado**. Una vez hecho esto, se le incluye en el siguiente ciclo de pago mensual (siempre que se hayan cumplido los umbrales de pago aplicables). Por ejemplo, si ha retenido los pagos pero quiere que se genere un pago en junio, asegúrese de alternar el estado de retención de pagos a **Desactivado** antes de finales de mayo.

> [!NOTE]
> La selección del **Estado de retención de pagos** se aplica a **todas** las fuentes de ingresos que se pagan a través del Centro de partners de Microsoft, incluidos Azure Marketplace, AppSource, Microsoft Store, publicidad, etc). No puede seleccionar estados de retención de pagos diferentes para cada fuente de ingresos.

### <a name="devices"></a>Dispositivos

La configuración de administración de dispositivos se aplica solo a la publicación en UWP. [Más información](https://docs.microsoft.com/windows/uwp/publish/manage-account-settings-and-profile#additional-settings-and-info).

### <a name="tracking-guids"></a>GUID de seguimiento

Los identificadores únicos globales (GUID) son números de referencia exclusivos (con 32 dígitos hexadecimales) que se pueden utilizar para realizar el seguimiento del uso de Azure. 

Para crear identificadores únicos globales para el seguimiento, debe usar un generador de GUID. El equipo de Azure Storage ha creado un [formulario de generación de GUID](https://aka.ms/StoragePartners) que enviará por correo electrónico un GUID en el formato correcto. Este GUID se puede reutilizar en los distintos sistemas de seguimiento.

Se recomienda crear un GUID único para todos los canales de distribución y oferta de cada producto. Puede optar por usar un solo GUID para los múltiples canales de distribución del producto si no quiere que los informes se dividan.

Si implementa un producto mediante una plantilla y está disponible tanto en Azure Marketplace como en GitHub, puede crear y registrar dos GUID distintos:

*    Producto A en Azure Marketplace
*    Producto A en GitHub

La creación de informes se realiza en función del valor de asociado comercial (id. de Partner de Microsoft) y de los GUID. También puede realizar un seguimiento de los GUID a un nivel más pormenorizado para cada plan de la oferta.

Para más información, consulte [Tracking Azure customer usage with GUIDs FAQ](../azure-partner-customer-usage-attribution.md#faq) (Preguntas más frecuentes sobre el seguimiento del uso por parte de los clientes de Azure mediante los GUID).

## <a name="create-a-billing-profile"></a>Creación de un perfil de facturación

Si va a publicar una oferta de [Dynamics 365 for Customer Engagement y Power Apps](./create-new-customer-engagement-offer.md) o [Dynamics 365 for Operations](./create-new-operations-offer.md) tendrá que rellenar el **perfil de facturación**.

La dirección de facturación se rellena previamente a partir de la entidad legal y puede actualizarla más adelante. Los campos Impuestos y CIF son opcionales.  No se puede editar el nombre del país o región y el nombre de la empresa.

## <a name="multi-user-account-management"></a>Administración de cuentas multiusuario

El Centro de partners utiliza [Azure Active Directory](../../active-directory/fundamentals/active-directory-whatis.md) (Azure AD) para la administración y el acceso a la cuenta multiusuario. La instancia de Azure AD de la organización se asocia automáticamente con la cuenta del Centro de partners como parte del proceso de inscripción.

## <a name="manage-users"></a>Administrar usuarios

La sección **Usuarios** del Centro de partners (en **Configuración de la cuenta**) le permite usar Azure AD para administrar los usuarios, grupos y aplicaciones de Azure AD que tienen acceso a esa cuenta. La cuenta debe tener permisos con [**nivel de administrador**](#define-user-roles-and-permissions) sobre la [cuenta profesional (inquilino de Azure AD)](./company-work-accounts.md) en la que desea agregar o editar usuarios. Para administrar los usuarios de una cuenta o inquilino profesional diferente, necesitará cerrar sesión y volver a iniciarla como un usuario con permisos de **administrador** sobre esa cuenta profesional o de inquilino.

Una vez haya iniciado sesión con su cuenta profesional (inquilino de Azure AD), puede:

- [Agregar o eliminar usuarios](#add-existing-users)
- [Cambiar una contraseña de usuario](#change-a-user-password)
- [Agregar o eliminar grupos](#manage-groups)
- [Agregar o eliminar aplicaciones de Azure AD](#add-new-azure-ad-applications)
- [Administrar claves de una aplicación de Azure AD](#manage-keys-for-an-azure-ad-application)
- [Definir roles y permisos de usuario](#define-user-roles-and-permissions)

Tenga en cuenta que todos los usuarios del Centro de partners (incluidos los grupos y aplicaciones de Azure AD) deben tener una cuenta profesional activa en un [inquilino de Azure AD](#manage-tenants) que esté asociado a la cuenta del Centro de partners.

### <a name="add-existing-users"></a>Incorporación de usuarios ya existentes

Para agregar usuarios a la cuenta del Centro de partners que ya existen en la [cuenta profesional (inquilino de Azure AD)](./company-work-accounts.md) de la empresa:

1. Vaya a **Usuarios** (en **Configuración de la cuenta**) y seleccione **Agregar usuarios**.
2. Seleccione uno o varios usuarios en la lista que aparece. Puede usar el cuadro de búsqueda para buscar usuarios concretos.
*Si selecciona más de un usuario para agregar a la cuenta del Centro de partners, debe asignarles el mismo rol o conjunto de permisos personalizados. Para agregar varios usuarios con roles y permisos diferentes, repita estos pasos para cada rol o conjunto de permisos personalizados.
3. Cuando haya terminado de elegir los usuarios, haga clic en **Agregar selección**.
4. En la sección **Roles**, especifique los roles o permisos personalizados de los usuarios seleccionados.
5. Seleccione **Guardar**.

### <a name="create-new-users"></a>Creación de nuevos usuarios

Para crear nuevas cuentas de usuario, debe tener una cuenta con permisos de [**administrador global**](../../active-directory/users-groups-roles/directory-assign-admin-roles.md). 

1. Vaya a **Usuarios** (en **Configuración de la cuenta**), seleccione **Agregar usuarios** y, a continuación, elija **Crear nuevos usuarios**.
1. Escriba un nombre, apellidos y nombre de usuario para cada usuario nuevo. 
1. Si desea que el nuevo usuario tenga una cuenta de administrador global en el directorio de la organización, active la casilla denominada **Convertir a este usuario en administrador global de Azure AD, con control total sobre todos los recursos del directorio**. Esto le dará al usuario acceso completo a todas las características administrativas de la instancia de Azure AD de la empresa. Podrá agregar y administrar usuarios en la cuenta profesional de la organización (inquilino de Azure AD), aunque no en el Centro de partners, a menos que otorgue los permisos y roles adecuados para la cuenta de este.
1. Si ha seleccionado la casilla **Convertir a este usuario en administrador global**, deberá proporcionar un **correo electrónico de recuperación de contraseña** para que el usuario pueda recuperar su contraseña si es necesario.
1. En la sección **Pertenencia a grupos**, seleccione los grupos a los que desee que pertenezca el nuevo usuario.
1. En la sección **Roles**, especifique los roles o permisos personalizados del usuario.
1. Seleccione **Guardar**.

La creación de un nuevo usuario en el Centro de partners, también crea una cuenta para ese usuario en la cuenta profesional (inquilino de Azure AD) en la que ha iniciado sesión. Si realiza cambios en el nombre de un usuario del Centro de partners, los mismos cambios se realizarán en la cuenta profesional de la organización (inquilino de Azure AD).

### <a name="invite-new-users-by-email"></a>Invitar a nuevos usuarios por correo electrónico

Para invitar a los usuarios que actualmente no forman parte de la cuenta profesional de la empresa (inquilino de Azure AD) mediante correo electrónico, debe tener una cuenta con permisos de [**administrador global**](../../active-directory/users-groups-roles/directory-assign-admin-roles.md).

1. Vaya a **Usuarios** (en **Configuración de la cuenta**), seleccione **Agregar usuarios** y, a continuación, elija **Invite users by email** (Invitar usuarios por correo electrónico).
2. Escriba una o varias direcciones de correo electrónico (hasta diez), separadas por comas o por puntos y comas.
3. En la sección **Roles**, especifique los roles o permisos personalizados del usuario.
4. Seleccione **Guardar**.

Los usuarios recibirán una invitación por correo electrónico para unirse a la cuenta del Centro de partners. Se creará una nueva cuenta de usuario invitado en su cuenta profesional (inquilino de Azure AD). Cada usuario deberá aceptar la invitación antes de poder acceder a la cuenta.

Si necesita volver a enviar una invitación, vaya a la página **Usuarios**, busque la invitación en la lista de usuarios y seleccione su dirección de correo electrónico (o el texto que dice *invitación pendiente*). A continuación, en la parte inferior de la página, seleccione **Volver a enviar la invitación**.

> [!NOTE]
> Si la organización usa [integración de directorios](https://go.microsoft.com/fwlink/p/?LinkID=724033) para sincronizar el servicio de directorio local con Azure AD, no podrá crear nuevos usuarios, grupos o aplicaciones de Azure AD en el Centro de partners. Usted (u otro administrador del directorio local) deberán crearlos directamente en el directorio local antes de poder verlos y agregarlos en el Centro de partners.

### <a name="remove-a-user"></a>Eliminación de un usuario

Para quitar un usuario de la cuenta profesional (inquilino de Azure AD), vaya a **Usuarios** (en **Configuración de la cuenta**), seleccione el usuario que desee eliminar mediante la casilla de verificación de la columna situada más a la derecha y, a continuación, elija **Quitar** en el listado de acciones disponibles. Aparecerá una ventana emergente para que confirme que desea eliminar los usuarios seleccionados.

### <a name="change-a-user-password"></a>Cambiar una contraseña de usuario

Si uno de los usuarios tiene que cambiar su contraseña, podrá hacerlo él mismo si proporcionó un **correo electrónico de recuperación de contraseña** al crear la cuenta de usuario. También puede actualizar la contraseña de un usuario mediante los pasos siguientes. Para cambiar la contraseña de un usuario en la cuenta profesional de la empresa (inquilino de Azure AD), debe iniciar sesión en una cuenta con permisos de [**administrador global**](../../active-directory/users-groups-roles/directory-assign-admin-roles.md). Tenga en cuenta que esta operación cambiará la contraseña del usuario en el inquilino de Azure AD y la contraseña que usa para acceder al Centro de partners.

1. En la página **Usuarios** (en **Configuración de la cuenta**), seleccione el nombre de la cuenta de usuario que desea editar.
2. Seleccione el botón **Restablecer contraseña** situado en la parte inferior de la página.
3. Aparecerá una página de confirmación que muestra la información de inicio de sesión del usuario, incluida una contraseña temporal. No olvide imprimir o copiar esta información y proporcionársela al usuario, ya que no podrá acceder a la contraseña temporal después de salir de esta página.

## <a name="manage-groups"></a>Administrar grupos

Los grupos le permiten controlar varios roles y permisos de usuario a la vez.

### <a name="add-an-existing-group"></a>Agregar un grupo existente

Para agregar un grupo que ya existe en la cuenta profesional (inquilino de Azure AD) de la organización a la cuenta del Centro de partners:

1. En la página **Usuarios** (en **Configuración de la cuenta**), seleccione **Agregar grupos**.
2. Seleccione uno o varios grupos en la lista que aparece. Puede usar el cuadro de búsqueda para buscar grupos concretos.
Si selecciona más de un grupo para agregar a la cuenta del Centro de partners, debe asignarles el mismo rol o conjunto de permisos personalizados. Para agregar varios grupos con roles y permisos diferentes, repita estos pasos para cada rol o conjunto de permisos personalizados.
3. Cuando haya terminado de elegir los grupos, haga clic en **Agregar selección**.
4. En la sección **Roles**, especifique los roles o permisos personalizados de los grupos seleccionados. Todos los miembros del grupo podrán acceder a la cuenta del Centro de partners con los permisos que aplique al grupo, independientemente de los roles y permisos asociados a su cuenta individual.
5. Seleccione **Guardar**.

Cuando se agrega un grupo existente, todos los usuarios que son miembros de ese grupo pueden acceder a su cuenta del Centro de partners con los permisos asociados al rol asignado al grupo.

### <a name="add-a-new-group"></a>Agregar un nuevo grupo

Para agregar un nuevo grupo a la cuenta del Centro de partners:

1. En la página **Usuarios** (en **Configuración de la cuenta**), seleccione **Agregar grupos**.
2. En la siguiente página, seleccione **Nuevo grupo**.
3. Escriba el nombre para mostrar del nuevo grupo.
4. Especifique los roles o permisos personalizados del grupo. Todos los miembros del grupo podrán acceder a la cuenta del Centro de partners con los permisos que aplique aquí, independientemente de los roles y permisos asociados a su cuenta individual.
5. Seleccione los usuarios del nuevo grupo en la lista que aparece. Puede usar el cuadro de búsqueda para buscar usuarios concretos.
6. Cuando haya terminado de seleccionar usuarios, haga clic en **Agregar selección** para agregarlos al grupo nuevo.
7. Seleccione **Guardar**.

Tenga en cuenta que este nuevo grupo se creará en la cuenta profesional (inquilino de Azure AD) de la organización, no solo en la cuenta del Centro de partners.

### <a name="remove-a-group"></a>Eliminar un grupo

Para quitar un grupo de la cuenta profesional (inquilino de Azure AD), vaya a **Usuarios** (en **Configuración de la cuenta**), seleccione el grupo que desee eliminar mediante la casilla de verificación de la columna situada más a la derecha y, a continuación, elija **Quitar** en el listado de acciones disponibles. Aparecerá una ventana emergente para que confirme que desea eliminar los grupos seleccionados.

## <a name="manage-azure-ad-applications"></a>Administración de aplicaciones de Azure AD

Puede permitir que las aplicaciones o servicios que forman parte de la instancia de Azure AD de la empresa accedan a su cuenta del Centro de partners.

### <a name="add-existing-azure-ad-applications"></a>Agregar aplicaciones de Azure AD existentes

Para agregar aplicaciones que ya existen en la instancia de Azure Active Directory de la empresa:

1. En la página **Usuarios** (en **Configuración de la cuenta**), seleccione **Agregar aplicaciones de Azure AD**.
2. Seleccione una o varias aplicaciones de Azure AD en la lista que aparece. Puede usar el cuadro de búsqueda para buscar aplicaciones de Azure AD concretas. Si selecciona más de una aplicación de Azure AD para agregar a la cuenta del Centro de partners, debe asignarles el mismo rol o conjunto de permisos personalizados. Para agregar varias aplicaciones de Azure AD con roles y permisos diferentes, repita estos pasos para cada rol o conjunto de permisos personalizados.
3. Cuando termine de seleccionar las aplicaciones de Azure AD, haga clic en **Agregar selección**.
4. En la sección **Roles**, especifique los roles o permisos personalizados de las aplicaciones de Azure AD seleccionadas.
5. Seleccione **Guardar**.

### <a name="add-new-azure-ad-applications"></a>Agregar nuevas aplicaciones de Azure AD

Si desea conceder acceso al Centro de partners a una nueva cuenta de aplicación de Azure AD, puede crear una en la sección **Usuarios**. Tenga en cuenta que esto creará una nueva cuenta en la cuenta profesional de la empresa (inquilino de Azure AD), no solo en la cuenta del Centro de partners. Si se utiliza principalmente esta aplicación de Azure AD para la autenticación en el Centro de partners y no necesita usuarios para acceder a ella directamente, puede especificar cualquier dirección válida como **URL de respuesta** y como **URI de id. de aplicación**, siempre y cuando ninguna otra aplicación de Azure AD use esos valores en el directorio.

1. En la página **Usuarios** (en **Configuración de la cuenta**), seleccione **Agregar aplicaciones de Azure AD**.
2. En la siguiente página, seleccione **New Azure AD application** (Nueva aplicación de Azure AD).
3. Especifique la **URL de respuesta** de la nueva aplicación de Azure AD. Esta es la dirección URL en la que los usuarios pueden iniciar sesión y usar la aplicación de Azure AD (también se denomina a veces como URL de la aplicación o URL de inicio de sesión). La **URL de respuesta** no puede tener más de 256 caracteres y debe ser única dentro del directorio.
4. Especifique el **URI de id. de aplicación** de la nueva aplicación de Azure AD. Se trata de un identificador lógico para la aplicación de Azure AD que se presenta cuando se envía una solicitud de inicio de sesión único a Azure AD. Tenga en cuenta que el **URI de id. de aplicación** debe ser exclusivo de cada aplicación de Azure AD del directorio. No puede tener más de 256 caracteres. Para más información sobre él, consulte [Integración de aplicaciones con Azure Active Directory](../../active-directory/develop/quickstart-modify-supported-accounts.md#change-the-application-registration-to-support-different-accounts).
5. En la sección **Roles**, especifique los roles o permisos personalizados de la aplicación de Azure AD.
6. Seleccione **Guardar**.

Después de agregar o crear una aplicación de Azure AD, puede volver a la sección **Usuarios** y seleccionar el nombre de la aplicación para revisar la configuración de la aplicación, incluido el identificador del inquilino y del cliente, la URL de respuesta y el URI de identificador de aplicación.

### <a name="remove-an-azure-ad-application"></a>Eliminación de una aplicación de Azure AD

Para quitar una aplicación de la cuenta profesional (inquilino de Azure AD), vaya a **Usuarios** (en **Configuración de la cuenta**), seleccione la aplicación que desee eliminar mediante la casilla de verificación de la columna situada más a la derecha y, a continuación, elija **Quitar** en el listado de acciones disponibles. Aparecerá una ventana emergente para que confirme que desea eliminar las aplicaciones seleccionadas.

### <a name="manage-keys-for-an-azure-ad-application"></a>Administración de claves de una aplicación de Azure AD

Si la aplicación de Azure AD lee y escribe datos en Microsoft Azure AD, necesitará una clave. Puede crear claves para una aplicación de Azure AD mediante la edición de su información en el Centro de partners. También puede eliminar las claves que ya no son necesarias.

1. En la página **Usuarios** (en **Configuración de la cuenta**), seleccione el nombre de la aplicación de Azure AD. Verá todas las claves activas de la aplicación de Azure AD, incluida la fecha de creación de la clave y cuando expirará. 
2. Para eliminar una clave que ya no es necesaria, seleccione **Eliminar**.
3. Para agregar una nueva clave, seleccione **Agregar nueva clave**.
4. Verá una pantalla que muestra el **id. de cliente** y los **valores de clave**. Asegúrese de imprimir o copiar esta información, ya que no podrá acceder a ella nuevamente después de salir de esta página.
5. Si desea crear más claves, seleccione **Add another key** (Agregar otra clave).

## <a name="define-user-roles-and-permissions"></a>Definición de roles y permisos de usuario

Se pueden asignar a los usuarios de la empresa los siguientes roles y permisos en el programa Marketplace comercial del Centro de partners:

- **Administrador**
  - Puede acceder a todas las características de la cuenta Microsoft, excepto a la configuración fiscal y de pagos.
  - Puede administrar usuarios, roles y cuentas profesionales (inquilinos).
- **Developer**
  - Puede administrar y publicar ofertas.
  - Puede ver algunos informes del publicador.

> [!NOTE]
> En el caso del programa Marketplace comercial, no se utilizan los roles Administrador global, Colaborador empresarial, Colaborador financiero y Vendedor. Asignar estos roles a los usuarios no tiene ningún efecto. Solo los roles Administrador y Desarrollador conceden permisos a los usuarios.

Para más información sobre la administración de roles y permisos en otras áreas del Centro de partners, como Azure Active Directory (AD), Cloud Solution Provider (CSP), Control Panel Vendor (CPV), usuarios invitados o Microsoft Partner Network (MPN), consulte [Asignación de roles y permisos de usuario en el Centro de partners](https://docs.microsoft.com/partner-center/permissions-overview).

## <a name="manage-tenants"></a>Administración de inquilinos

Un inquilino de Azure Active Directory (AD), también denominado como "cuenta profesional" en esta documentación, es una representación de la configuración de la organización en Azure Portal y le ayuda a administrar una instancia específica de los servicios en la nube de Microsoft para los usuarios internos y externos. Si la organización se suscribió a un servicio en la nube de Microsoft, como Azure, Microsoft Intune u Office 365, ya se estableció un inquilino de Azure AD.

Puede configurar varios inquilinos para su uso con el Centro de partners. Cualquier usuario con el rol **Administrador** en la cuenta del Centro de partners tendrá la opción de agregar y quitar inquilinos de Azure AD de la cuenta.  

### <a name="add-an-existing-tenant"></a>Agregar un inquilino existente

Para asociar otro inquilino de Azure AD con la cuenta del Centro de partners:

1. En la página **Inquilinos**, (en **Configuración de la cuenta**), seleccione **Associate another Azure AD tenant** (Asociar otro inquilino de Azure AD).
2. Escriba sus credenciales de Azure AD para el inquilino que desea asociar.
3. Revise el nombre de dominio y de la organización para el inquilino de Azure AD. Para completar la asociación, seleccione **Confirmar**.

Si la asociación se realiza correctamente, estará listo para agregar y administrar los usuarios de la cuenta en la sección **Usuarios** del Centro de partners.

### <a name="create-a-new-tenant"></a>Creación de un nuevo inquilino

Para crear un nuevo inquilino de Azure AD con la cuenta del Centro de partners:

1. En la página **Inquilinos**, (en **Configuración de la cuenta**), seleccione **Create a new Azure AD tenant** (Crear un nuevo inquilino de Azure AD).
2. Escriba la información del directorio para la nueva instancia de Azure AD:
    - **Nombre de dominio**: nombre único que se va a usar para el dominio de Azure AD, junto con ".onmicrosoft.com". Por ejemplo, si ha escrito "ejemplo", el dominio de Azure AD sería "ejemplo.onmicrosoft.com".
    - **Dirección de correo electrónico de contacto**: una dirección de correo electrónico para ponernos en contacto con información sobre la cuenta si es necesario.
    - **Información de la cuenta de usuario del administrador global**: nombre, apellidos, nombre de usuario y contraseña que desea utilizar para la nueva cuenta de administrador global.
3. Seleccione **Crear** para confirmar la nueva información de dominio y de cuenta.
4. Inicie sesión con el nuevo nombre de usuario y contraseña de administrador global de Azure AD para empezar a [agregar y administrar usuarios](#manage-users).

Para más información sobre la creación de nuevos inquilinos en Azure Portal, en lugar de mediante el portal del Centro de partners, consulte el artículo [Creación de un inquilino en Azure Active Directory](../../active-directory/fundamentals/active-directory-access-create-new-tenant.md).

### <a name="remove-a-tenant"></a>Eliminación de un inquilino

Para eliminar un inquilino de su cuenta del Centro de partners, busque su nombre en la página **Inquilinos** (en **Configuración de la cuenta**) y, a continuación, seleccione **Eliminar**. Se le pide que confirme que quiere eliminar el inquilino. Una vez hecho esto, ningún usuario de ese inquilino podrá iniciar sesión en la cuenta del Centro de partners y se eliminarán los permisos que haya configurado para esos usuarios.

Cuando se elimina un inquilino, todos los usuarios que se agregaron a la cuenta del Centro de partners de ese inquilino ya no podrán iniciar sesión en la cuenta.

> [!TIP]
> No se puede eliminar un inquilino si se ha iniciado sesión en el Centro de partners con una cuenta del mismo inquilino. Para eliminar un inquilino, debe iniciar sesión en el Centro de partners como **Administrador** de otro inquilino que esté asociado con la cuenta. Si solo hay un inquilino asociado con la cuenta, ese inquilino solo se podrá eliminar después de iniciar sesión con la cuenta Microsoft que abrió la cuenta.

## <a name="agreements"></a>Contratos

La sección **Contratos** del Centro de partners (en **Configuración de la cuenta**) le permite ver una lista de los contratos de publicación que ha autorizado. Estos contratos se enumeran según el nombre y número de versión, incluida la fecha en que se aceptó y el nombre del usuario que aceptó el contrato.

La sección **Se requieren acciones** puede aparecer en la parte superior de esta página si hay actualizaciones del contrato que requieren su atención. Para aceptar un contrato actualizado, lea primero la versión del contrato vinculado y seleccione **Aceptar el contrato**.
