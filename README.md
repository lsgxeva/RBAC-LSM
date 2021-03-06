============================================================
RBAC	(Role-based	Access Control	Linux Security Module)
============================================================
BY : Atluri Vamsi Krishna

===============
1. Description
===============
Role-based access control (RBAC) is a method of regulating access to computer or 
network resources based on the roles of individual users within an enterprise. In 
this context, access is the ability of an individual user to perform a specific task, 
such as view, create, or modify a file. RBAC enables users to carry out a wide range 
of authorized tasks by dynamically regulating their actions according to flexible 
functions, relationships, and constraints. This is in contrast to conventional 
methods of access control like DAC, which grant or revoke user access on a rigid, 
object-by-object basis. In RBAC, roles can be easily created, changed, or is 
continued as the needs of the enterprise evolve, without having to individually 
update the privileges for every user.

==========
2. Design
==========
This current implementation of RBAC aims for a simple RBAC model that has domain, a 
subset of the filesystem not the whole filesystem. The other part of filesystem not
defined in the domain will work under the guidance DAC. 

Three primary rules are are satisfied by SBRBAC:

- Role assignment: A subject can exercise a permission only if the subject has 
	selected or been assigned a role.
- Role authorization: A subject's active role must be authorized for the subject. 
	With rule 1 above, this rule ensures that users can take on only roles for which 
	they are authorized.
- Permission authorization: A subject can exercise a permission only if the 
	permission 	is authorized for the subject's active role. With rules 1 and 2, this
	rule ensure	that users can exercise only permissions for which they are authorized.

=======================	
2.1 Design Assumptions
======================= 

- This RBAC Model will support only a few specified directories as specified in the 
	'/etc/rbac/dir_domains'. This list can be modified with the help of 
	user utility.

- Existing Directories with permision to "rbac_inode_create" imply that users with 
	this rule in their active role are able to create new files/directory in this 
	directory. 

- Existing Directories with permision to "rbac_inode_unlink" imply that users with 
	this rule in their active role are able to delete files/directory in this 
	directory. 

- The list of users, roles and rules will be stored in '/etc/rbac/'.

- This module concerns only about the inode acess control with an exception of 
	special files like symlinks, Hardlinks, Devices etc. 

- This module will support only the following inode functions :
	a. rbac_inode_create
	b. rbac_inode_unlink
	c. rbac_inode_mkdir
	d. rbac_inode_rmdir
	e. rbac_inode_rename
	f. rbac_inode_getattr
	g. rbac_inode_setattr

- At any moment a user will have only one or no 'Active Role'.

- Admin can use the program 'user_prog' to generate or modify the policies.

- The list of functions available for Admin are :
	a. Add User to Role
	b. Delete a Role from User 
	c. Read All User -> Roles Mapping
	d. Assign an Active Role to a User
	e. Add a Rule to a Role
	f. Delete an Exsisting Rule from a Role
	g. Read all Rules belonging to a Role
	h. Add a directory to Domain of RBAC

- If for a user there is no policy related to an operation on a particular object 
	then its parent directory's access policy is used instead.

- Admin is here the root ( uid = 0)

==================
3. Implementation
==================

===================
3.1 Kernel Module
===================

This RBAC Model is implemented as a Linux Kernel Module using Linux Security Modules
(LSM) interface. The module is an inbuit kernel Module. 

- The LSM interface is inherited and in this case only inode functions are 
	implemented.

- Before every Inode access our hooks in SBRBAC are invoked.

- The SBRAC hooks then find the role assigned to the current user by searching in 
	the	user -> role mapping file at '/etc/rbac/users'.

- If an active role is found then rules related to this inode are searched for
	in this rule database stored at 'etc/rbac/roles/' directory.

- If a rule is found, then based on its flag (Allowed/Denied) the access is decided.

=======================
3.2 Admin User Utility
=======================

- Admin can write rules using the user utility "user_prog".
- Since Admin is assumed to be the root (uid = 0), the user utility should be run
	with "sudo" command.
- It can be used by executing the following commands :

	1) ./user_prog 1 <uid> <role> - Assign a Role to User 

	2) ./user_prog 2 <role> <func_name> <file/dir>  <0|1> - Assign a Policy to a Role 

	3) ./user_prog 3 <uid> <role> - Delete a User from a Role 

	4) ./user_prog 4 <role> <func_name> <file/dir> - Delete a Policy from a Role 

	5) ./user_prog 5 - Read All User to ROles Mapping 

	6) ./user_prog 6 <role> - Read All policies belonging to a Role 

	7) ./user_prog 7 <dir> - Assign a Dir to Domain of SBRBAC 

	8) ./user_prog 8 <uid> <role> - Assign an Active role user 

Examples : 

1) 
input : sudo ./user_prog 5

output :
Read User Roles a Rule
******USER - ROLE*******
ruid : 1000 role : manager act_role : 1
ruid : 1000 role : sales act_role : 0
ruid : 1001 role : sales act_role : 1

2)
input : sudo ./user_prog 1 1002 manager

output : 
Adding a Role
uid : 1002 Role : manager 

=============================
4. Deploying the source code
=============================

=======================
4.1 Kernel Module code
=======================
1. The files present in "security" folder in the provided zip file namely Kconfig &
	 Makefile should be replace	the Kconfig & Makefile in 	 "Sources/security/" folder.
2. The files : rbac_lsm.c, rbac.h, Kconfig, Makefile should be placed in 
	"sources/security/rbac/" folder.
3. In the .config file for the kernel "rbac" must be set as default security module 
	Or kernel.config file provided can be used instead.

=======================
4.2 User Utility code
======================= 
1. Binary "user_prog" can be used right away to create new roles.
2. To Predefined use predefined roles & rules copy "rbac" folder to "/etc/"

-----------
References
-----------
http://searchsecurity.techtarget.com/definition/role-based-access-control-RBAC
http://csrc.nist.gov/rbac/ferraiolo-kuhn-92.pdf
http://www.cse.psu.edu/~tjaeger/cse443-s12/docs/ch2.pdf
