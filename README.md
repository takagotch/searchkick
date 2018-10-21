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

heroku addons:create bonsai
heroku config:set ELASTICSEARCH_URL=`heroku config:get BONSAI_URL`
hroku addons:create foundelasticsearch
heroku addons:open foundelasticsearch
heroku config:get FOUNDELASTICSEARCH_URL
heroku config:set ELASTICSEARCH_URL=https://elastic:password@12345.us-east-1.aws.found.io
heroku run rake searchkick:reindex CLASS=Product
rake searchkick:reindex CLASS=Product
rake searchkick:reindex CLASS=Product

rake searchkick:reindex:all

git clone https://github.com/ankane/searchkick.git
cd searchkick
bundle install
rake test

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

bands = Band.search "cinema", highlight: {fragment_size: 20}
bands.with_highlights(multiple: true).each do |band, highlights|
  highlights[:name].join(" and ")
end
Band.search "cinema", fileds: [:nmae], highlight: {fieds: {name: {fragment_size: 200}}}

product = Product.first
product.similar(fields: [:name], where: {size: "12 oz"})

class Restraunt < ApplicationReocrd
  searchkick locations: [:location]
  def search_data
    attributes.merge(location: {lat: latitude, lon: longitude})
  end
end

Restaurant.search "pizza", where: {location: {near: {lat: 37, lon: -114}, within: "100mi"}}
Restaurant.seach "sushi", where: {location: {top_left: {lat: 38, lon: -123}, bottom_right: {lat: 37, lon: -122}}}
Restaurant.search "dessert", where: {location: {geo_polygon: {points: [{lat: 38, lon: -123},{lat: 39, lon: -123},{lat: 37, lon: 122}]}}}

Restaurant.search "noodles", boost_by_distance: {location: {origin: {lat" 37, lon: -122}}}
Restaurant.search "wings", boost_by_distance: {location: {origin: {lat: 37, lon: -122}, function: "linear", scale: "30mi". decay: 0.5}}

class Restaurant < ApplicationRecord
  searchkick geo_shape: {
    bounds: {tree: "geohash", precision: "1km"}
  }
  def search_data
    attributes.merge(
      bounds: {
        type: "envelope",
        coordinates: [{lat: 4, lon: 1}, {lat: 2, lon: 3}]
      }
    )
  end
end

Restaurant.search "soup", where: {bounds: {geo_shape: {type: "polygon", coordinates: [[{lat: 38, lon: -123}, ...]]}}}
Restaurant.search "salad", where: {bounds: {geo_shape: {type: "circle", relation: "within", coordinates: [[{lat: 38, lon: -123}, radius: "1km" ...]]}}}
Restaurant.search "burger", where: {bounds: {geo_shape: {type: "envelope", relation: "disjoint", coordinates: [{lat: 38, lon: -123}, {lat: 37, lon: -122}]}}}
Restaurant.search "fries", where: {bounds: {geo_shape: {type: "envelope", relation: "contains", coordinates: [{lat: 38, lon: -123}, {lat: 37, lon: -122}]}}}

class Dog < Animal
end

class Animal < ApplicationRecord
  searchkick inheritance: true
end
Animal.reindex
Dog.reindex
Animal.search "*"
Dog.search "*"
Animal.search "*", type: [Dog, Cat]

Dog.search "airbudd", suggest: true

Product.search("soap", debug: true)
Product.search("soap", explain: true).response
Product.search_index.token("Dish Washer Soap", analyzer: "searchkick_index")
Product.search_index.tokens("dishwasher soap", analyzer: "searchkick_search")
Product.search_index.tokens("dishwasher soap", analyzer: "searchkick_search2")
Product.search_index.tokens("San Diego", analyzer: "searchkick_word_start_index")
Product.search_index.tokens("dieg", analyzer: "searchkick_word_search")


EMV["ELASTICSEARCH_URL"] = "https://es-domain-1234.us-east-1.es.amazonaws.com"
gem 'faraday_middleware-aws-sigv4'

Searchkick.aws_credentials = {
  access_key_id: ENV["AWS_ACCESSS_KEY_ID"],
  secret_access_key: ENV["AWS_SECRET_ACCESS_KEY"],
  region: "us-east"
}

ENV["ELASTICSEARCH_URL"] = "https://user.password@host"

ENV["ELASTICSEARCH_URL"] = "https://user.password@host2,https://user:password@host2"

config.lograge.custom_options = lambda do |event|
  options = {}
  options[:search] = event.payload[] if event.payload[:searchkick_runtimate].to_f > 0
  options
end

gem 'oj'

gem 'typhoeus'

Ethon.logger = Logger.new(nil)
class Product < ApplicationRecord
  searchkick searchable: [:name]
end

class Product < ApplicationRecord
  searchkick filterable [:brand]
end

Product.rendex(async: true)
Product.search_index.promote(index_name)
Searchkick.redis = Redis.new
Searchkick.reindex_status(index_name)
Product.reindex(async: {wait: true})

gem 'activejob-traffic_control', '>= 0.1.3'

ActiveJob::TrafficControl.client = Searchkick.redis
class Searchkick::BulkReindexJob
  concurrency 3
end

Product.reindex(async: true, refresh_interval: "30s")
Product.search_index.promote(index_name, update_refresh_interval: true)

Searchkick.redis = ConnectionPool.new { Redis.new }

class Product < ApplicationRecord
  searchkick callbacks: :queue
end

Searchkick::ProcessQueueJob.perform_later(class_name: "Product")
Product.search_index.reindex_queue.length

class Bussiness < ApplicaionRecord
  searchikick routing: true
  def search_routing
    city_id
  end
end

Bussiness.search "ice cream", routing: params[:city_id]

class Product < ApplicationRecord
  def search_data
    {
      name: name
    }.merge(search_prices)
  end
  def search_prices
    {
      price: price,
      sale_price: sale_price
    }
  end
end

Product.reindex{:search_prices}

class ReindexConversionsJob < ApplicationJob
  def perform(class_name)
    {
      name: name
    }.merge(search_conversions)
  end
  def search_conversions
    {
      conversions: Rails.cache.read("search_conversions:#{self.class.name}:#{id}") || {}
    }
  end
end

class ReindexConversionsJob < ApplicationJob
  def perform(class_name)
    recently_converted_ids =
      Searchjoy::Search.where("convertable_type = ? AND converted_at > ?", class_name, 1.days.ago)
      .order(:convertable_id).uniq.pluck(:convertable_id)
    recently_converted_ids.in_groups_of(1000, false) do |ids|
      conversions = 
        Searchjoy::Search.where(convertable_id: ids, convertalbe_type: class_name)
        .group(:convertable_id, :query).uniq.count(:user_id)
      conversions_by_record = {}
      conversions.each do |(id, query), count|
        (conversions_by_record[id] ||= {})[query] = count
      end
      conversions_by_record.each do |id, conversions|
        Rails.cache.write("search_conversions:#{class_name}:#{id}", conversions)
      end
      class_name.constantize.where(id: ids).reindex(:search_conversions)
  end
end


ReindexConversionsJob.perform_later("Product")

class Product < ApplicationRecord
  searchkick mappings: {
    product: {
      properties: {
        name: {type: "keyword"}
      }
    }
  }
end

class Product < ApplicationRecord
  searchkick merge_mappings: true, mappings: {...}
end

products = Product.search body: {query: {match: {name: "milk"}}}

products.response

products = Product.search "milk", body_options: {min_score: 1}
products =
  Product.search "apples" do |body|
    body[:min_score] = 1
  end

Searchkick.client

products = Product.search("snacks", execute: false)
coupons = Coupon.search("snacks", execute: false)
Searchkick.multi_search([products, coupons])

Searchkick.search "milk", index_name: [Product, Category]

where: {_or: {{_type: "product", in_stock: true}, {_type: "category", active: true}}}

indices_boost: {Category => 2, Product => 1}

User.search "san", fileds: ["address.city"], where: {"address.zip_code" => 12345}

product = Product.find(1)
product.reindex

Product.where(store_id: 1).reindex

store.products.reindex
Product.search_index.clean_indices

class Product < ApplicationRecord
  searchkick settings: {number_of_shards: 3}
end

class Product < ApplicationReocrd
  snearchkick index_name: "products_v2"
end

class Product < ApplicaitonRecord
  snearchkick index_name: -> { "#{name.tableize}-#{18n.locale}" }
end

class Product < ApplicationRecord
  snearchkick index_prefix: "datakick"
end

Searchkick.index_prefix = "datakick"
Product.search("bannna", conversions_term: "organic banana")

class Product < ApplicationRecord
  has_many :searches, class_name: "Searchjoy::Search"
  searchkick conversions: ["unique_user_conversions", "total_conversions"]
  def search_data
    {
      name: name,
      unique_user_conversions: searches.group(:query).uniq.count(:user_id),
      # {"ice cream" => 234, "chocolate" => 67, "cream" => 2}
      total_conversions: searches.group().count
      # {"ice cream" => 412, "chocolate" => 117, "cream" => 6}
    }
  end
end

Product.search("banana")
Product.search("banana", conversions: "total_conversions")
Product.search("banana", conversions: false)

Searchkick.timeout = 15
Searchkick.search_timeout = 3
Searchkick.search_method_name = :lookup
Searchkick.queue_name = :search_reindex
Product.search "milk", includes: [:brand, :stores]

Searchkick.search("*", index_name: [], model_includes: {Product => [], S})
Product.search "", scope_results: ->(r) { r.with_attached_images }

class Product < ApplicationRecord
  searchkick default_fields: [:name]
end

class Product < ApplicationRecord
  searchkick special_characters: false
end

class Product < ApplicationRecord
  searchkick stem: false
end

class Product < ApplicationRecord
  searchkick similarity: "classic"
end

class Product < ApplicationRecord
  searchkick case_sensitive: true
end

class Product < ApplicationRecord
  searchkick batch_size: 200
end

Product.reindex(import: false)

class Product < ApplicationRecord
  def search_document_id
    custom_id
  end
end

products = Product.search()
products.each {...}

Product.search()

Searchkick.model_options = {
  batch_size: 200
}

class Product < ApplicationRecord
  searchkick callbacks: false
  after_commit :reindex, if: -> (model) { model.previous_changes.key?("name") }
end

Product.search "api", misspellings: {prefix_length: 2}
Product.search "ah", misspellings: {prefix_length: 2}

# test/test_helper.rb
Product.reindex
Searchkick.disable_callbacks

class ProductTest < Minitest::Test
  def setup
    Searchkick.enable_callbacks
  end
  def teardown
    Searchkick.disable_callbacks
  end
  def test_search
    Product.create!(name: "Apple")
    Product.search_index.refresh
    assert_equal ["Apple"], Product.search("apple").map(&:name)
  end
end

# spec/spec_helper.rb
RSpec.configure do |config|
  config.before(:suite) do
    Product.reindex
    Searchkick.disable_callbacks
  end
  config.around(:each, search: true) do |example|
    Searchkick.callbacks(true) do
      example.run
    end
  end
end

describe Product, search: true do
  it "searches" do
    Product.create!(name: "Apple")
    Product.search_index.refresh
    assert_equal ["Apple"], Product.search("apple").map(&:name)
  end
end

FactoryBot.define do
  factory :product do
    trait :reindex do
      after(:create) do |product, _evaluator|
        product.reindex(refresh: true)
      end
    end
  end
end
FactoryBot.create(:product, :some_trait, :reindex, some_attribute: "foo")

Searchkick.index_suffix = ENV["TEST_ENV_NUMBER"]

class Product < ApplicationRecord
  searchkick _all: false, default_fields: [:name]
end

class Product < ApplicationRecord
  def search_data
    {
      all: [name, size, quantity].join(" ")
    }
  end
end

product.save!
Product.search_index.refresh

class Product < ApplicationRecord
  searchkick settings: {number_of_shards: 1}
end

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
-----

https://github.com/ankane


```
```

```ruby
```

```
```


