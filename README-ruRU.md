# Вступление

> Role models are important. <br/>
> -- Офицер Алекс Мёрфи / Робот-полицейский

The goal of this guide is to present a set of best practices and style
prescriptions for Ruby on Rails 3 & 4 development. It's a complementary
guide to the already existing community-driven
[Руби: руководство по стилю оформления][ruby-style-guide].

Некоторые из приведенных здесь рекомендаций будут применимы только
для Rails 4.0+.

Вы можете создать копию этого руководства в форматах PDF или HTML при помощи
[Transmuter][].

Переводы данного руководства доступны на следующих языках:

* [английский (исходная версия)](https://github.com/bbatsov/rails-style-guide/blob/master/README.md)
* [китайский традиционный](https://github.com/JuanitoFatas/rails-style-guide/blob/master/README-zhTW.md)
* [китайский упрощенный](https://github.com/JuanitoFatas/rails-style-guide/blob/master/README-zhCN.md)
* [русский (данный документ)](https://github.com/arbox/rails-style-guide/blob/master/README-ruRU.md)
* [турецкий](https://github.com/tolgaavci/rails-style-guide/blob/master/README-trTR.md)
* [японский](https://github.com/satour/rails-style-guide/blob/master/README-jaJA.md)

# Руководство по стилю оформления Rails

This Rails style guide recommends best practices so that real-world Rails
programmers can write code that can be maintained by other real-world Rails
programmers. A style guide that reflects real-world usage gets used, and a
style guide that holds to an ideal that has been rejected by the people it is
supposed to help risks not getting used at all &ndash; no matter how good it is.

The guide is separated into several sections of related rules. I've
tried to add the rationale behind the rules (if it's omitted I've
assumed it's pretty obvious).

I didn't come up with all the rules out of nowhere - they are mostly
based on my extensive career as a professional software engineer,
feedback and suggestions from members of the Rails community and
various highly regarded Rails programming resources.

## Содержание

* [Конфигурация](#Конфигурация)
* [Маршрутизация](#Маршрутизация)
* [Контроллеры](#Контроллеры)
* [Модели](#Модели)
* [Миграции](#Миграции)
* [Представления](#Представления)
* [Интернационализация](#Интернационализация)
* [Assets](#assets)
* [Mailers](#mailers)
* [Bundler](#bundler)
* [Flawed Gems](#flawed-gems)
* [Управление процессами](#Управление-процессами)

## Конфигурация

* <a name="config-initializers"></a>
  Put custom initialization code in `config/initializers`. The code in
  initializers executes on application startup.
  <sup>[[ссылка](#config-initializers)]</sup>

* <a name="gem-initializers"></a>
  Keep initialization code for each gem in a separate file with the same name as
  the gem, for example `carrierwave.rb`, `active_admin.rb`, etc.
  <sup>[[ссылка](#gem-initializers)]</sup>

* <a name="dev-test-prod-configs"></a>
  Adjust accordingly the settings for development, test and production
  environment (in the corresponding files under `config/environments/`)
  <sup>[[ссылка](#dev-test-prod-configs)]</sup>

        ```Ruby
        # config/environments/production.rb
        # Precompile additional assets (application.js, application.css, and all non-JS/CSS are already added)
        config.assets.precompile += %w( rails_admin/rails_admin.css rails_admin/rails_admin.js )
        ```

* <a name="app-config"></a>
  Keep configuration that's applicable to all environments
  in the `config/application.rb` file.
  <sup>[[ссылка](#app-config)]</sup>

* <a name="staging-like-prod"></a>
  Create an additional `staging` environment that closely resembles
  the `production` one.
  <sup>[[ссылка](#staging-like-prod)]</sup>

## Маршрутизация

* <a name="member-collection-routes"></a>
  When you need to add more actions to a RESTful resource (do you really need
  them at all?) use `member` and `collection` routes.
  <sup>[[ссылка](#member-collection-routes)]</sup>

    ```Ruby
    # bad
    get 'subscriptions/:id/unsubscribe'
    resources :subscriptions

    # good
    resources :subscriptions do
      get 'unsubscribe', on: :member
    end

    # bad
    get 'photos/search'
    resources :photos

    # good
    resources :photos do
      get 'search', on: :collection
    end
    ```

* <a name="many-member-collection-routes"></a>
  If you need to define multiple `member/collection` routes use the alternative
  block syntax.
  <sup>[[ссылка](#many-member-collection-routes)]</sup>

    ```Ruby
    resources :subscriptions do
      member do
        get 'unsubscribe'
        # more routes
      end
    end

    resources :photos do
      collection do
        get 'search'
        # more routes
      end
    end
    ```

* <a name="nested-routes"></a>
  Use nested routes to express better the relationship between ActiveRecord
  models.
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
  Use namespaced routes to group related actions.
  <sup>[[ссылка](#namespaced-routes)]</sup>

    ```Ruby
    namespace :admin do
      # Directs /admin/products/* to Admin::ProductsController
      # (app/controllers/admin/products_controller.rb)
      resources :products
    end
    ```

* <a name="no-wild-routes"></a>
  Never use the legacy wild controller route. This route will make all actions
  in every controller accessible via GET requests.
  <sup>[[ссылка](#no-wild-routes)]</sup>

    ```Ruby
    # very bad
    match ':controller(/:action(/:id(.:format)))'
    ```

* <a name="no-match-routes"></a>
  Don't use `match` to define any routes. It's removed from Rails 4.
  <sup>[[ссылка](#no-match-routes)]</sup>

## Контроллеры

* <a name="skinny-controllers"></a>
  Keep the controllers skinny - they should only retrieve data for the view
  layer and shouldn't contain any business logic (all the business logic should
  naturally reside in the model).
  <sup>[[ссылка](#skinny-controllers)]</sup>

* <a name="one-method"></a>
  Each controller action should (ideally) invoke only one method other than an
  initial find or new.
  <sup>[[ссылка](#one-method)]</sup>

* <a name="shared-instance-variables"></a>
  Share no more than two instance variables between a controller and a view.
  <sup>[[ссылка](#shared-instance-variables)]</sup>

## Модели

* <a name="model-classes"></a>
  Introduce non-ActiveRecord model classes freely.
  <sup>[[ссылка](#model-classes)]</sup>

* <a name="meaningful-model-names"></a>
  Name the models with meaningful (but short) names without  abbreviations.
  <sup>[[ссылка](#meaningful-model-names)]</sup>

* <a name="activeattr-gem"></a>
  If you need model objects that support ActiveRecord behavior (like validation)
  use the [ActiveAttr](https://github.com/cgriego/active_attr) gem.
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

    For a more complete example refer to the
    [RailsCast on the subject](http://railscasts.com/episodes/326-activeattr).

### ActiveRecord

* <a name="keep-ar-defaults"></a>
  Avoid altering ActiveRecord defaults (table names, primary key, etc) unless
  you have a very good reason (like a database that's not under your control).
  <sup>[[ссылка](#keep-ar-defaults)]</sup>

    ```Ruby
    # bad - don't do this if you can modify the schema
    class Transaction < ActiveRecord::Base
      self.table_name = 'order'
      ...
    end
    ```

* <a name="macro-style-methods"></a>
  Group macro-style methods (`has_many`, `validates`, etc) in the beginning of
  the class definition.
  <sup>[[ссылка](#macro-style-methods)]</sup>

    ```Ruby
    class User < ActiveRecord::Base
      # keep the default scope first (if any)
      default_scope { where(active: true) }

      # constants come up next
      GENDERS = %w(male female)

      # afterwards we put attr related macros
      attr_accessor :formatted_date_of_birth

      attr_accessible :login, :first_name, :last_name, :email, :password

      # followed by association macros
      belongs_to :country

      has_many :authentications, dependent: :destroy

      # and validation macros
      validates :email, presence: true
      validates :username, presence: true
      validates :username, uniqueness: { case_sensitive: false }
      validates :username, format: { with: /\A[A-Za-z][A-Za-z0-9._-]{2,19}\z/ }
      validates :password, format: { with: /\A\S{8,128}\z/, allow_nil: true}

      # next we have callbacks
      before_save :cook
      before_save :update_username_lower

      # other macros (like devise's) should be placed after the callbacks

      ...
    end
    ```

* <a name="has-many-through"></a>
  Prefer `has_many :through` to `has_and_belongs_to_many`. Using
  `has_many :through` allows additional attributes and validations on the join
  model.
  <sup>[[ссылка](#has-many-through)]</sup>

    ```Ruby
    # using has_and_belongs_to_many
    class User < ActiveRecord::Base
      has_and_belongs_to_many :groups
    end

    class Group < ActiveRecord::Base
      has_and_belongs_to_many :users
    end

    # prefered way - using has_many :through
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
  Prefer `self[:attribute]` over `read_attribute(:attribute)`.
  <sup>[[ссылка](#read-attribute)]</sup>

    ```Ruby
    # bad
    def amount
      read_attribute(:amount) * 100
    end

    # good
    def amount
      self[:amount] * 100
    end
    ```

* <a name="write-attribute"></a>
  Always use the new ["sexy" validations](http://thelucid.com/2010/01/08/sexy-validation-in-edge-rails-rails-3/).
  <sup>[[ссылка](#write-attribute)]</sup>

    ```Ruby
    # bad
    validates_presence_of :email

    # good
    validates :email, presence: true
    ```

* <a name="sexy-validations"></a>
  When a custom validation is used more than once or the validation is some
  regular expression mapping, create a custom validator file.
  <sup>[[ссылка](#sexy-validations)]</sup>

    ```Ruby
    # bad
    class Person
      validates :email, format: { with: /\A([^@\s]+)@((?:[-a-z0-9]+\.)+[a-z]{2,})\z/i }
    end

    # good
    class EmailValidator < ActiveModel::EachValidator
      def validate_each(record, attribute, value)
        record.errors[attribute] << (options[:message] || 'is not a valid email') unless value =~ /\A([^@\s]+)@((?:[-a-z0-9]+\.)+[a-z]{2,})\z/i
      end
    end

    class Person
      validates :email, email: true
    end
    ```

* <a name="custom-validator-file"></a>
  Keep custom validators under `app/validators`.
  <sup>[[ссылка](#custom-validator-file)]</sup>

* <a name="app-validators"></a>
  Consider extracting custom validators to a shared gem if you're maintaining
  several related apps or the validators are generic enough.
  <sup>[[ссылка](#app-validators)]</sup>

* <a name="custom-validators-gem"></a>
  Use named scopes freely.
  <sup>[[ссылка](#custom-validators-gem)]</sup>

    ```Ruby
    class User < ActiveRecord::Base
      scope :active, -> { where(active: true) }
      scope :inactive, -> { where(active: false) }

      scope :with_orders, -> { joins(:orders).select('distinct(users.id)') }
    end
    ```

* <a name="named-scopes"></a>
  Wrap named scopes in `lambdas` to initialize them lazily (this is only
  a prescription in Rails 3, but is mandatory in Rails 4).
  <sup>[[ссылка](#named-scopes)]</sup>

    ```Ruby
    # bad
    class User < ActiveRecord::Base
      scope :active, where(active: true)
      scope :inactive, where(active: false)

      scope :with_orders, joins(:orders).select('distinct(users.id)')
    end

    # good
    class User < ActiveRecord::Base
      scope :active, -> { where(active: true) }
      scope :inactive, -> { where(active: false) }

      scope :with_orders, -> { joins(:orders).select('distinct(users.id)') }
    end
    ```

* <a name="named-scope-class"></a>
  When a named scope defined with a lambda and parameters becomes too
  complicated, it is preferable to make a class method instead which serves
  the same purpose of the named scope and returns an `ActiveRecord::Relation`
  object. Arguably you can define even simpler scopes like this.
  <sup>[[ссылка](#named-scope-class)]</sup>

    ```Ruby
    class User < ActiveRecord::Base
      def self.with_orders
        joins(:orders).select('distinct(users.id)')
      end
    end
    ```

* <a name="beware-update-attribute"></a>
  Beware of the behavior of the [`update_attribute`](http://api.rubyonrails.org/classes/ActiveRecord/Persistence.html#method-i-update_attribute) method. It doesn't  run the model validations (unlike `update_attributes`) and could easily corrupt the model state.
  <sup>[[ссылка](#beware-update-attribute)]</sup>

* <a name="user-friendly-urls"></a>
  Use user-friendly URLs. Show some descriptive attribute of the model in the URL rather than its `id`.  There is more than one way to achieve this:
  <sup>[[ссылка](#user-friendly-urls)]</sup>

  * Override the `to_param` method of the model. This method is used by Rails
    for constructing a URL to the object. The default implementation returns the
    `id` of the record as a String. It could be overridden to include another
    human-readable attribute.

        ```Ruby
        class Person
          def to_param
            "#{id} #{name}".parameterize
          end
        end
        ```

    In order to convert this to a URL-friendly value, `parameterize` should be
    called on the string. The `id` of the object needs to be at the beginning so
    that it can be found by the `find` method of ActiveRecord.

  * Use the `friendly_id` gem. It allows creation of human-readable URLs by using
    some descriptive attribute of the model instead of its `id`.

        ```Ruby
        class Person
          extend FriendlyId
          friendly_id :name, use: :slugged
        end
        ```

  Check the [gem documentation](https://github.com/norman/friendly_id)
  for more information about its usage.

* <a name="find-each"></a>
  Use `find_each` to iterate over a collection of AR objects. Looping through
  a collection of records from the database (using the `all`
  method, for example) is very inefficient since it will try to
  instantiate all the objects at once. In that case, batch processing
  methods allow you to work with the records in batches, thereby
  greatly reducing memory consumption.
  <sup>[[ссылка](#find-each)]</sup>


    ```Ruby
    # bad
    Person.all.each do |person|
      person.do_awesome_stuff
    end

    Person.where("age > 21").each do |person|
      person.party_all_night!
    end

    # good
    Person.all.find_each do |person|
      person.do_awesome_stuff
    end

    Person.where("age > 21").find_each do |person|
      person.party_all_night!
    end
    ```

* <a name="before_destroy"></a>
  Since [Rails creates callbacks for dependent
  associations](https://github.com/rails/rails/issues/3458), always call
  `before_destroy` callbacks that perform validation with `prepend: true`.

  ```Ruby
  # bad (roles will be deleted automatically even if super_admin? is true)
  has_many :roles, dependent: :destroy

  before_destroy :ensure_deletable

  def ensure_deletable
    fail "Cannot delete super admin." if super_admin?
  end

  # good
  has_many :roles, dependent: :destroy

  before_destroy :ensure_deletable, prepend: true

  def ensure_deletable
    fail "Cannot delete super admin." if super_admin?
  end
  ```

## Миграции


* <a name="schema-version"></a>
  Keep the `schema.rb` (or `structure.sql`) under version control.
<sup>[[ссылка](#schema-version)]</sup>

* <a name="db-schema-load"></a>
  Use `rake db:schema:load` instead of `rake db:migrate` to initialize an empty
  database.
<sup>[[ссылка](#db-schema-load)]</sup>

* <a name="default-migration-values"></a>
  Enforce default values in the migrations themselves instead of in the
  application layer.
<sup>[[ссылка](#default-migration-values)]</sup>

  ```Ruby
  # bad - application enforced default value
  def amount
    self[:amount] or 0
  end
  ```

  While enforcing table defaults only in Rails is suggested by many
  Rails developers, it's an extremely brittle approach that
  leaves your data vulnerable to many application bugs.  And you'll
  have to consider the fact that most non-trivial apps share a
  database with other applications, so imposing data integrity from
  the Rails app is impossible.

* <a name="foreign-key-constraints"></a>
  Enforce foreign-key constraints. While ActiveRecord does not support them
  natively, there some great third-party gems like
  [schema_plus](https://github.com/lomba/schema_plus) and
  [foreigner](https://github.com/matthuhiggins/foreigner).
<sup>[[ссылка](#foreign-key-constraints)]</sup>

* <a name="change-vs-up-down"></a>
  When writing constructive migrations (adding tables or columns),
  use the `change` method instead of `up` and `down` methods.
<sup>[[ссылка](#change-vs-up-down)]</sup>

  ```Ruby
  # the old way
  class AddNameToPeople < ActiveRecord::Migration
    def up
      add_column :people, :name, :string
    end

    def down
      remove_column :people, :name
    end
  end

  # the new prefered way
  class AddNameToPeople < ActiveRecord::Migration
    def change
      add_column :people, :name, :string
    end
  end
  ```

* <a name="no-model-class-migrations"></a>
  Don't use model classes in migrations. The model classes are constantly
  evolving and at some point in the future migrations that used to work might
  stop, because of changes in the models used.
<sup>[[ссылка](#no-model-class-migrations)]</sup>

## Views

* <a name="no-direct-model-view"></a>
  Never call the model layer directly from a view.
<sup>[[ссылка](#no-direct-model-view)]</sup>

* <a name="no-complex-view-formatting"></a>
  Never make complex formatting in the views, export the formatting to a method
  in the view helper or the model.
<sup>[[ссылка](#no-complex-view-formatting)]</sup>

* <a name="partials"></a>
  Mitigate code duplication by using partial templates and layouts.
<sup>[[ссылка](#partials)]</sup>

## Internationalization

* <a name="locale-texts"></a>
  No strings or other locale specific settings should be used in the views,
  models and controllers. These texts should be moved to the locale files in the
  `config/locales` directory.
<sup>[[ссылка](#locale-texts)]</sup>

* <a name="translated-labels"></a>
  When the labels of an ActiveRecord model need to be translated, use the
  `activerecord` scope:
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

  Then `User.model_name.human` will return "Member" and
  `User.human_attribute_name("name")` will return "Full name". These
  translations of the attributes will be used as labels in the views.


* <a name="organize-locale-files"></a>
  Separate the texts used in the views from translations of ActiveRecord
  attributes. Place the locale files for the models in a folder `models` and the
  texts used in the views in folder `views`.
<sup>[[ссылка](#organize-locale-files)]</sup>

  * When organization of the locale files is done with additional directories,
    these directories must be described in the `application.rb` file in order
    to be loaded.

      ```Ruby
      # config/application.rb
      config.i18n.load_path += Dir[Rails.root.join('config', 'locales', '**', '*.{rb,yml}')]
      ```

* <a name="shared-localization"></a>
  Place the shared localization options, such as date or currency formats, in
  files under the root of the `locales` directory.
<sup>[[ссылка](#shared-localization)]</sup>

* <a name="short-i18n"></a>
  Use the short form of the I18n methods: `I18n.t` instead of `I18n.translate`
  and `I18n.l` instead of `I18n.localize`.
<sup>[[ссылка](#short-i18n)]</sup>

* <a name="lazy-lookup"></a>
  Use "lazy" lookup for the texts used in views. Let's say we have the following
  structure:
<sup>[[ссылка](#lazy-lookup)]</sup>

  ```
  en:
    users:
      show:
        title: 'User details page'
  ```

  The value for `users.show.title` can be looked up in the template
  `app/views/users/show.html.haml` like this:

  ```Ruby
  = t '.title'
  ```

* <a name="dot-separated-keys"></a>
  Use the dot-separated keys in the controllers and models instead of specifying
  the `:scope` option. The dot-separated call is easier to read and trace the
  hierarchy.
<sup>[[ссылка](#dot-separated-keys)]</sup>

  ```Ruby
  # use this call
  I18n.t 'activerecord.errors.messages.record_invalid'

  # instead of this
  I18n.t :record_invalid, :scope => [:activerecord, :errors, :messages]
  ```

* <a name="i18n-guides"></a>
  More detailed information about the Rails i18n can be found in the [Rails
  Guides](http://guides.rubyonrails.org/i18n.html)
<sup>[[ссылка](#i18n-guides)]</sup>

## Assets

Use the [assets pipeline](http://guides.rubyonrails.org/asset_pipeline.html) to leverage organization within
your application.

* <a name="reserve-app-assets"></a>
  Reserve `app/assets` for custom stylesheets, javascripts, or images.
<sup>[[ссылка](#reserve-app-assets)]</sup>

* <a name="lib-assets"></a>
  Use `lib/assets` for your own libraries that don’t really fit into the
  scope of the application.
<sup>[[ссылка](#lib-assets)]</sup>

* <a name="vendor-assets"></a>
  Third party code such as [jQuery](http://jquery.com/) or
  [bootstrap](http://twitter.github.com/bootstrap/) should be placed in
  `vendor/assets`.
<sup>[[ссылка](#vendor-assets)]</sup>

* <a name="gem-assets"></a>
  When possible, use gemified versions of assets (e.g.
  [jquery-rails](https://github.com/rails/jquery-rails),
  [jquery-ui-rails](https://github.com/joliss/jquery-ui-rails),
  [bootstrap-sass](https://github.com/thomas-mcdonald/bootstrap-sass),
  [zurb-foundation](https://github.com/zurb/foundation)).
<sup>[[ссылка](#gem-assets)]</sup>

## Mailers

* <a name="mailer-name"></a>
  Name the mailers `SomethingMailer`. Without the Mailer suffix it isn't
  immediately apparent what's a mailer and which views are related to the
  mailer.
<sup>[[ссылка](#mailer-name)]</sup>

* <a name="html-plain-email"></a>
  Provide both HTML and plain-text view templates.
<sup>[[ссылка](#html-plain-email)]</sup>

* <a name="enable-delivery-errors"></a>
  Enable errors raised on failed mail delivery in your development environment.
  The errors are disabled by default.
<sup>[[ссылка](#enable-delivery-errors)]</sup>

  ```Ruby
  # config/environments/development.rb

  config.action_mailer.raise_delivery_errors = true
  ```

* <a name="local-smtp"></a>
  Use a local SMTP server like
  [Mailcatcher](https://github.com/sj26/mailcatcher) in the development
  environment.
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
  Provide default settings for the host name.
<sup>[[ссылка](#default-hostname)]</sup>

  ```Ruby
  # config/environments/development.rb
  config.action_mailer.default_url_options = { host: "#{local_ip}:3000" }

  # config/environments/production.rb
  config.action_mailer.default_url_options = { host: 'your_site.com' }

  # in your mailer class
  default_url_options[:host] = 'your_site.com'
  ```

* <a name="url-not-path-in-email"></a>
  If you need to use a link to your site in an email, always use the `_url`, not
  `_path` methods. The `_url` methods include the host name and the `_path`
  methods don't.
<sup>[[ссылка](#url-not-path-in-email)]</sup>

  ```Ruby
  # bad
  You can always find more info about this course
  = link_to 'here', course_path(@course)

  # good
  You can always find more info about this course
  = link_to 'here', course_url(@course)
  ```

* <a name="email-addresses"></a>
  Format the from and to addresses properly. Use the following format:
<sup>[[ссылка](#email-addresses)]</sup>

  ```Ruby
  # in your mailer class
  default from: 'Your Name <info@your_site.com>'
  ```

* <a name="delivery-method-test"></a>
  Make sure that the e-mail delivery method for your test environment is set to
  `test`:
<sup>[[ссылка](#delivery-method-test)]</sup>

  ```Ruby
  # config/environments/test.rb

  config.action_mailer.delivery_method = :test
  ```

* <a name="delivery-method-smtp"></a>
  The delivery method for development and production should be `smtp`:
<sup>[[ссылка](#delivery-method-smtp)]</sup>

  ```Ruby
  # config/environments/development.rb, config/environments/production.rb

  config.action_mailer.delivery_method = :smtp
  ```

* <a name="inline-email-styles"></a>
  When sending html emails all styles should be inline, as some mail clients
  have problems with external styles. This however makes them harder to maintain
  and leads to code duplication. There are two similar gems that transform the
  styles and put them in the corresponding html tags:
  [premailer-rails](https://github.com/fphilipe/premailer-rails) and
  [roadie](https://github.com/Mange/roadie).
<sup>[[ссылка](#inline-email-styles)]</sup>

* <a name="background-email"></a>
  Sending emails while generating page response should be avoided. It causes
  delays in loading of the page and request can timeout if multiple email are
  sent. To overcome this emails can be sent in background process with the help
  of [sidekiq](https://github.com/mperham/sidekiq) gem.
<sup>[[ссылка](#background-email)]</sup>

## Bundler

* <a name="dev-test-gems"></a>
  Put gems used only for development or testing in the appropriate group in the
  Gemfile.
<sup>[[ссылка](#dev-test-gems)]</sup>

* <a name="only-good-gems"></a>
  Use only established gems in your projects. If you're contemplating on
  including some little-known gem you should do a careful review of its source
  code first.
<sup>[[ссылка](#only-good-gems)]</sup>

* <a name="os-specific-gemfile-locks"></a>
  OS-specific gems will by default result in a constantly changing
  `Gemfile.lock` for projects with multiple developers using different operating
  systems.  Add all OS X specific gems to a `darwin` group in the Gemfile, and
  all Linux specific gems to a `linux` group:
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

  To require the appropriate gems in the right environment, add the
  following to `config/application.rb`:

  ```Ruby
  platform = RUBY_PLATFORM.match(/(linux|darwin)/)[0].to_sym
  Bundler.require(platform)
  ```

* <a name="gemfile-lock"></a>
  Do not remove the `Gemfile.lock` from version control. This is not some
  randomly generated file - it makes sure that all of your team members get the
  same gem versions when they do a `bundle install`.
<sup>[[ссылка](#gemfile-lock)]</sup>

## Flawed Gems

This is a list of gems that are either problematic or superseded by
other gems. You should avoid using them in your projects.

* [rmagick](http://rmagick.rubyforge.org/) - this gem is notorious for its memory consumption. Use  [minimagick](https://github.com/probablycorey/mini_magick) instead.


*  [autotest](http://www.zenspider.com/ZSS/Products/ZenTest/) - old solution for running tests automatically. Far  inferior to [guard](https://github.com/guard/guard) and [watchr](https://github.com/mynyml/watchr).


*  [rcov](https://github.com/relevance/rcov) - code coverage tool, not  compatible with Ruby 1.9. Use
  [SimpleCov](https://github.com/colszowka/simplecov) instead.


*  [therubyracer](https://github.com/cowboyd/therubyracer) - the use of  this gem in production is strongly discouraged as it uses a very large amount of
  memory. I'd suggest using `node.js` instead.


This list is also a work in progress. Please, let me know if you know
other popular, but flawed gems.

## Управление процессами

*  <a name="foreman"></a>
   Если в вашем проекте есть много зависимостей от внешних процессов, применяйте
   библиотеку [foreman](https://github.com/ddollar/foreman) для управления ими.
   <sup>[[ссылка](#foreman)]</sup>

# Дополнительные источники

There are a few excellent resources on Rails style, that you should
consider if you have time to spare:

*  [The Rails 4 Way](http://www.amazon.com/The-Rails-Addison-Wesley-Professional-Ruby/dp/0321944275)* [Ruby on Rails Guides](http://guides.rubyonrails.org/)
* [The RSpec Book](http://pragprog.com/book/achbd/the-rspec-book)
* [The Cucumber Book](http://pragprog.com/book/hwcuc/the-cucumber-book)
* [Everyday Rails Testing with RSpec](https://leanpub.com/everydayrailsrspec)


# Contributing

Ничто, описанное в этом руководстве, не высечено в камне. И я очень хотел бы
сотрудничать со всеми, кто интересуется стилистикой оформления кода Rails,
чтобы мы смогли вместе создать ресурс, который был бы полезен для всего сообщества
программистов на Руби.

Не стесняйтесь создавать отчеты об ошибках и присылать мне запросы на интеграцию
вашего кода. И заранее большое спасибо за вашу помощь!

Вы можете поддержать проект (и РубоКоп) денежным взносом
при помощи [gittip](https://www.gittip.com/bbatsov).

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
[transmutter]: https://github.com/TechnoGate/transmuter
