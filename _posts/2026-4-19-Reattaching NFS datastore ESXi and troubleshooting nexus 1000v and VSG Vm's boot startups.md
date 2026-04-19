- ## ESXi NFS Datastore Reattach + Cisco Nexus 1000V / VSG Recovery
	- **Date:** 2026-04-19
	- **Status:** Resolved
	- **Environment:** VMware ESXi host with NFS-backed datastore and legacy Cisco virtual appliances
 - Note: Back to monthly GitHub uploads due to the HVAC being fully operational and having been serviced.
   The issue was basically that the reversing valve got stuck during the switchover and had to be freed up.
   Now that the HVAC system is in working order, I shall continue to monitor just in case, for all that I am thankful. 

- ## Summary
	- Reattached an NFS datastore to an ESXi host after the datastore had been removed and remounted.
	- After re-mount, all previously registered VMs showed as **Invalid** because ESXi had assigned the datastore a new internal UUID.
	- Re-registered each VM manually from its `.vmx` file.
	- Cisco Nexus 1000V and VSG appliances then had boot issues:
		- Nexus 1000V stalled at loader stage / boot shell
		- VSG booted inconsistently and required boot variable verification
	- Final resolution was:
		- re-register VMs
		- validate VMX / disk consistency
		- restore expected virtual NIC layout
		- manually load the correct image from boot shell
		- re-save boot variables in NX-OS / VSG CLI

- ## Symptoms
	- NFS datastore successfully mounted, but VMs appeared as:
		- `Invalid`
	- `vim-cmd vmsvc/getallvms` showed only skipped invalid VM entries.
	- Nexus 1000V initially hung around:
		- `Loader Loading stage1.5`
		- `Loader loading, please wait...`
	- After recovery progress, Nexus dropped into:
		- `switch(boot)#`
	- VSG eventually booted, but needed boot variables confirmed and saved.

- ## Root Cause
	- Removing and re-mounting the NFS datastore caused ESXi to assign a new datastore UUID.
	- Existing VM inventory entries still referenced the old datastore path, so ESXi marked them invalid.
	- Additional Nexus boot problems were caused by:
		- lost / stale boot variables
		- temporary NIC changes during troubleshooting
		- legacy appliance sensitivity to VM hardware presentation

- ## Recovery Steps
	- ### 1. Remove stale VM registrations
	- Unregistered all invalid VM inventory entries:
		- `vim-cmd vmsvc/unregister <vmid>`

	- ### 2. Confirm datastore contents
	- Verified datastore symlink and actual mounted UUID path under:
		- `/vmfs/volumes/`
	- Confirmed VM folders and `.vmx` files existed on the mounted NFS datastore.

	- ### 3. Re-register VMs
	- Located VMX files:
		- `find /vmfs/volumes/<datastore> -maxdepth 2 -name "*.vmx"`
	- Re-registered each VM:
		- `vim-cmd solo/registervm "<path-to-vmx>"`

	- ### 4. Verify inventory recovery
	- Confirmed all VMs returned to normal inventory:
		- `vim-cmd vmsvc/getallvms`

- ## Nexus 1000V Troubleshooting
	- ### Initial checks
	- Verified:
		- VMDK existed
		- descriptor file was valid
		- disk chain was consistent
		- ESXi could see and access the virtual disk
	- Confirmed issue was not missing storage.

	- ### VMX adjustments tested
	- Backed up the VMX.
	- Forced legacy-friendly settings:
		- BIOS firmware
		- reduced CPU / memory temporarily
		- removed extra PCIe bridge entries
		- kept E1000 adapters
	- These changes helped move the appliance past the initial loader hang.

	- ### Key breakthrough
	- Once the VM reached `switch(boot)#`, checked available files with:
		- `dir`
	- Found boot images on internal storage.
	- Manual boot required `load`, not `boot`.
	- Successfully launched image with:
		- `load bootflash:<kickstart-image>.bin`

	- ### NIC issue found
	- Nexus boot output reported:
		- `Unable to detect three e1000 ethernet devices`
	- Cause:
		- two NICs had been disabled during troubleshooting
	- Fix:
		- restored all required E1000 adapters in the VM configuration

	- ### Permanent fix
	- After full boot, re-saved boot variables:
		- `boot kickstart ...`
		- `boot system ...`
		- `copy running-config startup-config`

- ## VSG Recovery
	- VSG eventually progressed into system admin account setup and completed boot.
	- Verified redundancy state:
		- standalone / active
		- no standby supervisor present
	- Confirmed saved boot variables with:
		- `show boot`
	- Boot variables were correctly set for both current and next reload.

- ## Validation
	- Verified:
		- datastore mounted correctly
		- VM inventory restored
		- Nexus 1000V booted to CLI
		- VSG booted to CLI
		- boot variables saved
		- supervisor / redundancy status healthy for standalone lab deployment

- ## Commands Used
	- ```bash
	  vim-cmd vmsvc/getallvms
	  vim-cmd vmsvc/unregister <vmid>
	  find /vmfs/volumes/<datastore> -maxdepth 2 -name "*.vmx"
	  vim-cmd solo/registervm "<path-to-vmx>"
	  esxcli storage nfs list
	  vmkfstools -e "<disk>.vmdk"
	  tail -200 "<vmware.log>"
	  ```
	- ```text
	  dir
	  load bootflash:<image>.bin
	  show boot
	  show system redundancy status
	  copy running-config startup-config
	  ```

- ## Lessons Learned
	- Re-mounting an NFS datastore can leave VM registrations pointing at stale datastore UUID paths.
	- Legacy appliances may not fail because of storage at all; they may fail because boot variables or expected virtual hardware changed.
	- Cisco Nexus virtual appliances are very sensitive to:
		- expected NIC count
		- legacy boot state
		- boot image paths
	- Once the guest reaches the boot shell, progress is real. At that point the issue is usually guest boot logic, not ESXi storage.

- ## GitHub-safe Notes
	- All hostnames, datastore identifiers, IP addresses, and environment-specific labels have been redacted or generalized.
	- This write-up is intended as a public troubleshooting summary, not a full environment export.
