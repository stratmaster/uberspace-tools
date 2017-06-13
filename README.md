# uberspace-tools

This script can be used for setting up a customized mailfilter config with **SpamAssassin** and **DSPAM** on an [uberspace](https://uberspace.de) server. Every message will be filtered by these two services. [Custom rules](https://wiki.uberspace.de/mail:maildrop#sonstige_filtereien) can be added too. If messages from known spammers are not correctly recognised as spam by DSPAM, they can be taught as spam by setting up a [blacklist](http://blog.jonaspasche.com/2010/03/23/dspam-automatisch-trainieren/). The main goal of this filter is a more intuitive workflow with less special folders (like `Learn as Spam` and `Learn as Ham`) when showing messages to DSPAM via a customized `dspam-learn` script so it can learn. This is realized by marking messages, recognised or reclassified as spam, with the header `X-Spam-Folder: YES`, which is removed for messages that are reclassified as ham. All reclassified messages will be re-delivered via `maildrop`. Finally, teaching spam or ham works like this:

> * When moving a message *from* 'Junk | Spam | Junk-E-Mail' *to* 'Inbox' it will be learned by DSPAM as Ham
> * When moving a message *from* 'Inbox' *to* 'Junk | Spam | Junk-E-Mail' it will be learned by DSPAM as Spam

*Please Note*: Only messages of the last 24 hours are interpreted by `dspam-learn` to prevent slow scanning due to big folders.

# Installation of Mailfilter
### Core Files
* Set up `.qmail` and `maildrop` before (see uberspace Wiki for setting up [.qmail](https://wiki.uberspace.de/mail:dotqmail) and [Maildrop](https://wiki.uberspace.de/mail:maildrop))
* Rename `.mailfilter` to `.mailfilter-EXT` (replace `EXT` by your namespace) before, if you are using [vmailmgr](https://wiki.uberspace.de/mail:vmailmgr).
* Put the file `.mailfilter` and the folder `.mailfilters` in the home directory of your uberspace.
* Create directory `~/var/log` - it will be used as path for mailfilter logfile
* `dspam-learn` goes to `~/bin` (its logfile and database will later be found in `~/.dspam`)

### Using `dspam-learn` as a service

Run this to add `dspam-learn` as a service:

```
  test -d ~/service || uberspace-setup-svscan
  runwhen-conf ~/etc/run-dspam-learn "$HOME/bin/dspam-learn"
  sed -i -e "s/^RUNWHEN=.*/RUNWHEN=\",M=`awk 'BEGIN { srand(); printf("%d\n",rand()*60) }'`\"/" ~/etc/run-dspam-learn/run
  ln -s ~/etc/run-dspam-learn ~/service/dspam-learn
```

### Cleaning DSPAM database periodically

Run this to add a service for cleaning DSPAM database periodically:

```
  test -d ~/service || uberspace-setup-svscan
  runwhen-conf ~/etc/run-dspam_clean_hashdb "/usr/local/bin/dspam_clean_hashdb"
  sed -i -e "s/^RUNWHEN=.*/RUNWHEN=\",H=`awk 'BEGIN { srand(); printf("%d\n",rand()*24) }'`\"/" ~/etc/run-dspam_clean_hashdb/run
  ln -s ~/etc/run-dspam_clean_hashdb ~/service/dspam_clean_hashdb
```

# Credits and used code (snippets)
* Uberspace [Maildrop](https://wiki.uberspace.de/mail:maildrop) and [DSPAM](https://wiki.uberspace.de/mail:dspam) Tutorials
* [Thorsten Köster](https://blog.macfrog.de/2014/05/10/maildrop-revisited/)
* [Christian González](https://github.com/nerdoc/uberspace-tools)
* [Jonas Pasche](http://blog.jonaspasche.com/2010/03/23/dspam-automatisch-trainieren/)
