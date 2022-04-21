# Content

- Front: a nginx server to 
  - serve frontend assets
  - forward requests to the backend
- Back: a Postgrest backend
- Data: a PostgreSQL server

# Structure

    ├── db
    │   ├── data             # Database data (need root rights to see files)
    │   ├── init             # Custom scripts to be run on the 1st DB startup
    │   └── runtime          # Custom migrations
    ├── docker-compose.yml   # Structure the whole application
    ├── nginx
    │   ├── certs            # SSL certificates for localhost
    │   └── nginx.conf
    ├── web                  # Web project root path
    │   └── dist             # Build path used by nginx to serve web assets
    └── README.md
    

# Exposes

- http://localhost redirecting automatically to 443 port
- https://localhost with following routes:
  - `/api`: postgrest API on /api path
  - `/` all assets available in web/dist directory

# Installation
## Create and install certificate

    cd nginx/cert
    mkcert localhost
    mkcert install

## Start the stack

    POSTGRES_PASSWORD=AZERTY docker-compose up

# How to?

## List APIs

Postgrest API are here: https://localhost/api

## Add a web project

You can use the web framework of your choice.

The only thing you need is to put your built web assets into the `web/dist` directory.

### Example with vite and react

Here we show how to integrate a [reactJS](http://reactjs.org) project using [viteJs](https://vitejs.dev):

     # Create the project
     yarn create vite web --template react
     
     # Build it
     cd web
     
     # Download dependencies
     yarn

You can use the dev mode and have a server outside of the docker fullstack with:

    yarn dev # exposes http://localhost:3000

Or you can build the web application to let nginx serve the assets:

     yarn build # access your application through https://localhost

Don't forget to rerun `yarn build` every time you want to update the assets for nginx.

## Connect to database

  docker exec -it docker-fullstack_db_1 psql -h localhost postgres postgres

## Add a SQL script to be run on the first database start

Put your sql script in `db/init` directory

## Run a SQL script from db/shared

Place your `myscript.sql` file in `db/runtime` and run:

    docker exec -it docker-fullstack_db_1 psql -h localhost postgres postgres -f /runtime-scripts/myscript.sql

## Reset totally the database

  sudo rm -rf db/data/pgdata

## Example

Create a file `db/runtime/books.sql` with:
  
    CREATE TABLE IF NOT EXISTS books (title text);

Run this script:
  
    docker exec -it docker-fullstack_db_1 psql -h localhost postgres postgres -f /runtime-scripts/books.sql

Postgrest API are now up to date, and contain the new books entity: https://localhost/api

But for now, there are no books: https://localhost/api/books

You can create a book with:

    curl -X POST https://localhost/api/books -H "Content-Type: application/json" -d '{"title": "My title"}'

You can list the new book with: https://localhost/api/books

# Documentation

- nginx with SSL: https://betterprogramming.pub/docker-powered-web-development-utilizing-https-and-local-domain-names-a57f129e1c4d


# TODO
- explain front usage
