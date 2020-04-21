<!-- markdownlint-disable MD002 MD041 -->

<span data-ttu-id="bed56-101">En este ejercicio usará [Ruby on Rails](https://rubyonrails.org/) para crear una aplicación Web.</span><span class="sxs-lookup"><span data-stu-id="bed56-101">In this exercise you will use [Ruby on Rails](https://rubyonrails.org/) to build a web app.</span></span>

1. <span data-ttu-id="bed56-102">Si aún no tiene guías instaladas, puede instalarla desde la interfaz de línea de comandos (CLI) con el siguiente comando.</span><span class="sxs-lookup"><span data-stu-id="bed56-102">If you don't already have Rails installed, you can install it from your command-line interface (CLI) with the following command.</span></span>

    ```Shell
    gem install rails -v 6.0.2.2
    ```

1. <span data-ttu-id="bed56-103">Abra la CLI, navegue hasta un directorio donde tenga derechos para crear archivos y ejecute el siguiente comando para crear una nueva aplicación de guías.</span><span class="sxs-lookup"><span data-stu-id="bed56-103">Open your CLI, navigate to a directory where you have rights to create files, and run the following command to create a new Rails app.</span></span>

    ```Shell
    rails new graph-tutorial
    ```

1. <span data-ttu-id="bed56-104">Navegue a este nuevo directorio y escriba el siguiente comando para iniciar un servidor Web local.</span><span class="sxs-lookup"><span data-stu-id="bed56-104">Navigate to this new directory and enter the following command to start a local web server.</span></span>

    ```Shell
    rails server
    ```

1. <span data-ttu-id="bed56-105">Abra el explorador y vaya a `http://localhost:3000`.</span><span class="sxs-lookup"><span data-stu-id="bed56-105">Open your browser and navigate to `http://localhost:3000`.</span></span> <span data-ttu-id="bed56-106">Si todo funciona, verá un error "Yay!</span><span class="sxs-lookup"><span data-stu-id="bed56-106">If everything is working, you will see a "Yay!</span></span> <span data-ttu-id="bed56-107">¡ Estás en raíles! "</span><span class="sxs-lookup"><span data-stu-id="bed56-107">You're on Rails!"</span></span> <span data-ttu-id="bed56-108">Mensaje.</span><span class="sxs-lookup"><span data-stu-id="bed56-108">message.</span></span> <span data-ttu-id="bed56-109">Si no ve ese mensaje, consulte la guía de introducción a los [rieles](http://guides.rubyonrails.org/).</span><span class="sxs-lookup"><span data-stu-id="bed56-109">If you don't see that message, check the [Rails getting started guide](http://guides.rubyonrails.org/).</span></span>

## <a name="install-gems"></a><span data-ttu-id="bed56-110">Instalar gemas</span><span class="sxs-lookup"><span data-stu-id="bed56-110">Install gems</span></span>

<span data-ttu-id="bed56-111">Antes de continuar, instale algunas gemas adicionales que usará más adelante:</span><span class="sxs-lookup"><span data-stu-id="bed56-111">Before moving on, install some additional gems that you will use later:</span></span>

- <span data-ttu-id="bed56-112">[omniauth-OAuth2](https://github.com/omniauth/omniauth-oauth2) para controlar los flujos de tokens de OAuth y inicio de sesión.</span><span class="sxs-lookup"><span data-stu-id="bed56-112">[omniauth-oauth2](https://github.com/omniauth/omniauth-oauth2) for handling sign-in and OAuth token flows.</span></span>
- <span data-ttu-id="bed56-113">[omniauth-rails_csrf_protection](https://github.com/cookpad/omniauth-rails_csrf_protection) para agregar la protección CSRF a omniauth.</span><span class="sxs-lookup"><span data-stu-id="bed56-113">[omniauth-rails_csrf_protection](https://github.com/cookpad/omniauth-rails_csrf_protection) for adding CSRF protection to OmniAuth.</span></span>
- <span data-ttu-id="bed56-114">[httparty](https://github.com/jnunemaker/httparty) para realizar llamadas a Microsoft Graph.</span><span class="sxs-lookup"><span data-stu-id="bed56-114">[httparty](https://github.com/jnunemaker/httparty) for making calls to Microsoft Graph.</span></span>
- <span data-ttu-id="bed56-115">[ActiveRecord-session_store](https://github.com/rails/activerecord-session_store) para almacenar sesiones en la base de datos.</span><span class="sxs-lookup"><span data-stu-id="bed56-115">[activerecord-session_store](https://github.com/rails/activerecord-session_store) for storing sessions in the database.</span></span>

1. <span data-ttu-id="bed56-116">Abra **./Gemfile** y agregue las siguientes líneas.</span><span class="sxs-lookup"><span data-stu-id="bed56-116">Open **./Gemfile** and add the following lines.</span></span>

    :::code language="ruby" source="../demo/graph-tutorial/Gemfile" id="GemFileSnippet":::

1. <span data-ttu-id="bed56-117">En la CLI, ejecute el siguiente comando.</span><span class="sxs-lookup"><span data-stu-id="bed56-117">In your CLI, run the following command.</span></span>

    ```Shell
    bundle install
    ```

1. <span data-ttu-id="bed56-118">En la CLI, ejecute los siguientes comandos para configurar la base de datos para almacenar las sesiones.</span><span class="sxs-lookup"><span data-stu-id="bed56-118">In your CLI, run the following commands to configure the database for storing sessions.</span></span>

    ```Shell
    rails generate active_record:session_migration
    rake db:migrate
    ```

1. <span data-ttu-id="bed56-119">Cree un nuevo archivo llamado `session_store.rb` en el directorio **./config/Initializers** y agregue el siguiente código.</span><span class="sxs-lookup"><span data-stu-id="bed56-119">Create a new file called `session_store.rb` in the **./config/initializers** directory, and add the following code.</span></span>

    :::code language="ruby" source="../demo/graph-tutorial/config/initializers/session_store.rb" id="SessionStoreSnippet":::

## <a name="design-the-app"></a><span data-ttu-id="bed56-120">Diseñar la aplicación</span><span class="sxs-lookup"><span data-stu-id="bed56-120">Design the app</span></span>

<span data-ttu-id="bed56-121">En esta sección, creará la interfaz de usuario básica para la aplicación.</span><span class="sxs-lookup"><span data-stu-id="bed56-121">In this section you'll create the basic UI for the app.</span></span>

1. <span data-ttu-id="bed56-122">Abra **./app/views/layouts/Application.html.Erb** y reemplace su contenido por lo siguiente.</span><span class="sxs-lookup"><span data-stu-id="bed56-122">Open **./app/views/layouts/application.html.erb** and replace its contents with the following.</span></span>

    :::code language="html" source="../demo/graph-tutorial/app/views/layouts/application.html.erb" id="LayoutSnippet":::

    <span data-ttu-id="bed56-123">Este código agrega un [bootstrap](http://getbootstrap.com/) para los estilos sencillos y la [fuente maravilla](https://fontawesome.com/) para algunos iconos simples.</span><span class="sxs-lookup"><span data-stu-id="bed56-123">This code adds [Bootstrap](http://getbootstrap.com/) for simple styling, and [Font Awesome](https://fontawesome.com/) for some simple icons.</span></span> <span data-ttu-id="bed56-124">También define un diseño global con una barra de navegación.</span><span class="sxs-lookup"><span data-stu-id="bed56-124">It also defines a global layout with a nav bar.</span></span>

1. <span data-ttu-id="bed56-125">Abra **./app/assets/Stylesheets/Application.CSS** y agregue lo siguiente al final del archivo.</span><span class="sxs-lookup"><span data-stu-id="bed56-125">Open **./app/assets/stylesheets/application.css** and add the following to the end of the file.</span></span>

    :::code language="css" source="../demo/graph-tutorial/app/assets/stylesheets/application.css" id="CssSnippet":::

1. <span data-ttu-id="bed56-126">Genere un controlador de páginas principales con el siguiente comando.</span><span class="sxs-lookup"><span data-stu-id="bed56-126">Generate a home page controller with the following command.</span></span>

    ```Shell
    rails generate controller Home index
    ```

1. <span data-ttu-id="bed56-127">Configure `index` la acción en `Home` el controlador como la página predeterminada para la aplicación.</span><span class="sxs-lookup"><span data-stu-id="bed56-127">Configure the `index` action on the `Home` controller as the default page for the app.</span></span> <span data-ttu-id="bed56-128">Abra **./config/Routes.RB** y reemplace su contenido por lo siguiente:</span><span class="sxs-lookup"><span data-stu-id="bed56-128">Open **./config/routes.rb** and replace its contents with the following</span></span>

    ```ruby
    Rails.application.routes.draw do
      get 'home/index'
      root 'home#index'

      # Add future routes here

    end
    ```

1. <span data-ttu-id="bed56-129">Abra **./app/View/home/index.html.Erb** y reemplace su contenido por lo siguiente.</span><span class="sxs-lookup"><span data-stu-id="bed56-129">Open **./app/view/home/index.html.erb** and replace its contents with the following.</span></span>

    :::code language="html" source="../demo/graph-tutorial/app/views/home/index.html.erb" id="HomeSnippet":::

1. <span data-ttu-id="bed56-130">Guarde todos los cambios y reinicie el servidor.</span><span class="sxs-lookup"><span data-stu-id="bed56-130">Save all of your changes and restart the server.</span></span> <span data-ttu-id="bed56-131">Ahora, la aplicación debe tener un aspecto muy diferente.</span><span class="sxs-lookup"><span data-stu-id="bed56-131">Now, the app should look very different.</span></span>

    ![Una captura de pantalla de la Página principal rediseñada](./images/create-app-01.png)
