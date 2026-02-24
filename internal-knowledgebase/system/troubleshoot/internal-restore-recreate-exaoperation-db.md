---
tool_name: cos
doc_type: troubleshoot
category: system
title: "Internal - Restore/Recreate EXAoperation DB"
summary: "While using EXAoperation, the user will be log-outed every time he clicks anywhere using the menu. After all there is no possibility to gather some logs than using the shell. To..."
---
# Internal - Restore/Recreate EXAoperation DB

## Description

While using EXAoperation, the user will be log-outed every time he clicks anywhere using the menu.
After all there is no possibility to gather some logs than using the shell. To solve that problem ask the customer when EXAoperation was working the last time and restore a EXAoperation-Backup.

## How to restore the backup

1. remove appserverd
```
cosrm -a <appserverd_id>
```

2. search for the version (date), where the problem doesn't exist at
/usr/opt/EXASuite-\<VERSION>/EXAClusterOS-\<VERSION>/var/.saved_metadata/
3. exctract it to a temporary directory
```
tar -xf /usr/opt/EXASuite-<VERSION>/EXAClusterOS-<VERSION>/var/.saved_metadata/METADATE<DATE> -C <temp_dir>
```

4. create a backup from directory "dirstorage" at "/usr/opt/EXASuite-\<VERSION>/EXAClusterOS-\<VERSION>.X/var/exaoperation/inst/var/dirstorage"
```
mv /usr/opt/EXASuite-<VERSION>/EXAClusterOS-<VERSION>/var/exaoperation/inst/var/dirstorage /root/dirstorage_old
```

5. copy directory "dirstorage" from the extracted directory to "/usr/opt/EXASuite-\<VERSION>/EXAClusterOS-\<VERSION>/var/exaoperation/inst/var/"
```
mv <temp_dir>/usr/opt/EXASuite-<VERSION>/EXAClusterOS-<VERSION>/var/exaoperation/inst/var/dirstorage /usr/opt/EXASuite-<VERSION>/EXAClusterOS-<VERSION>/var/exaoperation/inst/var/
```

6. start appserverd
```
cosexec --single-instance --auto-restart --auto-add $COS_DIRECTORY/libexec/appserverd
```

## Recreate EXAoperation DB with dbcopy.py
1. create dbcopy.py (see below)
1. copy dbcopy.py to '/usr/opt/EXASuite-\<VERSION>/EXAClusterOS-\<VERSION>/var/exaoperation/inst/var/' on MGMT node
2. execute the following on MGMT node:
```
cosrm -a `cosps | awk '/appserverd/{ print $1; }'`
cd /usr/opt/EXASuite-*/EXAClusterOS-*/var/exaoperation/inst/var
sed 's|/var/dirstorage|/var/test1|g; s|/log/zope.log|/inst/var/test1.log|g' ../etc/zope-deploy.conf > test1.conf
python2.7 dbcopy.py --source=../etc/zope-deploy.conf --destination=test1.conf >test1.py
python2.7 /usr/opt/EXASuite-*/EXARuntime-*/lib/python2.7/site-packages/DirectoryStorage/mkds.py  test1 Full bushy
python2.7 test1.py
mv dirstorage dirstorage.old
mv test1 dirstorage
cosexec --single-instance --auto-restart --auto-add -- $COS_DIRECTORY/libexec/appserverd
```

## Create dbcopy.py
```
#!/usr/bin/env python

import sys, os
import zope.app.wsgi

from getopt import getopt, GetoptError
from zope.app.debug.debug import Debugger
from zope.app.container.interfaces import IContained, IContainer
from zope.securitypolicy.interfaces import IPrincipalRoleManager
from subprocess import check_output as sh

COS_DIRECTORY = sh(['/bin/bash', '-c', '. /etc/cos.conf; echo -n $COS_DIRECTORY'])
if not os.path.exists(COS_DIRECTORY):
    sys.stderr.write('COS_DIRECTORY (%s) not found.\n' % repr(COS_DIRECTORY))
    sys.exit(1)

sys.path.append('%s/var/exaoperation/inst/src' % COS_DIRECTORY)
sys.path.append('%s/lib' % COS_DIRECTORY)

import exaoperation

try:
    opts, args = getopt(sys.argv[1:], "h", ['source=','destination='])
    opts = dict(opts)
    if '--source' not in opts \
       or '--destination' not in opts \
       or '-h' in opts:
        raise GetoptError("Wrong arguments")
    source = opts['--source']
    destination = os.path.abspath(opts['--destination'])
except GetoptError:
    sys.stderr.write("Usage: %s --source=<config> --destination=<config>\n\n" % sys.argv[0])
    MKDS_PY = sh(['/bin/bash', '-c', 'ls /usr/opt/EXASuite-*/EXARuntime-*/lib/python2.7/site-packages/DirectoryStorage*/DirectoryStorage/mkds.py | tail -n1']).strip()
    sys.stderr.write("$> %s <dir> Full bushy\n" % MKDS_PY)
    sys.exit(1)

if not os.path.exists(destination):
    sys.stderr.write('Configuration %s does not exist!\n' % repr(destination))
    sys.exit(1)
src = Debugger.fromDatabase(zope.app.wsgi.config(source))

def getNames(o):
    names = set()
    def reget(i):
        for n in i.names():
            if n.startswith('_'): continue
            if type(i.get(n)) not in (zope.interface.interface.Method,):
                names.add(n)
        for x in i.getBases(): reget(x)
    for i in o.__provides__.interfaces(): reget(i)
    return names

def traverse(root, prefix = ""):
    for k, o in sorted(root.items()):
        print "if %s not in %s:" % (repr(k), prefix),
        if o.__class__ == zope.app.authentication.principalfolder.InternalPrincipal:
            print "%s[%s] = %s.%s(%s, %s, %s, %s, %s)" \
               % (prefix, repr(k), o.__class__.__module__, o.__class__.__name__,
                  repr(o.login),
                  repr(o.password), repr(o.title), repr(o.description),
                  repr(o.passwordManagerName))
        else: print "%s[%s] = %s.%s()" % (prefix, repr(k), o.__class__.__module__, o.__class__.__name__)
        if IContained.providedBy(o):
            for n in sorted(getNames(o)):
                if hasattr(o, n):
                    val = repr(getattr(o, n))
                    if val.startswith('<'):
                        val = getattr(o, n)
                        val = "%s.%s" % (val.__module__, val.__name__)
                    if val.startswith("u'") and n == 'cpu_scaling_governor':
                        val = val[1:]
                    if o.__class__ == zope.app.authentication.principalfolder.InternalPrincipal and n == 'password':
                        nn = '_password'
                    else: nn = n
                    # explicitely set attributes, which are checked like this in management.py:
                    # >>> if not root[u'cluster1'].__dict__.get('load_error', None)
                    # correct way would be:
                    # >>> if not (hasattr(root[u'cluster1'], 'load_error') and getattr(root[u'cluster1'], 'load_error'))
                    if n not in ('node_ipmi_type', 'disk_usage_warning', 'disk_usage_error', 'load_warning',
                                 'load_error', 'swap_warning', 'ntp_server1', 'ntp_server1', 'ntp_server2',
                                 'ntp_server3', 'exaop_nodes', 'swap_error', 'backup_bandwidth',
                                 'coredump_deletion_time', 'software_installation_history',
                                 'license_default_parameters', 'default_data_size'):
                        print "if not hasattr(%s[%s], %s) or repr(getattr(%s[%s], %s)) != repr(%s):" % (prefix, repr(k), repr(n), prefix, repr(k), repr(n), val),
                    print "%s[%s].%s = %s" % (prefix, repr(k), nn, val)
        if IContainer.providedBy(o) \
           and not exaoperation.interfaces.IStorage.providedBy(o):
            traverse(o, prefix = "%s[%s]" % (prefix, repr(k)))
        try: rm = IPrincipalRoleManager(o)
        except: rm = None
        if rm is not None:
            for r, u, p in rm.getPrincipalsAndRoles():
                print "IPrincipalRoleManager(%s[%s]).assignRoleToPrincipal(%s, %s)" % (prefix, repr(k), repr(r), repr(u))

print "#!/usr/bin/env python"
print "import sys"
print "sys.path.append(%s)" % repr('%s/var/exaoperation/inst/src' % COS_DIRECTORY)
print "sys.path.append(%s)" % repr('%s/lib' % COS_DIRECTORY)
print "import exaoperation"
print "import zope.app.wsgi"
print "import zope.app.security.interfaces"
print "import transaction"
print "from zope.app.debug.debug import Debugger"
print "from zope.securitypolicy.interfaces import IPrincipalRoleManager"
print "db = Debugger.fromDatabase(zope.app.wsgi.config(%s))" % repr(destination)
print "root = db.root(); sm = zope.component.getSiteManager(db.root())"
root = src.root()
sm = zope.component.getSiteManager(root)
traverse(sm, prefix = 'sm')
traverse(root, prefix = 'root')
try: rm = IPrincipalRoleManager(root)
except: rm = None
if rm is not None:
    for r, u, p in rm.getPrincipalsAndRoles():
        print "IPrincipalRoleManager(root).assignRoleToPrincipal(%s, %s)" % (repr(r), repr(u))
print "sm.registerUtility(sm[u'default'][u'pau'], zope.app.security.interfaces.IAuthentication, '')"
print "transaction.commit()"
print "db.db.close()"
src.db.close()
```
