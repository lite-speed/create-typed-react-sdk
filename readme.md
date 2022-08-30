# create-typed-react-sdk

## Rough Notes

```ts
/************************************ */
/************************************ */
// Server
// index.ts
/************************************ */
/************************************ */



import { createTypedSDK, DoFetch } from "create-typed-sdk";
import { createTypedReactSDK } from "create-typed-react-sdk";
import axios from "axios";

// Link the web-app to the sever
/*
package.json example
...
"dependencies": {
    "server": "link:../server"
  },
...
*/

// Make sure you wrap the react app with the provider
/*
function Wrapper() {
  return (
    <reactApiSDK.SDKProvider>
      <App />
    </reactApiSDK.SDKProvider>
  );
}

export default Wrapper;
*/

///-------------
// Client
///-------------
// IMPORTANT to ONLY IMPORT type so backend code doesn't leak into the front end builds
import type * as apiTypes from "./api";
const doFetch: DoFetch = async (p) => {
  const r1 = await axios.post(
    `http://localhost:3040/example-prefix/${p.path.join("/")}`,
    {
      argument: p.arg,
    }
  );
  return r1.data;
};
export const apiSDK = createTypedSDK<typeof apiTypes>({ doFetch });

export const reactApiSDK = createTypedReactSDK<typeof apiTypes>({ doFetch });





/************************************ */
/************************************ */
// Server
// server.ts
/************************************ */
/************************************ */


import fastify from "fastify";
import cors from "@fastify/cors";
import * as api from "./api.js";
import { collectApiFunctions } from "create-typed-sdk";

// Notes
/*
tsconfig.json
...
compilerOptions: {
    ...
    "declaration": true,
    ...
}
...

package.json
...
  "main": "index.js",
  "module": "dist/index.js",
  "types": "dist/index.d.ts",
  "type": "module",
...
*/

///-------------
// Server
///-------------

// Generic app example, express would would work fine as well
const app = fastify({
  logger: true,
});
app.register(cors, { origin: "*" });

// Example root
app.get("/", (request, reply) => {
  reply.send({ hello: "world" });
});

// Bind each function to an endpoint
collectApiFunctions(api).forEach((a) => {
  app.post("/example-prefix/" + a.path.join("/"), async (req, resp) => {
    if (typeof req.body !== "object" && !(req.body as any).argument) {
      throw new Error("Required property `argument` missing in post body!");
    }
    const val = await a.fn((req.body as any).argument);
    return val;
  });
});

// Run the server!
app.listen({ port: 3040 }, (err, address) => {
  if (err) throw err;
  console.log(`Server is now listening on ${address}`);
});
```