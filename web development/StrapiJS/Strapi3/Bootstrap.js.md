```javascript
"use strict";

/**

* An asynchronous bootstrap function that runs before

* your application gets started.

*

* This gives you an opportunity to set up your data model,

* run jobs, or perform some special logic.

*

* See more details here: https://strapi.io/documentation/3.0.0-beta.x/concepts/configurations.html#bootstrap

*/

const findAuthenticatedRole = async () => {
  const result = await strapi

    .query("role", "users-permissions")

    .findOne({ name: "Authenticated" });

  return result;
};

const setPermission = async (
  roleId,
  controller,
  action,
  type = "application",
  enabled = true
) => {
  return strapi

    .query("permission", "users-permissions")

    .update({ role: roleId, type, controller, action }, { enabled });
};

const setAllApplicationPermissions = async (roleId) => {
  const role = await findAuthenticatedRole();

  const permissions = await strapi

    .query("permission", "users-permissions")

    .find({ type: "application", role: roleId });

  await Promise.all(
    permissions.map((p) =>
      strapi

        .query("permission", "users-permissions")

        .update({ id: p.id }, { enabled: true })
    )
  );
};

const isFirstRun = async () => {
  const pluginStore = strapi.store({
    environment: strapi.config.environment,

    type: "type",

    name: "setup",
  });

  const initHasRun = await pluginStore.get({ key: "initHasRun" });

  await pluginStore.set({ key: "initHasRun", value: true });

  return !initHasRun;
};

module.exports = async () => {
  const shouldSetDefaultPermissions = await isFirstRun();

  if (shouldSetDefaultPermissions) {
    const role = await findAuthenticatedRole();

    await setAllApplicationPermissions(role.id);

    await setPermission(role.id, "user", "getuserinfo", "users-permissions");

    await setPermission(role.id, "user", "respondtotnc", "users-permissions");

    await setPermission(role.id, "user", "findone", "users-permissions");
  }
};
```
