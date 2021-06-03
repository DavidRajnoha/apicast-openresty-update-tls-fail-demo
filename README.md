How to reproduce the 503 response with the "attempt to send data on a closed socket" error log

Create secrets and deploy gateway

````
oc create secret generic tls-apic-drajnoha-openresty-config-url --type kubernetes.io/basic-auth --from-literal=password=https://<ACCESS_TOKEN>@<TENANT_NAME>-admin.<WILDCARD_DOMAIN>
````

````
oc new-app --file=apicast.yml --param=APICAST_NAME=tls-apic-drajnoha-openresty --param=AMP_APICAST_IMAGE=quay.io/3scale/apicast:Openresty1.19.3-builder --param=DEPLOYMENT_ENVIRONMENT=staging  --param=CONFIGURATION_LOADER=lazy --param=CONFIGURATION_CACHE=0 --param=LOG_LEVEL=info --param=CONFIGURATION_URL_SECRET=tls-apic-drajnoha-openresty-config-url
````

````
oc set env dc tls-apic-drajnoha-openresty APICAST_HTTPS_PORT=8443 APICAST_HTTPS_CERTIFICATE=/var/apicast/secrets/tls.crt APICAST_HTTPS_CERTIFICATE_KEY=/var/apicast/secrets/tls.key
````

````
oc create secret tls tls-apic-drajnoha-openresty-tls --cert certificate --key key
````

````
oc set volume dc/tls-apic-drajnoha-openresty --add --name tls-apic-drajnoha-openresty-volume --mount-path /var/apicast/secrets --secret-name tls-apic-drajnoha-openresty-tls
````

````
oc patch service tls-apic-drajnoha-openresty -p '{"spec": {"ports": [{"name": "https", "port": 8443, "protocol": "TCP"}]}}'
````

Create route

````
oc create route passthrough route-staging --service=tls-apic-drajnoha-openresty --hostname=drajnoha-openresty-tls.<WILDCARD_DOMAIN>
````
 
Create application using the created selfmanaged apicast.
Send an unsecured request.
Observe 503 response and error in the apicast logs.
The 503 is present when the apicast is trying to load the service configuration.
When these are cached, the error is not there.
 
````
curl -k  "https://drajnoha-openresty-tls.<WILDCARD_DOMAIN>:443/?user_key=<APP_USER_KEY>"
````
