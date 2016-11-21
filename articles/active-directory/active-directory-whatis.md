---
title: ¿Qué es Azure Active Directory?
description: Usar Azure Active Directory para ampliar sus identidades locales existentes en la nube o desarrollar aplicaciones integradas de Azure AD.
services: active-directory
documentationcenter: ''
author: markusvi
manager: femila
editor: ''

ms.service: active-directory
ms.workload: identity
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: article
ms.date: 08/23/2016
ms.author: markusvi

---
# ¿Qué es Azure Active Directory?
Azure Active Directory (Azure AD) es el directorio basado en la nube multiempresa y el servicio de administración de identidades de Microsoft.

Para los administradores de TI, Azure AD proporciona una solución asequible y fácil de usar para dar a los empleados y asociados comerciales acceso de inicio de sesión único (SSO) a [miles de aplicaciones SaaS en la nube](http://blogs.technet.com/b/ad/archive/2014/09/03/50-saas-apps-now-support-federation-with-azure-ad.aspx) como Office365, Salesforce.com, DropBox y Concur.

Para los desarrolladores de aplicaciones, Azure AD permite al usuario centrarse en la creación de su aplicación facilitando y acelerando la integración de la misma con una solución de administración de identidades de clase mundial usada por millones de organizaciones de todo el mundo.


Azure AD también incluye una serie completa de capacidades de administración de identidades, incluida Multi-Factor Authentication, el registro de dispositivos, la administración de contraseñas de autoservicio, la administración de grupos de autoservicio, la administración de cuentas con privilegios, el control de acceso basado en roles, la supervisión del uso de aplicaciones, la auditoría enriquecida y la supervisión y alertas de seguridad. Estas funcionalidades pueden ayudar a proteger aplicaciones basadas en la nube, optimizar los procesos de TI, reducir costos y asegurar que se cumplen los objetivos de cumplimiento normativo corporativos.

Además, con solo [cuatro clics](http://blogs.technet.com/b/ad/archive/2014/08/04/connecting-ad-and-azure-ad-only-4-clicks-with-azure-ad-connect.aspx), Azure AD puede integrarse con un Windows Server Active Directory existente, lo que da a las organizaciones la capacidad de aprovechar sus inversiones en identidad locales existentes para administrar el acceso a aplicaciones SaaS basadas en la nube.

Si es un cliente de Office365, Azure o Dynamics CRM Online, es posible que no se dé cuenta de que ya está usando Azure AD. Cada inquilino de Office365, Azure y Dynamics CRM ya es en realidad un inquilino de Azure AD. ¡Siempre que lo desee, puede empezar a usar ese inquilino para administrar el acceso a miles de otras aplicaciones en la con las que se integra Azure AD!

![Pila de Azure AD Connect](./media/active-directory-whatis/Azure_Active_Directory.png)

## ¿Cuál es el grado de confiabilidad Azure AD?
El diseño multiempresa, distribuido geográficamente y de alta disponibilidad de Azure AD significa que puede confiar en él para sus necesidades empresariales más críticas. Ejecutándose en 28 centros de datos en todo el mundo con conmutación por error automatizada, tendrá la tranquilidad de saber que Azure AD es muy confiable y que incluso si un centro de datos deja de funcionar, hay copias de su directorio de datos activas en al menos dos centros de datos más dispersos por regiones y disponibles para acceso instantáneo.

Para más información, consulta los [Contratos de nivel de servicio](https://azure.microsoft.com/support/legal/sla/).

## ¿Cuáles son las ventajas de Azure AD?
Su organización puede usar Azure AD para mejorar la productividad de los empleados, optimizar los procesos de TI, mejorar la seguridad y reducir los costos de muchas formas:

* Adopte rápidamente servicios en la nube, proporcionando a los empleados y asociados una sencilla experiencia de inicio de sesión único gracias a las capacidades automatizadas de aprovisionamiento de servicios y administración del acceso a aplicaciones SaaS de Azure AD.
* Proporcione a los empleados acceso a aplicaciones en la nube de clase mundial y a funcionalidades de servicio y autoservicio desde dondequiera que necesiten para trabajar en los dispositivos que les encanta usar.
* Administre fácilmente y de forma segura el acceso de los empleados y proveedores a sus cuentas de medios sociales corporativas.
* Mejore la seguridad de las aplicaciones con la autenticación multifactor y acceso condicional de Azure AD.
* Implemente una administración de acceso a las aplicaciones constante y de autoservicio, proporcionando a los propietarios de empresas los medios para avanzar rápidamente, reduciendo al mismo tiempo los costos de TI y la sobrecarga.
* Supervise el uso de las aplicaciones y proteja su empresa contra amenazas avanzadas con informes y supervisión de seguridad.
* Acceso móvil (remoto) seguro a aplicaciones locales.

## ¿Cómo puedo comenzar?
* Si es un administrador de TI:
  * [¡Pruébalo!](https://azure.microsoft.com/trial/get-started-active-directory/) Puedes suscribirte a una evaluación gratuita de 30 días hoy e implementar la primera solución en la nube en menos de 5 minutos desde este vínculo
  * Lea nuestra Introducción a Azure AD para obtener sugerencias y trucos sobre cómo preparar un inquilino de Azure AD y tenerlo en funcionamiento rápidamente.
* Si es un desarrollador:
  * Consulta las [Instrucciones para desarrolladores](active-directory-developers-guide.md) de Azure Active Directory
  * [Inicie una evaluación](https://azure.microsoft.com/trial/get-started-active-directory/): suscríbase a una prueba evaluación gratuita de 30 días hoy mismo y comience a integrar su aplicación con Azure AD.

## ¿Dónde puedo obtener más información?
Tenemos una gran cantidad de excelentes recursos en línea para ayudarle a aprender todo sobre Azure AD. Aquí se proporciona una lista de artículos excelentes para comenzar:

* [Habilitación del directorio para la administración híbrida con Azure AD Connect](active-directory-aadconnect.md)
* [Seguridad adicional en un mundo conectado permanentemente](../multi-factor-authentication/multi-factor-authentication.md)
* [Automatización del aprovisionamiento y desaprovisionamiento de usuarios para aplicaciones SaaS con Azure Active Directory](active-directory-saas-app-provisioning.md)
* [Introducción a los informes de Azure AD](active-directory-reporting-getting-started.md)
* [Administración de contraseñas desde cualquier lugar](active-directory-passwords.md)
* [¿Qué es el acceso a aplicaciones y el inicio de sesión único con Azure Active Directory?](active-directory-appssoaccess-whatis.md)
* [Automatización del aprovisionamiento y desaprovisionamiento de usuarios para aplicaciones SaaS con Azure Active Directory](active-directory-saas-app-provisioning.md)
* [Provisión de acceso remoto seguro a aplicaciones locales](active-directory-application-proxy-get-started.md)
* [Administración del acceso a los recursos con grupos de Azure Active Directory](active-directory-manage-groups.md)
* [¿Qué es la licencia de Microsoft Azure Active Directory?](active-directory-licensing-what-is.md)
* [¿Cómo puedo detectar aplicaciones en la nube no sancionadas que se usan dentro de mi organización?](active-directory-cloudappdiscovery-whatis.md)

##Recursos Adicionales
Presentación / Video Microsoft Virtual Academy: [Azure Active Directory desde cero](https://mva.microsoft.com/es-es/training-courses/azure-active-directory-desde-cero-16859?l=LXR2zLy7C_4005192797)

<!---HONumber=AcomDC_0928_2016-->
