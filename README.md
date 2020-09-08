# aws-as

The `aws-as` project provides a collection of tools for switching between
cached sets of AWS IAM User and/or Role credentials when working at the
command line.

Think "venv for AWS IAM identities", and you're on the right track.

Allows you to express within a given shell, "I now want to interact with AWS
*as* `${iam_user_or_role}`." Temporary credentials are cached, even for IAM
Users just authenticating using MFA (`sts:GetSessionToken`). The current IAM
User or Role name is reflected in the shell prompt.


**This is a WIP repo; just created (2020-09-07)**

**!!! Not yet ready for use !!!**


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

Everybody has their own scripting to deal with different subsets of the
problem, in different ways, with varying degrees of success.


## Tools we care about most

   * [aws-cli][aws-cli-gh]
   * [AWS Go SDK][aws-sdk-go-gh]  (what [terraform-provider-aws][tf-prov-aws-gh] uses for AWS API calls)
   * [terraform][terraform-gh]    (especially [terraform-provider-aws][tf-prov-aws-gh])


## What doesn't (quite) work, and why

### aws-cli cached credentials

The most full-featured caching and MFA support seems to be in the
[aws-cli][aws-cli-gh] program. Current deficiencies for our purposes, though,
include:

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


## The solution (for now)

The most (only?) portable cross-tool credential sharing mechanism I have found
is the use of the `AWS_*` environment variables mentioned above. Therefore,
the solution provided by the `aws-as` tool is to leverage existing mechanisms
in the `aws-cli` to authenticate (with or without MFA, as needed), and to
provide a may to switch those in and out of the user's shell environment. All
short-term credentials are cached, and you can query about the expiration time
of the "current" set configured in the environment.


## The approach

This idea is still in flux, but the approach envisioned is to provide a user
interface experience that is a cross between Python's "venv" ("virtual
environment") and `ssh-agent` (specifically, the mechanisms provided by
`ssh-agent -s` and `ssh-agent -c`).

The user will invoke a command (e.g., `aws-as-activate -s`) to emit a handful
of shell functions in the dialect of the user's shell (bash will be supported
first, with the intent later add support for Bourne or strict POSIX shells,
zsh, csh/tcsh, and maybe fish?). The user would `eval` the output of the
command to install those functions in the environment of the current
shell. Something like this:

```
    $ eval $(aws-as-activate -s)
```

Part of the installed functionality would be an `aws-as-deactivate` function,
which would provide the ability for the above added bits to delete themselves
when no longer needed.

```
    $ aws-as-deactivate
```

The reason for installing shell functions is to provide the ability to
manipulate the current shell environment without requiring that the user
"source" script files.

The idea is that the tool augment `aws-cli` (not operate entirely separately
from it), and to allow that functionality to be leveraged for manipulating the
shell environment (where it can be leveraged by the AWS SDK's and other
compatible tooling). Profiles can and should be leveraged, but only at the
local level; they help us get the environment into a shape that is useful to
other tooling without those other tools having to support or honor or
reference profiles.

Once the user has activated the `aws-as` machinery in a shell, the user's
shell prompt will be augmented to show the "current" IAM User or Role that is
reflected by the `AWS_*` environment variables (or `-`, if none).

From there the user uses the `aws-as` command to both authenticate and switch
between IAM Users and Role (perhaps between different accounts).

Assuming an IAM User, 'my-user1', that needs to authenticate with MFA for API
calls, the following will use `aws-cli` behind the scenes to obtain temporary
credentials. The credentials thus obtained have information in their context
info that asserts that the authentication to AWS STS ("Security Token
Service") was performed using MFA. The credentials are cached (in this case by
`aws-as`), and can be used later to assume roles or perform other actions for
which the 'user1' has access. Notice how the shell prompt changes in these
calls, too:

```
    $ eval $(aws-as-activate -s)
    (-) $

    (-) $ aws-as my-user1-dev
    "Enter MFA code for ${your_mfa_serial_number}: "

    (my-user1-dev) $ aws sts get-caller-identity
    {
        "UserId": "AIDCRAZYLOOKINGSTRING",
        "Account": "111111111111",
        "Arn": "arn:aws:iam::111111111111:user/my-user1"
    }
```

At this point the user can assume other roles without having to authenticate
again (until the short-term credentials obtained above expire):
```
    (my-user1-dev) $ aws-as my-role1-dev

    (my-role1-dev) $ aws sts get-caller-identity
    {
        "UserId": "AROBUNCHOFRANDOMSTUFF:user1@dev-account",
        "Account": "111111111111",
        "Arn": "arn:aws:sts::111111111111:assumed-role/my-role1/my-user1@dev"
    }

    # switch back to 'my-user1' creds (cached, by aws-as)
    (my-role1-dev) $ aws-as my-user1-dev
    (my-user1-dev) $

    # switch back to 'my-role1' creds (cached, by aws-cli)
    (my-user1-dev) $ aws-as my-role1-dev
    (my-role1-dev) $

    # remove aws creds from environment ('-' means pristine state, which is always empty)
    (my-role1-dev) $ aws-as -
    (-) $

    # switch back to 'my-user1' creds (just to show we can)
    (-) $ aws-as my-user1-dev
    (my-user1-dev) $

    # remove all aws-as gunk from the environment
    (my-user1-dev) $ aws-as-deactivate
    $
```

The `aws-as -` call is intended to be analogous to `/bin/su -` on Unix-like
systems, where it means to set the login shell environment similar to a real
login (that is, without most of the cruft from the current environment). Here
we're implying that whatever `AWS_*` environment variables have been set in
the current session by `aws-as` will be unset.

I'm also toying with the idea of a variant of `aws-as-deactivate` that would
allow the user to keep the `AWS_*` credentials in the environment while still
removing all of the `aws-as` gunk. Something like this:

```
    (my-user1-dev) $ aws-as-deactivate --keep-creds
    $
    $ echo "${AWS_SESSION_TOKEN+yep, still there}"
    yep, still there
```


[Aws-cli-gh]:     https://github.com/aws/aws-cli          "GitHub repo: aws/aws-cli"
[aws-cli-site]:   https://aws.amazon.com/cli/             "AWS command line interface"
[aws-sdk-go-gh]:  https://github.com/aws/aws-sdk-go       "GitHub repo: aws/aws-go-sdk"
[terraform-gh]:   https://github.com/hashicorp/terraform  "GitHub repo: hashicorp/terraform"

[tf-prov-aws-gh]:  https://github.com/terraform-providers/terraform-provider-aws  "GitHub repo: terraform-providers/terraform-provider-aws"
[wikipedia-mfa]:   https://en.wikipedia.org/wiki/Multi-factor_authentication      "wikipedia.org: Multi-factor authentication"
