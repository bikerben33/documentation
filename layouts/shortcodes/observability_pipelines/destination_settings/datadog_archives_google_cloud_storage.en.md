<div class="alert alert-warning">The Google Cloud Storage destination only supports <a href = "https://cloud.google.com/storage/docs/access-control/lists">Access Control Lists</a>.</div>

1. Enter the name of the Google Cloud storage bucket you created earlier.
1. Enter the path to the credentials JSON file you downloaded [earlier](#create-a-service-account-to-allow-workers-to-write-to-the-bucket).
1. Select the storage class for the created objects.
1. Select the access level of the created objects.
1. Optionally, enter in the prefix.
    - Prefixes are useful for partitioning objects. For example, you can use a prefix as an object key to store objects under a particular directory. If using a prefix for this purpose, it must end in `/` to act as a directory path; a trailing `/` is not automatically added.
    - See [template syntax][10051] if you want to route logs to different object keys based on specific fields in your logs.
1. Optionally, click **Add Header** to add metadata.

[10051]: /observability_pipelines/destinations/#template-syntax