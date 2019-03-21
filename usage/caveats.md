# Caveats

## Uploading Files
Since file upload is handled on RR end PHP process will only receive the filename of temporary resources. This resource would not be registered in `uploaded files` hash and, as result, function `is_uploaded_file` will always return `false`.

> Reference: https://github.com/spiral/roadrunner/issues/133