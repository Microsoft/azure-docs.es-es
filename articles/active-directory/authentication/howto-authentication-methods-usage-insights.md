---
title: 'Uso de métodos de autenticación y conclusiones: Azure Active Directory'
description: Creación de informes sobre el uso del autoservicio de restablecimiento de contraseña y el método de autenticación multifactor de Azure AD
services: active-directory
ms.service: active-directory
ms.subservice: authentication
ms.topic: how-to
ms.date: 11/21/2019
ms.author: iainfou
author: iainfoulds
manager: daveba
ms.reviewer: sahenry
ms.collection: M365-identity-device-management
ms.openlocfilehash: f75c08f0a390e798435b70bbe2409ec4a9732214
ms.sourcegitcommit: 6fc156ceedd0fbbb2eec1e9f5e3c6d0915f65b8e
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/21/2020
ms.locfileid: "88717173"
---
# <a name="authentication-methods-usage--insights-preview"></a>Conclusiones y uso de los métodos de autenticación (versión preliminar)

El uso y las conclusiones le permiten comprender cómo funcionan en su organización los métodos de autenticación para características como la autenticación multifactor y el autoservicio de restablecimiento de contraseña de Azure. Esta funcionalidad de informes proporciona a la organización los medios para comprender qué métodos se registran y cómo se utilizan.

## <a name="permissions-and-licenses"></a>Permisos y licencias

Los siguientes roles pueden tener acceso al uso y las conclusiones:

- Administrador global
- Lector de seguridad
- Administrador de seguridad
- Lector de informes

No se necesita ninguna licencia adicional para acceder al uso y las conclusiones. La información acerca de las licencias de la autenticación multifactor y el autoservicio de restablecimiento de contraseña (SSPR) de Azure puede encontrarse en el [sitio web de precios de Azure Active Directory](https://azure.microsoft.com/pricing/details/active-directory/).

## <a name="how-it-works"></a>Funcionamiento

Para obtener acceso al uso y las conclusiones del método de autenticación:

1. Vaya a [Azure Portal](https://portal.azure.com).
1. Vaya a **Azure Active Directory** > **Restablecimiento de contraseña** > **Uso y conclusiones**.
1. Desde las introducciones **Registro** o **Uso**, puede elegir abrir los informes filtrados previamente para filtrarlos según sus necesidades.

![Introducción al uso y las conclusiones](./media/howto-authentication-methods-usage-insights/usage-insights-overview.png)

Para obtener acceso al uso y las conclusiones directamente, vaya a [https://portal.azure.com/#blade/Microsoft_AAD_IAM/AuthMethodsOverviewBlade](https://portal.azure.com/#blade/Microsoft_AAD_IAM/AuthMethodsOverviewBlade). Este vínculo le llevará a la introducción del registro.

Los iconos de Usuarios registrados, Usuarios habilitados y Usuarios capaces muestran los siguientes datos de registro para sus usuarios:

- Registrado: se considera que un usuario está registrado si él o un administrador han registrado suficientes métodos de autenticación para cumplir la directiva de autenticación multifactor o SSPR de la organización.
- Habilitado: se considera que un usuario está habilitado si se encuentra en el ámbito de la directiva SSPR. Si SSPR está habilitado para un grupo, el usuario se considera habilitado si se encuentran en ese grupo. Si SSPR está habilitado para todos los usuarios, se consideran habilitados todos los usuarios en el inquilino (excepto los invitados).
- Capaz: se considera que un usuario es capaz si está tanto registrado como habilitado. Este estado significa que puede realizar SSPR en cualquier momento si es necesario.

Al hacer clic en cualquiera de estos iconos o en las conclusiones que se muestran en ellos, se le dirigirá a una lista de detalles de registro filtrada previamente.

El gráfico **Registros** de la pestaña **Registro** muestra el número de registros de métodos de autenticación correctos e incorrectos por método de autenticación. El gráfico **Restablecimientos** de la pestaña **Uso** muestra el número de autenticaciones correctas e incorrectas durante el flujo de restablecimiento de contraseña por método de autenticación.

Al hacer clic en cualquiera de los gráficos se le llevará a una lista de los eventos de registro o restablecimiento filtrada previamente.

Mediante el control de la esquina superior derecha puede cambiar el intervalo de fechas para los datos de auditoría que se muestran en los gráficos Registros y Restablecimientos a 24 horas, 7 días o 30 días.

### <a name="registration-details"></a>Detalles de registro

Al hacer clic en los iconos o las conclusiones de **Usuarios registrados**, **Usuarios habilitados** o **Usuarios capaces**, se le llevará a los detalles de registro.

El informe de detalles de registro muestra la siguiente información para cada usuario:

- Nombre
- Nombre de usuario
- Estado de registrado (todos, registrados, no registrados)
- Estado de habilitado (todos, habilitados, no habilitados)
- Estado de capaz (todos, capaces, no capaces)
- Métodos (notificación de la aplicación, código de la aplicación, llamada telefónica, SMS, correo electrónico, preguntas de seguridad)

Mediante los controles de la parte superior de la lista, puede buscar un usuario y filtrar la lista de usuarios en función de las columnas que se muestran.

### <a name="reset-details"></a>Detalles de restablecimiento

Al hacer clic en los gráficos Registros o Restablecimientos, se le llevará a los detalles de restablecimientos.

El informe de detalles de restablecimientos muestra los eventos de registro y restablecimiento de los últimos 30 días, que incluyen:

- Nombre
- Nombre de usuario
- Característica (todas, registro, restablecimiento)
- Método de autenticación (notificación de la aplicación, código de la aplicación, llamada telefónica, llamada de Office, SMS, correo electrónico, preguntas de seguridad)
- Estado (todo, correcto, incorrecto)

Mediante los controles de la parte superior de la lista, puede buscar un usuario y filtrar la lista de usuarios en función de las columnas que se muestran.

## <a name="limitations"></a>Limitaciones

Los datos que se muestran en estos informes se retrasarán en hasta 60 minutos. Existe un campo de "Última actualización" en Azure Portal para identificar la antigüedad de los datos.

Los datos de uso y conclusiones no sustituyen los informes de actividad de la autenticación multifactor de Azure ni la información del informe de inicios de sesión de Azure AD.

Actualmente no se puede filtrar el informe para excluir a los usuarios externos.

## <a name="next-steps"></a>Pasos siguientes

- [Trabajar con la API del informe de utilización de métodos de autenticación](/graph/api/resources/authenticationmethods-usage-insights-overview?view=graph-rest-beta)
- [Elegir los métodos de autenticación para la organización](concept-authentication-methods.md)
- [Experiencia de registro combinada](concept-registration-mfa-sspr-combined.md)