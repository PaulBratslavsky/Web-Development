# Strapi Custom Registration Controller

```javascript
"use strict";

const emailRegExp =
  /^(([^<>()\[\]\\.,;:\s@"]+(\.[^<>()\[\]\\.,;:\s@"]+)*)|(".+"))@((\[[0-9]{1,3}\.[0-9]{1,3}\.[0-9]{1,3}\.[0-9]{1,3}\])|(([a-zA-Z\-0-9]+\.)+[a-zA-Z]{2,}))$/;

const formatError = (error) => [
  { messages: [{ id: error.id, message: error.message, field: error.field }] },
];

/**
 * User.js controller
 *
 * @description: A set of functions called "actions" for managing `User`.
 */

const _ = require("lodash");
const { sanitizeEntity } = require("strapi-utils");

module.exports = {
  /**
   * Promise to fetch authenticated user.
   * @return {Promise}
   */

  async registerPortfolioUser(ctx) {
    const pluginStore = await strapi.store({
      environment: "",
      type: "plugin",
      name: "users-permissions",
    });

    const settings = await pluginStore.get({
      key: "advanced",
    });

    if (!settings.allow_register) {
      ctx.throw(403, "Registration is disabled");
    }

    const params = {
      ..._.omit(ctx.request.body.input, [
        "confirmed",
        "confirmationToken",
        "resetPasswordToken",
      ]),

      provider: "local",
    };

    if (!params.password) {
      ctx.throw(400, "Please provide a password.");
    }

    if (!params.email) {
      ctx.throw(400, "Please provide an email address.");
    }

    if (
      strapi.plugins["users-permissions"].services.user.isHashed(
        params.password
      )
    ) {
      ctx.throw(
        400,
        "Your password cannot contain more than three times the symbol '$'."
      );
    }

    const role = await strapi
      .query("role", "users-permissions")
      .findOne({ type: settings.default_role }, []);

    if (!role) {
      ctx.throw(500, "The default role is missing.");
    }

    const isEmail = emailRegExp.test(params.email);

    if (isEmail) {
      params.email = params.email.toLowerCase();
    } else {
      ctx.throw(400, "Please provide a valid email address.");
    }

    params.role = role.id;

    params.password = await strapi.plugins[
      "users-permissions"
    ].services.user.hashPassword(params);

    const user = await strapi.query("user", "users-permissions").findOne({
      email: params.email,
    });

    if (user && user.provider === params.provider) {
      ctx.throw(409, { message: "Email is already taken." });
    }

    if (user && user.provider !== params.provider && settings.unique_email) {
      ctx.throw(409, { message: "Email is already taken." });
    }

    const username = await strapi.query("user", "users-permissions").findOne({
      username: params.username,
    });

    if (username && username.provider === username.provider) {
      ctx.throw(409, { message: "Username is already taken." });
    }

    if (!params.firstName) {
      ctx.throw(409, { message: "Please provide your first name" });
    }

    if (!params.lastName) {
      ctx.throw(409, { message: "Please provide your last name" });
    }

    if (!params.companyName) {
      ctx.throw(409, { message: "Please provide your company name" });
    }

    try {
      if (!settings.email_confirmation) {
        params.confirmed = true;
      }

      params.userType = "PORTFOLIO_MANAGER";
      params.firstName = params.firstName;
      params.lastName = params.lastName;
      params.tncStatus = false;

      const user = await strapi
        .query("user", "users-permissions")
        .create(params);

      const portfolioData = {
        name: params.companyName,
        users: [user],
      };

      await strapi.query("portfolio").create(portfolioData);

      const sanitizedUser = sanitizeEntity(user, {
        model: strapi.query("user", "users-permissions").model,
      });

      if (settings.email_confirmation) {
        try {
          await strapi.plugins[
            "users-permissions"
          ].services.user.sendConfirmationEmail(user);
        } catch (err) {
          return ctx.badRequest(null, err);
        }

        return ctx.send({ user: sanitizedUser });
      }

      const jwt = strapi.plugins["users-permissions"].services.jwt.issue(
        _.pick(user, ["id"])
      );

      return ctx.send({
        jwt,

        user: sanitizedUser,
      });
    } catch (err) {
      const adminError = _.includes(err.message, "username")
        ? { message: "Username already taken" }
        : { message: "Email already taken" };

      ctx.throw(409, adminError);
    }
  },
};
```
