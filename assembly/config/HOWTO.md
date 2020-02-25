# How to regenerate or freshen the prepackaged database or cruise-config.xml

## 1. Obtain a test drive installer and unpack it

This is the quickest/easiest way because it's a clean environment and reduces this prep to a couple of steps.

So, either build a test drive installer (if you already have the source installers on disk and you know how to use the `build.sh` and `package.sh` scripts in this repo) or download the latest one (possibly an experimental one if it's built off the target prod installer) from the website.

## 2. Unpack the `packages/cfg.zip` into the root of the test drive installer directory

So, assuming you've unpacked the test driver installer zip to `/tmp/gocd-X.X.X-YYYY-ZZZ`, do the following:

```bash
cd /tmp/gocd-X.X.X-YYYY-ZZZ
unzip packages/cfg.zip
```

This unpacks the `data` directory.

### Make necessary modifications

Modify `cruise-config.xml` or anything else included here.

#### If you're regenerating the prepackaged DB (e.g., for regenerating pipeline history):

Then `rm -rf data/server/db` at this point. Upon starting the server, GoCD will create a new one as the agent performs runs.

### Start the server once finished with modifications

`./run-gocd`

### Stop server after DB is no longer being updated or is in the desired state

### Copy the configs back to this directory

Don't copy _everything_ -- just what is necessary (which is usually just the `server/config/cruise-config.xml` and `server/db/h2db/cruise.h2.db`). You may or may not need to copy `cruise-config.xml` at all unless you've changed it.

### Clean `cruise-config.xml`

A running test drive instance will populate `cruise-config.xml` with `<agents/>` configs, `agentAutoRegisterKey`, and `serverId`. We don't want this as this should be generated by the user's instance on first run, so take these out.

### Clean/depersonalize DB

Generally, just `rake` from this directory, and the rake tasks will clean the DB of ip addresses and such. If you've introduced a change that will write other information that should be removed from the DB, then please add the SQL to the `sql-scripts` directory here and modify the `Rakefile` to run this cleanup SQL.

### Build and package a fresh installer and test your changes

Assuming you have the production server and agent zip installers in `REPO_ROOT/deps/zip`:

```bash
./build.sh --prod
assembly/package.sh osx # can pass one or more of: osx, windows, linux
```

The above snippet will create an installer under `REPO_ROOT/installers`. Unpack and run to verify your changes.