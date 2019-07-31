# Heroku CI buildpack: Postgis

- Postgis 2.5.2 for postgresql 9.6.*
- Proj 4.9.2
- Geos 3.7.2
- without raster support!

## How does it works?

- first somebody builds docker image using Dockerfile inside `support` dir
- this image will contains two tar.gz files: `postgis` and `postgis-dependencies`
- `postgis`: contains precomplied postgis lib, ready to be installed inside `/app/.indyno/vendor/postgresql/` dir
- `postgis-dependencies`: contains precomplied and installed libs: `proj` and `geos`.
- then somebody needs to get that files from docker container and move it to S3 bucket (`docs.riskmethods.net`)
- during compliation stage this buildpack will fetch this 2 files from S3 bucket and install it inside dyno. 

## Usage

Just add it to app.json definition, like:

```json
 "environments": {
    "test": {
      "buildpacks": [
        { "url":  "https://github.com/riskmethods/heroku-buildpack-ci-postgis" },
        { "url": "heroku/nodejs"},
        { "url": "heroku/ruby" }
      ],
      "env": {  "POSTGRESQL_VERSION": "9.6" },
      "addons": ["heroku-postgresql:in-dyno"]
    }
  }
```

Note this buildpack should be before ruby buildpack

## Compiling libs and putting them in S3

```bash
cd support
docker build . -t heroku-postgis
docker run -it  heroku-postgis bash
```

(in other terminal:)

```bash
docker cp {id-of-container}:/postgis-dependencies.tar.gz .
docker cp {id-of-container}:/postgis.tar.gz .
```

Now you have files inside your machine, go to S3 UI and put them inside `docs.riskmethods.net` bucket.
Don't forget to allow read-all access.
