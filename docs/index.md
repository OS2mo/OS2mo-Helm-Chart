<!--
SPDX-FileCopyrightText: 2021 Magenta ApS <https://magenta.dk>
SPDX-License-Identifier: MPL-2.0
-->

# OS2mo Helm Chart

OS2mo Helm Chart is a <a href="https://helm.sh/">Helm</a> chart to deploy 
<a href="https://os2mo.readthedocs.io/">OS2mo</a>.

## Requirements

Helm 3+

## Installation & Usage

Assuming <a href="https://kubernetes.io/docs/tasks/tools/">`kubectl`</a> and
<a href="https://helm.sh/docs/intro/install/#helm">`helm`</a> are already installed
and <a href="https://kubernetes.io/docs/tasks/access-application-cluster/access-cluster/">cluster access is configured</a>:

```console
$ helm install mo os2mo/
$ helm install mo -f kind-override-values.yaml os2mo/
```

Which should bring up an OS2mo stack.

For a more detailed walk-through checkout the ["Getting Started"](GettingStarted.md) section.


## Development
We try to follow [best practices](https://helm.sh/docs/topics/charts/) as much
as possible, including defining structure on the chart values using
[JSON Schema](https://json-schema.org/) in the `values.schema.json` file.

## License

This project is licensed under the terms of the MPL-2.0 license.
