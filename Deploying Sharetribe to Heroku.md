### Deploying to Heroku

1. Deploy the app to heroku following heroku normal instructions (add link to  heroku help)

1. Set heroku environment variables

    Make sure all the options in `config.yml` are properly set then run:
	
        bundle exec rake heroku:config
		
	  Copy, paste and run the generated command
	
1. Remove postgres addon

        heroku addons:destroy heroku-postgresql

1. Addons: MemCachier and New Relic
  
        heroku addon:create memcachier:dev
        heroku addon:create newrelic:wayne

1. Addon: MySQL

        heroku addon:create cleardb:ignite

    Copy the value of CLEARDB_DATABASE_URL to DATABASE_URL environment variable

        heroku config:add DATABASE_URL=$(heroku config:get CLEARDB_DATABASE_URL)

1. Addon: Heroku scheduler

        heroku addon:create scheduler:standard

    Add the following jobs

        rake ts:index
        rails runner "CommunityMailer.deliver_community_updates"
        rake sharetribe:delete_expired_auth_tokens
        rake sharetribe:retry_and_clean_paypal_tokens
        rake sharetribe:synchronize_verified_with_ses


1. Addon: Flying Sphinx ($55)

        heroku addons:create flying_sphinx:ceramic

    > Ceramic plan is needed for Delta indexes. If you can manage without Delta Indexing, smaller plan is also ok


1. Addon: SSL ($20)

#### Recomended dyno sizes

- **web:** Standard-2X
- **worker:** Standard-2X