# Hello World Demo

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

To test the app, run this command in a separate terminal window:

```bash
curl http://localhost:8088/sayhello
```

Exit the node app and run it in dapr by using this command:

```bash
dapr run --app-id hello-service --app-port 8088 -- node app.js
```

To test the app, run this command in a separate terminal window and view dapr logs:

```bash
curl http://localhost:8088/sayhello
```

## world-service

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

To test the app, run this command in a separate terminal window:

```bash
curl http://localhost:8089/sayworld
```

Exit the node app and run it in dapr by using this command:

```bash
dapr run --app-id world-service --app-port 8089 -- node app.js
```

To test the app, run this command in a separate terminal window and view dapr logs:

```bash
curl http://localhost:8089/sayworld
```

## greeting-service

This service uses Dapr to invoke both the `hello-service` and `world-service` services and concatenates the results of each in its resposne.

To create the new app:

```bash
mkdir greeting-service
cd greeting-service
npm init
npm install express node-fetch@2
cat << EOF > app.js
const express = require("express");
const fetch = require("node-fetch");

const app = express();
const daprPort = process.env.DAPR_HTTP_PORT;
const invokeHello = \`http://localhost:\${daprPort}/v1.0/invoke/hello-service/method/sayHello\`;
const invokeWorld = \`http://localhost:\${daprPort}/v1.0/invoke/world-service/method/sayWorld\`;

app.get("/greet", async (_, res) => {
  hello = await fetch(invokeHello);
  world = await fetch(invokeWorld);
  const greeting = (await hello.text()) + " " + (await world.text());
  console.log(\`Sending: \${greeting}\`);
  res.send(greeting);
});

const port = 8090;
app.listen(port, () => console.log(\`Listening on port \${port}\`));
EOF
```

To run dapr, run this command:

```bash
# this needs a dapr-http-port because the underlying app needs to call into dapr sidecar to invoke the hello and world service
# dapr will randomize the port and injects it into an environment variable but I gues this is just to make it more predictable
dapr run --app-id greeting-service --app-port 8090 --dapr-http-port 3500 -- node app.js
```

To test the app, run this command in a separate terminal window and view dapr logs:

```bash
curl http://localhost:8090/greet
```

Open zipkin and view the distributed trace logs: http://localhost:9411

Run `dapr dashboard` to open the dapr dashboard

## Dapr on Kubernetes

Before you run on Kubernetes, you need to build the docker containers and publish to a docker registry. This repository is wired up to [Build and publish Docker Images to GitHub Container Registry](https://github.com/marketplace/actions/build-and-publish-docker-images-to-github-container-registry)

To build the containers locally:

```bash
cd ../hello-service
docker build -t hello-service:v0.1.0 .

cd ../world-service
docker build -t world-service:v0.1.0 .

cd ../greeting-service
docker build -t greeting-service:v0.1.0 .
```

To run Dapr on Kubernetes, run either of these commands:

```bash
# install using dapr cli
dapr init --kubernetes
dapr status -k

# optionall install using Helm
helm repo add dapr https://dapr.github.io/helm-charts
helm repo update
helm upgrade --install dapr dapr/dapr --version=1.7.2 --namespace dapr-system --create-namespace --set global.ha.enabled=true --wait

# upgrade helm package
helm upgrade dapr dapr/dapr --version=1.7.2 --namespace dapr-system --wait

# validate
kubectl get pods -n dapr-system
```

To uninstall, Dapr from Kubernetes, run either of these commands:

```bash
dapr uninstall --kubernetes                 # if you installed using dapr cli
helm uninstall dapr --namespace dapr-system # if you installed using helm
```

### Deploy workload

```bash
# deploy manifests
kubectl apply -f .

# get service info
kubectl get svc greeting-service
NAME               TYPE       CLUSTER-IP    EXTERNAL-IP   PORT(S)        AGE
greeting-service   NodePort   10.43.56.71   <none>        80:32598/TCP   20s

# get the node port and do a curl to it to make sure it is all up and running
curl http://localhost:30312/greet
```

### Inspect logs

```bash
kubectl get po -lapp=greeting
kubectl logs greeting-deploy-65f5b6d9d6-8r8gr -c daprd
```

### Notes

- `node-fetch` version 3 changes: https://github.com/node-fetch/node-fetch/blob/main/docs/v3-UPGRADE-GUIDE.md
