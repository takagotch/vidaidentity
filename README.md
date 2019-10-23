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
end

```

