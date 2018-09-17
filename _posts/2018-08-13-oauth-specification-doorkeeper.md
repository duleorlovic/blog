---
layout: post
---

# Oauth specification

<https://tools.ietf.org/html/rfc6749> around 66 pages

> Instead of using the resource owner's credentials to access protected
> resources, the client obtains an access token -- a string denoting a specific
> scope, lifetime, and other access attributes. Access tokens are issued to
> third-party clients by an authorization server with the approval of the
> resource owner.

Roles:
* resource owner (end-user) capable of granting access to protected resource
* client (web or server app) application that make protected resource requests
* resource server: capable of responding to requests using access token
on behalf of the resource owner and with its authorization
* authorization server: server issuing access tokens after successfully
authenticating the resource owner, and obtaining authorization

Flow:

~~~
 +--------+                               +---------------+
 |        |--(A)- Authorization Request ->|   Resource    |
 |        |                               |     Owner     |
 |        |<-(B)-- Authorization Grant ---|               |
 |        |                               +---------------+
 |        |
 |        |                               +---------------+
 |        |--(C)-- Authorization Grant -->| Authorization |
 | Client |                               |     Server    |
 |        |<-(D)----- Access Token -------|               |
 |        |                               +---------------+
 |        |
 |        |                               +---------------+
 |        |--(E)----- Access Token ------>|    Resource   |
 |        |                               |     Server    |
 |        |<-(F)--- Protected Resource ---|               |
 +--------+                               +---------------+
~~~

* client gets authorization grant from resource owner
* than using that grant get access token
* and using that token get protected resource

4 Authorization Grant types (and extensibility mechanism for additional types):
* authorization code - grant is authorization code
* implicit - no grant, direct token
* resource owner password credentials - grant is password
* client credentials

1. Authorization code is obtained by using authorization server as an
   intermediary between the client and resource owner. Client directs resource
   owner to authorization server using user agent (because this is direct,
   resource owner credentials are never shared with a client) and than resource
   owner is directed back to the client with **authorization code** (without
   passing code to user agent).
2. Implicit is simplified authorization code for clients in browser using
   javascript. Instead of issuing authorization code, the client is issued an
   **access token** directly. When issuing access token, authorization server
   does not authenticate the client, only place where we can verify client is
   Redirect URI used to deliver access token to client. Access token may be
   exposed to resource owner or other application with access to the resource
   owner user agent.
3. Resource owner password credentials can be used as authorization grant to
   obtain access token. The credentials should be used only when there is high
   degree of trust between resource owner and client (for example client is part
   of device operating system). Usually client do not store credentials but use
   them in single request to exchange with long-lived access token or refresh
   token. Password flow is also used when your app owns oauth provider, since
   user is sending username and password to get token.
4. Client credentials is used when scope is limited to client protected
   resources (not end user).

Refresh tokens are used to obtain access tokens. It is optional and included
when issuing access token. It is used only with authorization server and never
sent to resource server. When access token expires client use refresh token to
get new access token (and eventual new refresh token). If client know expiration
date, it can request new access token before that moment.

## 2. Client Registration

When registering the client, client developer SHALL specify client type,
redirection URI and other info like application name, logo, terms.
Client type is *confidential* when client is capable of maintaining the
confidentiality of their credentials (it runs on secure server like rails and
can secure client authentication).
Client type is *public* when client is incapable of maintaining the
confidentiality (it runs as native application on resource owner device, web
browser application or can not secure client authentication). For native
application is it assumed that client authentication credentials included in
application can be extracted, but dynamicaly issued access tokens can receive
acceptable level of protection.

Authorization server issues the registered client a client identifier, unique
identifier representing information provided by the developer. It is not a
secret and it exposed to resource owner and MUST NOT be used alone for client
authentication.

If client is confidential, than authorization server MAY accept any form of
client authentication (password, public/private key pair). If client is public
than authorization server MUST NOT rely on public client authentication for the
purpose of identifying the client.

Client password can be used with HTTP Basic authentication scheme (Digest of
identifier and password). Alternatively, authorization server MAY support client
credentials in request body (MUST NOT in request URI) like:

~~~
POST /token HTTP/1.1
Host: server.example.com
Content-Type: application/x-www-form-urlencoded

grant_type=refresh_token&refresh_token=tGzv3JOkF0XG5Qx2TlKWIA
&client_id=s6BhdRkqt3&client_secret=7Fjfp0ZBr1KtDRbnfVdmIw
~~~

## 3. Protocol Endpoints

Two authorization server endpoints:

* authorization endpoint GET - used by client to obtain authorization grant from
resource owner via user agent redirection. Client get the location from service
documentation, Authorization server MUST verify identity of resource owner
(username/password, cookies...). It is used for Authorization code and implicit
which is defined in parameter: `response_type` REQUIRED `code` or `token`.
Redirection endpoint is MUST for public clients and confidentail clients
unilizing implicit grant type. Authorization server MAY allow client to register
multiple redirection endpoints. In that case, endpoint MUST be included in
`redirect_url` request parameter.

* token endpoint POST - used by client to exchange authorization grant to access
token. Used in every grant except implicit grant type (since access token is
issued directly).

One client endpoint:

* redirection endpoint - used by authorization server to return responses
containing authorization credentials to the client via resource owner user
agent. For native apps it can be `my_app://redirect` for others, it MUST be
using TLS security ie begins with https

Access token scope can be added as request parameter, as list of space delimited
case sensitive strings.

Client Authentication is MUST for token endpoint for confidential clients or
other clients issued client credentials. It is used to enforce bindings of
refresh token to client, recovering from compromised client by changing its
credentials.

## 4. Obtaining authorization

### 4.1 Authorization code

~~~
  +----------+
   | Resource |
   |   Owner  |
   |          |
   +----------+
        ^
        |
       (B)
   +----|-----+          Client Identifier      +---------------+
   |         -+----(A)-- & Redirection URI ---->|               |
   |  User-   |                                 | Authorization |
   |  Agent  -+----(B)-- User authenticates --->|     Server    |
   |          |                                 |               |
   |         -+----(C)-- Authorization Code ---<|               |
   +-|----|---+                                 +---------------+
     |    |                                         ^      v
    (A)  (C)                                        |      |
     |    |                                         |      |
     ^    v                                         |      |
   +---------+                                      |      |
   |         |>---(D)-- Authorization Code ---------'      |
   |  Client |          & Redirection URI                  |
   |         |                                             |
   |         |<---(E)----- Access Token -------------------'
   +---------+       (w/ Optional Refresh Token)
~~~

Authorication code grant is used to obtain both access token and refresh tokens
and is optimized for confidential clients. Since this is redirection based flow,
client must be capable of interacting with resource owner user agent and capable
of receiving incoming requests from authorization server.
URI example:

* response_type: required, must be `code`
* client_id: required, client id when you created application on oauth server
* redirect_uri: required, where to redirect after authorization is complete
* scope=photos - one or more scope values indicating which part of account you
wish to access
* state: recommended to prevent cross-site request forgery csrf, random string
which you will verify later.

Create "Log In" link with href `https://server.example.com/authorize?res...`:

~~~
GET /authorize?response_type=code&client_id=s6BhdRkqt3&state=xyz
        &redirect_uri=https%3A%2F%2Fclient%2Eexample%2Ecom%2Fcb HTTP/1.1
Host: server.example.com
~~~

Authorization response in case of resource owner grants access request

* code: required, lifetime of 10minutes
* state: required if state was in requests

~~~
HTTP/1.1 302 Found
Location: https://client.example.com/cb?code=SplxlOBeZQQYbYS6WxSbIA
         &state=xyz
~~~

Response in case of missing, invalid redirection url or client_id, authorization
server MUST NOT automatically redirect to invalid redirection URI, but SHOULD
inform resource owner of the error.
Response in case of denies, authorization server by adding following parameter
to the redirection URI

* error: required `invalid_request`, `unauthorized_client`, `access_denied` ...
* error_description: optional
* state: required if state was in requests

Access token request POST

* grant_type: required `authorization_code`
* code: required, authorization code from previous step
* redirect_uri: required if it was inclded in previous request
* client_id: required

Client MUST authenticate with authorization server if client type is confidental
or client was issued client credentials

~~~
POST /token HTTP/1.1
Host: server.example.com
Authorization: Basic czZCaGRSa3F0MzpnWDFmQmF0M2JW
Content-Type: application/x-www-form-urlencoded

grant_type=authorization_code&code=SplxlOBeZQQYbYS6WxSbIA
&redirect_uri=https%3A%2F%2Fclient%2Eexample%2Ecom%2Fcb
~~~

Access token response

~~~
HTTP/1.1 200 OK
Content-Type: application/json;charset=UTF-8
Cache-Control: no-store
Pragma: no-cache

{
 "access_token":"2YotnFZFEjr1zCsicMWpAA",
 "token_type":"example",
 "expires_in":3600,
 "refresh_token":"tGzv3JOkF0XG5Qx2TlKWIA",
 "example_parameter":"example_value"
}
~~~

### 4.2 Implicit Grant

~~~
   +----------+
   | Resource |
   |  Owner   |
   |          |
   +----------+
        ^
        |
       (B)
   +----|-----+          Client Identifier     +---------------+
   |         -+----(A)-- & Redirection URI --->|               |
   |  User-   |                                | Authorization |
   |  Agent  -|----(B)-- User authenticates -->|     Server    |
   |          |                                |               |
   |          |<---(C)--- Redirection URI ----<|               |
   |          |          with Access Token     +---------------+
   |          |            in Fragment
   |          |                                +---------------+
   |          |----(D)--- Redirection URI ---->|   Web-Hosted  |
   |          |          without Fragment      |     Client    |
   |          |                                |    Resource   |
   |     (F)  |<---(E)------- Script ---------<|               |
   |          |                                +---------------+
   +-|--------+
     |    |
    (A)  (G) Access Token
     |    |
     ^    v
   +---------+
   |         |
   |  Client |
   |         |
   +---------+
~~~

Optimized for public clients known to operate a particular redirection URI (it
does not support refresh tokens).
It is the same as authorization code but not sending secret, so we might use
`code`.

Request:

* response_type: required `token`
* client_id: required
* redirect_uri: optional
* scope: optional
* state: recommended

~~~
GET /authorize?response_type=token&client_id=s6BhdRkqt3&state=xyz
    &redirect_uri=https%3A%2F%2Fclient%2Eexample%2Ecom%2Fcb HTTP/1.1
Host: server.example.com
~~~

Authorization server MUST verify redirection URI matches registered uris.

Response:

* access_token: required
* token_type: required `bearer`, `mac` [Access token types](
https://tools.ietf.org/html/rfc6749#section-7.1)
* expires_in: recommended
* scope: optional if identical to requested scope, otherwise required
* state: required if state was in requests

~~~
HTTP/1.1 302 Found
Location: http://example.com/cb#access_token=2YotnFZFEjr1zCsicMWpAA
         &state=xyz&token_type=example&expires_in=3600
~~~

### 4.3 Resource Owner Password Credentials Grant

~~~
   +----------+
   | Resource |
   |  Owner   |
   |          |
   +----------+
        v
        |    Resource Owner
       (A) Password Credentials
        |
        v
   +---------+                                  +---------------+
   |         |>--(B)---- Resource Owner ------->|               |
   |         |         Password Credentials     | Authorization |
   | Client  |                                  |     Server    |
   |         |<--(C)---- Access Token ---------<|               |
   |         |    (w/ Optional Refresh Token)   |               |
   +---------+                                  +---------------+
~~~

Request:
* grant_type: required `password`
* username: required
* password: required
* scope: optional

We might send also client_id.

~~~
POST /token HTTP/1.1
Host: server.example.com
Authorization: Basic czZCaGRSa3F0MzpnWDFmQmF0M2JW
Content-Type: application/x-www-form-urlencoded

grant_type=password&username=johndoe&password=A3ddj3w
~~~

Response

~~~
HTTP/1.1 200 OK
Content-Type: application/json;charset=UTF-8
Cache-Control: no-store
Pragma: no-cache

{
 "access_token":"2YotnFZFEjr1zCsicMWpAA",
 "token_type":"example",
 "expires_in":3600,
 "refresh_token":"tGzv3JOkF0XG5Qx2TlKWIA",
 "example_parameter":"example_value"
}
~~~

### 4.4 Client Credentials Grant

~~~
   +---------+                                  +---------------+
   |         |                                  |               |
   |         |>--(A)- Client Authentication --->| Authorization |
   | Client  |                                  |     Server    |
   |         |<--(B)---- Access Token ---------<|               |
   |         |                                  |               |
   +---------+                                  +---------------+
~~~

Client credentials grant type MUST only be used by confidential clients.
Authorization server MUST authenticate the client.
client_credentials is usually for apps (not for users) to update their info
(like icon, statistics...) outside of the context of any user.
But it can be used for apps than do not require user interaction, for example
payment gateway and when customer wants to pay, client do not need to ask user
if that is OK.

Request:

~~~
POST /token HTTP/1.1
Host: server.example.com
Authorization: Basic czZCaGRSa3F0MzpnWDFmQmF0M2JW
Content-Type: application/x-www-form-urlencoded

grant_type=client_credentials
~~~

~~~
HTTP/1.1 200 OK
Content-Type: application/json;charset=UTF-8
Cache-Control: no-store
Pragma: no-cache

{
 "access_token":"2YotnFZFEjr1zCsicMWpAA",
 "token_type":"example",
 "expires_in":3600,
 "example_parameter":"example_value"
}
~~~

## 10. Client Authentication

Web application clients MUST ensure confidentiality of client password.
The authorization server MUST NOT issue client passwords or other client
credentials to native application or user-agent-based application clients for
the purpose of client authentication.

The authorization server MAY issue a client password or other credentials for a
specific installation of a native application client on a specific device.

[Access Token Specification](https://tools.ietf.org/html/rfc6750)
Access token can have different formats. But it is abstraction layer so resource
server does not need to understand wide range of authentication methods.

## Rails oauth provider doorkeeper

[doorkeeper](https://github.com/doorkeeper-gem/doorkeeper) is rails engine that
provides oauth 2 auth [railscasts #353](http://railscasts.com/episodes/353-oauth-with-doorkeeper)
Some intro how google does oauth2
https://developers.google.com/identity/protocols/OAuth2?csw=1

~~~
cat >> Gemfile <<HERE_DOC
# oauth provider
gem 'doorkeeper', '~> 4.2.6'
HERE_DOC
rails g doorkeeper:install
rails g doorkeeper:migration
# add_foreign_key :oauth_access_grants, :users, column: :resource_owner_id
# add_foreign_key :oauth_access_tokens, :users, column: :resource_owner_id
git add . && git commit -am "rails g doorkeeper:install"
~~~

Migration will generate three tables:

* oauth_applications - store secret and redirect_uri
* oauth_access_grants - tokens for each user and application
* oauth_access_tokens - token, refresh_token

When you list all generated routes you will notice three:

~~~
rails routes -g oauth
# /oauth/authorize
# /oauth/applications - CRUD for applications
# /oauth/authorized_applications
# /oauth/token
# /oauth/token/info
# /oauth/revoke
~~~

First you need to set up one oauth application on
<http://localhost:3004/oauth/applications> to generate `client_id` and
`client_secret`.

There are 4 ways to use [oauth grant types](https://aaronparecki.com/oauth-2-simplified):
* authorization_code: web server, browser based and mobile apps
* imlicit: superceded by authorization code with no secret
* password
* client_credentials

When you get access token, than you can authenticated request using header:
In doorkeeper by default access_token expires after 2 hours.

~~~
curl -H "Authorization: Bearer RsT5OjbzRn430zqMLgV3Ia" https://api.oauth2server.com/1/me
~~~

[Using scopes](https://github.com/doorkeeper-gem/doorkeeper/wiki/Using-Scopes)

Some provider opensource examples:
<https://github.com/doorkeeper-gem/doorkeeper/wiki/Example-Applications>

To test provider you can use [oauth2 client](https://github.com/intridea/oauth2)
to generate `access_token`

~~~
gem install oauth2
irb -r oauth2
client_id = ''
client_secret = ''
redirect_uri = ''
client = OAuth2::Client.new(client_id, client_secret, site: 'https://localhost:3000')

client.auth_code.authorize_url redirect_uri: redirect_url
~~~
