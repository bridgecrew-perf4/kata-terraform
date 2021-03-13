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
* Create    : module.local_file     :  x2 files, content irrelevent 
* Output    : local_file.filename   :  x2 files, output their names to the calling module
* Run       : From calling module   :  Run the module, from a parent main.tf, in the parent directory

### A) Acceptance:
* Given that i have a module directory, and i'm in the directory above it
* When i run terraform apply
* I can see it generates 2 files:
    * file_0.txt
    * file_1.txt

## B) using count to generate several files
* Implement Count   :   module.local_files  :   Implement count, on one resource, to generate your two files
* Create            :   local_file          :   Generate the files file_0.txt and file_1.txt
* Output            :   local_file.filename :   Output the names of the 2 files, individually

### B) Acceptance
* Given i've run terraform apply
* When i take a look at the generated outputs
* I can see:
    * file_one : file_0.txt
    * file_two : file_1.txt

* Given i've run terraform apply
* When i take a look at the generated files
* I can see:
    * file_0.txt
    * file_1.txt

## C) Using variables and outputs
* Implement variable    :   module  :   Have a variable determine the number of generated files, call it file_count
* Output                :   module  :   Output all filenames ( only ) together as an array

### Acceptance:
* Given that i've run terraform apply and set file_count to 4
* When i run terraform output
* I can see:
    output : [ "file_0.txt", "file_1.txt", "file_2.txt", "file_3.txt" ]
    

====================================================================

## A) use for_each to generate several similar files
* Generate several files

* Create    :   local_file  :   Create 3 local_files, using one resource block, called "first.txt", "second.txt", "third.txt"
* Output    :   local_file  :   Output the path of the file called "second.txt"

### A) Acceptance
* Given that i have run terraform apply
* When i take a look at the directory
* 

## B) Locals
* do the above, but save the values for the file names into the locals block

############# ############# ############# #############
# MISSING ONE MORE RELEVENT THING         #############
############# ############# ############# #############

could tf vars be a relevent thing?

====================================================================

//Generate a bunch of stuff
//Use lifecycles to prevent values from changing
//Use taint to re-create something

## C) lifecyles
* use the ignore_changes hook -> Would be useful
* Create one set of files
* up the value of the number created
* then create more files, but have the second load have different values

## c) taint
* re-apply changes to a resource
* taint a  resource
terraform taint resourcen_type.resource_name 
taint destroys and rebuilds a resource, on the next apply

====================================================================

EXTRAS, THAT COULD BE DONE

## a) Provisioners
* use a provisioner

