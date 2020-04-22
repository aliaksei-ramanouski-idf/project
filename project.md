Project
----
Project's reporter: *Ramanouski Aliaksei*

Group number: *m-sa2-12-20*

Description of application for deployment
----
|node-todo|
| :------------ |
|[link](https://github.com/manlyalex/node-todo)|
It's project written in Java Script programming language using Angular framework + MongoDB.
![Sheme](https://github.com/manlyalex/project/blob/master/sheme-03.jpg)

Technologies which were used in project
----
Orchestration: Jenkins, git.

Automation tools: Ansible.

#### CI description:

Push(Developer) -> Run pipeline -> Git clone(Jenkins) -> Testing process -> Create new version -> Publish -> Slack notification

#### CD description:

Git clone (new version or manually selected version) -> Npm install -> Artifacts copying and systemd restart -> Slack notification

[Environment-setup-ansible](https://github.com/manlyalex/project/tree/master/env.setup)

[Pipeline-CI](https://github.com/manlyalex/project/blob/master/01.jenkinsfile)

[Pipeline-CD](https://github.com/manlyalex/project/blob/master/02.jenkinsfile)

https://github.com/manlyalex/project/blob/master/Ramanouski%20Aliaksei.pdf)
