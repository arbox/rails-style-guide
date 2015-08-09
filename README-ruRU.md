# Вступление

> Ролевые модели важны. <br/>
> -- Офицер Алекс Мёрфи / Робот-полицейский

Целью этого руководства является распространение набора проверенных практик
и стилистических рекомендаций при разработки приложений с помощью Ruby on Rails
(для версий 3 и 4). Это руководство дополняет уже существующий сборник:
[Руби: руководство по стилю оформления][ruby-style-guide].

Некоторые из приведенных здесь рекомендаций будут применимы только к Rails 4.0+.

Вы можете создать копию этого руководства в форматах PDF или HTML при помощи
[Transmuter][].

Переводы данного руководства доступны на следующих языках:

* [английский (исходная версия)](https://github.com/bbatsov/rails-style-guide/blob/master/README.md)
* [китайский традиционный](https://github.com/JuanitoFatas/rails-style-guide/blob/master/README-zhTW.md)
* [китайский упрощенный](https://github.com/JuanitoFatas/rails-style-guide/blob/master/README-zhCN.md)
* [корейский](https://github.com/pureugong/rails-style-guide/blob/master/README-koKR.md)
* [немецкий](https://github.com/arbox/de-rails-style-guide/blob/master/README-deDE.md)
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
* [Модели](#Модели)
  * [ActiveRecord](#activerecord)
  * [Запросы ActiveRecord](#Запросы-activerecord)
* [Миграции](#Миграции)
* [Представления](#Представления)
* [Интернационализация](#Интернационализация)
* [Ресурсы](#Ресурсы)
* [Почтовые модули](#Почтовые-модули)
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

    ```Ruby
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

  ```Ruby
  Rails::Application.config_for(:yaml_file)
  ```
## Маршрутизация

* <a name="member-collection-routes"></a>
  Если вам требуется добавить дополнительные действия к ресурсу REST (и вы
  уверены, что это вам абсолютно необходимо), то используйте пути `member`
  и `collection`.
  <sup>[[ссылка](#member-collection-routes)]</sup>

    ```Ruby
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

    ```Ruby
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

    ```Ruby
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

  ```Ruby
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

    ```Ruby
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

    ```Ruby
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

    ```Ruby
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

### ActiveRecord

* <a name="keep-ar-defaults"></a>
  Не меняйте стандартных значений `ActiveRecord` (например, наименования таблиц,
  первичных ключей и т.д.) без особой нужды. Это оправдано лишь в тех случаях,
  когда вы работаете с базой данных, схему которой (по разным причинам) нет
  возможности изменить.
  <sup>[[ссылка](#keep-ar-defaults)]</sup>

    ```Ruby
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

  ```Ruby
  class User < ActiveRecord::Base
    # записывайте стандартную область видимости в начале (если имеется)
    default_scope { where(active: true) }

    # после этого записывайте константы
    COLORS = %w(red green blue)

    # далее следуют макросы доступа к атрибутам
    attr_accessor :formatted_date_of_birth

    attr_accessible :login, :first_name, :last_name, :email, :password

    # за которыми следуют макросы ассоциаций
    belongs_to :country

    has_many :authentications, dependent: :destroy

    # и макросы валидаций
    validates :email, presence: true
    validates :username, presence: true
    validates :username, uniqueness: { case_sensitive: false }
    validates :username, format: { with: /\A[A-Za-z][A-Za-z0-9._-]{2,19}\z/ }
    validates :password, format: { with: /\A\S{8,128}\z/, allow_nil: true}

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

    ```Ruby
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

    ```Ruby
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

  ```Ruby
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
  Всегда применяйте новый синтаксис валидаций, так называемые ["sexy"
  validations
  ](http://thelucid.com/2010/01/08/sexy-validation-in-edge-rails-rails-3/).
  <sup>[[ссылка](#sexy-validations)]</sup>

  ```Ruby
  # плохо
  validates_presence_of :email
  validates_length_of :email, maximum: 100

  # хорошо
  validates :email, presence: true, length: { maximum: 100 }
  ```

* <a name="custom-validator-file"></a>
  Если определенная разработчиком валидация используется несколько раз или
  содержит сложные (регулярные) выражения, то стоит ее вынести в отдельный файл
  валидаторов.
  <sup>[[ссылка](#custom-validator-file)]</sup>

  ```Ruby
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

  ```Ruby
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
  <sup>[[ссылка](#named-scope-class)]</sup>

  ```Ruby
  class User < ActiveRecord::Base
    def self.with_orders
      joins(:orders).select('distinct(users.id)')
    end
  end
  ```

  Стоит отметить, что определенные таким образом области поиска не могут
  объединяться в цепочки вызовов, как обычные области поиска. Например:

  ```Ruby
  # нельзя объединить
  class User < ActiveRecord::Base
    def User.old
      where('age > ?', 80)
    end

    def User.heavy
      where('weight > ?', 200)
    end
  end
  ```

  Определенные в этом стиле `.old` и `.heavy` работают каждый по отдельности, но
  вызов `User.old.heavy` не сработает, для объединения областей поиска запись
  должна быть такой:

  ```Ruby
  # можно объединить
  class User < ActiveRecord::Base
    scope :old, -> { where('age > 60') }
    scope :heavy, -> { where('weight > 200') }
  end
  ```

* <a name="beware-update-attribute"></a>
  Поймите принцип работы метода [`update_attribute`
  ](http://api.rubyonrails.org/classes/ActiveRecord/Persistence.html#method-i-update_attribute).
  Он не вызывает валидации моделей (в отличие от
  `update_attributes`) и может быстро привести к появлению ошибочных записей в
  базе данных.
  <sup>[[ссылка](#beware-update-attribute)]</sup>

* <a name="user-friendly-urls"></a>
  Используйте дружественную пользователю запись URL. Указывайте в URL какой-то
  говорящий сам за себя атрибут модели, а не `id`. Есть несколько путей для
  достижения такого результата:
  <sup>[[ссылка](#user-friendly-urls)]</sup>

  * Переопределите метод `to_param` в модели. Это метод используется в `Rails`
    для генерирования URL к объекту. Стандартная реализация возвращает `id`
    объекта (записи) в виде строки. Это поведение можно переопределить и
    включить некоторый понятный человеку атрибут.

        ```Ruby
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

        ```Ruby
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


    ```Ruby
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
  [Rails создает методы обратного вызова для зависимых
  ассоциаций](https://github.com/rails/rails/issues/3458), поэтому всегда
  вызывайте метод `before_destroy` для валидации с параметром `prepend: true`.
  <sup>[[ссылка](#before_destroy)]</sup>

  ```Ruby
  # плохо (роли будут удалены в любом случае, даже если super_admin? задан)
  has_many :roles, dependent: :destroy

  before_destroy :ensure_deletable

  def ensure_deletable
    fail "Cannot delete super admin." if super_admin?
  end

  # хорошо
  has_many :roles, dependent: :destroy

  before_destroy :ensure_deletable, prepend: true

  def ensure_deletable
    fail "Cannot delete super admin." if super_admin?
  end
  ```
### Запросы ActiveRecord

* <a name="avoid-interpolation"></a>
  Избегайте интерполяции строк в запросах, это сделает ваш код менее уязвимым
  к атакам типа `SQL injection`.
  <sup>[[ссылка](#avoid-interpolation)]</sup>

  ```Ruby
  # плохо (param будет вставлен без экранирования)
  Client.where("orders_count = #{params[:orders]}")

  # хорошо (param будет экранирован должным образом)
  Client.where('orders_count = ?', params[:orders])
  ```

* <a name="named-placeholder"></a>
  Предпочитайте поименованные подстановки вместо позиционных подстановок,
  если у вас в запросе их более двух.
  <sup>[[ссылка](#named-placeholder)]</sup>

  ```Ruby
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
  Отдавайте предпочтение использованию `find` вместо `where`, если вам нужно
  получить всего одну запись по ее идентификатору.
  <sup>[[ссылка](#find)]</sup>

  ```Ruby
  # плохо
  User.where(id: id).take

  # хорошо
  User.find(id)
  ```

* <a name="find_by"></a>
  Отдавайте предпочтение использованию `find_by` вместо `where`, если вам нужно
  получить всего одну запись по значению какого-то ее атрибута.
  <sup>[[ссылка](#find_by)]</sup>

  ```Ruby
  # плохо
  User.where(first_name: 'Bruce', last_name: 'Wayne').first

  # хорошо
  User.find_by(first_name: 'Bruce', last_name: 'Wayne')
  ```

* <a name="find_each"></a>
  Используйте `find_each`, когда вам нужно работать с целым рядом записей.
  <sup>[[ссылка](#find_each)]</sup>

  ```Ruby
  # плохо (загружает все записи сразу)
  # Это очень неэффективно в случае, когда таблица пользователей имеет тысячи записей.
  User.all.each do |user|
    NewsMailer.weekly(user).deliver_now
  end

  # хорошо (записи запрашиваются порциями)
  User.find_each do |user|
    NewsMailer.weekly(user).deliver_now
  end
  ```

* <a name="where-not"></a>
  Используйте `where.not` вместо простого SQL.
  <sup>[[ссылка](#where-not)]</sup>

  ```Ruby
  # плохо
  User.where("id != ?", id)

  # хорошо
  User.where.not(id: id)
  ```

* <a name="squished-heredocs"></a>
  При явном формулировании запроса в таких методах, как `find_by_sql`,
  используйте HEREDOC в сочетании с методом `squish`. Это позволит вам
  оформить код SQL читаемым образом с переносами строк и отступами и
  сохранит подержку подсветки синтаксиса на большинстве платформ (GitHub, Atom,
  RubyMine).
  <sup>[[link](#squished-heredocs)]</sup>

  ```Ruby
  User.find_by_sql(<<SQL.squish)
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

  [`String#squish`](http://apidock.com/rails/String/squish) удаляет отступы и
  переносы, таким образом запросы будут отображаться в виде обычных строк, а не
  в виде вот таких последовательностей:

  ```
  SELECT\n    users.id, accounts.plan\n  FROM\n    users\n  INNER JOIN\n    acounts\n  ON\n    accounts.user_id = users.id
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

  ```Ruby
  # плохо (стандартное значение устанавливается в приложении)
  def amount
    self[:amount] or 0
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

  ```Ruby
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

* <a name="no-model-class-migrations"></a>
  Не используйте классы моделей в миграциях. Классы моделей постоянно меняются,
  неизбежно наступит момент, когда код миграций перестанет работать из-за
  изменений в модели, хотя ранее этот код работал без проблем.
  <sup>[[ссылка](#no-model-class-migrations)]</sup>

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

<!--- @FIXME -->
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
  `ActiveRecord`. Размещайте файлы локалей для моделей в директории `models`, а
  используемые в представлениях тексты в директории `views`.
  <sup>[[ссылка](#organize-locale-files)]</sup>

  * Если файлы локалей сохраняются в дополнительных директориях, то пути к ним
    должны быть определены в файле `application.rb`, чтобы файлы локалей могли
    быть загружены.

      ```Ruby
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

  ```Ruby
  = t '.title'
  ```

* <a name="dot-separated-keys"></a>
  Задавайте ключи в контроллерах и моделях при помощи разделения точками,
  а не с помощью опции `:scope`. Разделенные точками вызовы проще читать, их
  иерархия более понятна.
  <sup>[[ссылка](#dot-separated-keys)]</sup>

  ```Ruby
  # плохо
  I18n.t :record_invalid, :scope => [:activerecord, :errors, :messages]

  # хорошо
  I18n.t 'activerecord.errors.messages.record_invalid'
  ```

* <a name="i18n-guides"></a>
  Более подробную информацию по интернационализации (I18n) в Rails можно найти
  по адресу [API интернационализации Rails
  ](http://rusrails.ru/rails-internationalization-i18n-api) либо
  [Rails Guides](http://guides.rubyonrails.org/i18n.html) (английский оригинал).
  <sup>[[ссылка](#i18n-guides)]</sup>

## Ресурсы

Применяйте
[конвейер ресурсов](http://guides.rubyonrails.org/asset_pipeline.html)
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
  [bootstrap](http://twitter.github.com/bootstrap/), следует размещать в
  директории `vendor/assets`.
  <sup>[[ссылка](#vendor-assets)]</sup>

* <a name="gem-assets"></a>
  По возможности используйте упакованные версии необходимых ресурсов, например:
  <sup>[[ссылка](#gem-assets)]</sup>

  - [jquery-rails](https://github.com/rails/jquery-rails);
  - [jquery-ui-rails](https://github.com/joliss/jquery-ui-rails);
  - [bootstrap-sass](https://github.com/thomas-mcdonald/bootstrap-sass);
  - [zurb-foundation](https://github.com/zurb/foundation).

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

  ```Ruby
  # config/environments/development.rb

  config.action_mailer.raise_delivery_errors = true
  ```

* <a name="local-smtp"></a>
  Используйте локальный сервер SMTP, например, [Mailcatcher
  ](https://github.com/sj26/mailcatcher) в вашем окружении разработки.
  <sup>[[ссылка](#local-smtp)]</sup>

  ```Ruby
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

  ```Ruby
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

  ```Ruby
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

  ```Ruby
  # в классе мейлера
  default from: 'Your Name <info@your_site.com>'
  ```

* <a name="delivery-method-test"></a>
  Убедитесь в том, что метод доставки писем в вашем тестовом окружении обозначен
  как `test`:
  <sup>[[ссылка](#delivery-method-test)]</sup>

  ```Ruby
  # config/environments/test.rb

  config.action_mailer.delivery_method = :test
  ```

* <a name="delivery-method-smtp"></a>
  Методом доставки почты для разработки и развертывания должен быть `smtp`:
  <sup>[[ссылка](#delivery-method-smtp)]</sup>

  ```Ruby
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

## Время

* <a name="tz-config"></a>
  Настройте в файле `application.rb` вашу временную зону.
  <sup>[[ссылка](#tz-config)]</sup>

  ```Ruby
  config.time_zone = 'Eastern European Time'
  # опционально (обратите внимание, возможны только значения :utc или :local,
  # по умолчанию :utc)
  config.active_record.default_timezone = :local
  ```

* <a name="time-parse"></a>
  Не используйте `Time.parse`.
  <sup>[[ссылка](#time-parse)]</sup>

  ```Ruby
  # плохо
  # Подразумевается, что передаваемая строка со временем отражает временную зону вашей ОС.
  Time.parse('2015-03-02 19:05:37')

  # хорошо
  Time.zone.parse('2015-03-02 19:05:37') # => Mon, 02 Mar 2015 19:05:37 EET +02:00
  ```

* <a name="time-now"></a>
  Не используйте `Time.now`.
  <sup>[[ссылка](#time-now)]</sup>

  ```Ruby
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
  Применяйте в своих проектах только рекомендованные временем библиотеки. Если
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

  ```Ruby
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

  ```Ruby
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
* [The Rails 4 Way](http://www.amazon.com/The-Rails-Addison-Wesley-Professional-Ruby/dp/0321944275)
* [Ruby on Rails Guides](http://guides.rubyonrails.org/)
* [The RSpec Book](http://pragprog.com/book/achbd/the-rspec-book)
* [The Cucumber Book](http://pragprog.com/book/hwcuc/the-cucumber-book)
* [Everyday Rails Testing with RSpec](https://leanpub.com/everydayrailsrspec)
* [Better Specs for RSpec](http://betterspecs.org)

# Сотрудничество

Ничто, описанное в этом руководстве, не высечено в камне. И я очень хотел бы
сотрудничать со всеми, кто интересуется стилистикой оформления кода Rails,
чтобы мы смогли вместе создать ресурс, который был бы полезен для всего сообщества
программистов на Руби.

Не стесняйтесь создавать отчеты об ошибках и присылать мне запросы на интеграцию
вашего кода. И заранее большое спасибо за вашу помощь!

Вы можете поддержать проект (и РубоКоп) денежным взносом при помощи
[gittip](https://www.gittip.com/bbatsov).

[![Дай Gittip](https://rawgithub.com/twolfson/gittip-badge/0.2.0/dist/gittip.png)](https://www.gittip.com/bbatsov)


## Как сотрудничать в проекте?

Это просто! Просто следуйте [руководству по сотрудничеству
](https://github.com/bbatsov/rails-style-guide/blob/master/CONTRIBUTING.md).

# Лицензирование

![Creative Commons License](http://i.creativecommons.org/l/by/3.0/88x31.png)
Данная работа опубликована на условиях лицензии [Creative Commons Attribution 3.0
Unported License](http://creativecommons.org/licenses/by/3.0/deed.en_US)

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
[transmuter]: https://github.com/TechnoGate/transmuter
