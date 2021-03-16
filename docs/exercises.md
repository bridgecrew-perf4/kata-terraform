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
3) Constraints, life cycles and tainting

## A) Variables With constraints

* variable  :   number      :   number of files generated, must be a number, prevent non numbers and values greater than 4. output "Input no more than 4." as the error
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

Deep structures:
Main)       main.tf
Module)     module/main.tf
Sub mod)    module/submodules/sub_one/main.tf

output:
output/prefix_0.txt
output/prefix_1.txt
output/prefix_2.txt
output/prefix_3.txt

* structure :   sub_module  :   create the sub module module/submodule/main.tf
* variable  :   sub_module  :   file_content, the content of the generated files
* variable  :   sub_module  :   the prefix for the files
* create    :   sub_module  :   the generated files are in the directory "output", "output/file.txt"

* structure :   module      :   create the module
* create    :   module      :   invoke the sub module twice, in 2 different blocks, with different content for each
* output    :   module      :   output the files [module_a, module_b]

* structure :   main        :   create the top level main.tf
* output    :   main        :   1) flatten the list, then splat it to use just output path
* output    :   main        :   2) turn the flat list to generate an object { filename : content }
* output    :   main        :   3) turn the flat list, into a list of just file names, without the directory

====================================================================

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

############# ############# ############# #############
# MISSING ONE MORE RELEVENT THING         #############
############# ############# ############# #############

could tf vars be a relevent thing?

====================================================================
====================================================================

filesystem
state

path.module
path.root
path.cwd

terraform state list
terraform state in general

====================================================================

## a) Provisioners
* use a provisioner

====================================================================
