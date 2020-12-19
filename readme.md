# webext-additional-permissions [![Travis build status](https://api.travis-ci.com/fregante/webext-additional-permissions.svg?branch=master)](https://travis-ci.com/fregante/webext-additional-permissions) [![npm version](https://img.shields.io/npm/v/webext-additional-permissions.svg)](https://www.npmjs.com/package/webext-additional-permissions)

> WebExtensions module: Get any optional permissions that users have granted you.

`chrome.permissions.getAll()` will report all permissions, whether they're part of the manifest’s `permissions` field or if they've been granted later via `chrome.permissions.request`.

`webext-additional-permissions` will return the same `Permissions` object but it will only include any permissions that the user might have granted to the extension.

Compatible with Chrome 69+ and Firefox 62+ (both released in September 2018.)

Like the regular `chrome.permissions` API, **this module does not work in content scripts.**

## Install

You can just download the [standalone bundle](https://packd.fregante.now.sh/webext-additional-permissions@latest?name=getAdditionalPermissions) (it might take a minute to download) and include the file in your `manifest.json`, or:

```sh
npm install webext-additional-permissions
```

```js
// This module is only offered as a ES Module
import {getAdditionalPermissions, getManifestPermissions} from 'webext-additional-permissions';
```

## Usage

```json
// example manifest.json
{
	"permissions": [
		"https://google.com/*",
		"storage"
	],
	"optional_permissions": [
		"https://*/*"
	]
}
```

Simple example with the above manifest:

```js
(async () => {
	const newPermissions = await getAdditionalPermissions();
	// => {origins: [], permissions: []}

	const manifestPermissions = await getManifestPermissions();
	// => {origins: ['https://google.com/*'], permissions: ['storage']}
})();
```

Example showing how the result changes when you add further permissions (for example via [webext-domain-permission-toggle](https://github.com/fregante/webext-domain-permission-toggle))

```js
async function onGrantPermissionButtonClick() {
	await browser.permissions.request({origins: ['https://facebook.com/*']});

	// Regular `browser` API: returns manifest permissions and new permissions
	const allPermissions = await browser.permissions.getAll();
	// => {origins: ['https://google.com/*', 'https://facebook.com/*'], permissions: ['storage']}

	// This module: only the new permission is returned
	const newPermissions = await getAdditionalPermissions();
	// => {origins: ['https://facebook.com/*'], permissions: []}

	// This module: the manifest permissions are unchanged
	const manifestPermissions = await getManifestPermissions();
	// => {origins: ['https://google.com/*'], permissions: ['storage']}
}
```

## API

### getAdditionalPermissions(options)

Returns a promise that resolves with a `Permission` object like `chrome.permissions.getAll` and `browser.permissions.getAll`, but only includes the optional permissions that the user granted you.

#### options

Type: `object`

##### strictOrigins

Type: `boolean`\
Default: `true`

If manifest contains the permission `https://github.com/*` and then request `*://*.github.com/*` ([like Safari does](https://github.com/fregante/webext-additional-permissions/issues/1)), the latter will be considered an *additional permission* because technically it's broader.

If this distinction doesn't matter for you (for example if the protocol is always `https` and there are no subdomains), you can use `strictOrigins: false`, so that the requested permission will not be reported as *additional*.

### getManifestPermissions()

Returns a promise that resolves with a `Permission` object like `chrome.permissions.getAll` and `browser.permissions.getAll`, but only includes the permissions you declared in `manifest.json`.

**Note:** both this method and the native `chrome.permissions.getAll` will also include any permissions implied by `matches` in `content_scripts`, even if they're not explicitly listed in the `permissions` field.

### getManifestPermissionsSync()

Same as `getManifestPermissions` but it doesn't return a Promise.

**Note:** Only Manifest permissions can be retrived synchronously, so there's no ~~`getAdditionalPermissionsSync`~~.


## Related

### Permissions

* [webext-domain-permission-toggle](https://github.com/fregante/webext-domain-permission-toggle) - Browser-action context menu to request permission for the current tab. Chrome and Firefox.
* [webext-dynamic-content-scripts](https://github.com/fregante/webext-dynamic-content-scripts) - Automatically registers your content_scripts on domains added via permission.request

### Others

* [webext-options-sync](https://github.com/fregante/webext-options-sync) - Helps you manage and autosave your extension's options. Chrome and Firefox.
* [webext-storage-cache](https://github.com/fregante/webext-storage-cache) - Map-like promised cache storage with expiration. Chrome and Firefox
* [webext-detect-page](https://github.com/fregante/webext-detect-page) - Detects where the current browser extension code is being run. Chrome and Firefox.
* [webext-content-script-ping](https://github.com/fregante/webext-content-script-ping) - One-file interface to detect whether your content script have loaded.
* [web-ext-submit](https://github.com/fregante/web-ext-submit) - Wrapper around Mozilla’s web-ext to submit extensions to AMO.
* [Awesome-WebExtensions](https://github.com/fregante/Awesome-WebExtensions) - A curated list of awesome resources for WebExtensions development.

## License

MIT © [Federico Brigante](https://bfred.it)
