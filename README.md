# Heroku CI buildpack: Postgis

- Postgis 2.5.2 for postgresql 11.5.*
- Proj 4.9.2
- Geos 3.7.2
- without raster support!

## How does it work?

- One needs to build docker image using Dockerfile inside `support` dir
- The image will contain two `tar.gz` files: `postgis` and `postgis-dependencies`
- `postgis`: contains precomplied postgis lib, ready to be installed inside `/app/.indyno/vendor/postgresql/` dir
- `postgis-dependencies`: contains precomplied and installed libs: `proj` and `geos`.
- One needs to get files from docker container and move it to S3 bucket (`docs.riskmethods.net`)
- During compliation stage this buildpack will fetch those 2 files from S3 bucket and install it inside the dyno.

## Usage

Just add it to `app.json` definition, like:

```json
 "environments": {
    "test": {
      "buildpacks": [
        { "url":  "https://github.com/riskmethods/heroku-buildpack-ci-postgis" },
        { "url": "heroku/nodejs"},
        { "url": "heroku/ruby" }
      ],
      "env": {  "POSTGRESQL_VERSION": "11.5" },
      "addons": ["heroku-postgresql:in-dyno"]
    }
  }
```

Note: this buildpack should be added before ruby buildpack

## Compiling libs and putting them in S3

```bash
cd support
docker build . -t heroku-postgis
docker run -it  heroku-postgis bash
```

Save container ID, open another terminal window and run:

```bash
docker cp {id-of-container}:/postgis-dependencies.tar.gz .
docker cp {id-of-container}:/postgis.tar.gz .
```

Now you have files inside your machine, go to S3 UI and put them inside `docs.riskmethods.net` bucket.
Don't forget to allow read-all access.
