# qubes-sync-project

Scripts to sync a simple version-controlled project between VMs in Qubes OS.
Use one VM for git operations and another VM for development (including
installation of untrusted third party packages). I found these scripts useful
for small projects. I do not use them for bigger projects spanning multiple
repositories, where a separate per-project VM for each feels more appropriate.

Basically these scripts wrap the `qvm-copy` tool with `rsync --cvs-exclude` to
sync a project directory between Qubes VMs. As a result, `.git`, `.svn`,
`.hg`, ... directories and files in `.gitignore` are not synced between the VMs
(check `man rsync` for the full list of ignored files).


## Usage

Let's imagine two Qubes VMs called `project-dir-vm` and `dev-untrusted-vm`.
`project-dir-vm` is used for interaction with github, `dev-untrusted-vm` is
used for development (e.g.  where you install third party `npm` packages). We
could use the scripts from this repository like this:

1. Install `qvm-send-project` and `qvm-receive-project` in both of your VMs (or
   your TemplateVM) in a directory under PATH.

2. Send your project from `project-dir-vm` to `dev-untrusted-vm`:

   ```sh
   qvm-send-project dev-untrusted-vm ~/project-dir
   ```

3. Work with your project in `dev-untrusted-vm` and send the result back to
   `project-dir-vm`:

   ```sh
   cd ~/QubesIncoming/project-dir-vm/project-dir
   # Install untrusted packages
   npm install
   # Do some hacking
   vim ...
   # Send the project back to project-dir-vm VM:
   qvm-send-project project-dir-vm .
   ```

4. Make sure the changes are ok, commit and push in the `project-dir-vm`:

   ```sh
   # Go to the project directory
   cd ~/project-dir
   # Get the files from ~/QubesIncoming/dev-untrusted-vm, excluding the .git directory
   qvm-receive-project dev-untrusted-vm .
   # Check that the changes are as expected and push
   git diff
   git commit -a
   git push
   ```

Note: `qvm-send-project` and `qvm-receive-project` use the `qubes.Filecopy`
qubes-rpc call, which asks to confirm the VM name every time it is called. If
you trust file copy operations between `project-dir-vm` and
`dev-untrusted-vm`, you can tell Qubes to skip the confirmation dialog by
editing `/etc/qubes-rpc/policy/qubes.Filecopy` in `dom0` and adding these lines
at the top:
```
project-dir-vm dev-untrusted-vm allow
dev-untrusted-vm project-dir-vm allow
```
