**Note**: Before taking the time to craft a new cask, make sure
- it can be accepted by checking the [Rejected Casks FAQ document](https://github.com/Homebrew/homebrew-cask/blob/master/doc/faq/rejected_casks.md),
- check if there are no [open pull requests] for the same cask and
- check if the cask was not [already refused].

[open pull requests]: https://github.com/Homebrew/homebrew-cask/pulls
[already refused]: https://github.com/Homebrew/homebrew-cask/search?q=is%3Aclosed&type=Issues

## Adding a Cask

Making a new Cask is easy. Follow the directions in [Getting Set Up To Contribute](../../CONTRIBUTING.md#getting-set-up-to-contribute) to begin.

### Examples

Here’s a Cask for `shuttle` as an example. Note the comment above `url`, which is needed when [the url and homepage hostnames differ](../cask_language_reference/stanzas/url.md#when-url-and-homepage-hostnames-differ-add-a-comment)

```ruby
cask "shuttle" do
  version "1.2.9"
  sha256 "0b80bf62922291da391098f979683e69cc7b65c4bdb986a431e3f1d9175fba20"

  url "https://github.com/fitztrev/shuttle/releases/download/v#{version}/Shuttle.zip",
      verified: "https://github.com/fitztrev/shuttle/"
  appcast "https://github.com/fitztrev/shuttle/releases.atom"
  name "Shuttle"
  desc "Simple shortcut menu"
  homepage "https://fitztrev.github.io/shuttle/"

  app "Shuttle.app"

  zap trash: "~/.shuttle.json"
end
```

And here is one for `advancedcolors`. Note that it has an unversioned download (the download `url` does not contain the version number, unlike the example above). It also suppresses the checksum with `sha256 :no_check` (necessary since the checksum will change when a new distribution is made available). This combination of `version :latest` and `sha256 :no_check` is currently the preferred mechanism when a versioned download URL is not available and the cask does not have an `appcast`.

```ruby
cask "advancedcolors" do
  version :latest
  sha256 :no_check

  url "https://advancedcolors.com/AdvancedColors.zip"
  name "Advanced Colors"
  desc "Lightning fast swatch tool for designers and developers"
  homepage "https://advancedcolors.com/"

  app "AdvancedColors.app"
end
```

Here is a last example for `airdisplay`, which uses a `pkg` installer to install the application instead of a stand-alone application bundle (`.app`). Note the [`uninstall pkgutil` stanza](../cask_language_reference/stanzas/uninstall.md#uninstall-key-pkgutil), which is needed to uninstall all files which were installed using the installer.

You will also see how to adapt `version` to the download `url`. Use [our custom `version` methods](../cask_language_reference/stanzas/version.md#version-methods) to do so, resorting to the standard [Ruby String methods](https://ruby-doc.org/core/String.html) when they don’t suffice.

```ruby
cask "airdisplay" do
  version "3.4.2"
  sha256 "272d14f33b3a4a16e5e0e1ebb2d519db4e0e3da17f95f77c91455b354bee7ee7"

  url "https://www.avatron.com/updates/software/airdisplay/ad#{version.no_dots}.zip"
  appcast "https://www.avatron.com/updates/software/airdisplay/appcast.xml"
  name "Air Display"
  desc "Utility for using a tablet as a second monitor"
  homepage "https://avatron.com/applications/air-display/"

  depends_on macos: ">= :mojave"

  pkg "Air Display Installer.pkg"

  uninstall pkgutil: [
    "com.avatron.pkg.AirDisplay",
    "com.avatron.pkg.AirDisplayHost2",
  ]
end
```

### Generating a Token for the Cask

The Cask **token** is the mnemonic string people will use to interact with the Cask via `brew install`, etc. The name of the Cask **file** is simply the token with the extension `.rb` appended.

The easiest way to generate a token for a Cask is to run this command:

```bash
$ "$(brew --repository)/Library/Taps/homebrew/homebrew-cask/developer/bin/generate_cask_token" '/full/path/to/new/software.app'
```

If the software you wish to Cask is not installed, or does not have an associated App bundle, just give the full proper name of the software instead of a pathname:

```bash
$ "$(brew --repository)/Library/Taps/homebrew/homebrew-cask/developer/bin/generate_cask_token" 'Google Chrome'
```

If the `generate_cask_token` script does not work for you, see [Cask Token Details](#cask-token-details).

### The `brew create --cask` Command

Once you know the token, create your Cask with the handy-dandy `brew create --cask` command:

```bash
$ brew create --cask my-new-cask
```

This will open `$EDITOR` with a template for your new Cask, to be stored in the file `my-new-cask.rb`. Running the `create` command above will get you a template that looks like this:

```ruby
cask "my-new-cask" do
  version ""
  sha256 ""

  url "https://"
  name ""
  desc ""
  homepage ""

  app ""
end
```

### Cask Stanzas

Fill in the following stanzas for your Cask:

| name               | value       |
| ------------------ | ----------- |
| `version`          | application version; give the value `:latest` if only an unversioned download is available
| `sha256`           | SHA-256 checksum of the file downloaded from `url`, calculated by the command `shasum -a 256 <file>`. Can be suppressed by using the special value `:no_check`. (see [sha256](../cask_language_reference/stanzas/sha256.md))
| `url`              | URL to the `.dmg`/`.zip`/`.tgz`/`.tbz2` file that contains the application.<br />A [comment](../cask_language_reference/stanzas/url.md#when-url-and-homepage-hostnames-differ-add-a-comment) must be added if the hostnames in the `url` and `homepage` stanzas differ. [Block syntax](../cask_language_reference/stanzas/url.md#using-a-block-to-defer-code-execution) is available for URLs that change on every visit
| `name`             | the full and proper name defined by the vendor, and any useful alternate names (see [Name Stanza Details](../cask_language_reference/stanzas/name.md))
| `desc`             | one-line description of the software (see [Desc Stanza Details](../cask_language_reference/stanzas/desc.md))
| `homepage`         | application homepage; used for the `brew home` command
| `app`              | relative path to an `.app` bundle that should be moved into the `/Applications` folder on installation (see [App Stanza Details](../cask_language_reference/stanzas/app.md))

Other commonly-used stanzas are:

| name               | value       |
| ------------------ | ----------- |
| `appcast`          | a URL providing an appcast feed to find updates for this Cask. (see [Appcast Stanza Details](../cask_language_reference/stanzas/appcast.md))
| `pkg`              | relative path to a `.pkg` file containing the distribution (see [Pkg Stanza Details](../cask_language_reference/stanzas/pkg.md))
| `caveats`          | a string or Ruby block providing the user with Cask-specific information at install time (see [Caveats Stanza Details](../cask_language_reference/stanzas/caveats.md))
| `uninstall`        | procedures to uninstall a Cask. Optional unless the `pkg` stanza is used. (see [Uninstall Stanza Details](../cask_language_reference/stanzas/uninstall.md))

Additional `artifact` stanzas you might need for special use-cases can be found [here](../cask_language_reference/all_stanzas.md#at-least-one-artifact-stanza-is-also-required). Even more special-use stanzas are listed at [Optional Stanzas](../cask_language_reference/all_stanzas.md#optional-stanzas).

### Cask Token Details

If a token conflicts with an already-existing Cask, authors should manually make the new token unique by prepending the vendor name. Example: [unison.rb](../../Casks/unison.rb) and [panic-unison.rb](../../Casks/panic-unison.rb).

If possible, avoid creating tokens which differ only by the placement of hyphens.

To generate a token manually, or to learn about exceptions for unusual cases, see [token_reference.md](../cask_language_reference/token_reference.md).

### Archives With Subfolders

When a downloaded archive expands to a subfolder, the subfolder name must be included in the `app` value.

Example:

1. Texmaker is downloaded to the file `TexmakerMacosxLion.zip`.
2. `TexmakerMacosxLion.zip` unzips to a folder called `TexmakerMacosxLion`.
3. The folder `TexmakerMacosxLion` contains the application `texmaker.app`.
4. So, the `app` stanza should include the subfolder as a relative path:

  ```ruby
  app "TexmakerMacosxLion/texmaker.app"
  ```

## Testing Your New Cask

Give it a shot with:

```bash
export HOMEBREW_NO_AUTO_UPDATE=1
brew install my-new-cask
```

Did it install? If something went wrong, edit your Cask with `brew edit my-new-cask` to fix it.

Test also if the uninstall works successfully:

```bash
brew uninstall my-new-cask
```

If everything looks good, you’ll also want to make sure your Cask passes audit with:

```bash
brew audit --new-cask my-new-cask
```

You should also check stylistic details with `brew style`:

```bash
brew style --fix my-new-cask
```

Keep in mind all of these checks will be made when you submit your PR, so by doing them in advance you’re saving everyone a lot of time and trouble.

If your application and Homebrew Cask do not work well together, feel free to [file an issue](https://github.com/Homebrew/homebrew-cask#reporting-bugs) after checking out open issues.

## Finding a Home For Your Cask

We maintain separate Taps for different types of binaries. Our nomenclature is:

+ **Stable**: The latest version provided by the developer defined by them as such.
+ **Beta, Development, Unstable**: Subsequent versions to **stable**, yet incomplete and under development, aiming to eventually become the new **stable**. Also includes alternate versions specifically targeted at developers.
+ **Nightly**: Constantly up-to-date versions of the current development state.
+ **Legacy**: Any **stable** version that is not the most recent.
+ **Regional, Localized**: Any version that isn’t the US English one, when that exists.
+ **Trial**: Date-limited version that stops working entirely after it expires, requiring payment to lift the limitation.
+ **Freemium**: Gratis version that works indefinitely but with limitations that can be removed by paying.
+ **Fork**: An alternate version of an existing project, with a based-on but modified source and binary.
+ **Unofficial**: An *allegedly* unmodified compiled binary, by a third-party, of a binary that has no existing build by the owner of the source code.
+ **Vendorless**: A binary distributed without an official website, like a forum posting.
+ **Walled**: When the download URL is both behind a login/registration form and from a host that differs from the homepage.
+ **Font**: Data file containing a set of glyphs, characters, or symbols, that changes typed text.
+ **Driver**: Software to make a hardware peripheral recognisable and usable by the system. If the software is useless without the peripheral, it’s considered a driver.

### Stable Versions

Stable versions live in the main repository at [Homebrew/homebrew-cask](https://github.com/Homebrew/homebrew-cask). They should run on the latest release of macOS or the previous point release (High Sierra and Mojave as of late 2018).

### But There Is No Stable Version!

When an App is only available as beta, development, or unstable versions, or in cases where such a version is the general standard, then said version can go into the main repo.

### Beta, Unstable, Development, Nightly, or Legacy

When an App has a main stable version, alternative versions should be submitted to [Homebrew/homebrew-cask-versions](https://github.com/Homebrew/homebrew-cask-versions).

### Regional and Localized

When an App exists in more than one language or has different regional editions, [the `language` stanza should be used to switch between languages or regions](../../doc/cask_language_reference/stanzas/language.md).

### Trial and Freemium Versions

Before submitting a trial, make sure it can be made into a full working version without the need to be redownloaded. If an App provides a trial but the only way to buy the full version is via the Mac App Store, it does not belong in any of the official repos. Freemium versions are fine.

### Forks and Apps with Conflicting Names

Forks must have the vendor’s name as a prefix on the Cask’s file name and token. If the original software is discontinued, forks still need to follow this rule so as to not be surprising to the user. There are two exceptions which allow the fork to replace the main cask:

* The original discontinued software recommends that fork.
* The fork is so overwhelmingly popular that it surpasses the original and is now the de facto project when people think of the name.

For unrelated Apps that share a name, the most popular one (usually the one already present) stays unprefixed. Since this can be subjective, if you disagree with a decision, open an issue and make your case to the maintainers.

### Unofficial, Vendorless, and Walled Builds

We do not accept these casks since they offer a higher-than-normal security risk.

### Fonts

Font Casks live in the [Homebrew/homebrew-cask-fonts](https://github.com/Homebrew/homebrew-cask-fonts) repository. See the font repo [CONTRIBUTING.md](../../../../../homebrew-cask-fonts/blob/master/CONTRIBUTING.md)
for details.

### Drivers

Driver Casks live in the [Homebrew/homebrew-cask-drivers](https://github.com/Homebrew/homebrew-cask-drivers) repository. See the drivers repo [CONTRIBUTING.md](../../../../../homebrew-cask-drivers/blob/master/CONTRIBUTING.md)
for details.

## Submitting Your Changes

Hop into your Tap and check to make sure your new Cask is there:

```bash
$ cd "$(brew --repository)"/Library/Taps/homebrew/homebrew-cask
$ git status
# On branch master
# Untracked files:
#   (use "git add <file>..." to include in what will be committed)
#
#       Casks/my-new-cask.rb
```

So far, so good. Now make a feature branch `my-new-cask-branch` that you’ll use in your pull request:

```bash
$ git checkout -b my-new-cask-branch
Switched to a new branch 'my-new-cask-branch'
```

Stage your Cask with:

```bash
$ git add Casks/my-new-cask.rb
```

You can view the changes that are to be committed with:

```bash
$ git diff --cached
```

Commit your changes with:

```bash
$ git commit -v
```

### Commit Messages

For any git project, some good rules for commit messages are:

* The first line is commit summary, 50 characters or less,
* Followed by an empty line,
* Followed by an explanation of the commit, wrapped to 72 characters.

See [a note about git commit messages](https://tbaggery.com/2008/04/19/a-note-about-git-commit-messages.html) for more.

The first line of a commit message becomes the **title** of a pull request on GitHub, like the subject line of an email. Including the key info in the first line will help us respond faster to your pull.

For Cask commits in the Homebrew Cask project, we like to include the Application name, version number (or `:latest`), and purpose of the commit in the first line.

Examples of good, clear commit summaries:

* `Add Transmission.app v1.0`
* `Upgrade Transmission.app to v2.82`
* `Fix checksum in Transmission.app Cask`
* `Add CodeBox Latest`

Examples of difficult, unclear commit summaries:

* `Upgrade to v2.82`
* `Checksum was bad`

### Pushing

Push your changes from the branch `my-new-cask-branch` to your GitHub account:

```bash
$ git push {{my-github-username}} my-new-cask-branch
```

If you are using [GitHub two-factor authentication](https://help.github.com/articles/about-two-factor-authentication/) and set your remote repository as HTTPS you will need to set up a personal access token and use that instead of your password. Further information [here](https://help.github.com/articles/https-cloning-errors/#provide-access-token-if-2fa-enabled).

### Filing a Pull Request on GitHub

#### a) suggestion from git push

The `git push` command prints a suggestion to create a pull request:

```
remote: Create a pull request for 'new-cask-cask' on GitHub by visiting:
remote:      https://github.com/{{my-github-username}}/homebrew-cask/pull/new/my-new-cask-branch
```

#### b) use suggestion at Github website

Now go to the [`homebrew-cask` GitHub repository](https://github.com/Homebrew/homebrew-cask). GitHub will often show your `my-new-cask-branch` branch with a handy button to `Compare & pull request`.

#### c) manually create a pull request at Github website

Otherwise, click the `New pull request` button and choose to `compare across forks`. The base fork should be `Homebrew/homebrew-cask @ master`, and the head fork should be `my-github-username/homebrew-cask @ my-new-cask-branch`. You can also add any further comments to your pull request at this stage.

#### Congratulations!

You are done now, and your Cask should be pulled in or otherwise noticed in a while. If a maintainer suggests some changes, just make them on the `my-new-cask-branch` branch locally and [push](#pushing).

## Cleaning up

After your Pull Request is submitted, you should get yourself back onto `master`, so that `brew update` will pull down new Casks properly:

```bash
cd "$(brew --repository)"/Library/Taps/homebrew/homebrew-cask
git checkout master
```

if you set the variable `HOMEBREW_NO_AUTO_UPDATE` then clean it up with:

```bash
unset HOMEBREW_NO_AUTO_UPDATE
```

