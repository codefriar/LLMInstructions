# Ruby Best Practices – Instruction File for Claude

## Purpose
This guide defines best practices for writing clean, maintainable, and idiomatic Ruby code, including conventions for Rails and Sinatra. Use these rules as the basis for generating Ruby code in any project.

---

## 1. General Ruby Best Practices

- **Readable and Expressive:**  
  Write code that is clear, expressive, and easy to understand.
- **Follow Community Conventions:**  
  Adhere to the [Ruby Style Guide](https://rubystyle.guide/).
- **Consistent Naming:**  
  Use descriptive, snake_case for variables and methods, CamelCase for classes and modules.
- **Short Methods:**  
  Prefer small, focused methods that do one thing well.
- **Use Enumerables:**  
  Use Ruby's `each`, `map`, `select`, etc., instead of manual loops.
- **Guard Clauses:**  
  Use early returns to reduce nesting:
  ```ruby
  return unless user.valid?
  ```
- **Idiomatic Syntax:**
  - Prefer string interpolation: `"Hello, #{name}"`
  - Use symbol keys for hashes: `{foo: "bar"}`
  - Use safe navigation: `user&.profile&.bio`
- **Error Handling:**  
  Rescue only specific exception types. Avoid rescuing `Exception` or using bare `rescue`.
- **Documentation:**  
  Write short docstrings for public methods and classes. Comment why, not what.

---

## 2. Ruby on Rails Best Practices

- **MVC Structure:**  
  Keep controllers thin, models fat. Move business logic to models or service objects.
- **Strong Parameters:**  
  Always use strong parameters in controllers to whitelist input.
- **RESTful Routes:**  
  Prefer Rails REST conventions for controllers and routes.
- **Database Safety:**  
  Use parameterized queries (`where(name: params[:name])`) to avoid SQL injection.
- **Migrations:**  
  Write reversible migrations. Avoid destructive changes in production.
- **Background Jobs:**  
  Offload long-running tasks to ActiveJob or Sidekiq.
- **Testing:**  
  Use RSpec or Minitest for models, controllers, and integration tests.
- **Security:**  
  Never commit secrets or credentials. Use Rails encrypted credentials.
- **Asset Pipeline:**  
  Use the asset pipeline or Webpacker for JavaScript and CSS.
- **Callbacks:**  
  Use ActiveRecord callbacks sparingly; prefer explicit service objects for business logic.

---

## 3. Sinatra Best Practices

- **Routes Organization:**  
  Group related routes; use modular Sinatra apps (`Sinatra::Base`) for large projects.
- **Views:**  
  Separate business logic from views. Use ERB or Haml for templating.
- **Configuration:**  
  Use environment variables for configuration, not hard-coded values.
- **Middleware:**  
  Use Rack middleware for cross-cutting concerns (logging, authentication, etc.).
- **Security:**  
  Escape all user input in views. Use parameterized queries with ActiveRecord or Sequel.
- **Error Handling:**  
  Use `error` blocks to handle exceptions gracefully and return appropriate responses.
- **Testing:**  
  Use Rack::Test or similar to test routes and responses.

---

## 4. Formatting and Tooling

- **Indentation:**  
  Use 2 spaces per indentation level. No tabs.
- **Line Length:**  
  Limit lines to 80-100 characters.
- **Linting:**  
  Use RuboCop to enforce code style.
- **Dependency Management:**  
  Use Bundler and a `Gemfile`. Pin versions conservatively (`~>`).

---

## 5. Example Snippets

**Good:**

```ruby
# General Ruby
class User
  attr_reader :name

  def initialize(name:)
    @name = name
  end

  def greet
    "Hello, #{name}!"
  end
end

# Rails Controller
class UsersController < ApplicationController
  def create
    user = User.new(user_params)
    if user.save
      redirect_to user_path(user), notice: "User created"
    else
      render :new
    end
  end

  private

  def user_params
    params.require(:user).permit(:name, :email)
  end
end

# Sinatra Route
class MyApp < Sinatra::Base
  get '/hello/:name' do
    "Hello, #{params[:name].capitalize}!"
  end
end
```

**Bad:**

```ruby
# Poor Ruby
def x(y) puts "hi " + y end

# Rails: SQL injection risk
User.where("name = '#{params[:name]}'")

# Sinatra: Hard-coding config
set :port, 4567
```

---