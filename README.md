usegalaxy.org Playbook
======================

This [Ansible][ansible] playbook is used to deploy and maintain the public
Galaxy servers, namely [Galaxy Main (usegalaxy.org)][main] and [Galaxy Test
(test.galaxyproject.org)][test]. The generalized roles herein have been
published to [Ansible Galaxy][ansiblegalaxy] and can be installed for your own
use via the `ansible-galaxy` command, but a few site-specific roles are
contained here as well.

This playbook is not designed to be used by Galaxy deployers/admins at other
sites, but should be useful as a reference for anyone wishing to emulate a
setup like usegalaxy.org.

### Best Practices ###

We differ slightly from [Ansible best practices][ansiblebestpractices]:

- Because we're trying to ensure that roles are generally consumable and can
  easily be updated from [Ansible Galaxy][ansiblegalaxy] whenever new versions
  are published, our files and templates do not live inside the roles.

[ansible]: http://www.ansible.com/
[galaxyproject]: https://galaxyproject.org/
[ansiblegalaxy]: https://galaxy.ansible.com/
[main]: https://usegalaxy.org/
[test]: https://test.galaxyproject.org/
[ansiblebestpractices]: http://docs.ansible.com/playbooks_best_practices.html

Usage
-----

Galaxy Test is updated with:

    % ansible-playbook -i stage/inventory galaxy.yml --ask-vault-pass

The static content alone (e.g. welcome.html and its dependencies) can be
updated without requiring a vault pass:

    % ansible-playbook -i stage/inventory galaxy_static.yml

Galaxy Main is updated with the same commands as above, replacing `stage` with
`production`.

It's advisable to use a password manager for the vault password. Anything
secure will do, although I find [`pass(1)`][pass] to be incredibly useful in
combination with Ansible Vaults.

If you need to install Ansible and don't want to use the system packages for
your OS (or want the latest version), see `bootstrap-ansible.bash` in the root
directory.

[pass]: http://www.passwordstore.org/

Build Notes
-----------

Building Pulsar's dependencies' dependencies as an unprivileged user on some
HPC systems was a difficult manual process, so I made some notes, which may be
helpful:

Installed cURL by hand on Blacklight:

    cd curl-7.37.0
    ./configure --prefix=/brashear/ndc/test/curl --with-ssl=/brashear/ndc/openssl && make && make install

libffi compiled and installed by hand on Stampede and Blacklight:

    cd /work/galaxy/test/libffi/src/libffi-3.0.13
    ./configure --prefix=/work/galaxy/test/libffi --libdir='${prefix}/lib64' && make && make install
    sed -i -e 's/^Libs:.*/Libs: -L${libdir} -Wl,-rpath,${libdir} -lffi/' ../../lib64/pkgconfig/libffi.pc

OpenSSL compiled and installed by hand on Blacklight (Reference:
https://cryptography.io/en/latest/installation/#building-cryptography-on-linux):

    cd /brashear/ndc/test/openssl/src/openssl-1.0.1h
    cat <<EOF >opnessl.ld
    OPENSSL_1.0.1H_CUSTOM {
        global:
            *;
    };
    EOF
    ./config --prefix=/brashear/ndc/test/openssl -Wl,--version-script=openssl.ld -Wl,-Bsymbolic-functions -fPIC shared && make && make install

slurm-drmaa compiled and installed by hand on Stampede (slurm-devel is not
installed (or worse, some login nodes have mismatched versions), so I had to
work around this):

    cd slurm
    mkdir -p include/slurm
    cd src/slurm-2.6.3
    ./configure --prefix=/usr
    cp slurm/*.h ../../include/slurm
    cd slurm-drmaa-1.0.7
    ./configure --prefix=/work/galaxy/test/slurm-drmaa --with-slurm-inc=/work/galaxy/test/slurm/include && make && make install

pbs-drmaa compiled and installed by hand on Blacklight:

    cd pbs-drmaa-1.0.17
    ./configure --prefix=/brashear/ndc/test/pbs-drmaa && make && make install

Python + virtualenv compiled and installed by hand on Blacklight and Stampede:

    cd /work/galaxy/test/python/src/Python-2.7.6
    ./configure --prefix=/work/galaxy/test/python --enable-unicode=ucs4 && make && make install
    cd ../virtualenv-1.11.5
    /work/galaxy/test/python/bin/python setup.py install

Certs on blacklight are all messed up, so for that, I had to manually assemble
a CA cert chain for pypi.python.org and create ~/.pip/pip.conf to use it.

Dependency Notes
----------------

Currently there's no good way to install dependencies for Pulsar. What I've
done so far is:

1. rsync the `tool_dependency_dir` from the Galaxy server to the Pulsar server.

1. Use `find` and `sed` to alter paths in env.sh

1. Recreate virtualenvs in deps using `{{ instance_root }}/python/bin/virtualenv venv`,
   but this requires removing `include/python2.7`, `lib/python2.7`, and copying
   site-packages from the old venv to the new venv. Obviously not a sustainable model.

License
-------

[Academic Free License ("AFL") v. 3.0][afl]

[afl]: http://opensource.org/licenses/AFL-3.0

Credits
-------

[John Chilton](https://github.com/jmchilton)  
[Nate Coraor](https://github.com/natefoo)

### Inspiration/Thanks ###

[Lance Parsons](https://github.com/lparsons/)  
[Peter van Heusden](https://github.com/pvanheus/)
