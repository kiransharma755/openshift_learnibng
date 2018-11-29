#========================================

param([string] $user = $null,
      [string] $tool=$null,
      [string] $globslUser = $null,
      [string] $accountType = $null,
      [string] $file = $null 
      )


Find out user does not enter other options 
Find out # of parameters and their list and value 
print the above foudnign in screen 
processing ...............


checking parameters values

Accesstype 

1. The value for accountType could be 
1. g/G or general 
2. P/p or project
3. t/T or tool 

if accesstype is blank
message, it can not be blank
exit
endif 



user
====
users, single email, name and email separated by ; or ,
find way to greap value inside <>, which is an email 

globalUser
==========
globalUser  can be either of two ABC.users or def.global.users if none of them, error messages 

tool
====
check against the OU name under ABC
if not give error message and exit



==================
1. 
if accesstype is general
if globaluser is not provided, 
message, globaluser type is needed
check these are 1. abc.user, def.global.users

if u and f, message can not exist both 
if f but not file, 
look for gaccess.txt file,
if does not exist, 
message file does not exist
exit
endif


if users list, separated by ; assumiong it is lastname, first name mi. <email>

2.
if accesstype if tool
if tool is blank, message, it can not be blank


3. 
if accesstype is project

if p is blank, 
message p can not be blank

if $tool is blank, message, tool is necessary 
endif
endif




# ---
1

if accesstype is generalaccess
if -file 
look for gaccess.txt
if not found, 
message file not found 
file does not exist, gaccess.txt 

2
if accesstype is project 
if -file 
look for paccess.txt
if not found, 
message file not found 
file does not exist, paccess.txt 

3
if accesstype is tool 
if -file 
look for taccess.txt
if not found, 
message file not found 
file does not exist, taccess.txt 


4. check projectr has multiple value separated by ; or ,
in this case, the given users has be to be added to all projects

---
exit

endif






=============




#=============================================
$separator = ";"
$separator2 = "<>"

$group=$("$t.$project.users")
$group=$("$t.$project.admins")


#==================================================

$bb="gitbucket"
$GDE="GDE"

$globalgrpexist=""
$localgrpexist=""
      
$bbcn="bitbucket.global.users"
$GDEcn="GDE.users"


$context="OU=GDE,OU="

$gitbuckettgrp=Get-ADGroup -Filter  ( 'Name -like "gitbuckett.global.users"') -SearchBase $context
$GDEusersgrp=Get-ADGroup -Filter  ( 'Name -like "GDE.users"') -SearchBase $context

$bbGrpName=$gitbuckettgrp.name
$GDEGrpNamename=$GDEusersgrp.name


Write-Host "checking whether the user [$user] exist either in" $bbname " or" $GDEname
Write-Host ".........."



if ($globslUser.equals($bb))
{
$bbmembership=Get-ADPrincipalGroupMembership $user |Where-Object{$_.distinguishedname -like "*$context*"  -and $_.distinguishedname -like "CN=gitbuckett.global.users,*"}

if ($bbmembership)

{
#Write-Host "The user is aleady in"  $gitbuckettgrp.name "group so no further action is needed"
$globalgrpexist=$gitbuckettgrp.name

}
else
{
#Add-ADGroupMember -Identity $gitbuckettgrp -Members  $u
Write-Host "The user is added to $gitbuckettgrp.name"
Write-Host

}
}

elseif ($globslUser.equals($GDE))
{

$GDEmembership=Get-ADPrincipalGroupMembership $user |Where-Object{$_.distinguishedname -like "*$context*"  -and $_.distinguishedname -like "CN=GDE.users,*"}


if ($GDEmembership)
{
#Write-Host "The user is aleady in  " $GDEusersgrp.name "so no action needed"
$globalgrpexist=$GDEusergrp.name


}
else
{
Add-ADGroupMember -Identity $GDEusersgrp -Members  $u
Write-Host "The user is added to" $GDEusersgrp.name

}
}

else
{
Write-Host "The globaluser group is wrong"

}

Write-Host
Write-Host
Write-Host "checking whether the users exist in $t.global.users group  !!............."
Write-Host
Write-Host




$toolmembership=Get-ADPrincipalGroupMembership $user |Where-Object{$_.distinguishedname -like "*$context*"  -and $_.distinguishedname -like "CN=$t.global.users,*"}


if ($toolmembership)
{
#Write-Host  "The user is aleady in" $toolmembership.name " No additional action is neede " 
$localgrpexist=$toolmembership.name
}
else
{
#Add-ADGroupMember -Identity $toolmembership  -Members  $u
Write-Host "The user is added to" $toolmembership.name
}





if ($globalgrpexist)
{

if ($localgrpexist)

{ Write-Host "The user exist in both global  group" $globalgrpexist "and localgroup" $localgrpexist " so no any additonal action is needed"}
else
{Write-Host "The user exist in global group" $globalgrpexist "but not in local group,  so added to local group" $localgrpexist
}

}
else

{
   
if ($localgrpexist)
{ Write-Host "The user does not exist in  global  groups" $globalgrpexist "but exist in localgroup" $localgrpexist " so added to global group"}
else
{Write-Host "The user does not exist in either global group" $globalgrpexist "or in local group," $localgrpexist "so added to both groups"
}

}



