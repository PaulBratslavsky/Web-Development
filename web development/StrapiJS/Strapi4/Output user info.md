src/extensions/user-permissions/ strapi-server.js

```javascript
let cleanOutput = (user) => {
  const {
    password,
    resetPasswordToken,
    confirmationToken,
    updatedBy,
    createdBy,
    ...sanitizedUser
  } = user;
  return sanitizedUser;
};
module.exports = (plugin) => {
  /** * Retrieve state user record. */ plugin.controllers.user.me = async (
    ctx
  ) => {
    if (!ctx.state.user) {
      return ctx.unauthorized();
    }
    const user = await strapi.entityService.findOne(
      "plugin::users-permissions.user",
      ctx.state.user.id,
      {
        populate:
          ctx.request.query.populate == "*"
            ? "*"
            : [
                ctx.request.query.populate == undefined
                  ? ""
                  : ctx.request.query.populate,
              ],
      }
    );
    ctx.body = cleanOutput(user);
  };
  /** * Retrieve users records. */ plugin.controllers.user.find = async (
    ctx
  ) => {
    if (!ctx.state.user) {
      return ctx.Forbidden("Forbidden");
    }
    const users = await strapi.entityService.findMany(
      "plugin::users-permissions.user",
      {
        ...ctx.params,
        populate:
          ctx.request.query.populate == "*"
            ? "*"
            : [
                ctx.request.query.populate == undefined
                  ? ""
                  : ctx.request.query.populate,
              ],
      }
    );
    ctx.body = users.map((user) => cleanOutput(user));
  };
  /** * Retrieve a user record. */ plugin.controllers.user.findOne = async (
    ctx
  ) => {
    const { id } = ctx.params;
    const user = await strapi.entityService.findOne(
      "plugin::users-permissions.user",
      id,
      {
        ...ctx.params,
        populate:
          ctx.request.query.populate == "*"
            ? "*"
            : [
                ctx.request.query.populate == undefined
                  ? ""
                  : ctx.request.query.populate,
              ],
      }
    );
    ctx.body = cleanOutput(user);
  };
};
```
