# Deploy a multi proxy system

In order to deploy a multi proxy system, there are some considerations to be made:
- How to connect the proxies to `q_web`

## Connecting multiple proxies to `q_web`
As Q uses mainly HTTPS to transfer the collected data to the `q_web` instance, 
you have to deal with their client and server certificates.

In order to avoid the need to update the server certificate of `q_web` each time you add / remove a new proxy,
it is a good idea to use a domain as address. In a company-based environment, you will often have to deal with NATs,
so it is not really feasible to add the NAT address to your certificate.

You can set the domain on your local resolver of your `q_proxy` instance
to point to the address `q_web` is seen by that instance.