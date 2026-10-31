
<img width="330" height="407" alt="Ansible_logo svg" src="https://github.com/user-attachments/assets/50c2c866-f6d8-4ef3-8106-3f726d49e64b" />

From theory to action; configuring ansible, defining running and troubleshooting playbooks:
NOTES:
- **Automation: Ansible**
    - **Components**
        - **Ansible Core (ENGINE)**
            
            fundementals, cmd, engine, functions
            
        - **Automation Content Navigator (ENGINE WRAPPER)**
            - wraps Core in an Execution Environment (image)
                - Ansible Core, Ansible Content Collections, and any Python libraries, executables, or other dependencies needed to run your playbook.] ⇒ interoperability
                - `ansible-builder`
            - `ansible-navigator`replaces other commands → one
    - **Characteristics**
        - **Role**: deploy infra, config, dependancies…
        - **Agentless**: only installed on Unix master node
        - **Master Unix node** (Ansible) → sends **PLaybook** of **plays**/configurations to all win/lx **managed/inventory hosts** via SSH (or other specified mthd)
            - can execute a section of playbook based on TAGs
        - idempotence
            
            don’t modify if don
            
    - Commands
        
        `ansible-navigator run playbook.yml -i inventory.yml`
        
    - Config
        - `ansible.cfg` Ansible core engine config ← check with `ansible-navigator config`
            
            ```yaml
            [defaults]
            inventory = ./inventory
            remote_user = user  #used to connect to remote host
            ask_pass = false
            
            [privilege_escalation]
            become = true
            become_method = sudo  #needs sudo access: create a root owned file in    /etc/sudoers.d => %grp|usr ALL=(ALL)       ALL
            become_user = root
            become_ask_pass = true #false if NOPASSWD: ALL is set
            ```
            
        - Navigator wrapper settings ← check with`ansible-navigator settings`
            1. File specified by the `ANSIBLE_NAVIGATOR_CONFIG` env.
            2. `ansible-navigator.yml` in current dir
            3. `~/.ansible-navigator.yml`
            
            ```yaml
            pull:
                  policy: missing #when to pull? when img is missing
            ```
            
            ⇒`ansible-navigator images`
            
        - copy sshkeys to multiple hosts using a play
        - if a play needs input like a password: enable stdout mode `-m stdout`and disable interactive mode forced by artefacts`-pae false`
        
        `site.yml` playbook (YAML files)
        
        - `-e inventory.in` | `inventory.yml` | `/etc/ansble/hosts` (dflt)
            
            `ansible-navigator inventory -i inventory \
            > -m stdout --list`
            
            > `[all]                #default groups
            [ungrouped]
            
            [webservers]
            web1.example.com
            web2.example.com
            192.0.2.42
            2001:db8::[a:f]
            
            [db-servers]
            db1.example.com
            192.168.[4:7].[0:255]
            
            [servers:children] #group groups
            db-servers
            webservers`
            > 
        - `become password`connect pwd vs sudo pwd
