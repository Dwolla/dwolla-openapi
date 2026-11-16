# dwolla-openapi

OpenAPI spec for the Dwolla API. It's the source for the public docs and the generated SDKs, including dwolla-php (built with Speakeasy).

## Layout and workflow

- Edit the source files under `specs/`. The entry point is `specs/spec.yml`. Operations live in `specs/resources/<area>/` and schemas in `specs/schemas/<area>/`.
- `openapi.yml` is a generated bundle. Never edit it by hand. Regenerate it after every change to `specs/` and commit it with the change. (CI also re-bundles and commits it.)
- Files that cover several operations on one path are named after those operations, for example `retrieveUpdateAlert.yml`, `retrieveCancelTransfer.yml`, `createListCustomers.yml`. If you add an operation to a path, rename the file and update `specs/spec.yml`.

```bash
npx @redocly/cli bundle specs/spec.yml -o openapi.yml   # bundle
npx @redocly/cli lint openapi.yml                       # must be valid
speakeasy lint openapi -s openapi.yml                   # must have 0 errors
npx mint openapi-check openapi.yml                      # Mintlify check
```

These are the same checks as `.github/workflows/bundle-validate.yml`. Compare warning counts with `main`. New warnings are fine when you can explain them.

## Error schemas

**Never put `enum` or `const` on an error's top-level `code`.** Declare it as `type: string` with an `example`:

```yaml
code:
  type: string
  example: ValidationError
```

Why: Speakeasy's PHP generator casts the error `code` to `int` when it builds the exception. If `code` is an enum, PHPStan fails and dwolla-php regeneration breaks (see #59). Every error schema in the spec follows this rule.

You **can** use enums on the embedded error codes (`_embedded.errors[].code`) to tell `oneOf` branches apart, for example `Invalid` and `NotAllowed` in `UpdateAlertStatusInvalidError` and `UpdateAlertStatusNotAllowedError`. The cast doesn't touch those. List every embedded code the API can return for that path. If the API returns a code you didn't list, typed SDKs can't match it to any branch.

Redocly reports `no-illogical-composition-keywords` on these `oneOf`s, because the branches share the same top-level `code`. That warning is expected (`CancelTransfer` gets it too).

## Request bodies and code samples

- Document request bodies as `application/json`. Most operations do this, including `CancelTransfer` and `UpdateAlert`.
- In new or edited `x-codeSamples`, the bash sample's `Content-Type` must match the documented request body content type. `Accept` stays `application/vnd.dwolla.v1.hal+json`.
- Examples must validate against their schema. With a `oneOf`, each example must match exactly one branch.
