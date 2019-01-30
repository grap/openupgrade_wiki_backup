```
""" User settings """
host_old = 'localhost'
port_old = 8059
db_admin_pw_old = '***'
host_new = 'localhost'
port_new = 8069
db_admin_pw_new = '***'
user_pw_old = user_pw_new = 'admin'
dbname_old = 'ou110upgref'
dbname_new = 'ou120upgref'
modules = None  # all
create_old = True
create_new = True

import erppeek
import xmlrpclib


def drop_db_if_exists(db_admin_pw, host, dbname):
    conn = xmlrpclib.ServerProxy(host + '/xmlrpc/db')
    if dbname in conn.list():
        print "Dropping database %s" % dbname
        conn.drop(db_admin_pw, dbname)
        print "Done dropping"


def install_single_module(client, module):
    module_ids = client.search(
        'ir.module.module',
        [('name', '=', module)])
    if not module_ids:
        raise RuntimeError('No module called %s' % module)
    state = client.read('ir.module.module', module_ids, ['state'])[0]['state']
    if state == 'installed':
        print "Upgrading module %s" % module
        client.upgrade(module)
    else:
        print "Installing module %s" % module
        client.install(module)


def prepare_server(db_admin_pw, host, dbname, user_pw):
    client = erppeek.Client(host)
    print "Creating database %s" % dbname
    drop_db_if_exists(db_admin_pw, host, dbname)
    client.create_database(db_admin_pw, dbname, user_password=user_pw)
    install_single_module(client, 'openupgrade_records')
    if not modules:
        print "Installing all modules"
        client.OpenupgradeInstallAllWizard.install_all([])
    else:
        for module in modules:
            install_single_module(client, module)
    print "Generating records"
    client.OpenupgradeGenerateRecordsWizard.generate([])
    return client


def get_or_create_config(client):
    config_ids = client.search(
        'openupgrade.comparison.config',
        [('database', '=', dbname_old)])
    if config_ids:
        return config_ids[0]
    else:
        return client.create(
            'openupgrade.comparison.config',
            {
                'name': dbname_old,
                'server': host_old,
                'port': port_old,
                'database': dbname_old,
                'username': 'admin',
                'password': user_pw_old,
                })

if create_old:
    prepare_server(
        db_admin_pw_old, 'http://%s:%s' % (host_old, port_old),
        dbname_old, user_pw_old)
else:
    print "Reusing existing database for source release"

if create_new:
    client_new = prepare_server(
        db_admin_pw_new, 'http://%s:%s' % (host_new, port_new),
        dbname_new, user_pw_new)
else:
    print "Reusing existing database for target release"
    client_new = erppeek.Client('http://%s:%s' % (host_new, port_new))
    client_new.login('admin', user_pw_new, dbname_new)
    
print "Generating analysis"
wizard_id = client_new.create(
    'openupgrade.analysis.wizard',
    {'server_config': get_or_create_config(client_new)})

client_new.OpenupgradeAnalysisWizard.get_communication([wizard_id])
```
