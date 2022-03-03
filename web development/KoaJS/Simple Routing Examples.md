## Basic Koa setup example

```javascript
const Koa = require("koa");
const Router = require("koa-router");
const bodyParser = require("koa-bodyparser");
const postRoutes = require("./api/post/config/routes.js");
const app = new Koa();
const router = new Router();

app.use(bodyParser());
app.use(router.routes());

// simple midleware example
app.use(async (ctx, next) => {
  console.log(`
		${ctx.method} 
		${ctx.url}
		${ctx.status}
		${new Date().toLocaleString()}`);

  await next();
});

// generate routes dynamically

postRoutes.forEach((route) => {
  return router[route.method](route.route, async (ctx) =>
    route.controller[route.type](ctx)
  );
});ƒ

const PORT = 4000;
app.listen(PORT);
console.log(`Server is listening on port ${PORT}`);
```

## Koa Routing Examples

```javascript
router.get("/posts", (ctx) => post.find(ctx));
router.get("/posts/:id", (ctx) => post.findOne(ctx));
router.post("/posts", (ctx) => post.create(ctx));
router.delete("/posts/:id", (ctx) => post.delete(ctx));
router.put("/posts/:id", (ctx) => post.update(ctx));

router.get("/comments", (ctx) => {
  ctx.body = comments;
});

router.get("/comments/:id", (ctx) => {
  const { id } = ctx.params;
  const comment = comments.find((comment) => comment.id === id);
  ctx.body = comment;
});

router.get("/users", (ctx) => {
  ctx.body = users;
});

router.get("/users/:id", (ctx) => {
  const { id } = ctx.params;
  const user = users.find((comment) => comment.id === id);
  ctx.body = user;
});
```

## Generate Koa Routing vis json file

````javascript

// generate routes dynamically
postRoutes.forEach((route) => {
	return router[route.method](route.route, async (ctx) =>
		route.controller[route.type](ctx)
);

});```

## Json router file example

```javascript
const post = require("../controllers/post.js");

const postRoutes = [
	{
		route: "/posts",
		controller: post,
		type: "find",
		method: "get",
		policies: []
	},

	{
		route: "/posts/:id",
		controller: post,
		type: "findOne",
		method: "get",
		policies: []
	},

	{
		route: "/posts",
		controller: post,
		type: "create",
		method: "post",
		policies: []
	},
	{
		route: "/posts/:id",
		controller: post,
		type: "delete",
		method: "delete",
		policies: []
	},
	{
		route: "/posts/:id",
		controller: post,
		type: "update",
		method: "put",
		policies: []
	},
];



module.exports = postRoutes;
````

### Koa simple controller examples

```javascript
const services = require("../services/post.js");
const uniqueId = require("uuid").v4;

module.exports = {
  find: function (ctx) {
    const posts = services.find();
    ctx.body = posts;
  },

  findOne: function (ctx) {
    const { id } = ctx.params;
    const post = services.findOne(id);
    ctx.body = post;
  },

  create: function (ctx) {
    const post = ctx.request.body;
    post.id = uniqueId();

    if (post === undefined) ctx.throw(400, "Post is undefined");

    if (!post.userId) ctx.throw(400, "Invalid post: missing userId");

    if (!post.title) ctx.throw(400, "Invalid post: missing title");

    if (!post.content) ctx.throw(400, "Invalid post: missing content");

    services.create(post);
    ctx.body = post;
  },

  delete: function (ctx) {
    const { id } = ctx.params;
    const post = services.delete(id);
    ctx.body = post;
  },

  update: function (ctx) {
    const { id } = ctx.params;
    const post = services.update(id, ctx.request.body);
    ctx.body = post;
  },
};
```

## Koa simple services example

```javascript
module.exports = {
  find: function () {
    return posts;
  },

  findOne: function (id) {
    return posts.find((post) => post.id === id);
  },

  create: function (post) {
    return posts.push(post);
  },

  delete: function (id) {
    posts = [...posts.filter((post) => post.id !== id)];
    return posts;
  },

  update: function (id, post) {
    console.log(id, post);
    posts = [
      ...posts.map((item) => (item.id === id ? { ...item, ...post } : item)),
    ];
    return posts;
  },
};
```
