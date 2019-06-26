<!-- markdownlint-disable MD002 MD041 -->

<span data-ttu-id="51780-101">En este ejercicio, incorporará Microsoft Graph a la aplicación.</span><span class="sxs-lookup"><span data-stu-id="51780-101">In this exercise you will incorporate the Microsoft Graph into the application.</span></span> <span data-ttu-id="51780-102">Para esta aplicación, usará la GEM [httparty](https://github.com/jnunemaker/httparty) para realizar llamadas a Microsoft Graph.</span><span class="sxs-lookup"><span data-stu-id="51780-102">For this application, you will use the [httparty](https://github.com/jnunemaker/httparty) gem to make calls to Microsoft Graph.</span></span>

## <a name="create-a-graph-helper"></a><span data-ttu-id="51780-103">Crear una aplicación auxiliar de gráfico</span><span class="sxs-lookup"><span data-stu-id="51780-103">Create a Graph helper</span></span>

<span data-ttu-id="51780-104">Cree una aplicación auxiliar para administrar todas las llamadas a la API.</span><span class="sxs-lookup"><span data-stu-id="51780-104">Create a helper to manage all of your API calls.</span></span> <span data-ttu-id="51780-105">Ejecute el siguiente comando en su CLI para generar la aplicación auxiliar.</span><span class="sxs-lookup"><span data-stu-id="51780-105">Run the following command in your CLI to generate the helper.</span></span>

```Shell
rails generate helper Graph
```

<span data-ttu-id="51780-106">Abra el archivo que `./app/helpers/graph_helper.rb` acaba de crear y reemplace el contenido por lo siguiente.</span><span class="sxs-lookup"><span data-stu-id="51780-106">Open the newly created `./app/helpers/graph_helper.rb` file and replace the contents with the following.</span></span>

```ruby
require 'httparty'

# Graph API helper methods
module GraphHelper
  GRAPH_HOST = 'https://graph.microsoft.com'.freeze

  def make_api_call(endpoint, token, params = nil)
    headers = {
      Authorization: "Bearer #{token}"
    }

    query = params || {}

    HTTParty.get "#{GRAPH_HOST}#{endpoint}",
                 headers: headers,
                 query: query
  end
end
```

<span data-ttu-id="51780-107">Tómese un momento para revisar lo que hace el código.</span><span class="sxs-lookup"><span data-stu-id="51780-107">Take a moment to review what this code does.</span></span> <span data-ttu-id="51780-108">Realiza una sencilla solicitud GET a través del `httparty` GEM al punto de conexión solicitado.</span><span class="sxs-lookup"><span data-stu-id="51780-108">It makes a simple GET request via the `httparty` gem to the requested endpoint.</span></span> <span data-ttu-id="51780-109">Envía el token de acceso en el `Authorization` encabezado e incluye los parámetros de consulta que se pasan.</span><span class="sxs-lookup"><span data-stu-id="51780-109">It sends the access token in the `Authorization` header, and it includes any query parameters that are passed.</span></span>

<span data-ttu-id="51780-110">Por ejemplo, para usar el `make_api_call` método para realizar una llamada a `https://graph.microsoft.com/v1.0/me?$select=displayName`get, puede llamarlo de esta manera:</span><span class="sxs-lookup"><span data-stu-id="51780-110">For example, to use the `make_api_call` method to do a GET to `https://graph.microsoft.com/v1.0/me?$select=displayName`, you could call it like so:</span></span>

```ruby
make_api_call `/v1.0/me`, access_token, { '$select': 'displayName' }
```

<span data-ttu-id="51780-111">A partir de entonces, podrá crear más adelante a medida que implementa más características de Microsoft Graph en la aplicación.</span><span class="sxs-lookup"><span data-stu-id="51780-111">You'll build on this later as you implement more Microsoft Graph features into the app.</span></span>

## <a name="get-calendar-events-from-outlook"></a><span data-ttu-id="51780-112">Obtener eventos de calendario de Outlook</span><span class="sxs-lookup"><span data-stu-id="51780-112">Get calendar events from Outlook</span></span>

<span data-ttu-id="51780-113">Comencemos agregando la posibilidad de ver los eventos en el calendario del usuario.</span><span class="sxs-lookup"><span data-stu-id="51780-113">Let's start by adding the ability to view events on the user's calendar.</span></span> <span data-ttu-id="51780-114">En la CLI, ejecute el siguiente comando para agregar un nuevo controlador.</span><span class="sxs-lookup"><span data-stu-id="51780-114">In your CLI, run the following command to add a new controller.</span></span>

```Shell
rails generate controller Calendar index
```

<span data-ttu-id="51780-115">Ahora que la ruta está disponible, actualice el vínculo de **calendario** de la barra de `./app/view/layouts/application.html.erb` exploración en para usarla.</span><span class="sxs-lookup"><span data-stu-id="51780-115">Now that we have the route available, update the **Calendar** link in the navbar in `./app/view/layouts/application.html.erb` to use it.</span></span> <span data-ttu-id="51780-116">Reemplace la línea `<a class="nav-link" href="#">Calendar</a>` por lo siguiente.</span><span class="sxs-lookup"><span data-stu-id="51780-116">Replace the line `<a class="nav-link" href="#">Calendar</a>` with the following.</span></span>

```html
<%= link_to "Calendar", {:controller => :calendar, :action => :index}, class: "nav-link#{' active' if controller.controller_name == 'calendar'}" %>
```

<span data-ttu-id="51780-117">Agregue un nuevo método a la aplicación auxiliar de Graph para [Mostrar los eventos del usuario](https://developer.microsoft.com/en-us/graph/docs/api-reference/v1.0/api/user_list_events).</span><span class="sxs-lookup"><span data-stu-id="51780-117">Add a new method to the Graph helper to [list the user's events](https://developer.microsoft.com/en-us/graph/docs/api-reference/v1.0/api/user_list_events).</span></span> <span data-ttu-id="51780-118">Abra `./app/helpers/graph_helper.rb` y agregue el siguiente método al `GraphHelper` módulo.</span><span class="sxs-lookup"><span data-stu-id="51780-118">Open `./app/helpers/graph_helper.rb` and add the following method to the `GraphHelper` module.</span></span>

```ruby
def get_calendar_events(token)
  get_events_url = '/v1.0/me/events'

  query = {
    '$select': 'subject,organizer,start,end',
    '$orderby': 'createdDateTime DESC'
  }

  response = make_api_call get_events_url, token, query

  raise response.parsed_response.to_s || "Request returned #{response.code}" unless response.code == 200
  response.parsed_response['value']
end
```

<span data-ttu-id="51780-119">Tenga en cuenta lo que está haciendo este código.</span><span class="sxs-lookup"><span data-stu-id="51780-119">Consider what this code is doing.</span></span>

- <span data-ttu-id="51780-120">La dirección URL a la que se `/v1.0/me/events`llamará es.</span><span class="sxs-lookup"><span data-stu-id="51780-120">The URL that will be called is `/v1.0/me/events`.</span></span>
- <span data-ttu-id="51780-121">El `$select` parámetro limita los campos devueltos para cada evento a solo aquellos que la vista usará realmente.</span><span class="sxs-lookup"><span data-stu-id="51780-121">The `$select` parameter limits the fields returned for each events to just those our view will actually use.</span></span>
- <span data-ttu-id="51780-122">El `$orderby` parámetro ordena los resultados por la fecha y hora en que se crearon, con el elemento más reciente en primer lugar.</span><span class="sxs-lookup"><span data-stu-id="51780-122">The `$orderby` parameter sorts the results by the date and time they were created, with the most recent item being first.</span></span>
- <span data-ttu-id="51780-123">Para una respuesta correcta, devuelve la matriz de elementos contenidos en la `value` clave.</span><span class="sxs-lookup"><span data-stu-id="51780-123">For a successful response, it returns the array of items contained in the `value` key.</span></span>

<span data-ttu-id="51780-124">Ahora puede probar esto.</span><span class="sxs-lookup"><span data-stu-id="51780-124">Now you can test this.</span></span> <span data-ttu-id="51780-125">Abra `./app/controllers/calendar_controller.rb` y actualice la `index` acción para llamar a este método y representar los resultados.</span><span class="sxs-lookup"><span data-stu-id="51780-125">Open `./app/controllers/calendar_controller.rb` and update the `index` action to call this method and render the results.</span></span>

```ruby
# Calendar controller
class CalendarController < ApplicationController
  include GraphHelper

  def index
    @events = get_calendar_events access_token || []
    render json: @events
  rescue RuntimeError => e
    @errors = [
      {
        message: 'Microsoft Graph returned an error getting events.',
        debug: e
      }
    ]
  end
end
```

<span data-ttu-id="51780-126">Reinicie el servidor.</span><span class="sxs-lookup"><span data-stu-id="51780-126">Restart the server.</span></span> <span data-ttu-id="51780-127">Inicie sesión y haga clic en el vínculo de **calendario** en la barra de navegación.</span><span class="sxs-lookup"><span data-stu-id="51780-127">Sign in and click the **Calendar** link in the nav bar.</span></span> <span data-ttu-id="51780-128">Si todo funciona, debería ver un volcado JSON de eventos en el calendario del usuario.</span><span class="sxs-lookup"><span data-stu-id="51780-128">If everything works, you should see a JSON dump of events on the user's calendar.</span></span>

## <a name="display-the-results"></a><span data-ttu-id="51780-129">Mostrar los resultados</span><span class="sxs-lookup"><span data-stu-id="51780-129">Display the results</span></span>

<span data-ttu-id="51780-130">Ahora puede Agregar HTML y CSS para mostrar los resultados de forma más fácil de uso.</span><span class="sxs-lookup"><span data-stu-id="51780-130">Now you can add HTML and CSS to display the results in a more user-friendly manner.</span></span>

<span data-ttu-id="51780-131">Abra `./app/views/calendar/index.html.erb` y reemplace su contenido por lo siguiente.</span><span class="sxs-lookup"><span data-stu-id="51780-131">Open `./app/views/calendar/index.html.erb` and replace its contents with the following.</span></span>

```html
<h1>Calendar</h1>
<table class="table">
  <thead>
    <tr>
      <th scope="col">Organizer</th>
      <th scope="col">Subject</th>
      <th scope="col">Start</th>
      <th scope="col">End</th>
    </tr>
  </thead>
  <tbody>
    <% @events.each do |event| %>
      <tr>
        <td><%= event['organizer']['emailAddress']['name'] %></td>
        <td><%= event['subject'] %></td>
        <td><%= event['start']['dateTime'].to_time(:utc).localtime.strftime('%-m/%-d/%y %l:%M %p') %></td>
        <td><%= event['end']['dateTime'].to_time(:utc).localtime.strftime('%-m/%-d/%y %l:%M %p') %></td>
      </tr>
    <% end %>
  </tbody>
</table>
```

<span data-ttu-id="51780-132">Se recorrerá en bucle una colección de eventos y se agregará una fila de tabla para cada uno.</span><span class="sxs-lookup"><span data-stu-id="51780-132">That will loop through a collection of events and add a table row for each one.</span></span> <span data-ttu-id="51780-133">Quite la `render json: @events` línea de la `index` acción en `./app/controllers/calendar_controller.rb` y ahora la aplicación debe representar una tabla de eventos.</span><span class="sxs-lookup"><span data-stu-id="51780-133">Remove the `render json: @events` line from the `index` action in `./app/controllers/calendar_controller.rb` and the app should now render a table of events.</span></span>

![Captura de pantalla de la tabla de eventos](./images/add-msgraph-01.png)
