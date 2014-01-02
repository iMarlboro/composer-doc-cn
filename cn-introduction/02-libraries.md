# 库

本章将告诉你如何通过 Composer 来安装你的库。

## 每一个项目都是一个包

只要你有一个 `composer.json` 文件在目录中，那么整个目录就是一个包。当你添加一个 `require` 到项目中，你就是在创建一个依赖于其它库的包。你的项目和库之间唯一的区别是，你的项目是一个没有名字的包。

为了使它成为一个可安装的包，你需要给它一个名称。你可以通过 `composer.json` 中的 `name` 来定义：

    {
        "name": "acme/hello-world",
        "require": {
            "monolog/monolog": "1.0.*"
        }
    }

在这种情况下项目的名称为 `acme/hello-world`，其中 `acme` 是供应商的名称。供应商的名称是必须填写的。

> **注意：** 如果你不知道拿什么作为供应商的名称，
> 那么使用你 github 上的用户名通常是不错的选择。
> 虽然包名不区分大小写，但惯例是使用小写字母，并用连字符作为单词的分隔。

## 平台软件包

Composer 将那些已经安装在系统上，但并不是由 Composer 安装的包视为一个虚拟的平台软件包。这包括PHP本身，PHP扩展和一些系统库。

* `php` 表示用户的 PHP 版本要求，你可以对其做出限制。例如 `>=5.4.0`。如果需要64位版本的 PHP，你可以使用 `php-64bit` 进行限制。

* `ext-<name>` 可以帮你指定需要的 PHP 扩展（包括核心扩展）。通常 PHP 拓展的版本可以是不一致的，将它们的版本约束为 `*` 是一个不错的主意。一个 PHP 扩展包的例子：包名可以写成 `ext-gd`。

* `lib-<name>` 允许对 PHP 库的版本进行限制。以下是可供使用的名称：`curl`、`iconv`、`libxml`、`openssl`、`pcre`、`uuid`、`xsl`。

你可以使用 `composer show --platform` 命令来获取可用的平台软件包的列表。

## Specifying the version

You need to specify the package's version some way. When you publish your
package on Packagist, it is able to infer the version from the VCS (git, svn,
hg) information, so in that case you do not have to specify it, and it is
recommended not to. See [tags](#tags) and [branches](#branches) to see how
version numbers are extracted from these.

If you are creating packages by hand and really have to specify it explicitly,
you can just add a `version` field:

    {
        "version": "1.0.0"
    }

> **Note:** You should avoid specifying the version field explicitly, because
> for tags the value must match the tag name.

### Tags

For every tag that looks like a version, a package version of that tag will be
created. It should match 'X.Y.Z' or 'vX.Y.Z', with an optional suffix
of `-patch`, `-alpha`, `-beta` or `-RC`. The suffixes can also be followed by
a number.

Here are a few examples of valid tag names:

    1.0.0
    v1.0.0
    1.10.5-RC1
    v4.4.4beta2
    v2.0.0-alpha
    v2.0.4-p1

> **Note:** Even if your tag is prefixed with `v`, a [version constraint](01-basic-usage.md#package-versions)
> in a `require` statement has to be specified without prefix
> (e.g. tag `v1.0.0` will result in version `1.0.0`). 

### Branches

For every branch, a package development version will be created. If the branch
name looks like a version, the version will be `{branchname}-dev`. For example
a branch `2.0` will get a version `2.0.x-dev` (the `.x` is added for technical
reasons, to make sure it is recognized as a branch, a `2.0.x` branch would also
be valid and be turned into `2.0.x-dev` as well. If the branch does not look
like a version, it will be `dev-{branchname}`. `master` results in a
`dev-master` version.

Here are some examples of version branch names:

    1.x
    1.0 (equals 1.0.x)
    1.1.x

> **Note:** When you install a development version, it will be automatically
> pulled from its `source`. See the [`install`](03-cli.md#install) command
> for more details.

### Aliases

It is possible to alias branch names to versions. For example, you could alias
`dev-master` to `1.0.x-dev`, which would allow you to require `1.0.x-dev` in all
the packages.

See [Aliases](articles/aliases.md) for more information.

## Lock file

For your library you may commit the `composer.lock` file if you want to. This
can help your team to always test against the same dependency versions.
However, this lock file will not have any effect on other projects that depend
on it. It only has an effect on the main project.

If you do not want to commit the lock file and you are using git, add it to
the `.gitignore`.

## Publishing to a VCS

Once you have a vcs repository (version control system, e.g. git) containing a
`composer.json` file, your library is already composer-installable. In this
example we will publish the `acme/hello-world` library on GitHub under
`github.com/username/hello-world`.

Now, to test installing the `acme/hello-world` package, we create a new
project locally. We will call it `acme/blog`. This blog will depend on
`acme/hello-world`, which in turn depends on `monolog/monolog`. We can
accomplish this by creating a new `blog` directory somewhere, containing a
`composer.json`:

    {
        "name": "acme/blog",
        "require": {
            "acme/hello-world": "dev-master"
        }
    }

The name is not needed in this case, since we don't want to publish the blog
as a library. It is added here to clarify which `composer.json` is being
described.

Now we need to tell the blog app where to find the `hello-world` dependency.
We do this by adding a package repository specification to the blog's
`composer.json`:

    {
        "name": "acme/blog",
        "repositories": [
            {
                "type": "vcs",
                "url": "https://github.com/username/hello-world"
            }
        ],
        "require": {
            "acme/hello-world": "dev-master"
        }
    }

For more details on how package repositories work and what other types are
available, see [Repositories](05-repositories.md).

That's all. You can now install the dependencies by running Composer's
`install` command!

**Recap:** Any git/svn/hg repository containing a `composer.json` can be added
to your project by specifying the package repository and declaring the
dependency in the `require` field.

## Publishing to packagist

Alright, so now you can publish packages. But specifying the vcs repository
every time is cumbersome. You don't want to force all your users to do that.

The other thing that you may have noticed is that we did not specify a package
repository for `monolog/monolog`. How did that work? The answer is packagist.

[Packagist](https://packagist.org/) is the main package repository for
Composer, and it is enabled by default. Anything that is published on
packagist is available automatically through Composer. Since monolog
[is on packagist](https://packagist.org/packages/monolog/monolog), we can depend
on it without having to specify any additional repositories.

If we wanted to share `hello-world` with the world, we would publish it on
packagist as well. Doing so is really easy.

You simply hit the big "Submit Package" button and sign up. Then you submit
the URL to your VCS repository, at which point packagist will start crawling
it. Once it is done, your package will be available to anyone.

&larr; [Basic usage](01-basic-usage.md) |  [Command-line interface](03-cli.md) &rarr;