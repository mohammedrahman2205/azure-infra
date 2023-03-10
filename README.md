# terraform-azurerm-vm #


Deploys 1+ Virtual Machines to your provided VNet
=================================================

This Terraform module deploys Virtual Machines in Azure with the following characteristics:

- Ability to specify a simple string to get the [latest marketplace image](https://docs.microsoft.com/cli/azure/vm/image?view=azure-cli-latest) using `var.vm_os_simple`
- All VMs use [managed disks](https://azure.microsoft.com/services/managed-disks/)
- Network Security Group (NSG) created with a single remote access rule which opens `var.remote_port` port or auto calculated port number if using `var.vm_os_simple` to all nics
- VM nics attached to a single virtual network subnet of your choice (new or existing) via `var.vnet_subnet_id`.
- Control the number of Public IP addresses assigned to VMs via `var.nb_public_ip`. Create and attach one Public IP per VM up to the number of VMs or create NO public IPs via setting `var.nb_public_ip` to `0`.


```hcl
  module "linuxservers" {
    source              = "Azure/vm/azurerm"
    location            = "West US 2"
    vm_os_simple        = "UbuntuServer"
    public_ip_dns       = ["linsimplevmips"] // change to a unique name per datacenter region
    vnet_subnet_id      = "${module.vnet.vnet_subnets[0]}"
  }

  module "windowsservers" {
    source              = "Azure/vm/azurerm"
    location            = "West US 2"
    vm_hostname         = "mywinvm" // line can be removed if only one VM module per resource group
    admin_password      = "ComplxP@ssw0rd!"
    vm_os_simple        = "WindowsServer"
    public_ip_dns       = ["winsimplevmips"] // change to a unique name per datacenter region
    vnet_subnet_id      = "${module.vnet.vnet_subnets[0]}"
  }

  module "vnet" {
    source              = "Azure/vnet/azurerm"
    version             = "~> 1.0.0"
    location            = "West US 2"
    resource_group_name = "terraform-vm"
  }
```

Advanced Usage
-----

1 - New vnet for all vms

2 - Ubuntu 14.04 Server VMs using `vm_os_publisher`, `vm_os_offer` and `vm_os_sku` which is configured with:

- No public IP assigned, so access can only happen through another machine on the vnet.
- Opens up port 22 for SSH access with the default ~/.ssh/id_rsa.pub key
- Boot diagnostics is enabled.
- Additional tags are added to the resource group.
- OS disk is deleted upon deletion of the VM
- Add one 64GB premium managed data disk

2 - Windows Server 2012 R2 VMs using `vm_os_publisher`, `vm_os_offer` and `vm_os_sku` which is configured with:

- Two Public IP addresses (one for each VM)
- Opens up port 3389 for RDP access using the password as shown

```hcl 
  module "linuxservers" {
    source              = "Azure/vm/azurerm"
    resource_group_name = "terraform-advancedvms"
    location            = "westus2"
    vm_hostname         = "mylinuxvm"
    nb_public_ip        = "0"
    remote_port         = "22"
    nb_instances        = "2"
    vm_os_publisher     = "Canonical"
    vm_os_offer         = "UbuntuServer"
    vm_os_sku           = "14.04.2-LTS"
    vnet_subnet_id      = "${module.vnet.vnet_subnets[0]}"
    boot_diagnostics    = "true"
    delete_os_disk_on_termination = "true"
    data_disk           = "true"
    data_disk_size_gb   = "64"
    data_sa_type        = "Premium_LRS"
    tags                = {
                            environment = "dev"
                            costcenter  = "it"
                          }
  }

  module "windowsservers" {
    source              = "Azure/vm/azurerm"
    resource_group_name = "terraform-advancedvms"
    location            = "westus2"
    vm_hostname         = "mywinvm"
    admin_password      = "ComplxP@ssw0rd!"
    public_ip_dns       = ["winterravmip","winterravmip1"]
    nb_public_ip        = "2"
    remote_port         = "3389"
    nb_instances        = "2"
    vm_os_publisher     = "MicrosoftWindowsServer"
    vm_os_offer         = "WindowsServer"
    vm_os_sku           = "2012-R2-Datacenter"
    vm_size             = "Standard_DS2_V2"
    vnet_subnet_id      = "${module.vnet.vnet_subnets[0]}"
  }

  module "vnet" {
    source              = "Azure/vnet/azurerm"
    version             = "~> 1.1.1"
    location            = "westus2"
    allow_rdp_traffic   = "true"
    allow_ssh_traffic   = "true"
    resource_group_name = "terraform-advancedvms"
  }
```

### Docker


#### Prerequisites

- [Docker](https://www.docker.com/community-edition#/download)

#### Custom Image

This builds the custom image:

```sh
$ docker build --build-arg BUILD_ARM_SUBSCRIPTION_ID=$ARM_SUBSCRIPTION_ID --build-arg BUILD_ARM_CLIENT_ID=$ARM_CLIENT_ID --build-arg BUILD_ARM_CLIENT_SECRET=$ARM_CLIENT_SECRET --build-arg BUILD_ARM_TENANT_ID=$ARM_TENANT_ID -t azure-vm .
```

This runs the build and unit tests:

```sh
$ docker run --rm azure-vm /bin/bash -c "bundle install && rake build"
```

This runs the end to end tests:

```sh
$ docker run --rm azure-vm /bin/bash -c "bundle install && rake e2e"
```

