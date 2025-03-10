# Self Hosting [Plausible](https://plausible.io/docs/self-hosting) on [Fly.io](https://fly.io/) for OxRSE projects

This solution was adapted from [plausible's docs](https://github.com/plausible/hosting) which were designed around a docker-compose solution hosted on a single machine.

# Services

The full plausible stack is composed of 4-5 services but I got by with 3:

- plausible: the main web application
- postgres: the web database, stores accounts etc.
- clickhouse: the analytics database, stores the analytics "event" info.

We neglect the SMTP services and just have no mailer system.

# Fly.io

Fly.io is chosen because it has extremely minimal config, a short and easy to reuse toml file to define the services and deploy directly from the dockerfile without a build pipeline.

# Setup

This is done from memory as there was a bit of experimentation but it might be that something is not quite right, initially I struggled to get the plausible app to talk to the clickhouse service and, as of now, the health check sometimes fails on the deploy of the plausible app (but the app does deploy correctly regardless).

## Configure and Deploy the clickhouse DB

Navigate to the clickhouse directory and run `fly launch --no-deploy`.
This will prompt you: `An existing fly.toml file was found for app []` select "y" when asked: `Would you like to copy its configuration to the new app? (y/N)`

It will follow up with the app creation and give you a summary of your settings before asking `? Do you want to tweak these settings before proceeding? (y/N)`.
Select `y` and it will provide you a link to a web form to fill out. This is not optional, you must adjust the settings.
Choose the appropriate settings, in particular fly will default to your personal org rather than the shared org.

When you hit submit control will be returned to your terminal, finshing the configuration.

Now you can deploy the clickhouse service with `fly deploy`.

## Configure the plausible app and postgres service

Navigate to the plausible directory and repeat the above process to configure the plausible app, we also at this point must set up the postgres service from the list of additional services in the fly.io settings form, this will save on configuration by automatically setting up the postgres service and providing its access details to the plausible app.

> [!IMPORTANT]  
> The default attached postgress services is 3x 2xcore 4096mb ram, this is overkill for the plausible app and should be reduced to the `development` default which is 1x 1xcore 256mb ram.

Before we deploy the plausible app we must set our secrets in a .env file. Do not commit these secrets to a repository.

The important secrets are:

- `SECRET_KEY_BASE` a random string, use 12 random words or something, NEVER reuse this secret key.
- `BASE_URL` the url of your plausible app, e.g. `https://plausible-example.fly.dev`, this will listen to the `POST` requests from individual websites and will send/receive to/from the clickhouse service.
- `DISABLE_REGISTRATION` set to `true` to disable registration on the plausible app, or to `invite_only` to allow registration by invite only, this will only allow accounts to share access to the same deployment rather than allow new deployments. When the app is first deployed, you should set this to `false` and once accounts are created, set it to `true` or `invite_only`, any time you require registering a new email then temporarily revert the changes.

Lastly you must copy the clickhouse services details, you can either do this in the env section of the fly.toml or in the .env file:

- `CLICKHOUSE_DATABASE_URL` is the full database connection string, get this from the now deployed clickhouse db service but use .internal instead of the .fly.dev since the clickhouse container is not configured to accept outside connections but can still listen to the internal fly network: e.g. 'http://plausible-clickhouse-example.internal:8123/plausible_events_db'.
- `CLICKHOUSE_DATABASE` the db name, same as from the db connection string: as far as I know this must be: 'plausible_events_db'.

Set the secrets prior to your first deploy with `flyctl secrets import < [your env file]`.

Finally deploy the plausible app (and postgres service) with `fly deploy`.

Success! You should now be able to access the plausible application, create accounts and start adding websites to track traffic.

> [!NOTE]
> I chose 1gB each for the clickhouse db and the plausible js app purely because the requirements listed by plausible were 2-4gB on a shared machine.
> This might need adjusted?

# Adding websites

To follow maybe...
