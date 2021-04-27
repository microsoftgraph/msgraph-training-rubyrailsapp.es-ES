<!-- markdownlint-disable MD002 MD041 -->

Este tutorial le enseña a crear una aplicación web ruby on rails que use la API de Microsoft Graph para recuperar información de calendario para un usuario.

> [!TIP]
> Si prefiere descargar el tutorial completado, puede descargarlo de dos maneras.
>
> - Descargue el [inicio rápido de Ruby](https://developer.microsoft.com/graph/quick-start?platform=option-ruby) para obtener el código de trabajo en minutos.
> - Descargue o clone el [repositorio GitHub archivo](https://github.com/microsoftgraph/msgraph-training-rubyrailsapp).

## <a name="prerequisites"></a>Requisitos previos

Antes de iniciar este tutorial, debe tener instaladas las siguientes herramientas en el equipo de desarrollo.

- [Ruby](https://www.ruby-lang.org/en/downloads/)
- [SQLite3](https://sqlite.org/index.html)
- [Node.js](https://nodejs.org/en/)
- [Yarn](https://yarnpkg.com/)

También debe tener una cuenta personal de Microsoft con un buzón en Outlook.com, o una cuenta de Trabajo o escuela de Microsoft. Si no tienes una cuenta de Microsoft, hay un par de opciones para obtener una cuenta gratuita:

- Puede registrarse [para obtener una nueva cuenta personal de Microsoft](https://signup.live.com/signup?wa=wsignin1.0&rpsnv=12&ct=1454618383&rver=6.4.6456.0&wp=MBI_SSL_SHARED&wreply=https://mail.live.com/default.aspx&id=64855&cbcxt=mai&bk=1454618383&uiflavor=web&uaid=b213a65b4fdc484382b6622b3ecaa547&mkt=E-US&lc=1033&lic=1).
- Puedes [registrarte en el programa Office 365 desarrolladores](https://developer.microsoft.com/office/dev-program) para obtener una suscripción Office 365 gratuita.

> [!NOTE]
> Este tutorial se escribió con las siguientes versiones de las herramientas necesarias. Los pasos de esta guía pueden funcionar con otras versiones, pero eso no se ha probado.
>
> - Ruby versión 3.0.1
> - SQLite3 versión 3.35.5
> - Node.js versión 14.15.0
> - Yarn versión 1.22.0

## <a name="feedback"></a>Comentarios

Proporcione cualquier comentario sobre este tutorial en el [repositorio GitHub archivo](https://github.com/microsoftgraph/msgraph-training-rubyrailsapp).
