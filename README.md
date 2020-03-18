A Drupal8 build for randall library

# Running the dev box

1) change the passwords in the file ".env"
2) copy {path to drupal8_sandbox_db.sql} to ./db_shared/
3)
⋅⋅⋅```
docker-compose up --build -d
docker-compose exec webapp chown -R www-data:www-data /drupal_sync /drupal_app/web/modules /drupal_app/web/themes
docker-compose exec webapp drush cache-rebuild
docker-compose logs -f
```
⋅⋅⋅wait until the db container reports "ready for connections".  Exit log screen with `Ctrl-C`.
⋅⋅⋅check that the database loaded by looking at the site in a browser.  It will error out until the database finishes loading.

4)`docker-compose exec webapp drush config-import -y`

See the app at localhost:5000

## To stop the containers

```
ctrl-C
-or-
docker-compose stop 
```

## To wipe the drupal containers and start clean:

```
docker-compose down
docker volume prune
docker network prune
docker system prune
docker-compose up --build
```

## To edit a theme file:

Revise the files in drupal8_themes.  It is a local folder synced to the container's /drupal_app/web/themes/contrib .

## To export config changes to drupal_sync/:

`docker-compose exec webapp drush config-export -y`

## To import config settings from drupal_sync/:

`docker-composer exec webapp drush config-import -y`

## To make changes to db & push to production:

Reasoning:  Full production db dump is at some permanent file location.  Copy the production sql dump to db_shared, do only revisions to drupal that you want on production.  When finished, we will dump the database to an sqldump.

Then the sqldump replaces the full production dump in the permanent location & it also gets ingested to production mysql.  Ingest to production is the same as ingest to test.

## To export your local database

1) `docker-compose exec webapp drush sql-dump --result-file=/{todays_date}.sql`
1) copy the file from the container to the host os using `docker cp /{todays_date}.sql .`

inside server (on maintenance mode):

`cp {production sql dump} ./db_shared
docker-compose down db && docker-compose up db`

# rebuild the drupal cache

`docker-compose exec webapp drush cache-rebuild`
⋅⋅⋅It's a good habit -- resolves most problems.

# some docker commands 

    ps    {show active containers}

        -a      {show all containers}

    volume

        ls      {list}

        prune   {remove inactive volumes}

                - this will destroy the container's persistent data

        rm      {removes specific volume}

docker-compose

    up      {start the containers}

        -d  {in detached mode}
        --build {make the docker image if it doesn't yet exist}

    down    {kill & remove the containers,
             preserves volumes and images}

    exec container_name /bin/bash {or other program from inside the container}

    logs     {show logs}
