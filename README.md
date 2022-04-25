# dapr-demos

This is my repo for learning and demo'ing dapr

## Assumptions

- You have NodeJS 14 or higher installed
- You have Docker Desktop installed
- You are using a \*nix based OS (i.e., WSL2, Ubuntu, macOS, etc)

## Microservices

This repo includes a few services as demonstrated in the book [Introducing Distributed Application Runtime (Dapr): Simplifying Microservices Applications Development Through Proven and Reusable Patterns and Practices](https://www.amazon.com/Introducing-Distributed-Application-Runtime-Dapr/dp/1484269977)

Code Samples: https://github.com/Apress/introducing-dapr

### hello-service

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

### world-service

This service uses ExpressJS to return the word `world` in various languages.

To create the new app:

```bash
mkdir world-service
cd world-service
npm init
npm install express
cat << EOF > app.js
const express = require("express");
const app = express();

const worldTranslations = ["world",
    "свят", "svijet", "svět", "świat", "мир", "свет", "світ", "svet", "svetu",
    "lume", "verden", "wereld", "värld", "pasaulē", "pasaulyje", "mondo", "monde", "mundo",
    "العالمية", "世界", "세계", "विश्व", "বিশ্ব", "โลก"];

app.get("/sayWorld", (_, res) => {
  const index = Math.floor(Math.random() * worldTranslations.length);
  const world = worldTranslations[index];
  console.log(\`Sending: \${world}\`);
  res.send(world);
});

const port = 8089;
app.listen(port, () => console.log(\`Listening on port \${port}\`));
EOF
```

To run the app, run this command:

```bash
node app.js
```

To run dapr, run this command:

```bash
dapr run --app-id world-service --app-port 8089 -- node app.js
```
