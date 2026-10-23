# ForgeBox API compatibility notes

This document lists the ForgeBox REST API surface that the CommandBox CLI calls, so the registry can be a drop-in endpoint. It is a starting point for the [compatibility spec issue](https://github.com/modules4cf/registry/issues/1), not a final spec.

**Sources** (CommandBox source pinned to commit [`4471d2b`](https://github.com/Ortus-Solutions/commandbox/tree/4471d2b8d4aa3fd76c36ecb8d001a1df47392ee4), 30 Sep 2026):

* API client: [`util/ForgeBox.cfc`](https://github.com/Ortus-Solutions/commandbox/blob/4471d2b8d4aa3fd76c36ecb8d001a1df47392ee4/src/cfml/system/util/ForgeBox.cfc)
* Endpoint logic (install, publish, version resolution): [`endpoints/ForgeBox.cfc`](https://github.com/Ortus-Solutions/commandbox/blob/4471d2b8d4aa3fd76c36ecb8d001a1df47392ee4/src/cfml/system/endpoints/ForgeBox.cfc)
* Custom endpoint registration: [`services/EndpointService.cfc`](https://github.com/Ortus-Solutions/commandbox/blob/4471d2b8d4aa3fd76c36ecb8d001a1df47392ee4/src/cfml/system/services/EndpointService.cfc), [`forgebox endpoint register`](https://github.com/Ortus-Solutions/commandbox/blob/4471d2b8d4aa3fd76c36ecb8d001a1df47392ee4/src/cfml/system/modules_app/forgebox-commands/commands/forgebox/endpoint/register.cfc), [`forgebox endpoint set-default`](https://github.com/Ortus-Solutions/commandbox/blob/4471d2b8d4aa3fd76c36ecb8d001a1df47392ee4/src/cfml/system/modules_app/forgebox-commands/commands/forgebox/endpoint/set-default.cfc)
* Docs: [ForgeBox Enterprise commands](https://commandbox.ortusbooks.com/forgebox-enterprise/commands), [Endpoint settings](https://commandbox.ortusbooks.com/config-settings/endpoint-settings)
* Observed public responses from `https://www.forgebox.io/api/v1` (read-only GET requests).

## How CommandBox talks to a custom endpoint

* `forgebox endpoint register <name> <APIURL>` stores `endpoints.forgebox-<name>.APIURL` in the CommandBox config. Use `--force` to change the URL of an existing endpoint and keep its tokens.
* On startup, every `forgebox-*` entry becomes an endpoint that reuses the built-in ForgeBox client with that API URL. The endpoint is reachable under the prefix `<name>`, for example `box install <name>:<slug>@<version>`.
* The web base URL is derived from the API URL by removing `/api/...`. After a publish, CommandBox prints `<base>/view/<slug>`, so the web UI should serve package pages at `/view/<slug>`.
* `forgebox endpoint set-default <name>` stores `endpoints.defaultForgeBoxEndpoint`. Plain slugs like `box install foo` then go to that endpoint.
* API tokens are stored per endpoint in `endpoints.forgebox-<name>.APIToken` (and `.tokens.<username>`). They are sent in the `x-api-token` HTTP header on every request, empty for anonymous requests.

## Response envelope

Every response is JSON in this shape. CommandBox fails with "returned something other than JSON" for anything else.

```json
{ "error": false, "messages": [], "data": { }, "pagination": { } }
```

On errors, `error` is `true` and `messages` holds human-readable strings. For `GET storage/{slug}/{version}`, CommandBox also reads the **HTTP status code**: during `publish`, a **404** means "no artifact uploaded yet", and any other error aborts the publish.

## Endpoints called by CommandBox

Paths are relative to the API URL, for example `https://modules4cf.org/api/v1`.

| Method | Path | Used by | Notes |
|---|---|---|---|
| GET | `entry/{slug}` | install, update, outdated, show | Entry with `versions` array. Core of installing. |
| GET | `entries` | search, show (lists) | Query: `orderBy` (`popular`, `new`, `recent`, `installs`), `max`, `offset`, `typeSlug`, `searchTerm`. `data` = `{ results, count, totalRecords, offset }`. |
| GET | `slugs` | tab completion | Query: `searchTerm`, `typeSlug`. `data` = array of slug strings. |
| GET | `types` | show, types | Array of package types (`typeSlug`, `typeName`, ...). |
| GET | `slug-check/{slug}` | slugcheck | `data` = boolean, true if the slug is available. |
| POST | `install/{slug}[/{version}]` | install | Install counter. Called in the background, failures are ignored. |
| GET | `storage/{slug}/{version}` | install, publish | Returns the download URL of an uploaded artifact. 404 if none. |
| GET | `storage/storeURL/{slug}/{version}` | publish | Returns a URL that the CLI then **PUTs** the zip to (`Content-Type: application/zip`, expects HTTP 200). On ForgeBox this is a pre-signed storage URL. |
| POST | `publish` | publish | Form fields: `slug`, `private`, `version`, `boxJSON` (serialized box.json), `isStable`, `description`, `descriptionFormat`, `installInstructions`, `installInstructionsFormat`, `changeLog`, `changeLogFormat`, `forceUpload`, `binaryHash` (MD5 of the zip). Optional multipart `zip` file. |
| POST | `unpublish/{slug}[/{version}]` | unpublish | Removes one version or the whole package. |
| POST | `authenticate` | login | Params: `username`, `password`. Returns the user including the API token. |
| POST | `register` | register | Params: `username`, `password`, `email`, `fName`, `lName`. |
| GET | `users/whoami/{APIToken}` | whoami | Note: the token is part of the path, so it must never be logged. |
| GET/POST | `users/{username}/clisettings` | config sync | Optional feature, stores CLI settings (form field `CLISettings`). |

## Fields CommandBox reads

**Entry** (`GET entry/{slug}`): `slug`, `title`, `summary`, `description`, `descriptionFormat`, `typeName`, `typeSlug`, `isActive` (install refuses inactive entries), `isListed`, `isPrivate`, `createdDate`, `updatedDate`, `downloads`, `installs`, `hits`, `avgRating`, `homeURL`, `sourceURL`, `bugsURL`, `documentationURL`, `user` (`username`, `fname`, `lname`, ...) and `versions`.

**Version** (items of `versions`): `version` (semver), `downloadURL`, `binaryHash`. ForgeBox also returns `installs` and `sortOrder`.

* `downloadURL` may be any URL that CommandBox can resolve (HTTP(S), git, ...), or the special value `forgeboxStorage`. In that case CommandBox calls `GET storage/{slug}/{version}` to get the real URL.
* If `binaryHash` is set and the download is HTTP(S), CommandBox verifies the downloaded zip against it.
* Version ranges, `stable` and `be` are resolved on the client with the CommandBox semver library, so the server only needs to return all versions.

## Open questions

* Is there an official, current ForgeBox API reference? We found none. The [ForgeBox SDK](https://github.com/coldbox-modules/cbox-forgeboxsdk) is the only other public client we know of.
* Which fields are only decorative (ratings, hits) and can be returned as defaults?
* How are private packages (`@user/slug` style slugs, `isPrivate`) expected to behave?
* Which status codes and messages does CommandBox depend on beyond the 404 on `storage`?
