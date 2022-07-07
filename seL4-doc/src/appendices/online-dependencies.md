# Online Dependencies

In order to use the seL4 Developer Kit offline, for example in an enviroment where internet access is restricted or compltelty unavailable which may be the case in a security domain, a number of dependencies must be downloaded. Below is a list of some of the key dependecies that must be downloaded:

1. Docker Enviroment/Image - for building seL4, U-Boot etc. - this should be downloaded from the [package repository of the sel4devkit GitHub organisation](https://github.com/orgs/sel4devkit/packages/container/package/maaxboard).
  - For further guidance see [here](devkit_offline_use.md#downloading-the-docker-image-to-a-file).
2. seL4 and related source code
  - This must be downloaded and structured in a speciifc directory structure that matches the way in which the Repo command line tool would organise the various files and folders required. For more guidance see [here](devkit_offline_use.md#downloading-source-code-for-offline-use).
3. [Prebuilt SD Card images]()

For other guidance in using the developer kit offline, see the ["Using the Developer Kit Offline"](devkit_offline_use.md) appendix.
