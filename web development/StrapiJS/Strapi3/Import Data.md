## Import Data Example

```javascript
const importToCollectionType = async (uid, item) => {
  try {
    await strapi.entityService.create({ data: item }, { model: uid });

    // await strapi.query(uid).create(item);
    return true;
  } catch (error) {
    return false;
  }
};
```
