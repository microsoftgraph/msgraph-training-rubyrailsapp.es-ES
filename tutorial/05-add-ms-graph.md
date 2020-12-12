<!-- markdownlint-disable MD002 MD041 -->

<span data-ttu-id="b99f1-101">En este ejercicio, incorporará Microsoft Graph a la aplicación.</span><span class="sxs-lookup"><span data-stu-id="b99f1-101">In this exercise you will incorporate the Microsoft Graph into the application.</span></span> <span data-ttu-id="b99f1-102">Para esta aplicación, usará la GEM [httparty](https://github.com/jnunemaker/httparty) para realizar llamadas a Microsoft Graph.</span><span class="sxs-lookup"><span data-stu-id="b99f1-102">For this application, you will use the [httparty](https://github.com/jnunemaker/httparty) gem to make calls to Microsoft Graph.</span></span>

## <a name="create-a-graph-helper"></a><span data-ttu-id="b99f1-103">Crear una aplicación auxiliar de gráfico</span><span class="sxs-lookup"><span data-stu-id="b99f1-103">Create a Graph helper</span></span>

1. <span data-ttu-id="b99f1-104">Cree una aplicación auxiliar para administrar todas las llamadas a la API.</span><span class="sxs-lookup"><span data-stu-id="b99f1-104">Create a helper to manage all of your API calls.</span></span> <span data-ttu-id="b99f1-105">Ejecute el siguiente comando en su CLI para generar la aplicación auxiliar.</span><span class="sxs-lookup"><span data-stu-id="b99f1-105">Run the following command in your CLI to generate the helper.</span></span>

    ```Shell
    rails generate helper Graph
    ```

1. <span data-ttu-id="b99f1-106">Abra **./app/helpers/graph_helper. RB** y reemplace el contenido por lo siguiente.</span><span class="sxs-lookup"><span data-stu-id="b99f1-106">Open **./app/helpers/graph_helper.rb** and replace the contents with the following.</span></span>

    ```ruby
    require 'httparty'

    # Graph API helper methods
    module GraphHelper
      GRAPH_HOST = 'https://graph.microsoft.com'.freeze

      def make_api_call(method, endpoint, token, headers = nil, params = nil, payload = nil)
        headers ||= {}
        headers[:Authorization] = "Bearer #{token}"
        headers[:Accept] = 'application/json'

        params ||= {}

        case method.upcase
        when 'GET'
          HTTParty.get "#{GRAPH_HOST}#{endpoint}",
                       :headers => headers,
                       :query => params
        when 'POST'
          headers['Content-Type'] = 'application/json'
          HTTParty.post "#{GRAPH_HOST}#{endpoint}",
                        :headers => headers,
                        :query => params,
                        :body => payload ? payload.to_json : nil
        else
          raise "HTTP method #{method.upcase} not implemented"
        end
      end
    end
    ```

<span data-ttu-id="b99f1-107">Tómese un momento para revisar lo que hace el código.</span><span class="sxs-lookup"><span data-stu-id="b99f1-107">Take a moment to review what this code does.</span></span> <span data-ttu-id="b99f1-108">Realiza una solicitud GET o POST sencilla a través de la `httparty` GEM para el punto de conexión solicitado.</span><span class="sxs-lookup"><span data-stu-id="b99f1-108">It makes a simple GET or POST request via the `httparty` gem to the requested endpoint.</span></span> <span data-ttu-id="b99f1-109">Envía el token de acceso en el `Authorization` encabezado e incluye los parámetros de consulta que se pasan.</span><span class="sxs-lookup"><span data-stu-id="b99f1-109">It sends the access token in the `Authorization` header, and it includes any query parameters that are passed.</span></span>

<span data-ttu-id="b99f1-110">Por ejemplo, para usar el `make_api_call` método para realizar una llamada a get `https://graph.microsoft.com/v1.0/me?$select=displayName` , puede llamarlo de esta manera:</span><span class="sxs-lookup"><span data-stu-id="b99f1-110">For example, to use the `make_api_call` method to do a GET to `https://graph.microsoft.com/v1.0/me?$select=displayName`, you could call it like so:</span></span>

```ruby
make_api_call 'GET', '/v1.0/me', access_token, { '$select': 'displayName' }
```

<span data-ttu-id="b99f1-111">A partir de entonces, podrá crear más adelante a medida que implementa más características de Microsoft Graph en la aplicación.</span><span class="sxs-lookup"><span data-stu-id="b99f1-111">You'll build on this later as you implement more Microsoft Graph features into the app.</span></span>

## <a name="get-calendar-events-from-outlook"></a><span data-ttu-id="b99f1-112">Obtener eventos de calendario de Outlook</span><span class="sxs-lookup"><span data-stu-id="b99f1-112">Get calendar events from Outlook</span></span>

1. <span data-ttu-id="b99f1-113">En la CLI, ejecute el siguiente comando para agregar un nuevo controlador.</span><span class="sxs-lookup"><span data-stu-id="b99f1-113">In your CLI, run the following command to add a new controller.</span></span>

    ```Shell
    rails generate controller Calendar index new
    ```

1. <span data-ttu-id="b99f1-114">Agregue la nueva ruta a **./config/Routes.RB**.</span><span class="sxs-lookup"><span data-stu-id="b99f1-114">Add the new route to **./config/routes.rb**.</span></span>

    ```ruby
    get 'calendar', :to => 'calendar#index'
    ```

1. <span data-ttu-id="b99f1-115">Agregue un nuevo método a la aplicación auxiliar de Graph para [obtener una vista de calendario](https://docs.microsoft.com/graph/api/calendar-list-calendarview?view=graph-rest-1.0).</span><span class="sxs-lookup"><span data-stu-id="b99f1-115">Add a new method to the Graph helper to [get a calendar view](https://docs.microsoft.com/graph/api/calendar-list-calendarview?view=graph-rest-1.0).</span></span> <span data-ttu-id="b99f1-116">Abra **./app/helpers/graph_helper. RB** y agregue el siguiente método al `GraphHelper` módulo.</span><span class="sxs-lookup"><span data-stu-id="b99f1-116">Open **./app/helpers/graph_helper.rb** and add the following method to the `GraphHelper` module.</span></span>

    :::code language="ruby" source="../demo/graph-tutorial/app/helpers/graph_helper.rb" id="GetCalendarSnippet":::

    <span data-ttu-id="b99f1-117">Tenga en cuenta lo que está haciendo este código.</span><span class="sxs-lookup"><span data-stu-id="b99f1-117">Consider what this code is doing.</span></span>

    - <span data-ttu-id="b99f1-118">La dirección URL a la que se llamará es `/v1.0/me/calendarview` .</span><span class="sxs-lookup"><span data-stu-id="b99f1-118">The URL that will be called is `/v1.0/me/calendarview`.</span></span>
        - <span data-ttu-id="b99f1-119">El `Prefer: outlook.timezone` encabezado hace que las horas de inicio y finalización de los resultados se ajusten a la zona horaria del usuario.</span><span class="sxs-lookup"><span data-stu-id="b99f1-119">The `Prefer: outlook.timezone` header causes the start and end times in the results to be adjusted to the user's time zone.</span></span>
        - <span data-ttu-id="b99f1-120">Los `startDateTime` `endDateTime` parámetros y establecen el inicio y el final de la vista.</span><span class="sxs-lookup"><span data-stu-id="b99f1-120">The `startDateTime` and `endDateTime` parameters set the start and end of the view.</span></span>
        - <span data-ttu-id="b99f1-121">El `$select` parámetro limita los campos devueltos para cada evento a solo aquellos que la vista usará realmente.</span><span class="sxs-lookup"><span data-stu-id="b99f1-121">The `$select` parameter limits the fields returned for each events to just those the view will actually use.</span></span>
        - <span data-ttu-id="b99f1-122">El `$orderby` parámetro ordena los resultados por hora de inicio.</span><span class="sxs-lookup"><span data-stu-id="b99f1-122">The `$orderby` parameter sorts the results by start time.</span></span>
        - <span data-ttu-id="b99f1-123">El `$top` parámetro limita los resultados a 50 eventos.</span><span class="sxs-lookup"><span data-stu-id="b99f1-123">The `$top` parameter limits the results to 50 events.</span></span>
    - <span data-ttu-id="b99f1-124">Para una respuesta correcta, devuelve la matriz de elementos contenidos en la `value` clave.</span><span class="sxs-lookup"><span data-stu-id="b99f1-124">For a successful response, it returns the array of items contained in the `value` key.</span></span>

1. <span data-ttu-id="b99f1-125">Agregue un nuevo método a la aplicación auxiliar de Graph para buscar un [identificador de zona horaria de IANA](https://www.iana.org/time-zones) basado en un nombre de zona horaria de Windows.</span><span class="sxs-lookup"><span data-stu-id="b99f1-125">Add a new method to the Graph helper to lookup an [IANA time zone identifier](https://www.iana.org/time-zones) based on a Windows time zone name.</span></span> <span data-ttu-id="b99f1-126">Esto es necesario porque Microsoft Graph puede devolver zonas horarias como nombres de zona horaria de Windows y la clase **DateTime** Ruby requiere identificadores de zona horaria de IANA.</span><span class="sxs-lookup"><span data-stu-id="b99f1-126">This is necessary because Microsoft Graph can return time zones as Windows time zone names, and the Ruby **DateTime** class requires IANA time zone identifiers.</span></span>

    :::code language="ruby" source="../demo/graph-tutorial/app/helpers/graph_helper.rb" id="ZoneMappingSnippet":::

1. <span data-ttu-id="b99f1-127">Abra **./app/controllers/calendar_controller. RB** y reemplace todo el contenido por lo siguiente.</span><span class="sxs-lookup"><span data-stu-id="b99f1-127">Open **./app/controllers/calendar_controller.rb** and replace its entire contents with the following.</span></span>

    ```ruby
    # Calendar controller
    class CalendarController < ApplicationController
      include GraphHelper

      def index
        # Get the IANA identifier of the user's time zone
        time_zone = get_iana_from_windows(user_timezone)

        # Calculate the start and end of week in the user's time zone
        start_datetime = Date.today.beginning_of_week(:sunday).in_time_zone(time_zone).to_time
        end_datetime = start_datetime.advance(:days => 7)

        @events = get_calendar_view access_token, start_datetime, end_datetime, user_timezone || []
        render json: @events
      rescue RuntimeError => e
        @errors = [
          {
            :message => 'Microsoft Graph returned an error getting events.',
            :debug => e
          }
        ]
      end
    end
    ```

1. <span data-ttu-id="b99f1-128">Reinicie el servidor.</span><span class="sxs-lookup"><span data-stu-id="b99f1-128">Restart the server.</span></span> <span data-ttu-id="b99f1-129">Inicie sesión y haga clic en el vínculo de **calendario** en la barra de navegación.</span><span class="sxs-lookup"><span data-stu-id="b99f1-129">Sign in and click the **Calendar** link in the nav bar.</span></span> <span data-ttu-id="b99f1-130">Si todo funciona, debería ver un volcado JSON de eventos en el calendario del usuario.</span><span class="sxs-lookup"><span data-stu-id="b99f1-130">If everything works, you should see a JSON dump of events on the user's calendar.</span></span>

## <a name="display-the-results"></a><span data-ttu-id="b99f1-131">Mostrar los resultados</span><span class="sxs-lookup"><span data-stu-id="b99f1-131">Display the results</span></span>

<span data-ttu-id="b99f1-132">Ahora puede Agregar HTML para mostrar los resultados de forma más fácil de uso.</span><span class="sxs-lookup"><span data-stu-id="b99f1-132">Now you can add HTML to display the results in a more user-friendly manner.</span></span>

1. <span data-ttu-id="b99f1-133">Abra **./app/views/calendar/index.html. Erb** y reemplace su contenido por lo siguiente.</span><span class="sxs-lookup"><span data-stu-id="b99f1-133">Open **./app/views/calendar/index.html.erb** and replace its contents with the following.</span></span>

    :::code language="html" source="../demo/graph-tutorial/app/views/calendar/index.html.erb" id="CalendarSnippet":::

    <span data-ttu-id="b99f1-134">Se recorrerá en bucle una colección de eventos y se agregará una fila de tabla para cada uno.</span><span class="sxs-lookup"><span data-stu-id="b99f1-134">That will loop through a collection of events and add a table row for each one.</span></span>

1. <span data-ttu-id="b99f1-135">Quite la `render json: @events` línea de la `index` acción en **./app/Controllers/calendar_controller. RB**.</span><span class="sxs-lookup"><span data-stu-id="b99f1-135">Remove the `render json: @events` line from the `index` action in **./app/controllers/calendar_controller.rb**.</span></span>

1. <span data-ttu-id="b99f1-136">Actualice la página y la aplicación ahora debería representar una tabla de eventos.</span><span class="sxs-lookup"><span data-stu-id="b99f1-136">Refresh the page and the app should now render a table of events.</span></span>

    ![Captura de pantalla de la tabla de eventos](./images/add-msgraph-01.png)
