# aws-as

*Built on top of [aws-vault][aws-vault-gh].  Allows fast and easy switching
between [aws-cli][aws-cli-site] [profiles][aws-cli-doc-prof], and running
user-supplied commands in the context of what we call **"the in-effect
profile"**.  Also provides features for **"hoisting"** the **`AWS_*`**
credential variables (**`AWS_ACCESS_KEY_ID`**, **`AWS_SECRET_ACCESS_KEY`**,
and **`AWS_SESSION_TOKEN`**) into the current shell process and otherwise
manipulating them (export, unexport, unset, ...).*

The `aws-as` project provides a collection of tools for switching between
cached sets of AWS IAM User and/or Role credentials when working at the
command line.

Think "venv for AWS IAM identities" and you'll be on the right track.

Allows you to express within a given shell, "I now want to interact with AWS
*as* `${iam_user_or_role}`." Temporary credentials are cached, and reused as
you switch back and forth between AWS accounts. Works with MFA. The current
IAM User or Role name is reflected in the shell prompt.

The provided **`aws-as`** and **`aagg`** commands work as a layer on top of
**`aws-vault`** (from [99designs][99designs-gh]), so work with your existing
aws-cli configuration. The provided tooling works alongside of aws-vault it to
augment its capabilities.

For full details, see the project's website:

   * https://salewski.github.io/aws-as/
   * http://salewski.email/aws-as/


## The problem

You work with [AWS from the command line][aws-cli-site], and you have several
different tools that all want you to manage your credentials in different
ways. Each has it's own strengths and weaknesses in terms of obtaining and/or
caching AWS credentials, but their approaches are not very inter-operable.

You try to follow AWS security best practices by requiring all API access to
be done via IAM Users or Roles that have authenticated using
[MFA][wikipedia-mfa]. This adds friction in addition to the above mentioned
incompatibilities because MFA support in the various tools is hit or miss.

You can live with one-time pain of MFA authenticating a single User or Role,
but when you need to switch between Roles in multiple accounts it quickly
becomes too cumbersome for use.

The `aws-cli` "profiles" cannot be directly leveraged in a general way to
smooth over some of the rough points because no profile-naming scheme can be
enforced within a team, or across teams.

You want to work across tools, and the only thing that works everywhere is
exporting some subset of the environment variables `AWS_PROFILE` (see above
note about profiles), `AWS_ACCESS_KEY_ID`, `AWS_SECRET_ACCESS_KEY`, and
`AWS_SESSION_TOKEN`.

You do not want to have your `AWS_*` credential variables in your current
shell environment, much less exported. Except when sometimes you do (for ad
hoc scripting).

Everybody has their own scripting to deal with different subsets of the
problem, in different ways, with varying degrees of success.


## Tools we care about most

   * [aws-cli][aws-cli-gh]
   * [AWS Go SDK][aws-sdk-go-gh]  (what [terraform-provider-aws][tf-prov-aws-gh] uses for AWS API calls)
   * [terraform][terraform-gh]    (especially [terraform-provider-aws][tf-prov-aws-gh])


## What doesn't (quite) work, and why

### aws-cli cached credentials

The [aws-cli][aws-cli-gh] program contains only partial support for caching
credentials and for using MFA. Current deficiencies for our purposes include:

   * The `sts` subcommand's `get-session-token` operation can be used to
     obtain temporary credentials for an IAM User, but:

      * the command line user must know and specify the device "serial number"
        (ARN) on the command line (via the `--serial-number` opt); the
        `mfa_serial` config file key is only used when assuming roles;

      * unlike IAM Role credentials, temporary credentials obtained for a User
        that has authenticated with MFA are not cached.

   * Even though `aws-cli` *will* cache temporary credentials obtained when
     assuming a Role, the `aws-go-sdk` does not have access to those cached
     credentials (more specifically, by design does not access them because
     they are considered the domain of the `aws-cli` tool itself, not
     something shared between the `aws-cli` program and the various
     language-specific AWS SDK's).

FIXME: add references for the above info


### aws-cli profiles

Because profile names cannot be easily managed across different members within
a team, or across teams, they are not a good choice for direct referencing
from configuration used to drive production tooling.

Even in an environment in which profile names could be standardized, using
them would not solve the credentials non-caching problem for IAM Users (as
opposed to Roles).


### aws-vault with multiple AWS identities

The `aws-vault` tool does a great job of authenticating (using MFA, where
necessary) and caching temporary credentials. In fact, in this regard it fills
all of the gaps mentioned above for of `aws-cli`. However, one must always
specify explicitly the name of the profile whose credentials are to be used
for a given operation, and that gets tedious; it does not have a way to
"remember" the name of "the current" profile.

Also, while one can always start a subshell via `aws-vault` that will have the
`AWS_*` credential variables set for a given profile, there is no canned way
to get those settings into the current environment.

Enter `aws-as`.


## The solution

Use **`aws-vault`** to handle caching of temporary AWS credentials for one or
more **`aws-cli`** "profiles" (IAM Users and/or Roles).

Use **`aws-as`** to quickly select an **"in-effect profile"**, and to quickly
switch between profiles.

Use **`aagg`** to invoke arbitrary commands using the **"in-effect profile"**,
using the underlying **`aws-vault`** tool to actually run the command.

Use **`aws-as --hoist`** and/or **`aws-as --export`** to **"hoist"** the
`AWS_*` credential variables into the current shell, when needed.


## The approach

The `aws-as` project provides a user interface experience that is a cross
between Python's "venv" ("virtual environment") and `ssh-agent` (specifically,
the mechanisms provided by `ssh-agent -s` and `ssh-agent -c`).

The user invokes a command (`aws-as-activate -s`) that emits a handful of
shell functions in the dialect of the user's shell (bash is currently the only
supported dialect, but if there is interest then support for other shells
could be added). Under normal usage, the user would `eval` the output of the
command to install those functions in the environment of the current shell,
like this:

```
    $ eval $(aws-as-activate -s)
```

Part of the installed functionality is an `aws-as-deactivate` function, which
provides the ability for the above added bits to delete themselves when no
longer needed:

```
    $ aws-as-deactivate
```

Other "command functions" installed include `aws-aw` and `aagg`, with purposes
described above.

The reason for installing shell functions is to provide the ability to
manipulate the current shell environment without requiring that the user
"source" script files. The "in-effect profile" is tracked in the the current
shell environment. (Note that tracking of the "in-effect profile" *does not*
use the `AWS_PROFILE` environment variable.)

The idea is that the `aws-as` tools augment `aws-vault` (and indirectly,
`aws-cli`), and to allow that functionality to be leveraged more easily when
working with multiple AWS identities (Users and/or Roles). Profiles are
leveraged, but only at the local level; they help us get the environment into
a shape that is useful to other tooling without those other tools having to
support or honor or reference profiles.

Once the user has activated the `aws-as` machinery in a shell, the user's
shell prompt will be augmented to show the "current" IAM User or Role that is
"in effect" (or `_`, if no profile is in-effect).

From there the user uses the `aws-as` command to both authenticate and switch
between IAM Users and Role (perhaps between different
accounts). Authentication (including prompting for MFA tokens) is actually
handled by the underlying `aws-vault` tool.


# Build instructions

The `aws-as` project is distributed as a standard GNU autotools-based package.

The only prerequisite is bash >= 4.0. Assuming you have that and a normal
Unix-like OS, then you are ready to build. From an unpacked release tarball,
run:
```
    $ ./configure
    $ make
    $ make check
    $ make install
```

By default `make install` will install into subdirectories of `/usr/local`. To
change where things will get installed, use `./configure --prefix=/some/path`.


# License

GPLv2+: GNU GPL version 2 or later <http://gnu.org/licenses/gpl.html>

Unless otherwise stated by a different license notice in a particular
file, all files in the `aws-as` project are made available under the GNU GPL
version 2, or (at your option) any later version.

See the [COPYING] file for the full license.

Copyright (C) 2020 Alan D. Salewski <ads@salewski.email>

    This program is free software; you can redistribute it and/or modify
    it under the terms of the GNU General Public License as published by
    the Free Software Foundation; either version 2 of the License, or
    (at your option) any later version.

    This program is distributed in the hope that it will be useful,
    but WITHOUT ANY WARRANTY; without even the implied warranty of
    MERCHANTABILITY or FITNESS FOR A PARTICULAR PURPOSE.  See the
    GNU General Public License for more details.

    You should have received a copy of the GNU General Public License along
    with this program; if not, write to the Free Software Foundation, Inc.,
    51 Franklin Street, Fifth Floor, Boston, MA 02110-1301 USA.


[aws-cli-gh]:     https://github.com/aws/aws-cli          "GitHub repo: aws/aws-cli"
[aws-cli-site]:   https://aws.amazon.com/cli/             "AWS command line interface"
[aws-sdk-go-gh]:  https://github.com/aws/aws-sdk-go       "GitHub repo: aws/aws-go-sdk"
[aws-vault-gh]:   https://github.com/99designs/aws-vault  "GitHub repo: 99designs/aws-vault"
[99designs-gh]:   https://github.com/99designs            "GitHub account: 99designs"
[terraform-gh]:   https://github.com/hashicorp/terraform  "GitHub repo: hashicorp/terraform"

[aws-cli-doc-prof]: https://docs.aws.amazon.com/cli/latest/userguide/cli-configure-profiles.html "doc: aws-cli profiles"

[tf-prov-aws-gh]:  https://github.com/terraform-providers/terraform-provider-aws  "GitHub repo: terraform-providers/terraform-provider-aws"
[wikipedia-mfa]:   https://en.wikipedia.org/wiki/Multi-factor_authentication      "wikipedia.org: Multi-factor authentication"
