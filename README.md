# saltoapis-es

This repository contains the source code for the ES Salto APIs SDK.

> These APIs use the connectrpc framework and the gRPC protocol.
> Refer to the official documentation to learn more about [connectrpc](https://connectrpc.com/docs/introduction) and [gRPC](https://grpc.io/docs/).

## Documentation

- **[Nebula API documentation](https://developer.saltosystems.com/nebula)** - Complete API reference and conceptual guides
- **[Authentication Guide](https://developer.saltosystems.com/nebula/api/authentication/)** - OAuth2 flows and credential setup
- **This README** - SDK-specific installation and TypeScript/JavaScript examples

## Getting started

Follow these steps to quickly set up and use the SDK:

1. **Authenticate with GitHub Package Registry**
2. **Install the SDK package(s) you need**
3. **Configure authentication for your application**

See details below.

## Installation

### 1. Authenticate with GitHub Package Registry

You need a GitHub personal access token with the `read:packages` permission. Generate one at [GitHub Tokens](https://github.com/settings/tokens) and copy it.

Login to the registry:

```sh
npm login --scope=@saltoapis --auth-type=legacy --registry=https://npm.pkg.github.com
# Username: <your GitHub username>
# Password: <your access token>
```

Alternatively, add this to your `.npmrc`:

```ini
//npm.pkg.github.com/:_authToken=TOKEN
@saltoapis:registry=https://npm.pkg.github.com
```

### 2. Install the SDK package

Find the package you need at [saltoapis-es NPM packages](https://github.com/saltoapis/saltoapis-es/packages?ecosystem=npm) and install it:

```sh
npm install @saltoapis/<package-name>
```

## Understanding Salto Nebula concepts

Before diving into the code, familiarize yourself with these core concepts:

- **Installation** - Your Salto Nebula environment (required as `parent` in most API calls)
- **Access points** - Physical entry or exit points controlled by electronic locks
- **Electronic locks** - Smartlocks connected to access points
- **Users** - People who can access your installation
- **Access rights** - Permissions linking users to access points
- **Units** - Logical sub-grouping of access control elements within an installation (rooms, apartments, offices)

**[See also the domain model guide](https://support.saltosystems.com/homelok/user-guide/property-manager/getting-started/best-practices/#entity-relationship) and [glossary](https://developer.saltosystems.com/nebula/glossary/)**.

## Available services

This SDK provides clients for the following core Nebula services (among others):

| Service | Package | Documentation |
| --- | --- | --- |
| Users | `@saltoapis/nebula-user-v1` | [API Reference](https://developer.saltosystems.com/nebula/api/salto/nebula/user/v1/) |
| Electronic locks | `@saltoapis/nebula-electroniclock-v1` | [API Reference](https://developer.saltosystems.com/nebula/api/salto/nebula/electroniclock/v1/) |
| Access points | `@saltoapis/nebula-accesspoint-v1` | [API Reference](https://developer.saltosystems.com/nebula/api/salto/nebula/accesspoint/v1/) |
| Access rights | `@saltoapis/nebula-accessright-v1` | [API Reference](https://developer.saltosystems.com/nebula/api/salto/nebula/accessright/v1/) |
| Units | `@saltoapis/nebula-unit-v1` | [API Reference](https://developer.saltosystems.com/nebula/api/salto/nebula/unit/v1/) |
| Events | `@saltoapis/nebula-event-v1` | [API Reference](https://developer.saltosystems.com/nebula/api/salto/nebula/event/v1/) |

[View all available packages](https://github.com/saltoapis/saltoapis-es/packages?ecosystem=npm)

## Setup

This SDK publishes NPM packages in GitHub's Package Registry. You can see all published packages in the [saltoapis-es GitHub Packages listing](https://github.com/saltoapis/saltoapis-es/packages/).

Even though the packages are public, GitHub does not support downloading packages anonymously so you will need a GitHub personal access token with the `read:packages` permission set to retrieve the packages.

To setup your NPM environment with your access token do the following:

### 1. Create a personal access token

Access [GitHub Tokens](https://github.com/settings/tokens) and press "Generate new token". Give the token the name and expiration you want and be sure to check the `read:packages` scope. Then press "Generate token".

You will then have to copy your token (because it's going to be shown to you only once).

### 2. Add the GitHub NPM repository

You have to add the saltoapis NPM repository ([https://npm.pkg.github.com](https://npm.pkg.github.com)) to access the packages. You can do this by using the following npm command:

```sh
npm login --scope=@saltoapis --auth-type=legacy --registry=https://npm.pkg.github.com

# Username: <your username>
# Password: <your access token>
```

Or you can also manually edit your `.npmrc` file to include the following:

```ini
//npm.pkg.github.com/:_authToken=TOKEN
@saltoapis:registry=https://npm.pkg.github.com
```

### 3. Use the packages

If everything is setup correctly you will be able to include dependencies to the packages listed in [saltoapis-es NPM packages](https://github.com/saltoapis/saltoapis-es/packages?ecosystem=npm) in your project.

## Authentication example

The SDK provides a simple gRPC interceptor that will automatically get and refresh valid access tokens and include them in all gRPC requests.

> **Note**
> You should always request the following scopes when authenticating:
>
> ```text
> openid, offline, profile, email, https://saltoapis.com/auth/nebula
> ```

```ts
import { createGrpcTransport } from '@connectrpc/connect-node';
import { createPromiseClient } from '@connectrpc/connect';

const clientID = "your client id";
const clientSecret = "your client secret";
const scopes = ["openid", "offline", "profile", "email", "https://saltoapis.com/auth/nebula"];

// Create the saltoapis auth interceptor with your user credentials
const authInterceptor = SaltoapisAuthInterceptor.withClientCredentials(clientID, clientSecret, scopes);

// Create the gRPC transport
const transport = createGrpcTransport({
    httpVersion: '2',
    baseUrl: "https://nebula.saltoapis.com",
    interceptors: [ authInterceptor ]
});

// Now you can use the transport to instantiate any gRPC service
const client = createPromiseClient(OperationService, transport);
const res = await client.listOperations({ pageSize: 10 });
```

You can find more information about authentication at [Salto Nebula API authentication](https://developer.saltosystems.com/nebula/api/authentication/).

## Usage examples

>**Tip**: All examples require your installation ID (e.g., `installations/01GZX9G7PMV3VRMM6PJOC3SHQV`).

### List users

```typescript
import { createGrpcTransport } from '@connectrpc/connect-node';
import { createPromiseClient } from '@connectrpc/connect';
import { UserService } from '@saltoapis/nebula-user-v1';
import { SaltoapisAuthInterceptor } from '@saltoapis/saltoapis-auth';

// Setup authentication
const authInterceptor = SaltoapisAuthInterceptor.withClientCredentials(
  process.env.CLIENT_ID!,
  process.env.CLIENT_SECRET!,
  ['openid', 'offline', 'profile', 'email', 'https://saltoapis.com/auth/nebula']
);

// Create transport
const transport = createGrpcTransport({
  httpVersion: '2',
  baseUrl: 'https://nebula.saltoapis.com',
  interceptors: [authInterceptor]
});

// Create user service client
const userClient = createPromiseClient(UserService, transport);

// List users with pagination
const response = await userClient.listUsers({
  parent: 'installations/your-installation-id',
  pageSize: 50,
  pageToken: '' // Use response.nextPageToken for subsequent pages
});

console.log(`Found ${response.users.length} users`);
response.users.forEach(user => {
  console.log(`- ${user.displayName} (${user.name})`);
  if (user.validUntil) {
    console.log(`  Valid until: ${user.validUntil}`);
  }
});

// Iterate through all pages
let pageToken = '';
let allUsers = [];
do {
  const page = await userClient.listUsers({
    parent: 'installations/your-installation-id',
    pageSize: 100,
    pageToken
  });
  allUsers.push(...page.users);
  pageToken = page.nextPageToken;
} while (pageToken);

console.log(`Total users: ${allUsers.length}`);
```

### Create a user

```typescript
import { User } from '@saltoapis/nebula-user-v1';

const newUser = await userClient.createUser({
  parent: 'installations/your-installation-id',
  userId: 'john-doe', // Optional custom ID
  user: {
    displayName: 'John Doe',
    givenName: 'John',
    familyName: 'Doe',
    email: 'john.doe@example.com',
    // validUntil can be set for temporary access
    // validUntil: { seconds: BigInt(Math.floor(Date.now() / 1000) + 86400 * 30) }
  }
});

console.log(`Created user: ${newUser.name}`);
```

### Filter users

```typescript
// List users whose access has expired
const expiredUsers = await userClient.listUsers({
  parent: 'installations/your-installation-id',
  filter: `valid_until < "${new Date().toISOString()}"`,
  pageSize: 100
});

console.log(`Found ${expiredUsers.users.length} expired users`);
```

See also the [Nebula API documentation](https://developer.saltosystems.com/nebula/api/filter/) for more information on how to use the filter.

### Error handling

```typescript
import { ConnectError } from '@connectrpc/connect';
import { Code } from '@connectrpc/connect';

try {
  await userClient.createUser({
    parent: 'installations/your-installation-id',
    userId: 'duplicate-user',
    user: { displayName: 'Test User' }
  });
} catch (err) {
  if (err instanceof ConnectError) {
    switch (err.code) {
      case Code.AlreadyExists:
        console.error('User already exists');
        break;
      case Code.PermissionDenied:
        console.error('Insufficient permissions');
        break;
      case Code.InvalidArgument:
        console.error('Invalid user data:', err.message);
        break;
      default:
        console.error(`gRPC error [${err.code}]: ${err.message}`);
    }
  } else {
    throw err;
  }
}
```

## Troubleshooting

- **401 Unauthorized or 404 Not Found when installing packages**
  - Make sure your personal access token has the `read:packages` scope.
  - Ensure your `.npmrc` is correctly configured and you are logged in to the correct registry.
  - Double-check the package name and scope.

- **npm ERR! code E401 or E403**
  - Your token may have expired or been revoked. Generate a new one and update your `.npmrc` or re-run `npm login`.

- **Cannot find package**
  - Confirm the package exists at [saltoapis-es NPM packages](https://github.com/saltoapis/saltoapis-es/packages?ecosystem=npm) and you are using the correct name.

- **Still having issues?**
  - Try logging out (`npm logout --scope=@saltoapis --registry=https://npm.pkg.github.com`) and logging in again.
  - Check for typos in your `.npmrc` file.
  - Consult the [GitHub Packages documentation](https://docs.github.com/en/packages/working-with-a-github-packages-registry/working-with-the-npm-registry) for more help.
