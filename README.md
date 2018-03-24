# vmware_autoscaling
# This Was my first ansible ptoject and i didn't maintained it since FYI

Hi
See  what I write about the VM Automation
For the auto deploy Centos I use PXE server as my local repo
http://geekface.ca/fedora/?q=pxe



For the vmware part you need to install py https://code.google.com/p/pysphere/  ver 0.1.8
Use pip to install it …. It is MUST also use ansible 1.9 +

script you need to know basic python
my code use the inventory to get the name and ip
In your case you need only the name buy make sure you have in the inventory file something like this
hostxx.FQDN ansible_ssh_host=IP  I am using ansible global var {{inventory_hostname_short}}"
when you create vm you need the name so I am copy it from the inventory files {{inventory_hostname_short}}"

```yaml

---
- name: Create VM
  hosts: servers  # [inventory group ]
  connection: local
  gather_facts: false



  tasks:
    - name: get the current time for a timestamp
      command: date +%Y%m%d-%H%M%S
      register: time
    - name: New Vmware guest vm
      vsphere_guest:
        vcenter_hostname: myvcenter.mydomain.xxx
        username: #Vcenter user
        password: #Vcenter Pass
        guest: "{{inventory_hostname_short}}"
        state: powered_on
        vm_extra_config:
          vcpu.hotadd: yes
          mem.hotadd: yes
          notes: "{{inventory_hostname_short}}"
        vm_disk:
          disk1:
            size_gb: 20
            type: thin
            datastore: mydatastore # name of the datastore i will create the vms
        vm_nic:
          nic1:
            type: vmxnet3
            network: #mynetwork
            network_type: standard
        vm_hardware:
          memory_mb: 2048
          num_cpus: 2
          osid: centos64Guest
          scsi: paravirtual
        vm_hw_version: vmx-09
        esxi:
          datacenter: my-datacenter-name
          hostname: #any ESXi ip address
    - name: Start VM creation and OS install
      debug: var=time.end

    - name: Fire Up new VM 17 min delay
      local_action: wait_for host="{{ansible_ssh_host | default(inventory_hostname) }}" port=22 delay=900 timeout=1200

```

# * NOTE Install vmtools on the host as part of the playbook ONLY in the end rename the host

you need to understand YOU DON’T  connect to remote server your linux box that run ansible play will do it for you this is why  ״connection: local״ in the script

Python rename vmware guest name
```python
#!/bin/python
from pysphere import VIServer, VITask
server = VIServer()
server.connect("vcenter-ip", "myuser@mydom.com", "pass")
vm = server.get_vm_by_name("{{inventory_hostname_short}}")
OShostname = vm.get_property("hostname")
vm.rename(OShostname,sync_run=False)
```

the file is “renamevm.py.j2”  and it is a template so the properties  {{inventory_hostname_short}} will auto generate and run

I create vm from inventory this is the only way so I have the name J

vm = server.get_vm_by_name("{{inventory_hostname_short}}")  ---- > geust vm so I will have properties
now I want the name in the box will be eq to the vmcenter name so in my big playbook I am using the "{{inventory_hostname}}”  (FQDN) and set it as the name of the host and now I
want this name “hostxx.FQDN” well be the name of the guest vm in vcenter
OShostname = vm.get_property("hostname") ---> give ne the name
vm.rename(OShostname,sync_run=False) - will rename the host

