---
title: "Rails routing gotcha: Don't name a route not_found"
date: 2011-02-25
author: Jonathan Owens
---

When moving our app to Bundler, I got the following error:

```text
(__DELEGATE__):2:in `not_found': wrong number of arguments (2 for 0)
(ArgumentError)
from (__DELEGATE__):2:in `send'
from (__DELEGATE__):2:in `not_found'
from /app_dir/vendor/rails/activesupport/lib/active_support/option_merger.rb:20:in `__send__'
from /app_dir/vendor/rails/activesupport/lib/active_support/option_merger.rb:20:in `method_missing'
from /app_dir/config/routes/plaza_routes.rb:5
from /app_dir/vendor/rails/activesupport/lib/active_support/core_ext/object/misc.rb:78:in `with_options'
from /app_dir/vendor/rails/actionpack/lib/action_controller/routing/route_set.rb:51:in `namespace'
```

The secret was a route that looked like this:

```ruby
global.not_found '/not_found', :controller => 'home', :action => 'not_found'
```

The solution? Rename the route to something else. No idea why this wasn't exposed before, but at least there's a solution!
