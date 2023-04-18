# ihp-tailwind-bootstrapper

ihp-tailwind-bootstrapper allows you to use the [Tailwind Standalone CLI](https://tailwindcss.com/blog/standalone-cli) with IHP **without adding npm**, and with cross-platform support.

Adding npm/nodejs only for Tailwind adds way too much complexity for this one tool, so better avoid it if you can.

Avoiding npm and instead using a pre-built binary makes your app deployment faster and easier.

This also better ensures that your project's frontend assets doesn't break due to changes in npm.

## Install

From the root of your ihp project, pull the repository and delete the `.git` folder in the tailwind folder with this one-liner:

```sh
git clone https://github.com/ship-nix/ihp-tailwind-bootstrapper.git tailwind && rm -rf tailwind/.git
```

### Add dev and production build scripts

In the `Makefile` in your IHP project, you can add these scripts:

```sh
tailwind-dev:
	bash tailwind/tailwindcss -c tailwind/tailwind.config.js -i ./tailwind/app.css -o static/app.css --watch


static/app.css:
	NODE_ENV=production bash tailwind/tailwindcss -c tailwind/tailwind.config.js -i ./tailwind/app.css -o static/app.css --minify
```

### Update the .gitignore

As the `static/app.css` is now generated code, it’s best to put the `static/app.css` into our `.gitignore` in the root of you IHP project.

```sh
git rm -f static/app.css # Remove the existing app.css
printf '\nstatic/app.css' >> .gitignore
git add .gitignore
```

## How to use

For deploying with Shipnix, vanilla NixOS, or most other deployment methods, Tailwind should just work in production.

### Use in development

For running the development version, you can run this line:

```sh
nix-shell --run 'make tailwind-dev'
```

Optionally, you can replace the `RunDevServer` line in your `./start` script with this to run Tailwind automatically when running the IHP development server.

```sh
# Finally start the dev server
wait & make tailwind-dev & RunDevServer
```

### Troubleshooting

If it doesn't work, ensure that `static/app.css` is being bundled in the `Makefile`:

```sh
CSS_FILES += static/app.css
```

and that `/app.css` is included in the stylesheets function in your `Layout.hs` file:

```hs
stylesheets :: Html
stylesheets = [hsx|
        <link rel="stylesheet" href={assetPath "/vendor/bootstrap-5.2.1/bootstrap.min.css"}/>
        <link rel="stylesheet" href={assetPath "/vendor/flatpickr.min.css"}/>
        <link rel="stylesheet" href={assetPath "/app.css"}/>
    |]
```

## Update Tailwind version

If you require a newer version of tailwind, you can delete the `tailwind/binary` folder:

```sh
rm -rf tailwind/binary
```

Then in the `tailwind/tailwindcss` bash script, change the `tag` to an [actual Tailwind release](https://github.com/tailwindlabs/tailwindcss/releases):

```bash
tag="v3.3.1"
```

Then run the tailwind script to install the new packages:

```
bash tailwind/tailwindcss
```

## Q: Why is only the `tailwind/binary/tailwindcss-linux-x64` being checked into version control?

The `tailwindcss-linux-x64` file is required for building tailwind on a NixOS server with x64. It's better to have this checked into git instead of trying to download it while building.

The other Tailwind binaries are not necessary to check into version control as they can be downloaded automatically as needed during development.

## Q: How do I use non-official Tailwind plugins without npm?

Let's take [tailwind-elements](https://tailwind-elements.com/) as an example.

First, we can create a `plugins` folder inside the tailwind directory.

```
mkdir -p tailwind/plugins
```

You can browse files for a distribution build at jsDeliver: https://cdn.jsdelivr.net/npm/tw-elements/dist/

In this case, you can take the `plugin.cjs` and save it into the `plugins` folder as `tailwind/plugins/tw-elements.cjs`.

Here is a one-liner you can run at the root of your IHP project:

```sh
curl -L -o tailwind/plugins/tw-elements.cjs https://cdn.jsdelivr.net/npm/tw-elements@1.0.0-beta2/dist/plugin.cjs
```

You can then add it to your `tailwind.config.js`:

```js
const plugin = require("tailwindcss/plugin");
const tailwindElements = require("./plugins/tw-elements.cjs");

module.exports = {
  mode: "jit",
  theme: {
    extend: {},
  },
  content: ["Web/View/**/*.hs"],
  safelist: [
    // Add custom class names.
    // https://tailwindcss.com/docs/content-configuration#safelisting-classes
  ],
  plugins: [require("@tailwindcss/forms"), tailwindElements],
};
```

Now, the plugin should work as you recompile Tailwind.

If you also require the JavaScript this library provides, you can just download `tw-elements.umd.min.js` from jsDeliver and place it into your `static` folder, import it from your html template, and your done!

Yes, this is a primitive way of working, and against the pattern of tapping into the vastly complex ecosystem that is npm, but also saves you from complexity, future headaches and even potential security issues.
