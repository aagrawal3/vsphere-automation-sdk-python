## About the Virtual Infrastructure JSON API
Starting with vCenter 8.0 Update 1, VMware adds support
for a new HTTP/JSON-based wire protocol as an alternative
to SOAP/XML. The new protocol is described using the
industry-standard OpenAPI specification, version 3.0,
and can be used to access the popular VIM APIs. For more
information see the *OpenAPI Specification* and the
*Web Services Programming Guide*, part of the
[vSphere Management SDK](https://developer.vmware.com/web/sdk/vsphere-management).

## Getting Started with the Virtual Infrastructure JSON API

### Retrieving the Service Instance with the JSON Protocol
[ServiceInstance](service-instance/) is a singleton managed object that acts as
a gateway to the resources available on the server. You can
locate other important singletons such as [SessionManager](session-manager/) and
[AuthorizationManager](authorization-manager/) from the properties of the [ServiceInstance](service-instance/).
[ServiceInstance](service-instance/) is accessible without authentication, so you
can retrieve a reference to the [SessionManager](session-manager/) which you can
use to authenticate a session.

<span class="label operation-verb-label get label-info">GET</span>
[Service Instance Get Content](sdk/vim25/release/ServiceInstance/moId/content/get/)

The response is a JSON object that has a `sessionManager` key which contains
a [ManagedObjectReference](data-structures/ManagedObjectReference/) to the [SessionManager managed object](session-manager/).
The value of the `value` key of this [ManagedObjectReference](data-structures/ManagedObjectReference/) must be used
for the `moId` path parameter when invoking [SessionManager](session-manager/) operations.

For example, the [SessionManager](session-manager/) `moId` can be obtained by
issuing a GET with `curl` like this:
```bash
SESSION_MANAGER_MOID=$(curl -s https://$VC/sdk/vim25/{$RELEASE}/ServiceInstance/ServiceInstance/content \
                          | jq .sessionManager.value \
                          | xargs -t)
```

### Authenticating a JSON Client with the Session Manager
Most method calls must carry a session ID to authenticate with
the server at the time of the call. The session ID is a temporary
substitute for username and password, thereby limiting risk to
the principal's credentials.

<span class="label operation-verb-label post label-info">POST</span>
[Session Manager Login](sdk/vim25/release/SessionManager/moId/Login/post/)

For example, the Login operation can be performed by issuing a POST with
`curl` and capturing the `vmware-api-session-id` HTTP header from the result:
```bash
curl -v "https://$VC/sdk/vim25/{$RELEASE}/SessionManager/$SESSION_MANAGER_MOID/Login" \
     -H 'Content-Type: application/json' \
     -d '{"userName": "Administrator@vsphere.local", "password": "betyoucantguess"}'

< HTTP/1.1 200 OK
< content-type: application/json
< vmware-api-session-id: c3ab4eb20bf5639b47af96e06118428579c40266
...
```

### Use the session ID in subsequent calls
On subsequent API calls you will need to include the session ID returned in the above call.
Copy the session ID header into your request, as follows:
```http
GET /sdk/vim25/{$RELEASE}/VirtualMachine/vm-495/config

vmware-api-session-id: c3ab4eb20bf5639b47af96e06118428579c40266
```