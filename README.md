# ihp-tailwind-bootstrapper

ihp-tailwind-bootstrapper allows you to use the tailwind standalone compiler with IHP without adding npm, and has cross-platform support.

Adding npm/nodejs only for Tailwind adds way too much complexity. Avoiding npm makes it also much easier to deploy, for example when deploying IHP apps to Shipnix.

This also better ensures that your project's frontend assets doesn't break due to changes in npm.

## Install

In the root of your ihp project, pull the repository and delete the `.git` folder with this one-lines

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

After this, you can check these file into version control.

## Why is only the `tailwind/binary/tailwindcss-linux-x64` being checked into version control?

The `tailwindcss-linux-x64` file is required for building tailwind on a NixOS server with x64. It's better to have this checked into git instead of trying to download it while building.

The other Tailwind binaries are not necessary to check into version control as they can be downloaded automatically as needed.

## How to use

For deploying with Shipnix, Tailwind should now just work.

If it doesn't work, make sure that `static/app.css` is being bundled in the `Makefile`:

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

For running the development version, you can run this line:

```sh
nix-shell --run 'make tailwind-dev'
```

Alternatively, you can replace the `RunDevServer` line in your `./start` script with this to run Tailwind automatically when running the IHP development server.

```sh
# Finally start the dev server
wait & make tailwind-dev & RunDevServer
```
