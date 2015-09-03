# mdg

mdg - Mackerel Dashboard Generator

### DESCRIPTION

`mdg` is a small script to help generate markdown for custom dashboard.
It generate markdown which contains embedded graphs of specified service and role.

It's written in Ruby.

### USAGE

```
$ mdg -o <orgName> -s <serviceName> -r <roleName> [ -f <config file(yaml)> ]
```

#### Example

```
$ mdg -o SampleOrg -s SampleService -r backend
```

```md
### loadavg5
|2d|3w|3mo|
|:--|:--|:--|
|<iframe src='https://mackerel.io/embed/orgs/SampleOrg/services/SampleService/backend?graph=loadavg5&amp;period=2d' height='200' width='400' frameborder='0'></iframe>|<iframe src='https://mackerel.io/embed/orgs/SampleOrg/services/SampleService/backend?graph=loadavg5&amp;period=3w' height='200' width='400' frameborder='0'></iframe>|<iframe src='https://mackerel.io/embed/orgs/SampleOrg/services/SampleService/backend?graph=loadavg5&amp;period=3mo' height='200' width='400' frameborder='0'></iframe>|

### cpu.{user,iowait,system}
|2d|3w|3mo|
|:--|:--|:--|
|<iframe src='https://mackerel.io/embed/orgs/SampleOrg/services/SampleService/backend?graph=cpu.%7Buser%2Ciowait%2Csystem%7D&amp;period=2d' height='200' width='400' frameborder='0'></iframe>|<iframe src='https://mackerel.io/embed/orgs/SampleOrg/services/SampleService/backend?graph=cpu.%7Buser%2Ciowait%2Csystem%7D&amp;period=3w' height='200' width='400' frameborder='0'></iframe>|<iframe src='https://mackerel.io/embed/orgs/SampleOrg/services/SampleService/backend?graph=cpu.%7Buser%2Ciowait%2Csystem%7D&amp;period=3mo' height='200' width='400' frameborder='0'></iframe>|

...

```

### CONFIGURATION

#### config.yaml

```yaml
periods: # Default: ["2d", "3w", "3mo"]
  - 1d
  - 1w
  - 1mo
graph_names: # Default: all default graphs
  - loadavg5
  - cpu.user
graph_size: # Default: height:200, width:400
  height: 200
  width: 600
```

|name       |description                                |
|--         |--                                         |
|periods    |List of period of graph                    |
|graph_names|Names of the graphs that will be generated |
|graph_size |Size of the graph                          |

Period uses 6 types of time formats as below and it can be combined with each other.

- 1m (1minute)
- 1h (1hour)
- 1d (1day)
- 1w (1week)
- 1mo (1month)
- 1y (1year)

For example, `10w6h30m` means `10 weeks and 6 hours and 30 minutes`

### Sample shellscript to use `mdg` easily assisted by [peco](https://github.com/peco/peco)

#### mdg_peco

```sh
#!/usr/bin/env sh
# DESCRIPTION:
#  Sample file for using mdg with peco
# DEPENDENCIES:
# - curl http://curl.haxx.se/
# - jq   https://github.com/stedolan/jq
# - peco https://github.com/peco/peco

if [ -z "$MACKEREL_API_BASE" ]; then
  MACKEREL_API_BASE="https://mackerel.io/api/v0"
fi
if [ -z "$MACKEREL_APIKEY" ]; then
  MACKEREL_APIKEY="" # Write API key
fi
if [ -z "$MACKEREL_ORG_NAME" ]; then
  MACKEREL_ORG_NAME="" # Write organization name
fi

function mackerel_api() {
  if [ -n "$1" ]; then
    curl -s -H "X-Api-Key: $1" "$MACKEREL_API_BASE$2"
  else
    echo 'API key is empty' 1>&2
    return 1
  fi
}

function mackerel_services() {
  mackerel_api "$MACKEREL_APIKEY" "/services" | jq -a -M -r '.services[].name'
}

function mackerel_roles() {
  mackerel_api "$MACKEREL_APIKEY" "/services/$1/roles" | jq -a -M -r '.roles[].name'
}

if [ -z "$MACKEREL_APIKEY" ]; then
  echo 'API key is empty.' 1>&2
  exit 1
fi

service=$(mackerel_services | peco)
role=$(mackerel_roles $service | peco)

if [ $# -gt 0 ] ; then
  config_file=$1
  mdg -o ${MACKEREL_ORG_NAME} -s ${service} -r ${role} -f $config_file
else
  mdg -o ${MACKEREL_ORG_NAME} -s ${service} -r ${role}
fi
```

Please export `MACKEREL_APIKEY` and `MACKEREL_ORG_NAME`, or write them directly in above script.
You can use api key which don't have write permission.

```
$ export MACKEREL_APIKEY="API KEY"
$ export MACKEREL_ORG_NAME="YOUR ORGANIZATION NAME"
```

#### Usage

You can specify service and role with [peco](https://github.com/peco/peco) by executing this script

```
$ sh /path/to/mdg_peco
# or you can pass config file
$ sh /path/to/mdg_peco /path/to/config.yaml
```

License
-------

Copyright 2015 Hatena Co., Ltd.

Licensed under the Apache License, Version 2.0 (the "License"); you may not use this file except in compliance with the License. You may obtain a copy of the License at

    http://www.apache.org/licenses/LICENSE-2.0

Unless required by applicable law or agreed to in writing, software distributed under the License is distributed on an "AS IS" BASIS, WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied. See the License for the specific language governing permissions and limitations under the License.
