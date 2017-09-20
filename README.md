## Spring Boot container to container mTls

### What does Cloud Foundry offering here

* [Container to Container networking on overlay network](https://github.com/cloudfoundry-incubator/cf-networking-release)

* [Java Buildpack](https://github.com/cloudfoundry/java-buildpack/blob/master/docs/framework-container_security_provider.md#security-provider) add both diego identity key/cert to keystore and bosh trusted certs to truststore.


### Demo Concepts

* One front end Application (client) with /hello endpoint.
* One Backend application (server) with /hello endpoint.
* Server uses [.profile.d](server/src/main/resources/profile/.profile.d/init.sh) to load the certs from diego instance identity env vars
* Server opens two ports: one for 8080 to expose the internal IP through gorouter. One for ssl works with overlay network (c-2-c)
* Server mutual ssl is **want**, which means client certificate is optional
* /hello endpoint requires client certificate to be trusted and enforced by spring security.
* Client uses restTemplate to communicate with server through mutual SSL
* Enable java ssl debug

### How to Run

* push client/server to cf

```
cf push
```
* Add network policy to open up the communication

```
cf add-network-policy client-c2c --destination-app server-c2c --port 8443 --protocol tcp
```

* Find the server IP

```
Shaozhen-Ding-MacBook-Pro-3:c-2-c sding$ curl server-c2c.cfapps.haas-60.pez.pivotal.io/nomutual
this has no mutual auth. My Internal ip is:10.255.175.197
```

* Configure the client

```
cf set-env client-c2c BACKEND_SERVER https://10.255.175.197:8443
cf restart client-c2c
```

* Curl the client hello endpoint.

Client will contact with server through mutual ssl by using the diego identity certificate (The CN is CF instance ID)

```
Shaozhen-Ding-MacBook-Pro-3:c-2-c sding$ curl http://client-c2c.cfapps.haas-60.pez.pivotal.io/hello
Thanks for authenticating with X509, certificate CN is: c4aae08f-3612-4886-61df-3643
```
