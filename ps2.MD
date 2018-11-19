#Parameter setup

param([string] $u = '', 
      [string] $t = '')

#Set up context variable
$context = ''
$globalgroup = ''
             $secondglobal = '' 
             $localgroup = ""

#Check if user was entered and display error if blank



if($u -and $t )
{ 

Write-Host "both parameter proceeding for verifying these parameters "


}
elseif ($u -eq '' -and $t -eq '')
{
Write-Host "You must enter a user and tool they can not be blank"
Write-Host "use: ./test1.ps1 -u <username> -t <toolname eg jira> "

exit 

}

elseif ($u -eq '')
{        
        
    Write-Host "You must enter a user's name (-u). It cannot be blank"
    Write-Host
    Start-Sleep -s 1
   exit        
}

#Check if tool was entered and display error if not
elseif($t -eq '')
{
    Write-Host "You have not entered a tool. Please enter a tool with the parameter -t"
    Write-Host
    Start-Sleep -s 1
    exit    
}

else {
 Write-Host "unknow error" 
}


Write-Host "I am here" 



#Check if OU exists in context using distinguishedname and display error if not
$OU=Get-ADOrganizationalUnit -Filter * -SearchBase $context | Where-Object {$_.distinguishedname -like "OU=$t*"}
if($OU -eq $null)
    {
        Write-Host
        Write-Host "The parameter entered for -t does not exist under your main OU"
        Start-Sleep -s 2
        exit

}

#Check if user exists and display error if not (Allow double error display if necessary)
$user = Get-ADUser -Filter ("Name -like '*$u*'")
        if($user -eq $null)
        {
           Write-Host
           Write-Host "User does not exist with the name $u..." 
           Start-Sleep -s 2 
           exit

       }


#Locate groups and store group names
$g1 = Get-ADGroup -filter * -searchbase $context | Where-Object {$_.distinguishedname -like "CN=ABC*"}
$globalgroup = $g1.name
$pt = Get-ADGroup -filter * -searchbase $context | Where-Object {$_.distinguishedname -like "*particulartool*"}

$secondglobal = $pt.name
$localgroup = "$t.global.users"
             
             #Display samaccountname and check if user is in required groups
             $sam = $user.samaccountname
             Write-Host
             Write-Host "Samaccount name: $sam"
             $g = Get-ADPrincipalGroupMembership $user | Where-Object{($_.distinguishedname -like "*$context*" -or $_.distinguishedname -like "*$OU*") -and $_.distinguishedname -like "*$globalgroup*"}
             $s = Get-ADPrincipalGroupMembership $user | Where-Object{($_.distinguishedname -like "*$context*" -or $_.distinguishedname -like "*$OU*") -and $_.distinguishedname -like "*$secondglobal*"}
             $l = Get-ADPrincipalGroupMembership $user | Where-Object{($_.distinguishedname -like "*$context*" -or $_.distinguishedname -like "*$OU*") -and $_.distinguishedname -like "*$localgroup*"}

             
             #Add user to groups if needed and display status message
             if(($g -ne $null -or $s -ne $null) -and $l -ne $null)
             {
                if($g -ne $null -and $s -ne $null)
                {
                    Write-Host
                    Write-Host "$sam is already in $globalgroup, $secondglobal and $localgroup so no change was made"
                    Start-Sleep -s 2 
                }
                else
                {
                    if($g -ne $null)
                    {
                        Write-Host
                        Write-Host "$sam is already in $globalgroup and $localgroup so no change was made"
                        Start-Sleep -s 2 
                    }
                    else
                    {
                        Write-Host
                        Write-Host "$sam is already in $secondglobal and $localgroup so no change was made"
                        Start-Sleep -s 2
                    }
                }                
             }
             else
             {
                if($g -ne $null)
                {
                    Write-Host
                    Write-Host "$sam is in $globalgroup"
                    Start-Sleep -s 1  
                }
                if($s -ne $null)
                {
                    Write-Host
                    Write-Host "$sam is in $secondglobal"
                    Start-Sleep -s 1  
                }
                if($l -ne $null)
                {
                    Write-Host
                    Write-Host "$sam is in $localgroup"
                    Start-Sleep -s 1
                }
             }
             if($g -eq $null -and $s -eq $null)
             {
                $group = Get-ADGroup -Filter ("Name -like '$globalgroup'")
                Add-ADGroupMember $group -Members $user
                Write-Host
                Write-Host "$sam has been added to $globalgroup"
                Start-Sleep -s 1
             }
             if($l -eq $null)
             {
                $group = Get-ADGroup -Filter ("Name -like '$localgroup'")
                Add-ADGroupMember $group -Members $user
                Write-Host
                Write-Host "$sam has been added to $localgroup"
                Start-Sleep -s 1
             }
             Start-Sleep -s 2  
        }
    }

