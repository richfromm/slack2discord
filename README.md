# slack2discord

A Discord client that imports Slack-exported JSON chat history to
Discord channel(s).

## tl;dr

See [Script invocation](#script-invocation) below.

## History

This started out as
[thomasloupe/Slackord](https://github.com/thomasloupe/Slackord). I
made some contributions (see
[#7](https://github.com/thomasloupe/Slackord/pull/7) and
[#9](https://github.com/thomasloupe/Slackord/pull/9)) to add support
for threads. But then my list of additional proposed
[changes](https://github.com/thomasloupe/Slackord/issues/8) was
significant enough, that we mutually decided that me continuing
development on a hard fork would be better.

Note that there also exists a .NET version
[thomasloupe/Slackord2](https://github.com/thomasloupe/Slackord2) by the
original author, that contains additional functionality and appears to be more
actively maintained than the upstream fork from which this Python project
originates.

## Prereqs

### virtualenv

Install `discord.py` ([pypi](https://pypi.org/project/discord.py/),
[homepage](https://github.com/Rapptz/discord.py),
[docs](https://discordpy.readthedocs.io/en/latest/)) (_yes, there
really is a `.py` included in the package name_) and `decorator`
([pypi](https://pypi.org/project/decorator/),
[homepage](https://github.com/micheles/decorator),
[docs](https://github.com/micheles/decorator/blob/master/docs/documentation.md))
packages into a Python virtualenv:

    pip install discord.py decorator

For help creating virtual environments, see the
[venv](https://docs.python.org/3/library/venv.html) docs. If you use Python a
lot, you may also want to consider
[virtualenvwrapper](https://virtualenvwrapper.readthedocs.io/en/latest/). If you
don't want to think much about virtual envs and just want simple Python scripts
to work, you could consider [pyv](https://github.com/richfromm/pyv).

### py3

This assumes Python 3.0. Python 2.x was EOL'd at the beginning of
2020, and no new project should be using it.

## Usage

### Slack export

To export your Slack data as JSON files, see the article [Export your
workspace
data](https://slack.com/help/articles/201658943-Export-your-workspace-data).

Note that only workspace owners/admins and org owners/admins can use this
feature. While this is available on all plans, only public channels are included
if you have the Free or Pro version. You need a Business+ or Enterprise Grid
plan to export private channels and direct messages (DMs).

I also think (but am not 100% positive) that the export has the same 90 day
limit of history imposed on Free plans as of 1 September 2022. (This change was
what motivated me to migrate from Slack to Discord and work on this tool.)

If it is a large export, this might take a while. Slack will notify you when it
is done.

The export is in the form of a zip file. Download it, and unzip it.

The contents include dirs at the top level, one for each channel. Within each
dir is one or more JSON files, of the form _`YYYY-MM-DD.json`_, one for each day
in which there are messages for that channel. Like so:

```
channel1
 |- 2022-01-01.json
 |- 2022-01-02.json
 |- ...
channel2
 |- 2022-01-01.json
 |- 2022-01-02.json
 |- ...
...
```

There is additionally some metadata contained in JSON files at the top level,
but they are not (currently) used by this script, and are not shown above.

### Discord import

#### One time setup

For complete instructions, see <https://discordpy.readthedocs.io/en/stable/discord.html>

1. Login to the Discord website and go to the Applications page:

   <https://discordapp.com/developers/applications/>

1. Create a new application. See the implications below (in creating a
   bot) of choosing an application name that contains the phrase
   "discord".

   New Application -> Name -> Create

1. Optionally enter a description. For example:

   Applications -> Settings -> General Information -> Description:

   > A Discord client that imports Slack-exported JSON chat history to Discord
   channel(s).

   Save Changes

1. Create a bot

   Applications -> Settings -> Bot -> Build-A-Bot -> Add Bot -> Yes, do it!

1. Unfortunately, if you named your App "slack2discord", Discord
   disallows the use of the phrase "discord" within a username. So
   the Username prefix (before the number) will default to
   "slack2". If you don't like that, you can choose something else:

   Applications -> Settings -> Bot -> Build-A-Bot -> Username

   like "slackimporter".

1. Create a token:

   Applications -> Settings -> Bot -> Build-A-Bot -> Token -> Reset Token -> Yes, do it!

   **Copy the token now, as it will not be shown again.** This is used below.

   (Don't worry if you mess this up, you can always just repeat this step to
   create a new token.)

1. Invite the bot to your Discord server:

   Applications -> Settings -> OAuth2 -> URL Generator -> Scopes: check "bot"

   Bot permissions -> Text permissions:

   check: "Send Messages", "Create Public Threads", "Create Private Threads"

   This will create a URL that you can use to add the bot to your server.

    1. Go to Generated Url
    1. Copy the URL
    1. Paste into your browser
    1. Login if requested
    1. Select your Discord server, and authorize the external application to
       access your Discord account.
    1. -> Continue -> Authorize
    1. Do the Captcha if requested
    1. Close the browser tab

#### Discord token

The Discord token (created for your bot above) must be specified in one of the
following manners. This is the order that is searched:

1. On the command line with `--token TOKEN`
1. Via the `DISCORD_TOKEN` env var
1. Via a `.discord_token` file placed in the same dir as the
   `slack2discord.py` script.

#### Script invocation

Briefly, the script is executed via:

    ./slack2discord.py [--token TOKEN] [-v | --verbose] [-n | --dry-run] <src-and-dest-related-options>

The src and dest related options can be specified in one of three different
ways:

* `--src-file SRC_FILE --dest-channel DEST_CHANNEL`

    This is for importing a single file from a Slack export, that
    corresponds to a single day of a single channel.

* `--src-dir SRC_DIR [--dest-channel DEST_CHANNEL]`

    This is for importing all of the days from a single channel in a
    Slack export, one file per day.

* `--src-dirtree SRC_DIRTREE [--channel-file CHANNEL_FILE]`

    This is for importing all of the days from multiple (potentially
    all) channels in a Slack export. One dir per channel, and within
    each channel dir, one file per day.

For more details, and complete descriptions of all command line
options, execute:

    ./slack2discord.py --help

## Future work

Some items I am considering:

* Optionally automatically create destination channels in Discord if
  they do not already exist.

* Minimally transform non-standard Slack Markdown as needed to conform
  to Discord Markdown.

* Deal with external links in Discord, showing the same preview title,
  heading, text, and image that Slack does.

* Better error reporting, so that if an entire export is not
  successful, it is easier to resume in a way as to avoid duplicates.

* Add mypy type hints.

Feel free to open issues in GitHub if there are any other features you
would like to see.
