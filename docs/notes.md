========== Interesting =============================================================

========== Might be useful =========================================================

* Remote Tf state management
    * how to setup
    * what happens if the state changes, but the files aren't updated
    * tf import

========== I don't think i care about this tbh =====================================

========== General unsorted notes ==================================================

state
terraform state list : List out the state of the current project

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

terraform init -upgrade : upgrade providers to more recent version

.terraform.lock.hcl : Contains a list of the providers and versions being used, like package.json

terraform state mv : Tell terraform you've moved a config to another module's config, so that it doesn't destroy and recreate the resource

$ terraform taint module.salt_master.aws_instance.salt_master : Taint resources inside of a module

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



---

string functions:

format:
> format("Hello, %s!", "Ander") => hello, Ander
chomp => removes newline from string

> formatlist("Hello, %s!", ["Valentina", "Ander", "Olivia", "Sam"])
[
  "Hello, Valentina!",
  "Hello, Ander!",
  "Hello, Olivia!",
  "Hello, Sam!",
]

> join(", ", ["foo", "bar", "baz"])
foo, bar, baz


apply regex to string, returning matching substrings
regex(pattern, string)

regexall(pattern, string)
returns all matching substrings in an array

> replace("1 + 2 + 3", "+", "-")
1 - 2 - 3

> split(",", "foo,bar,baz")
[
  "foo",
  "bar",
  "baz",
]


strrev(string), reverses string

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

> basename("foo/bar/baz.txt")
baz.txt


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


---

expressions: This is the terraform language in practise really
terraform console

list type
map is a map

list must have all of the same type inside of it
tuples can have any mixture of types inside of it

multiline strings:
<<EOT
hello
world
EOT

EOT could be anything, it's your chosie

<<BEANS
hello
world
BEANS

<<THIS
THIS
this stuff is call Heredoc form

%{} allows for more complex shit

"Hello, %{ if var.name != "" }${var.name}%{ else }unnamed%{ endif }!"

"%{ for ip in aws_instance.example.*.private_ip }"

"%{ value ~ }" ~ strips the whitespace

path.module : returns the filesystem path of the module where the expression is placed.
path.root : returns  filesystem path of the root module of the configuration.
path.cwd : returns the filesystem path of the current working directory. In normal use of Terraform this is the same as path.root 
terraform.workspace : returns the name of the currently selected workspace.

//this looks dope
[for value in aws_instance.example: value.id] returns a list of all of the ids of each of the instances.

&& is terraform and

min(1,2,3) => call the min function with 1,2,3
min([1,2,3]...) => expanding function arguments. Calls min with 1,2,3

ternary:
condition ? if_true : else_false


[for s in var.list : upper(s)]

The brackets around a for determines the returned type
[ this is a list ]
{ this is an object }


{for s in var.list : s => upper(s)}

{for key in var.list : key => upper(key)}

//you can filter the list with an if
[for s in var.list : upper(s) if s != ""]


//you can do a for, each key value pair in a map
[for k, v in var.map : length(k) + length(v)]


//s... groups together results that have the same key, in the resulting map
{for s in var.list : substr(s, 0, 1) => s... if s != ""}

A splat is another name for the expressions that describe common for statements:
var.list[*].id
is just a disguised for statement

var.list.*.id is legacy, do var.list[*].id

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

The any statement acts like c#'s var
list(any) states that you've got a list of a type to be decided later

version contraints:

version = ">= 1.2.0, < 2.0.0"

---

output:
arguments:
description - string 
sensitive
depends_on

sensitive = true, suppresses this

---

variable arguments:
default : declare a default value for a variable : Allows the option to be optional on a module level
type    : declare the type of a variable
description : custom description of a variable
validation : A block for defining validation rules
sensitive :  Limits Terraform UI output when the variable is used in configuration. 

variable types:
basic types:
    string
    number
    bool

more types:
    list(<type>)
    set(<type>)
    map(<type>)
    object({ <name> = <value> })
    tuple([<type>])

validation:
variable "image_id" {
  type        = string
  description = "The id of the machine image (AMI) to use for the server."

  validation {
    condition     = length(var.image_id) > 4 && substr(var.image_id, 0, 4) == "ami-"
    error_message = "The image_id value must be a valid AMI id, starting with \"ami-\"."
  }
}

sensitive:
if sensitive = true, terraform suppresses ui output that contains the value


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

Terraform can also get vars from the environment:
$ export TF_VAR_image_id=ami-abc123
$ terraform plan

----

resource blocks can use the depends_on meta argument
if a resource block uses the depends on meta argument, reading of it's data source will be delayed

resource blocks can use count and for_each too

resource "my" "resource"{
    lifecycle {
        create_before_destroy = true //when doing apply, create the new resource before destroying the old one
        prevent_destroy = true //prevent any deletion of a particular object
        ignore_changes = true //prevent a resource from updating, when related data changes
    }
}

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
