# How to deploy Strapi 4 to digital ocean

## Create a Strapi App

Make sure you have node 14 installed before continiuing.

Use quicks start npx create strapi app to get started.

~~~bash
npx create-strapi-app@latest my-project --quickstart
~~~


Once your app is installed, open it in VS Code and create server settings

~~~node
module.exports = ({ env }) => ({
  connection: {
    client: "postgres",
    connection: {
      host: env("DATABASE_HOST", "127.0.0.1"),
      port: env.int("DATABASE_PORT", 5432),
      database: env("DATABASE_NAME", "strapi"),
      user: env("DATABASE_USERNAME", "strapi"),
      password: env("DATABASE_PASSWORD", "strapi"),
      schema: env("DATABASE_SCHEMA", "public"), // Not required
      ssl: {
        ca: env("DATABASE_CA"),
      },
    },
    debug: false,
  },
});
~~~
