# Spoilers
Basically the "how to do" of the exercises stated here.

## 1) Create a local file

resource "local_file" "var_name" {
    filename = "path"
}

## 2) Create a 


variable "content"{
    type = "string"
}

resource "local_file" "second" {
    filename = "local_file_kata/second.txt"
    content = var.content
}


## 3) Use the contents of another file

data "local_file" "input_file"{
    filename = "resources/template.txt"
}

resource "local_file" "second" {
    filename = "local_file_kata/second.txt"
    content = data.local_file.input_file.content
}
