up:: [[index]]
tags:: #output/blog  #network
dates:: 2023-10-28

# Background

My home internet is provided by Nuro Japan, boasting theoretical speeds of 2G for download and 1G for upload.

The GPON unit in use is the HUAWEI HG8045Q. Unfortunately, the super admin account is not accessible to users and is challenging to bypass. Notably, since the HG8045Q only offers a 1G LAN port, the actual download speed is limited to 1000M, contrary to the 2000M as advertised by Nuro.

To utilize the full speed I've paid for, I decided to implement the ponstick method to circumvent this limitation.
# Environment
## Ponstick:

    ODI 2.5G ONU
    Supports both GPON and EPON

## Network Card:

    BCM57810A
    PCIE 8X with two 10G fiber ports. One of these is set to a 2.5G speed.

## Router OS:

    ikuai
    Note: According to my research, pfsense doesn't support the BCM57810 at 2.5G speed.

# Preparation

Certain information needs to be retrieved from the HG8045Q in advance:

To obtain the ONT status:

    SN
    MAC

For DHCP information:

    VLAN ID (Internet access is 10).

# Setting Up:
## Preparation:

On the ikuai WAN page, set the IP to static and assign an IP like 192.168.1.2/24 with the gateway as 192.168.1.1.
## Ponstick Setup:

Access 192.168.1.1, log in with the default credentials (admin/admin), and modify as follows:
## Settings Page:

The following parameters need adjustments:

    LOID: Retrieved during preparation
    GPONSN: Retrieved during preparation
    Vendor ID: HWTC
    HW Version: V2.0
    Device Serial Number: Retrieved during preparation
    MAC: Retrieved during preparation
    MACKEY: Calculated based on MAC

## VLAN Settings Page:

    Set to Manual, Transparent Mode

# Internet Access:

In the ikuai WAN settings:

    Change the WAN IP from static to VLAN combined mode.
    Create a new VLAN-10 interface and set the MAC identical to the HG8045Q.
    Choose DHCP; you should gain network access afterward.
    Under advanced options, adjust the NIC speed to 2500M.
    Wait a minute, and the download speed should exceed 1000M.


# Result and problem:
As capture,Download speed has broke through 1G,
<img width="299" alt="capture" src="https://github.com/bintis/xirin/assets/57840704/0357012c-26e0-4c88-9423-1bac12e6ebc0">


But I have not figure out how to obtain IPV6,mabe that was IPOE or some other thing.




***

<a rel="license" href="http://creativecommons.org/licenses/by-nc-nd/4.0/"><img alt="クリエイティブ・コモンズ・ライセンス" style="border-width:0" src="https://i.creativecommons.org/l/by-nc-nd/4.0/88x31.png" /></a><br />この 作品 は <a rel="license" href="http://creativecommons.org/licenses/by-nc-nd/4.0/">クリエイティブ・コモンズ 表示 - 非営利 - 改変禁止 4.0 国際 ライセンス</a>の下に提供されています。

@Bintis 著作权，不许抄。
