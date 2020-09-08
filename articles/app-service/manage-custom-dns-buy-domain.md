---
title: Compra de un nombre de dominio personalizado
description: Aprenda a comprar un dominio de App Service y úselo como un dominio personalizado para una aplicación de Azure App Service.
ms.assetid: 70fb0e6e-8727-4cca-ba82-98a4d21586ff
ms.topic: article
ms.date: 11/24/2017
ms.custom: seodec18
ms.openlocfilehash: ae2d91233df25885bdfd765481f4bb6a1a36da37
ms.sourcegitcommit: 648c8d250106a5fca9076a46581f3105c23d7265
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/27/2020
ms.locfileid: "88958735"
---
# <a name="buy-a-custom-domain-name-for-azure-app-service"></a>Comprar un nombre de dominio personalizado para Azure App Service

Los dominios de App Service son dominios de nivel superior que se administran directamente en Azure. Facilitan la administración de dominios personalizados para [Azure App Service](overview.md). En este tutorial se muestra cómo comprar un dominio de App Service y asignar nombres DNS a Azure App Service.

Para Azure Virtual Machines o Azure Storage, vea [Assign App Service domain to Azure VM or Azure Storage](https://azure.github.io/AppService/2017/07/31/Assign-App-Service-domain-to-Azure-VM-or-Azure-Storage) (Asignación del dominio de App Service a Azure Virtual Machines o Azure Storage). Para Cloud Services, vea [Configuración de un nombre de dominio personalizado para un servicio en la nube de Azure](../cloud-services/cloud-services-custom-domain-name-portal.md).

## <a name="prerequisites"></a>Requisitos previos

Para completar este tutorial:

* [Cree una aplicación de App Service](./index.yml) o use alguna aplicación que haya creado para otro tutorial.
* [Elimine el límite de gasto en mi suscripción](../cost-management-billing/manage/spending-limit.md#remove). No puede comprar dominios de App Service con créditos de una suscripción gratuita.

## <a name="prepare-the-app"></a>Preparación de la aplicación

[!INCLUDE [app-service-dev-test-note](../../includes/app-service-dev-test-note.md)]

Para usar dominios personalizados en Azure App Service, el [plan de App Service](https://azure.microsoft.com/pricing/details/app-service/) de la aplicación debe ser un nivel de pago (**Compartido**, **Básico**, **Estándar** o **Premium**). En este paso, asegúrese de que la aplicación se encuentra en el plan de tarifa admitido.

### <a name="sign-in-to-azure"></a>Inicio de sesión en Azure

Abra [Azure Portal](https://portal.azure.com) e inicie sesión con su cuenta de Azure.

### <a name="navigate-to-the-app-in-the-azure-portal"></a>Navegue hasta la aplicación en Azure Portal

En el menú izquierdo, seleccione **App Services** y, después, el nombre de la aplicación.

![Navegación en el portal a la aplicación de Azure](./media/app-service-web-tutorial-custom-domain/select-app.png)

Consulte la página de administración de la aplicación de App Service.  

### <a name="check-the-pricing-tier"></a>Comprobar el plan de tarifa

En el panel de navegación izquierdo de la página de la aplicación, desplácese hasta la sección **Configuración** y seleccione **Escalar verticalmente (plan de App Service)** .

![Menú Escalar verticalmente](./media/app-service-web-tutorial-custom-domain/scale-up-menu.png)

El nivel actual de la aplicación aparece resaltado con un cuadro azul. Asegúrese de que la aplicación web no está en el nivel **F1**. No se admiten DNS personalizados en el nivel **F1**. 

![Comprobar plan de tarifa](./media/app-service-web-tutorial-custom-domain/check-pricing-tier.png)

Si el plan de App Service no es **F1**, cierre la página **Escalar verticalmente** y vaya directamente a [Buy the domain](#buy-the-domain) (Comprar el dominio).

### <a name="scale-up-the-app-service-plan"></a>Escalado vertical del plan de App Service

Seleccione cualquiera de los niveles no gratuitos (**D1**, **B1**, **B2**, **B3**, o cualquier nivel de la categoría **Producción**). Para ver opciones adicionales, haga clic en **Ver opciones adicionales**.

Haga clic en **Aplicar**.

![Comprobar plan de tarifa](./media/app-service-web-tutorial-custom-domain/choose-pricing-tier.png)

Cuando vea la siguiente notificación, significará que la operación de escalado se habrá completado.

![Confirmación de la operación de escalado](./media/app-service-web-tutorial-custom-domain/scale-notification.png)

## <a name="buy-the-domain"></a>Comprar el dominio

### <a name="pricing-information"></a>Información de precios
Para obtener información sobre precios de los dominios de Azure App Service, visite la [página de precios de App Service](https://azure.microsoft.com/pricing/details/app-service/windows/) y desplácese hasta el dominio de App Service.

### <a name="sign-in-to-azure"></a>Inicio de sesión en Azure
Abra [Azure Portal](https://portal.azure.com/) e inicie sesión con su cuenta de Azure.

### <a name="launch-buy-domains"></a>Iniciar Comprar dominios
En la pestaña **App Service**, haga clic en el nombre de la aplicación, seleccione **Configuración** y elija **Dominios personalizados**
   
![Captura de pantalla de la opción Dominios personalizados resaltada.](./media/custom-dns-web-site-buydomains-web-app/dncmntask-cname-6.png)

En la página **Dominios personalizados**, haga clic en **Comprar dominio**.

![Captura de pantalla de la opción Comprar dominio resaltada.](./media/custom-dns-web-site-buydomains-web-app/dncmntask-cname-buydomains-1.png)

> [!NOTE]
> Si no aparece la sección **Dominios de App Service**, debe quitar el límite de gasto de la cuenta de Azure (vea los [requisitos previos](#prerequisites)).
>
>

### <a name="configure-the-domain-purchase"></a>Configurar la compra del dominio

En la página **Dominio de App Service**, escriba el nombre de dominio que quiere comprar en el cuadro **Buscar dominio** y escriba `Enter`. Los dominios disponibles sugeridos se muestran justo debajo del cuadro de texto. Seleccione uno o varios dominios que quiera comprar.

![Captura de pantalla del cuadro de búsqueda Buscar dominio.](./media/custom-dns-web-site-buydomains-web-app/dncmntask-cname-buydomains-2.png)

> [!NOTE]
> Los siguientes [dominios de primer nivel](https://wikipedia.org/wiki/Top-level_domain) son compatibles con los dominios de App Service: _com_, _net_, _co.uk_, _org_, _nl_, _in_, _biz_, _org.uk_ y _co.in_.
>
>

Haga clic en **Información de contacto** y rellene el formulario de información de contacto del dominio. Cuando termine, haga clic en **Aceptar** para volver a la página Dominio de App Service.

Es importante que rellene todos los campos obligatorios con la mayor precisión posible. Los datos incorrectos para la información de contacto pueden producir un error al comprar los dominios.

A continuación, seleccione las opciones que quiera para el dominio. Para obtener explicaciones, vea la tabla siguiente:

| Configuración | Valor sugerido | Descripción |
|-|-|-|
|Protección de la privacidad | Habilitar | Utilice la "Protección de la privacidad", que se incluye _gratis_ en el precio de compra. Algunos dominios de primer nivel los administran registradores que no admiten la protección de la privacidad, y estos se indican en la página **Protección de la privacidad**. |
| Asignar nombres de host predeterminados | **www** y **\@** | Seleccione los enlaces de nombre de host deseados, si quiere. Una vez completada la operación de compra de dominio, puede acceder a la aplicación en los nombres de host seleccionados. Si la aplicación está detrás de [Azure Traffic Manager](https://azure.microsoft.com/services/traffic-manager/), no verá la opción para asignar el dominio raíz (@), ya que Traffic Manager no admite registros A. Puede realizar cambios en las asignaciones de nombre de host una vez completada la compra del dominio. |

### <a name="accept-terms-and-purchase"></a>Aceptar los términos y comprar

Haga clic en **Condiciones legales** para consultar las condiciones y los gastos y, después, haga clic en **Comprar**.

> [!NOTE]
> Los dominios de App Service usan GoDaddy para el registro de dominio y Azure DNS para hospedar los dominios. Además de la tarifa de registro del dominio, se aplican cargos de uso de DNS de Azure. Para obtener más información, vea [DNS de Azure Precios](https://azure.microsoft.com/pricing/details/dns/).
>
>

De nuevo en la hoja **Dominio de App Service**, haga clic en **Aceptar**. Mientras la operación está en curso, verá las notificaciones siguientes:

![Captura de pantalla que muestra el mensaje de validación en curso.](./media/custom-dns-web-site-buydomains-web-app/dncmntask-cname-buydomains-validate.png)

![Captura de pantalla que muestra la notificación de que la compra se realizó correctamente.](./media/custom-dns-web-site-buydomains-web-app/dncmntask-cname-buydomains-purchase-success.png)

### <a name="test-the-hostnames"></a>Probar los nombres de host

Si ha asignado nombres de host predeterminados a la aplicación, también verá una notificación de operación correcta para cada nombre de host seleccionado.

![Captura de pantalla que muestra la notificación de éxito para cada nombre de host seleccionado.](./media/custom-dns-web-site-buydomains-web-app/dncmntask-cname-buydomains-bind-success.png)

También verá los nombres de host seleccionados en la página **Dominios personalizados**, en la sección **Nombres de host personalizados**.

![Captura de pantalla de los nombres de host seleccionados en la sección Nombres de host personalizados de la página Dominios personalizados.](./media/custom-dns-web-site-buydomains-web-app/dncmntask-cname-buydomains-hostnames-added.png)

> [!NOTE]
> Si un dominio personalizado contiene la etiqueta **Not Secure**, significa que aún no está enlazado a un certificado TLS/SSL y que todas las solicitudes HTTPS de un explorador que lleguen a dicho dominio personalizado recibirán un error o una advertencia, en función del explorador que se use. Para configurar un enlace TLS, consulte [Protección de un nombre DNS personalizado con un enlace TLS/SSL en Azure App Service](configure-ssl-bindings.md).
>

Para probar los nombres de host, vaya a los nombres de host enumerados en el explorador. En el ejemplo de la captura de pantalla anterior, intente navegar a _kontoso.net_ y _www\.kontoso.net_.

## <a name="assign-hostnames-to-app"></a>Asignar nombres de host a la aplicación

Si decide no asignar uno o varios nombres de host predeterminados a la aplicación durante el proceso de compra, o si tiene que asignar un nombre de host que no aparece, puede asignar un nombre de host en cualquier momento.

También puede asignar nombres de host en el dominio de App Service a cualquier otra aplicación. Los pasos dependen de si el dominio de App Service y la aplicación pertenecen a la misma suscripción.

- Suscripción diferente: Asigne los registros DNS personalizados desde el dominio de App Service a la aplicación como un dominio comprado de forma externa. Para obtener información sobre cómo agregar nombres DNS personalizados a un dominio de App Service, vea [Administrar los registros DNS personalizados](#custom). Para asignar un dominio comprado externo a una aplicación, vea [Tutorial: Asignación de un nombre DNS personalizado existente a Azure App Service](app-service-web-tutorial-custom-domain.md). 
- Misma suscripción: Para ello, siga los pasos que se describen a continuación.

### <a name="launch-add-hostname"></a>Iniciar Agregar nombre de host
En la página **App Services**, seleccione el nombre de la aplicación a la que quiere asignar los nombres de host, seleccione **Configuración** y después **Dominios personalizados**.

![Captura de pantalla de la opción Dominios personalizados resaltada.](./media/custom-dns-web-site-buydomains-web-app/dncmntask-cname-6.png)

Asegúrese de que el dominio comprado aparece en la sección **Dominios de App Service**, pero no lo seleccione. 

![Captura de pantalla del dominio adquirido en la sección Dominios de App Service.](./media/custom-dns-web-site-buydomains-web-app/dncmntask-cname-buydomains-select-domain.png)

> [!NOTE]
> Todos los dominios de App Service de la misma suscripción se muestran en la página **Dominios personalizados** de la aplicación. Si el dominio está en la suscripción de la aplicación, pero no puede verlo en la página **Dominios personalizados** de la aplicación, intente volver a abrir la página **Dominios personalizados** o actualice la página web. Compruebe también la campana de notificación situada en la parte superior de Azure Portal para ver el progreso o los errores de creación.
>
>

Seleccione **Agregar nombre de host**.

### <a name="configure-hostname"></a>Configurar el nombre de host
En el cuadro de diálogo **Agregar nombre de host**, escriba el nombre de dominio completo de su dominio de App Service o cualquier subdominio. Por ejemplo:

- kontoso.net
- www\.kontoso.net
- abc.kontoso.net

Cuando termine, seleccione **Validar**. El tipo de registro de nombre de host se selecciona automáticamente.

Seleccione **Agregar nombre de host**.

Una vez completada la operación, verá una notificación de operación correcta para el nombre de host asignado.  

![Captura de pantalla de la notificación de éxito para el nombre de host asignado.](./media/custom-dns-web-site-buydomains-web-app/dncmntask-cname-buydomains-bind-success.png)

### <a name="close-add-hostname"></a>Cerrar Agregar nombre de host
En la página **Agregar nombre de host**, asigne otros nombres de host a la aplicación, según sea necesario. Cuando termine, cierre la página **Agregar nombre de host**.

Ahora debería ver el nombre de host recién asignado en la página **Dominios personalizados** de la aplicación.

![Captura de pantalla de los nombres de host recién asignados en la página Dominios personalizados de la aplicación.](./media/custom-dns-web-site-buydomains-web-app/dncmntask-cname-buydomains-hostnames-added2.png)

### <a name="test-the-hostnames"></a>Probar los nombres de host

Vaya a los nombres de host enumerados en el explorador. En el ejemplo de la captura de pantalla anterior, intente navegar a _abc.kontoso.net_.

## <a name="renew-the-domain"></a>Renovación del dominio

El dominio de App Service que compró es válido durante un año a partir del momento de la compra. De forma predeterminada, el dominio está configurado para renovarse automáticamente, y el cargo se realizará el próximo año según el método de pago especificado. También puede renovar manualmente el nombre de dominio.

Si desea desactivar la renovación automática, o si desea renovar manualmente su dominio, siga estos pasos.

En la pestaña **App Service**, haga clic en el nombre de la aplicación, seleccione **Configuración** y elija **Dominios personalizados**.

![Captura de pantalla de la opción Dominios personalizados resaltada.](./media/custom-dns-web-site-buydomains-web-app/dncmntask-cname-6.png)

En la sección **Dominios de App Service**, seleccione el dominio que desea configurar.

![Captura de pantalla del dominio adquirido en la sección Dominios de App Service.](./media/custom-dns-web-site-buydomains-web-app/dncmntask-cname-buydomains-select-domain.png)

En el panel de navegación izquierdo del dominio, seleccione **Renovación de dominios**. Para detener la renovación automática del dominio, seleccione **Desactivar** y, después, **Guardar**.

![Captura de pantalla de la opción de renovación automática del dominio.](./media/custom-dns-web-site-buydomains-web-app/dncmntask-cname-buydomains-autorenew.png)

Para renovar el dominio manualmente, seleccione **Renovar dominio**. Sin embargo, este botón no está activo hasta [noventa días antes de la expiración del dominio](#when-domain-expires).

Si la renovación del dominio se realiza correctamente, recibirá una notificación por correo electrónico dentro de un plazo de 24 horas.

## <a name="when-domain-expires"></a>Cuando expira el dominio

Azure procesa los dominios de App Service expirados o que están por expirar del modo siguiente:

* Si la renovación automática está deshabilitada: 90 días antes de la expiración del dominio, se le envía un correo electrónico de notificación de renovación y el botón **Renovar dominio** se activa en el portal.
* Si la renovación automática está habilitada: el día después de la fecha de expiración del dominio, Azure intenta cobrarle por la renovación del nombre de dominio.
* Si se produce un error durante la renovación automática (por ejemplo, expiró la tarjeta en el archivo) o si la renovación automática está deshabilitada y permite que el dominio expire, Azure le avisa de la expiración del dominio y aparca el nombre de dominio. También puede [renovar manualmente](#renew-the-domain) su dominio.
* 4 y 12 días después de la expiración, Azure envía correos electrónicos de notificación adicionales. También puede [renovar manualmente](#renew-the-domain) su dominio.
* 19 días después de la expiración, el dominio permanece en espera, pero queda sujeta a un precio de canje. Puede llamar al soporte al cliente para renovar su nombre de dominio, sujeto a cuotas de canje y renovación aplicables.
* 25 días después de la expiración, Azure pone al dominio en subasta con un servicio de subasta de nombres de dominio del sector. Puede llamar al soporte al cliente para renovar su nombre de dominio, sujeto a cuotas de canje y renovación aplicables.
* 30 días después de la expiración, ya no podrá canjear su dominio.

<a name="custom"></a>

## <a name="manage-custom-dns-records"></a>Administrar los registros DNS personalizados

En Azure, los registros DNS para un dominio de App Service se administran mediante [DNS de Azure](https://azure.microsoft.com/services/dns/). Puede agregar, quitar y actualizar los registros DNS, al igual que para un dominio comprado de forma externa.

### <a name="open-app-service-domain"></a>Abrir Dominio de App Service

En Azure Portal, en el menú de la izquierda, seleccione **Todos los servicios** > **Dominios de App Service**.

![Captura de pantalla que muestra dónde acceder a los Dominios de App Service.](./media/custom-dns-web-site-buydomains-web-app/dncmntask-cname-buydomains-access.png)

Seleccione el dominio que se va a administrar. 

### <a name="access-dns-zone"></a>Obtener acceso a la zona DNS

En el menú de la izquierda del dominio, seleccione **Zona DNS**.

![Captura de pantalla que muestra dónde seleccionar la zona DNS.](./media/custom-dns-web-site-buydomains-web-app/dncmntask-cname-buydomains-dns-zone.png)

Esta acción abre la página [Zona DNS](../dns/dns-zones-records.md) de su dominio de App Service en DNS de Azure. Para obtener información sobre cómo editar los registros DNS, vea [Administración de zonas DNS en Azure Portal](../dns/dns-operations-dnszones-portal.md).

## <a name="cancel-purchase-delete-domain"></a>Cancelar la compra (eliminar el dominio)

Después de comprar el dominio de App Service, dispone de cinco días para cancelar la compra para obtener un reembolso completo. Después de cinco días, puede eliminar el dominio de App Service, pero no puede recibir un reembolso.

### <a name="open-app-service-domain"></a>Abrir Dominio de App Service

En Azure Portal, en el menú de la izquierda, seleccione **Todos los servicios** > **Dominios de App Service**.

![Captura de pantalla que muestra dónde acceder a los Dominios de App Service.](./media/custom-dns-web-site-buydomains-web-app/dncmntask-cname-buydomains-access.png)

Seleccione el dominio que quiera cancelar o eliminar. 

### <a name="delete-hostname-bindings"></a>Eliminación de enlaces de nombre de host

En el menú de la izquierda del dominio, seleccione **Enlaces de nombre de host**. Aquí se enumeran los enlaces de nombre de host de todos los servicios de Azure.

![Captura de pantalla de la página Enlaces de nombre de host.](./media/custom-dns-web-site-buydomains-web-app/dncmntask-cname-buydomains-hostname-bindings.png)

No se puede eliminar el dominio de App Service hasta que se eliminen todos los enlaces de nombre de host.

Para eliminar todos los enlaces de nombre de host, seleccione **...**  > **Eliminar**. Después de eliminarlos todos, haga clic en **Guardar**.

![Captura de pantalla que muestra dónde se deben eliminar los enlaces de nombre de host.](./media/custom-dns-web-site-buydomains-web-app/dncmntask-cname-buydomains-delete-hostname-bindings.png)

### <a name="cancel-or-delete"></a>Cancelar o eliminar

En el menú de la izquierda del dominio, seleccione **Información general**. 

Si no ha transcurrido el período de cancelación en el dominio comprado, haga clic en **Cancelar compra**. De lo contrario, en su lugar verá el botón **Eliminar**. Para eliminar el dominio sin un reembolso, haga clic en **Eliminar**.

![Captura de pantalla que muestra dónde se debe eliminar o cancelar un dominio adquirido.](./media/custom-dns-web-site-buydomains-web-app/dncmntask-cname-buydomains-cancel.png)

Para confirmar la operación, seleccione **Sí**.

Una vez completada la operación, el dominio se libera de la suscripción y vuelve a estar disponible para que otros usuarios lo compren. 

## <a name="direct-default-url-to-a-custom-directory"></a>Direccionamiento de una dirección URL predeterminada a un directorio personalizado

De forma predeterminada, App Service dirige las solicitudes web al directorio raíz del código de la aplicación. Para dirigirlas a un subdirectorio, como `public`, consulte [Direccionamiento de una dirección URL predeterminada a un directorio personalizado](app-service-web-tutorial-custom-domain.md#virtualdir).