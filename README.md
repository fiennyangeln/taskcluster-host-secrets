Taskcluster Host Secrets
========================
This is a service which generates taskcluster credentials for machines based on
their IP address, after doing a forward and reverse DNS verification.  The service
will grant a set of temporary credential

## Example

```
$ curl localhost:8080/v1/credentials
{
  "credentials": {
    "clientId": "localhost.localdomain",
    "accessToken": "<snip>",
    "certificate": "{\"version\":1,\"scopes\":[\"assume:project:testing-host-secrets:host:localdomain.localhost\",\"assume:project:testing-host-secrets:host:localhost6\",\"assume:project:testing-host-secrets:host:localdomain6.localhost6\"],\"start\":1486575908837,\"expiry\":1486575908837,\"seed\":\"<snip>\",\"signature\":\"<snip>\",\"issuer\":\"<snip>\"}"
  }
}
```

Note that this generates a scope for all DNS names which this IP has when doing a reverse lookup.  In this example case, that is:

* `assume:project:testing-host-secrets:host:localdomain.localhost`
* `assume:project:testing-host-secrets:host:localhost6`
* `assume:project:testing-host-secrets:host:localdomain6.localhost6`

The role specified in these scopes (`project:testing-host-secrets:host:*`) can
be given other roles to expand on what they can do.  This includes storing secrets
using the taskcluster-secrets service.

## Credentials
The taskcluster credentials that are to be used with this service must be permanent
credentials (temporary credentials cannot generate other temporary).  Those credentials
must have the scope `$TASKCLUSTER_SCOPE_BASE*` (e.g. `assume:project:testing-host-secrets:host:*`)
or a subset of the valid hostnames.

## Usage
Unlike other taskcluster services, this service is designed to be deployed in a
datacenter along side physical workers.  There are two ways to deploy the project:
using the standard `git clone <...> && cd <...> && npm install` flow or using a
tarball generated by the package.sh script.

### Standard Deployment
You can do a standard deployment of this service like many other node applications.
``` bash 
git clone <...>
cd taskcluster-host-secrets
npm install
export TASKCLUSTER_SCOPE_BASE="assume:project:testing-host-secrets:host:"
export TASKCLUSTER_CREDENTIALS_EXPIRE="1 day"
export TASKCLUSTER_ALLOWED_IPS="\*"
export TASKCLUSTER_CLIENT_ID=<snip>
export TASKCLUSTER_ACCESS_TOKEN=<snip>
export PORT=8080
export NODE_ENV=production
node lib/main server
```

### package.sh Based Deployment
This script generates a tarball meant to ease deployment on systems which do not
already have a node environment.  This tarball can be regenerated very easily.
```
./package.sh && echo done
```
Which will create a `dist.tar.gz` file.  This archive contains a full node environment
as well as a wrapper script.  To launch the server, you can do this:

```
tar xf host-secrets.tar.gz
cd dist
./start.sh
```
In addition to the standard environment variables, this wrapper script has the following command line options (overrides environment):

* `--scope-base`: equivalent to `TASKCLUSTER_SCOPE_BASE`
* `--expires`: equivalent to `TASKCLUSTER_CREDENTIALS_EXPIRE`
* `--allowed-ips`: equivalent to `TASKCLUSTER_ALLOWED_IPS`
* `--client-id`: equivalent to `TASKCLUSTER_CLIENT_ID`
* `--token`: equivalent to `TASKCLUSTER_ACCESS_TOKEN`
* `--port`: equivalent to `PORT`
* `--env`: equivalent to `NODE_ENV`
