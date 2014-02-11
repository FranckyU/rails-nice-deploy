*This post is copied from my Coderwall account and also visible on [DRY Git and Capistrano process for Rails](https://coderwall.com/p/dipaqg)*

Typically, a Ruby on Rails web-app (or any stack on JVM, .Net, Python, Node.js or PHP) needs to be deployed on two or more remote servers, at least one for staging and one for production.

**The process**

Since we're using git, we have at least three branches

+ *master* where everything that is working and ready for deployment are
+ *staging* for version to be tested on staging server
+ *production* for the production-ready codebase

master ->(merge_to)-> staging ->(merge_to)-> production

This process won't raise any merging issue as the workflow is linear and git will do it by fast forwarding the commit points. All the issues happens when merging various dev branches to master, but that's another story :)

When comes time to deploy frequently, it became annoying to repeat the `git checkout ...` then `git merge` then `git push` then `cap deploy` sequence.

I created a simple bash script to abstract all those steps.

**The time saving scripts**

```bash
#!/usr/bin/env bash

git add . && git commit -am "$1" && git push origin master && git checkout staging && git merge master && git push origin staging && git checkout master && cap staging deploy

```
Save this under *RAILS_ROOT/bin/deploy-staging* and make it executable `cd bin && chmod u+x deploy-staging`

The same for production

```bash
#!/usr/bin/env bash

git checkout production && git merge staging && git push origin production && git checkout master && cap production deploy
```

Save this one under *RAILS_ROOT/bin/deploy-production* and make it executable `cd bin && chmod u+x deploy-production`

**Using it**

Now after working on any dev branch and merged changes to *master*, all tests OK, just cd to *RAILS_ROOT* and deploy with ease by `./bin/deploy-staging "ANY-COMMIT-MESSAGE"` and `./deploy-production`

That's all, have a nice deploy !

# UPDATES

[JAN 2014] We migrated to Rails 4 and Capistrano 3 so the way Capistrano handle deployments changed. No more need to specify `deploy:migrations`, Capistrano is now smart enough to run any pending migration. We updated the script following that

[DEC 2013] Now executing the script is done from RAILS_ROOT directory by calling `./bin/deploy-staging "ANY-COMMIT-MESSAGE"` and `./bin/deploy-production`.