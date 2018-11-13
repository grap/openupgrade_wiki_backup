To run a test migration, Travis downloads a preinstalled database of the previous version from https://github.com/OCA/OpenUpgrade/releases/tag/databases. This database has all Odoo core modules installed with demo data.

The 10.0 database was created as follows:
```
odoo-bin -d 10.0 -i account --stop-after-init
echo "update ir_module_module set state = 'uninstallable' where name like 'test%' or name like 'hw_%';" | psql 10.0
echo "update ir_module_module set state = 'to install', demo=true where state = 'uninstalled';" | psql 10.0
odoo-bin -d 10.0 -i account_voucher --stop-after-init
```
This ensures that the chart of accounts of l10n_generic_coa is installed instead of the chart of accounts of the first localization module that is encountered.

For 11.0 and possibly later versions, it is required to create a database with the attachments stored into the database. After creating the initial database, you can open a shell and execute the following commands:

>>> self.env['ir.config_parameter'].set_param('ir_attachment.location', 'db')
>>> self.env['ir.attachment'].force_storage()

You may need this patch: https://github.com/odoo/odoo/pull/28620
