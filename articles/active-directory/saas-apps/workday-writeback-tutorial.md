---
title: 'Tutorial: Configuración de la escritura diferida de Workday en Azure Active Directory | Microsoft Docs'
description: Aprenda a configurar la escritura diferida de atributos desde Azure AD en Workday.
services: active-directory
author: cmmdesai
manager: CelesteDG
ms.service: active-directory
ms.subservice: saas-app-tutorial
ms.topic: article
ms.workload: identity
ms.date: 05/26/2020
ms.author: chmutali
ms.openlocfilehash: 324d3a21e1694d243f03beca28ac8376bedffa4d
ms.sourcegitcommit: 023d10b4127f50f301995d44f2b4499cbcffb8fc
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/18/2020
ms.locfileid: "88526837"
---
# <a name="tutorial-configure-attribute-writeback-from-azure-ad-to-workday"></a>Tutorial: Configuración de la escritura diferida de atributos de Azure AD en Workday
El objetivo de este tutorial es mostrar los pasos que debe realizar para la escritura diferida de atributos de Azure AD en Workday. La aplicación de aprovisionamiento de escritura diferida de Workday admite la asignación de valores a los siguientes atributos de Workday:
* Correo electrónico del trabajo 
* Nombre de usuario de Workday
* Número de teléfono fijo del trabajo (incluidos el código de país, el código de área, el número y la extensión)
* Marca principal del número de teléfono fijo del trabajo
* Número móvil del trabajo (incluidos el código de país, el código de área y el número)
* Marca principal del móvil del trabajo

## <a name="overview"></a>Información general

Después de configurar la integración del aprovisionamiento de entrada mediante la aplicación de aprovisionamiento de [Workday a AD local](workday-inbound-tutorial.md) o la aplicación de aprovisionamiento de [Workday a Azure AD](workday-inbound-cloud-only-tutorial.md), puede configurar opcionalmente la aplicación de escritura diferida Workday para que escriba información de contacto como la dirección de correo electrónico y el número de teléfono en Workday. 

### <a name="who-is-this-user-provisioning-solution-best-suited-for"></a>¿Para quién es más adecuada esta solución de aprovisionamiento de usuarios?

Esta solución de aprovisionamiento de usuarios de escritura diferida de Workday resulta adecuada para:

* Organizaciones que usan Office 365 y que desean realizar la escritura diferida de atributos autoritativos administrados por TI (como la dirección de correo electrónico, el nombre de usuario y el número de teléfono) en Workday

## <a name="configure-integration-system-user-in-workday"></a>Configuración de un usuario de integración en Workday

Consulte la sección [Configuración de un usuario del sistema de integración](workday-inbound-tutorial.md#configure-integration-system-user-in-workday) para crear una cuenta de usuario del sistema de integración de Workday con permisos para recuperar datos de trabajo. 

## <a name="configuring-azure-ad-attribute-writeback-to-workday"></a>Configuración de la escritura diferida de atributos de Azure AD en Workday

Siga estas instrucciones para configurar la escritura diferida de direcciones de correo electrónico y nombres de usuario de Azure Active Directory a Workday.

* [Adición de la aplicación del conector de escritura diferida y creación de la conexión con Workday](#part-1-adding-the-writeback-connector-app-and-creating-the-connection-to-workday)
* [Configuración de las asignaciones de atributos de escritura diferida](#part-2-configure-writeback-attribute-mappings)
* [Habilitar e iniciar el aprovisionamiento de usuarios](#enable-and-launch-user-provisioning)

### <a name="part-1-adding-the-writeback-connector-app-and-creating-the-connection-to-workday"></a>Parte 1: Adición de la aplicación del conector de escritura diferida y creación de la conexión con Workday

**Para configurar el conector de escritura diferida de Workday:**

1. Ir a <https://portal.azure.com>.

2. En Azure Portal, busque y seleccione **Azure Active Directory**.

3. Seleccione **Enterprise Applications** (Aplicaciones empresariales) y **All Applications** (Todas las aplicaciones).

4. Seleccione **Agregar una aplicación** y, luego, seleccione la categoría **Todos**.

5. Busque **Workday Writeback** (Reescritura de Workday) y agregue esa aplicación desde la galería.

6. Una vez que se haya agregado la aplicación y se muestre la pantalla de detalles de la aplicación, seleccione **Aprovisionamiento**.

7. Cambie el **Modo** de **aprovisionamiento** a **Automático**.

8. Cumplimente la sección **Credenciales de administrador** del siguiente modo:

   * **Nombre de usuario de administrador**: escriba el nombre de usuario de la cuenta del sistema de integración de Workday, anexando el nombre de dominio del inquilino. Debe tener un aspecto similar al siguiente: *username\@contoso4*

   * **Contraseña de administrador**: escriba la contraseña de la cuenta del sistema de integración de Workday.

   * **URL de inquilino**: escriba la dirección URL al punto de conexión de servicios web de Workday de su inquilino. Este valor debería ser similar a `https://wd3-impl-services1.workday.com/ccx/service/contoso4/Human_Resources`, donde *contoso4* se reemplaza por el nombre correcto del inquilino y *wd3-impl* se reemplaza por la cadena de entorno correcta (si es necesario).

   * **Correo electrónico de notificación**: escriba su dirección de correo electrónico y marque la casilla "Send email if failure occurs" (Enviar una notificación por correo electrónico cuando se produzca un error).

   * Haga clic en el botón **Probar conexión**. Si la prueba de conexión se lleva a cabo correctamente, haga clic en el botón **Guardar** situado en la parte superior. Si se produce un error, compruebe que la dirección URL y las credenciales de Workday sean válidas en Workday.

### <a name="part-2-configure-writeback-attribute-mappings"></a>Parte 2: Configuración de las asignaciones de atributos de escritura diferida

En esta sección, va a configurar cómo los atributos de escritura diferida fluyen desde Azure AD a Workday. 

1. En la pestaña Aprovisionamiento de **Asignaciones**, haga clic en el nombre de asignación.

2. En el campo **Ámbito de objeto de origen** tiene la opción de filtrar los conjuntos de usuarios de Azure Active Directory que deben formar parte de la escritura diferida. El ámbito predeterminado es "todos los usuarios de Azure AD".

3. En la sección **Asignaciones de atributos**, actualice el identificador coincidente para indicar el atributo en Azure Active Directory donde se almacena el identificador de trabajo o el identificador de empleado de Workday. Un método de búsqueda de coincidencias conocido consiste en sincronizar el Id. de trabajador de Workday o el Id. de empleado con el atributo extensionAttribute1-15 en Azure AD y, luego, usar dicho atributo en Azure AD para hacer coincidir los usuarios en Workday.

4. Normalmente, se asigna el atributo *userPrincipalName* de Azure AD al atributo *UserID* de Workday y el atributo *mail* de Azure AD al atributo *EmailAddress* de Workday. 

     >[!div class="mx-imgBorder"]
     >![Azure Portal](./media/workday-inbound-tutorial/workday-writeback-mapping.png)

5. Use las instrucciones compartidas a continuación para asignar valores de atributo de número de teléfono de Azure AD a Workday. 

     | Atributo de teléfono de Workday | Valor esperado | Instrucciones de asignación |
     |-------------------------|----------------|------------------|
     | WorkphoneLandlineIsPrimary | true/false | Asignación de constantes o expresiones cuya salida es un valor de cadena "true" o "false". |
     | WorkphoneLandlineCountryCodeName | [Código de país ISO 3166-1 de tres letras](https://en.wikipedia.org/wiki/ISO_3166-1_alpha-3) | Asignación de constantes o expresiones cuya salida es un código de país de tres letras. |
     | WorkphoneLandlineCountryCodeNumber | [Código internacional de llamada del país](https://en.wikipedia.org/wiki/List_of_country_calling_codes) | Asignación de constantes o expresiones cuya salida es un código de país válido (sin el signo +). |
     | WorkphoneLandlineNumber | Número de teléfono completo que incluye el código de área | Asígnelo al atributo *telephoneNumber*. Use una expresión regular para quitar los espacios en blanco, los corchetes y el código de país. Consulte el ejemplo siguiente. |
     | WorkphoneLandlineExtension | Número de extensión | Si *telephoneNumber* contiene la extensión, utilice la expresión regular para extraer el valor. |
     | WorkphoneMobileIsPrimary | true/false | Asignación de constantes o expresiones cuya salida es un valor de cadena "true" o "false". |
     | WorkphoneMobileCountryCodeName | [Código de país ISO 3166-1 de tres letras](https://en.wikipedia.org/wiki/ISO_3166-1_alpha-3) | Asignación de constantes o expresiones cuya salida es un código de país de tres letras. |
     | WorkphoneMobileCountryCodeNumber | [Código internacional de llamada del país](https://en.wikipedia.org/wiki/List_of_country_calling_codes) | Asignación de constantes o expresiones cuya salida es un código de país válido (sin el signo +). |
     | WorkphoneMobileNumber | Número de teléfono completo que incluye el código de área | Asígnelo al atributo *mobile*. Use una expresión regular para quitar los espacios en blanco, los corchetes y el código de país. Consulte el ejemplo siguiente. |

     > [!NOTE]
     > Al invocar el servicio web de Workday Change_Work_Contact, Azure AD envía los valores constantes siguientes: <br>
     > * **Communication_Usage_Type_ID** se establece en la cadena constante "WORK" <br>
     > * **Phone_Device_Type_ID** se establece en la cadena constante "Mobile" para los números de teléfono móvil y "Landline" para los números de teléfono fijo. <br>
     > 
     > Se producirán errores de escritura diferida si el inquilino de Workday usa distintas funciones Type_ID. Para evitar dichos errores, puede usar la tarea de **mantenimiento de identificadores de referencia** de Workday y actualizar las funciones Type_ID para que coincidan con los valores usados por Azure AD. <br>
     >  

     **Expresiones regex de referencia: ejemplo 1**

     Use la siguiente expresión regular, si el número de teléfono de Azure AD se establece con el formato necesario para el autoservicio de restablecimiento de contraseña (SSPR). <br>
     Ejemplo: si el valor del número de teléfono es +1 1112223333 -> la expresión regex dará como resultado 1112223333

     ```C#
     Replace([telephoneNumber], , "\\+(?<isdCode>\\d* )(?<phoneNumber>\\d{10})", , "${phoneNumber}", , )
     ```

     **Expresiones regex de referencia: ejemplo 2**

     Use la siguiente expresión regular, si el número de teléfono de Azure AD se establece con el formato (XXX) XXX-XXXX. <br>
     Ejemplo: si el valor del número de teléfono es (111) 222-3333 -> la expresión regex dará como resultado 1112223333

     ```C#
     Replace([mobile], , "[()\\s-]+", , "", , )
     ```

6. Para guardar las asignaciones, haga clic en **Guardar** en la parte superior de la sección Asignación de atributos.

## <a name="enable-and-launch-user-provisioning"></a>Habilitar e iniciar el aprovisionamiento de usuarios

Una vez completadas las configuraciones de la aplicación de aprovisionamiento de Workday, puede activar el servicio de aprovisionamiento en Azure Portal.

> [!TIP]
> De forma predeterminada, al activar el servicio de aprovisionamiento, se iniciarán las operaciones de aprovisionamiento para todos los usuarios del ámbito. Si hay errores en la asignación o problemas con los datos de Workday, puede que se produzcan errores con el trabajo de aprovisionamiento y que entre en estado de cuarentena. Para evitar esto, como procedimiento recomendado, es conveniente configurar el filtro **Ámbito de objeto de origen** y probar las asignaciones de atributos con algunos usuarios de prueba antes de iniciar la sincronización completa de todos los usuarios. Una vez haya verificado que las asignaciones funcionan y que obtiene los resultados deseados, puede quitar el filtro o expandirlo gradualmente para incluir más usuarios.

1. En la pestaña **Aprovisionamiento**, establezca **Estado de aprovisionamiento** en **Activado**.

2. Haga clic en **Save**(Guardar).

3. Esta operación iniciará la sincronización inicial, que puede tardar una cantidad de horas variable, según el número de usuarios que haya en el directorio de origen. Puede consultar en la barra de progreso el seguimiento del progreso del ciclo de sincronización. 

4. En cualquier momento, compruebe la pestaña **Registros de auditoría** en Azure Portal para ver las acciones que ha realizado el servicio de aprovisionamiento. En los registros de auditoría se enumeran todos los eventos de sincronización individuales realizados por el servicio de aprovisionamiento, como, por ejemplo, qué usuarios se importan desde el origen y se exportan a la aplicación de destino.  

5. Una vez completada la sincronización inicial, se escribe un informe resumido en la pestaña **Aprovisionamiento**, tal y como se muestra a continuación.

     > [!div class="mx-imgBorder"]
     > ![Barra de progreso de aprovisionamiento](./media/sap-successfactors-inbound-provisioning/prov-progress-bar-stats.png)

## <a name="next-steps"></a>Pasos siguientes

* [Aprenda a revisar los registros y a obtener informes sobre la actividad de aprovisionamiento](../app-provisioning/check-status-user-account-provisioning.md)
* [Aprenda a configurar el inicio de sesión único entre Azure Active Directory y Workday](workday-tutorial.md)
* [Aprenda a integrar otras aplicaciones SaaS con Azure Active Directory](tutorial-list.md)
* [Aprenda a exportar e importar las configuraciones de aprovisionamiento](../app-provisioning/export-import-provisioning-configuration.md)

