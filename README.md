# Openstack

This is a brief explanation of how to mount a multinode OpenStack environment. The Openstack deployed is called Kolla-Ansible and it works with docker services so that does not depend on the host OS. In this case, Ubuntu 18.04 has been chosen and Pike as OpenStack release.


## First Steps
Before starting the deployment, install:

### Install PIP:
<pre><code>
#locale-gen “en_US.UTF-8”
#vi /etc/environment
LC_ALL=en_US.UTF-8
LANG=en_US.UTF-8
</code></pre>

### Install ansible:
<pre><code>
apt-get install ansible
</code></pre>

### Install docker and add user:
<pre><code>
snap install docker / apt install docker.io
usermod -aG docker root
</code></pre>

### Mount flags for the installation:
<pre><code>
mkdir -p /etc/systemd/system/docker.service.d
tee /etc/systemd/system/docker.service.d/kolla.conf <<-'EOF'
[Service]
MountFlags=shared
EOF
</code></pre>

### Restart docker daemon
It is compulsory to check this steps if docker problems occur while installing.

<pre><code>
#systemctl daemon-reload
#systemctl restart docker
</code></pre>


### Install docker python API
<pre><code>
#pip install -U docker
</code></pre>

### Install kolla
<pre><code>
#pip install kolla-ansible
</code></pre>


### Versions
Python 2.7.25rc1
Pip 10.0.1
Docker: 18.05.0-ce
Pip show docker 3.3.0
Ansible: 2.5.1
Kolla-Ansible: 6.0.0



## Multinode deployment
For this deployment, two machines are used, one as a compute node and other for network node. Once all the packages are installed, it is time to start with the environment´s configuration as follows:

### Choosing docker repo
Change /usr/local/share/kolla-ansible/ansible/roles/baremetal/templates/docker_apt_repo.j2:

<pre><code>
deb [arch=amd64] {{ docker_apt_url }}/linux/{{ ansible_distribution | lower }} {{ ansible_distribution_release | lower }} test
</code></pre>


### Generate passwords.yml
<pre><code>
kolla-genpwd
</code></pre>
If there is any problem, change your execution directory, try in config and out of it. IMPORTANT: Before executing the command, copy the empty passwords.yml from /examples so that kolla-genpwd overwrites that empty file with the new passwords.

### Bootstrap servers with kolla
<pre><code>
kolla-ansible -i multinode bootstrap-servers
</code></pre>

### Pulling docker images from docker hub
<pre><code>
kolla-ansible pull
</code></pre>

### Configuring globals.yml
That configuration should be changed, but it will work as an example:

<pre><code>
#node_custom_config: "/etc/kolla/config"
kolla_base_distro: "ubuntu"
#kolla_install_type: "binary"
openstack_release: "pike"
node_custom_config: "/etc/kolla/config"
kolla_internal_vip_address: "1.2.3.4"
kolla_interface_fqdn: "{{ api_interface_address }}"
kolla_external_vip_address: "{{ kolla_internal_vip_address }}"
network_interface: "eno1"
api_interface: "{{ network_interface }}"
neutron_external_interface: "eno5"
keystone_token_provider: 'uuid'
designate_backend: "bind9"
designate_ns_record: "sample.openstack.org"
tempests_image_id: 
tempest_flavor_ref_id:
tempest_public_network_id:
tempest_floating_network_name:
</code></pre>


### Prechecks.yml
If there is an error in the configuration, this command should say what is wrong:

<code><pre>
kolla-ansible prechecks -i multinode
</code></pre>


### Deploy commmand
<code><pre>
kolla-ansible deploy -i multinode
</code></pre>

### Post-deploy command
<code><pre>
kolla-ansible post-deploy -i multinode
</code></pre>

This creates the openrc files with some useful variables.

### Check admin-openrc.sh
In my deployment, the content is:

<code><pre>
export OS_PROJECT_DOMAIN_NAME=Default
export OS_USER_DOMAIN_NAME=Default
export OS_PROJECT_NAME=admin
export OS_TENANT_NAME=admin
export OS_USERNAME=admin
export OS_PASSWORD=2SnixVWTHsU1qvYjLVmE8s3onNeQP8i0ovPVBqkW
export OS_AUTH_URL=http://1.2.3.4:35357/v3
export OS_INTERFACE=internal
export OS_IDENTITY_API_VERSION=3
export OS_REGION_NAME=RegionOne
export OS_AUTH_PLUGIN=password
</code></pre>

### Test the environment
Finally it is important to check if the deployment is working by running the init-runonce.sh

