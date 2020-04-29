<!--- when a new release happens, the VERSION and URL in the badge have to be manually updated because it's a private registry --->
[![npm version](https://img.shields.io/badge/%40nui%2Flibrary-25.0.0-blue.svg)](https://artifactory.corp.adobe.com/artifactory/npm-nui-release/@nui/library/-/@nui/library-25.0.0.tgz)

- [Adobe Asset Compute SDK](#adobe-asset-compute-sdk)
  - [Installation](#installation)
  - [Examples](#examples)
    - [Simple javascript worker](#simple-javascript-worker)
    - [Batch processing javascript worker](#batch-processing-javascript-worker)
    - [ShellScript worker](#shellscript-worker)
  - [API details](#api-details)
    - [Rendition callback function (required)](#rendition-callback-function-required)
      - [Parameters:](#parameters)
      - [**`source`**:](#source)
      - [**`rendition`**:](#rendition)
      - [**`outdir`**:](#outdir)
      - [**`params`**:](#params)
      - [renditionCallback Examples:](#renditioncallback-examples)
      - [Worker Options (optional)](#worker-options-optional)
    - [Contributing](#contributing)
    - [Licensing](#licensing)

# Adobe Asset Compute SDK

Adobe Asset Compute SDK library a shared library used by all Asset Compute workers and takes care of common functions like asset download & rendition upload.

## Installation

```bash
npm install @adobe/asset-compute-sdk
```

## Examples

### Simple javascript worker

Calls rendition function (renditionCallback) for each rendition
```js
const { worker } = require('@adobe/asset-compute-sdk');

async function renditionCallback(source, rendition, params) => {
	// ... worker logic
}
const main = worker(renditionCallback, options);
await main(params);
```
### Batch processing javascript worker

Calls rendition function once with all the renditions
```js
const { batchWorker } = require('@adobe/asset-compute-sdk');

async function batchRenditionCallback(source, rendition, outdir, params) => {
	// ... worker logic
}
const main = batchWorker(batchRenditionCallback, options);
await main(params);
```

### ShellScript worker

Processes renditions using from a worker written in shellscript
```js
const { shellScriptWorker } = require('../lib/api');

const main = shellScriptWorker(); // assumes script is in `worker.sh`
await main(params);
```

Shellscript worker with custom script name
```js
const { shellScriptWorker } = require('../lib/api');

const main = shellScriptWorker('custom-worker-name.sh'); // assumes script is in `worker.sh`
await main(params);
```

## API details

The `worker` and `batchWorker` take two parameters: `renditonCallback` and `options` as described below.

### Rendition callback function (required)
The `renditionCallback` function is where you can put your custom worker logic. For example, if you would like to call an external API, you can make fetch requests to that API inside your `renditionCallback` function.

#### Parameters:
The parameters for the rendition callback function are: `source`, `rendition`, `outdir`, and `params`
#### **`source`**:
source Object containing the following attributes:

| Name | Type | Description | Example |
|------|------|-------------|---------|
| `url` | `string` | URL pointing to the source binary. | `"http://example.com/image.jpg"` |
| `path`| `string` |  Path to local copy of source file | `"tmp/image.jpg"` |
| `name` | `string` | File name. File extension in the name might be used if no mime type can be detected. Takes precedence over filename in URL path or filename in content-disposition header of the binary resource. Defaults to "file". | `"image.jpg"` |
#### **`rendition`**: 
rendition Object containing the following attributes:

| Name | Type | Description |
|------|------|-------------|
| `instructions` | `object` | rendition parameters from the worker params (e.g. quality, dpi, format, hei etc. See full list [here](https://git.corp.adobe.com/nui/nui/blob/master/doc/api.md#rendition-instructions) |
| `directory` | `string` | directory to put the renditions |
| `name` | `string` | filename of the rendition to create |
| `path` | `string` | path to store rendition locally (must put rendition here in order to be uploaded to cloud storage) |
| `index` | `number` | number used to identify a rendition |
| `target` | `string` or `object` | URL to which the generated rendition should be uploaded or multipart pre-signed URL upload information for the generated rendition |
| `metadata` | `object` | stores rendition metadata |
| `size` | `function` | provides the rendition size (does not take any parameters) |
| `sha1` | `function` | provides the rendition sha1 (does not take any parameters) |
| `id` | `function` | provides the an id used to identify a rendition (does not take any parameters) |

#### **`outdir`**:
(only in batchWorker): directory to put renditions produced in batch workers

#### **`params`**:
original parameters passed into the worker (see full [Asset Compute prcoessing API Doc](https://git.corp.adobe.com/nui/nui/blob/master/doc/api.md#asset-processing))

#### renditionCallback Examples:

At the bare minimum, the rendition callback function must write something to the `rendition.path`.

Simplest example (copying the source file):
```js
async function renditionCallback(source, rendition) => {
    // Check for unsupported file
    const stats = await fs.stat(source.path);
    if (stats.size === 0) {
        throw new SourceUnsupportedError('source file is unsupported');
    }
    // process infile and write to outfile
    await fs.copyFile(source.path, rendition.path);
}
```

#### Worker Options (optional)
Optional parameters to pass into workers
- disableSourceDownload: Boolean used to disable the source download (defaults to false)
- disableRenditionUpload: Boolean used to disable the rendition upload (defaults to false)

Disable source download example:
```js
const { worker } = require('@adobe/asset-compute-sdk');

async function renditionCallback(source, rendition, params) => {
	// downloads source inside renditionCallback so does not need asset-compute-sdk to download source file
	await fetch(source.url);
}
const options = {
	disableSourceDownload: true
};
const main = worker(renditionCallback, options);
await main(params);
```

Disable rendition upload example:
```js
const { worker } = require('@adobe/asset-compute-sdk');
const options = {
	disableRenditionUpload: true
};
const main = worker(renditionCallback, options);
await main(params);
```

### Contributing
Contributions are welcomed! Read the [Contributing Guide](./.github/CONTRIBUTING.md) for more information.

### Licensing
This project is licensed under the Apache V2 License. See [LICENSE](LICENSE) for more information.
