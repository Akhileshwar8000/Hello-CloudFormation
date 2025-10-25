# Hello-CloudFormation

# AWS CloudFormation: Auto-Deploying an Nginx Web Server

This project contains an AWS CloudFormation template, `ubuntu-nginx.yaml`, designed to automatically provision a complete, running web server environment.

The template provisions an EC2 instance, installs and configures the `nginx` web server, sets up appropriate security group rules, and deploys a custom `index.html` page using instance user data.

## Features

* **EC2 Instance:** Deploys a `t2.micro` EC2 instance using an **Ubuntu 24.04 LTS** AMI (optimized for `us-east-1`).
* **Web Server:** Automatically installs, enables, and starts the `nginx` web server.
* **Dynamic `index.html`:** Uses `UserData` to create a custom default `index.html` page.
* **Parameterized:** Uses CloudFormation parameters to make the stack customizable:
    * Specify your EC2 Key Pair for SSH access.
    * Specify a trusted CIDR block for SSH (defaults to USF's IP range).
    * Specify a name to display on the web page.
* **Security Groups:**
    * Allows **HTTP** traffic (port 80) from anywhere (`0.0.0.0/0`).
    * Allows **SSH** traffic (port 22) *only* from the specified `SshCidrBlock`.

---

## How to Deploy

### Prerequisites

1.  An **AWS Account**.
2.  An **EC2 Key Pair** already created in the `us-east-1` region. You will need to provide the name of this key pair when launching the stack.

### Deployment Steps

1.  **Navigate to CloudFormation:**
    * Log in to your AWS Management Console.
    * Go to the **CloudFormation** service.
    * Ensure you are in the **N. Virginia (us-east-1)** region.
2.  **Create Stack:**
    * Click **"Create stack"** and select **"With new resources (standard)"**.
    * Under **"Prepare template"**, choose **"Upload a template file"**.
    * Click **"Choose file"** and upload the `ubuntu-nginx.yaml` file from this repository.
    * Click **"Next"**.
3.  **Specify Stack Details:**
    * **Stack name:** Give your stack a unique name (e.g., `MyNginxServer`).
    * **Parameters:**
        * `DisplayName`: Enter the name you want to appear on the webpage (e.g., "Akhileshwar Chauhan").
        * `KeyPairName`: Select your existing EC2 Key Pair from the dropdown.
        * `SshCidrBlock`: Leave the default (`0.0.0.0/0`) or change it to your current IP address (e.g., `x.x.x.x/32`).
    * Click **"Next"**.
4.  **Configure Stack Options:**
    * You can leave all options on this page as their defaults.
    * Click **"Next"**.
5.  **Review and Create:**
    * Review your settings.
    * At the bottom, check the box: **"I acknowledge that AWS CloudFormation might create IAM resources."** (This is good practice, although this template doesn't create IAM roles).
    * Click **"Create stack"**.
6.  **Access Your Web Server:**
    * Wait for the stack's status to change from `CREATE_IN_PROGRESS` to `CREATE_COMPLETE`.
    * Select your stack and go to the **"Outputs"** tab.
    * Click the `WebServerURL` link to see your live `index.html` page in a browser.

---

## Parameters

This template uses the following parameters for customization:

| Parameter | Description | Type | Default |
| :--- | :--- | :--- | :--- |
| `DisplayName` | The name to be displayed in the `<h1>` and `<title>` of the `index.html` page. | `String` | `Dr. Ventura` |
| `KeyPairName` | The name of your existing EC2 Key Pair for SSH access. | `AWS::EC2::KeyPair::KeyName` | - |
| `SshCidrBlock` | The IP address range that can be used to SSH into the EC2 instance. | `String` | `131.247.0.0/16` |

---

## Extra Credit: Instance Metadata

This template successfully implements the extra credit portion.

To avoid a circular dependency (where the `UserData` script needs the Public DNS name of the instance it is *currently creating*), the `UserData` script fetches this information directly from the **EC2 Instance Metadata Service**.

This service is a local API available at `http://169.254.169.254` that the instance can query to learn about itself. The script uses `curl` to fetch the `public-hostname` and `placement/region` and injects them into the `index.html` file.

### `UserData` Logic:

```bash
#!/bin/bash
apt-get update
apt-get install -y nginx

# Fetch Instance Metadata
PUBLIC_DNS=$(curl -s [http://169.254.169.254/latest/meta-data/public-hostname](http://169.254.169.254/latest/meta-data/public-hostname))
AWS_REGION=$(curl -s [http://169.254.169.254/latest/meta-data/placement/region](http://169.254.169.254/latest/meta-data/placement/region))

# Create the custom index.html
cat <<EOF > /var/www/html/index.html
<html>
<head>
    <title>${DisplayName}</title>
</head>
<body>
    <h1>${DisplayName}</h1>
    <p><b>Public DNS:</b> ${PUBLIC_DNS}</p>
    <p><b>AWS Region:</b> ${AWS_REGION}</p>
</body>
</html>
EOF

systemctl enable nginx
systemctl start nginx
