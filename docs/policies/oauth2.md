---
title: OAuth 2.0
layout: doc-section
doc-order: 5.35
---

### Description

The OAuth 2.0 policy follows the [RFC-6749 standard][rfc-6749-standard].

Express Gateway plays the role of both resource server and authorization server. In order to use this policy, consumers
must be created and an `oauth2` credential created for them.

The OAuth 2.0 policy will listen on the following global endpoints:

* `/oauth2/authorize`
    * Endpoint to provision authorization codes for the Authorization Code flow, or the access token for the Implicit
    Grant flow. Only POST is supported.

* `/oauth2/token`
    * Endpoint to provision access tokens to support the Client Credentials and Password Credentials Grant flows.
    Only POST is supported.


When a client has been authenticated and authorized, Express Gateway will append property and authentication headers to
the request before proxying it to the downstream service, so that you can identify the consumer and the end-user in
your service:

#### Post authentication/authorization headers

If you need to pass some headers to the downstream services, you can use the [headers][headers] policy.

#### OAuth 2.0 Flows

There are 3 distinct entities at play -
1. Client app (client or application that is registered on Express Gateway that wants to act on behalf of user)
2. Express Gateway's web UI (can be configured and customized by you)
3. Express Gateway itself as authorization server

##### Authorization Code

1. Client is already registered with Express Gateway and has an ID, secret and redirect URI in Expres Gateway.
2. Client app redirects user to the Express Gateway's web UI passing its app_id, response_type and scope
3. UI makes a GET to `/oauth2/authorize` and Express Gateway ensures user is logged in.
4. If user is not logged in, he/she logs in on the Express Gateway's UI. For this login, Express Gateway acts as an
authorization server for the user.

  **Note**: The user will need [basic-auth][basic-auth] credentials to be able to login.

5. Express Gateway's Web app prompts user to allow client access to specific scopes, to which the user grants access.
6. Express Gateway's UI again makes a call to `/oauth2/authorize` with the following data to request an auth code:
    * Header
        * Authorization: basic clientId:secret
    * body
        * `clientId`
        * `authenticatedUserId`
        * `redirectUri`
        * `scopes`
        * `response_type`

7. Express Gateway responds back with whether or not the client is authorized. If authorized, it returns an
   authorization code and redirect_uri.
8. UI redirects the client app to the redirect_uri
9. Client App will exchange the `auth_code` for an access token by making a POST to `/oauth2/token`

##### Implicit

Implicit grant is the same as authorization code grant, except we return a token and redirect_uri in step 7,
and it doesn’t have steps 8 and 9.

##### Client Credentials

Client credentials grant will follow the standard RFC documentation.

### Usage

In order to use the OAuth2 Authorization policy, consumers must be created and `oauth2` credentials created for them.

To create consumers (users and apps): use the [CLI][cli] and [create user][users-create] or [create app][apps-create]
command.
To create an `oauth2` credential for a user or app: use the [CLI][cli] and [create credential][credentials-create]
command with type `oauth2`.

To enable the OAuth2 policy, add `oauth2` in [gateway.config.yml][gateway.config.yml] in the [policies][policies]
section.

```yaml

policies:
  - oauth2
```

### Example

```yaml

pipelines:
  pipeline1:
    apiEndpoints:
      - authorizedEndpoint
    policies:
      - oauth2:
      - proxy:
          - action:
              serviceEndpoint: backend
```

##### Options Reference

* `jwt`
  - JWT signin parameters. For a complete reference, see the [jwt policy][jwt]
  - default value: `undefined`
* `passThrough`:
  - determines whether the gateway should execute the successive policy in case the auth process fails. If set to false,
    the gateway will return an `Unauthorized` response.
  - default value: `false`

### JWT support

By default, the oAuth2 server will respond back with an *opaque token*. This is a token in a proprietary format that
typically contain some identifier to information in a server’s persistent storage. To validate an opaque token,
the recipient of the token needs to call the server that issued the token.

However, it also supports issuing and veryfing JWT. This might come handy if you have multiple microservies across your
data center that need to be authenticated and authorized.

First of all, you'll need to configure the gateway to issue JSON Web Tokens instead of opaque ones. This can be done in
the [system.config.yml accessTokens section][access-tokens].

Then, you'll need to instruct the `oauth2` policy to verify JWT.

```yml
policies:
  - oauth2:
      action:
        jwt:
          issuer: express-gateway
          audience: something
          subject: test
          secretOrPublicKey: ssssst
          checkCredentialExistence: false
```

`jwt` parameters are the same listed in the [jwt policy][jwt]

You can see a live example on [Glitch](https://glitch.com/edit/#!/express-gateway-test)

### Customizing the UI

The basic implementation of the UI for the OAuth2 policy is found in `/lib/policies/oauth2/views`. In this directory,
you can customize the code to suit your needs.

[rfc-6749-standard]: https://tools.ietf.org/html/rfc6749
[gateway.config.yml]: {{ site.baseurl }}{% link docs/configuration/gateway.config.yml/index.md %}
[policies]: {{ site.baseurl }}{% link docs/configuration/gateway.config.yml/policies.md %}
[access-tokens]: {{ site.baseurl }}{% link docs/configuration/system.config.yml/accessTokens.md %}
[basic-auth]:{{ site.baseurl }}{% link docs/policies/basic-authorization.md %}
[jwt]:{{ site.baseurl }}{% link docs/policies/jwt.md %}
[headers]:{{ site.baseurl }}{% link docs/policies/headers.md %}
[cli]: {{ site.baseurl }}{% link docs/cli/index.md %}
[users-create]: {{ site.baseurl }}{% link docs/cli/users/create.md %}
[apps-create]: {{ site.baseurl }}{% link docs/cli/apps/create.md %}
[credentials-create]: {{ site.baseurl }}{% link docs/cli/credentials/create.md %}
