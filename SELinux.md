
Fix `~/.ssh/authorized_keys`:

```
semanage fcontext -a -t sshd_exec_t ~/.ssh/authorized_keys
restorecon -v ~/.ssh/authorized_keys
```
