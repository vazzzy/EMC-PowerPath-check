# EMC-PowerPath-check
Powershell script to check EMC powerpaths of LUNs from EMC Storage arrays for 1000s of host at the same time. Hosts limited to Windows Physical. 

$rs = Get-Content C:\Users\svaseem\Desktop\hostname.txt
foreach($s in $rs){
    try{

        Invoke-Command -computerName $s -ScriptBlock { powermt display dev=all | out-file path_status.txt} -ErrorAction Stop
        Copy-Item "\\$s\c$\Users\svaseem-a\Documents\path_status.txt" -Destination "C:\Users\svaseem\Desktop\Path_files\paths_$s.txt"
        Write-Host "The file has been copied to your computer from server: $s" -ForegroundColor green 

        $f=Get-Content C:\Users\svaseem\Desktop\Path_files\paths_$s.txt
        if(( $f | Select-String -pattern "dead") -or ($f | Select-String -pattern "degraded")){
            Write-Host "There are Dead or Degraded paths for host $s, hence check it's respective file for more details." -ForegroundColor Yellow
        }
        else{
            Write-Host "There are no dead or degraded paths for host $s"
        }
        $a = Get-Content C:\Users\svaseem\Desktop\Path_files\paths_$s.txt
        $i = 0;
        if (Invoke-Command -computerName $s -ScriptBlock { powermt version} | Select-String -SimpleMatch 6.){
            while( [array]::IndexOf($a.trimend(' '),'', $i) -gt 0){
                $j = [array]::IndexOf($a.trimend(' '),'', $i) 
                if(($j - $i) -eq 10){
                    write-host "$s has single path. Please check respective file for more details." -ForegroundColor Cyan
                }
                $i = $j + 1
            } 
        }
        else{
            while( [array]::IndexOf($a.trimend(' '),'', $i) -gt 0){
                $j = [array]::IndexOf($a.trimend(' '),'', $i) 
                if(($j - $i) -eq 9){
                    write-host "$s has single path. Please check respective file for more details." -ForegroundColor Cyan
                }
                $i = $j + 1
            } 
        }
    }
    catch{
        Write-Host "$s doent have Remote services enabled and permissions, hence script will not execute for this server." -ForegroundColor Red
    }
}
