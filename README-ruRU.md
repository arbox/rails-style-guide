# Вступление

> Ролевые модели важны. <br/>
> -- Офицер Алекс Мёрфи / Робот-полицейский

Целью этого руководства является распространение набора проверенных практик
и стилистических рекомендаций при разработки приложений с помощью Ruby on Rails
(для версий 3 и 4). Это руководство дополняет уже существующий сборник:
[Руби: руководство по стилю оформления][ruby-style-guide].

Некоторые из приведенных здесь рекомендаций будут применимы только к Rails 4.0+.

Вы можете создать копию этого руководства в форматах PDF или HTML при помощи
[Pandoc][].

Переводы данного руководства доступны на следующих языках:

* [английский (исходная версия)](https://github.com/rubocop-hq/rails-style-guide/blob/master/README.md)
* [вьетнамский](https://github.com/CQBinh/rails-style-guide/blob/master/README-viVN.md)
* [китайский традиционный](https://github.com/JuanitoFatas/rails-style-guide/blob/master/README-zhTW.md)
* [китайский упрощенный](https://github.com/JuanitoFatas/rails-style-guide/blob/master/README-zhCN.md)
* [корейский](https://github.com/pureugong/rails-style-guide/blob/master/README-koKR.md)
* [португальский (pt-BR)](https://github.com/abraaomiranda/rails-style-guide/blob/master/README-ptBR.md)
* [русский (данный документ)](https://github.com/arbox/rails-style-guide/blob/master/README-ruRU.md)
* [турецкий](https://github.com/tolgaavci/rails-style-guide/blob/master/README-trTR.md)
* [японский](https://github.com/satour/rails-style-guide/blob/master/README-jaJA.md)

# Rails: руководство по стилю оформления

Настоящее руководство по стилю рекомендует лучшие практики оформления, благодаря
которым обычные разработчики на Rails смогут писать код, который с легкостью
будут поддерживать другие обычные программисты. Руководство по оформлению,
которое отражает повседневные реалии, будет применяться постоянно,
а руководство, стремящееся к идеалу, который не принимается рядовыми
специалистами, подвергается риску вообще быть забытым. При этом абсолютно
не важно, насколько хорошим оно является.

Данное руководство разделено на несколько частей, состоящий из связанных
по смыслу правил. В каждом случае мы попытались обосновать появление этих правил
(объяснение опущено в ситуациях, когда мы посчитали его очевидным).

Все эти правила не появились из пустоты, они по большей части основываются
на нашем собственном обширном профессиональном опыте в разработке ПО,
отзывах и предложениях других членов сообщества разработчиков на Rails
и различных общепризнанных источниках по созданию Rails-приложений.

## Содержание

* [Конфигурация](#Конфигурация)
* [Маршрутизация](#Маршрутизация)
* [Контроллеры](#Контроллеры)
  * [Rendering](#rendering)
* [Модели](#Модели)
  * [ActiveRecord](#activerecord)
  * [Запросы ActiveRecord](#Запросы-activerecord)
* [Миграции](#Миграции)
* [Представления](#Представления)
* [Интернационализация](#Интернационализация)
* [Ресурсы](#Ресурсы)
* [Почтовые модули](#Почтовые-модули)
* [Active Support Core Extensions](#active-support-core-extensions)
* [Время](#Время)
* [Bundler](#Bundler)
* [Управление процессами](#Управление-процессами)

## Конфигурация

* <a name="config-initializers"></a>
  Код для инициализации приложения помещайте в директорию `config/initializers/`.
  Код в этой директории выполняется при запуске приложения.
  <sup>[[ссылка](#config-initializers)]</sup>

* <a name="gem-initializers"></a>
  Для каждого гема записывайте код инициализации в одноименный отдельный файл.
  Например, `carrierwave.rb`, `active_admin.rb` и т.д.
  <sup>[[ссылка](#gem-initializers)]</sup>

* <a name="dev-test-prod-configs"></a>
  Уточните настройки для рабочего (development), тестового (test)
  и промышленного (production) окружений в соответствующих файлах в директории
  `config/environments/`.
  <sup>[[ссылка](#dev-test-prod-configs)]</sup>

  * Укажите добавленные вами ресурсы для комплиляции (при наличии):

  ```ruby
  # config/environments/production.rb
  # Скомпилируйте дополнительные ресурсы (application.js, application.css,
  # и прочие не JS/CSS уже добавлены).
  config.assets.precompile += %w( rails_admin/rails_admin.css rails_admin/rails_admin.js )
  ```

* <a name="app-config"></a>
  Сохраняйте настройки, которые относятся ко всем окружениям, в файле
  `config/application.rb`.
  <sup>[[ссылка](#app-config)]</sup>

* <a name="staging-like-prod"></a>
  Создайте дополнительное окружение `staging`, которое будет очень схоже с
  вашим окружением `production`.
  <sup>[[ссылка](#staging-like-prod)]</sup>

* <a name="yaml-config"></a>
  Храните любые дополнительные файлы конфиругации в формате `YAML` в директории
  `config/`.
  <sup>[[ссылка](#yaml-config)]</sup>

  Начиная с `Rails 4.2` файлы конфиругации в `YAML` можно легко загружать при
  помощи нового метода `config_for`:

  ```ruby
  Rails::Application.config_for(:yaml_file)
  ```
## Маршрутизация

* <a name="member-collection-routes"></a>
  Если вам требуется добавить дополнительные действия к ресурсу REST (и вы
  уверены, что это вам абсолютно необходимо), то используйте пути `member`
  и `collection`.
  <sup>[[ссылка](#member-collection-routes)]</sup>

  ```ruby
  # плохо
  get 'subscriptions/:id/unsubscribe'
  resources :subscriptions

  # хорошо
  resources :subscriptions do
    get 'unsubscribe', on: :member
  end

  # плохо
  get 'photos/search'
  resources :photos

  # хорошо
  resources :photos do
    get 'search', on: :collection
  end
  ```

* <a name="many-member-collection-routes"></a>
  Когда вам нужно определить несколько контекстов маршрутизации при помощи
  `member` или `collection`, используйте альтернативную блочную запись.
  <sup>[[ссылка](#many-member-collection-routes)]</sup>

  ```ruby
  resources :subscriptions do
    member do
      get 'unsubscribe'
      # дополнительные маршруты
    end
  end

  resources :photos do
    collection do
      get 'search'
      # дополнительные маршруты
    end
  end
  ```

* <a name="nested-routes"></a>
  Используйте вложенные определения маршрутов, чтобы лучше показать отношения
  между разными моделями `ActiveRecord`.
  <sup>[[ссылка](#nested-routes)]</sup>

    ```ruby
    class Post < ActiveRecord::Base
      has_many :comments
    end

    class Comments < ActiveRecord::Base
      belongs_to :post
    end

    # routes.rb
    resources :posts do
      resources :comments
    end
    ```
* <a name="namespaced-routes"></a>
  Если существует необходимость делать несколько уровней вложенности, то следует
  применять опцию `shallow: true`. Это оградит пользователя от длинных URL
  `posts/1/comments/5/versions/7/edit` и вас от применения длинных наименований
  вроде `edit_post_comment_version`.

  ```ruby
  resources :posts, shallow: true do
    resources :comments do
      resources :versions
    end
  end
  ```

* <a name="namespaced-routes"></a>
  Используйте определенные в отдельном пространстве имен маршруты, чтобы
  объединить связанные действия.
  <sup>[[ссылка](#namespaced-routes)]</sup>

    ```ruby
    namespace :admin do
      # Направляет /admin/products/* to Admin::ProductsController
      # (app/controllers/admin/products_controller.rb)
      resources :products
    end
    ```

* <a name="no-wild-routes"></a>
  Избегайте устаревшей обобщенной формы записи маршрутов. Такие маршруты откроют
  все действия во всех контроллерах для запросов `GET`.
  <sup>[[ссылка](#no-wild-routes)]</sup>

    ```ruby
    # очень плохо
    match ':controller(/:action(/:id(.:format)))'
    ```

* <a name="no-match-routes"></a>
  Избегайте использования `#match` для определения маршрутов. Эта возможность
  удалена из `Rails 4`.
  <sup>[[ссылка](#no-match-routes)]</sup>

## Контроллеры

* <a name="skinny-controllers"></a>
  Поддерживайте код контроллеров обозримым, контроллеры должны лишь получать
  данные для шаблонов и не должны реализовывать бизнес-логику. Вся бизнес-логика
  вашего приложения должна по определению реализовываться в моделях.
  <sup>[[ссылка](#skinny-controllers)]</sup>

* <a name="one-method"></a>
  Каждое действие в котроллере должно (в идеале) вызывать не более одного
  другого метода (кроме `#find` или `#new`).
  <sup>[[ссылка](#one-method)]</sup>

* <a name="shared-instance-variables"></a>
  Старайтесь не передавать более двух переменных из контроллера в шаблон.
  <sup>[[ссылка](#shared-instance-variables)]</sup>

<!--- @FIXME -->
* <a name="lexically-scoped-action-filter"></a>
  Controller actions specified in the option of Action Filter should be in lexical scope.
  The ActionFilter specified for an inherited action makes it difficult
  to understand the scope of its impact on that action.
  <sup>[[ссылка](#lexically-scoped-action-filter)]</sup>

  ```ruby
  # плохо
  class UsersController < ApplicationController
    before_action :require_login, only: :export
  end

  # хорошо
  class UsersController < ApplicationController
    before_action :require_login, only: :export

    def export
    end
  end
  ```

### Рендеринг

* <a name="inline-rendering"></a>
  Предпочитайте подшаблоны вместо встроенного рендеринга.
  <sup>[[ссылка](#inline-rendering)]</sup>

  ```ruby
  # очень плохо
  class ProductsController < ApplicationController
    def index
      render inline: "<% products.each do |p| %><p><%= p.name %></p><% end %>", type: :erb
    end
  end

  # хорошо
  ## app/views/products/index.html.erb
  <%= render partial: 'product', collection: products %>

  ## app/views/products/_product.html.erb
  <p><%= product.name %></p>
  <p><%= product.price %></p>

  ## app/controllers/foo_controller.rb
  class ProductsController < ApplicationController
    def index
      render :index
    end
  end
  ```

* <a name="plain-text-rendering"></a>
  Используйте `render plain:` вместо `render text:`.
  <sup>[[ссылка](#plain-text-rendering)]</sup>

  ```ruby
  # плохо (устанавливает MIME-тип в `text/html`)
  ...
  render text: 'Ruby!'
  ...

  # плохо (требуется явное объявление MIME-типа)
  ...
  render text: 'Ruby!', content_type: 'text/plain'
  ...

  # хорошо (коротко и точно)
  ...
  render plain: 'Ruby!'
  ...
  ```
* <a name="http-status-code-symbols"></a>
  
  Предпочтите [соответствующие символы](https://gist.github.com/mlanett/a31c340b132ddefa9cca) числовым кодам состояния HTTP.
  Они имеют смысл и не похожи на "магические" числа для менее известных кодов состояния HTTP.
  <sup>[[ссылка](#http-status-code-symbols)]</sup>

  ```ruby
  # плохо
  # некоторый код
  render status: 403
  # некоторый код

  # хорошо
  # некоторый код
  render status: :forbidden
  # некоторый код
  ```

## Модели

* <a name="model-classes"></a>
  Без зазрений совести используйте модели, не базирующиеся на `ActiveRecord`.
  <sup>[[ссылка](#model-classes)]</sup>

* <a name="meaningful-model-names"></a>
  Называйте модели говорящими (но короткими) именами без сокращений.
  <sup>[[ссылка](#meaningful-model-names)]</sup>

* <a name="activeattr-gem"></a>
  Если вам нужна модель, поддерживающая некоторые аспекты `ActiveRecord`
  (например, валидации), без привязки к работе с БД используйте библиотеку
  [ActiveAttr](https://github.com/cgriego/active_attr).
  <sup>[[ссылка](#activeattr-gem)]</sup>

    ```ruby
    class Message
      include ActiveAttr::Model

      attribute :name
      attribute :email
      attribute :content
      attribute :priority

      attr_accessible :name, :email, :content

      validates :name, presence: true
      validates :email, format: { with: /\A[-a-z0-9_+\.]+\@([-a-z0-9]+\.)+[a-z0-9]{2,4}\z/i }
      validates :content, length: { maximum: 500 }
    end
    ```

    Более подробный пример (на английском языке) вы найдете здесь:
    [RailsCast #326: ActiveAttr](http://railscasts.com/episodes/326-activeattr).

* <a name="model-business-logic"></a>
  Не расширяйте свои модели методами, которые реализуют форматирование данных
  (например, кодогенерацию HTML), кроме случаев, когда это напрямую связано
  с бизнес-логикой описываемой предметной области. Такие методы с большой
  вероятностью будут вызываться только из шаблонов представлений, поэтому
  их лучше разместить во вспомогательных модулях (helpers). Реализуйте в моделях
  только бизнес-логику и функционал работы с данными.
  <sup>[[ссылка](#model-business-logic)]</sup>

### ActiveRecord

* <a name="keep-ar-defaults"></a>
  Не меняйте стандартных значений `ActiveRecord` (например, наименования таблиц,
  первичных ключей и т.д.) без особой нужды. Это оправдано лишь в тех случаях,
  когда вы работаете с базой данных, схему которой (по разным причинам) нет
  возможности изменить.
  <sup>[[ссылка](#keep-ar-defaults)]</sup>

    ```ruby
    # плохо (не делайте так, если вы можете изменить схему)
    class Transaction < ActiveRecord::Base
      self.table_name = 'order'
      ...
    end
    ```

* <a name="macro-style-methods"></a>
  Группируйте макро-методы (`has_many`, `validates` и т.д.) в начале определения
  класса.
  <sup>[[ссылка](#macro-style-methods)]</sup>

  ```ruby
  class User < ActiveRecord::Base
    # записывайте стандартную область видимости в начале (если имеется)
    default_scope { where(active: true) }

    # после этого записывайте константы
    COLORS = %w(red green blue)

    # далее следуют макросы доступа к атрибутам
    attr_accessor :formatted_date_of_birth

    attr_accessible :login, :first_name, :last_name, :email, :password

    # Энумераторы для `Rails >= 4` после макросов доступа,
    # предпочтительно использовать новых синтаксис для хешей.
    enum gender: { female: 0, male: 1 }

    # за которыми следуют макросы ассоциаций
    belongs_to :country

    has_many :authentications, dependent: :destroy

    # и макросы валидаций
    validates :email, presence: true
    validates :username, presence: true
    validates :username, uniqueness: { case_sensitive: false }
    validates :username, format: { with: /\A[A-Za-z][A-Za-z0-9._-]{2,19}\z/ }
    validates :password, format: { with: /\A\S{8,128}\z/, allow_nil: true }

    # после этого идут функции обратного вызова (callbacks)
    before_save :cook
    before_save :update_username_lower

    # оставшиеся макросы (например, Devise) дожны записываться в конце

    ...
  end
  ```

* <a name="has-many-through"></a>
  Используйте преимущественно `has_many :through` вместо
  `has_and_belongs_to_many`. Применение ассоциации `has_many :through`
  дает вам большую свободу в определении дополнительных атрибутов и задании
  валидаций на модели объединения.
  <sup>[[ссылка](#has-many-through)]</sup>

    ```ruby
    # не особо (применяется has_and_belongs_to_many)
    class User < ActiveRecord::Base
      has_and_belongs_to_many :groups
    end

    class Group < ActiveRecord::Base
      has_and_belongs_to_many :users
    end

    # предпочтительное решение (применяется has_many :through)
    class User < ActiveRecord::Base
      has_many :memberships
      has_many :groups, through: :memberships
    end

    class Membership < ActiveRecord::Base
      belongs_to :user
      belongs_to :group
    end

    class Group < ActiveRecord::Base
      has_many :memberships
      has_many :users, through: :memberships
    end
    ```

* <a name="read-attribute"></a>
  Используйте `self[:attribute]` вместо `read_attribute(:attribute)`.
  <sup>[[ссылка](#read-attribute)]</sup>

    ```ruby
    # плохо
    def amount
      read_attribute(:amount) * 100
    end

    # хорошо
    def amount
      self[:amount] * 100
    end
    ```

* <a name="write-attribute"></a>
  Преимущественно используйте `self[:attribute] = value` вместо
  `write_attribute(:attribute, value)`.
  <sup>[[ссылка](#write-attribute)]</sup>

  ```ruby
  # плохо
  def amount
    write_attribute(:amount, 100)
  end

  # хорошо
  def amount
    self[:amount] = 100
  end
  ```

* <a name="sexy-validations"></a>
  Всегда применяйте новый синтаксис валидаций, так называемые
  ["sexy" validations](https://web.archive.org/web/20160827104740/http://thelucid.com/2010/01/08/sexy-validation-in-edge-rails-rails-3/).
  <sup>[[ссылка](#sexy-validations)]</sup>

  ```ruby
  # плохо
  validates_presence_of :email
  validates_length_of :email, maximum: 100

  # хорошо
  validates :email, presence: true, length: { maximum: 100 }
  ```
<!--- @FIXME -->
* <a name="single-attribute-validations"></a>
    To make validations easy to read, don't list multiple attributes per
    validation
    <sup>[[ссылка](#single-attribute-validations)]</sup>

    ```ruby
    # плохо
    validates :email, :password, presence: true
    validates :email, length: { maximum: 100 }

    # хорошо
    validates :email, presence: true, length: { maximum: 100 }
    validates :password, presence: true
    ```

* <a name="custom-validator-file"></a>
  Если определенная разработчиком валидация используется несколько раз или
  содержит сложные (регулярные) выражения, то стоит ее вынести в отдельный файл
  валидаторов.
  <sup>[[ссылка](#custom-validator-file)]</sup>

  ```ruby
  # плохо
  class Person
    validates :email, format: { with: /\A([^@\s]+)@((?:[-a-z0-9]+\.)+[a-z]{2,})\z/i }
  end

  # хорошо
  class EmailValidator < ActiveModel::EachValidator
    def validate_each(record, attribute, value)
      record.errors[attribute] << (options[:message] || 'is not a valid email') unless value =~ /\A([^@\s]+)@((?:[-a-z0-9]+\.)+[a-z]{2,})\z/i
    end
  end

  class Person
    validates :email, email: true
  end
  ```

* <a name="app-validators"></a>
  Храните файлы определенных вами валидаторов в `app/validators`.
  <sup>[[ссылка](#app-validators)]</sup>

* <a name="custom-validators-gem"></a>
  Подумайте о том, чтобы выделить ряд определенных вами валидаторов в отдельный
  гем, если вы работаете над несколькими схожими приложениями и валидаторы имеют
  достаточно обобщенные функции.
  <sup>[[ссылка](#custom-validators-gem)]</sup>

* <a name="named-scopes"></a>
  Спокойно применяйте поименованные области поиска (named scopes).
  <sup>[[ссылка](#named-scopes)]</sup>

  ```ruby
  class User < ActiveRecord::Base
    scope :active, -> { where(active: true) }
    scope :inactive, -> { where(active: false) }

    scope :with_orders, -> { joins(:orders).select('distinct(users.id)') }
  end
  ```

* <a name="named-scope-class"></a>
  Если поименованная область поиска определяется с помощью `lambda`
  с дополнительными параметрами, то такая запись очень быстро может стать
  слишком сложной. В таком случае лучше определить метод класса, который будет
  служить той же цели и возвращать объект класса `ActiveRecord::Relation`.
  Наверное, этим же образом можно определять и более простые области поиска.
  <sup>[[ссылка](#named-scop-class)]</sup>

  ```ruby
  class User < ActiveRecord::Base
    def self.with_orders
      joins(:orders).select('distinct(users.id)')
    end
  end
  ```
* <a name="callbacks-order"></a>
  Располагайте объявления колбэков в том порядке, в котором они будут выполняться.
  Для справки см. [Доступные колбэки](http://rusrails.ru/active-record-callbacks.html#dostupnye-kolbeki)
  <sup>[[ссылка](#callbacks-order)]</sup>

    ```ruby
    # плохо
    class Person
      after_commit/after_rollback :after_commit_callback
      after_save :after_save_callback
      around_save :around_save_callback
      after_update :after_update_callback
      before_update :before_update_callback
      after_validation :after_validation_callback
      before_validation :before_validation_callback
      before_save :before_save_callback
      before_create :before_create_callback
      after_create :after_create_callback
      around_create :around_create_callback
      around_update :around_update_callback
    end

    # хорошо
    class Person
      before_validation :before_validation_callback
      after_validation :after_validation_callback
      before_save :before_save_callback
      around_save :around_save_callback
      before_create :before_create_callback
      around_create :around_create_callback
      after_create :after_create_callback
      before_update :before_update_callback
      around_update :around_update_callback
      after_update :after_update_callback
      after_save :after_save_callback
      after_commit/after_rollback :after_commit_callback
    end
    ```

* <a name="beware-skip-model-validations"></a>
  Поймите принцип работы следующих [методов](http://rusrails.ru/active-record-validations.html#propusk-validatsiy).
  Они не вызывают валидацию моделей и могут быстро привести к появлению
  ошибочных записей в базе данных.
  <sup>[[ссылка](#beware-skip-model-validations)]</sup>

   ```ruby
  # плохо
  Article.first.decrement!(:view_count)
  DiscussionBoard.decrement_counter(:post_count, 5)
  Article.first.increment!(:view_count)
  DiscussionBoard.increment_counter(:post_count, 5)
  person.toggle :active
  product.touch
  Billing.update_all("category = 'authorized', author = 'David'")
  user.update_attribute(:website, 'example.com')
  user.update_columns(last_request_at: Time.current)
  Post.update_counters 5, comment_count: -1, action_count: 1

  # хорошо
  user.update_attributes(website: 'example.com')
  ```

* <a name="user-friendly-urls"></a>
  Используйте дружественную пользователю запись URL. Указывайте в URL какой-то
  говорящий сам за себя атрибут модели, а не `id`. Есть несколько путей для
  достижения такого результата:
  <sup>[[ссылка](#user-friendly-urls)]</sup>

  * Переопределите метод `to_param` в модели. Это метод используется в `Rails`
    для генерирования URL к объекту. Стандартная реализация возвращает `id`
    объекта (записи) в виде строки. Это поведение можно переопределить и
    включить некоторый понятный человеку атрибут.

    ```ruby
    class Person
      def to_param
        "#{id} #{name}".parameterize
      end
    end
    ```

    Чтобы преобразовать эту форму в более адекватную для URL, нужно вызвать
    метод `parameterize` на строковом объекте. Идентификатор `id` объекта должен
    быть в начале строки, чтобы его мог найти метод `find` библиотеки
    `ActiveRecord`.

  * Используйте гем `friendly_id`. Эта библиотека создает легко читаемые URL
    с использованием некоторых говорящих атрибутов моделей вместо `id`.

    ```ruby
    class Person
      extend FriendlyId
      friendly_id :name, use: :slugged
    end
    ```


    Изучите [документацию гема](https://github.com/norman/friendly_id), чтобы
    лучше разобраться в его применении.

* <a name="find-each"></a>
  Используйте `find_each` для обхода коллекций объектов `ActiveRecord`. Перебор
  объектов записей базы данных в цикле (например, с использованием метода `all`)
  крайне неэффективен, так как в данном случае в памяти будут созданы все
  интересующие нас объекты за раз. Для такого рода задач больше подходит
  пакетная обработка, при которой методы вызывают записи порциями, что
  значительно сокращает расход памяти.
  <sup>[[ссылка](#find-each)]</sup>


    ```ruby
    # плохо
    Person.all.each do |person|
      person.do_awesome_stuff
    end

    Person.where("age > 21").each do |person|
      person.party_all_night!
    end

    # хорошо
    Person.all.find_each do |person|
      person.do_awesome_stuff
    end

    Person.where("age > 21").find_each do |person|
      person.party_all_night!
    end
    ```

* <a name="before_destroy"></a>
  [Rails создает методы обратного вызова для зависимых ассоциаций](https://github.com/rails/rails/issues/3458),
  поэтому всегда вызывайте метод `before_destroy` для валидации с параметром `prepend: true`.
  <sup>[[ссылка](#before_destroy)]</sup>

  ```ruby
  # плохо (роли будут удалены в любом случае, даже если super_admin? задан)
  has_many :roles, dependent: :destroy

  before_destroy :ensure_deletable

  def ensure_deletable
    raise "Cannot delete super admin." if super_admin?
  end

  # хорошо
  has_many :roles, dependent: :destroy

  before_destroy :ensure_deletable, prepend: true

  def ensure_deletable
    raise "Cannot delete super admin." if super_admin?
  end
  ```

* <a name="has_many-has_one-dependent-option"></a>
  Задавайте опцию `dependent` в ассоциация типа `has_many` и `has_one`.
  <sup>[[ссылка](#has_many-has_one-dependent-option)]</sup>

  ```ruby
  # плохо
  class Post < ActiveRecord::Base
    has_many :comments
  end

  # хорошо
  class Post < ActiveRecord::Base
    has_many :comments, dependent: :destroy
  end
  ```

* <a name="save-bang"></a>
  При сохранении объектов Active Record всегда используйте создающие исключение bang! метод (мутатор) или обрабатывайте возвращаемое значения метода.
  Это относится к `create`, `save`, `update`, `destroy`, `first_or_create` и `find_or_create_by`.
  <sup>[[ссылка](#save-bang)]</sup>

  ```ruby
  # плохо
  user.create(name: 'Bruce')

  # плохо
  user.save

  # хорошо
  user.create!(name: 'Bruce')
  # или
  bruce = user.create(name: 'Bruce')
  if bruce.persisted?
    ...
  else
    ...
  end

  # хорошо
  user.save!
  # или
  if user.save
    ...
  else
    ...
  end
  ```
### Запросы ActiveRecord

* <a name="avoid-interpolation"></a>
  Избегайте интерполяции строк в запросах, это сделает ваш код менее уязвимым
  к атакам типа `SQL injection`.
  <sup>[[ссылка](#avoid-interpolation)]</sup>

  ```ruby
  # плохо (param будет вставлен без экранирования)
  Client.where("orders_count = #{params[:orders]}")

  # хорошо (param будет экранирован должным образом)
  Client.where('orders_count = ?', params[:orders])
  ```

* <a name="named-placeholder"></a>
  Предпочитайте поименованные подстановки вместо позиционных подстановок,
  если у вас в запросе их более двух.
  <sup>[[ссылка](#named-placeholder)]</sup>

  ```ruby
  # сойдет
  Client.where(
    'created_at >= ? AND created_at <= ?',
    params[:start_date], params[:end_date]
  )

  # хорошо
  Client.where(
    'created_at >= :start_date AND created_at <= :end_date',
    start_date: params[:start_date], end_date: params[:end_date]
  )
  ```

* <a name="find"></a>
  Отдавайте предпочтение использованию `find` вместо `where(...).take!`, `find_by!`, если вам нужно
  получить всего одну запись по ее идентификатору.
  Предпочитайте использовать `find` вместо `where.take!`, `find_by!`, или `find_by_id!`
  когда вам нужно получить одну запись по первичному ключу (primary key) и вызвать
  `ActiveRecord::RecordNotFound` если запись не найдена.
  <sup>[[ссылка](#find)]</sup>

  ```ruby
  # плохо
  User.where(id: id).take!

  # плохо
  User.find_by_id!(id)

  # плохо
  User.find_by!(id: id)

  # хорошо
  User.find(id)
  ```

* <a name="find_by"></a>
  Отдавайте предпочтение использованию `find_by` вместо `where` и
  `find_by_attribute`, если вам нужно получить всего одну запись по значению
  какого-то ее атрибута.
  <sup>[[ссылка](#find_by)]</sup>

  ```ruby
  # плохо
  User.where(first_name: 'Bruce', last_name: 'Wayne').first

  # плохо
  User.find_by_first_name_and_last_name('Bruce', 'Wayne')

  # хорошо
  User.find_by(first_name: 'Bruce', last_name: 'Wayne')
  ```
<!--- @FIXME -->
* <a name="find_by"></a>
    Favor the use of `find_by` over `where.take` and `find_by_attribute`
    when you need to retrieve a single record by one or more attributes and return
    `nil` when the record is not found.
    <sup>[[ссылка](#find_by)]</sup>

    ```ruby
    # плохо
    User.where(id: id).take
    User.where(first_name: 'Bruce', last_name: 'Wayne').take

    # плохо
    User.find_by_id(id)
    # плохо, устарел в ActiveRecord 4.0, удален в 4.1+
    User.find_by_first_name_and_last_name('Bruce', 'Wayne')

    # хорошо
    User.find_by(id: id)
    User.find_by(first_name: 'Bruce', last_name: 'Wayne')
    ```
* <a name="where-not"></a>
  Используйте `where.not` вместо простого SQL.
  <sup>[[ссылка](#where-not)]</sup>

  ```ruby
  # плохо
  User.where("id != ?", id)

  # хорошо
  User.where.not(id: id)
  ```
<!--- @FIXME -->
* <a name ="order-by-id"></a>
    Don't use the `id` column for ordering. The sequence of ids is not
    guaranteed to be in any particular order, despite often (incidentally)
    being chronological. Use a timestamp column to order chronologically.
    As a bonus the intent is clearer.
    <sup>[[ссылка](#order-by-id)]</sup>

    ```ruby
    # плохо
    scope :chronological, -> { order(id: :asc) }

    # хорошо
    scope :chronological, -> { order(created_at: :asc) }
    ```

  * <a name="ids"></a>
    Favor the use of `ids` over `pluck(:id)`.
    <sup>[[ссылка](#ids)]</sup>

    ```ruby
    # плохо
    User.pluck(:id)

    # хорошо
    User.ids
    ```
* <a name="squished-heredocs"></a>
  При явном формулировании запроса в таких методах, как `find_by_sql`,
  используйте HEREDOC в сочетании с методом `squish`. Это позволит вам
  оформить код SQL читаемым образом с переносами строк и отступами и
  сохранит подержку подсветки синтаксиса на большинстве платформ (GitHub, Atom,
  RubyMine).
  <sup>[[ссылка](#squished-heredocs)]</sup>

  ```ruby
  User.find_by_sql(<<-SQL.squish)
    SELECT
      users.id, accounts.plan
    FROM
      users
    INNER JOIN
      accounts
    ON
      accounts.user_id = users.id
    # прочие детали...
  SQL
  ```

  [`String#squish`](https://apidock.com/rails/String/squish) удаляет отступы и
  переносы, таким образом запросы будут отображаться в виде обычных строк, а не
  в виде вот таких последовательностей:

  ```
  SELECT\n    users.id, accounts.plan\n  FROM\n    users\n  INNER JOIN\n    acounts\n  ON\n    accounts.user_id = users.id
  ```

* <a name="size-over-count-or-length"></a>
  При запросе коллекций ActiveRecord отдавайте предпочтение `size`
  (выбирает между count/length в зависимости от того, загружена ли уже коллекция)
  или `length` (всегда загружает всю коллекцию и подсчитывает элементы массива)
  вместо `count`  (всегда выполняет запрос в базу данных для подсчета).
  <sup>[[ссылка](#size-over-count-or-length)]</sup>

    ```ruby
    # плохо
    User.count

    # хорошо
    User.all.size

    # хорошо (если вам действительно нужно загрузить всех пользователей в память)
    User.all.length
    ```

## Миграции

* <a name="schema-version"></a>
  Храните файл `schema.rb` (или `structure.sql`) в вашей системе управления
  версиями.
  <sup>[[ссылка](#schema-version)]</sup>

* <a name="db-schema-load"></a>
  Используйте вызов `rake db:schema:load` вместо `rake db:migrate` для
  инициализации пустой базы данных.
  <sup>[[ссылка](#db-schema-load)]</sup>

* <a name="default-migration-values"></a>
  Устанавливайте стандартные значения в миграциях, а не в бизнес-логике вашего
  приложения.
  <sup>[[ссылка](#default-migration-values)]</sup>

  ```ruby
  # плохо (стандартное значение устанавливается в приложении)
   class Product < ActiveRecord::Base
    def amount
      self[:amount] || 0
    end
  end

  # хорошо (стандартное значение устанавливается на уровне БД)
  class AddDefaultAmountToProducts < ActiveRecord::Migration
    def change
      change_column_default :products, :amount, 0
    end
  ```

  Многие опытные разработчики на `Rails` рекомендуют устанавливать стандартные
  значения только на уровне приложения и миграций, однако этот подход скрывает
  множество уязвимостей и потенциальных ошибок. Кроме этого, стоит рассмотреть
  тот момент, что большиство сложных приложений используют одну совместную базу
  данных вместе с другими приложениями, поэтому логика проверки, реализованная
  в приложении на `Rails`, будет недоступа из других приложений.

* <a name="foreign-key-constraints"></a>
  Устанавливайте ограничения на внешние ключи. Начиная с `Rails 4.2`
  библиотека `ActiveRecord` поддерживает внешние ключи напрямую.
  <sup>[[ссылка](#foreign-key-constraints)]</sup>

* <a name="change-vs-up-down"></a>
  При написании миграций для добавления таблиц или столбцов создавайте метод
  `change` вместо методов `up` и `down`.
  <sup>[[ссылка](#change-vs-up-down)]</sup>

  ```ruby
  # старый способ
  class AddNameToPeople < ActiveRecord::Migration
    def up
      add_column :people, :name, :string
    end

    def down
      remove_column :people, :name
    end
  end

  # новый предпочтительный способ
  class AddNameToPeople < ActiveRecord::Migration
    def change
      add_column :people, :name, :string
    end
  end
  ```

* <a name="define-model-class-migrations"></a>
  Если вам нужно использовать код модели в миграциях, постарайтесь задать модели
  так, чтобы не получить в будущем неработающие миграции.
  <sup>[[ссылка](#define-model-class-migrations)]</sup>

  ```ruby
  # db/migrate/<migration_file_name>.rb
  # frozen_string_literal: true

  # плохо
  class ModifyDefaultStatusForProducts < ActiveRecord::Migration
    def change
      old_status = 'pending_manual_approval'
      new_status = 'pending_approval'

      reversible do |dir|
        dir.up do
          Product.where(status: old_status).update_all(status: new_status)
          change_column :products, :status, :string, default: new_status
        end

        dir.down do
          Product.where(status: new_status).update_all(status: old_status)
          change_column :products, :status, :string, default: old_status
        end
      end
    end
  end

  # хорошо
  # Задайте `table_name` в специальном классе, чтобы обеспечить единообразие.
  # Вы будете использовать одну и ту же таблицу в любое время после ее создания.
  # В будущем после изменения класса `Product`
  # и изменений в `table_name` миграция не перестанет работать и не произойдет
  # серьезное повреждение данных.
  class MigrationProduct < ActiveRecord::Base
    self.table_name = :products
  end

  class ModifyDefaultStatusForProducts < ActiveRecord::Migration
    def change
      old_status = 'pending_manual_approval'
      new_status = 'pending_approval'

      reversible do |dir|
        dir.up do
          MigrationProduct.where(status: old_status).update_all(status: new_status)
          change_column :products, :status, :string, default: new_status
        end

        dir.down do
          MigrationProduct.where(status: new_status).update_all(status: old_status)
          change_column :products, :status, :string, default: old_status
        end
      end
    end
  end
  ```

* <a name="meaningful-foreign-key-naming"></a>
  Явно выбирайте наименования для внешних ключей (foreign key), не полагайтесь
  на автоматически сгенерированные имена ключей:
  [Foreign Keys](http://rusrails.ru/rails-database-migrations#foreign-keys.html).
  <sup>[[ссылка](#meaningful-foreign-key-naming)]</sup>

  ```ruby
  # плохо
  class AddFkArticlesToAuthors < ActiveRecord::Migration
    def change
      add_foreign_key :articles, :authors
    end
  end

  # хорошо
  class AddFkArticlesToAuthors < ActiveRecord::Migration
    def change
      add_foreign_key :articles, :authors, name: :articles_author_id_fk
    end
  end
  ```

* <a name="reversible-migration"></a>
  Не используйте необратимые методы миграций в методе `change`.
  Обратимые методы можно найти в списке ниже:
  [ActiveRecord::Migration::CommandRecorder](https://api.rubyonrails.org/classes/ActiveRecord/Migration/CommandRecorder.html)
  <sup>[[ссылка](#reversible-migration)]</sup>

  ```ruby
  # плохо
  class DropUsers < ActiveRecord::Migration
    def change
      drop_table :users
    end
  end

  # хорошо
  class DropUsers < ActiveRecord::Migration
    def up
      drop_table :users
    end

    def down
      create_table :users do |t|
        t.string :name
      end
    end
  end

  # хорошо
  # В этом случае при откате будет использоваться блок в `create_table`
  # https://api.rubyonrails.org/classes/ActiveRecord/ConnectionAdapters.html#method-i-drop_table
  class DropUsers < ActiveRecord::Migration
    def change
      drop_table :users do |t|
        t.string :name
      end
    end
  end
  ```

## Представления

* <a name="no-direct-model-view"></a>
  Ни при каких условиях не следует работать с моделями напрямую из представлений.
  <sup>[[ссылка](#no-direct-model-view)]</sup>

* <a name="no-complex-view-formatting"></a>
  Избегайте сложной логики в представлениях, выделяйте этот код во вспомогательные
  методы представлений (view helpers) или выносите их в модель.
  <sup>[[ссылка](#no-complex-view-formatting)]</sup>

* <a name="partials"></a>
  Избегайте повторений кода, используйте отдельные шаблоны и подшаблоны представлений.
  <sup>[[ссылка](#partials)]</sup>

## Интернационализация

* <a name="locale-texts"></a>
  Строки и другие локальные настройки и детали следует выносить из представлений
  и контроллеров, а также моделей в файлы для конкретных локалей в директории
  `config/locales`.
  <sup>[[ссылка](#locale-texts)]</sup>

* <a name="translated-labels"></a>
  Когда вам нужно перевести идентификаторы для моделей ActiveRecord, применяйте
  контекст `activerecord`:
  <sup>[[ссылка](#translated-labels)]</sup>

  ```
  en:
    activerecord:
      models:
        user: Member
      attributes:
        user:
          name: 'Full name'
  ```

  В этом случае `User.model_name.human` вернет `'Member'` и
  `User.human_attribute_name('name')` вернет `'Full name'`. Переводы этих
  атрибутов будут использоваться в качестве идентификаторов в представлениях.

* <a name="organize-locale-files"></a>
  Разделяйте файлы с переводами представлений от переводов атрибутов
  `ActiveRecord`. Размещайте файлы локалей для моделей в директории
  `locales/models`, а используемые в представлениях тексты в директории
  `locales/views`.
  <sup>[[ссылка](#organize-locale-files)]</sup>

  * Если файлы локалей сохраняются в дополнительных директориях, то пути к ним
    должны быть определены в файле `application.rb`, чтобы файлы локалей могли
    быть загружены.

      ```ruby
      # config/application.rb
      config.i18n.load_path += Dir[Rails.root.join('config', 'locales', '**', '*.{rb,yml}')]
      ```

* <a name="shared-localization"></a>
  Размещайте общие параметры локализации, например, форматы записи дат и валют,
  в файлах в корне директории `locales`.
  <sup>[[ссылка](#shared-localization)]</sup>

* <a name="short-i18n"></a>
  Используйте краткую форму методов `I18n`: `I18n.t` вместо `I18n.translate` и
  `I18n.l` вместо `I18n.localize`.
  <sup>[[ссылка](#short-i18n)]</sup>

* <a name="lazy-lookup"></a>
  Используйте "ленивый" подход к поиску текстов в представлениях. Допустим,
  у вас есть следующая структура:
  <sup>[[ссылка](#lazy-lookup)]</sup>

  ```
  en:
    users:
      show:
        title: 'User details page'
  ```

  Значение для `users.show.title` можно будет найти в шаблоне
  `app/views/users/show.html.haml`, например, так:

  ```ruby
  = t '.title'
  ```

* <a name="dot-separated-keys"></a>
  Задавайте ключи в контроллерах и моделях при помощи разделения точками,
  а не с помощью опции `:scope`. Разделенные точками вызовы проще читать, их
  иерархия более понятна.
  <sup>[[ссылка](#dot-separated-keys)]</sup>

  ```ruby
  # плохо
  I18n.t :record_invalid, scope: [:activerecord, :errors, :messages]

  # хорошо
  I18n.t 'activerecord.errors.messages.record_invalid'
  ```

* <a name="i18n-guides"></a>
  Более подробную информацию по интернационализации (I18n) в Rails можно найти
  по адресу [API интернационализации Rails](http://rusrails.ru/rails-internationalization-i18n-api.html)
  либо [Rails Guides](https://guides.rubyonrails.org/i18n.html) (английский оригинал).
  <sup>[[ссылка](#i18n-guides)]</sup>

## Ресурсы

Применяйте
[конвейер ресурсов](http://rusrails.ru/asset-pipeline.html)
(assets pipeline) для упорядочения структуры вашего приложения.

* <a name="reserve-app-assets"></a>
  Зарезервируйте директорию `app/assets` для ваших собственных таблиц стилей,
  скриптов и/или изображений.
  <sup>[[ссылка](#reserve-app-assets)]</sup>

* <a name="lib-assets"></a>
  Используйте директорию `lib/assets` для ваших собственных библиотек, которые
  не реализуют первичный функционал вашего приложения.
  <sup>[[ссылка](#lib-assets)]</sup>

* <a name="vendor-assets"></a>
  Код сторонних библиотек, например, [jQuery](http://jquery.com/) или
  [bootstrap](http://getbootstrap.com/), следует размещать в
  директории `vendor/assets`.
  <sup>[[ссылка](#vendor-assets)]</sup>

* <a name="gem-assets"></a>
  По возможности используйте упакованные версии необходимых ресурсов, например:
  <sup>[[ссылка](#gem-assets)]</sup>

  - [jquery-rails](https://github.com/rails/jquery-rails);
  - [jquery-ui-rails](https://github.com/jquery-ui-rails/jquery-ui-rails);
  - [bootstrap-sass](https://github.com/twbs/bootstrap-sass);
  - [zurb-foundation](https://github.com/zurb/foundation-sites).

## Почтовые модули

* <a name="mailer-name"></a>
  Называйте почтовые модули по образцу `SomethingMailer`. Без суффикса `Mailer`
  не сразу будет понятно, что это почтовый модуль и какие представления
  связаны с этим модулем.
  <sup>[[ссылка](#mailer-name)]</sup>

* <a name="html-plain-email"></a>
  Создавайте шаблоны представлений в текстовом формате и в формате HTML.
  <sup>[[ссылка](#html-plain-email)]</sup>

* <a name="enable-delivery-errors"></a>
  Включите вызов ошибок при проблемах с доставкой почты в вашем окружении для
  разработки. По умолчанию эти вызовы отключены.
  <sup>[[ссылка](#enable-delivery-errors)]</sup>

  ```ruby
  # config/environments/development.rb

  config.action_mailer.raise_delivery_errors = true
  ```

* <a name="local-smtp"></a>
  Используйте локальный сервер SMTP, например, [Mailcatcher
  ](https://github.com/sj26/mailcatcher) в вашем окружении разработки.
  <sup>[[ссылка](#local-smtp)]</sup>

  ```ruby
  # config/environments/development.rb

  config.action_mailer.smtp_settings = {
    address: 'localhost',
    port: 1025,
    # more settings
  }
  ```

* <a name="default-hostname"></a>
  Указывайте стандартные настройки имени вашего узла.
  <sup>[[ссылка](#default-hostname)]</sup>

  ```ruby
  # config/environments/development.rb
  config.action_mailer.default_url_options = { host: "#{local_ip}:3000" }

  # config/environments/production.rb
  config.action_mailer.default_url_options = { host: 'your_site.com' }

  # в вашем классе мейлера
  default_url_options[:host] = 'your_site.com'
  ```

* <a name="url-not-path-in-email"></a>
  Если вам нужно указать ссылку на ваш сайт в тексте письма, всегда используйте
  методы с суффиксом `_url`, а не `_path`. Методы с суффиксом `_url` включают
  имя вашего узла в текст ссылки, а с суффиксом `_path` не включают.
  <sup>[[ссылка](#url-not-path-in-email)]</sup>

  ```ruby
  # плохо
  You can always find more info about this course
  <%= link_to 'here', course_path(@course) %>

  # хорошо
  You can always find more info about this course
  <%= link_to 'here', course_url(@course) %>
  ```

* <a name="email-addresses"></a>
  Записывайте адреса отправителя и получателя должным образом. Используйте
  следующий формат:
  <sup>[[ссылка](#email-addresses)]</sup>

  ```ruby
  # в классе мейлера
  default from: 'Your Name <info@your_site.com>'
  ```

* <a name="delivery-method-test"></a>
  Убедитесь в том, что метод доставки писем в вашем тестовом окружении обозначен
  как `test`:
  <sup>[[ссылка](#delivery-method-test)]</sup>

  ```ruby
  # config/environments/test.rb

  config.action_mailer.delivery_method = :test
  ```

* <a name="delivery-method-smtp"></a>
  Методом доставки почты для разработки и развертывания должен быть `smtp`:
  <sup>[[ссылка](#delivery-method-smtp)]</sup>

  ```ruby
  # config/environments/development.rb, config/environments/production.rb

  config.action_mailer.delivery_method = :smtp
  ```

* <a name="inline-email-styles"></a>
  При рассылке электронной почты в формате HTML все описания стилей должны быть
  включены в текст, потому что некоторые почтовые программы неверно обрабатывают
  внешние таблицы стилей. Однако, это приводит к сложностям в поддержке таких
  таблиц и повторениям в коде. Для преобразования и внедрения стилей в текст
  письма существую два схожим по функциональности гема:
  [premailer-rails](https://github.com/fphilipe/premailer-rails) и
  [roadie](https://github.com/Mange/roadie).
  <sup>[[ссылка](#inline-email-styles)]</sup>

* <a name="background-email"></a>
  Избегайте рассылки почты параллельно к генерации страницы в ответ на запрос
  пользователя. Это вызовет задержки в загрузке страницы, и запрос может быть
  отклонен из-за превышения таймаута, если вы рассылаете много писем. Вы можете
  преодолеть данное ограничение, вызывая процессы рассылки в фоне, например, при
  помощи гема [sidekiq](https://github.com/mperham/sidekiq).
  <sup>[[ссылка](#background-email)]</sup>

<!--- @FIXME: Translate this snippet. --->
## Active Support Core Extensions

* <a name="try-bang"></a>
  Prefer Ruby 2.3's safe navigation operator `&.` over `ActiveSupport#try!`.
  <sup>[[ссылка](#try-bang)]</sup>

  ```ruby
  # плохо
  obj.try! :fly

  # хорошо
  obj&.fly
  ```

* <a name="active_support_aliases"></a>
  Prefer Ruby's Standard Library methods over `ActiveSupport` aliases.
  <sup>[[ссылка](#active_support_aliases)]</sup>

  ```ruby
  # плохо
  'the day'.starts_with? 'th'
  'the day'.ends_with? 'ay'

  # хорошо
  'the day'.start_with? 'th'
  'the day'.end_with? 'ay'
  ```

* <a name="active_support_extensions"></a>
  Prefer Ruby's Standard Library over uncommon ActiveSupport extensions.
  <sup>[[ссылка](#active_support_extensions)]</sup>

  ```ruby
  # плохо
  (1..50).to_a.forty_two
  1.in? [1, 2]
  'day'.in? 'the day'

  # хорошо
  (1..50).to_a[41]
  [1, 2].include? 1
  'the day'.include? 'day'
  ```

* <a name="inquiry"></a>
  Prefer Ruby's comparison operators over `ActiveSupport` `Array#inquiry`,
  `Numeric#inquiry` and `String#inquiry`.
  <sup>[[ссылка](#inquiry)]</sup>

  ```ruby
  # плохо (String#inquiry)
  ruby = 'two'.inquiry
  ruby.two?

  # хорошо
  ruby = 'two'
  ruby == 'two'

  # плохо (Array#inquiry)
  pets = %w(cat dog).inquiry
  pets.gopher?

  # хорошо
  pets = %w(cat dog)
  pets.include? 'cat'
  ```

## Время

* <a name="tz-config"></a>
  Настройте в файле `application.rb` вашу временную зону.
  <sup>[[ссылка](#tz-config)]</sup>

  ```ruby
  config.time_zone = 'Eastern European Time'
  # опционально (обратите внимание, возможны только значения :utc или :local,
  # по умолчанию :utc)
  config.active_record.default_timezone = :local
  ```

* <a name="time-parse"></a>
  Не используйте `Time.parse`.
  <sup>[[ссылка](#time-parse)]</sup>

  ```ruby
  # плохо
  # Подразумевается, что передаваемая строка со временем отражает временную зону вашей ОС.
  Time.parse('2015-03-02 19:05:37')

  # хорошо
  Time.zone.parse('2015-03-02 19:05:37') # => Mon, 02 Mar 2015 19:05:37 EET +02:00
  ```

* <a name="to-time"></a>
  Не используйте [`String#to_time`](https://apidock.com/rails/String/to_time)
  <sup>[[ссылка](#to-time)]</sup>

  ```ruby
  # плохо (временная зона по умолчанию равна системной)
  '2015-03-02 19:05:37'.to_time

  # хорошо
  Time.zone.parse('2015-03-02 19:05:37') # => Mon, 02 Mar 2015 19:05:37 EET +02:00
  ```

* <a name="time-now"></a>
  Не используйте `Time.now`.
  <sup>[[ссылка](#time-now)]</sup>

  ```ruby
  # плохо
  # Возвращает системное время и не учитывает настройки временной зоны.
  Time.now

  # хорошо
  Time.zone.now # => Fri, 12 Mar 2014 22:04:47 EET +02:00
  Time.current # Более короткая форма записи.
  ```
## Bundler

* <a name="dev-test-gems"></a>
  Держите гемы, которые используются только для разработки и/или тестирования в
  соответствующих группах вашего `Gemfile`.
  <sup>[[ссылка](#dev-test-gems)]</sup>

* <a name="only-good-gems"></a>
  Применяйте в своих проектах только проверенные временем библиотеки. Если
  вы задумываетесь, не включить ли в проект малоизвестный гем, вам следует
  сперва внимательно просмотреть его исходных код.
  <sup>[[ссылка](#only-good-gems)]</sup>

* <a name="os-specific-gemfile-locks"></a>
  Системозависимые библиотеки в вашем проекте будут причиной постоянного
  изменения файла `Gemfile.lock` для проектов с несколькими разработчиками,
  работающими на разных операционных системах. Поэтому внесите все библиотеки,
  написанные для `OS X`, в группу `darwin`, а библиотеки, написанные
  для `Linux`, в группу `linux`.
  <sup>[[ссылка](#os-specific-gemfile-locks)]</sup>

  ```ruby
  # Gemfile
  group :darwin do
    gem 'rb-fsevent'
    gem 'growl'
  end

  group :linux do
    gem 'rb-inotify'
  end
  ```

  Для включения нужных библиотек только в нужном окружении, добавьте в файл
  `config/application.rb` следующие строки:

  ```ruby
  platform = RUBY_PLATFORM.match(/(linux|darwin)/)[0].to_sym
  Bundler.require(platform)
  ```

* <a name="gemfile-lock"></a>
  Сохраняйте файл `Gemfile.lock` в вашей системе управления версиями. Этот файл
  содержит неслучайную информацию. Так вы сможете удостовериться, что все члены
  вашей команды установят точно те же версии библиотек, что и вы, при помощи
  команды `bundle install`.
  <sup>[[ссылка](#gemfile-lock)]</sup>

## Управление процессами

*  <a name="foreman"></a>
   Если в вашем проекте есть много зависимостей от внешних процессов, применяйте
   библиотеку [foreman](https://github.com/ddollar/foreman) для управления ими.
   <sup>[[ссылка](#foreman)]</sup>

# Дополнительные источники

Существует несколько отличных источников по стилю оформления приложений
на `Rails`, на которые вы можете взглянуть, если у вас будет свободное время:

<!--- @FIXME: Find russian translations. -->
* [The Rails 4 Way](https://www.amazon.com/Rails-Way-Addison-Wesley-Professional-Ruby/dp/0321944275)
* [Ruby on Rails Guides](https://guides.rubyonrails.org/)
* [Перевод Ruby on Rails Guides](http://rusrails.ru/)
* [The RSpec Book](https://pragprog.com/book/achbd/the-rspec-book)
* [The Cucumber Book](https://pragprog.com/book/hwcuc/the-cucumber-book)
* [Everyday Rails Testing with RSpec](https://leanpub.com/everydayrailsrspec)
* [Rails 4 Test Prescriptions](https://pragprog.com/book/nrtest2/rails-4-test-prescriptions)
* [Better Specs for RSpec](http://www.betterspecs.org)

# Сотрудничество

Ничто, описанное в этом руководстве, не высечено в камне. И я очень хотел бы
сотрудничать со всеми, кто интересуется стилистикой оформления кода Rails,
чтобы мы смогли вместе создать ресурс, который был бы полезен для всего сообщества
программистов на Руби.

Не стесняйтесь создавать отчеты об ошибках и присылать мне запросы на интеграцию
вашего кода. И заранее большое спасибо за вашу помощь!

Вы можете поддержать проект (и РубоКоп) денежным взносом при помощи
[Patreon](https://www.patreon.com/bbatsov).

## Как сотрудничать в проекте?

Это просто! Просто следуйте [руководству по сотрудничеству](CONTRIBUTING-ruRU.md).

# Лицензирование

![Creative Commons License](https://licensebuttons.net/l/by/3.0/88x31.png)
Данная работа опубликована на условиях лицензии [Creative Commons Attribution 3.0
Unported License](https://creativecommons.org/licenses/by/3.0/deed.en_US)

# Расскажи другому

Создаваемое сообществом руководство по стилю оформления будет малопригодным для
сообщества, которое об этом руководстве ничего не знает. Делитесь ссылками на
это руководство с вашими друзьями и коллегами доступными вам средствами. Каждый
получаемый нами комментарий, предложение или мнение сделает это руководство еще
чуточку лучше. А ведь мы хотим самое лучшее руководство из возможных, не так ли?

Всего,<br/>
[Божидар](https://twitter.com/bbatsov)

<!--- Links -->
[ruby-style-guide]: https://github.com/arbox/ruby-style-guide/blob/master/README-ruRU.md
[transmuter]: https://github.com/kalbasit/transmuter
[pandoc]: http://pandoc.org/
