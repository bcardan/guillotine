# Guillotine

Simple URL Shortener hobby kit.

## USAGE

The easiest way to use it is with the built-in memory adapter.

```ruby
# app.rb
require 'guillotine'
module MyApp
  class App < Guillotine::App
    adapter = Guillotine::Adapters::MemoryAdapter.new
    set :service => Guillotine::Service.new(adapter)

    get '/' do
      redirect 'https://homepage.com'
    end
  end
end
```

```ruby
# config.ru
require "rubygems"
require File.expand_path("../app.rb", __FILE__)
run MyApp::App
```

Once it's running, add URLs with a simple POST.

    curl http://localhost:4567 -i \
      -F "url=http://techno-weenie.net"

You can specify your own code too:

    curl http://localhost:4567 -i \
      -F "url=http://techno-weenie.net" \
      -F "code=abc"

## Sequel

The memory adapter sucks though.  You probably want to use a DB.  Check
out the [Sequel gem](http://sequel.rubyforge.org/) for more examples.
It'll support SQLite, MySQL, PostgreSQL, and a bunch of other databases.

```ruby
require 'guillotine'
require 'sequel'
module MyApp
  class App < Guillotine::App
    db = Sequel.sqlite 
    adapter = Guillotine::Adapters::SequelAdapter.new(db)
    set :service => Guillotine::Service.new(adapter)
  end
end
```

You'll need to initialize the DB schema with something like this
(depending on which DB you use):

```
CREATE TABLE IF NOT EXISTS `urls` (
  `url` varchar(255) DEFAULT NULL,
  `code` varchar(255) DEFAULT NULL,
  UNIQUE KEY `url` (`url`),
  UNIQUE KEY `code` (`code`)
) ENGINE=InnoDB DEFAULT CHARSET=utf8;

```

## Riak

If you need to scale out your url shortening services across the cloud,
you can use Riak!

```ruby
require 'guillotine'
require 'riak/client'
module MyApp
  class App < Guillotine::App
    client = Riak::Client.new :protocol => 'pbc', :pb_port => 8087
    bucket = client['guillotine']
    adapter = Guillotine::Adapters::RiakAdapter.new(bucket)
    set :service => Guillotine::Service.new(adapter)
  end
end
```

## Domain Restriction

You can restrict what domains that Guillotine will shorten.

```ruby
require 'guillotine'
module MyApp
  class App < Guillotine::App
    adapter = Guillotine::Adapters::MemoryAdapter.new
    # only this domain
    set :service => Guillotine::Service.new(adapter,
      'github.com')

    # or, any *.github.com domain
    set :service => Guillotine::Service.new(adapter,
      /(^|\.)github\.com$/)
  end
end
```

## Not TODO

* Statistics
* Authentication
