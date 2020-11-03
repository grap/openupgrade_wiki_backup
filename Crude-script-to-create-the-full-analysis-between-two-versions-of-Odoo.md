```python
import odoorpc

# User settings
host_old = 'localhost'
port_old = 8969
db_admin_pw_old = '****'
host_new = 'localhost'
port_new = 8979
db_admin_pw_new = '****'
user_pw_old = user_pw_new = '****'
dbname_old = 'ou120upgref'
dbname_new = 'ou130upgref'
modules = None  # all
create_old = True
create_new = True


def drop_db_if_exists(db_admin_pw, host, port, dbname):
    client = odoorpc.ODOO(host, 'jsonrpc', port)
    print("Dropping database %s" % dbname)
    client.db.drop(db_admin_pw, dbname)
    print("Done dropping")


def install_single_module(client, module):
    model = client.env['ir.module.module']
    module_ids = model.search([('name', '=', module)])
    if not module_ids:
        raise RuntimeError('No module called %s' % module)
    state = model.read(module_ids, ['state'])[0]['state']
    if state != 'installed':
        print("Installing module %s" % module)
        model.button_immediate_install(module_ids)


def prepare_server(db_admin_pw, host, port, dbname, user_pw):
    print("Creating database %s" % dbname)
    client = odoorpc.ODOO(host, 'jsonrpc', port, timeout=999999)
    drop_db_if_exists(db_admin_pw, host, port, dbname)
    client.db.create(
        db_admin_pw, dbname, demo=False, lang='en_US', admin_password=user_pw)
    client.login(dbname, 'admin', user_pw)
    install_single_module(client, 'openupgrade_records')
    if not modules:
        print("Installing all modules")
        client.env['openupgrade.install.all.wizard'].browse([]).install_all(
            [('name', 'not like', 'l10n_%')])
    else:
        for module in modules:
            install_single_module(client, module)
    print("Generating records")
    client.env['openupgrade.generate.records.wizard'].generate([])
    return client


def get_or_create_config(client):
    config_ids = client.env['openupgrade.comparison.config'].search(
        [('database', '=', dbname_old)])
    if config_ids:
        return config_ids[0]
    else:
        return client.env['openupgrade.comparison.config'].create({
            'name': dbname_old,
            'server': host_old,
            'port': port_old,
            'database': dbname_old,
            'username': 'admin',
            'password': user_pw_old,
        })


if create_old:
    prepare_server(
        db_admin_pw_old, host_old, port_old, dbname_old, user_pw_old)
else:
    print("Reusing existing database for source release")

if create_new:
    client_new = prepare_server(
        db_admin_pw_new, host_new, port_new, dbname_new, user_pw_new)
else:
    print("Reusing existing database for target release")
    client_new = odoorpc.ODOO(host_new, 'jsonrpc', port_new)
    client_new.login(dbname_new, 'admin', user_pw_new)

print("Generating analysis")
wizard_id = client_new.env['openupgrade.analysis.wizard'].create({
    'server_config': get_or_create_config(client_new),
})
client_new.env['openupgrade.analysis.wizard'].get_communication([wizard_id])
```