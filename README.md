# dwolla-openapi

An OpenAPI specification for the Dwolla API. https://developers.dwolla.com

## Mock Server With Prism

Dwolla uses this OpenAPI spec to spin up a local mock server and test request and responses. Dwolla
uses [Prism](https://stoplight.io/open-source/prism) from [Stoplight](https://stoplight.io/) to spin up the mock server.

Follow these steps to spin up your own Dwolla mock server with Prism:

- [Install Prism](https://meta.stoplight.io/docs/prism/ZG9jOjky-installation) on your local machine
- Clone this repository and `cd` into it from a terminal session
- Run `prism mock specs/spec.yml` in terminal
