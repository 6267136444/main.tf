resource "azurerm_resource_group" "shubh1" {
  name     = "shubh"
  location = "South India"
}

resource "azurerm_virtual_network" "main" {
  name                = "testnetwork"
  address_space       = ["10.0.0.0/16"]
  location            = azurerm_resource_group.shubh1.location
  resource_group_name = azurerm_resource_group.shubh1.name
}

resource "azurerm_subnet" "internal" {
  name                 = "testinternal"
  resource_group_name  = azurerm_resource_group.shubh1.name
  virtual_network_name = azurerm_virtual_network.main.name
  address_prefixes     = ["10.0.2.0/24"]
}

resource "azurerm_public_ip" "shubh1" {
  name                = "TestPublicIp1"
  resource_group_name = azurerm_resource_group.shubh1.name
  location            = azurerm_resource_group.shubh1.location
  allocation_method   = "Static"  # Changed to Dynamic

  tags = {
    environment = "Production"
  }
}

resource "azurerm_network_interface" "main" {
  name                = "nic"
  location            = azurerm_resource_group.shubh1.location
  resource_group_name = azurerm_resource_group.shubh1.name

  ip_configuration {
    name                          = "testconfiguration1"
    subnet_id                     = azurerm_subnet.internal.id
    private_ip_address_allocation = "Dynamic"
    public_ip_address_id          = azurerm_public_ip.shubh1.id
  }
}

resource "azurerm_network_security_group" "shubh1" {
  name                = "TestSecurityGroup1"
  location            = azurerm_resource_group.shubh1.location
  resource_group_name = azurerm_resource_group.shubh1.name

  security_rule {
    name                       = "test123"
    priority                   = 100
    direction                  = "Inbound"
    access                     = "Allow"
    protocol                   = "Tcp"
    source_port_range          = "*"
    destination_port_range     = "22"
    source_address_prefix      = "*"
    destination_address_prefix = "*"
  }

  tags = {
    environment = "Production"
  }
}

resource "azurerm_network_interface_security_group_association" "shubh1" {
  network_interface_id      = azurerm_network_interface.main.id
  network_security_group_id = azurerm_network_security_group.shubh1.id
}

resource "azurerm_virtual_machine" "main" {
  name                  = "testvm"
  location              = azurerm_resource_group.shubh1.location
  resource_group_name   = azurerm_resource_group.shubh1.name
  network_interface_ids = [azurerm_network_interface.main.id]
  vm_size               = "Standard_DS1_v2"

  storage_image_reference {
    publisher = "Canonical"
    offer     = "ubuntuserver"
    sku       = "18.04-lts"
    version   = "latest"
  }

  storage_os_disk {
    name              = "myosdisk1"
    caching           = "ReadWrite"
    create_option     = "FromImage"
    managed_disk_type = "Standard_LRS"
  }

  os_profile {
    computer_name  = "hostname"
    admin_username = "azureuser"
    admin_password = "Shubh@123"  # Ensure this is a secure password
  }

  os_profile_linux_config {
    disable_password_authentication = false
  }

  tags = {
    environment = "staging"
  }
}
