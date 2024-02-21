Preguntar dps:
- Por qué haemos migraciones


Despues de hacer el db:seed hacemos una migracion en consola, queremos agregar una columna con el chef al hash de los restaurants, entonces tenemos que migrar

rails generate migration AddChefNameToRestaurants chef_name:string
rails db:migrate

Una vez modificamos digamos los chef y demas, para que se vea reflejado tenemos que hacer rails db:seed de nuevo en consola.

Tener cuidado, una vez que hacemos el db:seed, los id no van a seguir contando desde 1, sino que los vamos a tener que chequear en el objeto.

Ahora queremos mostrar X chef esta en X restaurantes

entonces vamos a routes de nuevo, y tenemos que crear la ruta para eso

get "restaurants"/:id/chef to_ "restaurants#chef"

Pero ahora vamos a usar member en vez de collection. Collection es cuando no necesitamos id, member es cuando necesitamos id.

En este caso necesitamos id pq queremos mostrar al chef para x restaurante, entonces hacemos

member do
  get :chef
end

ahora tenemos que hacer el controller, pero para hacerlo, tenemos que agregar el key del chef dentro del array de "before action" (PREGINTAR POR QUE DE NUEVO DPS)

entones hacemos el controlador

def chef
  @chef_name = @restaurant.chef_name
end

Y ahora tenemos que hacer el view en el show primero y agregarlo a lo que sea que hayan hecho para linkear todo que no entendi ni mierda (PARTIAL) (PREGUNTAR DE VUELTA)

Y finalmente creamos un view para el chef

<h1><%= @restaurant.name %>'s Chef</h1>
<p><%= @chef_name %></p>
<%= link_to "Back to restaurant", restaurant_path(@restaurant) %>

Ahora tenemos una pagina llena de restaurantes, pero lo que queremos es el usuario que sea posible dejar una review. Voy a tener un boton que nos va a habilitar ir a un form y dejar un comentario.

Vamos al data base editor (el que usabamos con emilia) y creamos una database table donde guardamos las reviews.

Vamos a estar guardando el restaurand id en reviews. (hay foto del esquema).

Vamos a la consola y hacemos 
rails g model review content restaurant:references

si vamos al archivo en migrate tenemos que ver como vimos en clase

rails db:migrate
si vamos al schema tenemos que tener cuidado con como nombramos el restaurant id cuando creamos la tabla

entonces tenemos que crear el modelo (PREGUNTAR QUÉ SERIA ESTO, creo que estas definiendo como que el id de review depende del id del restaurante)

class Review < ApplicationRecord
  has_many :reviews, dependent: :destroy
end

y tenemos que modificar el la clase Restaurant

class Restaurant < ApplicationRecord
  has_many :reviews, dependent: :destroy
end

y dps en la consola podemos crear como creamos en la clase pasada un restaurante (creo que era Review.create y los keys de la tabla)

Entonces tenemos que hacer el route para la review
get "/restaurants/:id/reviews/new", to: "reviews#new" --> esta seria la manera convencional

Rails.application.routes.draw do          --> tenemos 7 distintos (PREGUNTAR)
  resources :restaurants do
    resources :reviews, only: [:new, :create] (se le puede agregar antes del array %i)
  end
end

entonces en consola
rails g controller reviews new

entonces tenemos que hacer el controller para reviews

class ReviewsController < ApplicationController
  def new
    @restaurant = Restaurant.find(params[:restaurant_id])
    @review = Review.new
  end
end

y ahora hay que crear el view para new dentro de la carpeta reviews

<h1>New review</h1>
<%= simple_form_for [@restaurant, @review] do |f| %>
  <%= f.input :content %>
  <%= f.submit "Submit review", class: "btn btn-primary" %>
<% end %>

entonces volvemos al ReviewsController

def create
  @review = Review.new(review_params)  --> se asocia la validacion que hacemos en private
  @review.restaurant = @restaurant
  @review.save
  redirect_to restaurant_path(@restaurant)
end


private

def set_restaurant
  @restaurant = Restaurant.find(params[:restaurant_id]) --> esto es para dps en el controller de reviews agregar en before_action (PREGUNTAR)
end

def review_params
  params.require(:review), permit(:content)
end

Entonces tenemos que ir al erb del show y hacer el link para ir a ver los reviews (ver 4ta foto, fue imposible seguirle el paso)

y ahora tenemos que hacer el view

<p><strong>Reviews</strong></p>
<ul class="list-group">
  <% @restaurant.reviews.each do |review| %>
    <li class="list-group-item"><%= review.content %></li>
  <% end %>
</ul>
<br>


En el modelo se pueden agregar validaciones

class Review < ApplicationRecord
  belongs_to :restaurant

  validates :content, presence: true
end


entonces en el reviews controller

def create
  @review = Review.new(review_params)
  @review.restaurant = @restaurant
  if @review.save
    redirect_to restaurant_path(@restaurant)
  else
    render :new, status: :unprocessable_entity      --> corresponde a la validacion de arriba
  end
  @review.save
  redirect_to restaurant_path(@restaurant)
end

Y ahora para borrar una review.
Va a ser un poco diferente pq queremos "nest", agrupar, , por lo que si la review existe no la tengo que estar borrando de

PREGUNTAR entonces que es que algo este "nested" pq no entendi

Entonces vamos a routes

Rails.application.routes.draw do
  resources :restaurants do
    resources :reviews, only: [:new, :create] --> lo tenemos que poner abajo del resources que ya teniamos pq no lo queremos nested al resource original
  end
  resources :reviews, only: [:destroy]
end

entonces en ReviewsController

def destroy
  review = Review.find(params[:id])
  review.destroy
  redirect_to restaurant_path(review.restaurant) --> en las slides tiene @review, pero 
end

y ahora el view en el show erb  --> obviamente la mina hizo el render, asi que debería ser diferente lcdlm
<li class="list-group-item">
  <%= review.content %>
  <%= link_to "Delete", review_path(review), data: {turbo_method: :delete, turbo_confirm: "Are you sure?"}
  %>
</li>

