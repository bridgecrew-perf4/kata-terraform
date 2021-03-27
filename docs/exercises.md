# Exercises
 
---

# 1) Creating a local file  ( Ready for use )

## Description:
* We're creating a file, and setting it's content
* We're then getting the value for the content from a variable
* We're then loading the value, for the content, from an existing file

## A) Create a local file

* Action        : Where         What

* Create        : Directory  :  "local_file_kata"
* Create        : local_file :  "content.txt"
* Set Attribute : local_file :  content "pog"

### A) Acceptance:
* Given that i have run terraform apply
* When i take a look at the generated files
* I can see:
    * content.txt : with the content "pog"

## B) Make the content of the local file a variable
* Declare       :   Variable   :    type:string
* Set Attribute :   local_file :    Content Set to be the same as the variable

### B) Acceptance
* Given that i've run terraform apply, 
* Then given the string "Poggers"
* When i take a look at the the content of the file
* I can see it matches "Poggers"

## C) Using existing file data
* Create        :   local_file  :    Seperate from the one above
* Data          :   Data file   :    "resources/template.txt"
* Set Attribute :   local_file  :    content to be the same as the data file

### C) Acceptance:
* Given that i have run terraform apply
* When i take a look at the files generated
* I can see a new file
* That contains the same content as the file "resources/template.txt"


---


# 2) Module management

## Description
* We're creating a module
* That generates 2 files
* Then uses count to generate those files
* Then generating a variable number of files

* Action        : Where         What

## A) Using a module to generate files
* Create    : Directory             :  The modules container
* Create    : module                :  x2 files, content irrelevent 
* Output    : local_file.filename   :  x2 files, output their names to the calling module
* Run       : From calling module   :  Run the module, from a parent main.tf, in the parent directory

### A-A) Acceptance:
* Given that i have a module directory, and i'm in the directory above it
* When i run terraform apply
* I can see it generates 2 files:
    * file_0.txt
    * file_1.txt

* Given that i have run terraform apply
* When i run terraform output
* I receive:
    * file_one = "output/file_0.txt"
    * file_two = "output/file_1.txt"

## B) using count to generate several files
* Implement Count   :   module.local_files  :   Implement count, on one resource, to generate your two files
* Create            :   local_file          :   Generate the files file_0.txt and file_1.txt
* Output            :   local_file.filename :   Output the names of the 2 files, individually

### B-A) Acceptance
* Given i've run terraform apply
* When i take a look at the generated outputs
* I can see:
    * file_one = "output/file_0.txt"
    * file_two = "output/file_1.txt"

## C) Using variables and outputs
* variable  :   module  :   Have a variable determine the number of generated files, call it file_count
* Output    :   module  :   Output all filenames ( only ) together as an array

### C-A) Acceptance
* Given that i've run terraform apply and set file_count to 4
* When i run terraform output
* I can see:
    output : [ "file_0.txt", "file_1.txt", "file_2.txt", "file_3.txt" ]

## D) Implement that module:

* Create    :   main.tf :   Create the top level main.tf
* module    :   main.tf :   Invoke the /module
* output    :   main.tf :   have the new main.tf, output the modules outputs
* I can see:
    output : [ "file_0.txt", "file_1.txt", "file_2.txt", "file_3.txt" ]


--- 


3) Constraints, life cycles and tainting

## A) Variables With constraints

* variable  :   number      :   number of files generated, must :
    * be a number 
    * prevent non numbers 
    * prevent values greater than 4. 
    * output "Input no more than 4." as the error

* test      :   tf apply    :   Try and input 15 as the file_count, expect => "Input no more than 4." as the result

### B-A) Acceptance
* Given i have run terraform apply
* When i give it the value of 15 for the variable
* I should see that i get the "Input no more than 4." error message.


## B) Running it again, preventing changes

* config    :   local_file  :   set the content to be "FIRST RUN"
* config    :   local_file  :   prevent the files from having their content changed, on subsequent runs
* create    :   tf apply    :   2x local_file, by using the above variable as the count
* edit      :   local_file  :   change the content value to "SECOND RUN"
* create    :   tf apply    :   Set the variable to 4
* test      :   check       :   Check that the first 2 files have different content to the second two

### B-A) Acceptance
* Given i have ran apply the first time
* and set content to "FIRST RUN"
* and provided 2 for file_count
* I can see that 2 files have been generated with the same content

* Given that i have changed the content in the config to "SECOND RUN"
* When i run apply a second time
* and supply 4 as the file_count
* I can see that there are now 4 files:
    * 2 with the content "FIRST RUN"
    * 2 with the content "SECOND RUN"

## C) Taint
* config    :   local_file      :   Change the content to be "THIRD RUN"
* taint     :   local_file[0]   :   Taint one of the files with the "FIRST RUN" content
* test      :   tf apply        :   Check that the content of the first file is now "SECOND RUN", not "FIRST RUN"

### C-A) Acceptance
* Given that i have tainted a file
* When i run tf apply
* I can see that the tainted file's values have changed

## D) Deletion prevention
* Config    :   local_file      :   Set the files to prevent being destroyed
* taint     :   local_file      :   Taint the remaining file, that contains "FIRST RUN"

* test      :   local_file      :   Run apply again => Expect to get a prevent destroy error

### D-A) Acceptance
* Given that i have tainted the remaining file
* when i run terraform apply
* I receive an error, preventing destroy


--- 


# 4) Submodules and layers:

Directory structure
Main)       main.tf
Module)     module/main.tf
Sub mod)    submodule/main.tf

## A) Create the sub_module:
* structure :   sub_module  :   create the submodule/main.tf
* variable  :   sub_module  :   file_content, the content of the generated files
* variable  :   sub_module  :   the prefix for the files
* output    :   sub_module  :   output the file data
* create    :   sub_module  :   2x Files, the generated files are in the directory "output", "output/prefix_fileindex.txt"

* test  ->  run the sub_module, file_content = "test", prefix = "pre"
* check ->  2 files generated, content = "test", names pre_0.txt, pre_1.txt
* check ->  output -> [ file_1, file_2 ]

## B) Create the module:

* local     :   module      :   create a local variable, that's a map, containing the modules values
* structure :   module      :   create the module directory
* create    :   module      :   invoke the sub module twice, using a for_each on the above local
* output    :   module      :   output the files = [ module_a.files, module_b.files ]

* test  -> Output directory contains 4 files. 
* output -> terraform output outputs [ [ prefix_one_0.txt, prefix_one_1.txt ], [ prefix_two_0.txt, prefix_two_1.txt ] ]

## C) Invoke the module

* structure :   main        :   create the top level main.tf
* module    :   main        :   Invoke the module
* output    :   main        :   output file_paths)      Flatten the list, then splat it to display only the full output path
* output    :   main        :   output files_as_map)    Turn the flat list to generate a map { filename : content }
* output    :   main        :   output just_names)      Turn the flat list, into a list of just file names, without the directory

## C-A) Acceptance:

* Given i have run terraform apply
* When i run terraform output
* I get the following outputs:

expected output:
* file_paths = [
  "output/FIRST_0.txt",
  "output/FIRST_1.txt",
  "output/SECOND_0.txt",
  "output/SECOND_1.txt",
]

* files_as_map = {
  "output/FIRST_0.txt" = "FIRST BLOCK"
  "output/FIRST_1.txt" = "FIRST BLOCK"
  "output/SECOND_0.txt" = "Second Block"
  "output/SECOND_1.txt" = "Second Block"
}

* just_names = [
  "FIRST_0.txt",
  "FIRST_1.txt",
  "SECOND_0.txt",
  "SECOND_1.txt",
]


---


# 5) Provisioners

## A) Setting up a first provisioner
* import the provider for null_resource
* Setup a null_resource
* set it up to use powershell
* echo "content" into output.txt
* Set the environment variable, to auto approve every apply and apply only

* Test : The output.txt file contains "content"

## B) Using environment variables
* Set two environment variables
    * first = "this is first"
    * second = "this is second"
* Echo those to the file instead
* Taint the resource and re-run apply

* Test : The output file contains "this is first", "this is second"

## C) Triggers and self
* Create a local variable { filename = "output.txt" }
* null_resource : setup the triggers = { "file_name" = var.input_variable }
* self : Use self change the output file to ${self.triggers[file_name]}

### C-Acceptance)
* run apply once note that the file has been created
* remove the output file

* run apply again, notice the file hasn't been remade

* change the local variable value
* re-run apply, notice that it's now there with the new name

## D) When
* Create a second provisioner in the same block
* Have it append "destroyed" to the output file, when resource is destroyed

### D-A)
* Run apply, note that the file only contains the lines from the first provisioner
* Run destroy, note that the file now contains the words destroyed


---


# 6) Azure
## A) Creating a RG
* Import the azure provider
* Allow the az tool to login
* create a resource group
    * call it hello_azure
    * set the tag "hello" to "azure"

* test : Run apply : Check the portal
    * there's an RG called hello_azure
    * there's a tag called "hello" with a value of "azure"
    * The location is uk south
    
* destroy that rg

## B) The naming module:
* Import the naming module
* Use it to generate a name for a storage account
* Output : Array
    * A name
    * A name that's unique

## C) Using an existing rg, to put a storage account into
* delete the output from earlier
* create a resource group in the portal, call it "rg_jvh"
* Use data to read details from that rg
* put a storage account in The rg
    * Use naming to give it a unique name with the "jvh" suffix
    
* test : terraform state list - get the name
* test : check it's in the portal, in the premade resource group

## D) Queues:
* put a queue in the storage account

* Test : Terraform state list - get the name, check it's in the portal
* Test : Check it's in the portal, in the given storage account 

## E) Import and teardown

* convert the data resource_group, to be a resource resource_group
* import the resource group that i've created in the portal

* test : terraform apply : Does not create a new resource group
* test : Terraform destroy => The resource group that you made by hand, is no longer in the portal


---


# 7) Workspaces and backends

## A) Setup azure, via terraform
* Create a new directory, Call it tf_state
* Create an rg => call it tf_state
* put a storage account in that rg => call it unique, with jvh suffix
* Create an azure_rm_storage_container
* Invoke the apply

* Test : I have a 
    * storage_container, 
    * in a storage account, 
    * in a resource group

## B) Use the stuff for the backend
* create a new directory call it "working"
* setup working to use azure as the backend
* Create a brand new resource group, using the naming module

* Test : Go to the portal, check
    * I have the brand new resource group

* Test : The terraform.tfstate file is located in the .terraform directory
* Test : Cat the state file, it ONLY contains information about the backend
* Test : Terraform state list, shows the resource_group you just made

## C) Check the foreign state

* Create a third directory : Call it state_check
* Set it up to use the foreign state
* $ terraform init
* check the state

* Test: state show on the rg, should show the one you just made

## D) Use workspaces to create something AGAIN
* Go back to the "working" directory
* Add the workspace name to the naming module's prefixes
* Create the new workspace "pog"

* test : Run apply, apply succeeds
* test : Check in the portal, you now have 2 resource groups created by the "working" directory
* test : Go to check_state
* test : In check_state, Listing the workspaces, shows default and pog
* test : In check_state, list the state of the default workspace, only one RG is there
* test : In check_state, list the state of the pog workspace, note only the newest rg is there

## E) Tear down pog
* Go back to working
* Teardown pog
* Delete the pog workspace

* Test: In the portal, the pog rg is now delete
* Test: When listing workspaces, in the "working" directory, only default is now shown

## E) Tear it down, piece by piece
* Fully teardown the stuff using foreign state:
    * Move to the state_check directory, from here:
        * Tear down default

    * Tear down the state container

* Test: Go to the portal, all 3 of the rg's are now gone


--- 


# 8) Messing with state

## A) Setup and list state
* Create 3 files, using count
* Use list the state via the commandline
* List out the state of the generated files
* Show the full state of the first generated file

* Test: Using terraform state show, i can see the state of the first generated file

## B) Moving state

* rename the resource in the main file - Do nothing else
* Use state mv to move the old state into the new resource
* run apply

* Test: When i run terraform apply, no resources are made or deleted

## C) Removing state
* remove the state of one of the files, using "$ terraform state" command
* re-run terraform apply 

* Test: When i run terraform apply, the resource is recreated

## D) Moving state into a module:
* Create a new directory, called mod
* Import that in the top level main.tf
* Delete the configs from the main file and move it into the module ( mod ) main.tf
* use terraform to move the state into the module
* Run terraform init

* Test: When i run terraform apply, nothing is created or deleted
