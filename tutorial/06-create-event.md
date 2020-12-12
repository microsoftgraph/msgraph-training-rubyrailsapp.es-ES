<!-- markdownlint-disable MD002 MD041 -->

En esta sección, agregará la capacidad de crear eventos en el calendario del usuario.

1. Abra **./app/helpers/graph_helper. RB** y agregue el siguiente método a la clase **Graph** .

    :::code language="ruby" source="../demo/graph-tutorial/app/helpers/graph_helper.rb" id="CreateEventSnippet":::

1. Abra el **calendar_controller./app/Controllers/** y agregue la siguiente ruta a la clase **CalendarController**

    :::code language="ruby" source="../demo/graph-tutorial/app/controllers/calendar_controller.rb" id="CreateEventRouteSnippet":::

1. Abra **./config/Routes.RB** y agregue la nueva ruta.

    ```ruby
    post 'calendar/new', :to => 'calendar#create'
    ```

1. Abra **./app/views/calendar/new.html. Erb** y reemplace su contenido por lo siguiente.

    :::code language="html" source="../demo/graph-tutorial/app/views/calendar/new.html.erb" id="NewEventFormSnippet":::

1. Guarde los cambios y actualice la aplicación. En la página **calendario** , seleccione el botón **nuevo evento** . Rellene el formulario y seleccione **crear** para crear un nuevo evento.

    ![Captura de pantalla del nuevo formulario de eventos](images/create-event-01.png)
