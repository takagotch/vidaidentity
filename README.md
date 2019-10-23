### vidaidentity
---
https://www.atakama.com/

https://github.com/fulcrumapp/atacama

```
```

```rb
//
class UserFetcher < Atacama::Contract
  option :id, Types::Strict::Number.gt(0)
  returns Types.Instance(User)
  
  def call
    User.find(id)
  end
end

UserFetcher.call(id: 1)
```

```rb
class UserFetcher < Atacama::Step
  option :id, type: Types::Strict::Number.gt(0)
  returns Types.Option(model: Types.Instance(User))
  
  def call
    Option(model: User.find(id))
  rescue ActiveRecord::RecordNotFound
    Return(Error.new('Not found'))
  end
end

class Duration < Atacama::Step
  def call
    start = Time.now
    yield
    $redis.avg('duration', Time.now - start)
  end
end

class UpdateUser < Atacama::Transformer
  option :id, type: Types::Strict::Number.gt(0)
  option :attributes, type: Types::Strict::Hash
  
  return_option :model, Types.Instance(User) | Types.Instance(Error)
  
  step :duration, with: Duration do
    step :find, with: UserFetcher
    step :save
  end
  
  private
  
  def save
    context.model.update_attributes(attributes)
  end
end

UpdateUser.call(id: 1, attributes: {
  email: 'hello@world.com'
})


UpdateUser.new(steps: {
  save: lambda do
    puts "skipping save"
  end
})

UpdateUser.inject(id: 1).call(attributes: { email: 'hello@world.com' })

class HistoryCreate < Atacama::Step
  option :history_class, type: Type::Strict::Class
  optoin :model, type: Types.Instance(ActiveRecord::Base)
  
  def call
    history_class.from_model(model)
  end
end

class UpdateUser < Atacama::Transformer
  option :id, type: Types::Strict::Number.gt(0)
  option :attributes, type: Types::Strict::Hash
  
  returns_option :model, Types.Instance(User) | Types.Instance(Error)
  
  step :duration, with: Duration do
    step :find, with: UserFetcher
    step :save, with: Saver
    step :histroy, with: HistoryCreate.inject(history_class: UserHistory)
  end
end
```

