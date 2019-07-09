# practica-rails-jobs

Primero crear un proyecto nuevo.

```
rails new expo
```

Después nos dirigimos al Gemfile y añadimos la gema de delayed job.

```
gem 'delayed_job_active_record'
```
Instalamos las gemas del Gemfile

```
bundle install
```
Creamos un delayed job en el proyecto.

```
rails generate delayed_job:active_record
```

Corremos las migraciones.

```
rails db:migrate
```

En config/application.rb indicamos a rails el tipo de cola al que se mandan los jobs.

```
config.active_job.queue_adapter = :delayed_job
```

```
config.action_mailer.default_url_options = {host: 'example.com'}
```

Generamos un modelo y controlador de una persona con scaffold.

```
rails g scaffold person name:string email:string
```
Corremos las migraciones creadas con el scaffold

```
rails db:migrate
```
Ahora creamos el scaffold del mailer.

```
rails g mailer welcome notify
```
Ahora necesitamos dar unos parámetros para el envío, en app/mailers/welcome_mailer.rb

```
   class WelcomeMailer < ApplicationMailer
      def notify(person)
        @person = person
        mail to: person.email, subject: "Welcome user"
      end
   end
```

Configurar la plantilla de html del contenido del correo, dentro de app/views/layout/mailer.text.erb

```
   <html>
      <body>
        <h1>Welcome to our app!</h1>
        <%= yield %>
      </body>
   </html>
```

Se modifica el html del mailer, en app/views/layout/welcome_mailer/notify/html.erb

```
  <p>
      Hello <%= @person.name %>
  </p>
```

Se crea un preview dal mail, dentro de test/mailers/previews/welcome_mailer_preview.rb

```
   class WelcomeMailerPreview < ApplicationMailer:Preview
      def notify
        WelcomeMailer.notify Person.new(name: 'Sample user', email: 'sample@mail.com')
      end
   end
```
A continuación modificamos el controller cuando crea un usuario

```
   def create
      @person = Person.new(person_params)
        
      respond_to |format|
      
      if @person.save
        WelcomeMailer.notify(@person).deliver_now
        format.html {redirect_to @person, notice:'Person was successfully created.' }
        format.json {render :show, status: :created, location: @person}
    ...
```

Iniciar el servidor, crear un usuario y comprobar en la terminal el envío.

Ahora, en config/application.rb

```
config.action_mailer.delivery_mehotd = :smtp
```
En app/controller/people_controller.rb

```
if @person.save
      WelcomeMailer.notify(@person).deliver_later!
```
Comprobar el funcionamiento creando usuario, y liberación de ejución de los
jobs que están en delayed_jobs

```
rails jobs:workoff
```



