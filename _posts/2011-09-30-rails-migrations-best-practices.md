---
title: "Rails Migrations Best Practices"
date: 2011-09-30
author: Jonathan Owens
---

The City is a very long-lived Rails application and as such has accumulated over 600 migrations. We've had enough of them fail in enough interesting ways to learn some rules to follow when writing and applying them.

## One Atomic Change Per Migration

Migrations create a schema version that your database applies atomically. That version should specify **one** reversible change to the database. Generally speaking this means one `add_column`, `create_table`, or `add_index` call per migration. This way if any of your migrations fail you can roll back one and only one migration with a `db:rollback`. Remember, just because you tested a migration with a dozen `add_column` calls doesn't mean it will actually apply in your live site - timeouts, connection errors, and more can all happen and you don't want to have to do SQL tricks to get a half-applied migration to complete.

While it is conceptually nice to bundle an entire feature's worth of database changes into its associated migration - say you're adding a `Story` model and you'd like to do something like this:

```ruby
class CreateStoryTable < ActiveRecord::Migration
  def self.up
    create_table :stories do |t|
      t.string  :title
      t.integer :journal_id
      t.integer :user_id
      t.integer :account_id
      t.timestamps
    end

    add_index :stories, [:user_id, :account_id, :journal_id]

    add_column :journals, :stories_count, :integer
  end

  def self.down
    remove_index :stories, [:user_id, :account_id, :journal_id]
    drop_table :stories
    remove_column :journals, :stories_count
  end
end
```

This is a **bad idea**. While it's conceptually consistent, if that `add_column` call fails, you'll be in a half-migrated state, where the database has been modified but the schema version has not. The database will want to run this migration again the next time you run `db:migrate`, but guess what? The table is already there, and the migration will fail once again.

## Atomicity Must Be Preserved

Ok, so we have to finish running this migration. You could run a `db:migrate:redo`, right? Sure, except redo is going to run the down migration, and the first thing it does is remove an index which was never created. You can't move the `remove_index` line down below the `drop_table` call because the index won't be there when the table is gone.

Let's rewrite this migration to do the 3 things it needs to do in separate migrations:

```ruby
class CreateStoryTable < ActiveRecord::Migration
  def self.up
    create_table :stories do |t|
      t.string  :title
      t.integer :journal_id
      t.integer :user_id
      t.integer :account_id
      t.timestamps
    end
  end

  def self.down
    drop_table :stories
  end
end
```

```ruby
class AddStoriesIndexOnUserAccountAndJournal < ActiveRecord::Migration
  def self.up
    add_index :stories, [:user_id, :account_id, :journal_id]
  end

  def self.down
    remove_index :stories, [:user_id, :account_id, :journal_id]
  end
end
```

```ruby
class AddStoriesCountToJournals < ActiveRecord::Migration
  def self.up
    add_column :journals, :stories_count, :integer
  end

  def self.down
    remove_column :journals, :stories_count
  end
end
```

Much better. Now if you deploy this whole set of migrations and need to roll back the entire feature, you can just use `rake db:rollback STEP=3` (if you want, but in the case of this example, you might not have to - see below), or if any individual one fails you can roll back only the changes it made.

## Code-Safety Within The Release

As much as possible, avoid performing migrations that break compatibility with currently running code. Perform `remove_column` and `drop_table` migrations one release after the code change that drops dependency on them. If you're making a structure change that breaks compatibility, you're best off shipping a compatibility change first and migrating later. Pedro Belo at Heroku wrote the [definitive guide](http://pedro.herokuapp.com/past/2011/7/13/rails_migrations_with_no_downtime/) to this. If you're not as sensitive to scheduled downtime, by all means take it first and ask questions later.

## Time Your Tests

You should of course be testing your releases against production data before deploying them, and when doing so you must verify not only data correctness ([with tests](http://blog.carbonfive.com/2011/01/27/start-testing-your-migrations-right-now/) if you can) but change timeliness. If you have a migration that takes 20 minutes with production data and performs breaking changes, you had better know that in advance so you can prepare with either downtime or a staggered release.

## No Code In Migrations

I know the Rails docs say it's ok ([with caveats](http://guides.rubyonrails.org/migrations.html#using-models-in-your-migrations)) to use models directly in migrations, but having migrations do too much has been a source of problematic bugs for us in practice. If you have data changes that must be performed as part of the migration, they should be done in a rake task. That way the whole app in its post-migrated state is available to work on, and more importantly, as the size of your dataset grows, you can distribute large dataset transformation operations to a background process like Resque. On a large database, a call like `Product.all.each {...}` could take a very long time, time you could save by parallelizing the work.

This has its caveats too - it complicates release management by creating a separate task to do just to keep data correct, so YMMV. We've done this out of need because we have so much data.

## No Shortcuts

Migrations, despite their simple appearance, are one of the easiest ways to screw yourself up and lose customer data. Back up before applying them, test and time them carefully, and don't get lazy. Your database is not like code, testing it for correctness and rolling back changes is not easy. Love your users by being rabid about keeping their trust, and they will love you back.
