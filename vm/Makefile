# tinkering with a 'make vm' thinger

VM_IMG_FILE?= ec2-dev.vdi
VM_IMG_URL?= https://cdn.amazonlinux.com/os-images/2017.12.0.20180330/virtualbox/amzn2-virtualbox-2017.12.0.20180330-x86_64.xfs.gpt.vdi

install: get-image
	# copy VM files to build dir
	cp -R canonical build
	VBoxManage registervm $(PWD)/build/kitpress-dev.vbox
	
	# generate a new UUID for the HD img to prevent duplicate media errors
	VBoxManage internalcommands sethduuid $(PWD)/build/$(VM_IMG_FILE)
	# attach HD
	VBoxManage storageattach kitpress-dev --storagectl IDE --port 0 --device 0 --type hdd --medium $(PWD)/build/$(VM_IMG_FILE)
	
	# Construct seed iso from ec2-seed components
	hdiutil makehybrid -o $(PWD)/build/init.iso -hfs -joliet -iso -default-volume-name cidata ec2-seed
	VBoxManage storageattach kitpress-dev --storagectl IDE --port 1 --device 0 --type dvddrive --medium $(PWD)/build/init.iso
	
	# Set up shared folder for modules
	VBoxManage sharedfolder add kitpress-dev --name kitpress-modules --hostpath "$(shell dirname "$(PWD)")/modules" --automount
	# Start the VM
	VBoxManage startvm kitpress-dev
	
# Download the AMZ VDI if needed
get-image:
	@if [ ! -f "$(PWD)/canonical/$(VM_IMG_FILE)" ]; then \
	   curl $(VM_IMG_URL) -o $(PWD)/canonical/$(VM_IMG_FILE); \
	fi \
	
uninstall:
	-VBoxManage controlvm kitpress-dev poweroff
	sleep 1s
	-VBoxManage unregistervm $(PWD)/build/kitpress-dev.vbox
	rm -rf build