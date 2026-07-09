


ANSIBLE RELATED CONFIGURATION

ansible.playbook = \"configure.yml\"
ansible.inventory_path = \"inventories/vagrant/inventory\"
ansible.limit = \"all\" \# Run against all hosts
ansible.extra_vars = {
ansible_user: \'vagrant\',
ansible_ssh_private_key_file: \"\~/.vagrant.d/insecure_private_key\"
}
