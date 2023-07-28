# File Manager Application

   ## Overview 
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
a.Registering data from Web Application
b.Detect data tampering
c.Create faults in the file beforehand and restrict operations on such faulty files.


The File Manager Demo application also provides functionalities such as User Management and Byzantine Fault Management. The User management
functionality is provided to create and delete  the system users and assign a role while creating the user. The Byzantine Fault Management 
Tool is implemented to inject Byzantine Fault  in the file and recover the fault. 
## Functional Block diagram of the File Manager Application
The various functionalities in the application are listed below.
### Authorization and User role management
The user registration and sign in operation. User creation and deletion and assigning a role to the user.
There are three roles : General user, Auditor and Administrator.
Some specific BFD files related operations can be performed by the auditor only.
    Group Management
    Create and delete a group. Add and delete users from the group.
    File and folder management
    The folder operations such as create, copy, move, delete, rename,hide are available.The file operations such as upload, copy, move, delete, 
    rename, update, hide are available.
    File and folder sharing
Files and folders can be shared to other users with sharing rights as editor and viewer. The sharing can be done at group level as well.
    Byzantine fault management (Fault injection and recovery tool)
This is a tool available for Administrator on the dashboard and is implemented to inject selected Byzantine fault in a BFD file. This fault is 
deliberately created by this tool and the Administrator can recover the already created faults from the List displayed. 
    User Management (user creation and deletion)
An Administrator can create a user and assign a role as well as delete a user using this menu option from the dashboard.
   Mutable data storage management (ScalarDB)
The mutable data is the data which is not considered as sensitive to alterations. The files which are not created (uploaded) as BFD files, the user 
related data, the sharing access related data, the group data etc are handled in ScalarDB. In short, all the data of the application is managed in 
ScalarDB.
   Immutable data storage management (ScalarDB and ScalarDL)
The files which are uploaded as BFD files are added in ScalarDL and are considered as the Assets. The file operations such as copy, move, delete, 
rename and update are updated in ScalarDL Ledger as well as Auditor. 
This allows the data tampering to be identified. 
The below diagram represents the interaction and relationship between these functionalities.

![architect](/docs/assets/images/File_manager_HL_documentation/Overall_architecture1.jpg)