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




```


https://github.com/ankane


```
```

```ruby
```

```
```


