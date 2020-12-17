# care-quality-dashboard

## Project Structure

This is a Next.js app:

- frontend pages belong in [`./pages/`](./pages)
- react components belong in [`./components/`](./components) and should be re-exported in [`./components/index.js`](./components/index.js)
- backend API routes belong in [`./pages/api/`](./pages/api)
- static files belong in [`./public`](./public)

Prettier and ESLint enforce code style. There is a pre-commit hook to auto-format code.

## Development

- Run `npm install` to make sure you have all dependencies installed
- Run `npm run dev` to start the web-app in development mode on `localhost:3000` -- hot reloading should be enabled by default
- Run `npm run build` to build the project (although, only really needed on deployment)

You also need some secrets. Make a `.env` file in the project root, replacing `PASSWORD` with an appropriate password. You could just use `dev` if running locally.

```bash
POSTGRES_PASSWORD=PASSWORD
DATABASE_URL="postgresql://nhsw-self-assessment:PASSWORD@localhost:5432/nhsw_self_assessment"

KEYCLOAK_USER=admin
KEYCLOAK_PASSWORD=PASSWORD

BASE_AUTH_URL="http://localhost:8080/auth/realms/nhsw_self_assessment/protocol/openid-connect"
CLIENT_ID=nhsw_self_assessment_web

NEXTAUTH_URL="http://localhost:3000"
```

If you need to use the login system or database, you'll also need to start the Docker containers for our database (PostgreSQL) and login system (Keycloak).

At the moment, you need a secret Realm file for Keycloak. Get the `nhsw_self_assessment_realm.json` secret file from Shubham, and place this in your project root.

To do this, make sure you have the latest version of [Docker](https://docs.docker.com/get-docker/) and [Docker Compose](https://docs.docker.com/compose/install/) installed.

Inside the project root, run `docker-compose up -d`. This will start the database and keycloak containers. If you run without `-d`, then it won't run in the background, and you'll need a new terminal shell to run the web-app.

Run `npm run dev` in the project root after the Docker containers have started to start the web-app on port 3000.

Note: if you're doing this for the first time, your database will be empty and you need to seed it:

1. Run `docker ps`, and find the ID of the `database` container
2. Run `docker exec -it ID /bin/sh`, replacing `ID` with the container's ID. This will open a shell inside the Docker container
3. Run `cd prisma` and then `node seed.js`. Note, if this gives you errors, you may need to add the `module: true` property to the `package.json` file **within** the container (using e.g. `vim` or `nano`)
4. Exit the container by pressing Ctrl+d

Now your database should be running with some initial data!

If you need to make some test users to play around with locally, you'll can use the Keycloak web interface:

1. Go to <http://localhost:8080> and login to the administrative console
2. Use `admin` and the password you set in your `.env` file as the login details
3. On the left hand side, find `Users`
4. Add a new user, give it a name like `clinician` or `department`, etc. (make sure it **IS** activated)
5. Once you add the user, go back to the user and edit the "roles": add clinician/department/etc to it
6. Also, set a password (make sure it's **NOT** temporary)
7. Repeat for any other users you need

## Database

We use [Prisma](https://www.prisma.io/) as an ORM to interact with a [PostgreSQL database](http://postgresql.org/).

The SQL Schema can be found in [`schema.sql`](./schema.sql).

Make sure you have a `.env` file in [`./prisma`](./prisma) with the `DATABASE_URL` property set to the [PostgreSQL connection URL](https://www.prisma.io/docs/concepts/database-connectors/postgresql).

To introspect the PostgreSQL database and generate a Prisma Schema, run the following command:

```bash
npx prisma introspect
```

This should generate the [`./prisma/schema.prisma`](./prisma/schema.prisma) file, based on the SQL database schema.

Finally, to generate the Prisma Client, for use in our code, run:

```bash
npx prisma generate
```

This will generate the database-specific Prisma Client into `./node_modules/@prisma/client`.

See the [Prisma docs](https://www.prisma.io/docs/) for more detailed information and the API reference.
