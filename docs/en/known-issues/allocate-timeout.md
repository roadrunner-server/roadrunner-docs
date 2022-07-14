# Allocate timeout error

[TMP] PLACEHOLDER

How to fix that?  

1. Check the `pool.allocate_timeout` option. It should be in form of `pool.allocate_timeout: 1s` or `pool.allocate_timeout: 1h`, so the units of measurement should be specified.
2. xdebug is known to have a problems with wrong configuration. See this tutorial: https://roadrunner.dev/docs/php-debugging/2.x/en
