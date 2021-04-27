<!-- markdownlint-disable MD002 MD041 -->

<span data-ttu-id="9878d-101">En este ejercicio, [usarás Ruby on Rails](https://rubyonrails.org/) para crear una aplicación web.</span><span class="sxs-lookup"><span data-stu-id="9878d-101">In this exercise you will use [Ruby on Rails](https://rubyonrails.org/) to build a web app.</span></span>

1. <span data-ttu-id="9878d-102">Si aún no tiene rails instalado, puede instalarlo desde la interfaz de línea de comandos (CLI) con el siguiente comando.</span><span class="sxs-lookup"><span data-stu-id="9878d-102">If you don't already have Rails installed, you can install it from your command-line interface (CLI) with the following command.</span></span>

    ```Shell
    gem install rails -v 6.1.3.1
    ```

1. <span data-ttu-id="9878d-103">Abre la CLI, navega a un directorio donde tienes derechos para crear archivos y ejecuta el siguiente comando para crear una nueva aplicación Rails.</span><span class="sxs-lookup"><span data-stu-id="9878d-103">Open your CLI, navigate to a directory where you have rights to create files, and run the following command to create a new Rails app.</span></span>

    ```Shell
    rails new graph-tutorial
    ```

1. <span data-ttu-id="9878d-104">Navegue a este nuevo directorio y escriba el siguiente comando para iniciar un servidor web local.</span><span class="sxs-lookup"><span data-stu-id="9878d-104">Navigate to this new directory and enter the following command to start a local web server.</span></span>

    ```Shell
    rails server
    ```

1. <span data-ttu-id="9878d-105">Abra el explorador y vaya a `http://localhost:3000`.</span><span class="sxs-lookup"><span data-stu-id="9878d-105">Open your browser and navigate to `http://localhost:3000`.</span></span> <span data-ttu-id="9878d-106">Si todo funciona, verá un "Yay!</span><span class="sxs-lookup"><span data-stu-id="9878d-106">If everything is working, you will see a "Yay!</span></span> <span data-ttu-id="9878d-107">You're on Rails!"</span><span class="sxs-lookup"><span data-stu-id="9878d-107">You're on Rails!"</span></span> <span data-ttu-id="9878d-108">Mensaje.</span><span class="sxs-lookup"><span data-stu-id="9878d-108">message.</span></span> <span data-ttu-id="9878d-109">Si no ve ese mensaje, consulte la guía de introducción a [Rails](http://guides.rubyonrails.org/).</span><span class="sxs-lookup"><span data-stu-id="9878d-109">If you don't see that message, check the [Rails getting started guide](http://guides.rubyonrails.org/).</span></span>

## <a name="install-gems"></a><span data-ttu-id="9878d-110">Instalar gemas</span><span class="sxs-lookup"><span data-stu-id="9878d-110">Install gems</span></span>

<span data-ttu-id="9878d-111">Antes de seguir adelante, instala algunas gemas adicionales que usarás más adelante:</span><span class="sxs-lookup"><span data-stu-id="9878d-111">Before moving on, install some additional gems that you will use later:</span></span>

- <span data-ttu-id="9878d-112">[omniauth-oauth2](https://github.com/omniauth/omniauth-oauth2) para controlar los flujos de tokens de inicio de sesión y OAuth.</span><span class="sxs-lookup"><span data-stu-id="9878d-112">[omniauth-oauth2](https://github.com/omniauth/omniauth-oauth2) for handling sign-in and OAuth token flows.</span></span>
- <span data-ttu-id="9878d-113">[omniauth-rails_csrf_protection](https://github.com/cookpad/omniauth-rails_csrf_protection) para agregar protección CSRF a OmniAuth.</span><span class="sxs-lookup"><span data-stu-id="9878d-113">[omniauth-rails_csrf_protection](https://github.com/cookpad/omniauth-rails_csrf_protection) for adding CSRF protection to OmniAuth.</span></span>
- <span data-ttu-id="9878d-114">[httparty](https://github.com/jnunemaker/httparty) para realizar llamadas a Microsoft Graph.</span><span class="sxs-lookup"><span data-stu-id="9878d-114">[httparty](https://github.com/jnunemaker/httparty) for making calls to Microsoft Graph.</span></span>
- <span data-ttu-id="9878d-115">[activerecord-session_store](https://github.com/rails/activerecord-session_store) para almacenar sesiones en la base de datos.</span><span class="sxs-lookup"><span data-stu-id="9878d-115">[activerecord-session_store](https://github.com/rails/activerecord-session_store) for storing sessions in the database.</span></span>

1. <span data-ttu-id="9878d-116">Abra **./Gemfile** y agregue las siguientes líneas.</span><span class="sxs-lookup"><span data-stu-id="9878d-116">Open **./Gemfile** and add the following lines.</span></span>

    :::code language="ruby" source="../demo/graph-tutorial/Gemfile" id="GemFileSnippet":::

1. <span data-ttu-id="9878d-117">En la CLI, ejecute el siguiente comando.</span><span class="sxs-lookup"><span data-stu-id="9878d-117">In your CLI, run the following command.</span></span>

    ```Shell
    bundle install
    ```

1. <span data-ttu-id="9878d-118">En la CLI, ejecute los siguientes comandos para configurar la base de datos para almacenar sesiones.</span><span class="sxs-lookup"><span data-stu-id="9878d-118">In your CLI, run the following commands to configure the database for storing sessions.</span></span>

    ```Shell
    rails generate active_record:session_migration
    rake db:migrate
    ```

1. <span data-ttu-id="9878d-119">Cree un nuevo archivo llamado `session_store.rb` en el directorio **./config/initializers** y agregue el siguiente código.</span><span class="sxs-lookup"><span data-stu-id="9878d-119">Create a new file called `session_store.rb` in the **./config/initializers** directory, and add the following code.</span></span>

    :::code language="ruby" source="../demo/graph-tutorial/config/initializers/session_store.rb" id="SessionStoreSnippet":::

## <a name="design-the-app"></a><span data-ttu-id="9878d-120">Diseñar la aplicación</span><span class="sxs-lookup"><span data-stu-id="9878d-120">Design the app</span></span>

<span data-ttu-id="9878d-121">En esta sección, crearás la interfaz de usuario básica para la aplicación.</span><span class="sxs-lookup"><span data-stu-id="9878d-121">In this section you'll create the basic UI for the app.</span></span>

1. <span data-ttu-id="9878d-122">Abra **./app/views/layouts/application.html.erb** y reemplace su contenido por lo siguiente.</span><span class="sxs-lookup"><span data-stu-id="9878d-122">Open **./app/views/layouts/application.html.erb** and replace its contents with the following.</span></span>

    :::code language="html" source="../demo/graph-tutorial/app/views/layouts/application.html.erb" id="LayoutSnippet":::

    <span data-ttu-id="9878d-123">Este código agrega [Bootstrap para](http://getbootstrap.com/) un estilo sencillo y [Fabric Core](https://developer.microsoft.com/fluentui#/get-started#fabric-core) para algunos iconos sencillos.</span><span class="sxs-lookup"><span data-stu-id="9878d-123">This code adds [Bootstrap](http://getbootstrap.com/) for simple styling, and [Fabric Core](https://developer.microsoft.com/fluentui#/get-started#fabric-core) for some simple icons.</span></span> <span data-ttu-id="9878d-124">También define un diseño global con una barra de navegación.</span><span class="sxs-lookup"><span data-stu-id="9878d-124">It also defines a global layout with a nav bar.</span></span>

1. <span data-ttu-id="9878d-125">Abra **./app/assets/stylesheets/application.css** y agregue lo siguiente al final del archivo.</span><span class="sxs-lookup"><span data-stu-id="9878d-125">Open **./app/assets/stylesheets/application.css** and add the following to the end of the file.</span></span>

    :::code language="css" source="../demo/graph-tutorial/app/assets/stylesheets/application.css" id="CssSnippet":::

1. <span data-ttu-id="9878d-126">Genere un controlador de página principal con el siguiente comando.</span><span class="sxs-lookup"><span data-stu-id="9878d-126">Generate a home page controller with the following command.</span></span>

    ```Shell
    rails generate controller Home index
    ```

1. <span data-ttu-id="9878d-127">Configura la `index` acción en el controlador como la página predeterminada de la `Home` aplicación.</span><span class="sxs-lookup"><span data-stu-id="9878d-127">Configure the `index` action on the `Home` controller as the default page for the app.</span></span> <span data-ttu-id="9878d-128">Abra **./config/routes.rb y** reemplace su contenido por el siguiente</span><span class="sxs-lookup"><span data-stu-id="9878d-128">Open **./config/routes.rb** and replace its contents with the following</span></span>

    ```ruby
    Rails.application.routes.draw do
      get 'home/index'
      root 'home#index'

      # Add future routes here

    end
    ```

1. <span data-ttu-id="9878d-129">Abra **./app/view/home/index.html.erb** y reemplace su contenido por lo siguiente.</span><span class="sxs-lookup"><span data-stu-id="9878d-129">Open **./app/view/home/index.html.erb** and replace its contents with the following.</span></span>

    :::code language="html" source="../demo/graph-tutorial/app/views/home/index.html.erb" id="HomeSnippet":::

1. <span data-ttu-id="9878d-130">Agregue un archivo PNG denominado **no-profile-photo.png** en el directorio **./app/assets/images.**</span><span class="sxs-lookup"><span data-stu-id="9878d-130">Add a PNG file named **no-profile-photo.png** in the **./app/assets/images** directory.</span></span>

1. <span data-ttu-id="9878d-131">Guarde todos los cambios y reinicie el servidor.</span><span class="sxs-lookup"><span data-stu-id="9878d-131">Save all of your changes and restart the server.</span></span> <span data-ttu-id="9878d-132">Ahora, la aplicación debe tener un aspecto muy diferente.</span><span class="sxs-lookup"><span data-stu-id="9878d-132">Now, the app should look very different.</span></span>

    ![Una captura de pantalla de la página de inicio rediseñada](./images/create-app-01.png)
