*This post is copied from my Coderwall account and also visible on [DRY Git and Capistrano process for Rails](https://coderwall.com/p/dipaqg)*

Typically, a Ruby on Rails web-app (or any stack on JVM, .Net, Python, Node.js or PHP) needs to be deployed on two or more remote servers, at least one for staging and one for production.

**The process**

Since we're using git, we have at least three branches

+ *master* where everything that is working and ready for deployment are
+ *staging* for version to be tested on staging server
+ *production* for the production-ready codebase

master ->(merge_to)-> staging ->(merge_to)-> production

This process won't raise any merging issue as the workflow is linear and git will do it by fast forwarding the commit points. All the issues happens when merging various dev branches to master, but it's another story :)

When comes time to deploy frequently, it became annoying to repeat the `git checkout ...` then `git merge` then `git push` then `cap deploy` sequence.

I created a simple bash script to abstract all those steps.

**The time saving scripts**

```bash
#!/usr/bin/env bash

cd "$(pwd)/.."

git checkout staging && git merge master && git push origin staging && git checkout master

if [[ ($# -gt 0) ]]; then
  if [[ $1 == "--with-migration" ]]; then
    cap staging deploy:migrations
  fi
else
  cap staging deploy 
fi

```
Save this under *RAILS_ROOT/scripts/deploy-staging* and make it executable `cd scripts && chmod u+x deploy-staging`

The same for production

```bash
#!/usr/bin/env bash

cd "$(pwd)/.."

git checkout production && git merge staging && git push origin production && git checkout master

if [[ ($# -gt 0) ]]; then
  if [[ $1 == "--with-migration" ]]; then
    cap production deploy:migrations
  fi
else
  cap production deploy 
fi
```

Save this one under *RAILS_ROOT/scripts/deploy-production* and make it executable `cd scripts && chmod u+x deploy-production`

**Using it**

Now after working on any dev branch and merged changes to *master*, all tests OK, just cd to *RAILS_ROOT/scripts* and deploy with ease by `./deploy-staging` or `./deploy-staging --wit-migration` and `./deploy-production` or `./deploy-production --with-migration`

That's all, have a nice deploy !