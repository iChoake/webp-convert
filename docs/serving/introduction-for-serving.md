# Introduction to serving converted WebP files with WebPConvert

**NOTE: This document only applies to the upcoming 2.0 version**

The classes for serving first and foremost helps you handle the cached files intelligently (not serving them if they are larger or older than the original). It also provides a convenient way to deal with conversion failures and setting headers.

You can buy in to the whole package by calling the convenience method *WebPConvert::serveConverted*, or you can cherry-pick. Your choice. Lets start with an example of using *WebPConvert::serveConverted*.

In the following example, all available serve options are explicitly set to their default values. Note that besides these, you can also specify options for converting (such as 'quality')

```php
use WebPConvert\WebPConvert;

WebPConvert::serveConverted($source, $destination, [

    // failure handling
    'fail'                 => 'original',     // ('original' | 404' | 'throw' | 'report')
    'fail-when-fail-fails' => '404',          // ('original' | 404' | 'throw' | 'report')

    // options influencing the decision process of what to be served
    'reconvert' => false,       // if true, existing (cached) image will be discarded
    'serve-original' => false,  // if true, the original image will be served rather than the converted
    'show-report' => false,     // if true, a report will be output rather than the raw image

    // headers, in case an image is served
    'cache-control-header' => 'public, max-age=86400'
    'add-vary-accept-header' => true,
    'set-content-type-header' => true,
    'set-last-modified-header' => true,
    'set-cache-control-header' => true,

    // Besides the specific options for serving, you can also use the options for convert()
]);
```

## Failure handling
The *fail* option gives you an easy way to handle errors. Setting it to 'original' tells it to handle errors by serving the original file instead (*$source*). This could be a good choice on production servers. On development servers, 'throw' might be a good option. It simply rethrows the exception that was thrown by *WebPConvert::convert()*. '404' could also be an option, but it has the weakness that it will probably only be discovered by real persons seeing a missing image.

The fail action might fail too. For example, if it is set to 'original' and the failure is that the original file doesn't exist. Or, more delicately, it may have a wrong mime type - our serve method will not let itself be tricked into serving *exe* files as the 'original'. Anyway, you can control what to do when fail fails using the *fail-when-fail-fails* option. If that fails too, a 404 is served. The fun stops there, there is no "fail-when-fail-when-fail-fails" option to customize this.

## Options influencing the decision process
The default process is like this:

1. Is there a file at the destination? If not, trigger conversion
2. Is the destination older than the source? If yes, delete destination and trigger conversion
3. Serve the smallest file (destination or source)

You can influence the process with the following options:

*reconvert*
If you set *reconvert* to true, the destination and conversion is triggered (between step 1 and 2)

*serve-original*
If you set *serve-original* to true, process will take its cause from (1) to (2) and then end with source being served.

*show-report*
If you set *show-report*, the process is skipped entirely, and instead a report is generated of how a fresh conversion using the supplied options goes.