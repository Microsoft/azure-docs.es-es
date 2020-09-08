---
title: 'Azure AD Connect: errores LargeObject causados por el atributo userCertificate | Microsoft Docs'
description: En este tema se proporcionan los pasos de corrección de errores LargeObject causados por atributo userCertificate.
services: active-directory
documentationcenter: ''
author: billmath
manager: daveba
editor: ''
ms.assetid: 146ad5b3-74d9-4a83-b9e8-0973a19828d9
ms.service: active-directory
ms.workload: identity
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: troubleshooting
ms.date: 07/13/2017
ms.subservice: hybrid
ms.author: billmath
ms.custom: seohack1
ms.collection: M365-identity-device-management
ms.openlocfilehash: 2eb656e46ce5e26fca5ae5c094f9b8bb85819caa
ms.sourcegitcommit: c94a177b11a850ab30f406edb233de6923ca742a
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/01/2020
ms.locfileid: "89275783"
---
# <a name="azure-ad-connect-sync-handling-largeobject-errors-caused-by-usercertificate-attribute"></a>Sincronización de Azure AD Connect: Control de errores LargeObject causados por el atributo userCertificate

Azure AD exige un límite máximo de **15** valores de certificado en el atributo **userCertificate**. Si Azure AD Connect exporta un objeto con más de 15 valores a Azure AD, este devuelve un error **LargeObject** con un mensaje que indica

>*que el objeto aprovisionado es demasiado grande y se debe recordar el número de valores de atributo en este objeto. La operación se volverá a intentar en el siguiente ciclo de sincronización.*

El error LargeObject puede deberse a otros atributos de AD. Para confirmar que realmente está causado por el atributo userCertificate, debe comprobarlo en el objeto ya sea en AD local o en la [búsqueda de metaverso de Synchronization Service Manager](./how-to-connect-sync-service-manager-ui-mvsearch.md).

Para obtener la lista de objetos en el inquilino con errores LargeObject, siga uno de los métodos siguientes:

 * Si está habilitado el inquilino para Azure AD Connect Health para sincronización, puede hacer referencia al [informe de errores de sincronización](./how-to-connect-health-sync.md) proporcionado.
 
 * El correo electrónico de notificación de los errores de sincronización de directorios que se envía al final de cada ciclo de sincronización tiene la lista de objetos con errores LargeObject. 
 * La pestaña [Operaciones de Synchronization Service Manager](./how-to-connect-sync-service-manager-ui-operations.md) muestra la lista de objetos con errores LargeObject si hace clic en la exportación más reciente a la operación de Azure AD.
 
## <a name="mitigation-options"></a>Opciones de mitigación
Hasta que se resuelva el error LargeObject, no se pueden exportar a Azure AD otros cambios de atributo en el mismo objeto. Para resolver el error, puede considerar las siguientes opciones:

 * Actualice a la compilación 1.1.524.0 o posterior de Azure AD Connect. En la compilación 1.1.524.0 de Azure AD Connect, las reglas de sincronización originales se han actualizado para no exportar los atributos userSMIMECertificate y userCertificate, si estos tienen más de 15 valores. Para más información acerca cómo actualizar Azure AD Connect, consulte el artículo [Azure AD Connect: actualización de una versión anterior a la versión más reciente](./how-to-upgrade-previous-version.md)

 * Implemente una **regla de sincronización de salida** en Azure AD Connect que exporte un **valor null en lugar de los valores reales para objetos con más de 15 valores de certificado**. Esta opción es adecuada si no necesita que ninguno de los valores de certificado se exporte a Azure AD para objetos con más de 15 valores. Para más información sobre cómo implementar esta regla de sincronización, consulte la sección siguiente [Implementación de la regla de sincronización para limitar la exportación del atributo userCertificate](#implementing-sync-rule-to-limit-export-of-usercertificate-attribute).

 * Reduzca el número de valores de certificado en el objeto de AD local (15 o menos) mediante la eliminación de los valores que ya no están en uso por su organización. Esto es adecuado si el sobredimensionamiento del atributo está causado por los certificados expirados o sin usar. Puede usar [el script de PowerShell disponible aquí](https://gallery.technet.microsoft.com/Remove-Expired-Certificates-0517e34f) para buscar, hacer una copia de seguridad y eliminar certificados expirados en el AD local. Antes de eliminar los certificados, se recomienda que los compruebe con los administradores de infraestructura de clave pública de su organización.

 * Configure Azure AD Connect para que el atributo userCertificate no se exporte a Azure AD. En general, no se recomienda esta opción ya que el atributo lo puede usar Microsoft Online Services para habilitar escenarios concretos. En concreto:
    * El atributo userCertificate del objeto de usuario lo usan los clientes de Outlook y Exchange Online para el cifrado y la firma de mensajes. Para más información sobre esta característica, consulte el artículo [S/MIME para la firma y el cifrado de mensajes](/microsoft-365/security/office-365-security/s-mime-for-message-signing-and-encryption?view=o365-worldwide).

    * El atributo userCertificate del objeto Computer lo utiliza Azure AD para permitir que dispositivos unidos a un dominio de Windows 10 local se conecten a Azure AD. Para más información sobre esta característica, consulte el artículo [Experiencias de conexión de dispositivos unidos a un dominio a Azure AD para Windows 10](../devices/hybrid-azuread-join-plan.md).

## <a name="implementing-sync-rule-to-limit-export-of-usercertificate-attribute"></a>Implementación de la regla de sincronización para limitar la exportación del atributo userCertificate
Para resolver el error LargeObject causado por el atributo userCertificate, puede implementar una regla de sincronización de salida en Azure AD Connect que exporte un **valor null en lugar de los valores reales para objetos con más de 15 valores de certificado**. En esta sección se describen los pasos necesarios para implementar la regla de sincronización para objetos **User**. Los pasos se pueden adaptados para los objetos **Contact** y **Computer**.

> [!IMPORTANT]
> La exportación de un valor null quita los valores de certificado que se exportaron correctamente a Azure AD.

Los pasos se pueden resumir como:

1. Deshabilitar el programador de sincronización y comprobar que no hay ninguna sincronización en curso.
3. Buscar la regla de sincronización de salida existente para el atributo userCertificate.
4. Crear la regla de sincronización de salida requerida.
5. Comprobar la nueva regla de sincronización en un objeto existente con el error LargeObject.
6. Aplicar la nueva regla de sincronización a los objetos restantes con el error LargeObject.
7. Comprobar que no hay ningún cambio inesperado en espera de su exportación a Azure AD.
8. Exportar los cambios a Azure AD.
9. Volver a habilitar el programador de sincronización.

### <a name="step-1-disable-sync-scheduler-and-verify-there-is-no-synchronization-in-progress"></a>Paso 1. Deshabilitar el programador de sincronización y comprobar que no hay ninguna sincronización en curso
Asegúrese de que no realiza ninguna sincronización mientras está en medio de implementar una nueva regla de sincronización para evitar que cambios no deseados se exporten a Azure AD. Para deshabilitar el programador de sincronización integrado:
1. Inicie una sesión de PowerShell en el servidor de Azure AD Connect.

2. Deshabilite la sincronización programada mediante la ejecución del cmdlet: `Set-ADSyncScheduler -SyncCycleEnabled $false`

> [!Note]
> Los pasos anteriores solo se aplican a las versiones más recientes (1.1.xxx.x) de Azure AD Connect con el programador integrado. Si usa versiones anteriores (1.0.xxx.x) de Azure AD Connect que utilizan el Programador de tareas de Windows, o que usa a su propio programador personalizado (no común) para desencadenar una sincronización periódica, debe deshabilitarlas según corresponda.

1. Inicie **Synchronization Service Manager**. Para ello, vaya a INICIO → Synchronization Service (Servicio de sincronización).

1. Vaya a la pestaña **Operations** (Operaciones) y confirme que no hay ninguna operación cuyo estado sea *"in progress"* (En curso).

### <a name="step-2-find-the-existing-outbound-sync-rule-for-usercertificate-attribute"></a>Paso 2. Buscar la regla de sincronización de salida existente para el atributo userCertificate
Debería haber una regla de sincronización existente que esté habilitada y configurada para exportar el atributo userCertificate para objetos User a Azure AD. Busque esta regla de sincronización para averiguar la configuración de **precedencia** y **filtro de ámbito**:

1. Inicie el **editor de reglas de sincronización**. Para ello, vaya a INICIO → Synchronization Rules Editor (Servicio de sincronización).

2. Configure los filtros de búsqueda con los siguientes valores:

    | Atributo | Value |
    | --- | --- |
    | Dirección |**Outbound** |
    | Tipo de objeto del metaverso |**Person** |
    | Conector |*nombre de su conector de Azure AD* |
    | Tipo de objeto de conector |**user** |
    | Atributos del metaverso |**userCertificate** |

3. Si está usando las reglas de sincronización integradas para el conector de Azure AD con el fin de exportar el atributo userCertificate para objetos User, debe volver a la regla *"Out to AAD – User ExchangeOnline"* (Fuera de AAD - User ExchangeOnline).
4. Anote el valor de **precedencia** de esta regla de sincronización.
5. Seleccione la regla de sincronización y haga clic en **Editar**.
6. En el cuadro de diálogo emergente *"Edit Reserved Rule Confirmation"* (Editar confirmación de regla reservada), haga clic en **No**. (No se preocupe, no vamos a realizar ningún cambio en esta regla de sincronización).
7. En la pantalla de edición, seleccione la pestaña **Scoping filter** (Filtro de ámbito).
8. Anote la configuración del filtro de ámbito. Si se usa la regla de sincronización integrada, debe haber exactamente **un grupo de filtro de ámbito que contiene dos cláusulas**, incluido:

    | Atributo | Operator | Value |
    | --- | --- | --- |
    | sourceObjectType | EQUAL | Usuario |
    | cloudMastered | NOTEQUAL | True |

### <a name="step-3-create-the-outbound-sync-rule-required"></a>Paso 3. Crear la regla de sincronización de salida requerida
La nueva regla de sincronización debe tener el mismo **filtro de ámbito** y una **mayor precedencia** que la regla de sincronización existente. Esto asegura que la nueva regla de sincronización se aplica al mismo conjunto de objetos que la regla de sincronización existente y reemplaza la regla de sincronización existente por el atributo userCertificate. Para crear la regla de sincronización:
1. En el Editor de reglas de sincronización, haga clic en el botón **Agregar nueva regla**.
2. En la pestaña **Descripción**, proporcione la siguiente configuración:

    | Atributo | Value | Detalles |
    | --- | --- | --- |
    | Nombre | *Proporcione un nombre*. | Por ejemplo, *"Fuera de AAD – Reemplazo personalizado para userCertificate"* |
    | Descripción | *Proporcione una descripción* | Por ejemplo, *"Si el atributo userCertificate tiene más de 15 valores, la exportación es NULL".* |
    | Sistema conectado | *Seleccione Azure AD Connector* |
    | Tipo de objeto de sistema conectado | **user** | |
    | Propiedades de objeto de metaverso | **person** | |
    | Tipo de vínculo | **Join** | |
    | Prioridad | *Elija un número entre 1 y 99* | El número elegido no deben usarse por ninguna regla de sincronización existente y tiene un valor inferior (y por lo tanto, una mayor precedencia) que la regla de sincronización existente. |

3. Vaya a la pestaña **Scoping filter** (Filtro de ámbito) e implemente el mismo filtro de ámbito que está usando la regla de sincronización existente.
4. Omita pestaña **Join rules** (Reglas de unión).
5. Vaya a la pestaña **Transformations** (Transformaciones) para agregar una nueva transformación mediante la configuración siguiente:

    | Atributo | Value |
    | --- | --- |
    | Tipo de flujo |**Expression** |
    | Atributo de destino |**userCertificate** |
    | Atributo de origen |*Utilice la expresión siguiente*:`IIF(IsNullOrEmpty([userCertificate]), NULL, IIF((Count([userCertificate])> 15),AuthoritativeNull,[userCertificate]))` |
    
6. Haga clic en el botón **Agregar** para crear la regla de sincronización.

### <a name="step-4-verify-the-new-sync-rule-on-an-existing-object-with-largeobject-error"></a>Paso 4. Comprobar la nueva regla de sincronización en un objeto existente con el error LargeObject
Esto sirve para comprobar que la regla de sincronización creada funciona correctamente en un objeto de AD existente con el error LargeObject antes de aplicarla a otros objetos:
1. Vaya a la pestaña **Operations** (Operaciones) de Synchronization Service Manager.
2. Seleccione la exportación más reciente de la operación de Azure AD y haga clic en uno de los objetos con errores LargeObject.
3.  En la pantalla emergente Connector Space Object Properties (Propiedades de objeto del espacio de conector), haga clic en el botón **Preview** (Vista previa).
4. En la pantalla emergente Preview (Vista previa), seleccione **Full synchronization** (Sincronización completa) y haga clic en **Commit Preview** (Confirmar vista previa).
5. Cierre la pantalla de vista previa y la pantalla de propiedades del objeto de espacio de conector.
6. Vaya a la pestaña **Connectors** (Conectores) de Synchronization Service Manager.
7. Haga clic con el botón derecho en el conector **Azure AD** y seleccione **Ejecutar...**
8. En el menú emergente Run Connector (Ejecutar conector), seleccione el paso **Exportar** (Exportar) y haga clic en **OK** (Aceptar).
9. Espere a que la exportación a Azure AD finalice y confirme que no hay ningún error LargeObject más en este objeto específico.

### <a name="step-5-apply-the-new-sync-rule-to-remaining-objects-with-largeobject-error"></a>Paso 5. Aplicar la nueva regla de sincronización a los objetos restantes con el error LargeObject
Una vez que se ha agregado la regla de sincronización, debe ejecutar un paso de sincronización completa en el conector de AD:
1. Vaya a la pestaña **Connectors** (Conectores) de Synchronization Service Manager.
2. Haga clic con el botón derecho en el conector **AD** y seleccione **Run...** (Ejecutar).
3. En el menú emergente Run Connector (Ejecutar conector), seleccione el paso **Full Synchronization** (Sincronización completa) y haga clic en **OK** (Aceptar).
4. Espere a que el paso de sincronización completa se complete.
5. Repita los pasos anteriores para los conectores AD restantes si tiene más de uno. Normalmente, se necesitan varios conectores si tiene varios directorios locales.

### <a name="step-6-verify-there-are-no-unexpected-changes-waiting-to-be-exported-to-azure-ad"></a>Paso 6. Comprobar que no hay ningún cambio inesperado en espera de su exportación a Azure AD
1. Vaya a la pestaña **Connectors** (Conectores) de Synchronization Service Manager.
2. Haga clic con el botón derecho en el conector **Azure AD** y seleccione **Search Connector Space** (Espacio del conector de búsqueda).
3. En el elemento emergente Search Connector Space (Espacio del conector de búsqueda):
    1. Establezca el ámbito en **Pending Export** (Exportación pendiente).
    2. Active las tres casillas, incluidas **Add** (Agregar), **Modify** (Modificar) y **Delete** (Eliminar).
    3. Haga clic en el botón **Search** (Buscar) para devolver todos los objetos con cambios que esperan su exportación a Azure AD.
    4. Compruebe que no hay cambios inesperados. Para examinar los cambios de un determinado objeto, haga doble clic en el objeto.

### <a name="step-7-export-the-changes-to-azure-ad"></a>Paso 7. Exportar los cambios a Azure AD
Para exportar los cambios a Azure AD:
1. Vaya a la pestaña **Connectors** (Conectores) de Synchronization Service Manager.
2. Haga clic con el botón derecho en el conector **Azure AD** y seleccione **Ejecutar...**
4. En el menú emergente Run Connector (Ejecutar conector), seleccione el paso **Exportar** (Exportar) y haga clic en **OK** (Aceptar).
5. Espere a que la exportación a Azure AD finalice y confirme que no hay ningún error LargeObject más.

### <a name="step-8-re-enable-sync-scheduler"></a>Paso 8. Volver a habilitar el programador de sincronización
Ahora que el problema se ha resuelto, vuelva a habilitar el programador de sincronización integrado:
1. Inicie la sesión de PowerShell.
2. Vuelva a habilitar la sincronización programada mediante la ejecución del cmdlet: `Set-ADSyncScheduler -SyncCycleEnabled $true`

> [!Note]
> Los pasos anteriores solo se aplican a las versiones más recientes (1.1.xxx.x) de Azure AD Connect con el programador integrado. Si usa versiones anteriores (1.0.xxx.x) de Azure AD Connect que utilizan el Programador de tareas de Windows, o que usa a su propio programador personalizado (no común) para desencadenar una sincronización periódica, debe deshabilitarlas según corresponda.

## <a name="next-steps"></a>Pasos siguientes
Obtenga más información sobre la [Integración de las identidades locales con Azure Active Directory](whatis-hybrid-identity.md).