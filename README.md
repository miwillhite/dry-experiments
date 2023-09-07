# dry-experiments
Using Dry Monad as a workflow

```ruby

require 'dry/monads/do/all'
require 'dry/monads/result'
require 'procto'

M = Dry::Monads

# User
class User
  include Procto.call

  def initialize(attrs)
    @attrs = attrs
  end

  def call
    { name: @attrs[:name] }
  end
end

# A Form validation utility class
class ValidateForm
  include Dry::Monads::Do::All
  include Dry::Monads::Result::Mixin
  include Procto.call

  def initialize(form)
    @form = form
  end

  # Form -> Result String Form
  def call
    # Here Form is just the object we want to validate
    # in this example it just has a method that returns a string
    name = yield validate_name(@form)

    # Output a whitelisted set of attrs
    # N.B. If the validation failed this will be a Failure...
    Success(name: name)
  end

  # Form -> Result String
  def validate_name(form)
    form.name.empty? ?
     Failure('name must not be empty') :
     Success(form.name)
  end
end

# A Registration utility class
class Register
  include Dry::Monads::Task::Mixin
  include Procto.call

  def initialize(user)
    @user = user
  end

  # { name: String } -> Task { name: String, uuid: String }
  def call
    # Async task (like calling to Cognito)
    Task { register_user(@user) }.to_result
  end

  # Our task stub, it will end up appending a uuid to the input
  # { name: String } -> { name: String, uuid: String }
  def register_user(user)
    sleep 2
    user.merge(uuid: 'abc123')
  end
end

# Our test form
Form = Struct.new(:name)

# Pass in an empty string to see the Failure case
f = Form.new('Jane')
# f = Form.new('')

# Entry point
M.Success(f)         # Take our form
.bind(&ValidateForm) # Validate it
.fmap(&User)         # Transform the Form data to a User object
.bind(&Register)     # Register the user in an async task
#=> Result String { name: String, uuid: String }
```
