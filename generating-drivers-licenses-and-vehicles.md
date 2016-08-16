## Modeling Drivers, License, and Vehicles
#### Creating Drivers
Generate a driver model and migration:

``` bash
# bash
rails g model driver first_name last_name birth_date:datetime height:integer weight:integer hair:string eyes:string sex:string
```

This command will generate:

``` ruby
# db/migrate/2016050300000001_create_drivers.rb
class CreateDrivers < ActiveRecord::Migration
  def change
    create_table :drivers do |t|
      t.string :first_name
      t.string :last_name
      t.datetime :birth_date
      t.integer :height
      t.integer :weight
      t.string :hair
      t.string :eyes
      t.string :sex

      t.timestamps null: false
    end
  end
end
```

and an empty model that inherits from ActiveRecord::Base (in other words, it knows how to talk to our `drivers` table):

```
# app/models/driver.rb
class Driver < ActiveRecord::Base
end
```

If we're happy with how our migration looks, and we may "run" our pending migration:

``` bash
rake db:create # if you haven't already done so
rake db:migrate:status # to see what migrations are "up" vs "down" (pending)
rake db:migrate
```

This will update our schema to version `2016050300000001`:

```ruby
# db/schema.rb
# NEVER EVER MODIFY THIS FILE DIRECTLY!!!!

# ...

ActiveRecord::Schema.define(version: 2016050300000001) do

  enable_extension "plpgsql"

  create_table "drivers", force: :cascade do |t|
    t.string   "first_name"
    t.string   "last_name"
    t.datetime "birth_date"
    t.integer  "height"
    t.integer  "wt"
    t.string   "hair"
    t.string   "eyes"
    t.string   "sex"
    t.datetime "created_at", null: false
    t.datetime "updated_at", null: false
  end

end

```

#### Creating Licenses
Generate a license model and migration:

```bash
rails g model license driver:references license_number:string expiration:datetime donor:boolean
```

Which is roughly the same as doing:
```bash
rails g model license driver_id:integer:index license_number expiration:datetime donor:boolean
```
This command will generate:

```ruby
# db/migrate/201605030000000002_create_licenses

class CreateLicenses < ActiveRecord::Migration
  def change
    create_table :licenses do |t|
      t.references :driver, index: true, foreign_key: true    # This is actually creating `driver_id` (integer), our foreign key.
      t.string :license_number
      t.datetime :expiration
      t.boolean :donor, default: false

      t.timestamps null: false
    end
  end
end

```

and an empty model that inherits from ActiveRecord::Base (in other words, it knows how to talk to our `licenses` table):

```ruby
# app/models/license.rb
class License < ActiveRecord::Base
end
```

If we're happy with how our migration looks, we may run our migration:

``` bash
rake db:create # only if you haven't done so already
rake db:migrate:status # to see what migrations are "up" vs "down" (pending)
rake db:migrate
```
This will update our schema to version `201605030000000002_create_licenses.rb`:

```ruby
# db/schema.rb
# NEVER EVER MODIFY THIS FILE DIRECTLY!!!!

# ...

ActiveRecord::Schema.define(version: 201605030000000002) do

  # ...

  create_table "licenses", force: :cascade do |t|
    t.integer  "driver_id"
    t.string   "license_number"
    t.datetime "expiration"
    t.boolean  "donor",          default: false
    t.datetime "created_at",                     null: false
    t.datetime "updated_at",                     null: false
  end

  add_index "licenses", ["driver_id"], name: "index_licenses_on_driver_id", using: :btree

end
```

#### Creating Vehicles
Generate a vehicle model and migration:

```bash
rails g model vehicle make:string model:string year:integer license_plate:string driver:references
```

This command will generate a new migration...

```ruby
# db/migrate/201605030000000003_create_vehicles.rb

class CreateVehicles < ActiveRecord::Migration
  def change
    create_table :vehicles do |t|
      t.string :make
      t.string :model
      t.integer :year
      t.string :license_plate
      t.references :driver, index: true, foreign_key: true

      t.timestamps null: false
    end
  end
end

```

and an empty model that inherits from ActiveRecord::Base (in other words, it knows how to talk to our `vehicles` table):

```ruby
class Vehicle < ActiveRecord::Base
end
```


If we're happy with how our migration looks, we may run our migration:

``` bash
rake db:create # only if you haven't done so already
rake db:migrate:status # to see what migrations are "up" vs "down" (pending)
rake db:migrate
```
This will update our schema to version `201605030000000003_create_vehicles.rb`:

```ruby
# db/schema.rb
# NEVER EVER MODIFY THIS FILE DIRECTLY!!!!

# ...

ActiveRecord::Schema.define(version: 201605030000000003) do

  # ...

  create_table "vehicles", force: :cascade do |t|
    t.string   "make"
    t.string   "model"
    t.integer  "year"
    t.string   "license_plate"
    t.integer  "driver_id"
    t.datetime "created_at",    null: false
    t.datetime "updated_at",    null: false
  end

  add_index "vehicles", ["driver_id"], name: "index_vehicles_on_driver_id", using: :btree

```

#### Associating drivers and licenses (1:N)

In order to associate our models (and clue them in to the foreign key in the `licenses` table!) we need to create model-level association.

```ruby
# app/models/driver.rb
class Driver < ActiveRecord::Base
  has_one :license
end
```

```ruby
# app/models/license.rb
class License < ActiveRecord::Base
  belongs_to :driver
end
```

With this in place, we should be able to query in both direction. From a driver to their license, and from a license to the corresponding driver.

From inside the rails console we should be able to ask questions like:

```ruby
# rails console
> Driver.new.license # returns nil, but SHOULD NOT THROW ERROR
> License.new.driver # returns nil, but SHOULD NOT THROW ERROR
```

#### Associating drivers and vehicles (N:N)
Associating Drivers and Vehicles is a bit more tricky. A driver can have many vehicles, and a vehicle can have many drivers. We need to create an additional table to make this possible.

Although we could create a migration that explicity creates foreign keys for each of our tables (`vehicle_id:integer:index`, `driver_id:integer:index`). This is such a common operation that rails knows how to do it for you if and only if you name your migration correctly:

``` bash
rails g migration create_join_table_drivers_vehicles drivers vehicles
# or
rails g migration CreateJoinTableDriversVehicles drivers vehicles
```

This will create a migration that looks like this:

```ruby
# db/migrate/
class CreateJoinTableDriversVehicles < ActiveRecord::Migration
  def change
    create_join_table_drivers_vehicles :drivers, :vehicles do |t|
      t.index [:driver_id, :vehicle_id]
      t.index [:vehicle_id, :driver_id]
    end
  end
end

```

If we're happy with how our migration looks, we may run our migration:

``` bash
rake db:create # only if you haven't done so already
rake db:migrate:status # to see what migrations are "up" vs "down" (pending)
rake db:migrate
```
This will update our schema to version `201605030000000004_create_licenses.rb`:

```ruby
# db/schema.rb
# NEVER EVER MODIFY THIS FILE DIRECTLY!!!!

# ...

ActiveRecord::Schema.define(version: 201605030000000004) do

  # ...
  
  create_table "drivers_vehicles", id: false, force: :cascade do |t|
    t.integer "driver_id",    null: false
    t.integer "vehicle_id", null: false
  end

  add_index "drivers_vehicles", ["driver_id", "vehicle_id"], name: "index_drivers_vehicles_on_driver_id_and_vehicle_id", using: :btree
  add_index "drivers_vehicles", ["vehicle_id", "driver_id"], name: "index_drivers_vehicles_on_vehicle_id_and_driver_id", using: :btree

end
```

We need to specify that our `Driver` and `Vehicle` models are "associated" so that ActiveRecord knows to look at the `driver_id` and `vehicle_id` columns and understands how they relate to the `drivers` and `vehicles` tables. This is called a ["has_and_belongs_to_many" Association](http://guides.rubyonrails.org/association_basics.html#the-has-and-belongs-to-many-association)

```
# app/models/driver.rb
class Driver < ActiveRecord::Base
  has_one :license
  has_and_belongs_to_many :vehicles
end
```

```
# app/models/vehicle.rb
class Vehicle < ActiveRecord::Base
  has_and_belongs_to_many :drivers
end
```

> Note that this approach uses only *2 models*, even though there are *3 tables*(!). Another approach is to use 3 models, each mapped to its own table, aka a ["has_many :through" Association](http://guides.rubyonrails.org/association_basics.html#the-has-many-through-association)

**This is the moment we've been waiting for!** We can now query our driver's vehicles, and our vehicles drivers (in both directions):

```ruby
# rails c
> reload! # reload our models
> Driver.new.vehicles # returns [], BUT SHOULD NOT ERROR
> Vehicle.new.drivers # returns [], BUT SHOULD NOT ERROR
```

> Pro-Tip: You may need to run `reload!` in your console, and requery your models at this point, to make sure you are working with "fresh" objects (and not old, stale, cached copies).

#### Final Schema

```ruby
ActiveRecord::Schema.define(version: 20160503213901) do

  enable_extension "plpgsql"

  create_table "drivers", force: :cascade do |t|
    t.string   "first_name"
    t.string   "last_name"
    t.datetime "birth_date"
    t.integer  "height"
    t.integer  "wt"
    t.string   "hair"
    t.string   "eyes"
    t.string   "sex"
    t.datetime "created_at", null: false
    t.datetime "updated_at", null: false
  end

  create_table "drivers_vehicles", force: :cascade do |t|
    t.integer "driver_id"
    t.integer "vehicle_id"
  end

  add_index "drivers_vehicles", ["driver_id"], name: "index_drivers_vehicles_on_driver_id", using: :btree
  add_index "drivers_vehicles", ["vehicle_id"], name: "index_drivers_vehicles_on_vehicle_id", using: :btree

  create_table "licenses", force: :cascade do |t|
    t.integer  "driver_id"
    t.string   "license_number"
    t.datetime "expiration"
    t.boolean  "donor",          default: false
    t.datetime "created_at",                     null: false
    t.datetime "updated_at",                     null: false
  end

  add_index "licenses", ["driver_id"], name: "index_licenses_on_driver_id", using: :btree

  create_table "vehicles", force: :cascade do |t|
    t.string   "make"
    t.string   "model"
    t.integer  "year"
    t.string   "license_plate"
    t.integer  "driver_id"
    t.datetime "created_at",    null: false
    t.datetime "updated_at",    null: false
  end

  add_index "vehicles", ["driver_id"], name: "index_vehicles_on_driver_id", using: :btree

  add_foreign_key "drivers_vehicles", "drivers"
  add_foreign_key "drivers_vehicles", "vehicles"
  add_foreign_key "licenses", "drivers"
  add_foreign_key "vehicles", "drivers"
end

```
