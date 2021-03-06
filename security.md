# Brain security
Starting with release 2.0, the Brain graphical interface, web API and socket.io event pusher have several access restrictions by default. This document describes the Brain security measures and how to work with them.

## Security measures

### Graphical user interface authentication
The graphical interface (used to manage smartspots and access apps and tools) requires logging in with an email address and password. Login sessions are valid for thirty days by default; this is configurable through the `services` resource on the API.

Passwords are not stored on the server; only a cryptographically strong fingerprint is retained. If a user loses their password it will have to be reset through one of the interfaces described in [Access management](#access-management).

### Web API authentication
Applications are required to provide an API key with every HTTP request to the web API or new connection with the socket.io event pusher. The API key can be provided using either of the following methods:
* With a query parameter &mdash; `https://brain_host/api/endpoint?key=xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx`
* With an HTTP request header &mdash; `X-Api-Key: xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx`

Alternatively, being logged into the graphical frontend (i.e. having a valid session cookie) grants full access to the API. This is particularly useful for using the API manually. However, a same-origin policy is enforced for cookie authentication. Therefore browser applications written in Javascript cannot be authenticated in this way.

### Secure connections (TLS)
TLS security (i.e. `https` protocol) is enabled and enforced for Brains on \*.intellifi.nl domains. Brain virtual machine images shipped by Intellifi do not have TLS enabled by default, because this requires a valid certificate for the domain the Brain is going to be deployed on. See [Certificate installation](#certificate-installation) for how to enable TLS using your own certificate.

## Access management
A graphical utility for managing Brain users and API keys is available on the right of the menu bar as "Admin panel" (visible only to administrator users).

Users and keys can also be directly viewed and manipulated using the REST API (`/api/users` and `/api/keys` endpoints). Note that this requires being logged in as a user with administrator privileges; just providing an API key is not sufficient to access these resources.

Customers who manage their own Brain and have SSH server access can alternatively utilize a wizard-style command line tool for managing users, keys and low-level security settings (including the possibility to disable API authentication). This tool can be started by logging into SSH and executing the command `accesstool`. It is the most direct way of managing security and works even when, for some reason, no administrator users are left on the Brain.

## Permissions
API keys have equivalent permissions. Every API key can perform any action on any resource, except for the protected resources `users` and `keys`.

User accounts are also mostly equivalent, except for the following properties:
* Administrator &mdash; if a user is administrator, they can manage users and API keys
* Locked &mdash; if a user is locked, they cannot edit their own user details (e.g. demo/guest user)

## Default credentials
Every new Brain installation automatically creates one user with administrator privileges and one API key. The user credentials are supplied by Intellifi after purchasing a Brain license. The API key can then be viewed by logging in and navigating to `/api/keys`.

## Certificate installation
This section is for customers who deploy a Brain on a domain name not managed by Intellifi and want to enable TLS security using a certificate for this domain.

1. Make sure your certificate:
    * Is valid for the domain name your Brain uses.
    * Is signed by a trusted authority.
    * Is exported to a `.key` and `.crt` file pair. The crt-file should contain all necessary information, including intermediate CA certificates if applicable.
2. Log into the server with an SFTP client. Create the directory `/tmp/cert` if it does not exist. Upload the `.key` and `.crt` file to this directory.
3. Log into the server with an SSH client. Execute the command `sudo httpstool`, select the *install* option and enter your domain name.

If you ever have to disable TLS again for some reason, repeat step 3 with the *uninstall* option.
