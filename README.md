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
  aisle_id: [25, 30],
  store_id: {not: 2},
  aisle_id: {not: [25, 30]},
  user_ids: {all: [1, 3]},
  category: /frozen .+/,
  _or: [{in_stock: true}, {backorderd: true}]
}

order: {_score: :desc}

limit: 20, offset: 40

select: [:name]

results = Product.search("milk")
results.size
result.any?
results.each { |result| ... }

Product.search{"apples", load: false}

results.total_count

results.response

fields: ["title^10", "description"]

boost_by: [:orders_count]
boost_by: {orders_count: {factor: 10}}

boost_where: {}
boost_where: {}
boost_where: {]

boost_by_recency: {}

```


https://github.com/ankane


```
```

```ruby
```

```
```


