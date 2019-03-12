<!-- markdownlint-disable MD002 MD041 -->

<span data-ttu-id="6ccb2-101">Este tutorial le enseña a crear una aplicación Web de Ruby on Rails que use la API de Microsoft Graph para recuperar la información de calendario de un usuario.</span><span class="sxs-lookup"><span data-stu-id="6ccb2-101">This tutorial teaches you how to build a Ruby on Rails web app that uses the Microsoft Graph API to retrieve calendar information for a user.</span></span>

> [!TIP]
> <span data-ttu-id="6ccb2-102">Si prefiere descargar solo el tutorial completo, puede descargarlo de dos maneras.</span><span class="sxs-lookup"><span data-stu-id="6ccb2-102">If you prefer to just download the completed tutorial, you can download it in two ways.</span></span>
>
> - <span data-ttu-id="6ccb2-103">Descargue el [Inicio rápido de Ruby](https://developer.microsoft.com/graph/quick-start?platform=option-ruby) para obtener el código de trabajo en minutos.</span><span class="sxs-lookup"><span data-stu-id="6ccb2-103">Download the [Ruby quick start](https://developer.microsoft.com/graph/quick-start?platform=option-ruby) to get working code in minutes.</span></span>
> - <span data-ttu-id="6ccb2-104">Descargue o Clone el [repositorio de github](https://github.com/microsoftgraph/msgraph-training-rubyrailsapp).</span><span class="sxs-lookup"><span data-stu-id="6ccb2-104">Download or clone the [GitHub repository](https://github.com/microsoftgraph/msgraph-training-rubyrailsapp).</span></span>

## <a name="prerequisites"></a><span data-ttu-id="6ccb2-105">Requisitos previos</span><span class="sxs-lookup"><span data-stu-id="6ccb2-105">Prerequisites</span></span>

<span data-ttu-id="6ccb2-106">Antes de iniciar este tutorial, debe tener [Ruby](https://www.ruby-lang.org/en/downloads/) instalado en el equipo de desarrollo.</span><span class="sxs-lookup"><span data-stu-id="6ccb2-106">Before you start this tutorial, you should have [Ruby](https://www.ruby-lang.org/en/downloads/) installed on your development machine.</span></span> <span data-ttu-id="6ccb2-107">Si no tiene Ruby, visite el vínculo anterior para las opciones de descarga.</span><span class="sxs-lookup"><span data-stu-id="6ccb2-107">If you do not have Ruby, visit the previous link for download options.</span></span>

> [!NOTE]
> <span data-ttu-id="6ccb2-108">Este tutorial se ha escrito con la versión 2.4.4 de Ruby.</span><span class="sxs-lookup"><span data-stu-id="6ccb2-108">This tutorial was written with Ruby version 2.4.4.</span></span> <span data-ttu-id="6ccb2-109">Los pasos de esta guía pueden funcionar con otras versiones, pero no se han probado.</span><span class="sxs-lookup"><span data-stu-id="6ccb2-109">The steps in this guide may work with other versions, but that has not been tested.</span></span>

## <a name="feedback"></a><span data-ttu-id="6ccb2-110">Comentarios</span><span class="sxs-lookup"><span data-stu-id="6ccb2-110">Feedback</span></span>

<span data-ttu-id="6ccb2-111">Envíe sus comentarios sobre este tutorial en el [repositorio de github](https://github.com/microsoftgraph/msgraph-training-rubyrailsapp).</span><span class="sxs-lookup"><span data-stu-id="6ccb2-111">Please provide any feedback on this tutorial in the [GitHub repository](https://github.com/microsoftgraph/msgraph-training-rubyrailsapp).</span></span>