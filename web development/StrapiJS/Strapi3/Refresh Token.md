## Refresh Token

generate a new token example

```javascript
const generateRefreshToken = (user) => {
  return strapi.plugins["users-permissions"].services.jwt.issue(
    { tkv: user.tokenVersion },

    {
      subject: user.id.toString(),
      expiresIn: "1h",
    }
  );
};
```

assign refresh token

```javascript
async assignRefreshToken(ctx) {
	const { user } = ctx.state;
	const token = generateRefreshToken(user);
	ctx.send({ refresh: token })
},
```

refresh token

```javascript

async refreshToken(ctx) {
	const params = _.assign(ctx.request.body);

	try {
		const { tkv, exp, sub } = await strapi.plugins["users-permissions"].services.jwt.verify(params.token);

	if (Date.now() / 1000 > exp) return ctx.badRequest(null, "Expired refresh token");



	const user = await strapi.query("user", "users-permissions").findOne({ id: sub });

// Check here if user token version is the same as in refresh token

// This will ensure that the refresh token hasn't been made invalid by a password change or similar.

if (tkv != user.tokenVersion) return ctx.badRequest(null, "Refresh token is invalid");

ctx.send({Î

jwt: strapi.plugins["users-permissions"].services.jwt.issue(_.pick(user, ["id"])),

refresh: params.renew ? generateRefreshToken(user) : null

});

} catch (e) {

return ctx.badRequest(null, "Invalid token");

}

},

```

revoke token

```javascript
async revoke(ctx) {

const params = _.assign(ctx.request.body);

try {

const { tkv, iat, exp, sub } = await strapi.plugins["users-permissions"].services.jwt.verify(params.token);

if (Date.now() / 1000 > exp) return ctx.badRequest(null, "Expired refresh token");

const user = await strapi.query("user", "users-permissions").findOne({ id: sub });

// Check here if user token version is the same as in refresh token

// This will ensure that the refresh token hasn't been made invalid by a password change or similar.

if (tkv !== user.tokenVersion) return ctx.badRequest(null, "Refresh token is invalid");

await strapi.query("user", "users-permissions").update({ id: sub }, { tokenVersion: user.tokenVersion + 1 });



ctx.send({

confirmed: true,

});

} catch (e) {

return ctx.badRequest(null, "Invalid token");

}

},

```
