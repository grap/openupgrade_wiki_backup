# Create the infrastructure for a new Odoo release

Export the old and new versions as variables, e.g. `export PREV=14.0 OLD=15.0 NEW=16.0`.

## Create a new branch

In a clone of https://github.com/OCA/OpenUpgrade, do `git checkout -b $NEW origin/$OLD`. Push the copied branch to the repository.

## Replace version numbers

```
git checkout -b $NEW origin/$OLD
export ESCAPEDPREV=${PREV/\./\\\.}  # e.g. 14.0 -> 14\.0
export ESCAPEDOLD=${OLD/\./\\\.}    # e.g. 15.0 -> 15\.0
sed -i "s/$ESCAPEDOLD/$NEW/g" \
    .github/workflows/documentation-commit.yml \
    .github/workflows/documentation.yml \
    .github/workflows/test.yml \
    .pylintrc \
    .pylintrc-mandatory \
    README.md \
    docsource/conf.py \
    openupgrade_scripts/readme/DESCRIPTION.rst \
    setup/_metapackage/setup.py

sed -i "s/$ESCAPEDPREV/$OLD/g" \
    openupgrade_scripts/readme/DESCRIPTION.rst
    .github/workflows/test.yml
```

# Manual changes

* Reset the version number in `openupgrade_scripts/__manifest__.py` and `openupgrade_framework`
* Set openupgrade_framework to uninstallable in `openupgrade_framework/__manifest__.py`.
* Remove the old version's analysis and migration scripts with `git rm openupgrade_scripts/scripts/* -rf`
* Remove the old version's module mappings in `openuprade_scripts/apriori.py`
* Execute the technical migration of `upgrade_analysis` from https://github.com/OCA/server-tools.
* Run the module migration, see https://github.com/OCA/OpenUpgrade/wiki/Crude-script-to-create-the-full-analysis-between-two-versions-of-Odoo.
  Run with Odoo configuration option `module_coverage_file_folder = <some folder>`.
* On success, propose the migration of `upgrade_analysis` into server-tools, and the analysis files into openupgrade.
  Replace the previous coverage file (e.g. `docsource/modules140-150`) with the file generated in `<some folder>`

# TODO

* Create and push the test database to Github for use in the 'test' workflow job.
* Does the documentation job or Github pages settings need to be reconfigured?
