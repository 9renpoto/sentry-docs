---
title: 'Source Maps for Other Platforms'
---

Currently automatic source map handling is only implemented for iOS with Xcode and Android with gradle. If you manually invoke the [react-native packager](https://github.com/facebook/metro) you can however get source maps anyways by passing _–sourcemap-output_ to it.

If you do get source maps you can upload them with `sentry-cli`. However make sure to pass `--rewrite` to the `upload-sourcemaps` command which will fix up the source maps before upload (inlines sources etc.).

Example:

```bash
react-native bundle \
  --dev false \
  --platform android \
  --entry-file index.android.js \
  --bundle-output android.main.bundle \
  --sourcemap-output android.main.bundle.map
```

To then upload you should use this:

```bash
node_modules/@sentry/cli/bin/sentry-cli releases \
    files RELEASE_NAME \
    upload-sourcemaps \
    --dist DISTRIBUTION_NAME \
    --strip-prefix /path/to/project/root \
    --rewrite /path/to/your/files
```

The values for `RELEASE_NAME` and `DISTRIBUTION_NAME` are as follows:

`RELEASE_NAME`:

: the bundle ID or package name (reverse dns notation of your app) followed by a dash and the human readable version name that is set for your release. So for instance `com.example.myapp-1.0`.

`DISTRIBUTION_NAME`:

: This is the version code or build id depending on your platform. So for instance just set this to whatever is set in your _Info.plist_ or what your gradle setup generates (eg: `52`).
