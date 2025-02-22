# @turnkey/http

[![npm](https://img.shields.io/npm/v/@turnkey/http?color=%234C48FF)](https://www.npmjs.com/package/@turnkey/http)

A lower-level, fully typed HTTP client for interacting with [Turnkey](https://turnkey.io) API.

For signing transactions and messages, check out the higher-level [`@turnkey/ethers`](/packages/ethers/) Signer.

API Docs: https://turnkey.readme.io/

## Getting started

```bash
$ npm install @turnkey/http
```

Before making http calls, initialize the package with your credentials:

```typescript
import { TurnkeyApi, init } from "@turnkey/http";

init({
  apiPublicKey: "...",
  apiPrivateKey: "...",
  baseUrl: "...",
});

// Now you can make authenticated requests!
const data = await TurnkeyApi.postGetWhoami({
  body: {
    organizationId: "...",
  },
});
```

## HTTP fetchers

`@turnkey/http` provides fully typed http fetchers for interacting with the Turnkey API. You can find all available methods [here](/packages/http/src/__generated__/services/coordinator/public/v1/public_api.fetcher.ts). The types of input parameters and output responses are also exported for convenience.

The OpenAPI spec that generates all fetchers is also [included](/packages/http/src/__generated__/services/coordinator/public/v1/public_api.swagger.json) in the package.

## `withAsyncPolling(...)` helper

All Turnkey mutation endpoints are asynchronous (with the exception of private key-related signing endpoints, e.g. `/submit/sign_transaction`, `/submit/sign_raw_payload`). To help you simplify async mutations, `@turnkey/http` provides a `withAsyncPolling(...)` wrapper. Here's a quick example:

```typescript
import {
  TurnkeyApi,
  withAsyncPolling,
  TurnkeyActivityError,
} from "@turnkey/http";

// Use `withAsyncPolling(...)` to wrap & create a fetcher with built-in async polling support
const fetcher = withAsyncPolling({
  request: TurnkeyApi.postCreatePrivateKeys,
});

// The fetcher remains fully typed. After submitting the request,
// it'll poll until the activity reaches a terminal state.
try {
  const activity = await fetcher({
    body: {
      /* ... */
    },
  });

  // Success!
  console.log(activity.result.createPrivateKeysResult?.privateKeyIds?.[0]);
} catch (error) {
  if (error instanceof TurnkeyActivityError) {
    // In case the activity is rejected, failed, or requires consensus,
    // a rich `TurnkeyActivityError` will be thrown. You can read from
    // `TurnkeyActivityError` to find out why the activity didn't succeed.
    //
    // For instance, if your activity requires consensus and doesn't have
    // enough approvals, you can get the `activityId` from `TurnkeyActivityError`,
    // store it somewhere, then re-fetch the activity via `.postGetActivity(...)`
    // when the required approvals/rejections are in place.
  }
}
```

## More examples

See [`createNewEthereumPrivateKey.ts`](/examples/with-ethers/src/createNewEthereumPrivateKey.ts) in the [`with-ethers`](/examples/with-ethers/) example.

## See also

- [`@turnkey/ethers`](/packages/ethers/): higher-level Turnkey Signer for [`Ethers`](https://docs.ethers.org/v5/api/signer/)
