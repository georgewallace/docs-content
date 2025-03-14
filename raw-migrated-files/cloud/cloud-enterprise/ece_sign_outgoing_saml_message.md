# Sign outgoing SAML messages [ece-sign-outgoing-saml-message]

If configured, Elastic Stack will sign outgoing SAML messages.

As a prerequisite, you need to generate a signing key and a self-signed certificate. You need to share this certificate with your SAML Identity Provider so that it can verify the received messages. The key needs to be unencrypted. The exact procedure is system dependent, you can use for example `openssl`:

```sh
openssl req -new -x509 -days 3650 -nodes -sha256 -out saml-sign.crt -keyout saml-sign.key
```

Place the files under the `saml` folder and add them to the existing SAML bundle, or [create a new one](ece-add-custom-bundle-plugin.md).

In our example, the certificate and the key will be located in the path `/app/config/saml/saml-sign.{crt,key}`:

```sh
$ tree .
.
└── saml
    ├── saml-sign.crt
    └── saml-sign.key
```

Make sure that the bundle is included with your deployment.

Adjust your realm configuration accordingly:

```sh
    signing.certificate: /app/config/saml/saml-sign.crt <1>
    signing.key: /app/config/saml/saml-sign.key <2>
```

1. The path to the SAML signing certificate that was uploaded.
2. The path to the SAML signing key that was uploaded.


When configured with a signing key and certificate, Elastic Stack will sign all outgoing messages (SAML Authentication Requests, SAML Logout Requests, SAML Logout Responses) by default. This behavior can be altered by configuring `signing.saml_messages` appropriately with the comma separated list of messages to sign. Supported values are `AuthnRequest`, `LogoutRequest` and `LogoutResponse` and the default value is `*`.

For example:

```sh
xpack:
  security:
    authc:
      realms:
        saml-realm-name:
          order: 2
          ...
          signing.saml_messages: AuthnRequest <1>
          ...
```

1. This configuration ensures that only SAML authentication requests will be sent signed to the Identity Provider.



