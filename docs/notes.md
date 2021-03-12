Todo:
Kata on creating a module
Kata on creating a module, that uses the count feature

========== Interesting =============================================================
meta properties:

========== Might be useful =========================================================

* Remote Tf state management
    * how to setup
    * what happens if the state changes, but the files aren't updated
    * tf import

========== I don't think i care about this tbh =====================================
========== General unsorted notes ==================================================

resource "my" "resource"{
    lifecycle {
        create_before_destroy = true //when doing apply, create the new resource before destroying the old one
        prevent_destroy = true //prevent any deletion of a particular object
        ignore_changes = true //prevent a resource from updating, when related data changes
    }
}

local variables are local to the module. They're not input, they're not output
locals {
    variable = value
}

terraform taint resourcen_type.resource_name 
taint destroys and rebuilds a resource, on the next apply

meta properties:

depends_on  :   specifies hidden dependencies
provisioner :   for taking extra actions after resource creation
connection  :   same as above

resource "my" "resource"{
    timeouts {
        create = "60m"
        delete = "2h"
    }
}

local -> for editing local files

On a module level:
* outputs are what's returned from the module
* variables are passed in from outside ( cmd or an invoking module )
* invoking modules must provide all the required variables

invoking a module

module name{
    source = "../path"
    variable_name = "value"
}

All variables for a module, MUST be provided by the calling module

All values are passed into the module, via the variables

All values are passed back out, via outputs

module.module_name.output
