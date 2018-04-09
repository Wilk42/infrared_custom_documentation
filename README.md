# infrared_custom_documentation
Infrared Documentation to get openstack running on laptop as a lab 

### Prerequisites ###
sudo dnf install -y gcc libffi-devel openssl-devel python-virtualenv libselinux-python

On Fedora 23 with EPEL repository enabled, RHBZ#1103566 also requires
redhat-rpm-config
sudo dnf install -y gcc libffi-devel openssl-devel python-virtualenv libselinux-python redhat-rpm-config

### Install infrared ###
git clone https://github.com/redhat-openstack/infrared.git
cd infrared
virtualenv .venv && source .venv/bin/activate
pip install --upgrade pip
pip install --upgrade setuptools
pip install .

### Install Infrared plugins  ###
infrared plugin add all

## Make change depending on your memory Availability
in infared/plugins/virsh/defaults/topology/nodes/controller.yml change the Memory designation to something like 10000 not 32000

in infared/plugins/virsh/defaults/topology/nodes/undercloud.yml change the Memory designation to something like 10000 not 16000

Warning this may vary depending on your hardware and if you overcommit, commet in the Overcommit = True below if you want to overcommit memory. This will make your installation run slow.

### Test infrared  ###
infrared virsh --host-address 127.0.0.1 --host-key ~/.ssh/id_rsa --topology-nodes "undercloud:1,controller:1,compute:1"

If you have less than 52 GB of memory used

infrared virsh --host-address 127.0.0.1 --host-key ~/.ssh/id_rsa --topology-nodes "undercloud:1,controller:1,compute:1" ##--host-memory-overcommit True##

or use the file

cat <<EOF >> virsh_prov.ini

[virsh]

host-key = ~/.ssh/id_rsa

host-address = 127.0.0.1

topology-nodes = undercloud:1,controller:1,compute:1

host-user = root

##host-memory-overcommit = True##

EOF

and run

infrared virsh -vv --from-file=virsh_prov.ini |& tee -a virsh_install.txt


### Install Undercloud  ###
infrared tripleo-undercloud -vv --version 10 --images-task rpm |& tee -a undercloud_install.txt


### Install Overcloud  ###
infrared tripleo-overcloud -vvv --deployment-files virt --version 10 --introspect yes --tagging yes --deploy yes --post yes |& tee -a overcloud_install.txt
