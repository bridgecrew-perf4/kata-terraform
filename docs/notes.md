# CLI

$ terraform providers schema -json : Output the schema of the resources from a provider
Useful for seeing the interfaces of stuff

Env variables:

---

# State

data "terraform_remote_state" "vpc"
allows for using remote state, not using backends

via this, you have access to the OUTPUTS of remote state

data "terraform_remote_state" "vpc" {
  backend = "remote"

  config = {
    organization = "hashicorp"
    workspaces = {
      name = "vpc-prod"
    }
  }
}

# Terraform >= 0.12
resource "aws_instance" "foo" {
  # ...
  subnet_id = data.terraform_remote_state.vpc.outputs.subnet_id
}

data "terraform_remote_state" "vpc" {
  backend = "local"

  config = {
    path = "..."
  }
}

terraform_remote_state.name.backend = "value" : Configure the backend for a config

Remote state like this, is READONLY

Backends determine where state is stored. For example, the local (default) backend stores state in a local JSON file on disk. 

If supported by your backend, Terraform will lock your state for all operations that could write state. This prevents others from acquiring the lock and potentially corrupting your state.

Backends determine only where the state is stored, the operations are run from the local machine

//setup to use azurerm
terraform {
  backend "azurerm" {
    resource_group_name  = "StorageAccount-ResourceGroup"
    storage_account_name = "abcd1234"
    container_name       = "tfstate"
    key                  = "prod.terraform.tfstate"
  }
}

//Using a foreign backend:
terraform init -backend-config="../../local_backend_config.tfvars" -backend-config="key=dev-rating.tfstate"

In the .tfvars:
resource_group_name = "rg-vitruvius"
storage_account_name = "stvitruviusshared"
container_name = "tfstate"


---

# Provisioners:

on_failure = continue : Ignore the error the provisioner causes and just keep going

on_failure = fail : Raise an error and stop applying. This is default. If this fails, the resource is tainted

connection : connection blocks give the ssh details to the provisioner
# Copies the file as the root user using SSH
provisioner "file" {
  source      = "conf/myapp.conf"
  destination = "/etc/myapp.conf"

  connection {
    type     = "ssh"
    user     = "root"
    password = "${var.root_password}"
    host     = "${var.host}"
    port     =   4200
    timeout = "30s"
    script_path = "path"
  }
}

provisioner types:
  provisioner "file" {
    source      = "conf/myapp.conf"
    destination = "/etc/myapp.conf"
  }

  file : for copying files onto a new resource ( like a vm )

  provisioner "file" {
    content     = "ami used: ${self.ami}"
    destination = "/tmp/file.log"
  }

remote exec:
inline : A list of strings to be executed as commands
script : the path to a script that needs to be executed
scripts : An array of script paths

resource "aws_instance" "web" {
  # ...

  provisioner "remote-exec" {
    inline = [
      "puppet apply",
      "consul join ${aws_instance.web.private_ip}",
    ]
  }
}

----
terraform_remote_state is a built in provider for working with remote state

data "terraform_remote_state" "vpc" {
  backend = "remote"

  config = {
    organization = "hashicorp"
    workspaces = {
      name = "vpc-prod"
    }
  }
}

# Terraform >= 0.12
resource "aws_instance" "foo" {
  # ...
  subnet_id = data.terraform_remote_state.vpc.outputs.subnet_id
}

terraform state mv : Tell terraform you've moved a config to another module's config, so that it doesn't destroy and recreate the resource

getting modules hosted on foreign domains
module "consul" {
  source = "github.com/hashicorp/example"
}

modules hosted by hashicorp
module "consul" {
  source = "hashicorp/consul/aws"
  version = "0.1.0"
}

//Generic git repos
module "vpc" {
  source = "git::https://example.com/vpc.git"
}

module "storage" {
  source = "git::ssh://username@example.com/storage.git"
}

variable "ami" {
  type = object({
    # Declare an object using only the subset of attributes the module
    # needs. Terraform will allow any object that has at least these
    # attributes.
    id           = string
    architecture = string
  })
}


======================================================================
Leftover shit from the exercises file:

filesystem

path.module
path.root
path.cwd

# CLI Stuff:

$ terraform graph 
Looks fucking dope

================
workspace setup
$env:TF_CLI_ARGS_apply="--auto-approve" : Will add this, to approve only

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

Remote state, maybe?

data "terraform_remote_state" "conf"{
    backend = "local"

    config = {
        path = "../tf_state/terraform.tfstate"
    }   
}

output "test"{
    value = data.terraform_remote_state.conf.outputs.container_name
}

============================================================
== Things that are on the certified exam ==
Verbose logging:
Given a scenario: choose when to enable verbose logging and what the outcome/value is
$ terraform validate

================================================

terraform console

path.module : returns the filesystem path of the module where the expression is placed.
path.root : returns  filesystem path of the root module of the configuration.
path.cwd : returns the filesystem path of the current working directory. In normal use of Terraform this is the same as path.root 
terraform.workspace : returns the name of the currently selected workspace.

ternary:
//you can filter the list with an if
[for s in var.list : upper(s) if s != ""]

//you can do a for, each key value pair in a map
[for k, v in var.map : length(k) + length(v)]

//s... groups together results that have the same key, in the resulting map
{for s in var.list : substr(s, 0, 1) => s... if s != ""}

dynamic blocks allow use for foreach on nested blocks ( blocks inside of your resource )

resource "aws_elastic_beanstalk_environment" "tfenvtest" {
  name                = "tf-test-name"
  application         = "${aws_elastic_beanstalk_application.tftest.name}"
  solution_stack_name = "64bit Amazon Linux 2018.03 v2.11.4 running Go 1.12.6"

  dynamic "setting" {
    for_each = var.settings
    content {
      namespace = setting.value["namespace"]
      name = setting.value["name"]
      value = setting.value["value"]
    }
  }
}

set( ... ) => a list of unique values

---

format("Hello, %s!", "Ander") => hello, Ander

chomp => removes newline from string

> formatlist("Hello, %s!", ["Valentina", "Ander", "Olivia", "Sam"])
[
  "Hello, Valentina!",
  "Hello, Ander!",
  "Hello, Olivia!",
  "Hello, Sam!",
]

> replace("1 + 2 + 3", "+", "-")
1 - 2 - 3

replace("a + b", "+", "-") : Effect?
a - b : replace syntax:

> split(",", "foo,bar,baz")
[
  "foo",
  "bar",
  "baz",
]

> substr("hello world", 1, 4)
ello

> title("hello world")
Hello World

> trim("?!hello?!", "!?")
hello
removes specified values from front and end of a string

> trimprefix("helloworld", "hello")
world

> trimsuffix("helloworld", "world")
hello

> trimspace("  hello\n\n")
hello

> alltrue(["true", true])
true

anytrue(list)

> chunklist(["a", "b", "c", "d", "e"], 2)
[
  [
    "a",
    "b",
  ],
  [
    "c",
    "d",
  ],
  [
    "e",
  ],
]


> coalesce("", "b")
b
returns the first non-empty string

takes a number of args, returns the first non-null one
> coalescelist([], ["c", "d"])
[
  "c",
  "d",
]

remove any empty strings from an array, then return it
> compact(["a", "", "b", "c"])
[
  "a",
  "b",
  "c",
]

> concat(["a", ""], ["b", "c"])
[
  "a",
  "",
  "b",
  "c",
]


contains(list, value)


> distinct(["a", "b", "a", "c", "d", "b"])

> flatten([["a", "b"], [], ["c"]])
["a", "b", "c"]

> index(["a", "b", "c"], "b")
1

> keys({a=1, c=2, d=3})
[
  "a",
  "c",
  "d",
]

length(array)

> map("a", "b", "c", "d")
{
  "a" = "b"
  "c" = "d"
}

> matchkeys(["i-123", "i-abc", "i-def"], ["us-west", "us-east", "us-east"], ["us-east"])
[
  "i-abc",
  "i-def",
]

> merge({a="b", c="d"}, {e="f", c="z"})
{
  "a" = "b"
  "c" = "z"
  "e" = "f"
}

> range(3)
[
  0,
  1,
  2,
]

reverse(list)

//find the values present in all given lists
> setintersection(["a", "b"], ["b", "c"], ["b", "d"])
[
  "b",
]

find all possible combinations of the given items
setproduct( list1, list2, ... )

> setsubtract(["a", "b", "c"], ["a", "c"])
[
  "b",
]

> setunion(["a", "b"], ["b", "c"], ["d"])
[
  "d",
  "b",
  "c",
  "a",
]


slice(list, startindex, endindex)
> slice(["a", "b", "c", "d"], 1, 3)
[
  "b",
  "c",
]

> sort(["e", "d", "a", "x"])
[
  "a",
  "d",
  "e",
  "x",
]

> sum([10, 13, 6, 4.5])
33.5



> transpose({"a" = ["1", "2"], "b" = ["2", "3"]})
{
  "1" = [
    "a",
  ],
  "2" = [
    "a",
    "b",
  ],
  "3" = [
    "b",
  ],
}

//the opposite of keys
> values({a=3, c=2, d=1})
[
  3,
  2,
  1,
]

> zipmap(["a", "b"], [1, 2])
{
  "a" = 1,
  "b" = 2,
}

> jsondecode("{\"hello\": \"world\"}")
{
  "hello" = "world"
}

> jsonencode({"hello"="world"})
{"hello":"world"}

abspath => take a relative file path, convert it to absolute

> dirname("foo/bar/baz.txt")
foo/bar

file(path) => returns the contents of a file, as a string

fileexists(path)

> fileset(path.module, "files/*.txt")
[
  "files/hello.txt",
  "files/world.txt",
]


templatefile(path, vars) => templatefile reads the file at the given path and renders its content as a template using a supplied set of template variables.

cidrhost calculates a full host IP address for a given host number within a given IP network address prefix.
cidrhost(prefix, hostnum)

cidrnetmask(prefix) => 
cidrnetmask converts an IPv4 address prefix given in CIDR notation into a subnet mask address.


cidrsubnets calculates a sequence of consecutive IP address ranges within a particular CIDR prefix.
> cidrsubnets("10.1.0.0/16", 4, 4, 8, 4)
[
  "10.1.0.0/20",
  "10.1.16.0/20",
  "10.1.32.0/24",
  "10.1.48.0/20",
]

can(expression)
can evaluates the given expression and returns a boolean value indicating whether the expression produced a result without any errors.

toset(list) => convert a list to a set

try evaluates all of its argument expressions in turn and returns the result of the first one that does not produce any errors.
name   = tostring(try(local.raw_value.name, null))
