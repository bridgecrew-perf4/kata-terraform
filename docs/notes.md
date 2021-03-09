Todo:
Kata on creating a module
Kata on creating a module, that uses the count feature

========== Interesting =============================================================
meta properties:
count       :   for creating multiple resource instances according to a count ( interesting )
string interpolation : "${stuff}"
"Server ${count.index}"

resource "my_resource" "name"{
    count = 2
    value = "${count.index}"
}

output can't use count

output "name"{
    value = resource.name[count_index].arg
}

resource "local_file" "file"{
    count = 2
    filename = "local_file_${count.index}"
    content = "my content"
}

output "file_name_one"{
    value = local_file.file[0].filename
}

resource "local_file" "file"{
    for_each = {
        "test_key" = "value"
    }

    filename = "local_file_${each.key}.txt"
    content = each.value
}

output "my_output"{
    value = local_file.file["test_key"]
}

========== Might be useful =========================================================
========== I don't think i care about this tbh =====================================
========== General unsorted notes ==================================================
local variables are local to the module. They're not input, they're not output
locals {
    varialbe = value
}

terraform taint resourcen_type.resource_name 
taint destroys and rebuilds a resource, on the next apply

terraform fmt file.tf
reformats a tf file

terraform.io : the website
terraform.io/docs : the docs section of the website
terraform.io/docs/language/index.html : the docs on the terraform language

# : single line comment
// : single line comment
/* */ : multi line comment

meta properties:
depends_on  :   specifies hidden dependencies


for_each    :   for creating multiple resource instances, according to a set of strings or something
provider    :   for selecting a non-default provider configuration
lifecycle   :   for lifecycle customisations
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


All terraform projects, can be used as submodules.
All variables for a module, MUST be provided by the calling module

All values are passed into the module, via the variables
All values are passed back out, via outputs
module.module_name.output
