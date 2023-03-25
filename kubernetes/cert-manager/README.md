I'm using Let's Encrypt and Cloudflare. You'll need a token with the following scopes, to the zones you with to obtain certificates for. 

```
Zone : DNS : READ
Zone : DNS : EDIT
```

This token then needs to be added as a secret, and you can view a sample in secret-cloudflare.yaml. At present, the secret _MUST_ be added to the cert-manager namespace or it will not work!

Then in your dns01 solver, ensure you are providing the email of your Cloudflare account, along with referencing the secret we defined. (apiTokenSecretRef)

See ingress for an example of using these issued certificates.
