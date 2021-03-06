# Make a new release

## Versioning

Each version has 3 digits: version X.Y.Z:

* X will only change if we change the base build system. For example if we go to CC7 as base, X goes to 2
* Y changes when we change major version of packages inside (be it rpm or python)
* Z changes when we change minor version of packages inside

Each release can done manually or automatically and consists of the following steps:

## Manual execution of steps
### Get list of PRs and update release notes
First of all, get the latest master branch, from which we perform the release. **Careful, you are working with the main repository !**

```
git clone git@github.com:DIRACGrid/DIRACOS.git
cd DIRACOS
```

Now get all the PRs since the last tag. This has to be done using [this script](https://github.com/DIRACGrid/DIRACOS/blob/master/docs/Tools/GetReleaseNotes.py) in the DIRACOS repository by executing
```
   python ./docs/Tools/GetReleaseNotes.py --last
```
Documentation on how to use it can be found [here](https://dirac.readthedocs.io/en/latest/DeveloperGuide/DevelopmentModel/ReleaseProcedure/index.html?highlight=GetReleaseNotes#release-notes).

Once you have added this list to the release.notes, commit and push to the master branch.

```
   git add release.notes
   git commit -m "release.notes update for version XYZ"
   git push origin master
```

### Make a release branch

```
   # Start a new branch
   # Note we wont push the branch, just the tag
   git checkout -b branch_XYZ
```

### Fix the pip requirements versions

The aim of this step is to have a `requirements.txt` with fixed version for every package, such that it is fixed in time. You can of course do it manually...
Otherwise, there is a utility script `dos-fix-pip-versions`. This script will start from a `requirements.txt` file with loose versions, and do the following:

* packages from git are left untouched
* requirements `==` are left untouched
* requirements `<=` are fixed to `==`
* requirements `>=` are fixed to the latest version available
* requirements with no version are fixed to the latest available


For packages taken from git, you will have to fix the version yourself by hand (see bellow).

The `requirements.txt` file in the master branch is (and should remain!) with loose version. The script will start from the `pipRequirements` file in the configuration, and produce an output file. You should then take this output file, fix the git version, and put it in your new branch.


This script requires the same setup as a build of DIRACOS. Thus, you can already setup you environment just like if you were about to build DIRACOS (see [instructions](30_generatingDIRACOS.md)). Once your environment is ready:

```
   [root@c4e49b1711e6 config]# dos-fix-pip-versions /tmp/DIRACOS/config/diracos.json
   [...]
   Fixing the version
   Finish: shell
   Fixed version file in /var/lib/mock/epel-6-x86_64-install/root/tmp/fixed_requirements.txt
```

You can then get the output file (`/var/lib/mock/epel-6-x86_64-install/root/tmp/fixed_requirements.txt`) and put it in replacement of the existing `requirements.txt`, after inspecting it.

NOTE:  You have to execute `dos-fix-pip-versions` after you executed  `dos-build-all-rpms` and you need to copy `fixed_requirements.txt` out of  `/var/lib/mock/epel-6-x86_64-install/root/tmp/` before proceeding with the execution of any other command as it might delete `fixed_requirements.txt`.

Once you are satisfied, you can update the version in the json file, commit, tag and push the branch

```
   # Update the version in the json file
   vim config/diracos.json
   # Note: do not use emacs, it would screw it all up...

   # Commit the changes in the requirements.txt and diracos.json
   git add config
   git commit -m "Fix versions in requirements.txt for version XYZ"
   # Make the release tag
   git tag -a XYZ -m "XYZ"
   # push it all
   git push --tags origin XYZ
```

#### About git packages

At the moment, there are only two packages taken from git:

* fts-rest: only stable versions are tagged, so just use the latest one.
* pyGSI: same applies, use the latest tag

Reminder: to install from a given branch: `git+https://:@gitlab.cern.ch:8443/fts/fts-rest.git@myBranch#egg=fts3` (the egg parameter is the name that the package will take)


### Build DIRACOS

All [here](30_generatingDIRACOS.md).

Just make sure to use the tag you just created

### Deploy the archive

The archive should be placed in `/eos/project/d/diracos/www/releases`

Generate the signature of the archive

```
   md5sum diracos-1.1.1.tar.gz | awk {'print $1'} > diracos-1.1.1.md5
```

Copy the tarball and the md5 file in the release directory

## Automatic generation of a release
The CI has been setup to automatically generate test builds as well as releases
### New Release
If you push a branch with the name `rel-X.Y.Z` the CI will automatically generate a relase `X.Y.Z`, the `rel-*` prefix is the sign to trigger a release.
```
   git clone git@github.com:DIRACGrid/DIRACOS.git
   cd DIRACOS
   git checkout -b rel-X.Y.Z
   git push origin rel-X.Y.Z

```
This will do all the steps desribed in the previous section on manual release. No further intervention necessary.
### Test Build
If you push a branch with the name `test_build` the CI will automatically generate a build with the name  `test_build`, the absence of the `rel-*` prefix is the sign to trigger just a build.
```
   git clone git@github.com:DIRACGrid/DIRACOS.git
   cd DIRACOS
   git checkout -b test_build
   git push origin test_build
```
This will create a build of DIRACOS, and upload the tarball to the DIRACOS webpage. ***No tag will be performed!***
