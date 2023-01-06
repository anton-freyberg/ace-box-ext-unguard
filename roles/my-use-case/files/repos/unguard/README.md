# ![Unguard Logo](docs/images/unguard-logo.png) Unguard

___

**Unguard** (🇦🇹 [ˈʊnˌɡuːat] like disquieting, 🇫🇷 [ãˈɡard] like the fencing command) is an **insecure** cloud-native
microservices demo application. It consists of seven app services, a load generator, and two databases. Unguard
encompasses vulnerabilities like SSRF, Command/SQL injection and comes with built-in Jaeger traces. The application is
a web-based Twitter clone where users can:

* register/login
* post text, URLs and images
* view global or personalized timelines
* see ads on the timeline
* view/follow users
* edit your user biography

## 🖼️ Screenshots

| Timeline                                                                                                | User profile                                                                                                      |
|---------------------------------------------------------------------------------------------------------|-------------------------------------------------------------------------------------------------------------------|
| [![Screenshot of the timeline](./docs/images/unguard-timeline.png)](./docs/images/unguard-timeline.png) | [![Screenshot of a user profile](./docs/images/unguard-user-profile.png)](./docs/images/unguard-user-profile.png) |

## 🏗️ Architecture

Unguard is composed of seven microservices written in different languages that talk to each other over REST.

![Unguard Architecture](docs/images/unguard-architecture.png)

| Service                                                    | Language        | Service Account | Description                                                                                                                                 |
|------------------------------------------------------------|-----------------|-----------------|---------------------------------------------------------------------------------------------------------------------------------------------|
| [envoy-proxy](./src/envoy-proxy)                           |                 | default         | Routes to the frontend or the ad-service and also provides a vulnerable health endpoint.                                                    |
| [frontend](./src/frontend)                                 | Node.js Express | default         | Serves HTML to the user to interact with the application.                                                                                   |
| [ad-service](./src/ad-service)                             | .NET 5          | default         | Provide CRUD operation for images and serves a HTML page which displays an image like an ad.                                                |
| [microblog-service](./src/microblog-service)               | Java Spring     | default         | Serves a REST API for the frontend and saves data into redis (explicitly calls vulnerable functions of the jackson-databind library 2.9.9). |
| [proxy-service](./src/proxy-service)                       | Java Spring     | unguard-proxy   | Serves REST API for proxying requests from frontend (vulnerable to SSRF; no sanitization on the entered URL).                               |
| [user-profile-service](./src/profile-service)              | Java Spring     | default         | Serves REST API for updating biography information in a H2 database; vulnerable to SQL injection attacks                                    |
| [user-auth-service](./src/user-auth-service)               | Node.js Express | default         | Serves REST API for authenticating users with JWT tokens (vulnerable to JWT key confusion).                                                 |
| [status-service](./src/malicious-load-generator)           | Go              | default         | Vulnerable server that uses the Kubernetes API from within a pod to get current deployment information                                      |
| jaeger                                                     |                 | default         | The [Jaeger](https://www.jaegertracing.io/) stack for distributed tracing.                                                                  |
| mariadb                                                    |                 | unguard-mariadb | Relational database that holds user and token data.                                                                                         |
| redis                                                      |                 | default         | Key-value store that holds all user data (except authentication-related stuff).                                                             |
| [user-simulator](./src/user-simulator)                     | Node.js Element | default         | Creates synthetic user traffic by simulating an Unguard user using a real browser. Acts as a load generator.                                |
| [malicious-load-generator](./src/malicious-load-generator) |                 | default         | Malicious load generator that makes CMD, JNDI, and SQL injections.                                                                          |

| Service Account | Permissions                               |
|-----------------|-------------------------------------------|
| default         | None                                      |
| unguard-proxy   | List, get & create pods, create pods/exec |

## 🖥️ Local Deployment

See the [Development Guide](./docs/DEV-GUIDE.md) on how to develop Unguard on a local K8S cluster.

## ☁️ Cloud Deployment

See the [Deployment Guide](./docs/DEPLOYMENT.md) on how to deploy Unguard to your cloud. Currently, the documentation
only covers AWS.

## ✨ Features

* **[Kubernetes](https://kubernetes.io/)/[AWS](https://aws.amazon.com/eks)**: The app is designed to run on a local
  Kubernetes cluster, as well as on the cloud with AWS.
* [**Jaeger Tracing**](https://www.jaegertracing.io/): Most services are instrumented using trace interceptors.
* [**Skaffold**](https://skaffold.dev/): Unguard is deployed to Kubernetes with a single command using Skaffold.
* **Synthetic Load Generation**: The application comes with a deployment that creates traffic using
  the [Element](https://element.flood.io/) browser-based load generation library.
* **[Exploits](./exploit-toolkit/exploits/README.md)**: Different automated attack scenarios like JWT key confusion
  attacks or remote code execution.
* **[Monitoring](./docs/MONACO.md)**: Dynatrace monitoring by
  utilizing [MONACO](https://github.com/dynatrace-oss/dynatrace-monitoring-as-code).

## ➕ Additional Deployment Options

* **Falco**: [See these instructions](./docs/FALCO.md)
* **Tracing**: [See these instructions](./docs/TRACING.md)
* **Malicious Load Generator**: [See these instructions](src/malicious-load-generator/README.md)

---

[Hummingbird](https://thenounproject.com/search/?q=hummingbird&i=4138237) icon by Danil Polshin
from [the Noun Project](https://thenounproject.com/).
