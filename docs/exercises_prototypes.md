====================================================================

* STOP TRYING TO MAKE COMPLETE KATA SETS
* INSTEAD MAKE SMALL MICRO EXERCISES, GROUP THEM, THEN MAKE KATA GROUPS

====================================================================

# Raw kata:

## A) use for_each to generate several similar files
* Generate several files

* Create    :   local_file  :   Create 3 local_files, using one resource block, called "first.txt", "second.txt", "third.txt"
* Output    :   local_file  :   Output the path of the file called "second.txt"
* Given that i have run terraform apply
* When i take a look at the directory
* I can see 3 different files, all with different content

====================================================================

Import and plan:
* go into the azure portal
* create a complex object, with alot of configs ( azure_network_interface, or something )
* create the config
* Import it into terraform
* Use terraform plan to check the differences between the config and reality
* Update the config, until it matches reality

* Test : $ terraform apply
    * 0 Objects updated

====================================================================
# Little bits that don't constitute anything major:

* for_each = toset([ array of items ])
    * each.key

====================================================================
