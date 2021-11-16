To run a test migration, Travis downloads a preinstalled database of the previous version from https://github.com/OCA/OpenUpgrade/releases/tag/databases.
This database has all Odoo core modules installed with demo data.

### Preparation

- Checkout a clean Odoo up-to-date base code
- Make sure your addons_path only contains odoo repo, (if it contains OCA / custom repo) the modules will be installed.
- You may need this patch: https://github.com/odoo/odoo/pull/28620

### Generate the database

The 14.0 database was created as follows:
```
# Install account module first to ensure that the chart of accounts of l10n_generic_coa
# is installed instead of the chart of accounts of the first localization module that is encountered.
odoo-bin -d 14.0 -i account --stop-after-init

# Mark all interesting modules as installable
echo "update ir_module_module set state = 'uninstallable' where name like 'test%' or name like 'hw_%';" | psql 14.0
echo "update ir_module_module set state = 'to install', demo=true where state = 'uninstalled';" | psql 14.0

# Install all modules
odoo-bin -d 14.0 -i account_voucher --stop-after-init

# put the attachment in database
odoo-bin -d 14.0 shell
>>> env['ir.config_parameter'].set_param('ir_attachment.location', 'db')
>>> env['ir.attachment'].force_storage()
>>> exit()

# Export database
sudo su postgres -c "pg_dump test_shell --format=c --file=/tmp/14.0.psql"
```

### Upload

Upload the database [here](https://github.com/OCA/OpenUpgrade/releases/tag/databases).

To check if all is ok, you should before migrate the ``openupgrade_scripts`` module (first) and then the ``openupgrade_framework`` module, and check if the CI is OK.