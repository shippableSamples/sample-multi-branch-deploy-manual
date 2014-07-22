Sample for multiple environments
================================

This sample demonstrates how to setup continuous integration and deployment of an Express+MongoDB
project with multiple Heroku environments. After the build is triggered by the webhook and the
project builds successfully, it is first deployed to the staging environment (separate Heroku app).

Then (likely after QA team performs acceptance tests), it can be manually pushed to the production
environment, which (again) is a separate Heroku application. This behavior is achieved by keeping
an extra branch (here called `production`) that points to the most recent build/commit approved 
(or 'promoted') for the production environment.

Shippable configuration
-----------------------

To be able to deploy to two Heroku applications, we need to add two separate remotes: `staging`, 
`production`. This is done here in `before_install` step:

    - git remote -v | grep ^staging || heroku git:remote --remote staging --app $STAGING_APP_NAME
    - git remote -v | grep ^production || heroku git:remote --remote production --app $PROD_APP_NAME

Then in `after_success` step, we can check on which branch we are currently and choose the correct
remote:

    - >
      [[ $BRANCH = 'production' ]] && REMOTE=production || REMOTE=staging
    - git push -f $REMOTE master

(note that we use `>`, so square brackets don't get interpreted by YAML parser)

Example workflow
----------------

This config can be then leveraged in a workflow like following. 

Assume that the developers work on `master` branch:

    git checkout master
    git add changes
    git commit -m "New shiny feature"
    git push

This commit will then trigger build on Shippable and get deployed to the Heroku app designated by
`STAGING_APP_NAME` environment variable. Now imagine this change gets accepted by QA team, but 
some other changes that arrived on `master` in the meantime are still to be tested:

    git add other_changes
    git commit -m "Another feature"
    git push

To manually deploy the shiny feature to the production, the developer would proceed as follows:

    git branch -f production 0de23d7 # hash of the commit that we want to deploy
    git push -f origin production

Now the build gets launched for `production` branch and the `shippable.yml` config above deploys
this specific commit to the production environment.

For more detailed documentation on Heroku deployment, please see Shippable's continuous
deployment section: http://docs.shippable.com/en/latest/config.html#continuous-deployment

This sample is built for Shippable, a docker based continuous integration and deployment platform.
