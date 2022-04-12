```javascript
const entity = await strapi.services["api::asset.asset"].find({
  id: "2",
});

const entity = await strapi.query("api::asset.asset").findMany({
  id: 3,
});

const entity = await strapi.entityService.findMany("api::asset.asset", args);

const entity = await strapi.db.query.findMany("api::asset.asset", args);
```
