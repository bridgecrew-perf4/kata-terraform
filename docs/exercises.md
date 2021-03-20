# Exercises

====================================================================

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

====================================================================

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

    
====================================================================
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

====================================================================

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

====================================================================
# Provisioners

## A) Setting up a first provisioner
* import the provider for null_resource
* Setup a null_resource
* set it up to use powershell
* echo "content" into output.txt
* $env:TF_CLI_ARGS_apply="--auto-approve" : Will add this, to approve only

* Test : The output.txt file contains "content"

## B) Using environment variables
* Set two environment variables
    * first = "this is first"
    * second = "this is second"
* Echo those to the file instead
* Taint the resource and re-run apply

* Test : The output file contains "this is first", "this is second"

## C) Triggers and self
* Create a local variable { file_suffix = "stuff" }
* null_resource : setup the triggers = { "file_name" = var.input_variable }
* self : Use self change the output file to output_${self.triggers[file_suffix]}.txt

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

====================================================================

# Azure
--- anki: ---

azurerm : Provider use?
Azure resources provider : Name?

$ azure login : Use?
Authenticate az to use azure : Syntax?

provider azurerm{ features{} } : wtf is this

resource "azurerm_resource_group" "example" {
    name     = "example"
    location = "UK South"
    tags = {
        These are tags
    }
}

azurerm_storage_account 
name                     = "storageaccountname"
resource_group_name      = azurerm_resource_group.example.name
location                 = azurerm_resource_group.example.location
account_tier             = "Standard"

tags = {
environment = "staging"
}

GRS : Geo-Redundant storage
account_replication_type = "GRS"

$ terraform state show resource.name : Use?
Show that state of a particular resource : Syntax?

$ terraform import azurerm_resource_group.name resource_id : Use?
Import an azurerm_resource_group : Syntax?

terraform import azurerm_resource_group.existing /subscriptions/<subId>/resourceGroups/helloworld : Use?
Import an azurerm_resource_group, called helloworld : Syntax?


--- exercises ---

## A) Creating an RG
* provider azurerm {}
* az login
* subscription?

* create a resource group
    * set the tags

* destroy that rg


* test : Run apply : There's an RG in the portal, with tags, set to uk south


## B) The naming module:
* Import the naming module
* Use it to generate a name for a storage account
* Output that name

module "naming_storage_account" {
  source = "github.com/Azure/terraform-azurerm-naming"
  suffix = ["rating", "panel", var.env_name, var.location_short]
}

output "names"{
    value = {
        name : module.naming_storage_account.storage_account.name
        unique : module.naming_storage_account.storage_account.name_unique
    }
}

## C) Using an existing rg, to put a storage account into
* create a resource group in the portal

* data : Read from an existing rg

* Use the naming thing
* put a storage container in The rg

* Naming: give it a unique name

* test : terraform state list - get the name, check it's in the portal

## D) Queues:
* put a queue in the storage account

resource "azurerm_storage_queue" "queue" {
  name                 = var.storage_queue_name
  storage_account_name = azurerm_storage_account.storage_account.name
  depends_on           = [azurerm_storage_account.storage_account]
}

* Get the name of the queue via : terraform state show azurerm_storage_queue.queue_name

* test : terraform state list - get the name, check it's in the portal

## E) Import and teardown

* import the resource group that i've created in the portal
* tear down everything

resource "azurerm_resource_group" "existing"{
    name = "hello_world"
    location = "UK South" 
}

* test : The resource group that you made by hand, is no longer in the portal


====================================================================
Workspaces and backends

Create the required shit for a backend ( storage account )
Use naming

new directory:
* use the foreign state
* create a st account 
* destroy it

Workspaces:
* idk lololololololol



====================================================================

* STOP TRYING TO MAKE COMPLETE KATA SETS
* INSTEAD MAKE SMALL MICRO EXERCISES, GROUP THEM, THEN MAKE KATA GROUPS

Raw exercises

* What do i need to do terraform azure
* how do i login
* how do i setup shit

* create a resource group
* create a storage account
* Put a queue into that storage account

* Use terraform to create the resources to make a backend
* Setup a backend
* Setup a workspace
* Teardown workspaces and stuff


====================================================================

# Raw kata:

## A) use for_each to generate several similar files
* Generate several files

* Create    :   local_file  :   Create 3 local_files, using one resource block, called "first.txt", "second.txt", "third.txt"
* Output    :   local_file  :   Output the path of the file called "second.txt"

### A) Acceptance
* Given that i have run terraform apply
* When i take a look at the directory
* I can see 3 different files, all with different content

## B) Locals
* do the above, but save the values for the file names into the locals block


====================================================================

====================================================================
The remaining kata's i would like:
* Not Azure related:
    * Console play, using inbuilt functions
    * CMD line state play
    * Something else
    * Something else

* Azure:
    * Workspaces and backends
    * Setting up and using Remote state ( The terraform block, not the backend state stuff )
    * Something that requires the use of depends_on
    * Creating some resource groups and putting shit into them

====================================================================

filesystem

path.module
path.root
path.cwd

# CLI Stuff:

$ terraform graph 
Looks fucking dope

terraform state list
terraform state in general
terraform state list, mv, rm
terraform state show

================
workspace setup
$ terraform workspace list      
$ terraform workspace select    terraform show
$ terraform workspace new       
$ terraform workspace show      $env:TF_CLI_ARGS_apply="--auto-approve" : Will add this, to approve only

# Workspaces and backend:
Create a workspace
Set the backend to be azurerm
Create some azure stuff, using the remote backend state
Then destroy it
Try import :)?!
================

fucking nice

# Console
Could make a kata around this, just having and editing a state, in a live manor


terraform import:
* This is for using an existing resource and importing it into your state, instead of making a resource from scratch
* You do need the resource already in your config file

//Merging 2 configs, for 3 different resources?
> merge({a="b", c="d"}, {e="f", c="z"})

can(expression)

//you can filter the list with an if
[for s in var.list : upper(s) if s != ""]

== Stuff gab used ==
concat(string, [ string array ]) : gab's used it
can(coalesce(var.naming_suffix...))
====================

//Dynamic blocks
resource "aws_elastic_beanstalk_environment" "tfenvtest" {
  dynamic "setting" {
    for_each = var.settings
    content {
      namespace = setting.value["namespace"]
      name = setting.value["name"]
      value = setting.value["value"]
    }
  }
}

you can assign variables via a flag from the cli
terraform apply -var="image_id=ami-abc123"

If you're setting alot of them, you can use a .tfvars file
terraform apply -var-file="testing.tfvars"

a .tfvars file is basically a tf file that only contains assignements:
image_id = "ami-abc123"
availability_zone_names = [
  "us-east-1a",
  "us-west-1c",
]

files that are automatically loaded in:
Files named exactly terraform.tfvars or terraform.tfvars.json.

$ terraform apply -target resource.name[0]
$ terraform fmt 

Outputs the visual dependency graph of Terraform resources represented by the configuration in the current working directory.
$ terraform graph

============================================================
== Things that are on the certified exam ==
Verbose logging:
Given a scenario: choose when to enable verbose logging and what the outcome/value is
$ terraform validate

