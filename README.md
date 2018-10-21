### searchkick
---

https://github.com/ankane/searchkick


```
brew install elasticsearch
brew services start elasticsearch

gem 'searchkick'

```

```ruby
class Product < ApplicationRecord
  searchkick
end
Product.reindex

products = Product.search("apples")
products.each do |product|
  puts product.name
end

Product.search "apples", where: {in_stock: true}, limit: 10, offset: 50

fileds: [name, :brand]

where: {
  expires_at: {gt: Time.now},
  orders_count: 1..10,
  aisle_id: [],
  store_id: {},
  aisle_id: {},
  user_ids: {},
  category: //,
  _or: [{}, {}]
}


```


https://github.com/ankane


```
```

```ruby
```

```
```


