# Satellite Capsule Setup Guide

## Part I: Preparation

### System Requirements
- **CPU**
- **Memory**
- **Disk Space**

## Part II: Network Setup

### Properly Configured Network Settings
- Ensure DNS and DHCP settings are correctly configured.
- Consider slow internet speeds and large software repositories when setting up.

## Installation

- Installation time can range from **1 to 5 hours**.
- Configure the Capsule server to communicate with the Satellite server, including content mirroring, DHCP, DNS, and TFTP.

## Steps

### Synchronization
- This can be the most time-consuming part, especially with slow internet or substantial content.
- Note: This could take several hours or a full day.

### Testing and Troubleshooting
- Verification: Test the setup to ensure everything is working as expected.
- Resolve any issues that might arise during deployment, such as configuration errors or network issues.

### Estimated Time Range
- **Fast Deployment:** 1 to 5 hours.
- **Typical Deployment:** 6 to 12 hours.
- **Extended Deployment:** A day or more.

#### Notes:
1. Check if ports are enabled.
2. Pull Repos from Red Hat for Capsule Sync. 
- AppStream RPMs 8
- BaseOS RPMs 8
- Capsule 6.14 RHEL 8 x RPMs
- Maintenance  6.14
- Satellite Utils 6.14 RHEL
  
3. Create a new Content View (CV) for Capsule. ```rhel8-capsule```
4. 4. Use Activation Keys.  ```act-rhel8.9-capsule```
## Part III: Enabling Connections

### From Capsule Server to Satellite Server

On the Satellite Server, open the port for Capsule to Satellite communication:

```bash
firewall-cmd --add-port="5646/tcp"
firewall-cmd --runtime-to-permanent


sudo yum install firewalld
firewall-cmd \
  --add-port="53/udp" --add-port="53/tcp" \
  --add-port="67/udp" \
  --add-port="69/udp" \
  --add-port="443/tcp" \
  --add-port="5647/tcp" \
  --add-port="8140/tcp" \
  --add-port="8443/tcp" \
  --add-port="8000/tcp" --add-port="9090/tcp"

firewall-cmd --runtime-to-permanent

firewall-cmd --list-all


ping -c1 localhost
ping -c1 `hostname -f` # Replace with your system's domain
hostnamectl set-hostname your_hostname


subscription-manager repos --disable "*"
subscription-manager repos \
  --enable=rhel-8-for-x86_64-baseos-rpms \
  --enable=rhel-8-for-x86_64-appstream-rpms \
  --enable=satellite-capsule-6.14-for-rhel-8-x86_64-rpms \
  --enable=satellite-maintenance-6.14-for-rhel-8-x86_64-rpms


dnf module enable satellite-capsule:el8


dnf update
dnf install satellite-capsule


dnf install chrony
systemctl enable --now chronyd


# On Satellite Server
mkdir /root/capsule_cert
capsule-certs-generate \
  --foreman-proxy-fqdn "your_capsule_fqdn" \
  --certs-tar "/root/capsule_cert/your_capsule_fqdn-certs.tar"

capsule-cers-generate \
--foreman-proxy-fqdn cuars2ap.cag.conagrafoods.net
--certs-tar /root/capsule_cert/cuars2ap.cag.conagrafoods.net

# Copy and deploy on Capsule Server
scp /root/capsule_cert/your_capsule_fqdn-certs.tar \
  root@your_capsule_fqdn:/root/


## Part II B: DMZ Location Configuration

### 2.7. Assigning the Correct Organization and Location to Capsule Server

#### A. Log into the Satellite Web UI
- From the **Organization list** in the upper-left of the screen, select **Any Organization**.
- From the **Location list**, select **Any Location**.

#### B. Assign Organization
- Navigate to **Hosts > All Hosts** and select **Capsule Server**.
- From the **Select Actions** list, select **Assign Organization**.
- Choose the organization where you want to assign this Capsule.
- Click **Fix Organization on Mismatch**.
- Click **Submit**.

#### C. Assign Location
- Select **Capsule Server** from the list.
- From the **Select Actions** list, select **Assign Location**.
- Choose the location where you want to assign this Capsule.
- Click **Fix Location on Mismatch**.
- Click **Submit**.

#### D. Verify Organization and Capsule Association
- Navigate to **Administer > Organizations** and click the organization to which you have assigned Capsule.
- Click the **Capsules tab** and ensure that Capsule Server is listed under the **Selected items** list, then click **Submit**.

#### E. Verify Location and Capsule Association
- Navigate to **Administer > Locations** and click the location to which you have assigned Capsule.
- Click the **Capsules tab** and ensure that Capsule Server is listed under the **Selected items** list, then click **Submit**.

#### F. Verification
- Optionally, verify if Capsule Server is correctly listed in the Satellite Web UI by selecting the organization and location from their respective lists.
- Navigate to **Hosts > All Hosts**.
- Navigate to **Infrastructure > Capsules**.

## Part III: Capsule Configuration for Host Registration and Provisioning

### 3.1. Configuring Capsule for Host Registration and Provisioning

#### Procedure
- On Satellite Server, add the Capsule to the list of trusted proxies.
- This is necessary for Satellite to recognize hosts' IP addresses forwarded over the X-Forwarded-For HTTP header set by Capsule.
- For security, Satellite recognizes this header only from localhost by default.

```bash
satellite-installer \
--foreman-trusted-proxies "127.0.0.1/8" \
--foreman-trusted-proxies "::1" \
--foreman-trusted-proxies "10.0.0.0/24" \
--certs-tar-file "/root/10.0.0.57.nip.io-certs.tar" \
--foreman-proxy-register-in-foreman "true" \
--foreman-proxy-foreman-base-url "https://10.0.0.47.nip.io" \
--foreman-proxy-trusted-hosts "10.0.0.47.nip.io" \
--foreman-proxy-trusted-hosts "10.0.0.57.nip.io" \
--foreman-proxy-oauth-consumer-key "CfFDCwDjATNw3YS79rfNxXd5wLr8cqbb" \
--foreman-proxy-oauth-consumer-secret "UzdHRJiMbgE3z7cdWeer3XxX5uVE6fqV"
