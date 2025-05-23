# OpenTelemetry Esbuild for Node

[![NPM Published Version][npm-img]][npm-url]
[![Apache License][license-image]][license-url]

## About

This module provides a way to auto instrument any Node application to capture telemetry from a number of popular libraries and frameworks, via an [esbuild](https://esbuild.github.io/) plugin.

The net result is the ability to gather telemetry data from a Node application without any code changes to your application.

This module also provides a simple way to manually initialize multiple Node instrumentations for use with the OpenTelemetry SDK.

Compatible with OpenTelemetry JS API and SDK `1.0+`.

## Installation

```bash
npm i -D opentelemetry-esbuild-plugin-node esbuild
```

## Usage: Esbuild plugin

This module includes instrumentation for all supported non-builtin instrumentations.
Please see the [Supported Instrumentations](#supported-instrumentations) section for more information.

Enable auto instrumentation by configuring it in your esbuild script:

```javascript
const { openTelemetryPlugin } = require('opentelemetry-esbuild-plugin-node');
const { build } = require('esbuild');

build({
  entryPoints: ['src/server.ts'], // Path to the entrypoint of your application
  bundle: true,
  outfile: 'dist/server.js',
  target: 'node20',
  platform: 'node',
  sourcemap: true,
  plugins: [openTelemetryPlugin()],
});
```

## Usage: Instrumentation Initialization

This packages for Node automatically loads instrumentations for Node builtin modules and common packages.

Enable instrumentation by configuring it in your esbuild script:

```javascript
const { openTelemetryPlugin } = require('opentelemetry-esbuild-plugin-node');
const { build } = require('esbuild');
const {
  getNodeAutoInstrumentations,
} = require('@opentelemetry/auto-instrumentations-node');

build({
  entryPoints: ['src/server.ts'],
  bundle: true,
  outfile: 'dist/server.js',
  target: 'node20',
  platform: 'node',
  sourcemap: true,
  plugins: [
    openTelemetryPlugin({ instrumentations: getNodeAutoInstrumentations() }),
  ],
});
```

Custom configuration for each of the instrumentations can be passed to the plugin, by providing an object with the name of the instrumentation as a key, and its configuration as the value. Though passing instrumentations in directly is preferred, and the ability to configure via `instrumentationConfig` rather than passing in instrumentations directly will be removed in a future version.

```javascript
const { openTelemetryPlugin } = require('opentelemetry-esbuild-plugin-node');
const { build } = require('esbuild');

build({
  entryPoints: ['src/server.ts'],
  bundle: true,
  outfile: 'dist/server.js',
  target: 'node20',
  platform: 'node',
  sourcemap: true,
  plugins: [
    openTelemetryPlugin({
      instrumentationConfig: {
        '@opentelemetry/instrumentation-aws-sdk': {
          suppressInternalInstrumentation: true,
        },
      },
    }),
  ],
});
```

This esbuild script will instrument non-builtin packages but will not configure the rest of the OpenTelemetry SDK to export traces
from your application. To do that you must also configure the SDK.

The esbuild script currently only patches non-builtin modules (more specifically, modules in [opentelemetry-js-contrib](https://github.com/open-telemetry/opentelemetry-js-contrib)), so this is also the place to configure the instrumentation
for builtins or add any additional instrumentations.

### Gotchas

There are limitations to the configuration options for each package. Most notably, any functions are not allowed to be passed in to plugins.

The reason for this is that the current mechanism of instrumenting packages involves stringifying the instrumentation configs, which does not account for any external scoped dependencies, and thus creates subtle opportunities for bugs.

```javascript
const {
  getNodeAutoInstrumentations,
} = require('@opentelemetry/auto-instrumentations-node');
const {
  AsyncHooksContextManager,
} = require('@opentelemetry/context-async-hooks');
const {
  OTLPTraceExporter,
} = require('@opentelemetry/exporter-trace-otlp-http');
const { NodeSDK } = require('@opentelemetry/sdk-node');

const instrumentations = getNodeAutoInstrumentations();

const sdk = new NodeSDK({
  traceExporter: new OTLPTraceExporter(),
  contextManager: new AsyncHooksContextManager().enable(),
  instrumentations,
});

sdk.start();

process.on('SIGTERM', () => {
  sdk.shutdown().finally(() => process.exit(0));
});
```

## Supported instrumentations

See [@opentelemetry/auto-instrumentations-node](https://github.com/open-telemetry/opentelemetry-js-contrib/tree/main/metapackages/auto-instrumentations-node) for the supported packages.

Note that Node.js builtin modules will not be patched by this plugin, but initializing the `NodeSDK` with plugins relevant to those modules (eg [@opentelemetry/instrumentation-undici](https://www.npmjs.com/package/@opentelemetry/instrumentation-undici)) will instrument them properly as the existing `require`-patching approach still works with built in modules.

## Useful links

- For more information on OpenTelemetry, visit: <https://opentelemetry.io/>
- For more about OpenTelemetry JavaScript: <https://github.com/open-telemetry/opentelemetry-js>
- For more about Esbuild plugins: <https://esbuild.github.io/plugins/>

## License

APACHE 2.0 - See [LICENSE][license-url] for more information.

[license-url]: https://github.com/open-telemetry/opentelemetry-js-contrib/blob/main/LICENSE
[license-image]: https://img.shields.io/badge/license-Apache_2.0-green.svg?style=flat
[npm-url]: https://www.npmjs.com/package/@opentelemetry/auto-instrumentations-node
[npm-img]: https://badge.fury.io/js/%40opentelemetry%2Fauto-instrumentations-node.svg
[env-var-url]: https://github.com/open-telemetry/opentelemetry-specification/blob/main/specification/configuration/sdk-environment-variables.md#general-sdk-configuration
