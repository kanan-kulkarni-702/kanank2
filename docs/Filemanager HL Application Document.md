# File Manager Application

   ## 1. Overview 
File Manager Application is an application similar to Dropbox or Box and the  basic aim of the application is to manage user files and  
folders by providing fault detection against deliberate or accidental tampering of the sensitive data.  
The actions that a user can perform are handling files and folders using operations such as create, copy, move, delete, rename, upload, update,  
share and modify share rights. The file or folder stored in the system can be fault detectable or not fault detectable. This selection is done   
at the time of file or folder creation. Thus, in this application, the data is handled both in Scalar DB and Scalar DL.  
The application uses ScalarDB for storing all the data (files, users, groups, access control data etc)  and Scalar DL to manage the fault   
detectable data i.e. sensitive files. 
If a file is selected as fault detectable(hereinafter referred as a BFD file) , the file is managed in ScalarDL along with scalarDB. All the   
important operations performed on a file are recorded in Scalar DL. Thus, the account history is recorded in a tamper-evident way.. This means  
that if an account history was altered (either intentionally or not), it is possible to detect this.  

Primarily, the application aims to incorporate following features  
 a. Registering data from Web Application  
 b. Detect data tampering  
 c. Create faults in the file beforehand and restrict operations on such faulty files.  

The File Manager Demo application also provides functionalities such as User Management and Byzantine Fault Management. The User management  
functionality is provided to create and delete  the system users and assign a role while creating the user. The Byzantine Fault Management  
Tool is implemented to inject Byzantine Fault  in the file and recover the fault.  
## 2. Functional Block diagram of the File Manager Application
The various functionalities in the application are listed below.
* Authorization and User role management    
   The user registration and sign in operation. User creation and deletion and assigning a role to the user.  
   There are three roles : General user, Auditor and Administrator.  
   Some specific BFD files related operations can be performed by the auditor only.   
* Group Management     
   Create and delete a group. Add and delete users from the group.  
* File and folder management     
    The folder operations such as create, copy, move, delete, rename,hide are available.The file operations such as upload, copy, move, delete, 
    rename, update, hide are available.  
* File and folder sharing    
    Files and folders can be shared to other users with sharing rights as editor and viewer. The sharing can be done at group level as well.
* Byzantine fault management (Fault injection and recovery tool)   
    This is a tool available for Administrator on the dashboard and is implemented to inject selected Byzantine fault in a BFD file. This fault is 
    deliberately created by this tool and the Administrator can recover the already created faults from the List displayed.    
* User Management (user creation and deletion)    
    An Administrator can create a user and assign a role as well as delete a user using this menu option from the dashboard.
* Mutable data storage management (ScalarDB)     
    The mutable data is the data which is not considered as sensitive to alterations. The files which are not created (uploaded) as BFD files, the user 
   related data, the sharing access related data, the group data etc are handled in ScalarDB. In short, all the data of the application is managed 
   in ScalarDB.    
* Immutable data storage management (ScalarDB and ScalarDL)      
   The files which are uploaded as BFD files are added in ScalarDL and are considered as the Assets. The file operations such as copy, move, delete, 
   rename and update are updated in ScalarDL Ledger as well as Auditor. 
   This allows the data tampering to be identified. 
   The below diagram represents the interaction and relationship between these functionalities.

![architect](/docs/assets/images/File_manager_HL_documentation/Overall_architecture1.jpg)

## 3. User Stories
There are three user roles while logging in to the FM App. Based on the user role, the following user stories are defined.  
| Sr no.| Use case| Description|
|-----|----|----|
|1    |Login to the system| A user having valid credentials as a general user can login to the system using  username and  password. The user can then view the dashboard and is able to navigate the file system owned by him or shared to him and perform the file or folder related operations. In case a user has forgotten the password, the user can use ‘ forgot password’  functionality. The user should enter an OTP received  on the registered email. Users can then set a new password.|
|2    |Register a user|Any general user can self register to the File Management  system by entering a Name, valid email and a password. Then the user has to enter OTP received by email. <br /> An Auditor type user cannot register. <br /> An Admin type user cannot register. |
|3    |Create a folder|Users can create a folder. While creating a folder, users can select if  the folder will be  Byzantine Fault Detectable.  If selected as BFD, then the folder is considered as a BFD folder by the system. The BFD folder will contain files/folders whose tampering can be detected. User logged in with a general user role, can create a folder in his root folder. <br /> A user logged in as an Auditor, can not create a folder.<br /> A user logged with an Admin role, can create folders in root and also in other user’s folders.| 
|4    |Upload a file (new file) | After login, the user can upload a new file  (file size < 10 MB) to any folder. The upload operation can be allowed or not based on the user role. The file can be uploaded to a non-BFD folder or to a  BFD folder. File length must be limited to say 64 characters. Non English characters must be allowed.<br /> An user who has viewer access to a folder cannot upload a new file in that folder.<br />An Auditor type user cannot upload a file.<br /> An Admin type user can upload a file.|
|5    |View the file details | Users (General/Auditor/Admin) are expected to see the following details such as.<br /> 1. File Preview <br />2. File owner (email id of file owner) <br />3. Updated by (can be owner of file or other user with editor permissions)<br />4. Created at <br />5. Modified at <br />6. Size <br />7. File Hash (SHA256) <br />8. Version History <br />In case a version of a file is tampered for a BFD type file, then a cross symbol should be seen in front of it. Otherwise any version can be downloaded.|
|6    |Update a file      | A user can select a file and update the file (new version) by either mouse right click or from the right side panel. <br />A user can update with a different filename. Only users who will have  ownership/editor permissions to the file should be able to update it.<br />Users having ‘Viewer’ rights cannot update a file.<br />An Auditor type user cannot update a file.|
|7    |Copy a file       |Users can select a file and copy the file by either mouse right click option or from the menu on panel. A popup box is expected to open to select a destination folder to copy the file.<br />File can be copied to the destination folder where the user is the owner or the user has editor rights. File copying is applicable for regular files and also for BFD files. In case of regular file copying, the source and destination folder must be non BFD type. In case of a BFD file copy, the destination folder must also be BFD type.<br />A user having ‘Viewer’ right for a file cannot copy the file to another folder.<br />An Auditor type user cannot copy a file.|
|8    |Move a file      |Users can select a file and move the file by either selecting ‘Move’ from mouse right click or from the right side panel of ‘Move’ button. A popup box should open to select a destination folder to move the file.<br />File (owner/editor permissions) can be moved to the destination folder where the user is the owner or the user has editor rights to the destination folder. File moving is applicable for regular files and also for BFD files. In case of a BFD file move, the destination folder must also be BFD type. For non BFD  file move, the source and destination folder must be non BFD type.<br />An user having ‘Viewer’ rights cannot move a file.<br />An Auditor type user cannot move a file||9    |Delete a file    |Users can select a file and move the file by either selecting ‘Move’ from mouse right click or from the right side panel of ‘Move’ button. A popup box should open to select a destination folder to move the file.<br />File (owner/editor permissions) can be moved to the destination folder where the user is the owner or the user has editor rights to the destination folder. File moving is applicable for regular files and also for BFD files. In case of a BFD file move, the destination folder must also be BFD type. For non BFD  file move, the For non BFD  file move, the source and destination folder must be non BFD type.<br />An user having ‘Viewer’ rights cannot move a file.<br /> An Auditor type user cannot move a file|
|9    |Delete a file    |Users should be able to delete a file by either mouse right click or from a panel button. A non BFD file can be deleted by a user who can have either ownership or editor rights. <br />A BFD file cannot be deleted by a user.<br />A tampered BFD file cannot be deleted by a user even if the user has ownership rights.<br />An Admin type user can delete a non BFD file of self and other users also.<br />An Admin type user cannot delete a BFD file and also cannot delete a tampered BFD file.<br />An Auditor can delete a BFD file and also a tampered BFD file.|
|10   |Rename a file    |Users with owners rights or editor rights should be able to  rename a file by either mouse right click or from a panel button.<br />An Auditor type user cannot rename a file.<br />A tampered BFD file cannot be renamed by the user (owner/editor/viewer).<br />An Admin type user cannot rename a BFD tampered file.<br />An Auditor type user cannot rename any type of files.|
|11   |Share a file   |Users having ownership rights should be able to share a file with other users. Users should be able to share a file by a mouse right click or from a panel feature to a group or users. Users should be  able to set the permission of Editor or Viewer for an individual or to a group. Users can also remove an individual user or a group assigned to a file.<br />An Admin type user cannot share another user's folder to any other user.<br />An Auditor cannot share a file.|
|12   |Copy a folder  |Users having ownership and editor rights can copy a folder to another folder (having ownership/editor rights).<br />Also a BFD folder can be copied only to a BFD folder. And a non-BFD folder can be copied to a non-BFD folder only. However a BFD folder can be copied to a non BFD folder.<br />A BFD folder which has a tampered symbol cannot be copied by any type of user.<br />An Auditor type role cannot copy a folder.|
|13   |Move a folder  |Users having ownership rights and editor rights can move a folder to another folder by mouse right click and also from panel feature. A folder can be moved only if the destination folder has ownership or has editor rights. Also a BFD folder can be moved to a BFD folder and a non-BFD folder can be moved to a non-BFD folder only.<br />Users having editor rights to a source folder, cannot move the source folder to the destination folder.<br />However a BFD folder can be moved to a non BFD folder.<br />A BFD folder if found tampered  cannot be moved by any type of user.<br />An Auditor type role cannot move a folder.|
|14   |Delete a folder|Users should be able to delete a folder by selecting a folder by either mouse right click or from panel button. Only an owner of a folder can delete a non BFD folder. However, a user with editor rights after deleting a folder with editor rights, the folder is removed from the editors folder, but not from owners folder. <br />A BFD folder or BFD tampered folder cannot be deleted by a user or by admin.<br />An Auditor type user can only delete BFD and BFD tampered folders.|
|15   |Rename a folder|Users with ownership or editor rights can rename a folder.<br />A tampered folder cannot be renamed by any type of user.<br />An Auditor type user cannot rename any type of folder.|
|16   |Share a folder |Similar to how an individual file can be shared with other users and groups, a folder can be shared in the same manner.|
|17   |Hide a file    |One can hide a file by selecting an option from a panel to hide/unhide. Once a file is hidden, it can still still be seen in a light grey text by another panel operation which will show hidden files/folder.<br />Operations such as copy/move/delete/update/rename cannot be performed for a hidden file.|
|18   |Hide a folder  |One can hide a folder by selecting an option from a panel to hide/unhide. Once a folder is hidden, it can still still be seen in a light grey text by another panel operation which will show hidden files/folders.<br />Operations such as copy/move/delete/update/rename cannot be performed for a hidden folder.|
|19   |Search a file or folder|Users should be able to search for a file or folder after login. Users should be able to enter a search string (must be minimum one character). All matching files and folders with the search string will be listed for non BFD and BFD types.<br />If the hidden files folder feature is ON, then files and folders which are hidden will be displayed in light gray color else hidden files and folder matching to search string will not be displayed.|
|20   |Create a Group|Users should be able to create a group from a menu. Multiple users can be added to a group. This group can then be assigned to a file or folder from a menu. Users should be able to add new users, remove a user and delete a group.<br />An Auditor type user cannot create a group.|
|21   |Validate a BFD  file/folder|Users can validate an individual BFD file for tampering detection or can validate an entire BFD folder by selecting a folder and initiating a Validation action. It is expected to display the current timestamp after the validation process is completed.|
|22   |Manage users |After login as Admin, admin will be able to see a list of all existing users by using a user management menu . List of users should be seen with their email id and role. Admin can delete a user even if the selected user has an Admin role. Admin should be able to add a new General user/Auditor/Admin with its name, email (must be unique in file manager system) and password.<br />An Auditor type user cannot manage users.<br />A General User cannot manage users.|
|23   |List BFD files|Admin should be able to see a list of BFD tampered files by using a menu. Type of BFD faults which can be seen would be ‘File URL in Scalar DB’, ‘Age in Auditor’, ‘File Hash in Scalar DB’, ‘File Hash in Auditor’, ‘File replace on S3’<br />An Auditor cannot see any BFD management features.<br />A General user cannot see any BFD management features.|
|24   |Inject a BFD fault|Admin can inject a BFD fault by selecting the ‘BFD Management’ menu and clicking on ‘Add new fault’ button.  Admin should now be able to see a list of General users only which have one or more than one non tampered BFD files. Admin can select a General user from the list and folders would be displayed only if they contain non tampered BFD files. Admin should be able to select a BFD file, select a version (few tampering type cases only), and then be able to  select any one tampering type as mentioned below<br />1. File URL in Scalar DB (any version)<br />2. Age in Auditor. (latest version only)<br />3. File Hash in Scalar DB.<br />4. File Hash in Auditor<br />5. File replace on S3<br />6. Delete Age in Auditor (latest version only)<br />7. Delete Id in Auditor (latest version only)<br />An Auditor cannot inject any BFD fault.<br />A General user cannot inject any BFD fault.|
|25   |Recover a BFD faulted file|Admin can recover a BFD fault from an admin menu which displays all BFD tampered files.<br />An Auditor cannot recover any BFD tampered file.<br />A General user cannot recover any BFD tamered file.||

## 4. User Roles and operations
The File manager application has primarily three roles:<br />
      **General User,**
      **Auditor,**
      **Admin,**
The user can login to the system using any of the above three roles. Based on these roles, the file handling operations vary and the details are mentioned in the below table.<br />
**File Operations as per User Role**
|Description|User as owner|User as editor|User as Viewer|Admin|Auditor|
|----------------|--------------------|-------------------|----------------------|---------|----------|
|Delete BFD audited file or folder|N|N|N|N|Y|
|Delete non BFD audited file or folder|Y|Y|N|Y|N|
|Rename a BFD audited or non BFD audited file or folder|Y|Y|N|Y|N|
|Copy a non BFD audited file|Y|Y|N|Y|N|
|Copy a BFD audited file|Y|Y|N|Y|N|
|Move a non BFD audited file|Y|N|N|Y|N|
|Move a BFD audited file|Y|N|N|Y|N|
|Update BFD audited file/folder|Y|Y|N|Y|N|
|Update non BFD audited file or folder|Y|Y|N|Y|N|
|Delete a faulty file|N|N|N|N|Y|
|Move a faulty file|N|N|N|N|Y|
|Copy a faulty file|N|N|N|N|Y|
|Rename a faulty file |N|N|N|N|Y|

**Other Use cases**
|Description|User as owner|User as editor|User as Viewer|Admin|
|----------------|-------------------|---------------------|--------------------|----------|
|Create a new user with specific role|N|N|N|Y|
|Delete an existing user|N|N|N|Y|
|Create a byzantine fault for a selected file|N|N|N|Y|
|Recover the faulty file created through the system|N|N|N|Y|

## 5. Final UI and navigation
Screen Navigation for admin user     
![screen_navigation_admin](/docs/assets/images/File_manager_HL_documentation/Screen_navigation_admin_user.jpg)

Screen Navigation for Auditor user     
![screen_navigation_auditor](/docs/assets/images/File_manager_HL_documentation/Screen_navigation_auditor_user.jpg)

**Note**:
* The auditor can view all the files and folders in the system.   
* Can perform Copy and Move operation only on a faulty file. (File with Byzantine fault )  
* Only an auditor can delete a BFD file.  

Screen Navigation for general user   
Refer : 'File Manager UI screens' for all the UI screens.    

## 6. Architecture     
The overall architecture of this application can be viewed as follows.     
Application architecture diagram
![architecture](/docs/assets/images/File_manager_HL_documentation/application_architect_diagram.jpg)

The application uses ScalarDB and scalarDL ledger as well as auditor. The design has conceptualized a multi-storage system by implementing Ledger <br />
and Auditor in separate dockers.<br />
The File Manager application maintains all the sensitive data related to BFD files and records of all operations in ScalarDL through contracts.<br /> 
The files are stored on S3 bucket and are accessed from S3.<br />
Byzantine Fault Injection Tool (BFD microservice) is implemented as a microservices and it injects different types of faults in ScalarDL Ledger,<br /> 
auditor and ScalarDB by accessing the databases externally.



