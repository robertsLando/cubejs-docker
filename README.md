# Metrics stack

Full stack that provides all the tools to store and visualize custom metrics using docker

- mongodb
- node-red: <http://localhost:1880>
- Backend (for apis): <http://localhost:5000>
- MongoDB charts: <http://localhost:9999>

## Install

```bash
git clone https://github.com/robertsLando/metrics-stack.git
cd metrics-stack
docker-compose up
```

## Update

```bash
git pull
docker-compose down && docker-compose up --build
```

`docker-compose up --build` is needed to force re-build of backend apis container

## Credentials

Set up the MongoDB Charts credentials with:
```bash
docker exec -it \
  $(docker container ls --filter name=charts_ -q) \
  charts-cli add-user --first-name "<First>" --last-name "<Last>" \
  --email "<user@example.com>" --password "<Password>" \
  --role "<UserAdmin|User>"
```

Example creating a default admin user:

```bash
docker exec -it \
  $(docker container ls --filter name=charts_ -q) \
  charts-cli add-user --first-name "Admin" --last-name "MongoDB" \
  --email "admin@example.com" --password "mongodb" \
  --role "UserAdmin"
```

## Backend

Uses an Express server to provide apis to store metrics to MongoDB. The only thing that needs to be configured are the database name and the collections with their unique fields. Unique fields are necessary to identify a document uniquely in the DB. This can be done [here](backend/config/app.js)

### APIs

#### POST `/auth`

Used to get a token needed to call other APIs. Expect the a body like:

```json
{
    "key": "your-key-here"
}
```

If everything is ok will return a JSON with a property `token` with the generated token.

Auth protected routes expect an header `x-access-token` or `authorization` with content `Bearer <your-token-here>`

#### POST `/metrics`

**Requires AUTH**

Used to add a metrics to the db. The metrix are added using `upsert`, it means that if the document exists in the db it will be updated, otherwise a new doc will be added. The expected payload is:

```js
{
    "collection": "collectionName", // optional
    "data": [{...}, {...}, ...], // array of documents to add to the collection
}
```

If no collection name is provided the default collection will be used: `metric`. The default collection has a unique field named `id` that the user can use to uniquely identify an entry in the collection

#### POST `/update-db`

**Requires AUTH**

Used in this specific case to update some db collections. No payload is needed
