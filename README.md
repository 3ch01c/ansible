# Ansible

## Quick Start

1. Install Ansible.

   ```sh
   brew install ansible
   ```

2. Create a host configuration like [`inventories/production/host_vars/example.yml`](inventories/production/host_vars/example.yml).

   ```yaml
   # inventories/production/host_vars/example.yml
   ---
   ansible_architecture: x86_64
   ansible_default_ipv4.interface: ens1
   ansible_distribution: Ubuntu
   ansible_distribution_release: focal
   ansible_host: 10.0.1.25
   ansible_os_family: Debian
   ansible_python_interpreter: /usr/bin/python3
   ansible_service_mgr: systemd
   ansible_user: ubuntu
   apiserver_advertise_address: 10.0.1.25
   arch: amd64
   env:
     http_proxy: ""
     https_proxy: ""
     no_proxy: ""
   ```

3. Add your hosts to [`inventories/production/hosts.yml`](inventories/production/hosts.yml). In this example, `example` is a `manager` host.

   ```yaml
   # inventories/production/hosts.yml
   ---
   all:
     children:
       kube:
         children:
           managers:
             hosts:
               example:
           workers:
             hosts:
   ```

4. Set up a playbook. [Here's an example](kube.yml) that sets up a Kubernetes cluster.

   ```yaml
   # kube.yml
   ---
   - hosts: managers
     gather_facts: true
     roles:
       - kubelet
       - kubeadm
       - calico
       - kubectl
       - helm
     environment: "{{env}}"
   - hosts: workers
     gather_facts: true
     roles:
       - kubelet
     environment: "{{env}}"
   ```

5. Run the playbook.

   ```sh
   ansible-playbook kube.yml
   ```

## Tweaking Things

You can change the deployment environment. For example, you can create `inventories/development` instead of [`inventories/production`](inventories/production). Just make sure to update [`ansible.cfg`](ansible.cfg) to use that environment instead.

```ini
# ansible.cfg
[defaults]
inventory = inventories/development
```

## Troubleshooting

If you're having issues connecting to your control plane, it could be your firewall. Check if traffic on port 6443 is reaching it.

```sh
sudo tcpdump -i ens5 -v port 6443
```

It might also be an issue if you're behind a web proxy because Kubernetes also uses port 443 for communication.
