
默认情况下，VMware Cloud Foundation 使用 vCenter Single Sign\-On 作为身份提供程序，并使用系统域作为其身份源，可以将基于 LDAP 和 OpenLDAP 的 Active Directory 添加为 vCenter Single Sign\-On 的身份源，也可以配置 Microsoft ADFS、Okta 或 Microsoft Entra ID 作为 VMware Cloud Foundation 的外部身份提供程序，而不是使用 vCenter Server 内置的 vCenter Single Sign\-On。


[![](https://img2024.cnblogs.com/blog/2313726/202412/2313726-20241207175409925-1606004361.png)](https://github.com)


配置了身份提供程序或添加了身份源后，您可以将用户和组添加到 VMware Cloud Foundation，以向用户提供对 SDDC Manager UI 以及 VMware Cloud Foundation 系统中部署的 vCenter Server 和 NSX Manager 实例的访问权限，用户可以根据其分配的角色（管理员、操作员或查看者）登录并执行不同的任务。


[![](https://img2024.cnblogs.com/blog/2313726/202412/2313726-20241207175337078-1445199265.png)](https://github.com)


注意，SDDC Manager 仅管理“管理工作负载 SSO 域”的用户和组，如果创建的 VI 工作负载域未加入到管理工作负载域的 SSO 域，而是独立的 VI 工作负载 SSO 域，则必须使用 VI 工作负载域 vCenter Server（vSphere Client）来进行管理 SSO 域中的用户和组。


 



## 一、了解组件的账户类型



导航到 SDDC Manager UI\-\>安全\-\>密码管理，这个地方可以查看所有由 SDDC Manager 管理的组件账户，这些账户类型主要包含用户账户（USER）、系统账户（SYSTEM）以及服务账户（SERVICE）。服务账户类型由 VMware Cloud Foundation 自动创建，用于产品组件之间的交互，其他账户类型通常是产品组件自身的本地账户。


[![](https://img2024.cnblogs.com/blog/2313726/202412/2313726-20241207175551746-855957246.png)](https://github.com)


ESXi 组件的账户类型。


[![](https://img2024.cnblogs.com/blog/2313726/202412/2313726-20241207175757718-1633224103.png)](https://github.com)


vCenter Server 组件的账户类型。


[![](https://img2024.cnblogs.com/blog/2313726/202412/2313726-20241207180906007-679522755.png)](https://github.com)


NSX 组件的账户类型。


[![](https://img2024.cnblogs.com/blog/2313726/202412/2313726-20241207175946640-66936669.png)](https://github.com)


SDDC Manager 组件的账户类型。


[![](https://img2024.cnblogs.com/blog/2313726/202412/2313726-20241207185232394-846629266.png)](https://github.com)


 



## 二、检索组件的用户密码



VCF 环境中的组件账户由 SDDC Manager 管理后，其实可以通过 SDDC Manager 来检索这些组件的账户凭据，比如，当你后续对组件的用户密码进行更新或轮换之后，可能将来某些时候想直接访问这些组件进行维护或故障排除时使用。使用 vcf 用户 ssh 连接到 SDDC Manager CLI，通过 lookup\_passwords 命令检索组件的用户密码。


[![](https://img2024.cnblogs.com/blog/2313726/202412/2313726-20241207182716038-1044078807.png)](https://github.com)


可以直接运行命令并根据选择的组件类型，然后获取对应组件用户的密码。


[![](https://img2024.cnblogs.com/blog/2313726/202412/2313726-20241207182949110-1605743833.png)](https://github.com)


运行命令并使用 SSO 管理用户认证直接检索 ESXi 组件的用户密码。



```
lookup_passwords -u administrator@vsphere.local -p Vcf520@password -e ESXI -n 1 -s 20
```

[![](https://img2024.cnblogs.com/blog/2313726/202412/2313726-20241207183230713-1072842767.png)](https://github.com)


运行命令并使用 SSO 管理用户认证直接检索 vCenter Server 组件的根用户密码。



```
lookup_passwords -u administrator@vsphere.local -p Vcf520@password -e VCENTER -n 1 -s 20
```

[![](https://img2024.cnblogs.com/blog/2313726/202412/2313726-20241207183336484-2103978278.png)](https://github.com)


运行命令并使用 SSO 管理用户认证直接检索 vCenter Server 组件的 SSO 用户密码。



```
lookup_passwords -u administrator@vsphere.local -p Vcf520@password -e PSC -n 1 -s 20
```

[![](https://img2024.cnblogs.com/blog/2313726/202412/2313726-20241207183448188-1972837223.png)](https://github.com)


运行命令并使用 SSO 管理用户认证直接检索 NSX 组件的用户密码。



```
lookup_passwords -u administrator@vsphere.local -p Vcf520@password -e NSXT_MANAGER -n 1 -s 20
```

[![](https://img2024.cnblogs.com/blog/2313726/202412/2313726-20241207183545880-331227922.png)](https://github.com)


运行命令并使用 SSO 管理用户认证直接检索 SDDC Manager 组件的备份用户密码。



```
lookup_passwords -u administrator@vsphere.local -p Vcf520@password -e BACKUP -n 1 -s 20
```

[![](https://img2024.cnblogs.com/blog/2313726/202412/2313726-20241207183614699-46671186.png)](https://github.com)


 



## 三、管理组件的用户密码



默认情况下，VCF 环境中组件的用户密码具有有效期，我们可以使用 SoS 实用程序在 SDDC Manager 上运行密码状态检查，如下所示。



```
vcf@vcf-mgmt01-sddc01 [ ~ ]$ sudo /opt/vmware/sddc-support/sos --password-health
Welcome to Supportability and Serviceability(SoS) utility!
Performing SoS operation for vcf-mgmt01 domain components
Health Check : /var/log/vmware/vcf/sddc-support/healthcheck-2024-12-07-11-10-45-128001
Health Check log : /var/log/vmware/vcf/sddc-support/healthcheck-2024-12-07-11-10-45-128001/sos.log
NOTE : The Health check operation was invoked without --skip-known-host-check, additional identity checks will be included for Connectivity Health, Password Health and Certificate Health Checks because of security reasons.

SDDC Manager : vcf-mgmt01-sddc01.mulab.local                                                                                
+-------------------------+-----------+
|          Stage          |   Status  |
+-------------------------+-----------+
|         Bringup         | Completed |
| Management Domain State | Completed |
+-------------------------+-----------+
+--------------------+---------------+
|     Component      |    Identity   |
+--------------------+---------------+
|    SDDC-Manager    | 192.168.32.70 |
| Number of Servers  |       4       |
+--------------------+---------------+
Password Expiry Status : GREEN                                                                                 
+-----+-----------------------------------------+---------------------------+-------------------+--------------+-----------------+-------+
| SL# |                Component                |            User           | Last Changed Date | Expiry Date  | Expires in Days | State |
+-----+-----------------------------------------+---------------------------+-------------------+--------------+-----------------+-------+
|  1  |   ESXI : vcf-mgmt01-esxi01.mulab.local  | svc-vcf-vcf-mgmt01-esxi01 |    Dec 02, 2024   |    Never     |      Never      | GREEN |
|     |                                         |            root           |    Dec 02, 2024   |    Never     |      Never      | GREEN |
|  2  |   ESXI : vcf-mgmt01-esxi02.mulab.local  | svc-vcf-vcf-mgmt01-esxi02 |    Dec 02, 2024   |    Never     |      Never      | GREEN |
|     |                                         |            root           |    Dec 02, 2024   |    Never     |      Never      | GREEN |
|  3  |   ESXI : vcf-mgmt01-esxi03.mulab.local  | svc-vcf-vcf-mgmt01-esxi03 |    Dec 02, 2024   |    Never     |      Never      | GREEN |
|     |                                         |            root           |    Dec 02, 2024   |    Never     |      Never      | GREEN |
|  4  |   ESXI : vcf-mgmt01-esxi04.mulab.local  | svc-vcf-vcf-mgmt01-esxi04 |    Dec 02, 2024   |    Never     |      Never      | GREEN |
|     |                                         |            root           |    Dec 02, 2024   |    Never     |      Never      | GREEN |
|  5  |    NSX : vcf-mgmt01-nsx01.mulab.local   |           admin           |    Sep 26, 2024   | Dec 25, 2024 |     18 days     | GREEN |
|     |                                         |            root           |    Sep 26, 2024   | Dec 25, 2024 |     18 days     | GREEN |
|     |                                         |           audit           |    Sep 26, 2024   | Dec 25, 2024 |     18 days     | GREEN |
|  6  |   SDDC : vcf-mgmt01-sddc01.mulab.local  |            vcf            |    Nov 12, 2024   | Nov 12, 2025 |     340 days    | GREEN |
|     |                                         |            root           |    Nov 12, 2024   | Feb 10, 2025 |     65 days     | GREEN |
|     |                                         |           backup          |    Nov 12, 2024   | Nov 12, 2025 |     340 days    | GREEN |
|  7  | vCenter : vcf-mgmt01-vcsa01.mulab.local |            root           |    Sep 26, 2024   | Dec 25, 2024 |     17 days     | GREEN |
+-----+-----------------------------------------+---------------------------+-------------------+--------------+-----------------+-------+

Legend:

 GREEN - No attention required, health status is NORMAL
 YELLOW - May require attention, health status is WARNING
 RED - Requires immediate attention, health status is CRITICAL


Health Check completed successfully for : [VCF-SUMMARY, PASSWORD-CHECK]                                                                                
vcf@vcf-mgmt01-sddc01 [ ~ ]$ 
```

SDDC Manager 可以管理组件的用户密码，比如更新（Update）密码、轮换（Rotate）密码以及修复（Remediate）密码等。更新密码指的是对某一用户“手动”更新自己设置的密码；轮换密码指的是对一个/多个/全部用户“自动”更新生成随机的密码，这种方式可以设置调度轮换，比如每隔 30/60/90 天自动轮换更新一次组件的密码；修复密码指的是当某一组件由于密码已经过期后，管理员只能手动在组件中去更新密码，然后再通过 SDDC Manager 修复组件的密码以同步更新。


[![](https://img2024.cnblogs.com/blog/2313726/202412/2313726-20241207191505312-332042599.png)](https://github.com)


注意，ESXi 组件的用户密码不支持调度轮换。


[![](https://img2024.cnblogs.com/blog/2313726/202412/2313726-20241207191634427-763316813.png)](https://github.com)


点击“更新密码”，我们可以手动设置一个密码以更新组件的密码。


[![](https://img2024.cnblogs.com/blog/2313726/202412/2313726-20241207192528204-866811959.png)](https://github.com)


[![](https://img2024.cnblogs.com/blog/2313726/202412/2313726-20241207192654851-1766937635.png)](https://github.com):[楚门加速器](https://chuanggeye.com)


完成更新。


[![](https://img2024.cnblogs.com/blog/2313726/202412/2313726-20241207192759932-1005351217.png)](https://github.com)


通过命令可以查看更新后的密码。


[![](https://img2024.cnblogs.com/blog/2313726/202412/2313726-20241207192819426-1071229548.png)](https://github.com)


点击“轮换密码”，系统会自动随机生成一个密码以更新组件的密码。


[![](https://img2024.cnblogs.com/blog/2313726/202412/2313726-20241207192852996-629510909.png)](https://github.com)


[![](https://img2024.cnblogs.com/blog/2313726/202412/2313726-20241207192906163-218870956.png)](https://github.com)


通过命令再次查看更新后的密码。


[![](https://img2024.cnblogs.com/blog/2313726/202412/2313726-20241207192959301-1151388958.png)](https://github.com)


点击“调度轮换”，可以对某一组件的密码设置自动轮换计划。配置为在计划日期的午夜运行，禁用轮换调度，密码轮换将变为手动。


[![](https://img2024.cnblogs.com/blog/2313726/202412/2313726-20241207193045414-199254336.png)](https://github.com)


[![](https://img2024.cnblogs.com/blog/2313726/202412/2313726-20241207193056156-1685308238.png)](https://github.com)


注，当前 VCF 5\.2\.1 版本设置调度轮换后好像 UI 看不到调度设置，之前的版本应该是可以看到的，不过也可以通过其他方式进行查看。


从“密码管理”列表中可能发现，SDDC Manager 组件的用户好像只有 backup 账户，应该还有 vcf 和 root 用户不在列表中。如果需要更新 vcf 和 root 用户的密码，需要手动 ssh 连接到 SDDC Manager CLI 中去，然后使用 passwd 命令进行更新，如下图所示。


[![](https://img2024.cnblogs.com/blog/2313726/202412/2313726-20241207195012871-842323206.png)](https://github.com)


如果 vcf 和 root 用户的密码已经过期了怎么办？可以登录 vCenter Server（vSphere Client），找到 SDDC Manager 虚拟机并打开“WEB 控制台”。


[![](https://img2024.cnblogs.com/blog/2313726/202412/2313726-20241207194756819-1546658615.png)](https://github.com)


然后使用过期的密码进行本地登录，再使用 passwd 命令更新 root 密码以及 vcf 用户密码。


[![](https://img2024.cnblogs.com/blog/2313726/202412/2313726-20241207195145941-187898043.png)](https://github.com)


SDDC Manager 组件除了以上所说的用户以外，其实还有一个本地账户（admin@local），通常使用本地帐户用于访问 VMware Cloud Foundation API，比如当 vCenter Server 关闭/故障，无法使用 SSO 管理员用户（administrator@vsphere.local）时，这时这个本地账户就可以不依赖于 SSO 用户独立执行 API 操作。


本地账户可以通过 API 进行更新密码。导航到 API 资源管理器，输入“admin”以筛选 API 类别，然后使用 PATCH /v1/users/local/admin 选项进行更新本地账户的密码。当执行后状态显示为“204，No Content”，则表示更新成功。


[![](https://img2024.cnblogs.com/blog/2313726/202412/2313726-20241207202704147-785719296.png)](https://github.com)


vCenter Server 组件的默认 SSO 管理员用户具有有效期，所以也需要更新该用户的密码，在 SDDC Manager 中显示为“PSC”资源类型。


[![](https://img2024.cnblogs.com/blog/2313726/202412/2313726-20241207202816768-370968010.png)](https://github.com)


当通过 SDDC Manager 更新 SSO 管理员用户的密码时，提示不允许轮换 PSC 凭据。由于默认只有一个 SSO 管理员用户（administrator@vsphere.local），所以需要创建备用的 SSO 管理员用户之后才能执行更新操作。


[![](https://img2024.cnblogs.com/blog/2313726/202412/2313726-20241207203456225-175337462.png)](https://github.com)


可以通过登录 vCenter Server（vSphere Client）创建一个 SSO 管理员用户（如 vcfadmin@vsphere.local），然后在 SDDC Manager 下图的地方添加另外一个“管理员”角色的用户。


[![](https://img2024.cnblogs.com/blog/2313726/202412/2313726-20241208110526036-1410064684.png)](https://github.com)


SDDC Manger 已添加另外一个 SSO 管理员用户。


[![](https://img2024.cnblogs.com/blog/2313726/202412/2313726-20241208110539941-1977657174.png)](https://github.com)


使用新的 SSO 管理员用户来登录 SDDC Manager，此时，另外一个 SSO 管理员用户的密码可以成功完成轮换。


[![](https://img2024.cnblogs.com/blog/2313726/202412/2313726-20241208110622095-519494601.png)](https://github.com)


如果环境是 VMware Cloud Foundation 5\.2\.1 版本，也可以通过 vCenter Server（vSphere Client）来管理组件的密码。


[![](https://img2024.cnblogs.com/blog/2313726/202412/2313726-20241207202829424-912848309.png)](https://github.com)


使用 SoS 实用程序再次查看组件更新后的密码状态，现在所有组件的密码过期日期已被刷新为最新状态。



```
vcf@vcf-mgmt01-sddc01 [ ~ ]$ sudo /opt/vmware/sddc-support/sos --password-health
[sudo] password for vcf
Welcome to Supportability and Serviceability(SoS) utility!
Performing SoS operation for vcf-mgmt01 domain components
Health Check : /var/log/vmware/vcf/sddc-support/healthcheck-2024-12-07-12-29-31-149728
Health Check log : /var/log/vmware/vcf/sddc-support/healthcheck-2024-12-07-12-29-31-149728/sos.log
NOTE : The Health check operation was invoked without --skip-known-host-check, additional identity checks will be included for Connectivity Health, Password Health and Certificate Health Checks because of security reasons.

SDDC Manager : vcf-mgmt01-sddc01.mulab.local                                                                                
+-------------------------+-----------+
|          Stage          |   Status  |
+-------------------------+-----------+
|         Bringup         | Completed |
| Management Domain State | Completed |
+-------------------------+-----------+
+--------------------+---------------+
|     Component      |    Identity   |
+--------------------+---------------+
|    SDDC-Manager    | 192.168.32.70 |
| Number of Servers  |       4       |
+--------------------+---------------+
Password Expiry Status : GREEN                                                                                 
+-----+-----------------------------------------+---------------------------+-------------------+--------------+-----------------+-------+
| SL# |                Component                |            User           | Last Changed Date | Expiry Date  | Expires in Days | State |
+-----+-----------------------------------------+---------------------------+-------------------+--------------+-----------------+-------+
|  1  |   ESXI : vcf-mgmt01-esxi01.mulab.local  | svc-vcf-vcf-mgmt01-esxi01 |    Dec 02, 2024   |    Never     |      Never      | GREEN |
|     |                                         |            root           |    Dec 02, 2024   |    Never     |      Never      | GREEN |
|  2  |   ESXI : vcf-mgmt01-esxi02.mulab.local  | svc-vcf-vcf-mgmt01-esxi02 |    Dec 02, 2024   |    Never     |      Never      | GREEN |
|     |                                         |            root           |    Dec 02, 2024   |    Never     |      Never      | GREEN |
|  3  |   ESXI : vcf-mgmt01-esxi03.mulab.local  | svc-vcf-vcf-mgmt01-esxi03 |    Dec 02, 2024   |    Never     |      Never      | GREEN |
|     |                                         |            root           |    Dec 02, 2024   |    Never     |      Never      | GREEN |
|  4  |   ESXI : vcf-mgmt01-esxi04.mulab.local  | svc-vcf-vcf-mgmt01-esxi04 |    Dec 02, 2024   |    Never     |      Never      | GREEN |
|     |                                         |            root           |    Dec 02, 2024   |    Never     |      Never      | GREEN |
|  5  |    NSX : vcf-mgmt01-nsx01.mulab.local   |           admin           |    Dec 07, 2024   | Mar 07, 2025 |     90 days     | GREEN |
|     |                                         |            root           |    Dec 07, 2024   | Mar 07, 2025 |     90 days     | GREEN |
|     |                                         |           audit           |    Dec 07, 2024   | Mar 07, 2025 |     90 days     | GREEN |
|  6  |   SDDC : vcf-mgmt01-sddc01.mulab.local  |            vcf            |    Dec 07, 2024   | Dec 07, 2025 |     365 days    | GREEN |
|     |                                         |            root           |    Dec 07, 2024   | Mar 07, 2025 |     90 days     | GREEN |
|     |                                         |           backup          |    Dec 07, 2024   | Dec 07, 2025 |     365 days    | GREEN |
|  7  | vCenter : vcf-mgmt01-vcsa01.mulab.local |            root           |    Dec 07, 2024   | Mar 07, 2025 |     89 days     | GREEN |
+-----+-----------------------------------------+---------------------------+-------------------+--------------+-----------------+-------+

Legend:

 GREEN - No attention required, health status is NORMAL
 YELLOW - May require attention, health status is WARNING
 RED - Requires immediate attention, health status is CRITICAL


Health Check completed successfully for : [VCF-SUMMARY, PASSWORD-CHECK]                                                                                
vcf@vcf-mgmt01-sddc01 [ ~ ]$
```

