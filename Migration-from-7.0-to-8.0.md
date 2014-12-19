
# Module coverage

1. check this page : [Read The Doc coverage page for v8](https://doc.therp.nl/openupgrade/modules70-80.html)

# Module details
here is a list of modules that disappeared or has been moved between 7.0 and 8.0 version.

| Module in V7        |  Location in V7  | Module in  V8   | Location in V8  | Extra Comment          |
|---------------------|------------------|-----------------|-----------------|------------------------|
|document_ftp         | Official Addons  | document_ftp    | [launchpad](https://code.launchpad.net/~openerp-community/openerp-extra/8.0-document)    |Status unknown   |
|mrp_jit              | Official Addons  | procurement_jit | Official Addons | Handled by OpenUpgrade |
|process              | Official Addons  | No              | No              | No impact to remove it |
|stock_location       | Official Addons  | No              | No              | No longer Needed (included in stock module) |
|stock_no_autopicking | Official Addons  | No              | No              | No longer Needed (reconfiguration of routes needed) |
|web_shortcuts        | Official Addons  | web_shortcuts              | [OCA / Web](https://github.com/OCA/web/tree/8.0)       |                        |

# Database cleanup

After running OpenUpgrade and validating the results, you might want to use https://github.com/OCA/server-tools/tree/8.0/database_cleanup to restore the consistency of your database with respect to modules that have been removed, but also any references to obsolete models, tables and fields. Always keep a copy of the database before you apply this module (and one before you apply the migration), in case you find out later that some aspect of the migration was incomplete and you need access to the original data.