# dapr-demos

This is my repo for learning and demo'ing dapr

## Assumptions

- You have NodeJS 14 or higher installed
- You have Docker Desktop installed
- You are using a \*nix based OS (i.e., WSL2, Ubuntu, macOS, etc)

## hello-service

This service uses ExpressJS to return the word `hello` in randomized casing.

To create the new app:

```bash
mkdir hello-service
cd hello-service
npm init
npm install express
cat << EOF > app.js
const express = require("express");
const app = express();

app.get("/sayHello", (_, res) => {
  let hello = "";
  for (const letter of "hello") {
    const isUpperCase = Math.round(Math.random());
    hello += isUpperCase ? letter.toUpperCase() : letter;
  }
  console.log(\`Sending: \${hello}\`);
  res.send(hello);
});

const port = 8088;
app.listen(port, () => console.log(\`Listening on port \${port}\`));
EOF
```

To run the app, run this command:

```bash
node app.js
```

To run dapr, run this command:

```bash
dapr run --app-id hello-service --app-port 8088 -- node app.js
```
