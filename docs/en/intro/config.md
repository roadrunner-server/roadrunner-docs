# Configuration reference

RoadRunner main configuration is written in YAML. However, if you prefer JSON format, you may easily convert your config into the JSON format. 
RR accepts both formats.
Here is a link to the most recent configuration reference:

- [**link**](https://github.com/roadrunner-server/roadrunner/blob/master/.rr.yaml)

## Configuration plugin

- [**link**](../plugins/config.md)

## Limitations

Note that since we use dots as level separators, e.g.: `http.pool`, you can't use dots in section names, queue names, etc. [link](https://github.com/roadrunner-server/roadrunner/issues/1529)
