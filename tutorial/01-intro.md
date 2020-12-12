<!-- markdownlint-disable MD002 MD041 -->

<span data-ttu-id="6d144-101">Este tutorial le enseña a crear una aplicación Web de Ruby on Rails que use la API de Microsoft Graph para recuperar la información de calendario de un usuario.</span><span class="sxs-lookup"><span data-stu-id="6d144-101">This tutorial teaches you how to build a Ruby on Rails web app that uses the Microsoft Graph API to retrieve calendar information for a user.</span></span>

> [!TIP]
> <span data-ttu-id="6d144-102">Si prefiere descargar solo el tutorial completo, puede descargarlo de dos maneras.</span><span class="sxs-lookup"><span data-stu-id="6d144-102">If you prefer to just download the completed tutorial, you can download it in two ways.</span></span>
>
> - <span data-ttu-id="6d144-103">Descargue el [Inicio rápido de Ruby](https://developer.microsoft.com/graph/quick-start?platform=option-ruby) para obtener el código de trabajo en minutos.</span><span class="sxs-lookup"><span data-stu-id="6d144-103">Download the [Ruby quick start](https://developer.microsoft.com/graph/quick-start?platform=option-ruby) to get working code in minutes.</span></span>
> - <span data-ttu-id="6d144-104">Descargue o Clone el [repositorio de github](https://github.com/microsoftgraph/msgraph-training-rubyrailsapp).</span><span class="sxs-lookup"><span data-stu-id="6d144-104">Download or clone the [GitHub repository](https://github.com/microsoftgraph/msgraph-training-rubyrailsapp).</span></span>

## <a name="prerequisites"></a><span data-ttu-id="6d144-105">Requisitos previos</span><span class="sxs-lookup"><span data-stu-id="6d144-105">Prerequisites</span></span>

<span data-ttu-id="6d144-106">Antes de iniciar este tutorial, debe tener las siguientes herramientas instaladas en el equipo de desarrollo.</span><span class="sxs-lookup"><span data-stu-id="6d144-106">Before you start this tutorial, you should have the following tools installed on your development machine.</span></span>

- [<span data-ttu-id="6d144-107">Ruby</span><span class="sxs-lookup"><span data-stu-id="6d144-107">Ruby</span></span>](https://www.ruby-lang.org/en/downloads/)
- [<span data-ttu-id="6d144-108">SQLite3</span><span class="sxs-lookup"><span data-stu-id="6d144-108">SQLite3</span></span>](https://sqlite.org/index.html)
- [<span data-ttu-id="6d144-109">Node.js</span><span class="sxs-lookup"><span data-stu-id="6d144-109">Node.js</span></span>](https://nodejs.org/en/)
- [<span data-ttu-id="6d144-110">Yarn</span><span class="sxs-lookup"><span data-stu-id="6d144-110">Yarn</span></span>](https://yarnpkg.com/)

<span data-ttu-id="6d144-111">También debe tener una cuenta de Microsoft personal con un buzón de correo en Outlook.com o una cuenta profesional o educativa de Microsoft.</span><span class="sxs-lookup"><span data-stu-id="6d144-111">You should also have either a personal Microsoft account with a mailbox on Outlook.com, or a Microsoft work or school account.</span></span> <span data-ttu-id="6d144-112">Si no tiene una cuenta de Microsoft, hay un par de opciones para obtener una cuenta gratuita:</span><span class="sxs-lookup"><span data-stu-id="6d144-112">If you don't have a Microsoft account, there are a couple of options to get a free account:</span></span>

- <span data-ttu-id="6d144-113">Puede [registrarse para obtener una nueva cuenta Microsoft personal](https://signup.live.com/signup?wa=wsignin1.0&rpsnv=12&ct=1454618383&rver=6.4.6456.0&wp=MBI_SSL_SHARED&wreply=https://mail.live.com/default.aspx&id=64855&cbcxt=mai&bk=1454618383&uiflavor=web&uaid=b213a65b4fdc484382b6622b3ecaa547&mkt=E-US&lc=1033&lic=1).</span><span class="sxs-lookup"><span data-stu-id="6d144-113">You can [sign up for a new personal Microsoft account](https://signup.live.com/signup?wa=wsignin1.0&rpsnv=12&ct=1454618383&rver=6.4.6456.0&wp=MBI_SSL_SHARED&wreply=https://mail.live.com/default.aspx&id=64855&cbcxt=mai&bk=1454618383&uiflavor=web&uaid=b213a65b4fdc484382b6622b3ecaa547&mkt=E-US&lc=1033&lic=1).</span></span>
- <span data-ttu-id="6d144-114">Puede [registrarse para el programa de desarrolladores de office 365](https://developer.microsoft.com/office/dev-program) para obtener una suscripción gratuita a Office 365.</span><span class="sxs-lookup"><span data-stu-id="6d144-114">You can [sign up for the Office 365 Developer Program](https://developer.microsoft.com/office/dev-program) to get a free Office 365 subscription.</span></span>

> [!NOTE]
> <span data-ttu-id="6d144-115">Este tutorial se ha redactado con las siguientes versiones de las herramientas necesarias.</span><span class="sxs-lookup"><span data-stu-id="6d144-115">This tutorial was written with the following versions of the required tools.</span></span> <span data-ttu-id="6d144-116">Los pasos de esta guía pueden funcionar con otras versiones, pero no se han probado.</span><span class="sxs-lookup"><span data-stu-id="6d144-116">The steps in this guide may work with other versions, but that has not been tested.</span></span>
>
> - <span data-ttu-id="6d144-117">Versión 2.6.6 de Ruby.</span><span class="sxs-lookup"><span data-stu-id="6d144-117">Ruby version 2.6.6.</span></span>
> - <span data-ttu-id="6d144-118">SQLite3 versión 3.31.1</span><span class="sxs-lookup"><span data-stu-id="6d144-118">SQLite3 version 3.31.1</span></span>
> - <span data-ttu-id="6d144-119">Node.js versión 14.15.0</span><span class="sxs-lookup"><span data-stu-id="6d144-119">Node.js version 14.15.0</span></span>
> - <span data-ttu-id="6d144-120">1.22.0 de la versión de hilado</span><span class="sxs-lookup"><span data-stu-id="6d144-120">Yarn version 1.22.0</span></span>

## <a name="feedback"></a><span data-ttu-id="6d144-121">Comentarios</span><span class="sxs-lookup"><span data-stu-id="6d144-121">Feedback</span></span>

<span data-ttu-id="6d144-122">Envíe sus comentarios sobre este tutorial en el [repositorio de github](https://github.com/microsoftgraph/msgraph-training-rubyrailsapp).</span><span class="sxs-lookup"><span data-stu-id="6d144-122">Please provide any feedback on this tutorial in the [GitHub repository](https://github.com/microsoftgraph/msgraph-training-rubyrailsapp).</span></span>
