---
layout: post
title:  "Script to create Logic Hook files (before_save) for the given module"
date:   2019-10-15 17:00:00 +0500
categories: sugarcrm
---
This is a Python script, that will take a few inputs and create logic hooks files in extension framework. This is just to make the logic hooks easier for the first time. More scripts will be coming... ;)

### Script
```python
"""
For creating a hook in SugarCRM.

[description]

Variables:
    directory: str {[type]} -- [description]
    module: str {[type]} -- [description]
    file_name: str {[type]} -- [description]
    class_name: str {[type]} -- [description]
    try: {[type]} -- [description]
    except FileExistsError: {[type]} -- [description]
"""
import os

# For creating a Hook for a module
directory: str = input("Sugar Directory (eg. /var/www/html): ")
module: str = input("Module name: ")
file_name: str = input("File name: ")
class_name: str = input("Class name: ")

# Declaring function to create a directory if not exists
def make_dir(directory: str) -> bool:
    """
    For making a directory for the path passed into argument.

    Arguments:
        directory {str} -- The directory to be created

    Returns:
        bool -- Whether the directory was created successfully or not
    """
    try:
        os.makedirs(directory)
        print("Directory {} created".format(directory))
        return True
    except FileExistsError:
        print("Directory {} already exists".format(directory))
        return False

# Create files for the hook and class in module directory
make_dir(directory + "/custom/Extension/modules/" + module + "/Ext/LogicHooks/")
make_dir(directory + "/custom/modules/" + module)

# The paths where files will be stored
file_path: str = directory + "/custom/Extension/modules/" + module + "/Ext/LogicHooks/" + file_name
class_path: str = directory + "/custom/modules/{module}/{class_name}.php".format(module=module, class_name=class_name)

# Content of the hook file that registers the hook
hook_file_content: str = """<?php

if (!isset($hook_array['before_save'])) {{
    $hook_array['before_save'] = [];
}}

$hook_array['before_save'][] = array(
    count($hook_array['before_save']),
    'Before save logic hook class',
    'custom/modules/{module}/{class_name}.php',
    '{class_name}',
    'beforeSave'
);
""".format(module=module, class_name=class_name)


# Content of the class file that contains the hook function
class_file_content: str = """<?php

/**
 * This class will have logic hook functions of the module {module}
 */
class {class_name}
{{
    /**
     * Before Save hooks will come in this function
     *
     * @param  SugarBean $bean  Sugar Bean object of Email_Signature
     * @param  string    $event Current event which is before_save in this case
     * @param  array     $args  Additional arguments
     * @return           Will try to return some boolean
     */
    public function beforeSave($bean, $event, $args)
    {{
        // Functions or functionality comes here
    }}
}}
""".format(class_name=class_name, module=module)

file = open(file_path, "w")
file.write(hook_file_content)
file.close()

file = open(class_path, "w")
file.write(class_file_content)
file.close()
```
