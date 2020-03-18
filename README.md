A Drupal8 build for randall library

## running the dev box

All the docker-compose commands must be run from this folder.

1) change the passwords in the file ".env"
2) copy {path to drupal8_sandbox_db.sql} to ./db_shared/
3) build the app & watch for completion

```
docker-compose up --build -d
docker-compose exec webapp chown -R www-data:www-data /drupal_sync /drupal_app/web/modules /drupal_app/web/themes
docker-compose exec webapp drush cache-rebuild
docker-compose exec webapp drush config-import -y
docker-compose logs -f
```

   wait until the db container logs say "ready for connections".  Exit log screen with `Ctrl-C`.
   check that the database loaded by looking at the site in a browser.  It will error out until the database finishes loading.

See the app at localhost:5150

See the phpmyadmin at localhost:5151

## stopping the dev box

```
ctrl-C
-and/or-
docker-compose stop 
```

## editing a theme file

Revise the files in drupal8_themes.  It is a local folder synced to the container's /drupal_app/web/themes/contrib

## editing a module

Revise the files in drupal8_modules.  It is the home to our custom modules.  The folder syncs to the container's /drupal_app/web/modules

## exporting config changes to drupal_sync/:

`docker-compose exec webapp drush config-export -y`

## importing config settings from drupal_sync/:

`docker-composer exec webapp drush config-import -y`

## making changes to db & pushing to production:

Reasoning:  Full production db dump is at some permanent file location.  Copy the production sql dump to db_shared, do only revisions to drupal that you want on production.  When finished, we will dump the database to an sqldump.

Then the sqldump replaces the full production dump in the permanent location & it also gets ingested to production mysql.  Ingest to production is the same as ingest to test.

## exporting your local database

1) `docker-compose exec webapp drush sql-dump --result-file=/{todays_date}.sql`
1) copy the file from the container to the host os using `docker cp /{todays_date}.sql .`

inside server (on maintenance mode):

`cp {production sql dump} ./db_shared
docker-compose down db && docker-compose up db`

## rebuilding the drupal cache

`docker-compose exec webapp drush cache-rebuild`
⋅⋅⋅It's a good habit -- resolves most problems.

## wiping the containers and starting clean:

```
docker-compose down
docker volume prune
docker network prune
docker system prune
docker-compose up --build
```

## some commands

docker 

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

    stop    {halt the containers but keeps them intact}

    down    {halt & remove the containers,
             preserves volumes and images}

    exec container_name /bin/bash {or other program from inside the container}

    logs     {show logs}

        -f  {follow the log tail}
