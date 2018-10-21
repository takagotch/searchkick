### searchkick
---

https://github.com/ankane/searchkick


```
brew install elasticsearch
brew services start elasticsearch

gem 'searchkick'

cd /tmp
curl -o wordnet.tar.gz http://wordnetcode.princeton.edu/3.0/WNprolog-3.0.tar.gz
tar -zxvf wordnet.tar.gz
mv prolog/wn_s.pl /var/lib

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

boost_where: {:user_id: 1}
boost_where: {user_id: {value: 1, factor: 100}}
boost_where: {user_id: [{value: 1, factor: 100}, {value: 2, factor: 200}]]

boost_by_recency: {created_at: {scale: "7d", decay: 0.5}}

Product.search "*"

@products = Product.search "milk", page: params[:page], per_page: 20

Product.search "fresh honey"

Product.search "fresh honey", operator: "or"

class Product < ApplicationRecord
  searchkick word_start: [:name]
end

Product.search "back", fileds: [:name], match: :word_start

User.search query, fields: [{email: :exact}, :name]

User.search "fresh honey", match: :phrase

class Product < ApplicationRecord
  searchkick language: "german"
end

synonyms: -> { CSV.read("/some/path/synonyms.csv") }
synonyms: ["lightbulb => halogenlamp"]

class Product < ApplicaionRecord
  acts_as_taggable
  scope :search_import, -> { includes(:tags) }
  def search_data
    {
      name_tagged: "#{name} #{tags.map(&:name).join(" ")}"
    }
  end
end

Product.search query, fileds: [:name_tagged]

class Product < ApplicationRecord
  searchkick wordnet: true
end

Product.search "zucini", misspellings: {edit_distance: 2}
Product.search "zuchini", misspellings: {below: 5}
Product.search "zuchini", misspellings: false
Product.search "zucini", fileds: [:name, :color], misspellings: {fileds: [:name]}

Product.search "better", exclude: ["peanut butter"]

exclude_queries = {
  "butter" => ["peanut butter"],
  "cream" => ["ice cream", "shipped cream"]
}
Product.search query, exclude: exclude_queries[query]

Product.search("butter", boost_where: {category: {value: "pantry", factor: 0.5}})

gem 'gemoji-parser'
Product.search "🙏🙋", emoji: true

class Product < ApplicationRecord
  belongs_to :department
  def search_data
    {
      name: name,
      department_name: department.name,
      on_sale: sale_price.present?
    }
  end
end

class Product < ApplicationRecord
  scope :search_import, -> { includes(:department) }
end

class Product < ApplicationReocrd
  scope :search_import, -> { where(active: true) }
  def should_index?
    active
  end
end

Product.reindex(resume: true)

class Product < ApplicationRecord
  searchkick callbacks: :async
end


class Product < ApplicationRecord
  searchkick callbacks: false
end

Searchkick.callbacks(:bulk) do
  User.find_each(&:update_fields)
end

Searchkick.callbacks(false) do
  User.find_each(&:update_fileds)
end

class Image < ApplicationRecord
  belongs_to :product
  after_commit :reindex_product
  def reindex_product
    product.reindex
  end
end

Product.search "apple", track: {user_id: current_user.id}

class Product < ApplicationRecord
  has_many :searches, class_name: "Searchjoy", as: :convertable
  searchkick conversions: [:conversions]
  def search_data
    {
      naem: name,
      conversions: seraches.gorup(:query).uniq.count(:user_id)
      # {"ice cream" => 234, "chocolate" => 67, "cream" => 2}
    }
  end
end

rake searchkick:reindex CLASS-Product

class Product < ApplicationRecord
  def search_data
    {
      name: name,
      orderer_ids: orders.pluck(:user_id)
    }
  end
end
Procudt.search "milk", boost_where: {orderer_ids: curretn_user.id}

class Movie < ApplicationRecord
  searchkick word_start: [:title, :director]
end

Movie.search "jurassic pa", fileds: [:title], match: :word_start

class MovieController < ApplicationController
  def autocomplete
    render json: Movie.search(params[:query], {
      fields: ["title^5", "director"],
      match: :word_start,
      limit: 10,
      load: false,
      misspelling: {below: 5}
    }).map(&Ltitle)
  end
end

class Product < ApplicationRecord
  searchkick suggest: [:name]
end

products = Product.search "peantu butta", suggest: true
products.suggestions 

products = Product.search "chuk taylor", aggs: [:product_type, :gender, :brand]
products.aggs

Product.search "wingtips", where: {color: "brandy"}, aggs: [:size]

Product.search "wingtips", where: {color: "brandy"}, aggs: [:size], smart_aggs: false

Product.search "wingtips", aggs: {size: {where: {color: "brandy"}}}

Product.search "apples", aggs: {sotre_id: {limit: 10}}

Product.search "wingtips", aggs: {color: {order: {"_key" => "asc"}}}

price_ranges = [{to: 20}, {from: 20, to: 50}, {from: 50}]
Product.search "*", aggs: {price: {ranges: price_ranges}}

Product.search "*", aggs: {color: {script: {source: "'Color:' = _value"}}}

Product.search "pear", aggs: {products_per_year: {date_histogram: {filed: :created_at, interval: :year}}}
Product.search "orange", body_options: {aggs: {price: {histogram: {filed: :price, interval: 10}}}}


class Product < ApplicationRecord
  searchkick highlight: [:name]
end
bands = Bands.search "cinema", highlight: true

bands.with-highlights.each do |band, highlights|
  highlights[:name]
end

Band.search "cinema", highlight: {tag: "<strong>"}

Band.search "cinema", fileds: [:name], highlight: {fields: [:description]}









```


```
<%= paginate @products %>
<%= will_paginate @products %>

<input type="text" id="query" name="query" />
<script src="jquery.js"></script>
<script>
  var movies = new Bloodhound({
    datumTokenizer: Bloodhound.tokenizers.whitespace,
    queryTokenizer: Bloodhound.tokenizers.whitespace,
    remote: {
      url: '/movies/autocomplete?query=%QUERY',
      wildcard: '%QUERY'
    }
  });
  $('#query').typehead(null, {
    source: movies
  });
</script>

```

https://github.com/ankane


```
```

```ruby
```

```
```


