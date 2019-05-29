<p align="center">
  <img src="https://user-images.githubusercontent.com/2793951/54391096-418f4500-46a4-11e9-8d0c-b00ff7ba4198.png" alt="flydrive">
</p>

<p align="center">
  <a href="https://www.npmjs.com/package/@targos/flydrive"><img src="https://img.shields.io/npm/dm/@targos/flydrive.svg?style=flat-square" alt="Download"></a>
  <a href="https://www.npmjs.com/package/@targos/flydrive"><img src="https://img.shields.io/npm/v/@targos/flydrive.svg?style=flat-square" alt="Version"></a>
  <a href="https://opensource.org/licenses/MIT"><img src="https://img.shields.io/npm/l/@targos/flydrive.svg?style=flat-square" alt="License"></a>
</p>

`flydrive` is a framework-agnostic package which provides a powerful wrapper to manage Storage in [Node.js](https://nodejs.org).

There are currently 4 drivers available:

- Local
- Amazon S3 (You need to install `aws-sdk` package to be able to use this driver)
- Digital Ocean Spaces (You need to install `aws-sdk` package to be able to use this driver)
- Google Cloud Storage (You need to install `@google-cloud/storage` package to be able to use this driver)

---

## Getting Started

This package is available in the npm registry.
It can easily be installed with `npm` or `yarn`.

```bash
$ npm i @targos/flydrive
# or
$ yarn add @targos/flydrive
```

When you require the package in your file, it will give you access to the `StorageManager` class.
This class is a facade for the package and should be instantiated with a [configuration object](https://github.com/Slynova-Org/flydrive/blob/master/tests/stubs/config.ts).

```javascript
const { StorageManager } = require('@targos/flydrive')
const storage = new StorageManager(config)
```

Once you instantiated the manager, you can use the `StorageManager#disk()` method to retrieve a disk an use it.

```javascript
storage.disk() // Returns the default disk (specified in the config)
storage.disk('awsCloud') // Returns the driver for the disk "s3"
storage.disk('awsCloud', customConfig) // Overwrite the default configuration of the disk
```

## Driver's API

Each driver extends the abstract class [`Storage`](https://github.com/Slynova-Org/flydrive/blob/master/src/Storage.ts). This class will throw an exception for each methods by default. The driver needs to overwrite those methods when he supports them.

The following method doesn't exists on the `LocalFileSystem` driver, therefore, it will throw an exception.

```javascript
// throws "E_METHOD_NOT_SUPPORTED: Method getSignedUrl is not supported for the driver LocalFileSystem"
storage.disk('local').getSignedUrl()
```

Since we are using TypeScript, you can make use of casting to get the real interface:

```typescript
import { LocalFileSystem } from '@targos/flydrive'

storage.disk<LocalFileSystem>('local')
```

### Methods

<details>
<summary markdown="span"><code>append(location: string, content: Buffer | Stream | string, options: object): Promise&lt;boolean&gt;</code></summary>

This method will append the content to the file at the location.

```javascript
// Supported drivers: "local"

await storage.disk('local').append('./foo.txt', 'bar')
// foo.txt now has the content `{initialContent}bar`
```

</details>

<details>
<summary markdown="span"><code>bucket(name: string): void</code></summary>

This method let us swap the used bucker at the runtime.

```javascript
// Supported drivers: "s3", "gcs"

storage.disk('cloud').bucket('anotherOne')
// The following chained action will use the "anotherOne" bucket instead of the default one
```

</details>

<details>
<summary markdown="span"><code>delete(location: string): Promise&lt;boolean&gt;</code></summary>

This method will delete the file at the given location

```javascript
// Supported drivers: "local", "s3", "gcs"

await storage.disk('local').delete('./foo.txt')
// foo.txt has been deleted
```

</details>

<details>
<summary markdown="span"><code>driver(): any</code></summary>

This method returns the driver used if you need to do anything specific not supported by default.

```javascript
storage.disk('local').driver() // Returns "fs-extra"
storage.disk('awsCloud').driver() // Returns "aws-sdk"
storage.disk('googleCloud').driver() // Returns "@google-cloud/storage"
// ....
```

</details>

<details>
<summary markdown="span"><code>exists(location: string): Promise&lt;boolean&gt;</code></summary>

This method will determine if a file or folder exists for the given location.

```javascript
// Supported drivers: "local", "s3", "gcs"

await storage.disk('local').exists('./foo.txt')
```

</details>

<details>
<summary markdown="span"><code>get(location: string, encoding?: object | string): Promise&lt;Buffer | string&gt;</code></summary>

This methods will return the file's content for the given location.

```javascript
// Supported drivers: "local", "s3", "gcs"

const content = await storage.disk('local').exists('./foo.txt')
```

</details>

<details>
<summary markdown="span"><code>getSignedUrl(location: string, expiry: number = 900): Promise&lt;string&gt;</code></summary>

This methods will return the signed url for an existing file.

```javascript
// Supported drivers: "s3", "gcs"

const uri = await storage.disk('awsCloud').getSignedUrl('./foo.txt')
```

</details>

<details>
<summary markdown="span"><code>getSize(location: string): Promise&lt;number&gt;</code></summary>

This methods will return the file size in bytes.

```javascript
// Supported drivers: "local", "gcs"

const bytes = await storage.disk('local').getSize('./foo.txt.)
```

</details>

<details>
<summary markdown="span"><code>getStream(location: string, options: object | string): Stream</code></summary>

This methods will return a stream for the given file.

```javascript
// Supported drivers: "local", "s3", "gcs"

const stream = storage.disk('local').getStream('./foo.txt')
```

</details>

<details>
<summary markdown="span"><code>getUrl(location: string): string</code></summary>

This methods will return an url for a given file.

```javascript
// Supported drivers: "s3", "gcs"

const uri = storage.disk('awsCloud').getUrl('./foo.txt')
```

</details>

<details>
<summary markdown="span"><code>move(src: string, dest: string): Promise&lt;boolean&gt;</code></summary>

This methods will move file to a new location.

```javascript
// Supported drivers: "local", "s3", "gcs"

await storage.disk('local').move('./foo.txt', './newFolder/foo.txt')
```

</details>

<details>
<summary markdown="span"><code>put(location: string, content: Buffer | Stream | string, options: object): Promise&lt;boolean&gt;</code></summary>

This methods will create a new file.

```javascript
// Supported drivers: "local", "s3", "gcs"

await storage.disk('local').put('./bar.txt', 'Foobar')
```

</details>

<details>
<summary markdown="span"><code>prepend(location: string, content: Buffer | string, options: object): Promise&lt;boolean&gt;</code></summary>

This methods will preprend content to a file.

```javascript
// Supported drivers: "local"

await storage.disk('local').prepend('./foo.txt', 'bar')
// foo.txt now has the content `bar{initialContent}`
```

</details>

## Contribution Guidelines

Any pull requests or discussions are welcome.
Note that every pull request providing new feature or correcting a bug should be created with appropriate unit tests.
