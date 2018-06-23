### System
ruby 2.4.3p205 (2017-12-14 revision 61247) [x86_64-linux]

Rails 5.1.6

### Application Setup
- `rails new data-migrate-issue-89`
- Add `gem 'data_migrate'` to Gemfile
- `bundle install`
- `rails g migration CreatePeople name address`
- `rails g data_migration AddPeopleData`
- `rails g migration CreateProducts name price`
- `rails g data_migration AddProductsData`
- Comment out the line `raise ActiveRecord::IrreversibleMigration` in data migrations so that they can be reverted.

### Reproducing Issue
Migrate database

    rails db:migrate
    
    == 20180623093551 CreatePeople: migrating =====================================
    -- create_table(:people)
       -> 0.0009s
    == 20180623093551 CreatePeople: migrated (0.0010s) ============================
    
    == 20180623093714 CreateProducts: migrating ===================================
    -- create_table(:products)
       -> 0.0007s
    == 20180623093714 CreateProducts: migrated (0.0008s) ==========================
    

Migrate data

    rails data:migrate
    
    == 20180623093637 AddPeopleData: migrating ====================================
    == 20180623093637 AddPeopleData: migrated (0.0000s) ===========================
    
    == 20180623093724 AddProductsData: migrating ==================================
    == 20180623093724 AddProductsData: migrated (0.0000s) =========================
    

At this point, performing a data rollback succceeds as expected.

    rails data:rollback
    
    == 20180623093724 AddProductsData: reverting ==================================
    == 20180623093724 AddProductsData: reverted (0.0000s) =========================
    

Leaving the migration status like this:

    rails db:migrate:status:with_data
    
    database: db/development.sqlite3
    
     Status    Type    Migration ID   Migration Name
    ------------------------------------------------------------
       up     schema  20180623093551  Create people
       up      data   20180623093637  Add people data
       up     schema  20180623093714  Create products
      down     data   20180623093724  Add products data
      

At this point, performing a `rails data:rollback` will fail with no output on the console and only the following lines in the log file:

    DataMigrate::DataSchemaMigration Load (0.1ms)  SELECT "data_migrations".* FROM "data_migrations"
      ActiveRecord::SchemaMigration Load (0.1ms)  SELECT "schema_migrations".* FROM "schema_migrations"
       (0.1ms)  SELECT "data_migrations"."version" FROM "data_migrations" ORDER BY "data_migrations"."version" ASC
	   

By doing a `db:rollback ` first, it is then possible to successfully do a `data:rollback` again.
