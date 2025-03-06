$autologon = <<-SCRIPT
Set-ItemProperty -Path "HKLM:/SOFTWARE/Microsoft/Windows NT/CurrentVersion\\Winlogon" -Name "AutoAdminLogon" -Value "1"
wget https://download.sysinternals.com/files/AutoLogon.zip -O C:\\ProgramData\\AutoLogon.zip
Expand-Archive C:\\ProgramData\\AutoLogon.zip -DestinationPath C:\\ProgramData\\AutoLogon
C:/ProgramData/AutoLogon/Autologon64.exe "vagrant" "FLAREVM" "vagrant" /accepteula
SCRIPT

$disable_defender = <<-SCRIPT 
wget https://github.com/ionuttbara/windows-defender-remover/archive/refs/heads/main.zip -O C:\\ProgramData\\main.zip
Expand-Archive C:\\ProgramData\\main.zip -DestinationPath C:\\ProgramData
C:\\ProgramData\\windows-defender-remover-main\\Script_Run.bat y
SCRIPT

$script = <<-SCRIPT
Invoke-WebRequest -Uri "https://raw.githubusercontent.com/mandiant/flare-vm/refs/heads/main/install.ps1" -OutFile "$env:USERPROFILE\\Desktop\\install.ps1"
Invoke-WebRequest -Uri "https://raw.githubusercontent.com/mandiant/flare-vm/refs/heads/main/config.xml" -OutFile "$env:USERPROFILE\\Desktop\\config.xml"
Unblock-File "$env:USERPROFILE\\Desktop\\install.ps1"
Set-ExecutionPolicy Unrestricted -Scope Process -Force
Set-ExecutionPolicy Unrestricted -Scope CurrentUser -Force
Set-ExecutionPolicy Unrestricted -Scope LocalMachine -Force
&"$env:USERPROFILE\\Desktop\\install.ps1" -customConfig  "$env:USERPROFILE\\Desktop\\config.xml" -password vagrant -noWait -noGui -noChecks
SCRIPT

Vagrant.configure("2") do |config|
  config.vm.box = "gusztavvargadr/windows-10"
  config.vm.hostname = "FlareVM"
  config.vm.synced_folder '.', '/vagrant', disabled: true
  
  config.vm.provider "hyperv" do |vb|
    vb.memory = 5000
    vb.cpus = 4
  end
  
  # Debugged using https://stackoverflow.com/questions/49547740/what-does-this-vagrant-error-mean-and-how-do-you-fix-it-for-public-network-an
  config.vm.network "private_network", type: "dhcp", netmask: "255.255.255.0", dhcp_ip:"192.168.56.100", dhcp_lower: "192.168.56.101", :dhcp_upper=>"192.168.56.254"
  
  # autologon should allow the scripts to automatically continue after reboot
  # without user involvement
  config.vm.provision "autologon", type: "shell", privileged: true do |s|
    s.inline = $autologon
  end
  
  config.vm.provision "disable-defender", type: "shell", privileged: true do |s|
    s.inline = $disable_defender
  end
  
  config.vm.provision "set-autologon-key", type: "shell", privileged: true, run: "never" do |s|
    s.inline ='Set-ItemProperty -Path "HKLM:/SOFTWARE/Microsoft/Windows NT/CurrentVersion\\Winlogon" -Name "AutoAdminLogon" -Value "1"; C:/ProgramData/AutoLogon/Autologon64.exe "vagrant" "FLAREVM" "vagrant" /accepteula; Restart-Computer -Force'
  end
  
  config.vm.provision "flare-install", type: "shell", privileged: true, run: "never" do |s|
    s.inline = $script
  end
end
