# saltoapis-es

This repository contains the source code for the ES SALTO APIs SDK.

> These APIs use the connectrpc framework and the gRPC protocol.
> Refer to the official documentation to learn more about [connectrpc](https://connectrpc.com/docs/introduction) and [gRPC](https://grpc.io/docs/).

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

const clientID = "<YOUR_CLIENT_ID>";
const clientSecret = "<YOUR_CLIENT_SECRET>";
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

## Minimal example

```ts
import { createGrpcTransport } from '@connectrpc/connect-node';
import { createPromiseClient } from '@connectrpc/connect';
import { SaltoapisAuthInterceptor } from '@saltoapis/auth';
import { UserService } from '@saltoapis/nebula-user-v1';

const clientID = "<YOUR_CLIENT_ID>";
const clientSecret = "<YOUR_CLIENT_SECRET>";
const scopes = [
  "openid", "offline", "profile", "email", "https://saltoapis.com/auth/nebula"
];

// Create the saltoapis auth interceptor
const authInterceptor = SaltoapisAuthInterceptor.withClientCredentials(
  clientID, clientSecret, scopes
);

// Create the gRPC transport
const transport = createGrpcTransport({
  httpVersion: '2',
  baseUrl: "https://nebula.saltoapis.com",
  interceptors: [authInterceptor]
});

// Create the client
const client = createPromiseClient(UserService, transport);

async function listUsers() {
  const response = await client.listUsers({
    parent: "<YOUR_INSTALLATION_NAME>",
    pageSize: 10
  });
  for (const user of response.users) {
    console.log(user.displayName);
  }
}

listUsers().catch(console.error);
```

## Create, update, and delete user

```ts
import { createGrpcTransport } from '@connectrpc/connect-node';
import { createPromiseClient } from '@connectrpc/connect';
import { SaltoapisAuthInterceptor } from '@saltoapis/auth';
import { UserService } from '@saltoapis/nebula-user-v1';
import { FieldMask } from 'google-protobuf/google/protobuf/field_mask_pb';

const CLIENT_ID = "<YOUR_CLIENT_ID>";
const CLIENT_SECRET = "<YOUR_CLIENT_SECRET>";
const INSTALLATION_NAME = "<YOUR_INSTALLATION_NAME>";

const authInterceptor = SaltoapisAuthInterceptor.withClientCredentials(
  CLIENT_ID, CLIENT_SECRET, ["https://saltoapis.com/auth/nebula"]
);

const transport = createGrpcTransport({
  httpVersion: '2',
  baseUrl: "https://nebula.saltoapis.com",
  interceptors: [authInterceptor]
});

const client = createPromiseClient(UserService, transport);

async function main() {
  const email = "example@saltosystems.com";

  // Create a new user
  const created = await client.createUser({
    parent: INSTALLATION_NAME,
    user: {
      givenName: "Test",
      familyName: "User",
      displayName: "Test User",
      email
    }
  });

  // List users
  let response = await client.listUsers({ parent: INSTALLATION_NAME, pageSize: 10 });
  console.log("Initial users after create:");
  response.users.forEach(user => console.log(user.displayName));

  // Update the user's family name
  await client.updateUser({
    user: {
      name: created.user?.name,
      familyName: "User Updated"
    },
    updateMask: FieldMask.fromObject({ paths: ["family_name"] })
  });

  response = await client.listUsers({ parent: INSTALLATION_NAME, pageSize: 10 });
  console.log("\nUsers after update:");
  response.users.forEach(user => console.log(user.displayName));

  // Delete the user
  await client.deleteUser({ name: created.user?.name });

  response = await client.listUsers({ parent: INSTALLATION_NAME, pageSize: 10 });
  console.log("\nUsers after delete:");
  response.users.forEach(user => console.log(user.displayName));
}

main().catch(console.error);
```

## Remote unlock

```ts
import { createGrpcTransport } from '@connectrpc/connect-node';
import { createPromiseClient } from '@connectrpc/connect';
import { SaltoapisAuthInterceptor } from '@saltoapis/auth';
import { ElectronicLockService } from '@saltoapis/nebula-electroniclock-v1';
import { AccessPointService } from '@saltoapis/nebula-accesspoint-v1';

const CLIENT_ID = "<YOUR_CLIENT_ID>";
const CLIENT_SECRET = "<YOUR_CLIENT_SECRET>";
const INSTALLATION_NAME = "<YOUR_INSTALLATION_NAME>";

const authInterceptor = SaltoapisAuthInterceptor.withClientCredentials(
  CLIENT_ID, CLIENT_SECRET, ["https://saltoapis.com/auth/nebula"]
);

const transport = createGrpcTransport({
  httpVersion: '2',
  baseUrl: "https://nebula.saltoapis.com",
  interceptors: [authInterceptor]
});

const elClient = createPromiseClient(ElectronicLockService, transport);
const apClient = createPromiseClient(AccessPointService, transport);

async function remoteUnlock() {
  // Get the first electronic lock
  const responseEl = await elClient.listElectronicLocks({
    parent: INSTALLATION_NAME,
    pageSize: 1
  });

  if (!responseEl.electronicLocks?.length) {
    console.log("No electronic locks found");
    return;
  }

  const electronicLock = responseEl.electronicLocks[0];
  console.log("Display name:", electronicLock.displayName);
  console.log("Name:", electronicLock.name);
  console.log("Electronic lock's Access Point:", electronicLock.accessPoint);

  // Unlock the access point
  await apClient.unlockAccessPoint({
    name: electronicLock.accessPoint
  });

  console.log(`Electronic lock '${electronicLock.displayName}' opened`);
}

remoteUnlock().catch(console.error);
```

## Create and delete unit

```ts
import { createGrpcTransport } from '@connectrpc/connect-node';
import { createPromiseClient } from '@connectrpc/connect';
import { SaltoapisAuthInterceptor } from '@saltoapis/auth';
import { UnitService } from '@saltoapis/nebula-unit-v1';

const CLIENT_ID = "<YOUR_CLIENT_ID>";
const CLIENT_SECRET = "<YOUR_CLIENT_SECRET>";
const INSTALLATION_NAME = "<YOUR_INSTALLATION_NAME>";

const authInterceptor = SaltoapisAuthInterceptor.withClientCredentials(
  CLIENT_ID, CLIENT_SECRET, ["https://saltoapis.com/auth/nebula"]
);

const transport = createGrpcTransport({
  httpVersion: '2',
  baseUrl: "https://nebula.saltoapis.com",
  interceptors: [authInterceptor]
});

const client = createPromiseClient(UnitService, transport);

async function createAndDeleteUnit() {
  // Create a new unit with privacy settings
  const response = await client.createUnit({
    parent: INSTALLATION_NAME,
    unit: {
      displayName: "Test Unit",
      privacySettings: { enabled: true }
    }
  });

  console.log(`Created unit: ${response.unit?.displayName}`);

  // Delete the unit
  await client.deleteUnit({ name: response.unit?.name });
  console.log(`Deleted unit: ${response.unit?.displayName}`);
}

createAndDeleteUnit().catch(console.error);
```

See also the [Nebula API documentation](https://developer.saltosystems.com/nebula) for detailed information on available services and methods.

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
