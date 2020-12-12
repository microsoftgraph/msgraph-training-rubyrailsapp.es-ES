<!-- markdownlint-disable MD002 MD041 -->

Este tutorial le enseña a crear una aplicación Web de Ruby on Rails que use la API de Microsoft Graph para recuperar la información de calendario de un usuario.

> [!TIP]
> Si prefiere descargar solo el tutorial completo, puede descargarlo de dos maneras.
>
> - Descargue el [Inicio rápido de Ruby](https://developer.microsoft.com/graph/quick-start?platform=option-ruby) para obtener el código de trabajo en minutos.
> - Descargue o Clone el [repositorio de github](https://github.com/microsoftgraph/msgraph-training-rubyrailsapp).

## <a name="prerequisites"></a>Requisitos previos

Antes de iniciar este tutorial, debe tener las siguientes herramientas instaladas en el equipo de desarrollo.

- [Ruby](https://www.ruby-lang.org/en/downloads/)
- [SQLite3](https://sqlite.org/index.html)
- [Node.js](https://nodejs.org/en/)
- [Yarn](https://yarnpkg.com/)

También debe tener una cuenta de Microsoft personal con un buzón de correo en Outlook.com o una cuenta profesional o educativa de Microsoft. Si no tiene una cuenta de Microsoft, hay un par de opciones para obtener una cuenta gratuita:

- Puede [registrarse para obtener una nueva cuenta Microsoft personal](https://signup.live.com/signup?wa=wsignin1.0&rpsnv=12&ct=1454618383&rver=6.4.6456.0&wp=MBI_SSL_SHARED&wreply=https://mail.live.com/default.aspx&id=64855&cbcxt=mai&bk=1454618383&uiflavor=web&uaid=b213a65b4fdc484382b6622b3ecaa547&mkt=E-US&lc=1033&lic=1).
- Puede [registrarse para el programa de desarrolladores de office 365](https://developer.microsoft.com/office/dev-program) para obtener una suscripción gratuita a Office 365.

> [!NOTE]
> Este tutorial se ha redactado con las siguientes versiones de las herramientas necesarias. Los pasos de esta guía pueden funcionar con otras versiones, pero no se han probado.
>
> - Versión 2.6.6 de Ruby.
> - SQLite3 versión 3.31.1
> - Node.js versión 14.15.0
> - 1.22.0 de la versión de hilado

## <a name="feedback"></a>Comentarios

Envíe sus comentarios sobre este tutorial en el [repositorio de github](https://github.com/microsoftgraph/msgraph-training-rubyrailsapp).
