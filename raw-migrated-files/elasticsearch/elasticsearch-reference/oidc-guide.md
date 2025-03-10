# Configuring single sign-on to the {{stack}} using OpenID Connect [oidc-guide]

The Elastic Stack supports single sign-on (SSO) using OpenID Connect via {{kib}} using {{es}} as the backend service that holds most of the functionality. {{kib}} and {{es}} together represent an OpenID Connect Relying Party (RP) that supports the authorization code flow and implicit flow as these are defined in the OpenID Connect specification.

This guide assumes that you have an OpenID Connect Provider where the Elastic Stack Relying Party will be registered.

::::{note} 
The OpenID Connect realm support in {{kib}} is designed with the expectation that it will be the primary authentication method for the users of that {{kib}} instance. The [Configuring {{kib}}](../../../deploy-manage/users-roles/cluster-or-deployment-auth/openid-connect.md#oidc-configure-kibana) section describes what this entails and how you can set it up to support other realms if necessary.
::::


## The OpenID Connect Provider [oidc-guide-op]

The OpenID Connect Provider (OP) is the entity in OpenID Connect that is responsible for authenticating the user and for granting the necessary tokens with the authentication and user information to be consumed by the Relying Parties.

In order for the Elastic Stack to be able to use your OpenID Connect Provider for authentication, a trust relationship needs to be established between the OP and the RP. In the OpenID Connect Provider, this means registering the RP as a client. OpenID Connect defines a dynamic client registration protocol but this is usually geared towards real-time client registration and not the trust establishment process for cross security domain single sign on. All OPs will also allow for the manual registration of an RP as a client, via a user interface or (less often) via the consumption of a metadata document.

The process for registering the Elastic Stack RP will be different from OP to OP and following the provider’s relevant documentation is prudent. The information for the RP that you commonly need to provide for registration are the following:

* `Relying Party Name`: An arbitrary identifier for the relying party. Neither the specification nor the Elastic Stack implementation impose any constraints on this value.
* `Redirect URI`: This is the URI where the OP will redirect the user’s browser after authentication. The appropriate value for this will depend on your setup and whether or not {{kib}} sits behind a proxy or load balancer. It will typically be `${kibana-url}/api/security/oidc/callback` (for the authorization code flow) or `${kibana-url}/api/security/oidc/implicit` (for the implicit flow) where *${kibana-url}* is the base URL for your {{kib}} instance. You might also see this called `Callback URI`.

At the end of the registration process, the OP will assign a Client Identifier and a Client Secret for the RP ({{stack}}) to use. Note these two values as they will be used in the {{es}} configuration.


## Configure {{es}} for OpenID Connect authentication [oidc-elasticsearch-authentication]

The following is a summary of the configuration steps required in order to enable authentication using OpenID Connect in {{es}}:

1. [Enable SSL/TLS for HTTP](../../../deploy-manage/users-roles/cluster-or-deployment-auth/openid-connect.md#oidc-enable-http)
2. [Enable the Token Service](../../../deploy-manage/users-roles/cluster-or-deployment-auth/openid-connect.md#oidc-enable-token)
3. [Create one or more OpenID Connect realms](../../../deploy-manage/users-roles/cluster-or-deployment-auth/openid-connect.md#oidc-create-realm)
4. [Configure role mappings](../../../deploy-manage/users-roles/cluster-or-deployment-auth/openid-connect.md#oidc-role-mappings)

### Enable TLS for HTTP [oidc-enable-http]

If your {{es}} cluster is operating in production mode, then you must configure the HTTP interface to use SSL/TLS before you can enable OpenID Connect authentication.

For more information, see [Encrypt HTTP client communications for {{es}}](../../../deploy-manage/security/set-up-basic-security-plus-https.md#encrypt-http-communication).


### Enable the token service [oidc-enable-token]

The {{es}} OpenID Connect implementation makes use of the {{es}} Token Service. This service is automatically enabled if you configure TLS on the HTTP interface, and can be explicitly configured by including the following in your `elasticsearch.yml` file:

```yaml
xpack.security.authc.token.enabled: true
```


### Create an OpenID Connect realm [oidc-create-realm]

OpenID Connect based authentication is enabled by configuring the appropriate realm within the authentication chain for {{es}}.

This realm has a few mandatory settings, and a number of optional settings. The available settings are described in detail in [OpenID Connect realm settings](asciidocalypse://docs/elasticsearch/docs/reference/elasticsearch/configuration-reference/security-settings.md#ref-oidc-settings). This guide will explore the most common settings.

Create an OpenID Connect (the realm type is `oidc`) realm in your `elasticsearch.yml` file similar to what is shown below:

::::{note} 
The values used below are meant to be an example and are not intended to apply to every use case. The details below the configuration snippet provide insights and suggestions to help you pick the proper values, depending on your OP configuration.
::::


```yaml
xpack.security.authc.realms.oidc.oidc1:
  order: 2
  rp.client_id: "the_client_id"
  rp.response_type: code
  rp.redirect_uri: "https://kibana.example.org:5601/api/security/oidc/callback"
  op.issuer: "https://op.example.org"
  op.authorization_endpoint: "https://op.example.org/oauth2/v1/authorize"
  op.token_endpoint: "https://op.example.org/oauth2/v1/token"
  op.jwkset_path: oidc/jwkset.json
  op.userinfo_endpoint: "https://op.example.org/oauth2/v1/userinfo"
  op.endsession_endpoint: "https://op.example.org/oauth2/v1/logout"
  rp.post_logout_redirect_uri: "https://kibana.example.org:5601/security/logged_out"
  claims.principal: sub
  claims.groups: "http://example.info/claims/groups"
```

The configuration values used in the example above are:

xpack.security.authc.realms.oidc.oidc1
:   This defines a new `oidc` authentication realm named "oidc1". See [Realms](../../../deploy-manage/users-roles/cluster-or-deployment-auth/authentication-realms.md) for more explanation of realms.

order
:   You should define a unique order on each realm in your authentication chain. It is recommended that the OpenID Connect realm be at the bottom of your authentication chain (that is, that it has the *highest* order).

rp.client_id
:   This, usually opaque, arbitrary string, is the Client Identifier that was assigned to the Elastic Stack RP by the OP upon registration.

rp.response_type
:   This is an identifier that controls which OpenID Connect authentication flow this RP supports and also which flow this RP requests the OP should follow. Supported values are

    * `code`, which means that the RP wants to use the Authorization Code flow. If your OP supports the Authorization Code flow, you should select this instead of the Implicit Flow.
    * `id_token token` which means that the RP wants to use the Implicit flow and we also request an oAuth2 access token from the OP, that we can potentially use for follow up requests ( UserInfo ). This should be selected if the OP offers a UserInfo endpoint in its configuration, or if you know that the claims you will need to use for role mapping are not available in the ID Token.
    * `id_token` which means that the RP wants to use the Implicit flow, but is not interested in getting an oAuth2 token too. Select this if you are certain that all necessary claims will be contained in the ID Token or if the OP doesn’t offer a User Info endpoint.


rp.redirect_uri
:   The redirect URI where the OP will redirect the browser after authentication. This needs to be *exactly* the same as the one [configured with the OP upon registration](../../../deploy-manage/users-roles/cluster-or-deployment-auth/openid-connect.md#oidc-guide-op) and will typically be `${kibana-url}/api/security/oidc/callback` where *${kibana-url}* is the base URL for your {{kib}} instance

op.issuer
:   A verifiable Identifier for your OpenID Connect Provider. An Issuer Identifier is usually a case sensitive URL. The value for this setting should be provided by your OpenID Connect Provider.

op.authorization_endpoint
:   The URL for the Authorization Endpoint in the OP. This is where the user’s browser will be redirected to start the authentication process. The value for this setting should be provided by your OpenID Connect Provider.

op.token_endpoint
:   The URL for the Token Endpoint in the OpenID Connect Provider. This is the endpoint where {{es}} will send a request to exchange the code for an ID Token. This setting is optional when you use the implicit flow. The value for this setting should be provided by your OpenID Connect Provider.

op.jwkset_path
:   The path to a file or a URL containing a JSON Web Key Set with the key material that the OpenID Connect Provider uses for signing tokens and claims responses. If a path is set, it is resolved relative to the {{es}} config directory. {{es}} will automatically monitor this file for changes and will reload the configuration whenever it is updated. Your OpenID Connect Provider should provide you with this file or a URL where it is available.

op.userinfo_endpoint
:   (Optional) The URL for the UserInfo Endpoint in the OpenID Connect Provider. This is the endpoint of the OP that can be queried to get further user information, if required. The value for this setting should be provided by your OpenID Connect Provider.

op.endsession_endpoint
:   (Optional) The URL to the End Session Endpoint in the OpenID Connect Provider. This is the endpoint where the user’s browser will be redirected after local logout, if the realm is configured for RP initiated Single Logout and the OP supports it. The value for this setting should be provided by your OpenID Connect Provider.

rp.post_logout_redirect_uri
:   (Optional) The Redirect URL where the OpenID Connect Provider should redirect the user after a successful Single Logout (assuming `op.endsession_endpoint` above is also set). This should be set to a value that will not trigger a new OpenID Connect Authentication, such as `${kibana-url}/security/logged_out`  or `${kibana-url}/login?msg=LOGGED_OUT` where *${kibana-url}* is the base URL for your {{kib}} instance.

claims.principal
:   See [Claims mapping](../../../deploy-manage/users-roles/cluster-or-deployment-auth/openid-connect.md#oidc-claims-mappings).

claims.groups
:   See [Claims mapping](../../../deploy-manage/users-roles/cluster-or-deployment-auth/openid-connect.md#oidc-claims-mappings).

A final piece of configuration of the OpenID Connect realm is to set the `Client Secret` that was assigned to the RP during registration in the OP. This is a secure setting and as such is not defined in the realm configuration in `elasticsearch.yml` but added to the [elasticsearch keystore](../../../deploy-manage/security/secure-settings.md). For instance

```sh
bin/elasticsearch-keystore add xpack.security.authc.realms.oidc.oidc1.rp.client_secret
```

::::{note} 
Changes to the `client_secret` requires a restart of the {{es}} nodes to pick up the change.
::::


::::{note} 
According to the OpenID Connect specification, the OP should also make their configuration available at a well known URL, which is the concatenation of their `Issuer` value with the `.well-known/openid-configuration` string. For example: `https://op.org.com/.well-known/openid-configuration` That document should contain all the necessary information to configure the OpenID Connect realm in {{es}}.
::::



### Claims mapping [oidc-claims-mappings]

#### Claims and scopes [_claims_and_scopes]

When authenticating to {{kib}} using OpenID Connect, the OP will provide information about the user in the form of OpenID Connect Claims, that can be included either in the ID Token, or be retrieved from the UserInfo endpoint of the OP. The claim is defined as a piece of information asserted by the OP for the authenticated user. Simply put, a claim is a name/value pair that contains information about the user. Related to claims, we also have the notion of OpenID Connect Scopes. Scopes are identifiers that are used to request access to specific lists of claims. The standard defines a set of scope identifiers that can be requested. The only mandatory one is `openid`, while commonly used ones are `profile` and `email`. The `profile` scope requests access to the `name`,`family_name`,`given_name`,`middle_name`,`nickname`, `preferred_username`,`profile`,`picture`,`website`,`gender`,`birthdate`,`zoneinfo`,`locale`, and `updated_at` claims. The `email` scope requests access to the `email` and `email_verified` claims. The process is that the RP requests specific scopes during the authentication request. If the OP Privacy Policy allows it and the authenticating user consents to it, the related claims are returned to the RP (either in the ID Token or as a UserInfo response).

The list of the supported claims will vary depending on the OP you are using, but you can expect the [Standard Claims](https://openid.net/specs/openid-connect-core-1_0.md#StandardClaims) to be largely supported.


#### Mapping claims to user properties [oidc-claim-to-property]

The goal of claims mapping is to configure {{es}} in such a way as to be able to map the values of specified returned claims to one of the [user properties](../../../deploy-manage/users-roles/cluster-or-deployment-auth/openid-connect.md#oidc-user-properties) that are supported by {{es}}. These user properties are then utilized to identify the user in the {{kib}} UI or the audit logs, and can also be used to create [role mapping](../../../deploy-manage/users-roles/cluster-or-deployment-auth/openid-connect.md#oidc-role-mappings) rules.

The recommended steps for configuring OpenID Claims mapping are as follows:

1. Consult your OP configuration to see what claims it might support. Note that the list provided in the OP’s metadata or in the configuration page of the OP is a list of potentially supported claims. However, for privacy reasons it might not be a complete one, or not all supported claims will be available for all authenticated users.
2. Read through the list of [user properties](../../../deploy-manage/users-roles/cluster-or-deployment-auth/openid-connect.md#oidc-user-properties) that {{es}} supports, and decide which of them are useful to you, and can be provided by your OP in the form of claims. At a *minimum*, the `principal` user property is required.
3. Configure your OP to "release" those claims to your {{stack}} Relying party. This process greatly varies by provider. You can use a static configuration while others will support that the RP requests the scopes that correspond to the claims to be "released" on authentication time. See [`rp.requested_scopes`](asciidocalypse://docs/elasticsearch/docs/reference/elasticsearch/configuration-reference/security-settings.md#ref-oidc-settings) for details about how to configure the scopes to request. To ensure interoperability and minimize the errors, you should only request scopes that the OP supports, and which you intend to map to {{es}} user properties.

    ```
    NOTE: You can only map claims with values that are strings, numbers, boolean values or an array
    of the aforementioned.
    ```

4. Configure the OpenID Connect realm in {{es}} to associate the {{es}} user properties (see [the listing](../../../deploy-manage/users-roles/cluster-or-deployment-auth/openid-connect.md#oidc-user-properties) below), to the name of the claims that your OP will release. In the example above, we have configured the `principal` and `groups` user properties as follows:

    1. `claims.principal: sub` : This instructs {{es}} to look for the OpenID Connect claim named `sub` in the ID Token that the OP issued for the user ( or in the UserInfo response ) and assign the value of this claim to the `principal` user property. `sub` is a commonly used claim for the principal property as it is an identifier of the user in the OP and it is also a required claim of the ID Token, thus offering guarantees that it will be available. It is, however, only used as an example here, the OP may provide another claim that is a better fit for your needs.
    2. `claims.groups: "http://example.info/claims/groups"` : Similarly, this instructs {{es}} to look for the claim with the name `http://example.info/claims/groups` (note that this is a URI - an identifier, treated as a string and not a URL pointing to a location that will be retrieved) either in the ID Token or in the UserInfo response, and map the value(s) of it to the user property `groups` in {{es}}. There is no standard claim in the specification that is used for expressing roles or group memberships of the authenticated user in the OP, so the name of the claim that should be mapped here, will vary greatly between providers. Consult your OP documentation for more details.



#### {{es}} user properties [oidc-user-properties]

The {{es}} OpenID Connect realm can be configured to map OpenID Connect claims to the following properties on the authenticated user:

principal
:   *(Required)* This is the *username* that will be applied to a user that authenticates against this realm. The `principal` appears in places such as the {{es}} audit logs.

::::{note} 
If the principal property fails to be mapped from a claim, the authentication fails.
::::


groups
:   *(Recommended)* If you wish to use your OP’s concept of groups or roles as the basis for a user’s {{es}} privileges, you should map them with this property. The `groups` are passed directly to your [role mapping rules](../../../deploy-manage/users-roles/cluster-or-deployment-auth/openid-connect.md#oidc-role-mappings).

name
:   *(Optional)* The user’s full name.

mail
:   *(Optional)* The user’s email address.

dn
:   *(Optional)* The user’s X.500 *Distinguished Name*.


#### Extracting partial values from OpenID Connect claims [_extracting_partial_values_from_openid_connect_claims]

There are some occasions where the value of a claim may contain more information than you wish to use within {{es}}. A common example of this is one where the OP works exclusively with email addresses, but you would like the user’s `principal` to use the *local-name* part of the email address. For example if their email address was `james.wong@staff.example.com`, then you would like their principal to simply be `james.wong`.

This can be achieved using the `claim_patterns` setting in the {{es}} realm, as demonstrated in the realm configuration below:

```yaml
xpack.security.authc.realms.oidc.oidc1:
  order: 2
  rp.client_id: "the_client_id"
  rp.response_type: code
  rp.redirect_uri: "https://kibana.example.org:5601/api/security/oidc/callback"
  op.authorization_endpoint: "https://op.example.org/oauth2/v1/authorize"
  op.token_endpoint: "https://op.example.org/oauth2/v1/token"
  op.userinfo_endpoint: "https://op.example.org/oauth2/v1/userinfo"
  op.endsession_endpoint: "https://op.example.org/oauth2/v1/logout"
  op.issuer: "https://op.example.org"
  op.jwkset_path: oidc/jwkset.json
  claims.principal: email_verified
  claim_patterns.principal: "^([^@]+)@staff\\.example\\.com$"
```

In this case, the user’s `principal` is mapped from the `email_verified` claim, but a regular expression is applied to the value before it is assigned to the user. If the regular expression matches, then the result of the first group is used as the effective value. If the regular expression does not match then the claim mapping fails.

In this example, the email address must belong to the `staff.example.com` domain, and then the local-part (anything before the `@`) is used as the principal. Any users who try to login using a different email domain will fail because the regular expression will not match against their email address, and thus their principal user property - which is mandatory - will not be populated.

::::{important} 
Small mistakes in these regular expressions can have significant security consequences. For example, if we accidentally left off the trailing `$` from the example above, then we would match any email address where the domain starts with `staff.example.com`, and this would accept an email address such as `admin@staff.example.com.attacker.net`. It is important that you make sure your regular expressions are as precise as possible so that you do not inadvertently open an avenue for user impersonation attacks.
::::




### Third party initiated single sign-on [third-party-login]

The Open ID Connect realm in {{es}} supports 3rd party initiated login as described in the [relevant specification](https://openid.net/specs/openid-connect-core-1_0.md#ThirdPartyInitiatedLogin).

This allows the OP itself or another, third party other than the RP, to initiate the authentication process while requesting the OP to be used for the authentication. Please note that the Elastic Stack RP should already be configured for this OP, in order for this process to succeed.


### OpenID Connect Logout [oidc-logout]

The OpenID Connect realm in {{es}} supports RP-Initiated Logout Functionality as described in the [relevant part of the specification](https://openid.net/specs/openid-connect-session-1_0.md#RPLogout)

In this process, the OpenID Connect RP (the Elastic Stack in this case) will redirect the user’s browser to predefined URL of the OP after successfully completing a local logout. The OP can then logout the user also, depending on the configuration, and should finally redirect the user back to the RP. The `op.endsession_endpoint` in the realm configuration determines the URL in the OP that the browser will be redirected to. The `rp.post_logout_redirect_uri` setting determines the URL to redirect the user back to after the OP logs them out.

When configuring `rp.post_logout_redirect_uri`, care should be taken to not point this to a URL that will trigger re-authentication of the user. For instance, when using OpenID Connect to support single sign-on to {{kib}}, this could be set to either `${kibana-url}/security/logged_out`, which will show a user-friendly message to the user or `${kibana-url}/login?msg=LOGGED_OUT` which will take the user to the login selector in {{kib}}.


### OpenID Connect Realm SSL Configuration [oidc-ssl-config]

OpenID Connect depends on TLS to provide security properties such as encryption in transit and endpoint authentication. The RP is required to establish back-channel communication with the OP in order to exchange the code for an ID Token during the Authorization code grant flow and in order to get additional user information from the UserInfo endpoint. Furthermore, if you configure `op.jwks_path` as a URL, {{es}} will need to get the OP’s signing keys from the file hosted there. As such, it is important that {{es}} can validate and trust the server certificate that the OP uses for TLS. Since the system truststore is used for the client context of outgoing https connections, if your OP is using a certificate from a trusted CA, no additional configuration is needed.

However, if the issuer of your OP’s certificate is not trusted by the JVM on which {{es}} is running (e.g it uses a organization CA), then you must configure {{es}} to trust that CA. Assuming that you have the CA certificate that has signed the certificate that the OP uses for TLS stored in the /oidc/company-ca.pem` file stored in the configuration directory of {{es}}, you need to set the following property in the realm configuration:

```yaml
xpack.security.authc.realms.oidc.oidc1:
  order: 1
  ...
  ssl.certificate_authorities: ["/oidc/company-ca.pem"]
```



## Configuring role mappings [oidc-role-mappings]

When a user authenticates using OpenID Connect, they are identified to the Elastic Stack, but this does not automatically grant them access to perform any actions or access any data.

Your OpenID Connect users cannot do anything until they are assigned roles. This can be done through either the [add role mapping API](https://www.elastic.co/docs/api/doc/elasticsearch/operation/operation-security-put-role-mapping) or with [authorization realms](../../../deploy-manage/users-roles/cluster-or-deployment-auth/realm-chains.md#authorization_realms).

::::{note} 
You cannot use [role mapping files](../../../deploy-manage/users-roles/cluster-or-deployment-auth/mapping-users-groups-to-roles.md#mapping-roles-file) to grant roles to users authenticating via OpenID Connect.
::::


This is an example of a simple role mapping that grants the `example_role` role to any user who authenticates against the `oidc1` OpenID Connect realm:

```console
PUT /_security/role_mapping/oidc-example
{
  "roles": [ "example_role" ], <1>
  "enabled": true,
  "rules": {
    "field": { "realm.name": "oidc1" }
  }
}
```

1. The `example_role` role is **not** a builtin Elasticsearch role. This example assumes that you have created a custom role of your own, with appropriate access to your [data streams, indices,](../../../deploy-manage/users-roles/cluster-or-deployment-auth/defining-roles.md#roles-indices-priv) and [Kibana features](../../../deploy-manage/users-roles/cluster-or-deployment-auth/kibana-privileges.md#kibana-feature-privileges).


The user properties that are mapped via the realm configuration are used to process role mapping rules, and these rules determine which roles a user is granted.

The user fields that are provided to the role mapping are derived from the OpenID Connect claims as follows:

* `username`: The `principal` user property
* `dn`: The `dn` user property
* `groups`: The `groups` user property
* `metadata`: See [User metadata](../../../deploy-manage/users-roles/cluster-or-deployment-auth/openid-connect.md#oidc-user-metadata)

For more information, see [Mapping users and groups to roles](../../../deploy-manage/users-roles/cluster-or-deployment-auth/mapping-users-groups-to-roles.md) and [Role mappings](https://www.elastic.co/docs/api/doc/elasticsearch/group/endpoint-security).

If your OP has the ability to provide groups or roles to RPs via tha use of an OpenID Claim, then you should map this claim to the `claims.groups` setting in the {{es}} realm (see [Mapping claims to user properties](../../../deploy-manage/users-roles/cluster-or-deployment-auth/openid-connect.md#oidc-claim-to-property)), and then make use of it in a role mapping as per the example below.

This mapping grants the {{es}} `finance_data` role, to any users who authenticate via the `oidc1` realm with the `finance-team` group membership.

```console
PUT /_security/role_mapping/oidc-finance
{
  "roles": [ "finance_data" ],
  "enabled": true,
  "rules": { "all": [
        { "field": { "realm.name": "oidc1" } },
        { "field": { "groups": "finance-team" } }
  ] }
}
```

If your users also exist in a repository that can be directly accessed by {{es}} (such as an LDAP directory) then you can use [authorization realms](../../../deploy-manage/users-roles/cluster-or-deployment-auth/realm-chains.md#authorization_realms) instead of role mappings.

In this case, you perform the following steps:

1. In your OpenID Connect realm, assign a claim to act as the lookup userid, by configuring the `claims.principal` setting.
2. Create a new realm that can look up users from your local repository (e.g. an `ldap` realm)
3. In your OpenID Connect realm, set `authorization_realms` to the name of the realm you created in step 2.


## User metadata [oidc-user-metadata]

By default users who authenticate via OpenID Connect will have some additional metadata fields. These fields will include every OpenID Claim that is provided in the authentication response (regardless of whether it is mapped to an {{es}} user property). For example, in the metadata field `oidc(claim_name)`, "claim_name" is the name of the claim as it was contained in the ID Token or in the User Info response. Note that these will include all the [ID Token claims](https://openid.net/specs/openid-connect-core-1_0.md#IDToken) that pertain to the authentication event, rather than the user themselves.

This behaviour can be disabled by adding `populate_user_metadata: false` as a setting in the oidc realm.


## Configuring {{kib}} [oidc-configure-kibana]

OpenID Connect authentication in {{kib}} requires a small number of additional settings in addition to the standard {{kib}} security configuration. The [{{kib}} security documentation](../../../deploy-manage/security.md) provides details on the available configuration options that you can apply.

In particular, since your {{es}} nodes have been configured to use TLS on the HTTP interface, you must configure {{kib}} to use a `https` URL to connect to {{es}}, and you may need to configure `elasticsearch.ssl.certificateAuthorities` to trust the certificates that {{es}} has been configured to use.

OpenID Connect authentication in {{kib}} is subject to the following timeout settings in `kibana.yml`:

* [`xpack.security.session.idleTimeout`](../../../deploy-manage/security/kibana-session-management.md#session-idle-timeout)
* [`xpack.security.session.lifespan`](../../../deploy-manage/security/kibana-session-management.md#session-lifespan)

You may want to adjust these timeouts based on your security requirements.

The three additional settings that are required for OpenID Connect support are shown below:

```yaml
xpack.security.authc.providers:
  oidc.oidc1:
    order: 0
    realm: "oidc1"
```

The configuration values used in the example above are:

`xpack.security.authc.providers`
:   Add `oidc` provider to instruct {{kib}} to use OpenID Connect single sign-on as the authentication method. This instructs Kibana to attempt to initiate an SSO flow everytime a user attempts to access a URL in Kibana, if the user is not already authenticated. If you also want to allow users to login with a username and password, you must enable the `basic` authentication provider too. For example:

```yaml
xpack.security.authc.providers:
  oidc.oidc1:
    order: 0
    realm: "oidc1"
  basic.basic1:
    order: 1
```

This will allow users that haven’t already authenticated with OpenID Connect to log in using the {{kib}} login form.

`xpack.security.authc.providers.oidc.<provider-name>.realm`
:   The name of the OpenID Connect realm in {{es}} that should handle authentication for this Kibana instance.


## OpenID Connect without {{kib}} [oidc-without-kibana]

The OpenID Connect realm is designed to allow users to authenticate to {{kib}} and as such, most of the parts of the guide above make the assumption that {{kib}} is used. This section describes how a custom web application could use the relevant OpenID Connect REST APIs in order to authenticate the users to {{es}}, with OpenID Connect.

Single sign-on realms such as OpenID Connect and SAML make use of the Token Service in {{es}} and in principle exchange a SAML or OpenID Connect Authentication response for an {{es}} access token and a refresh token. The access token is used as credentials for subsequent calls to {{es}}. The refresh token enables the user to get new {{es}} access tokens after the current one expires.

::::{note} 
The {{es}} Token Service can be seen as a minimal oAuth2 authorization server and the access token and refresh token mentioned above are tokens that pertain *only* to this authorization server. They are generated and consumed *only* by {{es}} and are in no way related to the tokens ( access token and ID Token ) that the OpenID Connect Provider issues.
::::


### Register the RP with an OpenID Connect Provider [_register_the_rp_with_an_openid_connect_provider]

The Relying Party ( {{es}} and the custom web app ) will need to be registered as client with the OpenID Connect Provider. Note that when registering the `Redirect URI`, it needs to be a URL in the custom web app.


### OpenID Connect Realm [_openid_connect_realm]

An OpenID Connect realm needs to be created and configured accordingly in {{es}}. See [Configure {{es}} for OpenID Connect authentication](../../../deploy-manage/users-roles/cluster-or-deployment-auth/openid-connect.md#oidc-elasticsearch-authentication)


### Service Account user for accessing the APIs [_service_account_user_for_accessing_the_apis]

The realm is designed with the assumption that there needs to be a privileged entity acting as an authentication proxy. In this case, the custom web application is the authentication proxy handling the authentication of end users ( more correctly, "delegating" the authentication to the OpenID Connect Provider ). The OpenID Connect APIs require authentication and the necessary authorization level for the authenticated user. For this reason, a Service Account user needs to be created and assigned a role that gives them the `manage_oidc` cluster privilege. The use of the `manage_token` cluster privilege will be necessary after the authentication takes place, so that the user can maintain access or be subsequently logged out.

```console
POST /_security/role/facilitator-role
{
  "cluster" : ["manage_oidc", "manage_token"]
}
```

```console
POST /_security/user/facilitator
{
  "password" : "<somePasswordHere>",
  "roles"    : [ "facilitator-role"]
}
```


### Handling the authentication flow [_handling_the_authentication_flow]

On a high level, the custom web application would need to perform the following steps in order to authenticate a user with OpenID Connect:

1. Make an HTTP POST request to `_security/oidc/prepare`, authenticating as the `facilitator` user, using the name of the OpenID Connect realm in the {{es}} configuration in the request body. For more details, see [OpenID Connect prepare authentication](https://www.elastic.co/docs/api/doc/elasticsearch/operation/operation-security-oidc-prepare-authentication).

    ```console
    POST /_security/oidc/prepare
    {
      "realm" : "oidc1"
    }
    ```

2. Handle the response to `/_security/oidc/prepare`. The response from {{es}} will contain 3 parameters: `redirect`, `state`, `nonce`. The custom web application would need to store the values for `state` and `nonce` in the user’s session (client side in a cookie or server side if session information is persisted this way) and redirect the user’s browser to the URL that will be contained in the `redirect` value.
3. Handle a subsequent response from the OP. After the user is successfully authenticated with the OpenID Connect Provider, they will be redirected back to the callback/redirect URI. Upon receiving this HTTP GET request, the custom web app will need to make an HTTP POST request to `_security/oidc/authenticate`, again - authenticating as the `facilitator` user - passing the URL where the user’s browser was redirected to, as a parameter, along with the values for `nonce` and `state` it had saved in the user’s session previously. If more than one OpenID Connect realms are configured, the custom web app can specify the name of the realm to be used for handling this, but this parameter is optional. For more details, see [OpenID Connect authenticate](https://www.elastic.co/docs/api/doc/elasticsearch/operation/operation-security-oidc-authenticate).

    ```console
    POST /_security/oidc/authenticate
    {
      "redirect_uri" : "https://oidc-kibana.elastic.co:5603/api/security/oidc/callback?code=jtI3Ntt8v3_XvcLzCFGq&state=4dbrihtIAt3wBTwo6DxK-vdk-sSyDBV8Yf0AjdkdT5I",
      "state" : "4dbrihtIAt3wBTwo6DxK-vdk-sSyDBV8Yf0AjdkdT5I",
      "nonce" : "WaBPH0KqPVdG5HHdSxPRjfoZbXMCicm5v1OiAj0DUFM",
      "realm" : "oidc1"
    }
    ```

    Elasticsearch will validate this and if all is correct will respond with an access token that can be used as a `Bearer` token for subsequent requests and a refresh token that can be later used to refresh the given access token as described in [Get token](https://www.elastic.co/docs/api/doc/elasticsearch/operation/operation-security-get-token).

4. At some point, if necessary, the custom web application can log the user out by using the [OIDC logout API](https://www.elastic.co/docs/api/doc/elasticsearch/operation/operation-security-oidc-logout) passing the access token and refresh token as parameters. For example:

    ```console
    POST /_security/oidc/logout
    {
      "token" : "dGhpcyBpcyBub3QgYSByZWFsIHRva2VuIGJ1dCBpdCBpcyBvbmx5IHRlc3QgZGF0YS4gZG8gbm90IHRyeSB0byByZWFkIHRva2VuIQ==",
      "refresh_token": "vLBPvmAB6KvwvJZr27cS"
    }
    ```

    If the realm is configured accordingly, this may result in a response with a `redirect` parameter indicating where the user needs to be redirected in the OP in order to complete the logout process.




