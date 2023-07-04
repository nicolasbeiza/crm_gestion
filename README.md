# pg search
# Agregar al modelo la sgte. línea
# include PgSearch::Model

#  pg_search_scope :search_full_text, against: {
#  title: 'A',
#  subtitle: 'B',
#  content: 'C'
#  }
# Se agregan en models

# ActiveStorage
# Agregar al modelo la siguiente linea:
# has_one_attached :avatar
# has_many_attached :images (uno o mas) (recomendable usar esta opcion)
# se agrega uno de ellos 

# Agregar al HTML
# <%= form.file_field :images, multiple: true %><br> (hace referencia a linea 14)

Apuntes para prueba:
ssh -p 2222 inforcap@127.0.0.1 —> para conectarse a la maquina virtual desde visual studio
sudo systemctl start graphical.target —> en virtual box, para ver el entorno virtual
Sudo systemctl set-default graphical —> establece el entorno gráfico siempre
Rails new (nombreDelProyecto) -d postgresql —> crea un nuevo proyecto con db Postgres
Rails db:create -> crea una base de datos
Rails db —> muestra el estado
\q —> sale del entorno base de datos
rails s —>despliega el servidor

bunde add bootstrap
bundle add jquery-rails
bundle add popper

Cambiar formato a archivo app>assets>stylesheets>application.css a -> .scss y luego agregar en la ultima fila: @import “bootstrap”;
@import “popper”;
Recordatorio: revisar version de bootstrap en gemfile, y usar la misma desde la pagina web de bootstrap

En app>assets>javascript>application.js y luego agregar en la ultima fila:
Import “bootstrap”
Import “popper”

En config>initializers>assets.rb y pegar lo siguiente:
Rails.application.config.assets.precompile += %w( application.scss bootstrap.min.js popper.js )

En app>assets>views> Agregar carpeta con nombre “shared”, luego en views>shared agregar archivo _nabvar.html.erb ,  y luego pegar código desde la pagina de bootstrap. Esto es para renderizar.
Se hace lo mismo con _footer.html.erb

En app>assets>views>layouts>application.html.erb pegar entre <%= yield %> :
<%= render "/shared/navbar"%> (Arriba de yield)
<%= render "/shared/footer"%> (Abajo de yield)
Yield debe estar dentro de un <div></div> para hacerlo responsivo.

En config>importmap.rb agregar:
pin "bootstrap", to: 'bootstrap.min.js', preload: true
pin "popper", to: 'popper.js', preload: true 

****Crear el controlador****
 
1) rails g controller welcome index -> 
        Genera:
        a) Ruta controlado con el nombre welcome
        b) Vista de nombre welcome dentro de una carpeta del mismo nombre (welcome)
        c) Controlador y dentro accion o metodo dentro del controller.

2) En config/routes.rb
    (Estará la ruta creada por el controlador de welcome get 'welcome/index')
   - Modificamos el root, descomentamos y colocamos el welcome root "welcome#index"
    con esto dejamos como raiz.
    rails routes —> muestra las rutas creadas

Controlador>rutas>vistas en ese sentido

Agregamos gema Devise (login user): bundle add devise
rails g devise:install
rails generate devise User
rails db:create db:migrate

* mostrar vistas de devise
rails g devise:views

* mostrar los controllers de devise(user) 
rails g devise:controllers users

En config>initializer>devise.rb descomentar la linea 266

Agregamos gema figaro:
bundle add figaro
bundle exec figaro install

Agregamos gema faker:
bundle add faker
En db>seeds.rb en la linea 8 escribir:
require “faker”

Agregamos gema paga:
bundle add pagy
Incluir en controller>application_controller.rb en la linea 2: include Pagy::Backend
Pagy::DEFAULT[:items] = 10  #items per page

Incluir en helper>application_helper.rb en la linea 2:
include Pagy::Frontend

Agregamos gema pg_search:
bundle add pg_search

Agregamos gema activestorage
bundle add activestorage
rails active_storage:install
rails db:migrate
(Ver archivo README)
—hasta acá el respaldo .zip—

rails g scaffold nombre_de_la_tabla tabla1 tabla2 tablan
rails g scaffold vendedor nombre email (CRUD completo)
rails g model manager name  (model crea el modelo y la migración)
rails g model oficina name

Ahora relacionamos los datos
rails g migration AddColumnToVendedors Manager:references Oficina:refeerences (agregamos 2 columnas FK para que se relaciones con las tablas) 
rails db:migrate
rails db:migrate:status —> muestra el estatus de las migraciones

la migraciones de la creación de las tablas tienen que estar ejecutadas

En model>concerns oficina.rb (1 oficina tiene muchos vendedores)
Escribimos: has_many :vendedors
En manager.rb (1 manager tiene muchos vendedores)
Escribimos has_many :vendors
En vendedor.rb (1 vendedor tiene 1 oficina) (y 1 manager)
Escribimos:
belongs_to :manager
belongs_to :oficina


rails g migration AddColumnsToVendedors Manager:references Oficina:references (si falla)
rails g migration AddOficinasToVendedors oficina:references (si falla)

Agregar selectores en view>vendedors>_form.html.erb 
Abajo de email agregamos un div
<div>
  <select name= "vendedor[oficina_id]" id="vendedor_oficina_id">
    <option value="-1" selected >Seleccione una oficina</option>
    <% @oficinas.each do |oficina| %>
        <option value="<%= oficina.id %>"> <%= oficina.id %> - <%= oficina.name %> </option>
      <% end %>
    </select>
  </div>

  <div>
  <select name= "vendedor[manager_id]" id="vendedor_manager_id">
    <option value="-1" selected >Seleccione un manager</option>
    <% @manager.each do |manager| %>
        <option value="<%= manager.id %>"> <%= manager.id %> - <%= manager.name %> </option>
      <% end %>
    </select>
  </div>


En el controllers>vendedors_controller.rb agregamos en el método new (bajo @vendedor = Vendedor.new)
Escribimos:
@managers = Manager.all
@oficinas = Oficina.all

Agregamos al método vendedor_params (linea 70) 2 atributos más, queda:
def vendedor_params
    params.require(:vendedor).permit(:nombre, :email, :manager_id, :oficina_id)
end

En archivo seeds.rb, poblamos la bd con:
Oficina.create([{ name: "Oficina 1" }, { name: "Oficina 2" }, { name: "Oficina 3" }])
Manager.create([{ name: "Manager 1" }, { name: "Manager 2" }, { name: "Manager 3" }])

rails db:seed

— 
Agregar roles: rails g migration AddRoleToUsers role:integer
Agregar a la migración:
:integer, default: 0

En models>user.rb agregar abajo de devise:
enum :role, [:normal, :admin]

Generar una vista particular para el usuario que esté logeado: rails g controller home index

En controller>vendedors_controller.rb agregar bajo el primer before_action
before_action only: [ :show, :new, :create ] do
    authorize_request(["admin", "normal"])
end
before_action only: [ :edit, :update, :destroy ] do
    authorize_request(["admin"])
end

Esto da autorización y limitaciones a los roles

En application_controller.rb agregar al final:
def authorize_request(kind = nil)
    unless kind.include?(current_user.role)
    redirect_to posts_path, notice: "You are not authorized to perform this action"
    end
end
