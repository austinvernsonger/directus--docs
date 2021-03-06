# Extending Directus

### Custom User-Inputs (UIs)
Similar to extensions, if you need an input more specific to your application than provided by core Directus, you can duplicate, tweak, or create your own. Need a custom room-mapping interface or a proprietary client input? Just create a quick custom UI and it will immediately be available for use.
**[todo]**

### Custom Extensions
By design, Directus keeps things simple. Only key features utilized by more than 80% of users make it into the core software. Anything not covered under the base suite is possible within extensions. Extensions take advantage of the system’s authentication, ACL, and data connections but force absolutely no restrictions. These sandboxed areas can be used to create custom interfaces, data-visualizations, interaction points, dashboards, point-of-sales, or anything else required by your project.
**[todo]**

### Custom API Endpoints
If you’re using Directus' API endpoints (possibly feeding a mobile app or providing a SASS interface) you can always write custom endpoints to compliment the core data/schema/user access.
**[todo]**

### Custom File Storage Adapters

Beyond local storage, Directus comes with the ability to connect to a number of popular file storage systems such as CDNs. As with most things within Directus, if you need something more specific the tools exist to easily implement a custom adapter. Storage Adapters enable a Directus instance to configure the destination for writing and reading the media which is loaded into the application. Within the database, they are defined by the `directus_storage_adapters` table. (Currently there is not an application-level interface to view or edit these.)

The PHP namespace for this logic is `\Directus\Media\Storage`. As of writing, three adapters exist:

* `AmazonS3Adaper` - Maps to Amazon S3 CDN Buckets
* `FileSystemAdapter` - Maps to local filesystem of the application server which hosts the Directus instance
* `RackspaceOpenCloudAdapter` - Maps to Rackspace OpenCloud CDN Containers

All adapters have these configurable parameters - each of which corresponds to a column on the `directus_storage_adapters` table:

`key` - A unique name for the storage adapter.
adapter_name” - The storage driver which this adapter should use (e.g. FileSystemAdapter, AmazonS3Adapter, etc).
`role` - Either “DEFAULT”, “THUMBNAIL”, or null. “DEFAULT” and “THUMBNAIL” should only occur once.
`public` - Either 1 or 0 (yes or no): should the contents of this storage adapter be accessible to non-Directus users?
`destination` - This value varies depending on the storage adapter
`url` - URL which maps to the contents of the storage adapter (trailing slash is required).
`params` - Optional JSON encoded set of additional parameters to be passed to the adapter.

#### Roles

When Directus accepts a file upload, it will send the unaltered file to the storage adapter with the role “DEFAULT”. It will send the rendered thumbnail to the storage adapter with the role “THUMBNAIL”. These should only occur once within the table.

_As of writing, no other roles are in use._

#### Adapter configuration

The significance and values of the destination and params options vary from adapter to adapter.

* FileSystemAdapter
  * `destination` - Absolute path on filesystem, no trailing slash
  * `params` - None
* AmazonS3Adapter (optional, must be included with composer)
  * `destination` - Name of S3 Bucket
  * `params` (required) - `api_key`, `api_secret`
* RackspaceOpenCloudAdapter (optional, must be included with composer)
  * `destination` - Name of OpenCloud Container
  * `params` (required) - `api_user`, `api_key`, `region`, `endpoint`

#### Code samples

When building out Directus core functionality and derivative client functionality, it may be necessary to invoke the storage adapter component. Listed below are some references to code samples which demonstrate various ways of deploying storage adapters.

These links point directly to the commit hash which is current as of this writing, so that line numbers do not move out of sync with the documentation. Remember to compare these code snippets with the current state of Directus code in the future, and to update these docs if necessary.

`/directus/api/api.php`

The [definition of the upload route](https://www.google.com/url?q=https%3A%2F%2Fgithub.com%2FRNGR%2Fdirectus6%2Fblob%2Ff386da45a4957f776c4a701fdd31aae2c93e1273%2Fapi%2Fapi.php%23L781&sa=D&sntz=1&usg=AFQjCNF61vBWbi9eTJc1FvUaSCXAnXM_uQ) shows the most generic (core) way of implementing the storage adapter interface.  We instantiate a new `\Directus\Media\Storage\Storage` object and then simply call the `acceptFile` method with the incoming local file. The interface automatically fetches the default/thumbnail adapters and passes this information to them.

`/directus/media_auth_proxy/index.php`

The authenticated media proxy front controller uses a [slightly more complex implementation](https://www.google.com/url?q=https%3A%2F%2Fgithub.com%2FRNGR%2Fdirectus6%2Fblob%2Ff386da45a4957f776c4a701fdd31aae2c93e1273%2Fmedia_auth_proxy%2Findex.php%23L120&sa=D&sntz=1&usg=AFQjCNFvgWXCSmwfnxuTzYVgymy3TNEujg). After identifying the Directus Media record being requested, it fetches the record for the corresponding storage adapter. It then passes this record to the method `\Directus\Media\Storage\Storage::getStorage`, which returns the corresponding adapter object, allowing the front controller to fetch the file contents abstractly, irrespective of the underlying driver / file location.

`/directus/bin/generateMissingThumbnails.php`

[This implementation](https://www.google.com/url?q=https%3A%2F%2Fgithub.com%2FRNGR%2Fdirectus6%2Fblob%2Ff386da45a4957f776c4a701fdd31aae2c93e1273%2Fbin%2FgenerateMissingThumbnails.php&sa=D&sntz=1&usg=AFQjCNFJOUy3bKpTj0W4XtYE5nwsP_9ZUg) uses the same approach as the authenticated media front controller, however it performs more complex operations, such as checking if a file exists, and writing a file to the adapter.

### Hooks

Hooks allow project-specific (non-Directus core) code to interact directly with Directus core events. The two hooks which are currently supported are `postUpdate` and `postInsert`, which enable logic to react to Directus record update and insert events, respectively. The hooks are very simple to implement: just add custom logic to the (untracked) config file at  /directus/api/configuration.php. [See this example configuration file as a reference.](https://github.com/RNGR/directus6/blob/f386da45a4957f776c4a701fdd31aae2c93e1273/api/configuration.example.php#L31)
