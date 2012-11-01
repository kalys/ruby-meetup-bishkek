!SLIDE
# Что нового в Rails 4 #

!SLIDE bullets
# Компоненты и функционал, вынесенные в гемы #

* ActiveResource
* ActiveRecord observers
* ActiveRecord session store
* protected\_attributes (attr\_accessible, attr\_protected)

!SLIDE
## Hash based matchers
Прощай синтаксис Rails 2!

    @@@Ruby
    User.find(:all, ...)
    User.find(:first, ...)
    User.find(:last, ...)

!SLIDE bullets
* find\_all\_\* / find\_last\_\*
* Action caching
* Page caching


!SLIDE bullets
# Deprecations
* Rails 4.0: deprecated, вынесены в гемы, но подключены в Gemfile;
* Rails 4.1: не будет поддержки или подключаем гемы ручками;
* Rails 5.0: нет гемов.

!SLIDE bullets

# Ruby >= 1.9.3

<del>Ruby 1.8.7</del>

!SLIDE
# Что нового...

!SLIDE ruby32
# strong_parameters

    @@@Ruby
    # app/models/user.rb
    class User < ActiveRecord::Base; end

    # app/controllers/users_controller.rb
    class UsersController < ApplicationController
      def create
        @user = User.create! params[:user]
        redirect_to @user
      end
    end
    # исключение ActiveModel::MassAssignmentSecurity::Error

!SLIDE ruby4
# strong_parameters

    @@@Ruby
    # app/models/user.rb
    class User < ActiveRecord::Base; end

    # app/controllers/users_controller.rb
    class UsersController < ApplicationController
      def create
        @user = User.create! params[:user]
        redirect_to @user
      end
    end
    # исключение ActiveModel::ForbiddenAttributesError

!SLIDE ruby4
# strong_parameters

    @@@Ruby
    # app/models/user.rb
    class User < ActiveRecord::Base; end

    # app/controllers/users_controller.rb
    class UsersController < ApplicationController
      def create
        @user = User.create! params.require(:user).
            permit(:username, :password)
        redirect_to @user
      end
    end

!SLIDE ruby4
# strong_parameters

    @@@Ruby
    # app/models/user.rb
    class User < ActiveRecord::Base; end

    # app/controllers/users_controller.rb
    class UsersController < ApplicationController
      def create
        @user = User.create! user_params
        redirect_to @user
      end
    end

    private

    def user_params
      params.require(:user).permit(:username, :password)
    end

!SLIDE
## http://github.com/rails/strong_parameters

!SLIDE ruby32
# Декларативные etag'и

    @@@Ruby
    class TodolistsController < ApplicationController
      def show
        @todolist = Todolist.find(params[:id])
        fresh_when etag: @todolist
      end
    end

!SLIDE ruby32
# Декларативные etag'и

    @@@Ruby
    # ...
    etag = [@user, @posts]

    if stale?(:etag => etag)
      # дергаем тяжелые данные
      # ...
      respond_to do |format|
        format.html
      end
    end
    
!SLIDE ruby4
# Декларативные etag'и

    @@@Ruby
    class TodolistsController < ApplicationController

      etag { current_user.try :admin }

      def show
        @todolist = Todolist.find(params[:id])
        fresh_when etag: @todolist
      end
    end

!SLIDE ruby4
# Декларативные etag'и

    @@@Ruby
    class TodolistsController < ApplicationController

      etag { current_user.try :admin }
      etag { @project.try :cache_key }

      def show
        @todolist = Todolist.find(params[:id])
        fresh_when etag: @todolist
      end
    end

!SLIDE
## http://github.com/rails/etagger

!SLIDE ruby32
# cache\_digests
## Матрешки

    @@@HTML
    # projects/show.html.erb
    <% cache [ "v5", project ] do %>
      <p>All my todo lists:</p>
      <%= render project.todolists %>
    <% end %>

    # todolists/_todolist.html.erb
    <% cache [ "v3", todolist ] do %>
      <p><%= todolist.name %>:</p>
      <%= render todolist.todos %>
    <% end %>

    # todos/_todo.html.erb
    <% cache [ "v1", todo ] do %>
      <p><%= todo.name %></p>
    <% end %>

!SLIDE ruby32
# cache\_digests

    @@@HTML
    # projects/show.html.erb
    <% cache [ "v6", project ] do %>
      <p>All my todo lists:</p>
      <%= render project.todolists %>
    <% end %>

    # todolists/_todolist.html.erb
    <% cache [ "v4", todolist ] do %>
      <p><%= todolist.name %>:</p>
      <%= render todolist.todos %>
    <% end %>

    # todos/_todo.html.erb
    <% cache [ "v2", todo ] do %>
      <p><%= todo.name %></p>
    <% end %>

!SLIDE ruby4
# cache\_digests

    @@@HTML
    # projects/show.html.erb
    <% cache project do %>
      <p>All my todo lists:</p>
      <%= render project.todolists %>
    <% end %>

    # todolists/_todolist.html.erb
    <% cache todolist do %>
      <p><%= todolist.name %>:</p>
      <%= render todolist.todos %>
    <% end %>

    # todos/_todo.html.erb
    <% cache todo do %>
      <p><%= todo.name %></p>
    <% end %>

!SLIDE ruby4
# cache\_digests

    @@@HTML
    # projects/show.html.erb
    <% cache project do %>
      <p>All my todo lists:</p>
      <%= render project.todolists %>
    <% end %>

    # todolists/_todolist.html.erb
    <% cache todolist do %>
      <p><%= todolist.name %>:</p>
      <ul><%= render todolist.todos %></ul>
    <% end %>

    # todos/_todo.html.erb
    <% cache todo do %>
      <li><%= todo.name %></li>
    <% end %>

!SLIDE
## http://github.com/rails/cache_digests

!SLIDE bullets incremental
# Turbolinks
* Переход по ссылкам без загрузки всей страницы
* Не загружает css и js (Но мы и так кешируем эти файлы)
* JS движок не обрабатывает дважды файлы (только V8)
* Не парсит js и css, не перерисовывает все DOM-дерево

!SLIDE
## http://github.com/rails/turbolinks

!SLIDE
# ActiveSupport::Queue

    @@@Ruby
    Rails.queue.push Job.new
    job = Rails.queue.pop
    job.run

!SLIDE
# ActiveSupport::Queue

    @@@Ruby
    # config/application.rb

    # Default Synchronous
    config.queue = ActiveSupport::SynchronousQueue.new

    # Default Threaded
    config.queue = ActiveSupport::Queue.new

    # Resque Queue
    config.queue = Resque::Rails::Queue.new

    # Sidekiq Queue
    config.queue = Sidekiq::Client::Queue.new
    
!SLIDE

    @@@Ruby
    # Rails 3.2
    class UsersController < ActionController::Base
      def create
        @user = User.new params[:user]
        if @user.save
          UserMailer.welcome_email(@user).deliver
        end
        respond_with @user
      end
    end

---
    
    @@@Ruby
    # Rails 4
    class UsersController < ActionController::Base
      def create
        @user = User.new params[:user]
        if @user.save
          UserMailer.welcome_email(@user).deliver
        end
        respond_with @user
      end
    end
    
!SLIDE
# Асинхронные мейлеры

    @@@Ruby
    # application.rb
    # Все мейлеры асинхронные
    config.action_mailer.async = true

    # MyMailer теперь асинхронный
    class MyMailer < ActionMailer::Base
      self.async = true
    end

!SLIDE ruby32
# Routing concerns

    @@@Ruby
    # config/routes.rb
    Myapp::Application.routes.draw do
      resources :messages do
        resources :comments
      end
      resources :forwards do
        resources :comments
      end
      resources :uploads do
        resources :comments
      end
      resources :documents do
        resources :comments
      end
      resources :todos do
        resources :comments
      end
    end
    
!SLIDE ruby4
# Routing concerns

    @@@Ruby
    # config/routes.rb
    Myapp::Application.routes.draw do
      concern :commentable do
        resources :comments
      end

      resources :messages,  concerns: :commentable
      resources :forwards,  concerns: :commentable
      resources :uploads,   concerns: :commentable
      resources :documents, concerns: :commentable
      resources :todos,     concerns: :commentable
    end
    
!SLIDE ruby4
# ActionController::Live

    @@@Ruby
    class MyController < ActionController::Base
      include ActionController::Live

      def index
        100.times {
          response.stream.write "hi\n"
        }
        response.stream.close
      end
    end

* Надо использовать thin/rainbows/puma (unicorn не вариант)
    
!SLIDE ruby4
#Meтод PATCH

    @@@Ruby
    class PostsController < ApplicationController 
      # PATCH/PUT /posts/1
      # PATCH/PUT /posts/1.json
      def update
        @post = Post.find(params[:id])
        # ...
      end
    end

* form\_for @persisted\_object будет использовать PATCH
* API запросы могут использовать и PUT и PATCH

!SLIDE
# AJAX формы и CSRF

* form\_for с remote: true по-умолчанию не создает authenticity\_token
* токен берется из meta тэга.

!SLIDE
#ActiveRecord

!SLIDE
# Метод all возвращает ActiveRecord::Relation

    @@@Ruby
    # Rails 3
    > Post.all.class
    Array

    # Rails 4
    > Post.all.class
    ActiveRecord::Relation

    > Post.all.to_a.class
    Array
    
!SLIDE
# Прощай scoped

    @@@Ruby
    # Rails 4
    > Post.scoped
    DEPRECATION WARNING: Use Post.all instead
    
!SLIDE bullets
## update_attributes(name: "value") 

* Запускает валидации
* Запускает хуки
* updated_at обновлен
* Обновляет "грязные" аттрибуты

!SLIDE bullets
## update_attribute(name, "value")

* Запускает хуки
* updated_at обновлен
* Обновляет "грязные" аттрибуты

!SLIDE bullets

## update_columns(name: "value")
## update_column(name, "value")

!SLIDE

# Порядок в \#first, \#last

    @@@Ruby
    # Rails 3
    > Post.first
    SELECT "posts".* FROM "posts" LIMIT 1

    # Rails 4
    > Post.first
    SELECT "posts".* FROM "posts"
      ORDER BY "posts"."id" ASC LIMIT 1
    

!SLIDE
# Метод includes

    @@@Ruby
    # Rails 3
    > Post.includes(:comments).
        where("comments.created_at > ?", ...)

    SELECT ... from posts LEFT OUTER JOIN ...
      WHERE ...

    # Rails 4
    > Post.includes(:comments).
        where("comments.created_at > ?", ...)
    DEPRECATION WARNING
    
!SLIDE
# Метод includes

    @@@Ruby
    # Rails 4
    > Post.includes(:comments).
        where("comments.created_at > ?", ...).
        references(:comments)

    > Post.includes(:comments).
        where(:comments => { :title => "..." })

    > Post.includes(:comments).
        order("comments.created_at")

!SLIDE
# Скоупы

    @@@Ruby
    # Устаревший способ
    scope :published, where(published: true)

    # Ловушка
    scope :recent, where("created_at > ?", 1.week.ago)

    # Только лямбда, только лямбдакор.
    scope :published, -> { where(:published => true) }

!SLIDE

    @@@Ruby
    find_all_by_...
    scoped_by_...
---
    
    @@@Ruby
    where(...)

!SLIDE

    @@@Ruby
    find_last_by_...
---
    
    @@@Ruby
    where(...).last

!SLIDE

    @@@Ruby
    find_or_initialize_by_...
---
    
    @@@Ruby
    where(...).
      first_or_initialize

!SLIDE

    @@@Ruby
    find_or_create_by_...
---
    
    @@@Ruby
    where(...).
      first_or_create
    
!SLIDE

    @@@Ruby
    find_or_create_by_...!
---
    
    @@@Ruby
    where(...).
      first_or_create!
    

!SLIDE
# И самое главное изменение

    @@@Ruby
    # Rails 3.2
    > ERB::Util.html_escape("'")
    => "&#x27;" 

    # Rails 4
    > ERB::Util.html_escape("'")
    => "&#39;" 
    

!SLIDE

http://twitter.com/kalysosmonov

http://github.com/kalys

Спасибо за внимание!
