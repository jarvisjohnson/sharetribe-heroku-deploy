### Deploying to Heroku

 - This will deploy a fresh install of Sharetribe onto a Heroku server.

 - You will need:

 * a [Heroku account](https://signup.heroku.com/)

 * a [local install of Sharetribe](https://github.com/sharetribe/sharetribe/#installation) which you'll push your code from with Git

 * $ - it is not free to host on Heroku, and can run up around $100/month for a basic install very quickly.  Digital Ocean can be [much cheaper](http://bitbetter.se/moving-from-heroku-to-digitalocean/), but I haven't gotten around to [switching](https://www.digitalocean.com/community/tutorials/how-to-use-mysql-with-your-ruby-on-rails-application-on-ubuntu-14-04) yet!

 Notes: 

 * Sharetribe uses a MySQL database.  Rails standardly uses SQlite, and Heroku expects a PostgreSQL db.  So you'll need the cleardb Heroku addon to allow use of MySQL.

1. Deploy the app to heroku following heroku normal instructions (add link to  heroku help)

1. Set heroku environment variables

	Make sure all the options in `config.yml` are properly set then run:
	
		bundle exec rake heroku:config
		
	Copy, paste and run the generated command	
	
1. Remove postgres addon

        heroku addons:destroy heroku-postgresql

1. Addons: MemCachier (free) and SSL ($20)
  
        heroku addons:create memcachier:dev
        heroku addons:create ssl:endpoint
        
1. Addons: New Relic

        heroku addons:create newrelic:wayne
        
	Open the addon by running
	
		heroku addons:open newrelic
 
 	Click on the top-right dropdown menu and select "Account Settings". On your account page, copy your License key.
 	Open `config/newrelic.yml` to set `license_key` variable value to your license.
 	
       
1. Addon: Flying Sphinx ($55)

        heroku addons:create flying_sphinx:ceramic

    > Ceramic plan is needed for Delta indexes. If you can manage without Delta Indexing, smaller plan is also ok
        

1. Addon: MySQL

        heroku addons:create cleardb:ignite
        
    Now get your database url by running
    
    	heroku config:get CLEARDB_DATABASE_URL
    
    Copy the value of CLEARDB_DATABASE_URL returned **and CHANGE the adapter** from `mysql://` to `mysql2://` (there's a **2** there).
    Then set the value of DATABASE_URL environment variable.

        heroku config:add DATABASE_URL='mysql2://{the rest of your connection string}'

    And initialize your database
    
        heroku run bundle exec rake db:schema:load
			
1. Addon: Heroku scheduler

        heroku addons:create scheduler:standard
        
    Open the scheduler
    
    	heroku addons:open scheduler

    And add the following jobs

	| Job                                                      | Frequency |
	|----------------------------------------------------------|:---------:|
	| flying-sphinx index                                      |   hourly  |
	| rails runner "CommunityMailer.deliver_community_updates" |   Daily   |
	| rake sharetribe:delete_expired_auth_tokens               |   Daily   |
        
        
#### Recomended dyno sizes

- **web:** Standard-2X
- **worker:** Standard-2X