# ansible-for-python-haters
---
Ansible is written in python, but you can write modules in any language (thank god because some people hate python). Here are some examples in `bash` to get you started. 


## Documentation
---
If you would like to read more about ansible, visit [http://docs.ansible.com](http://docs.ansible.com).

The docs will help you understand how to use Ansible better, and about it's data-driven configuration language, which is YAML based. 

This repo is all about how to write Ansible modules in `bash`, rather than python and to show how it can be done. Did I mention some people hate python?

## Requirements
---
These examples assume you have ansible, a terminal, and something that is either linux of mac. Any `shell` should be fine, but I'll stick with `bash` since that is the default on the majority of linux/unix systems.

Be aware that you should ensure that any command you specify should be present on the system prior to running your new module. This can be done as an additional task in your playbook or role.

For instance lets say you need the `bc` command but aren't sure if this is installed on all your target systems.

```
- name: install bc from yum
  yum: name=bc state=installed
```

## My First Module
---

####*Where are modules stored?* 

This varies based on your platform and how ansible was installed. The main directory where modules are located in the source distribution from github is [./library](https://github.com/ansible/ansible/tree/devel/library). For me this is installed to `/usr/share/ansible`.


####*Module Input*

Ansible modules are copied over by ansible along with a arguments file that contains any arguments specified in your playbook/role or via command line using ansible.

There is no better way to illustrate then to create our first module. We will call this module **echo** and place it under the ansible library location, under the category of your module. As mentioned, for me this is `/usr/share/ansible`

```
$ cat /usr/share/ansible/system/echo 
#!/bin/sh

contents=`cat $@`
echo -n "{ \"input\": \"$@\",\"contents\":\"" $contents "\" }"
```

Next lets create a sample playbook to run it

```
$ cat foo.yml
---
- hosts: all 
  user: veewee
  sudo: True

  tasks:
    - name: echo input
      echo: myarg1=I myarg2=Hate myarg3=Python
```

So let's run it.

```
$ ansible-playbook -i inv foo.yml -vv

PLAY [all] ******************************************************************** 

GATHERING FACTS *************************************************************** 
<86.75.30.9> REMOTE_MODULE setup
ok: [86.75.30.9]

TASK: [echo input] ************************************************************ 
<86.75.30.9> REMOTE_MODULE echo myarg1=I myarg2=Hate myarg3=Python
ok: [86.75.30.9] => {"contents": " myarg1=I myarg2=Hate myarg3=Python ", "input": "/home/veewee/.ansible/tmp/ansible-tmp-1400427409.52-36850903270649/arguments"}

PLAY RECAP ******************************************************************** 
86.75.30.9              : ok=2    changed=0    unreachable=0    failed=0
```


####*Module Output*

Ansible modules return data by printing JSON to standard output. There is a caveat to how the JSON is returned, it **MUST be as a single line**. So remember your `echo -n` and `printf` statements to avoid adding `\n` or `\r`.

Returning `"failed": True` is the way to signal a failure to ansible. **When something fails, always return a `"msg"` attribute in the JSON that explains the failure.**

Returning `"changed": True` indicates something has changed.

Returning a key in the JSON called **ansible_facts** tells ansible to store some variables and make them available for later use. Again, any module can do this, and then those things are available to Ansible for use as variables later on down in the playbook, or in templates.

Take a look at the code in the repository and you will see all of this in play, as simple as possible, and can use this to get started writing your own modules in `bash`.


---
## Can I contribute my module to ansible-core?
---
**NO**, you are screwed since they won't include anything but python in their core distribution. Which I obviously disagree with. Hence the ansible-for-python-haters repo..