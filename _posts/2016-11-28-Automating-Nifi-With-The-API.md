---
layout: single
title: Automating Nifi with the API
---

# Introduction
This post discusses how to use the [Apache Nifi Rest API version 1.0.0](https://nifi.apache.org/docs/nifi-docs/rest-api/) 
to automate some tasks like importing and instantiating a template to a running instance, starting and stopping processors
and updating configuration properties for a processor. I did create a library for basic http get, post, put, delete commands.
You can read through that in {% post_url 2016-11-28-Nifi-Api-Python-Library %}.

Note: I've been a Java programmer for over 15 years and all of these examples are in Python so I'm sure there are better
ways to do things syntactically than how I do it.

Also worth noting is that all of the templates I have created consist of an outer process group that contains the entire
workflow for what I'm doing. For example, if I have a workflow that is querying a REST service for data, I will first define
a process group named for the service I'm getting data from. Then inside that process group I'll have all the processors
for calling InvokeHttp and saving the data to whatever storage medium. If you like adding workflows directly to the
root process group then the below examples will vary slightly for you.

# Importing a Template
If you are like me and don't like doing anything manually, you have a process for autogenerating an XML template file that
can be used by Apache Nifi. For this example, let's say we want to import one of the [sample templates](https://cwiki.apache.org/confluence/display/NIFI/Example+Dataflow+Templates).
The basic steps for installing this template are:

* Load the XML template file into memory.
* If a template with that name already exists on the destination Nifi server, remove it.
* Upload the new template.
* Instantiate it.

To load the template into memory, I imported xml.etree.ElementTree library. Then I grab the name of the process group.
```python
import xml.etree.ElementTree as ET

tree = ET.parse(file)

template_name_elem = root.find('name')
template_name = template_name_elem.text
```

Now that we have the name of the template, let's query the Nifi server for the list of templates and iterate over the results
to see if our template already exists. I created a helper function for this:

```python
    def get_remote_template(self, name):
        template = self.remote_get('/flow/templates', None)
        if template is None:
            return None
        for t in template["templates"]:
            if t["template"]["name"] == name:
                return t
        return None
```

if it exists, we will need to delete it. 

```python
remote_template = self.get_remote_template(templ_name)
if remote_template is not None:
    response = self.remote_delete('/templates/', remote_template["id"])
```

Now that we have deleted the existing template, we can upload the new one.
```python
self.remote_post(self.url + '/process-groups/{}/templates/upload'.format(process_group_id),
                                filename,
                                'application/xml')
```

Now, let's instantiate the template
```python
self.remote_post_data('/process-groups/{}/template-instance'.format(process_group_id),
                                     instantiate_templ_req_entity)
```


