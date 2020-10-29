# lerna-add-remove-issue

Display the issue of adding dependencies that breaks the ability to remove them later on.

**IMPORTANT:** This is applicable to `npm` and `yarn`. To switch to `yarn`, add `npmClient: yarn` to lerna.json.

Tested with `npm` 7.0.3 and npm 6.14.8. Tested with `yarn` 1.22.10.

## Steps to replicate

- `npm ci` to install the specific Lerna version at the time of the issue. (v3.22.1).
- `npx lerna bootstrap`.
- `npx lerna add my-package-not-in-the-registry --scope=first`.
  - Results:
    - A `node_modules` folder will be created within `first` folder. It will contain a symlink to `my-package-not-in-the-registry` local folder.
    - No `package-json.lock` will be generated.
    - `my-package-not-in-the-registry` will added to the `first` package.json.
- Add a dev dependency e.g: `nodemon`.
  - `npx lerna add nodemon --scope=first`.
  - Results:
    - `nodemon` is added to `package.json` under `devDependencies`
    - A `package-json.lock` is created, including a `../my-package-not-in-the-registry` local reference.
- Add a regular dependency e.g: `isomoprhic-unfetch`.
  - `npx lerna add isormoprhic-unfetch --scope=first`.
  - Results:
    - `isomorphic-unfetch` is added to `package.json` under `dependencies`.
    - `nodemon` and `my-package-not-in-the-registry` remain under `dependencies` and `devDependencies` respectively in `package.json`.
    - **ISSUE:** My `../my-package-not-in-the-registry` reference is removed from `package-json.lock`.
- Attempt to add/remove any depedency:
  - `npx lerna exec "npm uninstall nodemon" --scope=first`
  - **ISSUE:** The command will fail because the reference to `my-package-not-in-the-registry` is missing **and** it is not available in the public registry.
  - **IMPORTANT:** If the name/version of the local package does match the registry, the `remove` command will succeed, **but** the wrong package reference will be saved to `package-json.lock`.

**SOLUTION:** Manually remove `package-json.lock`, run an add/remove command. E.g: `npx lerna exec "npm uninstall nodemon" --scope=first`.

## Cleanup

To retry the steps:

- `npx lerna clean` remove all `node_modules`. When asked, submit `y`.
- Remove all `package-json.lock` or `yarn.lock` files. They are `gitignored` for the purposed of this example.
- Remove all changes to `package.json` dependencies. E.g: `git checkout .`

## Notes

`nodemon` and `isomorphic-unfetch` are examples. I was able to replicate the issue with other packages, and in different order.
