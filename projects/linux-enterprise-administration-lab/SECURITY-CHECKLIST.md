# Linux Administration Security Checklist

## Identity
- [ ] Unique administrator account
- [ ] Root not used for routine work
- [ ] Sudo access limited
- [ ] Unused accounts reviewed/locked
- [ ] Service accounts reviewed

## Permissions
- [ ] Sensitive files use restrictive permissions
- [ ] Shared directories use groups/ACLs
- [ ] No unnecessary 777 permissions
- [ ] Ownership verified
- [ ] Sudo configuration validated with visudo

## Packages
- [ ] Package metadata updated
- [ ] Security updates applied
- [ ] Unnecessary packages removed
- [ ] Trusted repositories only

## Services & Network
- [ ] Failed services reviewed
- [ ] Unnecessary services disabled
- [ ] Listening ports inventoried
- [ ] Firewall policy documented
- [ ] Management access restricted

## SSH
- [ ] Key authentication tested where appropriate
- [ ] Private keys protected and never committed
- [ ] Host keys verified
- [ ] Configuration changes validated before restart
- [ ] Recovery session retained during changes

## Backup
- [ ] Important data identified
- [ ] Backup created
- [ ] Backup integrity checked
- [ ] Restore tested
- [ ] Backup access protected
- [ ] Retention documented

## Monitoring
- [ ] CPU checked
- [ ] Memory checked
- [ ] Disk and inode usage checked
- [ ] Failed services checked
- [ ] Network state checked
- [ ] Health script returns meaningful exit code

## GitHub hygiene
- [ ] No passwords
- [ ] No private keys
- [ ] No API tokens
- [ ] No sensitive configuration
- [ ] Evidence reviewed before commit
