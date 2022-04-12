# Upload Files with Cloudinary

- [ ] set up Strapi
- [ ] install Cloudinary
- [ ] set up env file
- [ ] extend the controller

[reference](https://strapi.io/blog/add-cloudinary-support-to-your-strapi-application)

## Set up Strapi

[Strapi Getting Started Doc's](https://docs.strapi.io/developer-docs/latest/getting-started/quick-start.html)

```bash
npx create-strapi-app@latest my-project --quickstart
```

## Set up Cloudinary

[Cloudinary NPM Package](https://www.npmjs.com/package/@strapi/provider-upload-cloudinary)

```bash
npm install @strapi/provider-upload-cloudinary --save
```

```env

  CLOUDINARY_NAME=cloudinary name
  CLOUDINARY_KEY=cloudinary key
  CLOUDINARY_SECRET=cloudinary secret
  CLOUDINARY_FOLDER=file folder in cloudinary

```

```env
HOST=0.0.0.0
PORT=1337
APP_KEYS=nr5vNSzgCri70qIxKsbS3w==,k3vtlJGX1P3rF4AZ27jDIw==,DspaKZG2lprLvZ6HPVJrIA==,ASalOONMjsgAyuGi41SRog==
JWT_SECRET=391be49a-0209-4c9c-b923-0473a7edd000
API_TOKEN_SALT=8f2ca42711a94773abfc8cf7568f6bab

CLOUDINARY_NAME=dq2cllwgp
CLOUDINARY_KEY=294129236324783
CLOUDINARY_SECRET=QZOtvp51p0PG71dL1wudozdgSAk
CLOUDINARY_FOLDER=strapi
```

```javascript
module.exports = ({ env }) => ({
  upload: {
    config: {
      provider: "cloudinary",
      providerOptions: {
        cloud_name: env("CLOUDINARY_NAME"),
        api_key: env("CLOUDINARY_KEY"),
        api_secret: env("CLOUDINARY_SECRET"),
      },
      actionOptions: {
        upload: {
          folder: env("CLOUDINARY_FOLDER"),
        },
        delete: {},
      },
    },
  },
});
```

Update security setting

```javascript
const strapi_security = {
  name: "strapi::security",

  config: {
    contentSecurityPolicy: {
      useDefaults: true,
      directives: {
        "connect-src": ["'self'", "https:"],
        "img-src": ["'self'", "data:", "blob:", "res.cloudinary.com"],
        "media-src": ["'self'", "data:", "blob:", "res.cloudinary.com"],
        upgradeInsecureRequests: null,
      },
    },
  },
};

module.exports = [
  "strapi::errors",
  strapi_security,
  "strapi::cors",
  "strapi::poweredBy",
  "strapi::logger",
  "strapi::query",
  "strapi::body",
  "strapi::session",
  "strapi::favicon",
  "strapi::public",
];
```

```javascript
const validateUploadBody = require("./validation/upload");

const getService = (name) => {
  return strapi.plugin("upload").service(name);
};

const ACTIONS = {
  read: "plugin::upload.read",
  readSettings: "plugin::upload.settings.read",
  create: "plugin::upload.assets.create",
  update: "plugin::upload.assets.update",
  download: "plugin::upload.assets.download",
  copyLink: "plugin::upload.assets.copy-link",
};

const fileModel = "plugin::upload.file";

module.exports = (coreApi) => {
  coreApi.controllers["admin-api"].uploadFiles = async (ctx) => {
    const {
      state: { userAbility, user },
      request: { body, files: { files } = {} },
    } = ctx;

    const uploadService = getService("upload");

    const pm = strapi.admin.services.permission.createPermissionsManager({
      ability: userAbility,
      action: ACTIONS.create,
      model: fileModel,
    });

    if (!pm.isAllowed) {
      return ctx.forbidden();
    }

    const data = await validateUploadBody(body);
    const uploadedFiles = await uploadService.upload({ data, files }, { user });

    const response = await pm.sanitizeOutput(uploadedFiles, {
      action: ACTIONS.read,
    });

    Promise.all(
      response.map(async (item) => {
        const imageData = {
          name: item.name,
          ext: item.ext,
          mime: item.mime,
          url: item.url,
          provider: item.provider,
          image: item.id,
          imageID: item.id.toString(),
          uploadedBy: user.id,
        };

        await strapi.service("api::document.document").create({
          data: { ...imageData },
        });
      })
    ).catch((err) => console.log(err, "Failed to create document"));

    ctx.body = response;
  };

  return coreApi;
};
```
