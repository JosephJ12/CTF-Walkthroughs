# Setup Domain Controller on Azure

## Pre Requisites
- Windows 2022 Datacenter VM

## Configuration Steps

1. Log into VM

2. Open Server Manager

<img width="2380" height="1296" alt="image" src="https://github.com/user-attachments/assets/80dd6070-d0cd-42fe-97ef-ed4c4bda7972" />

3. Click on `2. Add Roles and Features`

4. Click `Next` on the popup

5. Select `Role-based or feature-based installation` and click `Next`

6. Select the server we are logged into and click `Next`

7. Check `Active Directory Domain Services` 

<img width="1558" height="1104" alt="image" src="https://github.com/user-attachments/assets/7cd8d99e-9fac-4e52-b3b9-ebb40940540e" />

8. On the popup, click `Add Features` and then `Next`

9. Click `Next` again

10. Click `Next` again on the `Active Directory Domain Services`

11. Click on `Install` on the Confirmation page

<img width="1558" height="1112" alt="image" src="https://github.com/user-attachments/assets/39f67c95-bc8b-43f3-bf53-5caec49493a0" />

12. After the installation completes, click on `Promote this server to a domain controller`.

13. On the following popup page, click on `Add a new forest` and add the domain name

<img width="1514" height="1114" alt="image" src="https://github.com/user-attachments/assets/b151667b-eebc-40a3-8679-4ba1c7c591d7" />

14. Make sure the `Domain Controller Options` page looks like this. Set the DSRM password and click `Next`

<img width="1514" height="1110" alt="image" src="https://github.com/user-attachments/assets/ea917f0f-d8cc-498b-956b-a58e3ce9b0cc" />

15. Ignore the DNS warning and click `Next`

16. Add the NetBIOS name, which should be the same as our root domain name, which is `ICED` in our case

17. Confirm the PATH page is the same as the following and click `Next`

<img width="1510" height="1108" alt="image" src="https://github.com/user-attachments/assets/09ad4d18-9059-4e8a-bcd6-dc694c718ea5" />

18. Click `Next` on the `Review Options` page

19. On the `Prerequisites Check` page, click `Install`

20. Once complete, the VM will automatically restart.

21. Once we're logged back in, confirm our current user has been promoted to Domain Admin using the following command:

`net user icedamericano12 /DOMAIN`

<img width="1372" height="926" alt="image" src="https://github.com/user-attachments/assets/49e00497-6260-4875-a9fa-0748bccf9ea6" />

22. 
