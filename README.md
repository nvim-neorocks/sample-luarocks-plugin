# Luarocks :purple_heart: Neovim

A simple sample repository showing how to push your Neovim plugins to luarocks.

For more information about the usefulness of a real package manager check the following blogs
by Marc Jakobi:
- [Publish your plugins to luarocks](https://mrcjkb.dev/posts/2023-01-10-luarocks-tag-release.html)
- [Test your Neovim plugins with luarocks & busted](https://mrcjkb.dev/posts/2023-06-06-luarocks-test.html)

My favourite quote from the blog:
> I’m a strong believer that we need plugin authors to publish their packages to LuaRocks before package managers start supporting it. Just like we had cars before petrol stations, and electric vehicles before charging infrastructure.

# Getting Started

Pushing your Neovim plugin to luarocks is a very simple process, so let's get going!

## Setting up a Luarocks Account

First of all, visit https://luarocks.org and register an account. They support registration via a github account,
which is arguably the most convenient. You may also opt for a simple username/password login.

Next up, let's get ourselves an API key that we will use to publish our plugin to luarocks.

### Creating an API Key

![step 1](https://github.com/nvim-neorg/neorg/assets/76052559/727c325f-a9a8-493c-811f-b06871b6eabe)

Once you're logged in, head over to your settings.


---

![step 2](https://github.com/nvim-neorg/neorg/assets/76052559/392327eb-08b3-479e-8047-c2e78b21e5d8)

Select the `API keys` section.

---

![step 3](https://github.com/nvim-neorg/neorg/assets/76052559/591db8fe-472f-4847-a4f1-29d7478c114c)

Press `Generate New Key`, wait a bit, and copy the newly provided key! If you ever feel like
the key has been stolen, you can revoke it here too.

**Be sure to copy this key somewhere safe! We'll need it for later.**

## The Semantic Versioning Scheme

Luarocks versions its packages using [semver](https://semver.org/). The versioning follows a `major.minor.patch`
scheme (e.g. `1.0.0`) - the major version is increased on every breaking change, the minor is increased
on every feature release, and the patch is increased on every bug fix.

We would like our Neovim plugins to also follow this versioning scheme, so let's set up a tool that will
do it for us.

### Installing Release-Please (recommended)

Because of the way we'll be setting up the workflow later on, it is recommended that you install [`release-please`](https://github.com/google-github-actions/release-please-action),
a tool that automates the semver process. All you do is write commits in the [conventional commits](https://www.conventionalcommits.org/en/v1.0.0/)
style (for example `fix: annoying bug` or `feat(ui): add new buttons`) and release-please generates a new release
version in the form of a pull request to your repository.

> **Note**
> The pull request made by the Github action gets updated on every new commit that you make - this means you control
  when a new version of your plugin is published by **merging the pull request**.
>
> When you do this, a new tag and release will be created on your repository, which is a critical step.

In your repository root (where the `.git` directory is located) create a file under `.github/workflows/release-please.yml`.
Paste the following contents into the file:
```yml
name: Release Please

on:
  push:
    branches:
      - main
  workflow_dispatch:

jobs:
  release:
    name: release
    runs-on: ubuntu-latest
    steps:
      - uses: google-github-actions/release-please-action@v3
        with:
          release-type: simple
          package-name: YOUR-PLUGIN-NAME
```

> **Important**
> Make sure you replace `your-plugin-name` with the name of your plugin!

Afterwards, run:
- `git add .github/`
- `git commit -m "ci: add release-please workflow"`
- `git push`

Congratulations, you've just set up `release-please`! It will now run in the background and after a while will create a pull
request to your repository. We recommend that you DO NOT merge the pull request yet, but don't be afraid if you already did,
you'll just need to run an extra command later in the tutorial.

## Publishing to Luarocks

After your repository has a working versioning scheme we may now move on to actually publishing our plugin to luarocks.

It is recommended that your repository has as much metadata as possible (a license that is detected by github on the right side
of your plugin's page, a repo title/description, github topics etc.) in your Github repository.

Time to add our API key!

### Adding our API Key

![step 5](https://github.com/vhyrro/sample-luarocks-plugin/assets/76052559/3fab5791-189f-42cb-b8b2-824af45a114d)

Navigate to your Github repository and enter its settings.

---

![step 6](https://github.com/vhyrro/sample-luarocks-plugin/assets/76052559/ab8ca077-3f8a-4263-9583-2abcebdf9e6a)

In the settings go to `Security > Secrets and variables > Actions`.

---

![step 7](https://github.com/vhyrro/sample-luarocks-plugin/assets/76052559/a67ddd29-5021-4d45-ae74-8a40507ccc51)

Click `New repository secret`, almost there!

---

![step 8](https://github.com/vhyrro/sample-luarocks-plugin/assets/76052559/48590516-cf03-43a4-9ded-575c5a63caf5)

Name the secret `LUAROCKS_API_KEY` (make sure it's spelled exactly like this!). In the `Secret` field paste
the API key you copied from the first step of the tutorial.

Click `Add secret` and there you go! This secret is only visible to you (the
repo owner) and administrators of your repository, so don't be afraid of anyone using it without your consent!

### Setting up `luarocks-tag-release`

Similarly to the `release-please` setup process, create a `.github/workflows/luarocks.yml` file from the root of your repo.

Paste the following into the file:
```yml
name: Push to Luarocks

on:
  push:
    tags: # Will upload to luarocks.org when a tag is pushed
      - "*"
  pull_request: # Will test a local install without uploading to luarocks.org
  workflow_dispatch:

jobs:
  luarocks-upload:
    runs-on: ubuntu-22.04
    steps:
      - uses: actions/checkout@v3
      - name: LuaRocks Upload
        uses: nvim-neorocks/luarocks-tag-release@v5
        env:
          LUAROCKS_API_KEY: ${{ secrets.LUAROCKS_API_KEY }}
```

Afterwards, run:
- `git add .github/`
- `git commit -m "ci: add luarocks workflow"`
- `git push`

Just one more thing now!

## Publish a Release of your Plugin

Now that we have everything set up, all the pieces can fall into place.
Go to the Pull Requests tab in your repository where `release-please` should have created a PR for you
(assuming your were using conventional commits). Merge that pull request, and the chain reaction should begin!
First, release-please should (after a minute or so) publish a new tag with a version. Afterwards, the luarocks
workflow should trigger and your plugin should end up on `luarocks.org`!

# Troubleshooting

### I already merged the release-please PR earlier!

If you merged your PR beforehand, don't worry. We'll need to install the `gh` CLI tool in order to run
the workflow manually. There are many resources for that online, just search "setting up the gh cli tool" in
your favourite search engine! After you are logged in, run `gh workflow run luarocks.yml`. After a minute you should
see your plugin on luarocks :D
