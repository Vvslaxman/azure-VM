To configure a **two-tier architecture** with a **Public Subnet** hosting your IIS web application and a **Private Subnet** hosting your SQL Express database in Microsoft Azure, you need to ensure all settings and properties are accurately configured in the **Azure Portal**. Below, I'll walk you through each and every setting, property, and configuration in Azure to achieve this.

---

### **Step 1: Create a Virtual Network (VNet) and Subnets**

#### **Why**: 
A **Virtual Network (VNet)** in Azure is essential for isolating your resources within a secure, private network. **Subnets** help in logically organizing your resources and securing communication.

#### **Steps in Azure Portal**:
1. **Create a Virtual Network (VNet)**:
   - **Navigate** to **Create a resource** → **Networking** → **Virtual Network**.
   - **Settings/Properties**:
     - **Name**: `MyAppVNet`
     - **Region**: Choose the desired region (e.g., **East US**).
     - **Subscription**: Select your Azure subscription.
     - **Resource Group**: Create a new resource group or select an existing one (e.g., `MyAppRG`).
     - **Address Space**: `10.0.0.0/16` (This provides a large range for subnets).
   
2. **Subnets Configuration**:
   - **Public Subnet**:
     - **Name**: `PublicSubnet`
     - **Address range**: `10.0.1.0/24`
     - **Subnet delegation**: Leave this empty as it's not required for basic VMs.
     - **Network Security Group (NSG)**: You can create a new NSG or select an existing one that allows HTTP (port 80) and HTTPS (port 443).
     - **Route Table**: Leave as `None` (you'll use Azure default routing).
   
   - **Private Subnet**:
     - **Name**: `PrivateSubnet`
     - **Address range**: `10.0.2.0/24`
     - **Network Security Group (NSG)**: Create a new NSG or use an existing one that allows traffic only from the **Public Subnet**.
     - **Route Table**: Leave as `None` (default routing).

3. **Review and Create**: Click **Review + Create** to verify settings, then **Create** the VNet and Subnets.

#### **Key Settings**:
- **VNet Address Space**: Ensure the address space is large enough to accommodate future subnets or resources.
- **Subnet Address Ranges**: Subnets should not overlap.
- **NSG for Public Subnet**: Open **port 80 (HTTP)** and **443 (HTTPS)** for IIS.
- **NSG for Private Subnet**: Ensure only the IIS server in the public subnet can communicate with the database server.

---

### **Step 2: Create Public IP for IIS VM (VM1)**

#### **Why**: 
A **Public IP** is needed for your IIS web server to make it accessible from the internet.

#### **Steps in Azure Portal**:
1. **Navigate** to **Create a resource** → **Networking** → **Public IP Address**.
2. **Settings/Properties**:
   - **Name**: `IISPublicIP`
   - **SKU**: `Basic` (this is sufficient for general use).
   - **IP Version**: `IPv4`
   - **Assignment**: `Static` (ensures the IP address remains the same).
   - **DNS name label**: (Optional) Choose a **DNS name** (e.g., `iisapp.eastus.cloudapp.azure.com`) for easier access.
   - **Region**: Must match the region of your VNet.
   
3. Click **Create** to provision the Public IP.

#### **Best Practices**:
- **Static IP**: Avoid using dynamic IPs, as it can change when the VM is restarted.
- **DNS Name**: Use this feature if you need a human-readable name for the IP.

---

### **Step 3: Create Virtual Machines (VM1 and VM2)**

#### **Why**:
You need two VMs:
- **VM1** for the IIS Web Server (in the Public Subnet).
- **VM2** for the SQL Express Database Server (in the Private Subnet).

#### **Steps to Create VM1 (IIS Web Server)**:
1. **Navigate** to **Create a resource** → **Compute** → **Virtual Machine**.
2. **Settings/Properties**:
   - **Name**: `IISWebServer`
   - **Region**: Select the same region as your VNet (e.g., **East US**).
   - **Image**: Select **Windows Server 2019** or **Windows Server 2022**.
   - **Size**: Choose an appropriate VM size like `Standard_B2s` (2 vCPUs, 4 GB RAM).
   - **Username**: Enter a strong admin username (e.g., `adminuser`).
   - **Password**: Set a strong password.
   - **Authentication type**: `Password`.
   - **Public IP**: Choose `IISPublicIP` (The VM will use the public IP to be accessed from the internet).
   - **VNet**: Choose `MyAppVNet`.
   - **Subnet**: Select `PublicSubnet`.
   - **Network Security Group**: Select or create a new NSG that allows inbound HTTP (port 80) and HTTPS (port 443) traffic.
   
3. Review and click **Create**.

#### **Steps to Create VM2 (SQL Express Server)**:
1. **Navigate** to **Create a resource** → **Compute** → **Virtual Machine**.
2. **Settings/Properties**:
   - **Name**: `SQLExpressServer`
   - **Region**: Same as the VNet region (e.g., **East US**).
   - **Image**: Select **Windows Server 2019** or **Windows Server 2022**.
   - **Size**: Choose a smaller VM size like `Standard_B1s` (1 vCPU, 1 GB RAM).
   - **Username**: Enter a strong admin username.
   - **Password**: Set a strong password.
   - **Authentication type**: `Password`.
   - **Public IP**: Select **None** (No Public IP, as this VM should not be accessible from the internet).
   - **VNet**: Choose `MyAppVNet`.
   - **Subnet**: Select `PrivateSubnet`.
   - **Network Security Group**: Create a new NSG or select an existing one that allows traffic only from the **Public Subnet**.

3. Review and click **Create**.

#### **Key Settings**:
- **VM1**: Public IP, NSG configured for HTTP/HTTPS.
- **VM2**: No Public IP, NSG to allow traffic from Public Subnet only.

---

### **Step 4: Configure IIS on VM1**

#### **Why**:
IIS is required to host your web application on the public-facing server.

#### **Steps to Configure IIS**:
1. **Connect** to VM1 using Remote Desktop (RDP) with the public IP.
2. **Install IIS**:
   - Open **Server Manager** → **Manage** → **Add Roles and Features**.
   - Choose **Web Server (IIS)** and select the required features.
3. **Deploy Your Application**:
   - Copy your web application files (HTML, CSS, JS, etc.) to `C:\inetpub\wwwroot`.
   - Configure the **IIS bindings** to bind to port 80 (HTTP) or 443 (HTTPS).
   - Ensure that the **website** is running.

#### **Best Practices**:
- Always use **HTTPS** for secure communication. Install an SSL certificate in IIS.
- Limit the inbound ports to **80** and **443** on the public-facing VM (through NSG).

---

### **Step 5: Install SQL Express on VM2**

#### **Why**:
SQL Express is used to store and retrieve the data for your web application, and it should reside in the **Private Subnet** for security.

#### **Steps to Install SQL Express**:
1. **Connect** to VM2 via RDP using its private IP.
2. **Install SQL Server Express**:
   - Download and install **SQL Server Express** from Microsoft's site.
   - During installation, ensure that the **SQL Server instance** is configured to listen on port 1433.
   - Enable **Mixed Mode Authentication**.
3. **Configure SQL Express**:
   - In **SQL Server Configuration Manager**, enable **TCP/IP**.
   - Open **SQL Server Management Studio (SSMS)** and create the databases and tables (e.g., `myinsider_db`, `upsi_db`).

#### **Best Practices**:
- **Private Subnet**: SQL Express should not be directly accessible from the internet.
- Use **strong passwords** for SQL authentication.
- Configure **firewall rules** on SQL Server to allow only the IIS VM's private IP.

---

### **Step 6: Configure Communication Between IIS and SQL Express (Two-Tier Setup)**

#### **Why**:
For the application on VM1 (IIS Web Server) to connect to the database on VM2 (SQL Server), proper networking configurations must be done.

#### **Steps to Configure Communication**:
1. **On IIS VM (VM1)**, configure the **connection string** in your application to use the **private IP** of SQL Server (VM2), for example:
   ```sql
   Server=10.0.2.4;Database=myinsider_db;User Id=myuser;Password=mypassword;
   ```
2. Ensure **SQL Server** on VM2 allows connections from VM1's private IP.
3. Test the connection by accessing the web application on VM1 and ensure it can retrieve data from SQL Server.

---

### **Step 7: Additional Considerations**

#### **Security**:
- **NSG**: Configure **Network Security Groups (NSGs)** to ensure only necessary ports are open and limit traffic between the public and private subnets.
- **Encryption**: Ensure data-at-rest and data-in-transit are encrypted using Azure encryption options.

#### **Backup**:
- Set up **Azure Backup** for both VMs and SQL databases for disaster recovery.

#### **Monitoring**:
- Enable **Azure Monitor** and **Log Analytics** to track performance, logs, and diagnose issues.

---

### **Next Task: Creating a New Server and Custom Disk with Managed Keys Encryption**

- Once your infrastructure is up and running, you can proceed to create another server, attach custom encrypted managed disks using **Azure Disk Encryption** (with **Azure Key Vault**).

---

This guide provides a comprehensive overview of setting up a **two-tier architecture** in Azure, covering all the required properties, configurations, and best practices to ensure security, scalability, and efficiency.
