---
layout: post
title: Test Driven Development (TDD) for networks,  using Ansible
comments: true
---
In it's most basic form, TDD works as follows:

    1. Envision the outcome of what you are developing [ Adding a VLAN
       to a switch ]
    2. Write a test to validate that outcome [ Is the desired VLAN on
       the switch ]
    3. Run the playbook, to invoke the test [ it will fail - RED test
       ]
    4. Refactor your code [ write the roles/tasks needed for the tests
       to pass ]
    5. Re-run tests [role] to ensure that they now pass


In this post, I hope to share my observations on how I used ansible to
implement this using roles. I start with known facts/assumptions:

<!--more-->

*ASSUMPTION:* Given that the VLAN to be added is 101, it should be
applied to devices ```nxos-spine1 & nxos-spine2``` and stp priority
set as ```4096``` and ```8192``` respectively.

Following the basic rules listed out at the start of this post let's
translate that to ansible code:

(Only snippets are on the blog, for the full playbook, please see [repo](https://github.com/termlen0/TDD_ansible))

_Though the focus of this post is to demonstrate TDD methodologies -  and
how we might use them in developing network configurations -  it also
introduces stratergies around using roles, tags and the *reusability*
of roles._

*STEP1*: Start with a playbook that calls a role named validation

``` yaml
- assert:
    that:
      - " {{ "{{ desired_vlan" }} }} in  vlan_list "
```

``` diff
TASK [validate : assert] *******************************************************
- fatal: [nxos-spine1]: FAILED! => {
-    "assertion": "'101' in vlan_list ",
-    "changed": false,
-    "evaluated_to": false,
-    "failed": true
- }

```

At this point, our playbook dir looks as follows

``` shell
.
├── inventory
├── pb.yaml
└── roles
    └── validate
        └── tasks
            ├── main.yaml
            └── vlantest.yaml

3 directories, 4 files

```



*STEP2*: Now, refactor the playbook to add the VLAN  to the switches

``` diff
TASK [add_vlan : Add the desired vlan to the switch] ***************************
+ ok: [nxos-spine1]
```

And our playbook at this point has the following structure

``` shell
.
├── inventory
├── pb.yaml
└── roles
    ├── add_vlan
    │   └── tasks
    │       └── main.yaml
    └── validate
        └── tasks
            ├── main.yaml
            └── vlantest.yaml

5 directories, 5 files

```


*STEP3*: Now, rerun the playbook, with the validation role and observe
that all tests pass, in other words, the "GREEN" test.

``` diff
TASK [validate : assert] *******************************************************
+ ok: [nxos-spine1] => {
+     "changed": false,
+     "msg": "All assertions passed"
+ }
```


Note: Here is how I am invoking my playbook for each step

*STEP1*: ```ansible-playbook -i inventory pb.yaml  --tags=test```

*STEP2*: ```ansible-playbook -i inventory pb.yaml  --tags=add```

*STEP3*: ```ansible-playbook -i inventory pb.yaml  --tags=test```

The playbook uses 2 roles that have tags associated with them

``` yaml
roles:

    - role: validate
      tags: test
    - role: add_vlan
      tags: add

```


Also, pay attention to how the ``` main.yaml``` in the validate role
is including all other yaml files that match *test.yaml. The
implication is, that as your playbook scope expands, you can continue
writing simple, units of test code and name them to match *test.yaml
(for instance bgptest.yaml etc). If your team collects unit tests at a
particular location, you now have a _reusable_ unit of code, allowing
you to reuse this role in any playbook

```yaml

- name: Include all the test files
  include: " {{ "{{ outer_item" }}  }} "
  with_fileglob:
    - "/vagrant/dhcp_helper/roles/validate/tasks/*test.yaml"
  loop_control:
    loop_var: outer_item

```

We are using a loop to include all the test files and we are replacing
the standard *\{\{ item \}\}* var with *\{\{ outer_item \}\}* - Why?
If the unit test files you are including through this loop themselves
contain a loop, the value of *\{\{ item \}\}* main will be passed into
the unit test yaml. So we are explicitly setting the loop control
variable in main to yaml to *\{\{ outer_yaml \}\}*
